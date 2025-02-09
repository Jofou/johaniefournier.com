---
title: 'TyT2024W21 - EDA:Carbon Majors Emissions Data'
author: Johanie Fournier, agr., M.Sc.
date: "2024-10-24"
slug: TyT2024W21_EDA
categories:
  - rstats
  - tidymodels
  - tidytuesday
  - eda
tags:
  - eda
  - rstats
  - tidymodels
  - tidytuesday
summary: "This week we are exploring historical emissions data from Carbon Majors. They have complied a database of emissions data going back to 1854. In this first part, I start with some exploratory data analysis."
editor_options: 
  chunk_output_type: inline
---

<link href="index_files/libs/htmltools-fill-0.5.8.1/fill.css" rel="stylesheet" />
<script src="index_files/libs/htmlwidgets-1.6.4/htmlwidgets.js"></script>
<link href="index_files/libs/datatables-css-0.0.0/datatables-crosstalk.css" rel="stylesheet" />
<script src="index_files/libs/datatables-binding-0.33/datatables.js"></script>
<script src="index_files/libs/jquery-3.6.0/jquery-3.6.0.min.js"></script>
<link href="index_files/libs/dt-core-1.13.6/css/jquery.dataTables.min.css" rel="stylesheet" />
<link href="index_files/libs/dt-core-1.13.6/css/jquery.dataTables.extra.css" rel="stylesheet" />
<script src="index_files/libs/dt-core-1.13.6/js/jquery.dataTables.min.js"></script>
<link href="index_files/libs/crosstalk-1.2.1/css/crosstalk.min.css" rel="stylesheet" />
<script src="index_files/libs/crosstalk-1.2.1/js/crosstalk.min.js"></script>


