---
title: "Predicting MO with H2O Models from IRDA data"
author: Johanie Fournier, agr. 
date: "2023-11-01"
slug: IRDA-H2O
categories:
  - rstats
  - tidyverse
tags:
  - rstats
  - tidyverse
subtitle: ''
summary: "H2O is a powerful tools to have a general idea of the performance of different models."
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: false
projects: []
---

<script src="{{< blogdown/postref >}}index_files/htmlwidgets/htmlwidgets.js"></script>
<link href="{{< blogdown/postref >}}index_files/datatables-css/datatables-crosstalk.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index_files/datatables-binding/datatables.js"></script>
<script src="{{< blogdown/postref >}}index_files/jquery/jquery-3.6.0.min.js"></script>
<link href="{{< blogdown/postref >}}index_files/dt-core/css/jquery.dataTables.min.css" rel="stylesheet" />
<link href="{{< blogdown/postref >}}index_files/dt-core/css/jquery.dataTables.extra.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index_files/dt-core/js/jquery.dataTables.min.js"></script>
<link href="{{< blogdown/postref >}}index_files/crosstalk/css/crosstalk.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index_files/crosstalk/js/crosstalk.min.js"></script>

H2O is a powerful tools to have a general idea of the performance of different models. Unfortunately this resource is constantly updating which make this structure hard to maintain in a production setting.

So I mainly use it to know which model perform best with my data. This make me gain a considerable amount of time. From there I can work on the few best models a make solid code for production purpose.

Let’s see if I can make a template of this H2O technologies that will resist the test of time (and constant update 😆)!

## Get the data

Let’s look at the data of *Chaudière-Appalaches*

``` r
board_prepared <- pins::board_folder(path_to_file, versioned = TRUE)

data <- board_prepared %>%
  pins::pin_read("TS_chaudiere_appalaches") %>%
  select(pct_mo, sable_tf, sable, argile, limon, geometry) %>%
  sf::st_as_sf(.) %>%
  sf::st_centroid() %>%
  mutate(
    longitude = sf::st_coordinates(.)[, 1],
    latitude = sf::st_coordinates(.)[, 2]
  ) %>%
  as.data.frame() %>%
  select(-geometry) %>%
  mutate_all(as.numeric) %>%
  drop_na()
```

## Explore the data

-   Organic Matter distribution

``` r
data %>%
  select(pct_mo) %>%
  my_num_dist()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-3-1.png" width="2400" />

``` r
data <- data %>%
  filter(pct_mo > 0 & pct_mo < 10)
```

``` r
data %>%
  select(pct_mo) %>%
  my_num_dist()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-5-1.png" width="2400" />

**Let’s see how everything is correlated**

``` r
data %>%
  my_corr_num_graph(data)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-6-1.png" width="2400" />

    ## NULL

## Build the model

``` r
# Parallele Processing
doFuture::registerDoFuture()
n_cores <- parallel::detectCores()
future::plan(
  strategy = future::cluster,
  workers  = parallel::makeCluster(n_cores)
)
```

**Recipe**

``` r
# Define recipe
recipe_spec <- recipe(as.formula(pct_mo ~ .), data = data) %>%
  step_normalize(all_predictors())

tbl_prep <- recipe_spec %>%
  prep() %>%
  juice()

head(tbl_prep)
```

    ## # A tibble: 6 × 7
    ##   sable_tf   sable argile  limon longitude latitude pct_mo
    ##      <dbl>   <dbl>  <dbl>  <dbl>     <dbl>    <dbl>  <dbl>
    ## 1   -1.67  -0.0817 -0.458  0.419    -1.02    -0.359   2.38
    ## 2   -1.67  -0.0817 -0.458  0.419    -1.00    -0.357   2.38
    ## 3   -1.67  -0.0817 -0.458  0.419    -0.958   -0.359   2.38
    ## 4   -1.67  -0.0817 -0.458  0.419    -0.950   -0.370   2.38
    ## 5   -0.930 -0.762   1.26   0.268    -1.23    -0.358   9.44
    ## 6   -0.172  0.677  -0.515 -0.638    -1.23    -0.356   9.11

**CorrelationFunnel**

``` r
var_select <- tbl_prep %>%
  select(pct_mo) %>%
  correlationfunnel::binarize() %>%
  select(starts_with("pct_mo") & ends_with("_Inf")) %>%
  names()

