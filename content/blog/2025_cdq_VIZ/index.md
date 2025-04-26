---
title: 'Aminated Visualisation for Centre-du-Québec’s Precipitation'
author: Johanie Fournier, agr., M.Sc.
date: "2025-01-31"
slug: cdq_precipitation_VIZ
categories:
  - rstats
  - tidymodels
  - viz
tags:
  - rstats
  - tidymodels
  - viz
summary: "Building upon previous analyses and predictive modeling, I details the process of creating this visualization, including data preparation, disaggregation to daily levels, and kriging for enhanced spatial resolution. The post culminates in an animated map that illustrates precipitation trends and anomalies over time, providing valuable insights for climate analysis, agriculture, and water resource management."
editor_options: 
  chunk_output_type: inline
adsense:
  publisher-id: ca-pub-7674504334497845
filters:
- adsense
---

<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-7674504334497845" crossorigin="anonymous"></script>

<a href = "https://subscribepage.io/E3ia1B"> ![](petit.png)
</a>

<br>

Understanding precipitation patterns is essential for climate analysis, agriculture, and water resource management. In previous blog posts, we conducted an exploratory data analysis (EDA) to uncover key trends, spatial distributions, and relationships within the dataset. Then build an XGBoost model to predict precipitation using historical climate data. In this post, we will create an animated visualization of future precipitation predictions to better understand the trends and anomalies in the data.

# Get the Data

``` r
# Create raster from predicted data
year=1993:2023
for (y in year){

data<-readRDS("Data/data_spatial.rds") |>
  filter(year %in% y) |>
  select(-year) 

raster<-data|> 
  terra::vect() |> 
  terra::rast()

terra::values(raster) <- data$value

writeRaster(raster, paste0("Data/precipitation_",y,".tif"), overwrite=TRUE)
}

# Create a raster stack
rastlist <- list.files(path = "Data/", pattern='.tif', 
all.files=TRUE, full.names=TRUE)

allrasters <- rast(rastlist) 

#Changes name
names(allrasters)<-as.character(year)
```

# Disaggregate the data

To augment the resolution of the data, we will disaggregate the data to a daily level. This will allow us to better understand the trends and patterns in the data.

### Extraction the covariate

``` r
library(KrigR)
CDQ_rast <- terra::rast("Data/precipitation_raster.tif")

Covsls <- CovariateSetup(
  Training = CDQ_rast,
  Target = .009,
  Dir = Dir.Covariates,
  Keep_Global = TRUE
)
```

## Krigging

{{% youtube "jEaoYtSi4t8" %}}

``` r
QuickStart_Krig <- Kriging(
  Data = CDQ_rast, # data we want to krig as a raster object
  Covariates_training = Covs_ls[[1]], # training covariate as a raster object
  Covariates_target = Covs_ls[[2]], # target covariate as a raster object
  Equation = "GMTED2010", # the covariate(s) we want to use
  nmax = 40, # degree of localisation
  Cores = 3, # we want to krig using three cores to speed this process up
  FileName = "QuickStart_Krig", # the file name for our full kriging output
  Dir = Dir.Exports # which directory to save our final input in
)
```

``` r
precipitation_krigged <- terra::rast("Exports/QuickStart_Krig_Kriged.nc")

qc_sf <- rgeoboundaries::gb_adm2(country = "CAN") |>
  filter(shapeName %in% c("Centre-du-Québec")) |> 
  select(shapeName, geometry) 


precip_predicted <- precipitation_krigged |>
    terra::crop(
        qc_sf,
        snap = "in",
        mask = TRUE
    ) |>
    terra::project("EPSG:3978")
```

``` r
terra::plot(precip_predicted[[1]])
```

<img src="index.markdown_strict_files/figure-markdown_strict/unnamed-chunk-8-1.png" width="1260" />

# Animated Map

## Prepare the data

