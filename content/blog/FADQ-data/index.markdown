---
title: "FADQ historical crops data"
author: Johanie Fournier, agr. 
date: "2022-05-22"
slug: FADQ-data
categories:
  - rstats
  - tidyverse
tags:
  - rstats
  - tidyverse
subtitle: ''
summary: "I have a lot of things to try... so I need a lot of data to play with. Here I summarize you how I extract FADQ historic data. I'm going to place the tidy data in a repro on github and play with it for my next blogs."
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: false
projects: []
---

<script src="{{< blogdown/postref >}}index_files/kePrint/kePrint.js"></script>
<link href="{{< blogdown/postref >}}index_files/lightable/lightable.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index_files/htmlwidgets/htmlwidgets.js"></script>
<link href="{{< blogdown/postref >}}index_files/datatables-css/datatables-crosstalk.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index_files/datatables-binding/datatables.js"></script>
<script src="{{< blogdown/postref >}}index_files/jquery/jquery-3.6.0.min.js"></script>
<link href="{{< blogdown/postref >}}index_files/dt-core/css/jquery.dataTables.min.css" rel="stylesheet" />
<link href="{{< blogdown/postref >}}index_files/dt-core/css/jquery.dataTables.extra.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index_files/dt-core/js/jquery.dataTables.min.js"></script>
<link href="{{< blogdown/postref >}}index_files/crosstalk/css/crosstalk.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index_files/crosstalk/js/crosstalk.min.js"></script>

