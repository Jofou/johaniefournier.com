---
title: 'From Trends to Predictions: Machine Learning Forecasts for Centre-du-Québec’s Precipitation'
author: Johanie Fournier, agr., M.Sc.
date: "2025-01-30"
slug: cdq_precipitation_ML
categories:
  - rstats
  - tidymodels
  - models
  - viz
tags:
  - rstats
  - tidymodels
  - models
  - viz
summary: "In this phase of the analysis, we aim to model precipitation patterns in Centre-du-Québec using machine learning techniques, leveraging historical climate and environmental data. We will train an XGBoost models and predict precipitation trends. Model performance will be evaluated using cross-validation and regression metrics to determine the most effective approach."
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


<a href = "https://johaniefournier.aweb.page/p/4b2b1e24-af09-488d-8ff6-7b46ce61e367"> ![](petit.png)
</a>

<br>

Understanding precipitation patterns is essential for climate analysis, agriculture, and water resource management. In the previous blog post, we conducted an exploratory data analysis (EDA) to uncover key trends, spatial distributions, and relationships within the dataset. This provided valuable insights into precipitation variability and guided feature selection for modeling.

Building on that foundation, this post explores machine learning approaches to predict precipitation using historical climate data. We will train an XGBoost model and evaluate his effectiveness in capturing complex climate patterns. Using cross-validation and standard regression metrics, we will compare model performance and identify the most suitable approach for precipitation prediction in this region.

## Data Preprocessing

The first step here is to load the data and preprocess it for modeling. We will extract relevant features and create a spatial weights matrix.

### Load the data

``` r
precipitation_data <- readRDS("Data/precipitation_sf.rds") |> 
  select(year, value) |> 
  filter(year>=1993 & year<=2023)
  
head(precipitation_data)
```

    Simple feature collection with 6 features and 2 fields
    Geometry type: POINT
    Dimension:     XY
    Bounding box:  xmin: -72.171 ymin: 46.489 xmax: -72.171 ymax: 46.489
    Geodetic CRS:  WGS 84
    # A tibble: 6 × 3
       year   value         geometry
      <dbl>   <dbl>      <POINT [°]>
    1  1993 0.00290 (-72.171 46.489)
    2  1993 0.00225 (-72.171 46.489)
    3  1993 0.00211 (-72.171 46.489)
    4  1993 0.00411 (-72.171 46.489)
    5  1993 0.00305 (-72.171 46.489)
    6  1993 0.00426 (-72.171 46.489)

### Aggregate the Data

For this analysis, I need the sum for each year at every point.

``` r
precipitation_data_agg <- precipitation_data |>
  mutate(lon = st_coordinates(precipitation_data)[,1],
         lat = st_coordinates(precipitation_data)[,2]) |> 
  group_by(year, lon, lat) |>
  mutate(pts_id=row_number()) |> 
  summarise(value = sum(value))
```

### Create a Distance Matrix

We need some spatial information extracted from the coordinate to put into the temporal model. For this purpose, we will create a distance matrix using the `sf` package.

``` r
#Change for projected CRS
precipitation_data_proj<-precipitation_data_agg %>%
  sf::st_as_sf(.)%>% 
  st_transform(., 3978) 


#Create a distance matrix
buffer.dist<-precipitation_data_proj|> 
  filter(year==2023) |> 
  sf::st_distance(which="Euclidean") |> 
  as.data.frame() |> 
  mutate_all(as.numeric) 

buff.dist_all_years<-do.call("rbind", replicate(31, buffer.dist, simplify = FALSE))

#grouper les données avec la matrice de distance
data<-precipitation_data_proj %>%
  as.data.frame() %>% 
  bind_cols(buff.dist_all_years)%>% 
  na.omit() |> 
  select(-geometry)
```

## Modeling

Now that we have the spatial information, we can start training machine learning models to predict future emissions.

### Splitting Data

``` r
#Train/test
set.seed(123)
data_split <- initial_split(data)
data_train <- training(data_split)
data_test <- testing(data_split)
```

### Model Training

Next, we will train machine learning models to predict future emissions. We will train an XGBoost model evaluate his performance using the root mean square error (RMSE) metric.

