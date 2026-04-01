---
title: 'Integrating Soil Maps with High-Resolution LiDAR for Field-Scale Drainage Modeling in R (Part 1)'
author: Johanie Fournier, agr., M.Sc.
date: "2026-02-24"
draft: true
slug: 2026_LiDAR_soil_polygons_1
categories:
  - rstats
  - tidymodels
  - eda
  - viz
tags:
  - rstats
  - tidymodels
summary: "A practical R workflow to harmonize soil polygons and 1-m LiDAR terrain data for consistent field-scale agricultural drainage modeling."
editor_options: 
  chunk_output_type: inline
---

High-resolution LiDAR enables detailed identification of flow paths, depressions, and wetness patterns at sub-meter resolution. However, soil data --- which governs infiltration and water retention --- is typically represented as coarse polygon maps. When these datasets are combined without scale harmonization, spatial inconsistencies can lead to misleading drainage interpretations. In this article, I present a structured and reproducible R workflow to reconcile soil polygons and LiDAR-derived terrain indices. The focus is on resolving resolution mismatch and defining spatially consistent analytical units before modeling. This post lays the foundation for building reliable, field-scale drainage zoning tools.

[![](petit.png)](https://subscribepage.io/E3ia1B)

<br>

High-resolution LiDAR has enabled mapping of agricultural drainage patterns at the field scale with remarkable precision. We can identify flow paths, depressions, and wet areas at one-meter resolution. But in practice, soil data --- which controls infiltration, permeability, and water storage --- is still delivered as polygon maps at much coarser scales. When these two datasets are combined without careful harmonization, the result is a workflow that looks precise but can produce misleading drainage zones. For agronomists and farm advisors working at an operational scale, this mismatch matters. In this post, I show how to reconcile soil polygons and LiDAR-derived terrain indices in R to build a drainage workflow that is both spatially consistent and practically usable. To address this, we need a spatial workflow that explicitly accounts for resolution mismatch rather than ignoring it.

This article focuses on the spatial integration challenge between soil maps and LiDAR-derived terrain indices. Rather than jumping directly into overlay analysis, we first examine how scale mismatch influences drainage interpretation and then develop a structured R workflow to reconcile both datasets for operational agricultural use.

# data

Present the data for the field scenario
You are solving the resolution mismatch problem.

# The Resolution Trap in Drainage Mapping

LiDAR precision vs generalized soil maps.

# The Real-World Problem

Example: field drainage variability.
Explain why farmers and agronomists struggle to interpret drainage patterns.

# Why Soil and LiDAR Rarely Align

Explain:
LiDAR resolution (1--2 m)
soil survey polygons
spatial support mismatch
false precision
Keep this conceptual but practical.

# System Design Decision #1

Define the Spatial Foundation
Explain that before modeling drainage you must:
harmonize spatial support
standardize terrain metrics
prepare soil attributes
Introduce the system architecture.

# Building the Terrain Layer in R

Steps:
Import DEM
Hydrological conditioning
Terrain derivatives
Example metrics:
slope
curvature
flow accumulation
TWI

# Preparing Soil Data

Steps:
import soil polygons
inspect attributes
rasterize
attach drainage-relevant variables

# Reconciling Spatial Support

Discuss:
raster resolution
polygon-to-raster conversion
analytical unit concept
This is the core insight of the article.

# Result: A Harmonized Spatial Dataset

Show the final layers.
Explain that this becomes the foundation of the drainage modeling system.

# What Comes Next

Preview the next challenge:
Once soil and terrain data are harmonized, the next design decision is choosing the analytical unit used to model drainage variability.
Lead into Post 2

# Conclusion

This post is the first step in a broader framework for building operational drainage zoning tools using LiDAR, soil, and climate data.

# Sign up for the newsletter

[![](sign_up.png)](https://dashboard.mailerlite.com/forms/1478852/152663752035010469/share)

<br>

# Session Info

``` r
sessionInfo()
```

    R version 4.4.2 (2024-10-31)
    Platform: aarch64-apple-darwin20
    Running under: macOS Sequoia 15.6.1

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
     [1] reticulate_1.40.0    jofou.lib_0.0.0.9000 tidytuesdayR_1.1.2  
     [4] tictoc_1.2.1         KrigR_0.9.4          ncdf4_1.23          
     [7] ecmwfr_2.0.2         rgeoboundaries_1.3.1 terra_1.8-10        
    [10] sf_1.0-19            pins_1.4.0           fs_1.6.5            
    [13] timetk_2.9.0         yardstick_1.3.2      workflowsets_1.1.0  
    [16] workflows_1.1.4      tune_1.2.1           rsample_1.2.1       
    [19] parsnip_1.2.1        modeldata_1.4.0      infer_1.0.7         
    [22] dials_1.3.0          scales_1.3.0         broom_1.0.7         
    [25] tidymodels_1.2.0     recipes_1.1.0        doFuture_1.0.1      
    [28] future_1.34.0        foreach_1.5.2        skimr_2.1.5         
    [31] gganimate_1.0.9      forcats_1.0.0        stringr_1.5.1       
    [34] dplyr_1.1.4          purrr_1.0.2          readr_2.1.5         
    [37] tidyr_1.3.1          tibble_3.2.1         ggplot2_3.5.1       
    [40] tidyverse_2.0.0      lubridate_1.9.4      kableExtra_1.4.0    
    [43] inspectdf_0.0.12.1   openxlsx_4.2.7.1     knitr_1.49          

    loaded via a namespace (and not attached):
      [1] rstudioapi_0.17.1   jsonlite_1.8.9      magrittr_2.0.3     
      [4] magick_2.8.5        farver_2.1.2        rmarkdown_2.29     
      [7] vctrs_0.6.5         memoise_2.0.1       hoardr_0.5.5       
     [10] base64enc_0.1-3     htmltools_0.5.8.1   progress_1.2.3     
     [13] curl_6.1.0          parallelly_1.41.0   KernSmooth_2.23-26 
     [16] plyr_1.8.9          zoo_1.8-12          stars_0.6-7        
     [19] cachem_1.1.0        ggfittext_0.10.2    lifecycle_1.0.4    
     [22] iterators_1.0.14    pkgconfig_2.0.3     Matrix_1.7-2       
     [25] R6_2.5.1            fastmap_1.2.0       digest_0.6.37      
     [28] reshape_0.8.9       colorspace_2.1-1    furrr_0.3.1        
     [31] timechange_0.3.0    httr_1.4.7          abind_1.4-8        
     [34] compiler_4.4.2      intervals_0.15.5    proxy_0.4-27       
     [37] withr_3.0.2         backports_1.5.0     viridis_0.6.5      
     [40] DBI_1.2.3           MASS_7.3-64         lava_1.8.1         
     [43] rappdirs_0.3.3      classInt_0.4-11     tools_4.4.2        
     [46] units_0.8-5         zip_2.3.1           future.apply_1.11.3
     [49] nnet_7.3-20         glue_1.8.0          grid_4.4.2         
     [52] snow_0.4-4          generics_0.1.3      gtable_0.3.6       
     [55] countrycode_1.6.0   tzdb_0.4.0          class_7.3-23       
     [58] data.table_1.16.4   hms_1.1.3           sp_2.1-4           
     [61] xml2_1.3.6          pillar_1.10.1       splines_4.4.2      
     [64] lhs_1.2.0           tweenr_2.0.3        lattice_0.22-6     
     [67] FNN_1.1.4.1         renv_1.0.7          survival_3.8-3     
     [70] tidyselect_1.2.1    pbapply_1.7-2       gridExtra_2.3      
     [73] svglite_2.1.3       crul_1.5.0          xfun_0.50          
     [76] hardhat_1.4.0       timeDate_4041.110   stringi_1.8.4      
     [79] DiceDesign_1.10     yaml_2.3.10         evaluate_1.0.3     
     [82] codetools_0.2-20    httpcode_0.3.0      automap_1.1-12     
     [85] cli_3.6.3           rpart_4.1.24        systemfonts_1.2.1  
     [88] repr_1.1.7          munsell_0.5.1       spacetime_1.3-2    
     [91] Rcpp_1.0.14         doSNOW_1.0.20       globals_0.16.3     
     [94] png_0.1-8           parallel_4.4.2      gower_1.0.2        
     [97] prettyunits_1.2.0   GPfit_1.0-8         listenv_0.9.1      
    [100] viridisLite_0.4.2   ipred_0.9-15        xts_0.14.1         
    [103] prodlim_2024.06.25  e1071_1.7-16        gstat_2.1-2        
    [106] crayon_1.5.3        rlang_1.1.5         cowplot_1.1.3      
