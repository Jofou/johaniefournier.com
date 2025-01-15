---
title: 'TyT2024W21 - ML:Carbon Majors Emissions Data'
author: Johanie Fournier, agr., M.Sc.
date: "2024-10-25"
slug: TyT2024W21_ML
categories:
  - rstats
  - tidymodels
  - tidytuesday
  - models
tags:
  - rstats
  - tidymodels
  - tidytuesday
  - models
summary: "This week we're exploring historical emissions data from Carbon Majors. They have complied a database of emissions data going back to 1854. In this second part, I'm predicting carbon emission over space and time."
editor_options: 
  chunk_output_type: inline
---

<script src="index_files/libs/htmlwidgets-1.5.4/htmlwidgets.js"></script>
<link href="index_files/libs/datatables-css-0.0.0/datatables-crosstalk.css" rel="stylesheet" />
<script src="index_files/libs/datatables-binding-0.19/datatables.js"></script>
<script src="index_files/libs/jquery-3.6.0/jquery-3.6.0.min.js"></script>
<link href="index_files/libs/dt-core-1.10.20/css/jquery.dataTables.min.css" rel="stylesheet" />
<link href="index_files/libs/dt-core-1.10.20/css/jquery.dataTables.extra.css" rel="stylesheet" />
<script src="index_files/libs/dt-core-1.10.20/js/jquery.dataTables.min.js"></script>
<link href="index_files/libs/crosstalk-1.1.1/css/crosstalk.css" rel="stylesheet" />
<script src="index_files/libs/crosstalk-1.1.1/js/crosstalk.min.js"></script>