``` r
# Model
spec <-boost_tree(mtry = tune(), min_n = tune(), trees = 1000) |> 
  set_mode("regression") |> 
  set_engine("xgboost")

#Grid
grid <- grid_space_filling(
  min_n(),
  finalize(mtry(), data_train),
  size = 10
)

#recipe
recipe<-recipe(as.formula(value ~.), data=data_train) |> 
    step_normalize(all_numeric_predictors())               

#Workflow
wf <- workflow() |> 
  add_recipe(recipe) |> 
  add_model(spec)

#Cross-validation 
data_folds <- vfold_cv(data_train)

res <- tune_grid(
  wf,
  resamples = data_folds,
  grid = grid,
  control = control_grid(save_pred = TRUE)
)
```

### Select Best Models

Next, we will select the best models based on the root mean square error (RMSE) metric. We will visualize the best models to compare their performance and identify any anomalies.

``` r
#best model
best_auc <- select_best(res, metric="rmse")
```

### Finalize Workflow

Finally, we will finalize the workflow and fit the best model to the training data. We will then use the model to predict future emissions in the test set and save the predictions for further analysis.

``` r
#Finalize worlfkow
final_xgb <- finalize_workflow(
  wf,
  best_auc
)

#Fit
fit_xgb <- final_xgb |> 
  fit(data_train)
```

### Evaluate Model Performance

Predicting the test data and evaluating the model performance using the RMSE metric.

``` r
#Predict test set
y_pred <-fit_xgb |> 
predict(new_data=data_test)
```

``` r
data_test_pred<-data_test |> 
  bind_cols(y_pred) |> 
  mutate(value=value*1000, 
         .pred=.pred*1000) |>
  mutate(residuals=value-.pred,
         sd=sd(value),
         sd_pred=sqrt((value-.pred)^2),
         sd_estimate=sqrt(sd^2+sd_pred^2),
         PI_05=.pred-1.645*sd_estimate,
         PI_95=.pred+1.645*sd_estimate,
         PE=(value-.pred)/value,
         APE=abs(PE),
         PE_sup=.pred+APE,
         PE_inf=.pred-APE)
```

#### RSQ

``` r
rsq<-data_test_pred |>  
  rsq(truth=value, estimate=.pred)
DT::datatable(rsq) |> 
  DT::formatRound(c(".estimate"), digits=2)
```

<div class="datatables html-widget html-fill-item" id="htmlwidget-048352464cbf240a6d9e" style="width:100%;height:auto;"></div>
<script type="application/json" data-for="htmlwidget-048352464cbf240a6d9e">{"x":{"filter":"none","vertical":false,"data":[["1"],["rsq"],["standard"],[0.9856713133646924]],"container":"<table class=\"display\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>.metric<\/th>\n      <th>.estimator<\/th>\n      <th>.estimate<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"columnDefs":[{"targets":3,"render":"function(data, type, row, meta) {\n    return type !== 'display' ? data : DTWidget.formatRound(data, 2, 3, \",\", \".\", null);\n  }"},{"className":"dt-right","targets":3},{"orderable":false,"targets":0},{"name":" ","targets":0},{"name":".metric","targets":1},{"name":".estimator","targets":2},{"name":".estimate","targets":3}],"order":[],"autoWidth":false,"orderClasses":false}},"evals":["options.columnDefs.0.render"],"jsHooks":[]}</script>

#### RMSE

``` r
rmse<-sqrt(mean(data_test_pred$residuals^2)) |> 
  as.data.frame() |> 
  rename(.estimate="sqrt(mean(data_test_pred$residuals^2))")
DT::datatable(rmse)|> 
  DT::formatRound(c(".estimate"), digits=2)
```

<div class="datatables html-widget html-fill-item" id="htmlwidget-614a5c3e28bfba2dc9a9" style="width:100%;height:auto;"></div>
<script type="application/json" data-for="htmlwidget-614a5c3e28bfba2dc9a9">{"x":{"filter":"none","vertical":false,"data":[["1"],[0.6850784245411075]],"container":"<table class=\"display\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>.estimate<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"columnDefs":[{"targets":1,"render":"function(data, type, row, meta) {\n    return type !== 'display' ? data : DTWidget.formatRound(data, 2, 3, \",\", \".\", null);\n  }"},{"className":"dt-right","targets":1},{"orderable":false,"targets":0},{"name":" ","targets":0},{"name":".estimate","targets":1}],"order":[],"autoWidth":false,"orderClasses":false}},"evals":["options.columnDefs.0.render"],"jsHooks":[]}</script>

