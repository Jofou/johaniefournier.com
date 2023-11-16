---
title: "This is the begining of a cheat sheet!"
author: Johanie Fournier, agr. 
date: "2023-10-12"
slug: cheat-sheet
categories:
  - rstats
tags:
  - rstats
subtitle: ''
summary: "I will list here all the little snipset of code that I look up all the time."
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: false
projects: []
---



I can spend a ridiculously big amount of time looking for a certain peace of code that I made few months ago that will solve my current coding problem but can't remember how to code...

So instead of loosing my time in my archive files or on Google, I decide to start listing here all the snippet of code that I need on hand.

One of this day, I will organize all this into a beautiful cheat sheet, but for now there my little list!

## Geospatial

### CRS
I need to remember that in the coordinate reference system (CRS) of Quebec:
* WGS84:EPSG4326 is a geographic reference system meaning with longitude and latitude values in degree.
* NAD83:EPSG3978 is a projected reference system with longitude and latitude values in meter. **Better for any kind of calculation**

### Transform datable to sf object

```r
sf::st_as_sf(.) %>%
  sf::st_transform(., 3978)
```

### Add a buffer distance around points

```r
sf::st_buffer(dist = 20)
```

### Get coordinate from a sf object

```r
mutate(
  longitude = sf::st_coordinates(.)[, 1],
  latitude = sf::st_coordinates(.)[, 2]
)
```

## Markdown in R

### Header


```r
date: "`r format(Sys.time(), '%Y-%m-%d')`"

<script src="https://hypothes.is/embed.js" async></script>
```

### Setup

```r
knitr::opts_chunk$set(
  include = TRUE, # TRUE = run and include the chunk in document
  echo = FALSE, # FALSE = not display the code
  eval = TRUE, # FALSE = not run the code in all chunk
  comment = FALSE,
  message = FALSE,
  warning = FALSE
)
```

### Theme Map

```r
theme_map <- function(base_size=9, base_family="") { # 3
	require(grid)
	theme_bw(base_size=base_size, base_family=base_family) %+replace%
		theme(axis.line=element_blank(),
			  axis.text=element_blank(),
			  axis.ticks=element_blank(),
			  axis.title=element_blank(),
			  panel.background=element_blank(),
			  panel.border=element_blank(),
			  panel.grid=element_blank(),
			  panel.spacing=unit(0, "lines"),
			  plot.background=element_blank(),
			  legend.justification = c(0,0),
			  legend.position = c(0,0),
			  plot.title= element_text(size=20, hjust=0, color="black", face="bold"),
		)
```

### emo::ji!


```r
`r emo::ji("collision")`

`r emo::ji("popper")`

`r emo::ji("bomb")`

`r emo::ji("bug")`

`r emo::ji("chart")`

`r emo::ji("cry")`

`r emo::ji("disaster")`

`r emo::ji("fear")`

`r emo::ji("chart")`

`r emo::ji("flowers")`

`r emo::ji("laugh")`
```

## Parallel Processing

### Start

```r
doFuture::registerDoFuture()
n_cores <- parallel::detectCores() - 1
future::plan(
  strategy = future::cluster,
  workers = parallel::makeCluster(n_cores)
)
```

### Stop

```r
future::plan(future::sequential)
```


## Tidyverse data manipulation

### Replace Inf with NA

```r
df %>%
  mutate_if(is.numeric, list(~ na_if(., Inf))) %>%
  mutate_if(is.numeric, list(~ na_if(., -Inf)))
```


```r
df %>%
  mutate_at(vars(auc_evi), ~ na_if(., Inf))
```

### Rename in a loop

```r
lookup <- c(new_name = "old_name")

df %>%
  rename(any_of(lookup))
```

### Remove cap and accent

```r
df %>%
  mutate(new_name = tolower(stringi::stri_trans_general(old_name, "Latin-ASCII")))
```

### Read all rds file in folder

```r
data <- list.files(path = "path_to_files", pattern = ".rds", full.names = T) %>%
  map_dfr(readRDS) %>%
  bind_rows()
```

### Corrrelation Funnel

```r
library(correlationfunnel)

var <- "name_of_interest_variable"

var_select <- tbl_prep %>%
  select(var) %>%
  binarize() %>%
  select(starts_with(var) & ends_with("_Inf")) %>%
  names()

tbl_prep %>%
  binarize() %>%
  correlate(var_select) %>%
  plot_correlation_funnel() +
  labs(title = paste0(label, " - Correlation Funnel"))
```