tbl_prep %>%
  correlationfunnel::binarize() %>%
  correlationfunnel::correlate(var_select) %>%
  correlationfunnel::plot_correlation_funnel() +
  labs(title = "IRDA pct_mo - Correlation Funnel")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-9-1.png" width="2400" />

**H2O Models**

``` r
library(h2o)

# START H2O CLUSTER
h2o.init()
```

    ##  Connection successful!
    ## 
    ## R is connected to the H2O cluster: 
    ##     H2O cluster uptime:         23 hours 55 minutes 
    ##     H2O cluster timezone:       America/Toronto 
    ##     H2O data parsing timezone:  UTC 
    ##     H2O cluster version:        3.34.0.3 
    ##     H2O cluster version age:    2 years and 25 days !!! 
    ##     H2O cluster name:           H2O_started_from_R_johaniefournier_xex156 
    ##     H2O cluster total nodes:    1 
    ##     H2O cluster total memory:   2.98 GB 
    ##     H2O cluster total cores:    8 
    ##     H2O cluster allowed cores:  8 
    ##     H2O cluster healthy:        TRUE 
    ##     H2O Connection ip:          localhost 
    ##     H2O Connection port:        54321 
    ##     H2O Connection proxy:       NA 
    ##     H2O Internal Security:      FALSE 
    ##     H2O API Extensions:         Amazon S3, XGBoost, Algos, AutoML, Core V3, TargetEncoder, Core V4 
    ##     R Version:                  R version 4.1.1 (2021-08-10)

``` r
# h2o.shutdown(prompt = TRUE)

# H2O DATA PREP
train <- as.h2o(tbl_prep)
```

    ## 
      |                                                                            
      |                                                                      |   0%
      |                                                                            
      |======================================================================| 100%

