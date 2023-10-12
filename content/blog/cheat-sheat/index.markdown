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



I can spend a ridiculously big amout of time looking for a certain peace of code that I made few months ago that will solve my current coding problem but can't remember how to code...

So instead of loosing my time in my archive files or on Google, I decide to start listing here all the snippet of code that I need on hand.

One of this day, I will organize all this into a beautiful cheat sheet, but for now there my little list!

## Geospatial

### CRS
I need to remember that in the coordinate reference system (CRS) of Quebec:
* WGS84:EPSG:4326 is a geographic reference system meaning with longitude and latitude values in degree.
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



[^1]: A little disclosure: I only recommend products I would use myself and all opinions expressed here are my own. This post may contain affiliate links that at no additional cost to you, I may earn a small commission. Thanks for your support!
