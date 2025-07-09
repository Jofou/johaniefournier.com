---
title: 'TyT2024W21 - VIZ:Carbon Majors Emissions Data'
author: Johanie Fournier, agr.
date: '2024-10-31'
slug: TyT2024W21_VIZ
categories:
  - rstats
  - tidymodels
  - tidytuesday
  - viz
tags:
  - rstats
  - tidymodels
  - tidytuesday
  - viz
summary: "This week we are exploring historical emissions data from Carbon Majors. They have complied a database of emissions data going back to 1854. In the first and second part I did some EDA and created a spatio-temporal machine learning model. In this part, I'm creating an animated vizualisation of the data including the prediction."
---

[![](petit.png)](https://subscribepage.io/E3ia1B)

Subscribe button

<br>

This is my latest contribution to the [`#TidyTuesday` dataset](https://github.com/rfordatascience/tidytuesday) project, featuring a recent dataset on carbon major emissions.

In the first part, I did some Exploratory Data Analysis (EDA) to look at the data set and summarize the main characteristics. In the second part, I built a spatio-temporal machine learning model to predict future emissions. In this part, I'm creating an animated map of the data including the predictions.

{{% youtube "3xoz262R-qM" %}}

## Load the Data

``` r
data_with_pred<- read_rds("data/data_with_pred.rds") |> 
  left_join(read_rds("data/geocoded_data.rds"), by=c("parent_entity")) |> 
  rename(total_emissions_MtCO2e=.value) |> 
  mutate(date=year(as.Date(.index))) |>
  select(lon, lat, total_emissions_MtCO2e, date) 
```

## Animated Visualization

``` r
library(gganimate)
library(rnaturalearth)
library(rnaturalearthdata)

#world data
world <- ne_countries(scale = "medium", returnclass = "sf")

#Create the base map
base_map <- ggplot() +
 geom_sf(data=world, fill="#F7F7F2", color="white") +
 geom_point(data=data_with_pred, 
            aes(x=lon, 
                y=lat, 
                group=date, 
                color=total_emissions_MtCO2e,
                size=total_emissions_MtCO2e,
                alpha = 50)) +
  transition_time(date) +
  ggtitle('Carbon Majors Emissions for {frame_time}') +
  shadow_mark() +
  scale_color_gradient2(low = "darkgreen", mid="gold", high = "#F51400")+
  theme(legend.position = "none", 
        axis.title.x = element_blank(),
        axis.title.y = element_blank(),
        plot.title=element_text(hjust=0.5))
num_years <- max(data_with_pred$date) - min(data_with_pred$date) + 1

# Save the animation as a GIF
anim <- animate(base_map, nframes = num_years)
anim_save("data/ggmap_animation.gif", animation = anim)
```

``` r
# Read and display the saved GIF animation
animation <- magick::image_read("data/ggmap_animation.gif")
print(animation, info = FALSE)
```

![](index.markdown_strict_files/figure-markdown_strict/unnamed-chunk-4-1.gif)

## Conclusion

This animated map shows the evolution of carbon emissions over time. The size of the points represents the amount of emissions, while the color represents the intensity of the emissions. The map shows how emissions have evolved over time and how they are distributed geographically.

# Sign up for the newsletter

[![](sign_up.png)](https://dashboard.mailerlite.com/forms/1478852/152663752035010469/share)

Sign up

## Session Info

``` r
sessionInfo()
```

    R version 4.4.2 (2024-10-31)
    Platform: aarch64-apple-darwin20
    Running under: macOS Sequoia 15.5

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
     [7] pins_1.4.0           fs_1.6.5             timetk_2.9.0        
    [10] yardstick_1.3.2      workflowsets_1.1.0   workflows_1.1.4     
    [13] tune_1.2.1           rsample_1.2.1        parsnip_1.2.1       
    [16] modeldata_1.4.0      infer_1.0.7          dials_1.3.0         
    [19] scales_1.3.0         broom_1.0.7          tidymodels_1.2.0    
    [22] recipes_1.1.0        doFuture_1.0.1       future_1.34.0       
    [25] foreach_1.5.2        skimr_2.1.5          forcats_1.0.0       
    [28] stringr_1.5.1        dplyr_1.1.4          purrr_1.0.2         
    [31] readr_2.1.5          tidyr_1.3.1          tibble_3.2.1        
    [34] ggplot2_3.5.1        tidyverse_2.0.0      lubridate_1.9.4     
    [37] kableExtra_1.4.0     inspectdf_0.0.12.1   openxlsx_4.2.7.1    
    [40] knitr_1.49          

    loaded via a namespace (and not attached):
     [1] DBI_1.2.3           rlang_1.1.5         magrittr_2.0.3     
     [4] furrr_0.3.1         e1071_1.7-16        compiler_4.4.2     
     [7] png_0.1-8           systemfonts_1.2.1   vctrs_0.6.5        
    [10] lhs_1.2.0           pkgconfig_2.0.3     crayon_1.5.3       
    [13] fastmap_1.2.0       magick_2.8.5        backports_1.5.0    
    [16] rmarkdown_2.29      prodlim_2024.06.25  ggfittext_0.10.2   
    [19] tzdb_0.4.0          xfun_0.50           jsonlite_1.8.9     
    [22] progress_1.2.3      parallel_4.4.2      prettyunits_1.2.0  
    [25] R6_2.5.1            stringi_1.8.4       parallelly_1.41.0  
    [28] rpart_4.1.24        Rcpp_1.0.14         iterators_1.0.14   
    [31] future.apply_1.11.3 zoo_1.8-12          base64enc_0.1-3    
    [34] Matrix_1.7-2        splines_4.4.2       nnet_7.3-20        
    [37] timechange_0.3.0    tidyselect_1.2.1    rstudioapi_0.17.1  
    [40] yaml_2.3.10         timeDate_4041.110   blogdown_1.20      
    [43] codetools_0.2-20    listenv_0.9.1       lattice_0.22-6     
    [46] withr_3.0.2         evaluate_1.0.3      survival_3.8-3     
    [49] units_0.8-5         proxy_0.4-27        zip_2.3.1          
    [52] xts_0.14.1          xml2_1.3.6          pillar_1.10.1      
    [55] KernSmooth_2.23-26  renv_1.0.7          generics_0.1.3     
    [58] hms_1.1.3           munsell_0.5.1       globals_0.16.3     
    [61] class_7.3-23        glue_1.8.0          tools_4.4.2        
    [64] data.table_1.16.4   gower_1.0.2         grid_4.4.2         
    [67] ipred_0.9-15        colorspace_2.1-1    repr_1.1.7         
    [70] cli_3.6.3           DiceDesign_1.10     rappdirs_0.3.3     
    [73] viridisLite_0.4.2   svglite_2.1.3       lava_1.8.1         
    [76] gtable_0.3.6        GPfit_1.0-8         digest_0.6.37      
    [79] classInt_0.4-11     htmltools_0.5.8.1   lifecycle_1.0.4    
    [82] hardhat_1.4.0       MASS_7.3-64        