``` r
# Identify the response column
y <- "pct_mo"

# Identify the predictor columns (remove response and ID column)
x <- setdiff(names(train), c(y))

# H2O AutoML Training
aml <- h2o.automl(
  y = y,
  x = x,
  training_frame = train,
  project_name = paste0("H20", y),
  max_runtime_secs = 1800,
  max_models = 10,
  exclude_algos = c("DeepLearning"),
  seed = 123
)
```

    ## 
      |                                                                            
      |                                                                      |   0%
    ## 12:05:29.372: New models will be added to existing leaderboard H20pct_mo@@pct_mo (leaderboard frame=null) with already 21 models.
      |                                                                            
      |                                                                      |   1%
      |                                                                            
      |=                                                                     |   1%
      |                                                                            
      |=                                                                     |   2%
      |                                                                            
      |===                                                                   |   4%
      |                                                                            
      |====                                                                  |   5%
      |                                                                            
      |====                                                                  |   6%
    ## 12:06:06.208: StackedEnsemble_BestOfFamily_7_AutoML_6_20231101_120529 [StackedEnsemble best_of_family_1 (built with AUTO metalearner, using top model from each algorithm type)] failed: water.exceptions.H2OIllegalArgumentException: Failed to find the xval predictions frame. . .  Looks like keep_cross_validation_predictions wasn't set when building the models, or the frame was deleted.
      |                                                                            
      |=====                                                                 |   7%
      |                                                                            
      |=====                                                                 |   8%
      |                                                                            
      |======                                                                |   8%
      |                                                                            
      |======                                                                |   9%
      |                                                                            
      |=======                                                               |  10%
      |                                                                            
      |=======                                                               |  11%
      |                                                                            
      |=========                                                             |  12%
      |                                                                            
      |=========                                                             |  13%
      |                                                                            
      |==========                                                            |  14%
      |                                                                            
      |==========                                                            |  15%
      |                                                                            
      |===========                                                           |  16%
      |                                                                            
      |============                                                          |  17%
    ## 12:06:40.437: StackedEnsemble_BestOfFamily_8_AutoML_6_20231101_120529 [StackedEnsemble best_of_family_2 (built with AUTO metalearner, using top model from each algorithm type)] failed: water.exceptions.H2OIllegalArgumentException: Failed to find the xval predictions frame. . .  Looks like keep_cross_validation_predictions wasn't set when building the models, or the frame was deleted.
    ## 12:06:41.447: StackedEnsemble_AllModels_6_AutoML_6_20231101_120529 [StackedEnsemble all_2 (built with AUTO metalearner, using all AutoML models)] failed: water.exceptions.H2OIllegalArgumentException: Failed to find the xval predictions frame. . .  Looks like keep_cross_validation_predictions wasn't set when building the models, or the frame was deleted.
      |                                                                            
      |==============                                                        |  19%
      |                                                                            
      |==============                                                        |  20%
      |                                                                            
      |===============                                                       |  21%
      |                                                                            
      |================                                                      |  22%
      |                                                                            
      |================                                                      |  23%
      |                                                                            
      |=================                                                     |  24%
    ## 12:06:59.560: StackedEnsemble_BestOfFamily_9_AutoML_6_20231101_120529 [StackedEnsemble best_of_family_3 (built with AUTO metalearner, using top model from each algorithm type)] failed: water.exceptions.H2OIllegalArgumentException: Failed to find the xval predictions frame. . .  Looks like keep_cross_validation_predictions wasn't set when building the models, or the frame was deleted.
    ## 12:07:00.574: StackedEnsemble_AllModels_7_AutoML_6_20231101_120529 [StackedEnsemble all_3 (built with AUTO metalearner, using all AutoML models)] failed: water.exceptions.H2OIllegalArgumentException: Failed to find the xval predictions frame. . .  Looks like keep_cross_validation_predictions wasn't set when building the models, or the frame was deleted.
      |                                                                            
      |==================                                                    |  26%
    ## 12:07:01.586: StackedEnsemble_BestOfFamily_10_AutoML_6_20231101_120529 [StackedEnsemble best_of_family_4 (built with AUTO metalearner, using top model from each algorithm type)] failed: water.exceptions.H2OIllegalArgumentException: Failed to find the xval predictions frame. . .  Looks like keep_cross_validation_predictions wasn't set when building the models, or the frame was deleted.
      |                                                                            
      |===================                                                   |  27%
    ## 12:07:02.602: StackedEnsemble_AllModels_8_AutoML_6_20231101_120529 [StackedEnsemble all_4 (built with AUTO metalearner, using all AutoML models)] failed: water.exceptions.H2OIllegalArgumentException: Failed to find the xval predictions frame. . .  Looks like keep_cross_validation_predictions wasn't set when building the models, or the frame was deleted.
    ## 12:07:03.614: StackedEnsemble_BestOfFamily_11_AutoML_6_20231101_120529 [StackedEnsemble best_of_family_5 (built with AUTO metalearner, using top model from each algorithm type)] failed: water.exceptions.H2OIllegalArgumentException: Failed to find the xval predictions frame. . .  Looks like keep_cross_validation_predictions wasn't set when building the models, or the frame was deleted.
      |                                                                            
      |=====================                                                 |  30%
    ## 12:07:04.634: StackedEnsemble_AllModels_9_AutoML_6_20231101_120529 [StackedEnsemble all_5 (built with AUTO metalearner, using all AutoML models)] failed: water.exceptions.H2OIllegalArgumentException: Failed to find the xval predictions frame. . .  Looks like keep_cross_validation_predictions wasn't set when building the models, or the frame was deleted.
      |                                                                            
      |========================                                              |  34%
    ## 12:07:05.651: StackedEnsemble_BestOfFamily_12_AutoML_6_20231101_120529 [StackedEnsemble best_of_family_xgboost (built with xgboost metalearner, using top model from each algorithm type)] failed: water.exceptions.H2OIllegalArgumentException: Failed to find the xval predictions frame. . .  Looks like keep_cross_validation_predictions wasn't set when building the models, or the frame was deleted.
      |                                                                            
      |===========================                                           |  38%
    ## 12:07:06.671: StackedEnsemble_BestOfFamily_13_AutoML_6_20231101_120529 [StackedEnsemble best_of_family_gbm (built with gbm metalearner, using top model from each algorithm type)] failed: water.exceptions.H2OIllegalArgumentException: Failed to find the xval predictions frame. . .  Looks like keep_cross_validation_predictions wasn't set when building the models, or the frame was deleted.
    ## 12:07:07.678: StackedEnsemble_AllModels_10_AutoML_6_20231101_120529 [StackedEnsemble all_xgboost (built with xgboost metalearner, using all AutoML models)] failed: water.exceptions.H2OIllegalArgumentException: Failed to find the xval predictions frame. . .  Looks like keep_cross_validation_predictions wasn't set when building the models, or the frame was deleted.
      |                                                                            
      |============================                                          |  40%
    ## 12:07:08.688: StackedEnsemble_AllModels_11_AutoML_6_20231101_120529 [StackedEnsemble all_gbm (built with gbm metalearner, using all AutoML models)] failed: water.exceptions.H2OIllegalArgumentException: Failed to find the xval predictions frame. . .  Looks like keep_cross_validation_predictions wasn't set when building the models, or the frame was deleted.
      |                                                                            
      |=============================                                         |  42%
    ## 12:07:09.710: StackedEnsemble_BestOfFamily_14_AutoML_6_20231101_120529 [StackedEnsemble best_of_family_xglm (built with AUTO metalearner, using top model from each algorithm type)] failed: water.exceptions.H2OIllegalArgumentException: Failed to find the xval predictions frame. . .  Looks like keep_cross_validation_predictions wasn't set when building the models, or the frame was deleted.
      |                                                                            
      |================================                                      |  46%
    ## 12:07:10.721: StackedEnsemble_AllModels_12_AutoML_6_20231101_120529 [StackedEnsemble all_xglm (built with AUTO metalearner, using all AutoML models)] failed: water.exceptions.H2OIllegalArgumentException: Failed to find the xval predictions frame. . .  Looks like keep_cross_validation_predictions wasn't set when building the models, or the frame was deleted.
    ## 12:07:11.737: StackedEnsemble_BestOfFamily_15_AutoML_6_20231101_120529 [StackedEnsemble best_of_family (built with AUTO metalearner, using top model from each algorithm type)] failed: water.exceptions.H2OIllegalArgumentException: Failed to find the xval predictions frame. . .  Looks like keep_cross_validation_predictions wasn't set when building the models, or the frame was deleted.
      |                                                                            
      |=================================                                     |  48%
    ## 12:07:12.746: StackedEnsemble_Best1000_1_AutoML_6_20231101_120529 [StackedEnsemble best_N (built with AUTO metalearner, using best 1000 non-SE models)] failed: water.exceptions.H2OIllegalArgumentException: Failed to find the xval predictions frame. . .  Looks like keep_cross_validation_predictions wasn't set when building the models, or the frame was deleted.
      |                                                                            
      |======================================================================| 100%