This is my latest contribution to the [`#TidyTuesday` dataset](https://github.com/rfordatascience/tidytuesday) project, featuring a recent dataset on carbon major emissions.

In the [firs part](https://www.johaniefournier.com/blog/tyt2024w21_eda/), the goal was to do some Exploratory Data Analysis (EDA). To do so, I looked at the data structure, anomalies, outliers and relationships to ensure I have everithing I need for this second part.

In this second part, I will be predicting carbon emissions over space and time. I will be using machine learning techniques to predict future emissions based on historical data. I will be using the `spdep` package to create a spatial weights matrix, and the `modeltime` package to train machine learning models and make predictions for future emissions. Let's get started!

{{% youtube "3xoz262R-qM" %}}

## Data Preprocessing

The first step here is to add the GPS coordinate for all th 50 *parent_entity* in the dataset. To do so, we first need to find the headquarters location for each company.

### Get Headquarters Location

``` r
# Load the data
data<-read_rds("data/final_data.rds")
head(data)
```

    # A tibble: 6 × 3
      parent_entity_fct              date       observed
      <fct>                          <date>        <dbl>
    1 Abu Dhabi National Oil Company 1962-10-24    0.498
    2 Abu Dhabi National Oil Company 1963-10-24    1.05 
    3 Abu Dhabi National Oil Company 1964-10-24    4.17 
    4 Abu Dhabi National Oil Company 1965-10-24    6.19 
    5 Abu Dhabi National Oil Company 1966-10-24    7.56 
    6 Abu Dhabi National Oil Company 1967-10-24    8.00 

``` r
parent_entity<-data|>  
  mutate(parent_entity = as.character(parent_entity_fct))|> 
  select(parent_entity)|>  
  unique()

DT::datatable(parent_entity)
```

<div id="htmlwidget-01d78cbf3971deab4dad" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-01d78cbf3971deab4dad">{"x":{"filter":"none","vertical":false,"data":[["1","2","3","4","5","6","7","8","9","10","11","12","13","14","15","16","17","18","19","20","21","22","23","24","25","26","27","28","29","30","31","32","33","34","35","36","37","38","39","40","41","42","43","44","45","46","47","48","49","50"],["Abu Dhabi National Oil Company","Alpha Metallurgical Resources","Anglo American","Arch Resources","BHP","BP","British Coal Corporation","Chevron","China (Cement)","China (Coal)","CNPC","Coal India","ConocoPhillips","CONSOL Energy","Czechoslovakia","Eni","Equinor","ExxonMobil","Former Soviet Union","Gazprom","Glencore","Iraq National Oil Company","Kazakhstan","Kuwait Petroleum Corp.","Libya National Oil Corp.","Lukoil","National Iranian Oil Co.","Nigerian National Petroleum Corp.","Occidental Petroleum","ONGC India","Peabody Coal Group","Pemex","Pertamina","Petrobras","Petroleos de Venezuela","Petronas","Poland","QatarEnergy","Repsol","Rio Tinto","Rosneft","Russian Federation","RWE","Sasol","Saudi Aramco","Shell","Sonatrach","Surgutneftegas","TotalEnergies","Ukraine"]],"container":"<table class=\"display\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>parent_entity<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"order":[],"autoWidth":false,"orderClasses":false,"columnDefs":[{"orderable":false,"targets":0}]}},"evals":[],"jsHooks":[]}</script>

``` python
import pandas as pd
import requests

# Load data
data = r.parent_entity

# Function to get headquarters from Wikidata
def get_wikidata_headquarters(company):
    try:
        # Search for the company in Wikidata
        search_url = f"https://www.wikidata.org/w/api.php?action=wbsearchentities&search={company}&language=en&format=json"
        search_response = requests.get(search_url).json()
        
        if search_response['search']:
            # Get the Wikidata ID of the first search result
            wikidata_id = search_response['search'][0]['id']

            # Get detailed information from Wikidata using the ID
            entity_url = f"https://www.wikidata.org/wiki/Special:EntityData/{wikidata_id}.json"
            entity_response = requests.get(entity_url).json()

            # Extract the headquarters location
            entity_data = entity_response['entities'].get(wikidata_id, {})
            claims = entity_data.get('claims', {})
            headquarters_claim = claims.get('P159', [])
            
            if headquarters_claim:
                # Get the headquarters location label
                location_id = headquarters_claim[0]['mainsnak']['datavalue']['value']['id']
                location_url = f"https://www.wikidata.org/wiki/Special:EntityData/{location_id}.json"
                location_response = requests.get(location_url).json()

                location_data = location_response['entities'].get(location_id, {})
                location_label = location_data['labels'].get('en', {}).get('value', 'Unknown')
                
                return location_label
        return 'Headquarters not found'
    except Exception as e:
        return f'Error: {str(e)}'

# Assuming 'data' is your dataframe and 'parent_entity' is the column containing company names
data['headquarters'] = data['parent_entity'].apply(get_wikidata_headquarters)

# Display the updated dataframe
data.head()
```

                        parent_entity            headquarters
    0  Abu Dhabi National Oil Company               Abu Dhabi
    1   Alpha Metallurgical Resources  Headquarters not found
    2                  Anglo American                  London
    3                  Arch Resources               St. Louis
    4                             BHP  Headquarters not found

``` r
# Complete missing headquarters
headquarters<-py$data |> 
  mutate(headquarters=case_when(
    parent_entity=="Alpha Metallurgical Resources" ~ "Bristol, Tennessee, États-Unis",
    parent_entity=="BHP" ~ "Melbourne, Ausralie",
    parent_entity=="BP" ~ "Londres, Royaume-Uni",
    parent_entity=="British Coal Corporation" ~ "Hobart House, Grosvenor Place, London",
    parent_entity=="Chevron" ~ "San Ramon, Californie, États-Unis",
    parent_entity=="China (Cement)" ~ "China",
    parent_entity=="China (Coal)" ~ "Beijing, Chine",
    parent_entity=="Czechoslovakia" ~ "Czech Republic",
    parent_entity=="Eni" ~ "Rpme, Italie",
    parent_entity=="Former Soviet Union" ~ "Moscou, Russie",
    parent_entity=="Kazakhstan" ~ "Astana, Kazakhstan",
    parent_entity=="Kuwait Petroleum Corp." ~ "Kuwait City, Koweït",
    parent_entity=="Libya National Oil Corp." ~ "Tripoli, Libye",
    parent_entity=="National Iranian Oil Co." ~ "Tehran, Iran",
    parent_entity=="Nigerian National Petroleum Corp." ~ "Abuja, Nigeria",
    parent_entity=="ONGC India" ~ "India",
    parent_entity=="Peabody Coal Group" ~ "Saint-Louis, Missouri, États-Unis",
    parent_entity=="Poland" ~ "Poland",
    parent_entity=="Russian Federation" ~ "Russie",
    parent_entity=="Shell" ~ "Londres, Royaume-Uni",
    parent_entity=="Ukraine" ~ "Ukraine",
    TRUE ~ headquarters
  ))
```

### Get GPS Coordinates

Now that we have the headquarters locations, we can extract the GPS coordinates for each location using the `ggmap` package.

``` r
#Get GPS coordinate with ggmap
ggmap::register_google("Insert_your_key_here")
geocoded_data<-ggmap::geocode(headquarters$headquarters) |> 
  bind_cols(headquarters) |> 
    drop_na()

write_rds(geocoded_data, "data/geocoded_data.rds")
```

``` r
#Load my data
geocoded_data<-read_rds("data/geocoded_data.rds")
head(geocoded_data)
```

    # A tibble: 6 × 4
          lon   lat parent_entity                  headquarters                  
        <dbl> <dbl> <chr>                          <chr>                         
    1  54.4    24.5 Abu Dhabi National Oil Company Abu Dhabi                     
    2 -82.2    36.6 Alpha Metallurgical Resources  Bristol, Tennessee, États-Unis
    3  -0.128  51.5 Anglo American                 London                        
    4 -90.2    38.6 Arch Resources                 St. Louis                     
    5 145.    -37.8 BHP                            Melbourne, Ausralie           
    6  -0.128  51.5 BP                             Londres, Royaume-Uni          

Let's put it all together!

``` r
data_final<-data |> 
  mutate(parent_entity = as.character(parent_entity_fct)) |>
  right_join(geocoded_data, by=c("parent_entity")) |> 
  select(-parent_entity_fct, -headquarters) 
```

### Create a Spatial Weights Matrix

We need some spatial information extracted from the coordinate to put into the temporal model. We will use the GPS coordinates to create a k-nearest neighbors spatial weights matrix. This matrix will be used as spatial information of the observed emissions, which will be used as a predictor in the machine learning models.

``` r
# Extract coordinates
coords <- data_final |> 
  select(lon, lat)

# Create k-nearest neighbors spatial weights matrix (example with k = 4)
knn <- spdep::knearneigh(coords, k = 4)
nb <- spdep::knn2nb(knn)
listw_knn <- spdep::nb2listw(nb, style = "W")

# Compute spatial lag using the weights matrix
spatial_lag <- spdep::lag.listw(listw_knn, data_final$observed)

# Add the spatial lag
data_spatial <- data_final |> 
  mutate(spatial_lag = spatial_lag) |> 
  select(-lon, -lat)
```

``` r
top6_entity<-data_spatial |>
  select(parent_entity) |>
  unique() |> 
  top_n(6) 

data_spatial |> 
    group_by(parent_entity) |> 
    filter(parent_entity %in% top6_entity$parent_entity) |> 
    plot_time_series(date, observed,
                     .facet_ncol = 2,
                     .smooth = FALSE, 
                     .interactive = FALSE)
```

<img src="index.markdown_strict_files/figure-markdown_strict/unnamed-chunk-20-1.png" width="1260" />

## Modeling

Now that we have the spatial information, we can start training machine learning models to predict future emissions. We will use the `modeltime` package to train and evaluate the models.

### Extend, Nesting and Splitting Data

First, we need to extend the time series data to include future observations for the next 20 years, then nest the data by the parent entity, and finally split the data into training and testing sets.

``` r
nested_data_tbl <- data_spatial|> 
    group_by(parent_entity)|> 
  #Extend
    extend_timeseries(
        .id_var = parent_entity,
        .date_var = date,
        .length_future = 20
    )|> 
  #Nest
    nest_timeseries(
        .id_var = parent_entity,
        .length_future = 20
    )|> 
  #Split
    split_nested_timeseries(
        .length_test = 20
    )
```

### Model training

Next, we will train machine learning models to predict future emissions. We will train two XGBoost models with different learning rates and a Temporal Hierarchical Forecasting (THief) model.

``` r
# Recipe
rec_xgb <- recipe(observed ~ ., extract_nested_train_split(nested_data_tbl)) |> 
    step_timeseries_signature(date) |> 
    step_rm(date) |> 
    step_zv(all_predictors()) |> 
    step_dummy(all_nominal_predictors(), one_hot = TRUE)

# Models 
wflw_xgb_1 <- workflow()|> 
    add_model(boost_tree("regression", learn_rate = 0.35)|>  set_engine("xgboost"))|> 
    add_recipe(rec_xgb)

wflw_xgb_2 <- workflow()|> 
    add_model(boost_tree("regression", learn_rate = 0.50)|>  set_engine("xgboost"))|> 
    add_recipe(rec_xgb)

wflw_thief <- workflow()|> 
    add_model(temporal_hierarchy()|>  set_engine("thief"))|> 
    add_recipe(recipe(observed ~ ., extract_nested_train_split(nested_data_tbl)))


# Nested Modeling
parallel_start(6)
nested_modeltime_tbl <- nested_data_tbl|> 
    modeltime_nested_fit(
        model_list = list(
            wflw_xgb_1,
            wflw_xgb_2,
            wflw_thief
        ),
        control = control_nested_fit(
            verbose   = TRUE,
            allow_par = TRUE
        )
    )
```

### Review Errors

Before selecting the best model, we need to review any errors that occurred during the training process. We will also check the accuracy of the models and investigate any anomalies.

``` r
# #1) list errors
# errors<-nested_modeltime_tbl|> 
#   extract_nested_error_report()
# 
# #2) Spot too high accuracy
# nested_modeltime_tbl |> 
#     extract_nested_test_accuracy() |> 
#     table_modeltime_accuracy()
# 
# #3) Investigate
# nested_modeltime_tbl|> 
#     filter(parent_entity == "Lukoil") |> 
#     extract_nested_train_split()
# 
# nested_modeltime_tbl|> 
#     extract_nested_test_forecast() |> 
#     filter(parent_entity == "Former Soviet Union") |> 
#     group_by(parent_entity) |> 
#     plot_modeltime_forecast(.facet_ncol = 3,
#                             .interactive = FALSE)

#4) remove small time series
ids_small_timeseries <- c("Lukoil", "Rosneft", "Glencore","Surgutneftegas", "Ukraine")

nested_modeltime_subset_tbl <- nested_modeltime_tbl|> 
    filter(!parent_entity %in% ids_small_timeseries)
```

### Select Best Models

Next, we will select the best models based on the root mean square error (RMSE) metric. We will visualize the best models to compare their performance and identify any anomalies.

``` r
nested_best_tbl <- nested_modeltime_subset_tbl|> 
    modeltime_nested_select_best(metric = "rmse")

nested_best_tbl |> 
    extract_nested_test_forecast() |> 
    filter(parent_entity %in% c("CONSOL Energy", "Shell", "Former Soviet Union", "Anglo American","Chevron","BP")) |> 
    group_by(parent_entity) |> 
    plot_modeltime_forecast(.facet_ncol = 2, 
                            .legend_show = FALSE,
                            .interactive = FALSE)
```

<img src="index.markdown_strict_files/figure-markdown_strict/unnamed-chunk-28-1.png" width="1260" />

### Refit Models

Finally, we will refit the best models on the entire dataset to make predictions for future emissions. We will also review any errors that occurred during the refitting process and visualize the future forecasts for the selected companies.

``` r
# Refit
nested_best_refit_tbl <- nested_best_tbl|> 
    modeltime_nested_refit(
        control = control_refit(
            verbose   = TRUE,
            allow_par = TRUE
        )
    )

# Error last check
nested_best_refit_tbl|>  extract_nested_error_report()
```

    # A tibble: 0 × 4
    # ℹ 4 variables: parent_entity <chr>, .model_id <int>, .model_desc <chr>,
    #   .error_desc <chr>

``` r
# Visualize Future Forecast 
nested_best_refit_tbl|> 
    extract_nested_future_forecast()|> 
    filter(parent_entity %in% c("CONSOL Energy", "Shell", "Former Soviet Union", "Anglo American","Chevron","BP"))%>%
    group_by(parent_entity)|> 
    plot_modeltime_forecast(.facet_ncol = 2, 
                            .legend_show = FALSE,
                            .interactive = FALSE)
```

<img src="index.markdown_strict_files/figure-markdown_strict/unnamed-chunk-30-1.png" width="1260" />

## Save preds

Finally, we will save the predictions for future emissions to a file for further analysis and visualization.

``` r
nested_best_refit_tbl |> 
    extract_nested_future_forecast() |> 
    write_rds("data/data_with_pred.rds")
```

## Conclusion

In this post, I have shown how to predict carbon emissions over space and time using the Carbon Majors dataset. I have demonstrated how to preprocess the data, create a spatial weights matrix, and train machine learning models to predict future emissions. I have also shown how to evaluate the model's performance and make predictions for future emissions. This is just one example of how machine learning can be used to analyze environmental data and make predictions about future trends. I hope this post has been helpful and informative, and that it has inspired you to explore the Carbon Majors dataset further and apply machine learning techniques to other environmental datasets.

<a href = "https://johaniefournier.aweb.page/p/4b2b1e24-af09-488d-8ff6-7b46ce61e367"> ![](petit.png)

## Session Info

``` r
sessionInfo()
```

    R version 4.1.1 (2021-08-10)
    Platform: x86_64-apple-darwin17.0 (64-bit)
    Running under: macOS Big Sur 10.16

    Matrix products: default
    BLAS:   /Library/Frameworks/R.framework/Versions/4.1/Resources/lib/libRblas.0.dylib
    LAPACK: /Library/Frameworks/R.framework/Versions/4.1/Resources/lib/libRlapack.dylib

    locale:
    [1] en_US.UTF-8/en_US.UTF-8/en_US.UTF-8/C/en_US.UTF-8/en_US.UTF-8

    attached base packages:
    [1] stats     graphics  grDevices datasets  utils     methods   base     

    other attached packages:
     [1] jofou.lib_0.0.0.9000  reticulate_1.37.0     tidytuesdayR_1.0.2   
     [4] tictoc_1.2.1          terra_1.6-17          sf_1.0-5             
     [7] pins_1.0.1.9000       modeltime_1.0.0       fs_1.5.2             
    [10] timetk_2.6.1          yardstick_1.2.0       workflowsets_0.1.0   
    [13] workflows_0.2.4       tune_0.1.6            rsample_0.1.0        
    [16] parsnip_1.1.1         modeldata_0.1.1       infer_1.0.0          
    [19] dials_0.0.10          scales_1.3.0          broom_1.0.4          
    [22] tidymodels_0.1.4      recipes_0.1.17        doFuture_0.12.0      
    [25] future_1.22.1         foreach_1.5.1         skimr_2.1.5          
    [28] forcats_1.0.0         stringr_1.5.0         dplyr_1.1.2          
    [31] purrr_1.0.1           readr_2.1.4           tidyr_1.3.0          
    [34] tibble_3.2.1          ggplot2_3.5.1         tidyverse_2.0.0      
    [37] lubridate_1.9.2       kableExtra_1.3.4.9000 inspectdf_0.0.11     
    [40] openxlsx_4.2.4        knitr_1.36           

    loaded via a namespace (and not attached):
      [1] readxl_1.4.2         backports_1.4.1      systemfonts_1.0.3   
      [4] repr_1.1.7           sp_1.4-5             splines_4.1.1       
      [7] crosstalk_1.1.1      listenv_0.8.0        usethis_2.0.1       
     [10] digest_0.6.29        htmltools_0.5.8.1    fansi_0.5.0         
     [13] magrittr_2.0.3       doParallel_1.0.16    tzdb_0.1.2          
     [16] globals_0.14.0       ggfittext_0.9.1      gower_0.2.2         
     [19] RcppParallel_5.1.4   xts_0.12.1           svglite_2.0.0       
     [22] hardhat_1.3.0        timechange_0.1.1     prettyunits_1.1.1   
     [25] colorspace_2.0-2     rvest_1.0.3          rappdirs_0.3.3      
     [28] xfun_0.39            crayon_1.4.2         jsonlite_1.8.4      
     [31] survival_3.2-11      zoo_1.8-12           iterators_1.0.13    
     [34] glue_1.6.2           gtable_0.3.0         ipred_0.9-12        
     [37] webshot_0.5.2        future.apply_1.8.1   DBI_1.1.1           
     [40] Rcpp_1.0.13          spData_2.3.1         viridisLite_0.4.0   
     [43] progress_1.2.2       units_0.7-2          spdep_1.3-5         
     [46] GPfit_1.0-8          proxy_0.4-26         DT_0.19             
     [49] lava_1.6.10          StanHeaders_2.21.0-7 prodlim_2019.11.13  
     [52] htmlwidgets_1.5.4    httr_1.4.6           ellipsis_0.3.2      
     [55] wk_0.6.0             farver_2.1.0         pkgconfig_2.0.3     
     [58] deldir_2.0-4         sass_0.4.0           nnet_7.3-16         
     [61] utf8_1.2.2           labeling_0.4.2       tidyselect_1.2.0    
     [64] rlang_1.1.1          DiceDesign_1.9       munsell_0.5.0       
     [67] cellranger_1.1.0     tools_4.1.1          xgboost_1.4.1.1     
     [70] cli_3.6.1            generics_0.1.3       evaluate_0.14       
     [73] fastmap_1.2.0        yaml_2.2.1           zip_2.2.0           
     [76] s2_1.0.7             xml2_1.3.4           compiler_4.1.1      
     [79] rstudioapi_0.14      curl_5.2.3           png_0.1-7           
     [82] e1071_1.7-9          lhs_1.1.3            bslib_0.3.1         
     [85] stringi_1.7.5        blogdown_1.9.4       lattice_0.20-44     
     [88] Matrix_1.3-4         classInt_0.4-3       vctrs_0.6.5         
     [91] pillar_1.9.0         lifecycle_1.0.3      furrr_0.2.3         
     [94] jquerylib_0.1.4      data.table_1.14.2    R6_2.5.1            
     [97] renv_1.0.7           KernSmooth_2.23-20   parallelly_1.28.1   
    [100] codetools_0.2-18     boot_1.3-28          MASS_7.3-54         
    [103] anomalize_0.3.0      withr_2.5.0          parallel_4.1.1      
    [106] hms_1.1.3            grid_4.1.1           rpart_4.1-15        
    [109] timeDate_3043.102    class_7.3-19         rmarkdown_2.25      
    [112] base64enc_0.1-3     
