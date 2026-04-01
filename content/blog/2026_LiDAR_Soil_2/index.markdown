---
title: 'Choosing the Right Analytical Unit: Pixel-Based vs Soil-Unit Drainage Modeling in R (Part 2)'
author: Johanie Fournier, agr., M.Sc.
date: "2026-02-24"
draft: true
slug: 2026_LiDAR_soil_polygons_2
categories:
  - rstats
  - tidymodels
  - eda
  - viz
tags:
  - rstats
  - tidymodels
summary: "Compare pixel-based and soil-unit drainage modeling in R and learn how analytical unit choice affects field-scale hydrology interpretation."
editor_options: 
  chunk_output_type: inline
---




In drainage modeling, the choice of analytical unit fundamentally shapes interpretation. Should we model drainage variability at the pixel level using LiDAR-derived terrain indices, or aggregate terrain information within soil mapping units? Each approach carries statistical and spatial implications. In this article, I compare pixel-based and soil-unit-based drainage modeling strategies using R, examining how aggregation influences variability, interpretability, and operational relevance. Beyond code implementation, this post explores the conceptual importance of spatial support in agricultural hydrology and precision management.


[![](petit.png)](https://subscribepage.io/E3ia1B)

<br>

High-resolution LiDAR has enabled mapping of agricultural drainage patterns at the field scale with remarkable precision. We can identify flow paths, depressions, and wet areas at one-meter resolution. But in practice, soil data — which controls infiltration, permeability, and water storage — is still delivered as polygon maps at much coarser scales. When these two datasets are combined without careful harmonization, the result is a workflow that looks precise but can produce misleading drainage zones. For agronomists and farm advisors working at an operational scale, this mismatch matters. In this post, I show how to reconcile soil polygons and LiDAR-derived terrain indices in R to build a drainage workflow that is both spatially consistent and practically usable. To address this, we need a spatial workflow that explicitly accounts for resolution mismatch rather than ignoring it.


This article focuses on the spatial integration challenge between soil maps and LiDAR-derived terrain indices. Rather than jumping directly into overlay analysis, we first examine how scale mismatch influences drainage interpretation and then develop a structured R workflow to reconcile both datasets for operational agricultural use.


# Why Analytical Units Matter
Explain:
Different analytical units produce different interpretations of the same landscape.
Introduce:
pixel-based modeling
soil-unit aggregation

# System Design Decision #2
Selecting the Modeling Unit
Frame this as a system choice.
Explain that the modeling unit must match the decision scale.

#Pixel-Based Modeling Approach
Explain:
terrain-driven modeling
soil attributes assigned per pixel
Pros:
captures microtopography
Risks:
false soil precision

# Soil-Unit Aggregation Approach
Explain:
terrain metrics aggregated per soil unit
modeling occurs at polygon scale
Pros:
aligns with soil survey structure
Risks:
smoothing micro-topographic effects

# Comparative Implementation in R
Show:
pixel drainage index
aggregated terrain metrics
visual comparison
Maps and distributions are very powerful here.

# Interpreting the Differences
Explain:
variance changes
spatial smoothing
practical implications
This is where your expertise shines.

# Design Conclusion
State your recommendation clearly.
Example:
For operational drainage zoning, a hybrid strategy often provides the most robust interpretation.

# What Comes Next
Introduce the next step:
Once the modeling unit is defined, the final challenge is translating spatial variability into operational drainage zones.
Lead into Post 3

# Conclusion

This post is the first step in a broader framework for building operational drainage zoning tools using LiDAR, soil, and climate data.


# Sign up for the newsletter

[![](sign_up.png)](https://dashboard.mailerlite.com/forms/1478852/152663752035010469/share)

<br>

# Session Info



``` r
sessionInfo()
```

```
## R version 4.4.2 (2024-10-31)
## Platform: aarch64-apple-darwin20
## Running under: macOS Sequoia 15.6.1
## 
## Matrix products: default
## BLAS:   /Library/Frameworks/R.framework/Versions/4.4-arm64/Resources/lib/libRblas.0.dylib 
## LAPACK: /Library/Frameworks/R.framework/Versions/4.4-arm64/Resources/lib/libRlapack.dylib;  LAPACK version 3.12.0
## 
## locale:
## [1] en_US.UTF-8/en_US.UTF-8/en_US.UTF-8/C/en_US.UTF-8/en_US.UTF-8
## 
## time zone: America/Toronto
## tzcode source: internal
## 
## attached base packages:
## [1] stats     graphics  grDevices datasets  utils     methods   base     
## 
## other attached packages:
##  [1] reticulate_1.40.0    jofou.lib_0.0.0.9000 tidytuesdayR_1.1.2  
##  [4] tictoc_1.2.1         KrigR_0.9.4          ncdf4_1.23          
##  [7] ecmwfr_2.0.2         rgeoboundaries_1.3.1 terra_1.8-10        
## [10] sf_1.0-19            pins_1.4.0           fs_1.6.5            
## [13] timetk_2.9.0         yardstick_1.3.2      workflowsets_1.1.0  
## [16] workflows_1.1.4      tune_1.2.1           rsample_1.2.1       
## [19] parsnip_1.2.1        modeldata_1.4.0      infer_1.0.7         
## [22] dials_1.3.0          scales_1.3.0         broom_1.0.7         
## [25] tidymodels_1.2.0     recipes_1.1.0        doFuture_1.0.1      
## [28] future_1.34.0        foreach_1.5.2        skimr_2.1.5         
## [31] gganimate_1.0.9      forcats_1.0.0        stringr_1.5.1       
## [34] dplyr_1.1.4          purrr_1.0.2          readr_2.1.5         
## [37] tidyr_1.3.1          tibble_3.2.1         ggplot2_3.5.1       
## [40] tidyverse_2.0.0      lubridate_1.9.4      kableExtra_1.4.0    
## [43] inspectdf_0.0.12.1   openxlsx_4.2.7.1     knitr_1.49          
## 
## loaded via a namespace (and not attached):
##   [1] rstudioapi_0.17.1   jsonlite_1.8.9      magrittr_2.0.3     
##   [4] magick_2.8.5        farver_2.1.2        rmarkdown_2.29     
##   [7] vctrs_0.6.5         memoise_2.0.1       hoardr_0.5.5       
##  [10] base64enc_0.1-3     blogdown_1.20       htmltools_0.5.8.1  
##  [13] progress_1.2.3      curl_6.1.0          sass_0.4.9         
##  [16] parallelly_1.41.0   KernSmooth_2.23-26  bslib_0.8.0        
##  [19] plyr_1.8.9          stars_0.6-7         zoo_1.8-12         
##  [22] cachem_1.1.0        ggfittext_0.10.2    lifecycle_1.0.4    
##  [25] iterators_1.0.14    pkgconfig_2.0.3     Matrix_1.7-2       
##  [28] R6_2.5.1            fastmap_1.2.0       digest_0.6.37      
##  [31] reshape_0.8.9       colorspace_2.1-1    furrr_0.3.1        
##  [34] timechange_0.3.0    abind_1.4-8         httr_1.4.7         
##  [37] compiler_4.4.2      intervals_0.15.5    proxy_0.4-27       
##  [40] withr_3.0.2         backports_1.5.0     viridis_0.6.5      
##  [43] DBI_1.2.3           MASS_7.3-64         lava_1.8.1         
##  [46] rappdirs_0.3.3      classInt_0.4-11     tools_4.4.2        
##  [49] units_0.8-5         zip_2.3.1           future.apply_1.11.3
##  [52] nnet_7.3-20         glue_1.8.0          grid_4.4.2         
##  [55] snow_0.4-4          generics_0.1.3      gtable_0.3.6       
##  [58] countrycode_1.6.0   tzdb_0.4.0          class_7.3-23       
##  [61] data.table_1.16.4   hms_1.1.3           sp_2.1-4           
##  [64] xml2_1.3.6          pillar_1.10.1       splines_4.4.2      
##  [67] lhs_1.2.0           tweenr_2.0.3        lattice_0.22-6     
##  [70] FNN_1.1.4.1         renv_1.0.7          survival_3.8-3     
##  [73] tidyselect_1.2.1    pbapply_1.7-2       gridExtra_2.3      
##  [76] bookdown_0.42       svglite_2.1.3       crul_1.5.0         
##  [79] xfun_0.50           hardhat_1.4.0       timeDate_4041.110  
##  [82] stringi_1.8.4       DiceDesign_1.10     yaml_2.3.10        
##  [85] evaluate_1.0.3      codetools_0.2-20    httpcode_0.3.0     
##  [88] automap_1.1-12      cli_3.6.3           rpart_4.1.24       
##  [91] systemfonts_1.2.1   repr_1.1.7          munsell_0.5.1      
##  [94] jquerylib_0.1.4     spacetime_1.3-2     Rcpp_1.0.14        
##  [97] doSNOW_1.0.20       globals_0.16.3      png_0.1-8          
## [100] parallel_4.4.2      gower_1.0.2         prettyunits_1.2.0  
## [103] GPfit_1.0-8         listenv_0.9.1       viridisLite_0.4.2  
## [106] ipred_0.9-15        xts_0.14.1          prodlim_2024.06.25 
## [109] e1071_1.7-16        gstat_2.1-2         crayon_1.5.3       
## [112] rlang_1.1.5         cowplot_1.1.3
```