``` r
#Change layer name
names(precip_predicted)<-as.character(year)
 
# convert into a data frame
precip_predicted_df <- as.data.frame(
    precip_predicted,
    xy = TRUE, na.rm = TRUE
)

precip_predicted_long <- precip_predicted_df |>
    tidyr::pivot_longer(
        !c(x, y),
        names_to = "year",
        values_to = "value"
    )


# year ton integer and m to mm
precip_predicted_long<-precip_predicted_long |>
  mutate(year=as.integer(year),
         precip=value*1000) |>
  select(-value)


# create breaks

vmin <- min(precip_predicted_long$precip)
vmax <- max(precip_predicted_long$precip)

breaks <- classInt::classIntervals(
    precip_predicted_long$precip,
    n = 14,
    style = "equal"
)$brks
```

## Create the visual

``` r
#color
cols <- hcl.colors(
    n = length(breaks),
    palette = "Spectral",
    rev = TRUE
)

#theme
theme_for_the_win <- function(){
    theme_void() +
    theme(
        legend.position = "bottom",
        legend.title = element_text(
            size = 50, color = "grey10"
        ),
        legend.text = element_text(
            size = 30, color = "grey10"
        ),
        plot.title = element_text(
            size = 60, color = "grey10",
            hjust = .5, vjust = -1
        ),
        plot.subtitle = element_text(
            size = 70, color = "#c43c4e",
            hjust = .5, vjust = -1
        ), # plot.subtitle
        plot.margin = unit(
            c(
                t = 0, r = 0,
                l = 0, b = 0
            ), "lines"
        )
    )    
}

temp_map <- ggplot() +
    geom_raster(
        data = precip_predicted_long,
        aes(
            x = x, y = y,
            fill = precip
        )
    ) +
    scale_fill_gradientn(
        name = "Precipitation (mm)",
        colors = cols,
        limits = c(vmin, vmax),
        breaks = breaks,
        labels = round(
            breaks, 0
        )
    ) +
    guides(
        fill = guide_colorbar(
            direction = "horizontal",
            barheight = unit(
                1,
                units = "cm"
            ),
            barwidth = unit(
                50,
                units = "cm"
            ),
            title.position = "top",
            label.position = "bottom",
            title.hjust = .5,
            label.hjust = .5,
            nrow = 1,
            byrow = TRUE
        )
    ) +
    coord_sf(crs = "EPSG:3978") +
    labs(
        title = "Centre-du-Québec (1993-2023)",
        subtitle = "{round(as.integer(frame_time), 0)}"
    ) +
    theme_for_the_win()
```

``` r
timelapse_map <- temp_map +
    gganimate::transition_time(
        year
    ) +
    gganimate::ease_aes(
        "linear",
        interval = .1
    )
```

## Animate

``` r
animated_map <- gganimate::animate(
    timelapse_map,
    nframes = 100,
    duration = 20,
    start_pause = 3,
    end_pause = 30,
    height = 1200,
    width = 1200,
    units = "px",
    renderer = gganimate::gifski_renderer(
        loop = TRUE
    )
)

gganimate::anim_save(
    "precip CDQ animated.gif",
    animated_map
)
```

``` r
# Read and display the saved GIF animation
animation <- magick::image_read("precip CDQ animated.gif")
print(animation, info = FALSE)
```

![](index.markdown_strict_files/figure-markdown_strict/unnamed-chunk-13-1.gif)

# Conclusion

In this post, we created an animated visualization of future precipitation predictions for Centre-du-Québec. This visualization allows us to better understand the trends and anomalies in the data, which can help inform decision-making in agriculture, water resource management, and climate analysis. By disaggregating the data to a daily level and krigging the data, we were able to create a more detailed and accurate visualization of future precipitation patterns. This visualization can be used to identify areas of high and low precipitation, as well as trends over time. Overall, this animated visualization provides valuable insights into future precipitation patterns in Centre-du-Québec.

# Sign up for the newsletter

<a href = "https://dashboard.mailerlite.com/forms/1478852/152663752035010469/share"> ![](sign_up.png)
</a>

<br>

# Session Info