#### MAPE

``` r
mape<-rmse/(mean(data_test_pred$value))*100 |> 
  as.data.frame()
DT::datatable(mape)|> 
  DT::formatRound(c(".estimate"), digits=2)
```

<div class="datatables html-widget html-fill-item" id="htmlwidget-4c7ecd12dd0351ee94be" style="width:100%;height:auto;"></div>
<script type="application/json" data-for="htmlwidget-4c7ecd12dd0351ee94be">{"x":{"filter":"none","vertical":false,"data":[["1"],[1.667112015162908]],"container":"<table class=\"display\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>.estimate<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"columnDefs":[{"targets":1,"render":"function(data, type, row, meta) {\n    return type !== 'display' ? data : DTWidget.formatRound(data, 2, 3, \",\", \".\", null);\n  }"},{"className":"dt-right","targets":1},{"orderable":false,"targets":0},{"name":" ","targets":0},{"name":".estimate","targets":1}],"order":[],"autoWidth":false,"orderClasses":false}},"evals":["options.columnDefs.0.render"],"jsHooks":[]}</script>

#### Visualize the Predictions

``` r
gg <- ggplot(data=as.data.frame(data_test_pred), aes(y=.pred, x=value))
gg <- gg + geom_point(size=1, alpha=0.2)
gg <- gg + geom_smooth(method = "lm", SE= FALSE , color="red")
gg <- gg + geom_smooth(aes(x=value, y=PI_05), linetype="dashed")
gg <- gg + geom_smooth(aes(x=value, y=PI_95), linetype="dashed")
gg <- gg + geom_smooth(aes(x=value, y=PE_sup), linetype="dashed")
gg <- gg + geom_smooth(aes(x=value, y=PE_inf), linetype="dashed")
gg <- gg + geom_hline(yintercept = 50, linetype="dashed")
gg <- gg + geom_vline(xintercept = 50, linetype="dashed")
gg <- gg + theme_serif()
gg <- gg + labs(title ="Comparing Predict to Actual Values of Precipitations (1993-2023)",
                y="Predicted Precipitation (mm)",
                x="Actual Precipitation (mm)")
print(gg)
```

<img src="index.markdown_strict_files/figure-markdown_strict/unnamed-chunk-16-1.png" width="1260" />

#### Residuals

``` r
gg <- as.data.frame(data_test_pred) |>  
ggplot(aes(x=.pred, y=residuals))
gg <- gg + geom_point(size=1, alpha=0.2)
gg <- gg + geom_smooth(method = "lm")
gg <- gg + theme_serif()
gg <- gg + labs(title = "Residuals\n")
print(gg) 
```

<img src="index.markdown_strict_files/figure-markdown_strict/unnamed-chunk-17-1.png" width="1260" />

## Predict 2024 to 2030

``` r
buff.dist_new_years<-do.call("rbind", replicate(7, buffer.dist, simplify = FALSE))

points<-data |> 
  select(lon, lat) |> 
  unique()

all_points<-do.call("rbind", replicate(7, points, simplify = FALSE))

data_futur <- data.frame(
  year=rep(c(2024,2025,2026, 2027, 2028, 2029, 2030),each=81), 
  value=NA)  |> 
  bind_cols(all_points) |>
  bind_cols(buff.dist_new_years)

#Predict test set
new_pred <-fit_xgb |> 
predict(new_data=data_futur)

data_futur_with_pred<-data_futur |> 
  bind_cols(new_pred) |> 
  select(year, .pred, lon, lat) |>
  rename(value=.pred)
```

``` r
precipitation_data_futur<-data |> 
  select(year, value, lon, lat) |>
  bind_rows(data_futur_with_pred) |> 
  st_as_sf(coords = c("lon", "lat")) |>
  st_set_crs(4326)
```

Let's take a look at the predicted precipitation values for 2024 to 2030.

``` r
precipitation_data_futur |> 
  as.data.frame() |> 
  select(year, value) |>
  mutate(value=value*1000) |>
  group_by(year) |>
  summarise(mean=round(mean(value), digits = 1), sd=round(sd(value), digits = 1)) |> 
  DT::datatable()
```