``` r
# H2O AutoML Leaderboard
aml@leaderboard %>%
  as.data.frame() %>%
  mutate_if(is.numeric, round, digits = 3) %>%
  DT::datatable(options = list(pageLength = 16))
```

<div id="htmlwidget-1" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-1">{"x":{"filter":"none","vertical":false,"data":[["1","2","3","4","5","6","7","8","9","10","11","12","13","14","15","16","17","18","19","20","21","22","23","24","25","26","27","28","29","30","31"],["StackedEnsemble_AllModels_4_AutoML_5_20231031_160025","StackedEnsemble_BestOfFamily_5_AutoML_5_20231031_160025","StackedEnsemble_AllModels_3_AutoML_5_20231031_160025","StackedEnsemble_AllModels_5_AutoML_5_20231031_160025","StackedEnsemble_BestOfFamily_6_AutoML_5_20231031_160025","StackedEnsemble_AllModels_1_AutoML_5_20231031_160025","StackedEnsemble_AllModels_2_AutoML_5_20231031_160025","StackedEnsemble_BestOfFamily_3_AutoML_5_20231031_160025","StackedEnsemble_BestOfFamily_2_AutoML_5_20231031_160025","GBM_8_AutoML_6_20231101_120529","GBM_4_AutoML_5_20231031_160025","StackedEnsemble_BestOfFamily_4_AutoML_5_20231031_160025","StackedEnsemble_BestOfFamily_1_AutoML_5_20231031_160025","XGBoost_4_AutoML_6_20231101_120529","XGBoost_1_AutoML_5_20231031_160025","GBM_7_AutoML_6_20231101_120529","GBM_3_AutoML_5_20231031_160025","XGBoost_5_AutoML_6_20231101_120529","XGBoost_2_AutoML_5_20231031_160025","GBM_2_AutoML_5_20231031_160025","GBM_6_AutoML_6_20231101_120529","GBM_1_AutoML_5_20231031_160025","GBM_5_AutoML_6_20231101_120529","XGBoost_3_AutoML_5_20231031_160025","XGBoost_6_AutoML_6_20231101_120529","XRT_2_AutoML_6_20231101_120529","XRT_1_AutoML_5_20231031_160025","DRF_1_AutoML_5_20231031_160025","DRF_2_AutoML_6_20231101_120529","GLM_2_AutoML_6_20231101_120529","GLM_1_AutoML_5_20231031_160025"],[0.053,0.053,0.056,0.056,0.056,0.056,0.056,0.056,0.056,0.056,0.056,0.059,0.067,0.069,0.069,0.07,0.07,0.071,0.071,0.079,0.079,0.095,0.095,0.112,0.112,0.119,0.119,0.128,0.128,3.826,3.826],[0.229,0.231,0.236,0.236,0.236,0.236,0.236,0.236,0.236,0.237,0.237,0.243,0.26,0.263,0.263,0.264,0.264,0.267,0.267,0.281,0.281,0.308,0.308,0.334,0.334,0.344,0.344,0.357,0.357,1.956,1.956],[0.053,0.053,0.056,0.056,0.056,0.056,0.056,0.056,0.056,0.056,0.056,0.059,0.067,0.069,0.069,0.07,0.07,0.071,0.071,0.079,0.079,0.095,0.095,0.112,0.112,0.119,0.119,0.128,0.128,3.826,3.826],[0.073,0.074,0.068,0.087,0.088,0.087,0.087,0.087,0.087,0.087,0.087,0.068,0.11,0.113,0.113,0.105,0.105,0.113,0.113,0.119,0.119,0.144,0.144,0.171,0.171,0.179,0.179,0.182,0.182,1.622,1.622],[0.028,0.029,0.029,0.03,0.03,0.03,0.03,0.03,0.03,0.03,0.03,0.03,0.034,0.035,0.035,0.034,0.034,0.035,0.035,0.036,0.036,0.042,0.042,0.043,0.043,0.042,0.042,0.043,0.043,0.303,0.303]],"container":"<table class=\"display\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>model_id<\/th>\n      <th>mean_residual_deviance<\/th>\n      <th>rmse<\/th>\n      <th>mse<\/th>\n      <th>mae<\/th>\n      <th>rmsle<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"pageLength":16,"columnDefs":[{"className":"dt-right","targets":[2,3,4,5,6]},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false,"lengthMenu":[10,16,25,50,100]}},"evals":[],"jsHooks":[]}</script>

