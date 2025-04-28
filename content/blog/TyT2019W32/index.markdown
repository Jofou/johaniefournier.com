---
title: "TyT2019W32 - Bob Ross Painting"
author: Johanie Fournier, agr. 
date: "2019-08-07"
slug: TyT2019W32
categories:
  - rstats
  - tidyverse
  - tidytuesday
tags:
  - rstats
  - tidyverse
  - tidytuesday
subtitle: ''
summary: "Initially publish it on my wordpress blog. I put it here for reference purpose."
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: false
projects: []
---




## Get the data


```r
bob_ross <- readr::read_csv("https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2019/2019-08-06/bob-ross.csv")
```

```
## Rows: 403 Columns: 69
```

```
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: ","
## chr  (2): EPISODE, TITLE
## dbl (67): APPLE_FRAME, AURORA_BOREALIS, BARN, BEACH, BOAT, BRIDGE, BUILDING,...
```

```
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

## Explore the data


```r
summary(bob_ross)
```

```
##    EPISODE             TITLE            APPLE_FRAME       AURORA_BOREALIS   
##  Length:403         Length:403         Min.   :0.000000   Min.   :0.000000  
##  Class :character   Class :character   1st Qu.:0.000000   1st Qu.:0.000000  
##  Mode  :character   Mode  :character   Median :0.000000   Median :0.000000  
##                                        Mean   :0.002481   Mean   :0.004963  
##                                        3rd Qu.:0.000000   3rd Qu.:0.000000  
##                                        Max.   :1.000000   Max.   :1.000000  
##       BARN             BEACH            BOAT              BRIDGE       
##  Min.   :0.00000   Min.   :0.000   Min.   :0.000000   Min.   :0.00000  
##  1st Qu.:0.00000   1st Qu.:0.000   1st Qu.:0.000000   1st Qu.:0.00000  
##  Median :0.00000   Median :0.000   Median :0.000000   Median :0.00000  
##  Mean   :0.04218   Mean   :0.067   Mean   :0.004963   Mean   :0.01737  
##  3rd Qu.:0.00000   3rd Qu.:0.000   3rd Qu.:0.000000   3rd Qu.:0.00000  
##  Max.   :1.00000   Max.   :1.000   Max.   :1.000000   Max.   :1.00000  
##     BUILDING            BUSHES           CABIN            CACTUS        
##  Min.   :0.000000   Min.   :0.0000   Min.   :0.0000   Min.   :0.000000  
##  1st Qu.:0.000000   1st Qu.:0.0000   1st Qu.:0.0000   1st Qu.:0.000000  
##  Median :0.000000   Median :0.0000   Median :0.0000   Median :0.000000  
##  Mean   :0.002481   Mean   :0.2978   Mean   :0.1712   Mean   :0.009926  
##  3rd Qu.:0.000000   3rd Qu.:1.0000   3rd Qu.:0.0000   3rd Qu.:0.000000  
##  Max.   :1.000000   Max.   :1.0000   Max.   :1.0000   Max.   :1.000000  
##   CIRCLE_FRAME          CIRRUS            CLIFF             CLOUDS      
##  Min.   :0.000000   Min.   :0.00000   Min.   :0.00000   Min.   :0.0000  
##  1st Qu.:0.000000   1st Qu.:0.00000   1st Qu.:0.00000   1st Qu.:0.0000  
##  Median :0.000000   Median :0.00000   Median :0.00000   Median :0.0000  
##  Mean   :0.004963   Mean   :0.06948   Mean   :0.01985   Mean   :0.4442  
##  3rd Qu.:0.000000   3rd Qu.:0.00000   3rd Qu.:0.00000   3rd Qu.:1.0000  
##  Max.   :1.000000   Max.   :1.00000   Max.   :1.00000   Max.   :1.0000  
##     CONIFER          CUMULUS         DECIDUOUS       DIANE_ANDRE      
##  Min.   :0.0000   Min.   :0.0000   Min.   :0.0000   Min.   :0.000000  
##  1st Qu.:0.0000   1st Qu.:0.0000   1st Qu.:0.0000   1st Qu.:0.000000  
##  Median :1.0000   Median :0.0000   Median :1.0000   Median :0.000000  
##  Mean   :0.5261   Mean   :0.2134   Mean   :0.5633   Mean   :0.002481  
##  3rd Qu.:1.0000   3rd Qu.:0.0000   3rd Qu.:1.0000   3rd Qu.:0.000000  
##  Max.   :1.0000   Max.   :1.0000   Max.   :1.0000   Max.   :1.000000  
##       DOCK          DOUBLE_OVAL_FRAME       FARM              FENCE        
##  Min.   :0.000000   Min.   :0.000000   Min.   :0.000000   Min.   :0.00000  
##  1st Qu.:0.000000   1st Qu.:0.000000   1st Qu.:0.000000   1st Qu.:0.00000  
##  Median :0.000000   Median :0.000000   Median :0.000000   Median :0.00000  
##  Mean   :0.002481   Mean   :0.002481   Mean   :0.002481   Mean   :0.05955  
##  3rd Qu.:0.000000   3rd Qu.:0.000000   3rd Qu.:0.000000   3rd Qu.:0.00000  
##  Max.   :1.000000   Max.   :1.000000   Max.   :1.000000   Max.   :1.00000  
##       FIRE          FLORIDA_FRAME         FLOWERS             FOG         
##  Min.   :0.000000   Min.   :0.000000   Min.   :0.00000   Min.   :0.00000  
##  1st Qu.:0.000000   1st Qu.:0.000000   1st Qu.:0.00000   1st Qu.:0.00000  
##  Median :0.000000   Median :0.000000   Median :0.00000   Median :0.00000  
##  Mean   :0.002481   Mean   :0.002481   Mean   :0.02978   Mean   :0.05707  
##  3rd Qu.:0.000000   3rd Qu.:0.000000   3rd Qu.:0.00000   3rd Qu.:0.00000  
##  Max.   :1.000000   Max.   :1.000000   Max.   :1.00000   Max.   :1.00000  
##      FRAMED           GRASS            GUEST         HALF_CIRCLE_FRAME 
##  Min.   :0.0000   Min.   :0.0000   Min.   :0.00000   Min.   :0.000000  
##  1st Qu.:0.0000   1st Qu.:0.0000   1st Qu.:0.00000   1st Qu.:0.000000  
##  Median :0.0000   Median :0.0000   Median :0.00000   Median :0.000000  
##  Mean   :0.1315   Mean   :0.3524   Mean   :0.05459   Mean   :0.002481  
##  3rd Qu.:0.0000   3rd Qu.:1.0000   3rd Qu.:0.00000   3rd Qu.:0.000000  
##  Max.   :1.0000   Max.   :1.0000   Max.   :1.00000   Max.   :1.000000  
##  HALF_OVAL_FRAME        HILLS              LAKE            LAKES  
##  Min.   :0.000000   Min.   :0.00000   Min.   :0.0000   Min.   :0  
##  1st Qu.:0.000000   1st Qu.:0.00000   1st Qu.:0.0000   1st Qu.:0  
##  Median :0.000000   Median :0.00000   Median :0.0000   Median :0  
##  Mean   :0.002481   Mean   :0.04467   Mean   :0.3548   Mean   :0  
##  3rd Qu.:0.000000   3rd Qu.:0.00000   3rd Qu.:1.0000   3rd Qu.:0  
##  Max.   :1.000000   Max.   :1.00000   Max.   :1.0000   Max.   :0  
##    LIGHTHOUSE            MILL               MOON             MOUNTAIN    
##  Min.   :0.000000   Min.   :0.000000   Min.   :0.000000   Min.   :0.000  
##  1st Qu.:0.000000   1st Qu.:0.000000   1st Qu.:0.000000   1st Qu.:0.000  
##  Median :0.000000   Median :0.000000   Median :0.000000   Median :0.000  
##  Mean   :0.002481   Mean   :0.004963   Mean   :0.007444   Mean   :0.397  
##  3rd Qu.:0.000000   3rd Qu.:0.000000   3rd Qu.:0.000000   3rd Qu.:1.000  
##  Max.   :1.000000   Max.   :1.000000   Max.   :1.000000   Max.   :1.000  
##    MOUNTAINS          NIGHT            OCEAN           OVAL_FRAME     
##  Min.   :0.0000   Min.   :0.0000   Min.   :0.00000   Min.   :0.00000  
##  1st Qu.:0.0000   1st Qu.:0.0000   1st Qu.:0.00000   1st Qu.:0.00000  
##  Median :0.0000   Median :0.0000   Median :0.00000   Median :0.00000  
##  Mean   :0.2457   Mean   :0.0273   Mean   :0.08933   Mean   :0.09429  
##  3rd Qu.:0.0000   3rd Qu.:0.0000   3rd Qu.:0.00000   3rd Qu.:0.00000  
##  Max.   :1.0000   Max.   :1.0000   Max.   :1.00000   Max.   :1.00000  
##    PALM_TREES           PATH            PERSON            PORTRAIT       
##  Min.   :0.00000   Min.   :0.0000   Min.   :0.000000   Min.   :0.000000  
##  1st Qu.:0.00000   1st Qu.:0.0000   1st Qu.:0.000000   1st Qu.:0.000000  
##  Median :0.00000   Median :0.0000   Median :0.000000   Median :0.000000  
##  Mean   :0.02233   Mean   :0.1216   Mean   :0.002481   Mean   :0.007444  
##  3rd Qu.:0.00000   3rd Qu.:0.0000   3rd Qu.:0.000000   3rd Qu.:0.000000  
##  Max.   :1.00000   Max.   :1.0000   Max.   :1.000000   Max.   :1.000000  
##  RECTANGLE_3D_FRAME RECTANGULAR_FRAME      RIVER            ROCKS       
##  Min.   :0.000000   Min.   :0.000000   Min.   :0.0000   Min.   :0.0000  
##  1st Qu.:0.000000   1st Qu.:0.000000   1st Qu.:0.0000   1st Qu.:0.0000  
##  Median :0.000000   Median :0.000000   Median :0.0000   Median :0.0000  
##  Mean   :0.002481   Mean   :0.002481   Mean   :0.3127   Mean   :0.1911  
##  3rd Qu.:0.000000   3rd Qu.:0.000000   3rd Qu.:1.0000   3rd Qu.:0.0000  
##  Max.   :1.000000   Max.   :1.000000   Max.   :1.0000   Max.   :1.0000  
##  SEASHELL_FRAME          SNOW        SNOWY_MOUNTAIN    SPLIT_FRAME      
##  Min.   :0.000000   Min.   :0.0000   Min.   :0.0000   Min.   :0.000000  
##  1st Qu.:0.000000   1st Qu.:0.0000   1st Qu.:0.0000   1st Qu.:0.000000  
##  Median :0.000000   Median :0.0000   Median :0.0000   Median :0.000000  
##  Mean   :0.002481   Mean   :0.1861   Mean   :0.2705   Mean   :0.002481  
##  3rd Qu.:0.000000   3rd Qu.:0.0000   3rd Qu.:1.0000   3rd Qu.:0.000000  
##  Max.   :1.000000   Max.   :1.0000   Max.   :1.0000   Max.   :1.000000  
##    STEVE_ROSS       STRUCTURE           SUN            TOMB_FRAME      
##  Min.   :0.0000   Min.   :0.0000   Min.   :0.00000   Min.   :0.000000  
##  1st Qu.:0.0000   1st Qu.:0.0000   1st Qu.:0.00000   1st Qu.:0.000000  
##  Median :0.0000   Median :0.0000   Median :0.00000   Median :0.000000  
##  Mean   :0.0273   Mean   :0.2109   Mean   :0.09926   Mean   :0.002481  
##  3rd Qu.:0.0000   3rd Qu.:0.0000   3rd Qu.:0.00000   3rd Qu.:0.000000  
##  Max.   :1.0000   Max.   :1.0000   Max.   :1.00000   Max.   :1.000000  
##       TREE            TREES         TRIPLE_FRAME        WATERFALL      
##  Min.   :0.0000   Min.   :0.0000   Min.   :0.000000   Min.   :0.00000  
##  1st Qu.:1.0000   1st Qu.:1.0000   1st Qu.:0.000000   1st Qu.:0.00000  
##  Median :1.0000   Median :1.0000   Median :0.000000   Median :0.00000  
##  Mean   :0.8958   Mean   :0.8362   Mean   :0.002481   Mean   :0.09677  
##  3rd Qu.:1.0000   3rd Qu.:1.0000   3rd Qu.:0.000000   3rd Qu.:0.00000  
##  Max.   :1.0000   Max.   :1.0000   Max.   :1.000000   Max.   :1.00000  
##      WAVES            WINDMILL         WINDOW_FRAME          WINTER      
##  Min.   :0.00000   Min.   :0.000000   Min.   :0.000000   Min.   :0.0000  
##  1st Qu.:0.00000   1st Qu.:0.000000   1st Qu.:0.000000   1st Qu.:0.0000  
##  Median :0.00000   Median :0.000000   Median :0.000000   Median :0.0000  
##  Mean   :0.08437   Mean   :0.002481   Mean   :0.002481   Mean   :0.1712  
##  3rd Qu.:0.00000   3rd Qu.:0.000000   3rd Qu.:0.000000   3rd Qu.:0.0000  
##  Max.   :1.00000   Max.   :1.000000   Max.   :1.000000   Max.   :1.0000  
##   WOOD_FRAMED      
##  Min.   :0.000000  
##  1st Qu.:0.000000  
##  Median :0.000000  
##  Mean   :0.002481  
##  3rd Qu.:0.000000  
##  Max.   :1.000000
```

## Prepare the data


```r
data<-bob_ross %>% 
  janitor::clean_names() %>% 
  separate(episode, into = c("season", "episode"), sep = "E") %>% 
  mutate(season = str_extract(season, "[:digit:]+")) %>% 
  mutate_at(vars(season, episode), as.integer) %>% 
    select(-episode, -title) %>%
  gather(-season, key = "element", value = "count") %>%
  mutate(element = case_when(.$element %in% c("lake", "lakes") ~ "lake",
                             TRUE ~ as.character(.$element))) %>% 
  mutate(element = case_when(.$element %in% c("mountain", "mountains") ~ "mountain",
                             TRUE ~ as.character(.$element))) %>% 
  mutate(element = case_when(.$element %in% c("tree", "trees", "conifer", "palm_trees", "deciduous") ~ "tree",
                             TRUE ~ as.character(.$element))) %>% 
  mutate(element = case_when(.$element %in% c("person", "portrait") ~ "portrait",
                             TRUE ~ as.character(.$element))) %>% 
  mutate(element = case_when(.$element %in% c("cloud", "cumulus") ~ "cloud",
                             TRUE ~ as.character(.$element))) %>% 
  group_by(season, element) %>% 
  summarise(count=sum(count))