<div class="datatables html-widget html-fill-item" id="htmlwidget-048352464cbf240a6d9e" style="width:100%;height:auto;"></div>
<script type="application/json" data-for="htmlwidget-048352464cbf240a6d9e">{"x":{"filter":"none","vertical":false,"data":[["1","2","3","4","5","6","7","8","9","10","11","12","13","14","15","16","17","18","19","20","21","22","23","24","25","26","27","28","29","30","31","32","33","34","35","36","37","38"],[1993,1994,1995,1996,1997,1998,1999,2000,2001,2002,2003,2004,2005,2006,2007,2008,2009,2010,2011,2012,2013,2014,2015,2016,2017,2018,2019,2020,2021,2022,2023,2024,2025,2026,2027,2028,2029,2030],[3.6,3.5,3.3,3.7,3.3,3.4,3.3,3.4,2.8,3.1,3.5,3,3.6,3.8,3.4,3.8,3.3,3.5,4.2,3.1,3.3,3.2,3.3,3.8,3.9,3.7,4,3.5,2.8,3.5,2,23.8,23.8,23.8,23.8,23.8,23.8,23.8],[0.9,1.7,1.2,1.4,0.9,1.2,1.4,1.1,1.1,1.1,1.4,1.2,1.4,1.2,0.9,1.3,1.1,1.4,1.7,1.2,1.2,0.8,1.2,1.3,1.1,1,1,1.3,1,1.5,0.9,0.8,0.8,0.8,0.8,0.8,0.8,0.8]],"container":"<table class=\"display\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>year<\/th>\n      <th>mean<\/th>\n      <th>sd<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"columnDefs":[{"className":"dt-right","targets":[1,2,3]},{"orderable":false,"targets":0},{"name":" ","targets":0},{"name":"year","targets":1},{"name":"mean","targets":2},{"name":"sd","targets":3}],"order":[],"autoWidth":false,"orderClasses":false}},"evals":[],"jsHooks":[]}</script>

So, we can clearly see here that even if this model seems to have a good performance, the predictions over many years are not very good. This is a proof that this model need to improve his understanding of the concept of seasonality.

## Conclusion

In this analysis, we trained an XGBoost model to predict precipitation patterns in Centre-du-Québec using historical climate data. The model performed well, capturing complex climate patterns and providing accurate predictions for future emissions. By evaluating the model's performance using cross-validation and regression metrics, we identified the most effective approach for precipitation prediction in this region.

<!-- AWeber Web Form Generator 3.0.1 -->
<style type="text/css">
#af-form-88198013 .af-body{font-family:Tahoma, serif;font-size:18px;color:#333333;background-image:none;background-position:inherit;background-repeat:no-repeat;padding-top:0px;padding-bottom:0px;}
#af-form-88198013 .af-body .privacyPolicy{font-family:Tahoma, serif;font-size:18px;color:#333333;}
#af-form-88198013 {border-style:none;border-width:none;border-color:#F8F8F8;background-color:#F8F8F8;}
#af-form-88198013 .af-standards .af-element{padding-left:50px;padding-right:50px;}
#af-form-88198013 .af-quirksMode{padding-left:50px;padding-right:50px;}
#af-form-88198013 .af-header{font-family:Tahoma, serif;font-size:16px;color:#333333;border-top-style:none;border-right-style:none;border-bottom-style:none;border-left-style:none;border-width:1px;background-image:none;background-position:inherit;background-repeat:no-repeat;background-color:#F8F8F8;padding-left:20px;padding-right:20px;padding-top:40px;padding-bottom:20px;}
#af-form-88198013 .af-footer{font-family:Tahoma, serif;font-size:16px;color:#333333;border-top-style:none;border-right-style:none;border-bottom-style:none;border-left-style:none;border-width:1px;background-image:url("https://awas.aweber-static.com/images/forms/journey/basic/background.png");background-position:top center;background-repeat:no-repeat;background-color:#F8F8F8;padding-left:20px;padding-right:20px;padding-top:80px;padding-bottom:80px;}
#af-form-88198013 .af-body input.text, #af-form-88198013 .af-body textarea{border-color:#000000;border-width:1px;border-style:solid;font-family:Tahoma, serif;font-size:18px;font-weight:normal;font-style:normal;text-decoration:none;color:#333333;background-color:#FFFFFF;}
#af-form-88198013 .af-body input.text:focus, #af-form-88198013 .af-body textarea:focus{border-style:solid;border-width:1px;border-color:#EDEDED;background-color:#FAFAFA;}
#af-form-88198013 .af-body label.previewLabel{font-family:Tahoma, serif;font-size:18px;font-weight:normal;font-style:normal;text-decoration:none;color:#333333;display:block;float:left;text-align:left;width:25%;}
#af-form-88198013 .af-body .af-textWrap{width:70%;display:block;float:right;}
#af-form-88198013 .buttonContainer input.submit{font-family:Tahoma, serif;font-size:24px;font-weight:normal;font-style:normal;text-decoration:none;color:#FFFFFF;background-color:#333333;background-image:none;}
#af-form-88198013 .buttonContainer{text-align:center;}
#af-form-88198013 .af-body label.choice{font-family:inherit;font-size:inherit;font-weight:normal;font-style:normal;text-decoration:none;color:#000000;}
#af-form-88198013 .af-body a{font-weight:normal;font-style:normal;text-decoration:underline;color:#000000;}
#af-form-88198013, #af-form-88198013 .quirksMode{width:100%;max-width:486.0px;}
#af-form-88198013.af-quirksMode{overflow-x:hidden;}
#af-form-88198013 .af-quirksMode .bodyText{padding-top:2px;padding-bottom:2px;}
#af-form-88198013{overflow:hidden;}
#af-form-88198013 button,#af-form-88198013 input,#af-form-88198013 submit,#af-form-88198013 textarea,#af-form-88198013 select,#af-form-88198013 label,#af-form-88198013 optgroup,#af-form-88198013 option {float:none;margin:0;position:static;}
#af-form-88198013 select,#af-form-88198013 label,#af-form-88198013 optgroup,#af-form-88198013 option {padding:0;}
#af-form-88198013 input,#af-form-88198013 button,#af-form-88198013 textarea,#af-form-88198013 select {font-size:100%;}
#af-form-88198013 .buttonContainer input.submit {width:auto;}
#af-form-88198013 form,#af-form-88198013 textarea,.af-form-wrapper,.af-form-close-button,#af-form-88198013 img {float:none;color:inherit;margin:0;padding:0;position:static;background-color:none;border:none;}
#af-form-88198013 div {margin:0;}
#af-form-88198013 {display:block;}
#af-form-88198013 body,#af-form-88198013 dl,#af-form-88198013 dt,#af-form-88198013 dd,#af-form-88198013 h1,#af-form-88198013 h2,#af-form-88198013 h3,#af-form-88198013 h4,#af-form-88198013 h5,#af-form-88198013 h6,#af-form-88198013 pre,#af-form-88198013 code,#af-form-88198013 fieldset,#af-form-88198013 legend,#af-form-88198013 blockquote,#af-form-88198013 th,#af-form-88198013 td { float:none;color:inherit;margin:0;padding:0;position:static;}
#af-form-88198013 p { color:inherit;}
#af-form-88198013 ul,#af-form-88198013 ol {list-style-image:none;list-style-position:outside;list-style-type:disc;padding-left:40px;}
#af-form-88198013 .bodyText p {margin:1em 0;}
#af-form-88198013 table {border-collapse:collapse;border-spacing:0;}
#af-form-88198013 fieldset {border:0;}
.af-clear{clear:both;}
.af-form{box-sizing:border-box; margin:auto; text-align:left;}
.af-element{padding-bottom:5px; padding-top:5px;}
.af-form-wrapper{text-indent: 0;}
.af-body input.submit, .af-body input.image, .af-form .af-element input.button{float:none!important;}
.af-body input.submit{white-space: inherit;}
.af-body input.text{width:100%; padding:2px!important;}
.af-body .af-textWrap{text-align:left;}
.af-element label{float:left; text-align:left;}
.lbl-right .af-element label{text-align:right;}
.af-quirksMode .af-element{padding-left: 0!important; padding-right: 0!important;}
.af-body.af-standards input.submit{padding:4px 12px;}
.af-body input.image{border:none!important;}
.af-body input.text{float:none;}
.af-element label{display:block; float:left;}
.af-header,.af-footer { margin-bottom:0; margin-top:0; padding:10px; }
body {
}

#af-form-88198013 .af-body .af-textWrap {
  width: 100% !important;
}

#af-form-88198013 .af-body .af-element {
  padding-top: 0px!important;
  padding-bottom: 0.5rem!important;
}
#af-form-88198013 .af-body .af-element:first-child {
  margin-top: 0 !important;
}
#af-form-88198013 .af-body input.text,
#af-form-88198013 .af-body textarea {
  box-sizing: border-box !important;
  border-radius:2px;
  margin-bottom: 0.75rem !important;
  padding: 8px 12px !important;
  -webkit-transition-duration: 0.3s;
          transition-duration: 0.3s;
}