[FADQ](https://www.fadq.qc.ca/accueil/) here in Quebec, is generous enough to put on their website all the historical data of crop used for each Quebec’s fiend back to 2003. You can find it [here](https://www.fadq.qc.ca/documents/donnees/base-de-donnees-des-parcelles-et-productions-agricoles-declarees/)

I download all of it on my computer and you can find the process in [this github repro](https://github.com/Jofou/PINs/tree/master/FADQ/1%20Import).

Now, let’s see what we got in this data set!

## Get the data

``` r
board_prepared <- pins::board_folder(path_to_file, versioned = TRUE)

data <- board_prepared %>%
  pins::pin_read("centroid")

head(data)
```

    ##           x        y   an  idanpar suphec culture_fadq
    ## 1 -78.72440 48.41762 2003 12388602    5.9     paturage
    ## 2 -77.92157 48.23231 2003 13489650   12.5         foin
    ## 3 -79.38860 48.75419 2003  9809436   13.2         foin
    ## 4 -78.04803 48.36055 2003 12382926    9.6         foin
    ## 5 -79.38775 48.60837 2003 12384941   11.6         foin
    ## 6 -79.02590 48.69386 2003  9809848   12.5     paturage

## Explore the data

In this dataset, **IDANPAR** is a unique ID for each combination field/year. **X** and **Y** are GPS coordinate.

First, I want to look at the number of field we have for each year.

``` r
ggplot(data) +
  geom_histogram(aes(an),
    position = "identity", binwidth = 1,
    fill = "#DBBDC3", color = "black"
  ) +
  labs(
    title = "Frequency Histogram for Year",
    x = " "
  )
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-3-1.png" width="2400" />

The number of information available for each year is constant. This doesn’t represent all the cultivated field of Quebec, because adhesion to FADQ is not mandatory, but this gives a pretty good idea.

Let’s have a look at the crop.

``` r
data %>%
  count(culture_fadq) %>%
  filter(n >= 5000) %>%
  ggplot(aes(reorder(culture_fadq, n), n)) +
  geom_bar(stat = "identity", fill = "#DBBDC3", color = "black") +
  coord_flip() +
  labs(
    title = "Frequency Histogram for Crops",
    x = " "
  )
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-1.png" width="2400" />

Why is NA the main crop in this data? I looked in the original data and there are indeed a lot of missing crops. Since this information is not available, I think it’s safe to delete those lines.

Now let’s see how the field area look like.

``` r
ggplot(data) +
  geom_histogram(aes(suphec),
    position = "identity", binwidth = 1,
    fill = "#DBBDC3", color = "black"
  ) +
  labs(
    title = "Frequency Histogram for Field Area",
    x = " "
  )
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-5-1.png" width="2400" />

``` r
ggplot(data) +
  geom_boxplot(aes(suphec), fill = "#DBBDC3", color = "black") +
  labs(
    title = "Boxplot for Field Area",
    x = " "
  )
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-5-2.png" width="2400" />

``` r
data %>%
  select(suphec) %>%
  my_inspect_num()
```

<table class="table table-condensed" style="width: auto !important; ">
<caption>
Table 1: Numeric Variables Summary
</caption>
<thead>
<tr>
<th style="text-align:left;">
Name
</th>
<th style="text-align:right;">
Min
</th>
<th style="text-align:right;">
Q1
</th>
<th style="text-align:right;">
Med
</th>
<th style="text-align:right;">
Mean
</th>
<th style="text-align:right;">
Q3
</th>
<th style="text-align:right;">
Max
</th>
<th style="text-align:right;">
SD
</th>
<th style="text-align:right;">
NA (%)
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
suphec
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
2
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:right;">
4
</td>
<td style="text-align:right;">
6
</td>
<td style="text-align:right;">
227
</td>
<td style="text-align:right;">
5
</td>
<td style="text-align:right;">
0
</td>
</tr>
</tbody>
</table>

There are field of more than 200 ha in Quebec! 😨 Let me know, I don’t want to walk all this to get the soil samples 🤣 !

There is a lot of outliers in this. Dealing with outliers is always tricky. I don’t even have found the perfect recipe yet.

So before I can have fun with this data set, it need a bit of cleaning:

-   Remove *NA* crop
-   Simplify crop list
-   Remove outliers in field area

## Tidy data

### Simplify Crops

``` r
data_clean_crops <- data %>%
  filter(!is.na(culture_fadq)) %>%
  mutate(groupe = case_when(
    grepl("foin|panic|feverole|semis direct", culture_fadq) ~ "hay",
    grepl("avoine|ble|orge|seigle|sarrasin|triticale|millet|canola|sorgho|tournesol|epeautre|lin|chanvre", culture_fadq) ~ "cereals",
    grepl("pommes de terre", culture_fadq) ~ "potato",
    grepl("framboisier|framboise|fraisier|fraise|pommier|bleuet|bleuetier|arbustes|coniferes|gadellier|camerise|vigne|poire|canneberges|fruitiers et arbres", culture_fadq) ~ "trees and fruits",
    grepl("paturage", culture_fadq) ~ "pasture",
    grepl("soya", culture_fadq) ~ "soy",
    grepl("mais", culture_fadq) ~ "corn",
    grepl("haricot|chou|brocoli|melon|laitue|oignon|piment|celeris|carotte|panais|radis|rutabaga|zucchini|tomate|betterave|cornichon|rabiole|endive|ail|artichaut|asperge|aubergine|poireau|fines herbes|topinambour|celeri-rave|aneth|epinard|pois|rhubarbe|citrouille|courge|concombre|chou-fleur|tabac|gourganes|echalottes|feverole|navets", culture_fadq) ~ "vegetables",
    grepl("non-cultive|tourbe|engrais vert", culture_fadq) ~ "not cultivated",
    TRUE ~ as.character(.$culture_fadq)
  ))

data_clean_crops %>%
  count(groupe, sort = TRUE) %>%
  DT::datatable()
```

<div id="htmlwidget-1" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-1">{"x":{"filter":"none","vertical":false,"data":[["1","2","3","4","5","6","7","8","9"],["hay","corn","cereals","soy","pasture","not cultivated","vegetables","trees and fruits","potato"],[2124244,1304128,951038,852642,396077,154203,76294,12312,9000]],"container":"<table class=\"display\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>groupe<\/th>\n      <th>n<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"columnDefs":[{"className":"dt-right","targets":2},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false}},"evals":[],"jsHooks":[]}</script>

The crop list look much better now! ✨. Time to deal with the field area outliers.

I mainly use 3 ways of dealing with them:

-   1.  The 5% out

-   2.  [Hampel X84](https://dsf.berkeley.edu/jmh/papers/cleaning-unece.pdf) calculus, a robust outliers detection technique that label as outliers any point that is more than 1.4826 *x* MADs (median absolute deviation) away from the median.

-   3.  Box-Cox transformation

I usual look at the 2 first and chose the one that make the mean and the median the closest (good indication of normal distribution). If I can stay away from transformation I will.

### 5% Out

``` r
summary(data_clean_crops$suphec)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##   0.100   1.600   3.200   4.731   5.900 226.900

``` r
quantile(data_clean_crops$suphec, c(0.05, 0.95))
```

    ##   5%  95% 
    ##  0.5 13.9

``` r
data_clean_crops %>%
  filter(suphec <= 13.9) %>%
  ggplot() +
  geom_histogram(aes(suphec),
    position = "identity", binwidth = 1,
    fill = "#DBBDC3", color = "black"
  ) +
  labs(
    title = "Frequency Histogram for Field Area",
    x = " "
  )
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-7-1.png" width="2400" />

95% of the field contain in this dataset have an area smaller than 13.9 ha. This exclude a lot of field. Is all the field with more than 14 ha all outliers? I don’t think so…

### Hampel X85

``` r
var <- data_clean_crops$suphec

upper_limit <- median(var) + 1.4826 * 2 * mad(var)
upper_limit
```

    ## [1] 11.55279

``` r
data_clean_crops %>%
  filter(suphec <= upper_limit) %>%
  ggplot() +
  geom_histogram(aes(suphec),
    position = "identity", binwidth = 1,
    fill = "#DBBDC3", color = "black"
  ) +
  labs(
    title = "Frequency Histogram for Field Area",
    x = " "
  )
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-8-1.png" width="2400" />
OK, I can see that in this case the Hampel X84 method make the distribution of the variable more normal. But I still can’t consider all the field over 12 or 14 ha as outliers. And this doesn’t help much to normalize the shape of the curve.

### Box-Cox Transformation

I try to avoid transformation at any cost. It usually just make a mess in the interpretation of the model accuracy. In some case, I can’t avoid it. I like to use the Box-Cox method to choose the right transformation to use. This technique is quite simple. The most common transformation are presented here:

| `\(\lambda\)` | Transformation           |
|---------------|--------------------------|
| -2            | `\(\frac{1}{x^2}\)`      |
| -1            | `\(\frac{1}{x}\)`        |
| -0.5          | `\(\frac{1}{\sqrt(x)}\)` |
| 0             | `\(\log(x)\)`            |
| 0.5           | `\(\sqrt(x)\)`           |
| 1             | `\(x\)`                  |
| 2             | `\(x^2\)`                |

``` r
b <- MASS::boxcox(lm(data_clean_crops$suphec ~ 1))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-9-1.png" width="2400" />

``` r
lambda <- b$x[which.max(b$y)]
lambda
```

    ## [1] 0.1010101

The best transformation for the field area is the log transformation.

``` r
data_clean_crops %>%
  mutate(suphec_log = log(suphec)) %>%
  ggplot() +
  geom_histogram(aes(suphec_log),
    position = "identity", binwidth = 1,
    fill = "#DBBDC3", color = "black"
  ) +
  labs(
    title = "Frequency Histogram for Field Area",
    x = " "
  )
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-10-1.png" width="2400" />

Oh! We got something! The log transformation is the best way to make the field area variable look normal and I can keep all the big field.

### Final data set

``` r
data_clean <- data %>%
  filter(!is.na(culture_fadq)) %>%
  mutate(culture = case_when(
    grepl("foin|panic|feverole|semis direct", culture_fadq) ~ "hay",
    grepl("avoine|ble|orge|seigle|sarrasin|triticale|millet|canola|sorgho|tournesol|epeautre|lin|chanvre", culture_fadq) ~ "cereals",
    grepl("pommes de terre", culture_fadq) ~ "potato",
    grepl("framboisier|framboise|fraisier|fraise|pommier|bleuet|bleuetier|arbustes|coniferes|gadellier|camerise|vigne|poire|canneberges|fruitiers et arbres", culture_fadq) ~ "trees and fruits",
    grepl("paturage", culture_fadq) ~ "pasture",
    grepl("soya", culture_fadq) ~ "soy",
    grepl("mais", culture_fadq) ~ "corn",
    grepl("haricot|chou|brocoli|melon|laitue|oignon|piment|celeris|carotte|panais|radis|rutabaga|zucchini|tomate|betterave|cornichon|rabiole|endive|ail|artichaut|asperge|aubergine|poireau|fines herbes|topinambour|celeri-rave|aneth|epinard|pois|rhubarbe|citrouille|courge|concombre|chou-fleur|tabac|gourganes|echalottes|feverole|navets", culture_fadq) ~ "vegetables",
    grepl("non-cultive|tourbe|engrais vert", culture_fadq) ~ "not cultivated",
    TRUE ~ as.character(.$culture_fadq)
  )) %>%
  mutate(suphec_log = log(suphec)) %>%
  select(-culture_fadq)

summary(data_clean)
```

    ##        x                y               an          idanpar        
    ##  Min.   :-79.54   Min.   :44.99   Min.   :2003   Min.   : 9804822  
    ##  1st Qu.:-73.24   1st Qu.:45.65   1st Qu.:2007   1st Qu.:11704702  
    ##  Median :-72.43   Median :46.17   Median :2011   Median :13610854  
    ##  Mean   :-72.24   Mean   :46.48   Mean   :2011   Mean   :13976700  
    ##  3rd Qu.:-71.03   3rd Qu.:47.10   3rd Qu.:2016   3rd Qu.:16231983  
    ##  Max.   :-61.76   Max.   :50.25   Max.   :2021   Max.   :19363670  
    ##      suphec          culture            suphec_log    
    ##  Min.   :  0.100   Length:5879938     Min.   :-2.303  
    ##  1st Qu.:  1.600   Class :character   1st Qu.: 0.470  
    ##  Median :  3.200   Mode  :character   Median : 1.163  
    ##  Mean   :  4.731                      Mean   : 1.100  
    ##  3rd Qu.:  5.900                      3rd Qu.: 1.775  
    ##  Max.   :226.900                      Max.   : 5.425

🎉 I have a tidy dataset. You can fin it [here](https://github.com/Jofou/PINs/tree/master/FADQ/3%20Tidy/fadq_tidy/20220520T165247Z-b35d9)