#Quels sont les 3 éléments les plus importants dans les peintures:
sommaire<-data %>% 
  group_by(element) %>% 
  summarise(count=sum(count))

#Arbres: 1146
#montagnes:259
#nuages: 179

#modifier la base de données pour visualisation:
freq<-data %>% 
  mutate(element=ifelse(!element %in% c("tree", "mountain", "clouds"), "divers", element))

cat<-freq %>% 
  inspect_cat()

cat$levels$element

data_cat<-freq %>% 
  filter(element %in% c("tree", "mountain", "clouds"))

data_divers<-freq %>% 
  filter(!element %in% c("tree", "mountain", "clouds"))
```

## Visualize the data


```r
#Graphique
gg<-ggplot(data_cat, aes(x=season, y=count, group=element, color=element))
gg <- gg + geom_point(data=data_divers,size=4, alpha=0.3, color="#AFB7BB")
gg <- gg + geom_point(data=data_cat, size=4, alpha=0.7)
gg <- gg + scale_color_manual(values=c("#266DD3", "#F6AE2D","#33673B"))
gg <- gg + geom_smooth(data=data_cat,se=FALSE, size=2)
#retirer la légende
gg <- gg + theme(legend.position = "none")
#ajuster les axes 
gg <- gg + scale_y_continuous(breaks=seq(00,50,10), limits = c(0, 50))
gg <- gg + scale_x_continuous(breaks=seq(1,31,5), limits = c(1, 33.5))
#modifier le thème
gg <- gg +  theme(panel.border = element_blank(),
                    panel.background = element_blank(),
                    plot.background = element_blank(),
                    panel.grid.major.y= element_line(size=0.2,linetype="dotted", color="#6D7C83"),
                    panel.grid.major.x= element_blank(),
                    panel.grid.minor = element_blank(),
                    axis.line.x = element_line(color="#6D7C83"),
                    axis.line.y = element_line(color="#6D7C83"),
                    axis.ticks.y = element_blank(),
                    axis.ticks.x = element_blank())
#ajouter les étiquettes de données
gg<-gg + annotate(geom="text", x=31,y=40, label="Tree", color="#33673B", size=4, hjust=0,vjust=0, fontface="bold")
gg<-gg + annotate(geom="text", x=31,y=5, label="Cloud", color="#266DD3", size=4, hjust=0,vjust=0, fontface="bold")
gg<-gg + annotate(geom="text", x=31,y=11,  label="Mountain", color="#F6AE2D", size=4, hjust=0,vjust=0, fontface="bold")
#ajouter les titres
gg<-gg + labs(title=" ",
              subtitle="",
              y="Number", 
              x="Season")
gg<-gg + theme(plot.title    = element_blank(),
                 plot.subtitle = element_blank(),
                 axis.title.y  = element_text(hjust=1, vjust=0, size=12, color="#6D7C83", face="bold"),
                 axis.title.x  = element_text(hjust=0, vjust=0, size=12, color="#6D7C83", face="bold"),
                 axis.text.y   = element_text(hjust=0.5, vjust=0, size=12, color="#6D7C83", face="bold"), 
                 axis.text.x   = element_text(hjust=0.5, vjust=0, size=12, color="#6D7C83", face="bold"))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-5-1.png" width="100%" style="display: block; margin: auto;" />