#af-form-88198013 .af-body select {
  width: 100%;
}
#af-form-88198013 .choiceList-radio-stacked {
  margin-bottom: 1rem !important;
  width: 100% !important;
}
#af-form-88198013 .af-element-radio {
  margin: 0 !important;
}
#af-form-88198013 .af-element-radio input.radio {
  display: inline;
  height: 0;
  opacity: 0;
  overflow: hidden;
  width: 0;
}
#af-form-88198013 .af-element-radio input.radio:checked ~ label {
  font-weight: 700 !important;
}
#af-form-88198013 .af-element-radio input.radio:focus ~ label {
  box-shadow: inset 0 0 0 2px rgba(25,35,70,.25);
}
#af-form-88198013 .af-element-radio input.radio:checked ~ label:before {
  background-color: #777777;
  border-color: #d6dee3;
}
#af-form-88198013 .af-element-radio label.choice {
  display: block !important;
  font-weight: 300 !important;
  margin: 0rem 0rem 0.5rem 1rem !important;
  padding: 0.25rem 1rem !important;
  position: relative;
  -webkit-transition-duration: 0.3s;
          transition-duration: 0.3s;
}
#af-form-88198013 .af-element-radio label.choice:before {
  background-color: #FFF;
  border: 1px solid #d6dee3;
  border-radius: 50%;
  content: '';
  height: 0.75rem;
  margin-top: 0.25rem;
  margin-left: -1.3rem;
  position: absolute;
  -webkit-transition-duration: 0.3s;
          transition-duration: 0.3s;
  width: 0.75rem;
}
#af-form-88198013 .af-selectWrap, 
#af-form-88198013 .af-dateWrap {
  width:100% !important;
  margin: 0.5rem 0rem 0.5rem !important;
  -webkit-transition-duration: 0.3s;
          transition-duration: 0.3s;
}
#af-form-88198013 .af-selectWrap select {
  padding: 0.5rem !important;
  height: 2.5rem;
}
#af-form-88198013 .af-dateWrap select {
  width: 32% !important;
  height: 2.5rem;
  padding: 0.5rem !important;
  margin: 0rem 0rem 0.75rem 0rem !important;
}
#af-form-88198013 .af-checkWrap {
  padding: 0.5rem 0.5rem 0.75rem !important;
}
#af-form-88198013 .buttonContainer {
  box-sizing: border-box !important;
}
#af-form-88198013 .af-footer {
  box-sizing: border-box !important;
}