## Look at the results

**All 10 models**

``` r
model_ids <- as_tibble(aml@leaderboard$model_id)[, 1] %>%
  mutate(n = row_number())

model_ident <- model_ids %>%
  filter(model_id %in% "StackedEnsemble_AllModels_4_AutoML_5_20231031_160025")

model_all <- h2o.getModel(model_ident$model_id)
model_all
```

    ## Model Details:
    ## ==============
    ## 
    ## H2ORegressionModel: stackedensemble
    ## Model ID:  StackedEnsemble_AllModels_4_AutoML_5_20231031_160025 
    ## Number of Base Models: 10
    ## 
    ## Base Models (count by algorithm type):
    ## 
    ##     drf     gbm     glm xgboost 
    ##       2       4       1       3 
    ## 
    ## Metalearner:
    ## 
    ## Metalearner algorithm: gbm
    ## Metalearner cross-validation fold assignment:
    ##   Fold assignment scheme: AUTO
    ##   Number of folds: 5
    ##   Fold column: NULL
    ## Metalearner hyperparameters: 
    ## 
    ## 
    ## H2ORegressionMetrics: stackedensemble
    ## ** Reported on training data. **
    ## 
    ## MSE:  0.01119969
    ## RMSE:  0.1058286
    ## MAE:  0.04247019
    ## RMSLE:  0.01331242
    ## Mean Residual Deviance :  0.01119969
    ## 
    ## 
    ## 
    ## H2ORegressionMetrics: stackedensemble
    ## ** Reported on cross-validation data. **
    ## ** 5-fold cross-validation on training data (Metrics computed for combined holdout predictions) **
    ## 
    ## MSE:  0.0525015
    ## RMSE:  0.229132
    ## MAE:  0.07341821
    ## RMSLE:  0.02842482
    ## Mean Residual Deviance :  0.0525015