``` r
sessionInfo()
```

    R version 4.4.2 (2024-10-31)
    Platform: aarch64-apple-darwin20
    Running under: macOS Sequoia 15.3.2

    Matrix products: default
    BLAS:   /Library/Frameworks/R.framework/Versions/4.4-arm64/Resources/lib/libRblas.0.dylib 
    LAPACK: /Library/Frameworks/R.framework/Versions/4.4-arm64/Resources/lib/libRlapack.dylib;  LAPACK version 3.12.0

    locale:
    [1] en_US.UTF-8/en_US.UTF-8/en_US.UTF-8/C/en_US.UTF-8/en_US.UTF-8

    time zone: America/Toronto
    tzcode source: internal

    attached base packages:
    [1] stats     graphics  grDevices datasets  utils     methods   base     

    other attached packages:
     [1] jofou.lib_0.0.0.9000 reticulate_1.40.0    tidytuesdayR_1.1.2  
     [4] tictoc_1.2.1         terra_1.8-10         sf_1.0-19           
     [7] pins_1.4.0           modeltime_1.3.1      fs_1.6.5            
    [10] timetk_2.9.0         yardstick_1.3.2      workflowsets_1.1.0  
    [13] workflows_1.1.4      tune_1.2.1           rsample_1.2.1       
    [16] parsnip_1.2.1        modeldata_1.4.0      infer_1.0.7         
    [19] dials_1.3.0          scales_1.3.0         broom_1.0.7         
    [22] tidymodels_1.2.0     recipes_1.1.0        doFuture_1.0.1      
    [25] future_1.34.0        foreach_1.5.2        skimr_2.1.5         
    [28] forcats_1.0.0        stringr_1.5.1        dplyr_1.1.4         
    [31] purrr_1.0.2          readr_2.1.5          tidyr_1.3.1         
    [34] tibble_3.2.1         ggplot2_3.5.1        tidyverse_2.0.0     
    [37] lubridate_1.9.4      kableExtra_1.4.0     inspectdf_0.0.12.1  
    [40] openxlsx_4.2.7.1     knitr_1.49          

    loaded via a namespace (and not attached):
     [1] DBI_1.2.3           rlang_1.1.5         magrittr_2.0.3     
     [4] furrr_0.3.1         e1071_1.7-16        compiler_4.4.2     
     [7] png_0.1-8           systemfonts_1.2.1   vctrs_0.6.5        
    [10] lhs_1.2.0           pkgconfig_2.0.3     crayon_1.5.3       
    [13] fastmap_1.2.0       magick_2.8.5        backports_1.5.0    
    [16] rmarkdown_2.29      prodlim_2024.06.25  ggfittext_0.10.2   
    [19] tzdb_0.4.0          xfun_0.50           jsonlite_1.8.9     
    [22] progress_1.2.3      parallel_4.4.2      prettyunits_1.2.0  
    [25] R6_2.5.1            StanHeaders_2.32.10 stringi_1.8.4      
    [28] parallelly_1.41.0   rpart_4.1.24        Rcpp_1.0.14        
    [31] iterators_1.0.14    future.apply_1.11.3 zoo_1.8-12         
    [34] base64enc_0.1-3     Matrix_1.7-2        splines_4.4.2      
    [37] nnet_7.3-20         timechange_0.3.0    tidyselect_1.2.1   
    [40] rstudioapi_0.17.1   yaml_2.3.10         timeDate_4041.110  
    [43] blogdown_1.20       codetools_0.2-20    listenv_0.9.1      
    [46] lattice_0.22-6      withr_3.0.2         evaluate_1.0.3     
    [49] survival_3.8-3      units_0.8-5         proxy_0.4-27       
    [52] RcppParallel_5.1.10 zip_2.3.1           xts_0.14.1         
    [55] xml2_1.3.6          pillar_1.10.1       KernSmooth_2.23-26 
    [58] renv_1.0.7          generics_0.1.3      hms_1.1.3          
    [61] munsell_0.5.1       globals_0.16.3      class_7.3-23       
    [64] glue_1.8.0          tools_4.4.2         data.table_1.16.4  
    [67] gower_1.0.2         grid_4.4.2          ipred_0.9-15       
    [70] colorspace_2.1-1    repr_1.1.7          cli_3.6.3          
    [73] DiceDesign_1.10     rappdirs_0.3.3      viridisLite_0.4.2  
    [76] svglite_2.1.3       lava_1.8.1          gtable_0.3.6       
    [79] GPfit_1.0-8         digest_0.6.37       classInt_0.4-11    
    [82] htmltools_0.5.8.1   lifecycle_1.0.4     hardhat_1.4.0      
    [85] MASS_7.3-64        