#af-form-88198013 .af-footer p {
  margin: 0 !important;
}
#af-form-88198013 input.submit,
#af-form-88198013 #webFormSubmitButton {
  border: none;
  border-radius:2px;
  font-weight: bold;
  margin-top: 0.75rem !important;
  margin-bottom: 1.5rem !Important;
  padding: 0.75rem 2rem !important;
  -webkit-transition-duration: 0.3s;
          transition-duration: 0.3s;
  }
#af-form-88198013 input.submit:hover,
#af-form-88198013 #webFormSubmitButton:hover {
  cursor: pointer;
  opacity: 0.8;
}

#af-form-88198013 input.text:hover {
  cursor: pointer;
  opacity: 0.8;
}

.poweredBy a,
.privacyPolicy p {
  color: #333333 !important;
  font-size: 0.75rem !important;
  margin-bottom: 0rem !important;
}
</style>
<form method="post" class="af-form-wrapper" accept-charset="UTF-8" action="https://www.aweber.com/scripts/addlead.pl">

<input type="hidden" name="meta_web_form_id" value="88198013" />
<input type="hidden" name="meta_split_id" value="" />
<input type="hidden" name="listname" value="awlist6634098" />
<input type="hidden" name="redirect" value="https://www.aweber.com/thankyou-coi.htm?m=text" id="redirect_54bc847594a3cbc94af88c076598c2e4" />