I have worked extensively with spatial data over the past two years, so I decided to select suitable [`#TidyTuesday` dataset](https://github.com/rfordatascience/tidytuesday) and document what I have learned so far."

My latest contribution to the [`#TidyTuesday`](https://github.com/rfordatascience/tidytuesday) project featuring a recent dataset on carbon major emissions. The dataset is a compilation of emissions data from 1854 to 2019.

{{% youtube "3xoz262R-qM" %}}

## Goal

The overall goal of this blog series is to predict carbon emissions over time and space.

In this first part, the goal is to do some Exploratory Data Analysis (EDA) to look at the data set and summarize the main characteristics. To do so, I will look at the data structure, anomalies, outliers and relationships.

## Get the data

Let's start by reading in the data:

``` r
emissions <- readr::read_csv('https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2024/2024-05-21/emissions.csv')

library(skimr)
library(PerformanceAnalytics)

skim(emissions)
```

|                                                  |           |
|:-------------------------------------------------|:----------|
| Name                                             | emissions |
| Number of rows                                   | 12551     |
| Number of columns                                | 7         |
| \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_   |           |
| Column type frequency:                           |           |
| character                                        | 4         |
| numeric                                          | 3         |
| \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ |           |
| Group variables                                  | None      |

Data summary

**Variable type: character**

| skim_variable   | n_missing | complete_rate | min | max | empty | n_unique | whitespace |
|:--------------|---------:|------------:|----:|----:|------:|--------:|----------:|
| parent_entity   |         0 |             1 |   2 |  39 |     0 |      122 |          0 |
| parent_type     |         0 |             1 |  12 |  22 |     0 |        3 |          0 |
| commodity       |         0 |             1 |   6 |  19 |     0 |        9 |          0 |
| production_unit |         0 |             1 |   6 |  18 |     0 |        4 |          0 |

**Variable type: numeric**

| skim_variable          | n_missing | complete_rate |    mean |      sd |   p0 |     p25 |     p50 |     p75 |     p100 | hist  |
|:-------------|------:|--------:|-----:|-----:|---:|-----:|-----:|-----:|------:|:----|
| year                   |         0 |             1 | 1987.15 |   29.20 | 1854 | 1973.00 | 1994.00 | 2009.00 |  2022.00 | ▁▁▁▅▇ |
| production_value       |         0 |             1 |  412.71 | 1357.57 |    0 |   10.60 |   63.20 |  320.66 | 27192.00 | ▇▁▁▁▁ |
| total_emissions_MtCO2e |         0 |             1 |  113.22 |  329.81 |    0 |    8.79 |   33.06 |  102.15 |  8646.91 | ▇▁▁▁▁ |

``` r
chart.Correlation(select_if(emissions, is.numeric))
```

<img src="index.markdown_strict_files/figure-markdown_strict/unnamed-chunk-2-1.png" width="1260" />

So, we have a temporal dataset because their's a *year* column, 3 classifications columns (*parent_entity*, *parent_type*, *commodity*) and our variable of interest *total_emission_MtCO2e*.

## Trend over time

Is there a general trend over time?

``` r
sum_emissions_year<-emissions  |> 
  group_by(year) |>  
  summarise(sum=sum(total_emissions_MtCO2e)) |>  
  ungroup()

ggplot(data=sum_emissions_year, aes(x=year, y=sum))+
  geom_line()
```

<img src="index.markdown_strict_files/figure-markdown_strict/unnamed-chunk-3-1.png" width="1260" />

We can see a clear augmentation of carbon emissions over time.

The ultimate goal for this blog series will be to predict over time and space the carbon emission and visualize the result. To achieve that, we first need to understand more the relationship between *parent_entity* and *total_emission_MtCO2e*.

## Space trend

``` r
sum_emissions_entity<-emissions  |> 
  group_by(parent_entity) |>  
  summarise(sum=sum(total_emissions_MtCO2e)) |>  
  ungroup() |> 
  arrange(desc(sum))

DT::datatable(sum_emissions_entity) |> 
  DT::formatRound(columns=c("sum"), digits=0)
```

<div class="datatables html-widget html-fill-item" id="htmlwidget-b97cc6372248ab0fc41f" style="width:100%;height:auto;"></div>
<script type="application/json" data-for="htmlwidget-b97cc6372248ab0fc41f">{"x":{"filter":"none","vertical":false,"data":[["1","2","3","4","5","6","7","8","9","10","11","12","13","14","15","16","17","18","19","20","21","22","23","24","25","26","27","28","29","30","31","32","33","34","35","36","37","38","39","40","41","42","43","44","45","46","47","48","49","50","51","52","53","54","55","56","57","58","59","60","61","62","63","64","65","66","67","68","69","70","71","72","73","74","75","76","77","78","79","80","81","82","83","84","85","86","87","88","89","90","91","92","93","94","95","96","97","98","99","100","101","102","103","104","105","106","107","108","109","110","111","112","113","114","115","116","117","118","119","120","121","122"],["China (Coal)","Former Soviet Union","Saudi Aramco","Chevron","ExxonMobil","Gazprom","National Iranian Oil Co.","BP","Shell","Coal India","Poland","Pemex","Russian Federation","China (Cement)","ConocoPhillips","British Coal Corporation","CNPC","Peabody Coal Group","TotalEnergies","Abu Dhabi National Oil Company","Petroleos de Venezuela","Kuwait Petroleum Corp.","Iraq National Oil Company","Sonatrach","Rosneft","Occidental Petroleum","BHP","Petrobras","CONSOL Energy","Nigerian National Petroleum Corp.","Czechoslovakia","Petronas","Eni","QatarEnergy","Pertamina","Anglo American","Libya National Oil Corp.","Arch Resources","Lukoil","Kazakhstan","Equinor","RWE","Rio Tinto","Glencore","Alpha Metallurgical Resources","ONGC India","Sasol","Ukraine","Surgutneftegas","Repsol","Petroleum Development Oman","Sinopec","Egyptian General Petroleum","TurkmenGaz","Petoro","CNOOC","North Korea","Marathon Oil","Bumi Resources","Devon Energy","Singareni Collieries","Sonangol","Holcim Group","Novatek","Ecopetrol","Suncor Energy","Hess Corporation","Ovintiv","Czech Republic","Canadian Natural Resources","Cyprus AMAX Minerals","Westmoreland Mining","BASF","American Consolidated Natural Resources","Exxaro Resources Ltd","Bapco Energies","Adaro Energy","YPF","Cenovus Energy","APA Corporation","Banpu","PetroEcuador","EOG Resources","Alliance Resource Partners","Kiewit Mining Group","Heidelberg Materials","North American Coal","Chesapeake Energy","Syrian Petroleum","Cloud Peak","Vistra","Teck Resources","Inpex","Naftogaz","Coterra Energy","PTTEP","OMV Group","EQT Corporation","Southwestern Energy","Woodside Energy","UK Coal","Cemex","Santos","Pioneer Natural Resources","Murphy Oil","Orlen","Antero","Taiheiyo Cement","Continental Resources","Tourmaline Oil","Whitehaven Coal","Navajo Transitional Energy Company","Wolverine Fuels","Seriti Resources","Obsidian Energy","Vale","SM Energy","Adani Enterprises","CNX Resources","CRH","Tullow Oil","Slovakia"],[276458.0249208205,135112.6688134541,68831.53710380489,57897.85267803081,55105.09514381952,50686.97259605284,43111.69333717276,42530.41930217174,40674.05952866554,29391.30196291197,28749.881872077,25496.97022234476,23412.45978539556,23161.2674748,20222.25875925934,19745.3614159457,18950.90993746938,17735.42726075683,17583.55663893105,17383.17817809093,16900.99648865578,15921.81008178187,15188.33196259433,14954.69691853146,14294.59956267105,12907.16669954367,11042.11071327381,10799.30147744934,10490.13531215047,10243.43464254068,9618.470964791135,9130.304292377097,9074.625984212142,8404.877185456387,8269.728426420961,8162.762851991738,8146.450931798072,7969.293202633115,7835.304353996718,7768.913121570439,7738.844458521123,7584.75041806049,6767.000675032878,6329.254332315144,6127.223360615835,5917.363937169678,4991.747385326496,4969.162967105618,4734.523353898128,4584.290069151959,4386.740153922608,4374.180316995242,4318.011433178433,4222.7028687985,4173.518897855325,4147.451957024344,4103.678621736704,3804.403600619622,3762.012239598535,3296.652179660894,3290.960688606358,3281.231494026478,3172.614886290049,3096.416786768081,3095.756107814811,3072.275550157381,3026.038610257515,2992.801308716792,2737.350055841549,2640.050366333717,2568.883362910287,2339.073216094178,2313.434454053363,2239.722396937326,2159.901416117481,2127.181144190014,2068.445846560873,2038.976077887086,1965.298904840582,1963.916324219679,1942.734529039392,1922.402101642326,1806.078625803419,1777.070790470055,1688.6139943611,1683.69769631625,1643.728037707161,1608.841726679448,1578.270985100816,1476.157405382907,1393.537845977806,1307.518705770664,1256.043526751006,1252.490254799547,1183.725103803332,1079.567743705714,1014.161710503308,1000.977854306152,981.9748289959367,918.253426767465,881.959476578908,867.4431883275001,836.5037845974257,825.6754998658582,765.0184681492226,720.3095830088613,606.1997825621994,580.217273699,455.4242249976924,449.588109759621,428.3923757235158,389.7157568393082,384.7173502332316,361.3970365520375,355.7410347732798,317.1516463484545,316.1733421812088,316.0502462306778,227.0749033566748,216.74008502,210.9864231858376,104.499507804202]],"container":"<table class=\"display\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>parent_entity<\/th>\n      <th>sum<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"columnDefs":[{"targets":2,"render":"function(data, type, row, meta) {\n    return type !== 'display' ? data : DTWidget.formatRound(data, 0, 3, \",\", \".\", null);\n  }"},{"className":"dt-right","targets":2},{"orderable":false,"targets":0},{"name":" ","targets":0},{"name":"parent_entity","targets":1},{"name":"sum","targets":2}],"order":[],"autoWidth":false,"orderClasses":false}},"evals":["options.columnDefs.0.render"],"jsHooks":[]}</script>

We have a clear indication that country does not produce the same amount of carbon.

## Spatio-temporal trend

Can we link the spatial trend to the temporal trend? Let's find out by looking at the top 10 countries with the highest emissions.

``` r
top10_entity<-sum_emissions_entity |> 
  top_n(6, sum) |> 
  select(parent_entity)

emissions_top10<-emissions |> 
  filter(parent_entity %in% top10_entity$parent_entity) 

plot_data<-emissions_top10 |> 
  group_by(parent_entity, year) |>
  summarize(sum=sum(total_emissions_MtCO2e)) |>
  ungroup() |>
  mutate(date=as.Date(as.character(year), "%Y"),
         parent_entity_fct=as.factor(parent_entity)) |>
  select(parent_entity_fct, date, sum) |>
  pad_by_time(date, .by = "year")

plot_data |> 
    group_by(parent_entity_fct) |> 
    plot_time_series(
        .date_var    = date,
        .value       = sum,
        .interactive = FALSE,
        .facet_ncol  = 2,
        .facet_scales = "free",
    )
```

<img src="index.markdown_strict_files/figure-markdown_strict/unnamed-chunk-5-1.png" width="1260" />

Each *parent_entity* has its own trend over time.

## Anomalies and outliers

``` r
library(anomalize)

plot_data |> 
    group_by(parent_entity_fct) |> 
    time_decompose(sum) |> 
    anomalize(remainder) |>
  plot_anomalies(size_dots = 1, ncol = 2)
```

<img src="index.markdown_strict_files/figure-markdown_strict/unnamed-chunk-6-1.png" width="1260" />

``` r
plot_data |> 
    filter(parent_entity_fct=="Saudi Aramco") |> 
    time_decompose(sum) |> 
    anomalize(remainder) |>
  plot_anomaly_decomposition()
```

<img src="index.markdown_strict_files/figure-markdown_strict/unnamed-chunk-7-1.png" width="1260" />

So for simplicity, I will replace the anomalies detected by the trend for all the data. All the subsequent analysis will be done with the corrected data for the top 50 countries

``` r
top50_entity<-sum_emissions_entity |> 
  top_n(50, sum) |> 
  select(parent_entity)


final_data<-emissions |> 
  filter(parent_entity %in% top50_entity$parent_entity) |>
  group_by(parent_entity, year) |>
  summarize(sum=sum(total_emissions_MtCO2e)) |>
  ungroup() |>
  mutate(date=as.Date(as.character(year), "%Y"),
         parent_entity_fct=as.factor(parent_entity)) |>
  select(parent_entity_fct, date, sum) |>
  filter(parent_entity_fct %ni% c("Seriti Resources", "CNX Resources", "Navajo Transitional Energy Company"))|> 
  pad_by_time(date, 
              .by = "year", 
              .pad_value = NA) |> 
    group_by(parent_entity_fct) |> 
    time_decompose(sum) |> 
    anomalize(remainder) |> 
  mutate(observed=case_when(anomaly=="Yes" ~ trend,
                            TRUE ~ observed)) |> 
  select(parent_entity_fct, date, observed)
```

## Conclusion

In this first part, we have explored the dataset and identified the main characteristics. We have seen that the carbon emissions have increased over time and that the top 50 countries have different trends. We have also identified some anomalies and outliers that have been correct for the work to come in the next part.

<a href = "https://johaniefournier.aweb.page/p/4b2b1e24-af09-488d-8ff6-7b46ce61e367"> ![](petit.png)

## Session Info

``` r
sessionInfo()
```

    R version 4.4.2 (2024-10-31)
    Platform: aarch64-apple-darwin20
    Running under: macOS Sequoia 15.2

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
     [1] anomalize_0.3.0            PerformanceAnalytics_2.0.8
     [3] xts_0.14.1                 zoo_1.8-12                
     [5] jofou.lib_0.0.0.9000       reticulate_1.40.0         
     [7] tidytuesdayR_1.1.2         tictoc_1.2.1              
     [9] terra_1.8-10               sf_1.0-19                 
    [11] pins_1.4.0                 fs_1.6.5                  
    [13] timetk_2.9.0               yardstick_1.3.2           
    [15] workflowsets_1.1.0         workflows_1.1.4           
    [17] tune_1.2.1                 rsample_1.2.1             
    [19] parsnip_1.2.1              modeldata_1.4.0           
    [21] infer_1.0.7                dials_1.3.0               
    [23] scales_1.3.0               broom_1.0.7               
    [25] tidymodels_1.2.0           recipes_1.1.0             
    [27] doFuture_1.0.1             future_1.34.0             
    [29] foreach_1.5.2              skimr_2.1.5               
    [31] forcats_1.0.0              stringr_1.5.1             
    [33] dplyr_1.1.4                purrr_1.0.2               
    [35] readr_2.1.5                tidyr_1.3.1               
    [37] tibble_3.2.1               ggplot2_3.5.1             
    [39] tidyverse_2.0.0            lubridate_1.9.4           
    [41] kableExtra_1.4.0           inspectdf_0.0.12.1        
    [43] openxlsx_4.2.7.1           knitr_1.49                

    loaded via a namespace (and not attached):
      [1] rstudioapi_0.17.1   jsonlite_1.8.9      magrittr_2.0.3     
      [4] farver_2.1.2        rmarkdown_2.29      vctrs_0.6.5        
      [7] base64enc_0.1-3     blogdown_1.20       htmltools_0.5.8.1  
     [10] progress_1.2.3      curl_6.1.0          TTR_0.24.4         
     [13] sass_0.4.9          parallelly_1.41.0   bslib_0.8.0        
     [16] KernSmooth_2.23-26  htmlwidgets_1.6.4   cachem_1.1.0       
     [19] ggfittext_0.10.2    lifecycle_1.0.4     iterators_1.0.14   
     [22] pkgconfig_2.0.3     Matrix_1.7-2        R6_2.5.1           
     [25] fastmap_1.2.0       digest_0.6.37       colorspace_2.1-1   
     [28] furrr_0.3.1         crosstalk_1.2.1     labeling_0.4.3     
     [31] timechange_0.3.0    compiler_4.4.2      proxy_0.4-27       
     [34] bit64_4.6.0-1       withr_3.0.2         tseries_0.10-58    
     [37] backports_1.5.0     DBI_1.2.3           MASS_7.3-64        
     [40] lava_1.8.1          rappdirs_0.3.3      classInt_0.4-11    
     [43] tibbletime_0.1.9    tools_4.4.2         units_0.8-5        
     [46] lmtest_0.9-40       quantmod_0.4.26     zip_2.3.1          
     [49] future.apply_1.11.3 nnet_7.3-20         glue_1.8.0         
     [52] quadprog_1.5-8      nlme_3.1-166        grid_4.4.2         
     [55] generics_0.1.3      gtable_0.3.6        tzdb_0.4.0         
     [58] class_7.3-23        data.table_1.16.4   hms_1.1.3          
     [61] xml2_1.3.6          pillar_1.10.1       vroom_1.6.5        
     [64] splines_4.4.2       lhs_1.2.0           lattice_0.22-6     
     [67] padr_0.6.3          renv_1.0.7          survival_3.8-3     
     [70] bit_4.5.0.1         tidyselect_1.2.1    urca_1.3-4         
     [73] svglite_2.1.3       forecast_8.23.0     xfun_0.50          
     [76] hardhat_1.4.0       timeDate_4041.110   DT_0.33            
     [79] stringi_1.8.4       DiceDesign_1.10     yaml_2.3.10        
     [82] evaluate_1.0.3      codetools_0.2-20    cli_3.6.3          
     [85] rpart_4.1.24        systemfonts_1.2.1   jquerylib_0.1.4    
     [88] repr_1.1.7          munsell_0.5.1       Rcpp_1.0.14        
     [91] globals_0.16.3      png_0.1-8           parallel_4.4.2     
     [94] fracdiff_1.5-3      assertthat_0.2.1    gower_1.0.2        
     [97] prettyunits_1.2.0   sweep_0.2.5         GPfit_1.0-8        
    [100] listenv_0.9.1       viridisLite_0.4.2   ipred_0.9-15       
    [103] prodlim_2024.06.25  e1071_1.7-16        crayon_1.5.3       
    [106] rlang_1.1.5        