``` r
model_ident <- model_ids %>%
  filter(model_id %in% "StackedEnsemble_BestOfFamily_5_AutoML_5_20231031_160025")
model_best_of <- h2o.getModel(model_ident$model_id)
model_best_of
```

    ## Model Details:
    ## ==============
    ## 
    ## H2ORegressionModel: stackedensemble
    ## Model ID:  StackedEnsemble_BestOfFamily_5_AutoML_5_20231031_160025 
    ## Number of Base Models: 5
    ## 
    ## Base Models (count by algorithm type):
    ## 
    ##     drf     gbm     glm xgboost 
    ##       2       1       1       1 
    ## 
    ## Metalearner:
    ## 
    ## Metalearner algorithm: gbm
    ## Metalearner cross-validation fold assignment:
    ##   Fold assignment scheme: AUTO
    ##   Number of folds: 5
    ##   Fold column: NULL
    ## Metalearner hyperparameters: 
    ## 
    ## 
    ## H2ORegressionMetrics: stackedensemble
    ## ** Reported on training data. **
    ## 
    ## MSE:  0.01249409
    ## RMSE:  0.1117769
    ## MAE:  0.04235414
    ## RMSLE:  0.01408869
    ## Mean Residual Deviance :  0.01249409
    ## 
    ## 
    ## 
    ## H2ORegressionMetrics: stackedensemble
    ## ** Reported on cross-validation data. **
    ## ** 5-fold cross-validation on training data (Metrics computed for combined holdout predictions) **
    ## 
    ## MSE:  0.05346869
    ## RMSE:  0.231233
    ## MAE:  0.0735333
    ## RMSLE:  0.02866697
    ## Mean Residual Deviance :  0.05346869

``` r
model_ident <- model_ids %>%
  filter(model_id %in% "GBM_4_AutoML_5_20231031_160025")
model_gbm <- h2o.getModel(model_ident$model_id)
model_gbm
```

    ## Model Details:
    ## ==============
    ## 
    ## H2ORegressionModel: gbm
    ## Model ID:  GBM_4_AutoML_5_20231031_160025 
    ## Model Summary: 
    ##   number_of_trees number_of_internal_trees model_size_in_bytes min_depth
    ## 1             296                      296              550589        10
    ##   max_depth mean_depth min_leaves max_leaves mean_leaves
    ## 1        10   10.00000         17        392   143.48311
    ## 
    ## 
    ## H2ORegressionMetrics: gbm
    ## ** Reported on training data. **
    ## 
    ## MSE:  0.02034233
    ## RMSE:  0.1426266
    ## MAE:  0.05560683
    ## RMSLE:  0.01756616
    ## Mean Residual Deviance :  0.02034233
    ## 
    ## 
    ## 
    ## H2ORegressionMetrics: gbm
    ## ** Reported on cross-validation data. **
    ## ** 5-fold cross-validation on training data (Metrics computed for combined holdout predictions) **
    ## 
    ## MSE:  0.0562326
    ## RMSE:  0.2371341
    ## MAE:  0.08748792
    ## RMSLE:  0.03014461
    ## Mean Residual Deviance :  0.0562326
    ## 
    ## 
    ## Cross-Validation Metrics Summary: 
    ##                            mean       sd cv_1_valid cv_2_valid cv_3_valid
    ## mae                    0.087488 0.002646   0.087158   0.089488   0.083020
    ## mean_residual_deviance 0.056232 0.007035   0.052462   0.063129   0.045919
    ## mse                    0.056232 0.007035   0.052462   0.063129   0.045919
    ## r2                     0.987552 0.001681   0.988628   0.985815   0.989767
    ## residual_deviance      0.056232 0.007035   0.052462   0.063129   0.045919
    ## rmse                   0.236748 0.015113   0.229045   0.251256   0.214288
    ## rmsle                  0.030106 0.001708   0.029139   0.031774   0.027593
    ##                        cv_4_valid cv_5_valid
    ## mae                      0.088955   0.088818
    ## mean_residual_deviance   0.058409   0.061242
    ## mse                      0.058409   0.061242
    ## r2                       0.987488   0.986064
    ## residual_deviance        0.058409   0.061242
    ## rmse                     0.241681   0.247472
    ## rmsle                    0.031009   0.031014

``` r
h2o::h2o.varimp_plot(model_gbm)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-13-1.png" width="2400" />

``` r
# End Parallele Processing
future::plan(future::sequential)
```