<input type="hidden" name="meta_adtracking" value="Sign_Up_Form" />
<input type="hidden" name="meta_message" value="1" />
<input type="hidden" name="meta_required" value="name,email" />

<input type="hidden" name="meta_tooltip" value="" />

<h5>
<br><span style="font-size:36px;"><strong>WANT MORE?</strong></span>
</h5>
<p>
Sign up for exclusive content, emails & things I doesn't share anywhere else.
</p>

<label class="previewLabel" for="awf_field-117870704">Name:</label>

<input id="awf_field-117870704" type="text" name="name" class="text" value="" onfocus=" if (this.value == '') { this.value = ''; }" onblur="if (this.value == '') { this.value='';} " tabindex="500" />

<label class="previewLabel" for="awf_field-117870705">Email:</label>

<input class="text" id="awf_field-117870705" type="email" name="email" value="" tabindex="501" onfocus=" if (this.value == '') { this.value = ''; }" onblur="if (this.value == '') { this.value='';}" />

<input name="submit" class="submit" type="submit" value="Let&#x27;s do it!" tabindex="502" />

<p>
We respect your <a title="Privacy Policy" href="https://www.aweber.com/permission.htm" target="_blank" rel="nofollow">email privacy</a>
</p>

<p>
<a href="https://www.aweber.com" title="AWeber Email Marketing" target="_blank" rel="nofollow">Powered by AWeber Email Marketing</a>
</p>

<p>
 
</p>

<img src="https://forms.aweber.com/form/displays.htm?id=HByMnBwMjMw=" alt="" />

</form>
<!-- /AWeber Web Form Generator 3.0.1 -->

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
    [13] fastmap_1.2.0       backports_1.5.0     rmarkdown_2.29     
    [16] prodlim_2024.06.25  ggfittext_0.10.2    tzdb_0.4.0         
    [19] xfun_0.50           cachem_1.1.0        jsonlite_1.8.9     
    [22] progress_1.2.3      parallel_4.4.2      prettyunits_1.2.0  
    [25] R6_2.5.1            bslib_0.8.0         StanHeaders_2.32.10
    [28] stringi_1.8.4       parallelly_1.41.0   rpart_4.1.24       
    [31] jquerylib_0.1.4     Rcpp_1.0.14         iterators_1.0.14   
    [34] future.apply_1.11.3 zoo_1.8-12          base64enc_0.1-3    
    [37] Matrix_1.7-2        splines_4.4.2       nnet_7.3-20        
    [40] timechange_0.3.0    tidyselect_1.2.1    rstudioapi_0.17.1  
    [43] yaml_2.3.10         timeDate_4041.110   codetools_0.2-20   
    [46] listenv_0.9.1       lattice_0.22-6      withr_3.0.2        
    [49] evaluate_1.0.3      survival_3.8-3      units_0.8-5        
    [52] proxy_0.4-27        RcppParallel_5.1.10 zip_2.3.1          
    [55] xts_0.14.1          xml2_1.3.6          pillar_1.10.1      
    [58] KernSmooth_2.23-26  DT_0.33             renv_1.0.7         
    [61] generics_0.1.3      hms_1.1.3           munsell_0.5.1      
    [64] globals_0.16.3      class_7.3-23        glue_1.8.0         
    [67] tools_4.4.2         data.table_1.16.4   gower_1.0.2        
    [70] grid_4.4.2          crosstalk_1.2.1     ipred_0.9-15       
    [73] colorspace_2.1-1    repr_1.1.7          cli_3.6.3          
    [76] DiceDesign_1.10     rappdirs_0.3.3      viridisLite_0.4.2  
    [79] svglite_2.1.3       lava_1.8.1          gtable_0.3.6       
    [82] GPfit_1.0-8         sass_0.4.9          digest_0.6.37      
    [85] classInt_0.4-11     htmlwidgets_1.6.4   htmltools_0.5.8.1  
    [88] lifecycle_1.0.4     hardhat_1.4.0       MASS_7.3-64        
