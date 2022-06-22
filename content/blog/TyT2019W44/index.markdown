---
title: "TyT2019W44 - Show Some Informations"
author: Johanie Fournier, agr. 
date: "2019-10-31"
slug: TyT2019W44
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
nyc_squirrels <- readr::read_csv("https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2019/2019-10-29/nyc_squirrels.csv")
```

```
## Rows: 3023 Columns: 36
```

```
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: ","
## chr (14): unique_squirrel_id, hectare, shift, age, primary_fur_color, highli...
## dbl  (9): long, lat, date, hectare_squirrel_number, zip_codes, community_dis...
## lgl (13): running, chasing, climbing, eating, foraging, kuks, quaas, moans, ...
```

```
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

## Explore the data


```r
summary(nyc_squirrels)
```

```
##       long             lat        unique_squirrel_id   hectare         
##  Min.   :-73.98   Min.   :40.76   Length:3023        Length:3023       
##  1st Qu.:-73.97   1st Qu.:40.77   Class :character   Class :character  
##  Median :-73.97   Median :40.78   Mode  :character   Mode  :character  
##  Mean   :-73.97   Mean   :40.78                                        
##  3rd Qu.:-73.96   3rd Qu.:40.79                                        
##  Max.   :-73.95   Max.   :40.80                                        
##                                                                        
##     shift                date          hectare_squirrel_number
##  Length:3023        Min.   :10062018   Min.   : 1.000         
##  Class :character   1st Qu.:10082018   1st Qu.: 2.000         
##  Mode  :character   Median :10122018   Median : 3.000         
##                     Mean   :10119487   Mean   : 4.124         
##                     3rd Qu.:10142018   3rd Qu.: 6.000         
##                     Max.   :10202018   Max.   :23.000         
##                                                               
##      age            primary_fur_color  highlight_fur_color
##  Length:3023        Length:3023        Length:3023        
##  Class :character   Class :character   Class :character   
##  Mode  :character   Mode  :character   Mode  :character   
##                                                           
##                                                           
##                                                           
##                                                           
##  combination_of_primary_and_highlight_color color_notes       
##  Length:3023                                Length:3023       
##  Class :character                           Class :character  
##  Mode  :character                           Mode  :character  
##                                                               
##                                                               
##                                                               
##                                                               
##    location         above_ground_sighter_measurement specific_location 
##  Length:3023        Length:3023                      Length:3023       
##  Class :character   Class :character                 Class :character  
##  Mode  :character   Mode  :character                 Mode  :character  
##                                                                        
##                                                                        
##                                                                        
##                                                                        
##   running         chasing         climbing         eating       
##  Mode :logical   Mode :logical   Mode :logical   Mode :logical  
##  FALSE:2293      FALSE:2744      FALSE:2365      FALSE:2263     
##  TRUE :730       TRUE :279       TRUE :658       TRUE :760      
##                                                                 
##                                                                 
##                                                                 
##                                                                 
##   foraging       other_activities      kuks           quaas        
##  Mode :logical   Length:3023        Mode :logical   Mode :logical  
##  FALSE:1588      Class :character   FALSE:2921      FALSE:2973     
##  TRUE :1435      Mode  :character   TRUE :102       TRUE :50       
##                                                                    
##                                                                    
##                                                                    
##                                                                    
##    moans         tail_flags      tail_twitches   approaches     
##  Mode :logical   Mode :logical   Mode :logical   Mode :logical  
##  FALSE:3020      FALSE:2868      FALSE:2589      FALSE:2845     
##  TRUE :3         TRUE :155       TRUE :434       TRUE :178      
##                                                                 
##                                                                 
##                                                                 
##                                                                 
##  indifferent     runs_from       other_interactions   lat_long        
##  Mode :logical   Mode :logical   Length:3023        Length:3023       
##  FALSE:1569      FALSE:2345      Class :character   Class :character  
##  TRUE :1454      TRUE :678       Mode  :character   Mode  :character  
##                                                                       
##                                                                       
##                                                                       
##                                                                       
##    zip_codes     community_districts borough_boundaries city_council_districts
##  Min.   :10090   Min.   :11          Min.   :4          Min.   :19.00         
##  1st Qu.:12081   1st Qu.:19          1st Qu.:4          1st Qu.:19.00         
##  Median :12420   Median :19          Median :4          Median :19.00         
##  Mean   :11828   Mean   :19          Mean   :4          Mean   :19.07         
##  3rd Qu.:12423   3rd Qu.:19          3rd Qu.:4          3rd Qu.:19.00         
##  Max.   :12423   Max.   :23          Max.   :4          Max.   :51.00         
##  NA's   :3014                                                                 
##  police_precincts
##  Min.   :10      
##  1st Qu.:13      
##  Median :13      
##  Mean   :13      
##  3rd Qu.:13      
##  Max.   :18      
## 
```

## Prepare the data


```r
temps<-nyc_squirrels %>% 
  group_by(shift) %>% 
  summarise(n=n()) %>% 
  mutate(rel.freq =round(100 * n/sum(n), 0)) %>% 
  rename(type=shift) %>% 
  add_column(categorie="shift")

age<-nyc_squirrels %>% 
  group_by(age) %>% 
  summarise(n=n()) %>% 
  mutate(rel.freq =round(100 * n/sum(n), 0)) %>% 
  rename(type=age) %>% 
  add_column(categorie="age")

couleur<-nyc_squirrels %>% 
  group_by(primary_fur_color) %>% 
  summarise(n=n()) %>% 
  mutate(rel.freq =round(100 * n/sum(n), 0)) %>% 
  rename(type=primary_fur_color) %>% 
  add_column(categorie="couleur")

location<-nyc_squirrels %>% 
  group_by(location) %>% 
  summarise(n=n()) %>% 
  mutate(rel.freq =round(100 * n/sum(n), 0)) %>% 
  rename(type=location) %>% 
  add_column(categorie="emplacement")

activite<-nyc_squirrels %>% 
  select(running, chasing, climbing, eating, foraging) %>%
  gather(activite, valeur) %>% 
  filter(!valeur=="FALSE") %>% 
  group_by(activite) %>% 
  summarise(n=n()) %>% 
  mutate(rel.freq =round(100 * n/sum(n), 0)) %>% 
  rename(type=activite) %>% 
  add_column(categorie="activite")

vocalise<-nyc_squirrels %>% 
  select(kuks, quaas, moans) %>%
  gather(vocalise, valeur) %>% 
  filter(!valeur=="FALSE") %>% 
  group_by(vocalise) %>% 
  summarise(n=n()) %>% 
  mutate(rel.freq =round(100 * n/sum(n), 0)) %>% 
  rename(type=vocalise) %>% 
  add_column(categorie="vocalise")

tail<-nyc_squirrels %>% 
  select(tail_flags, tail_twitches) %>%
  gather(tail, valeur) %>% 
  filter(!valeur=="FALSE") %>% 
  group_by(tail) %>% 
  summarise(n=n()) %>% 
  mutate(rel.freq =round(100 * n/sum(n), 0)) %>% 
  rename(type=tail) %>% 
  add_column(categorie="tail")

human<-nyc_squirrels %>% 
  select(approaches, indifferent, runs_from) %>%
  gather(human, valeur) %>% 
  filter(!valeur=="FALSE") %>% 
  group_by(human) %>% 
  summarise(n=n()) %>% 
  mutate(rel.freq =round(100 * n/sum(n), 0)) %>% 
  rename(type=human) %>% 
  add_column(categorie="human")

data<-human %>% 
  bind_rows(tail, vocalise, activite, location, couleur, age, temps)
```

## Visualize the data


```r
#Graphique
gg<- ggplot(data=data,aes(x = reorder(categorie,desc(rel.freq)), y=rel.freq, fill=reorder(type, desc(rel.freq))))
gg <- gg + geom_chicklet(width = 0.8, radius = grid::unit(2, 'mm'))
gg <- gg + coord_flip()
gg <- gg + scale_fill_manual(values=c("#A86C30", "#A86C30", "#A86C30", "#A86C30", "#A86C30", "#A86C30","#A86C30", "#DED4BB", "#A86C30", "#DED4BB", "#DED4BB", "#DED4BB","#DED4BB", "#DED4BB", "#DED4BB", "#DED4BB", "#DED4BB", "#DED4BB","#DED4BB", "#DED4BB", "#DED4BB", "#DED4BB", "#DED4BB", "#DED4BB"), na.value="#DED4BB")
#retirer la légende
gg <- gg +  theme(legend.position = "none")
#étiquettes de données
gg<-gg + annotate(geom="text", x=1,y=1, label="55 % prefer the afternoon", color="black", size=6, hjust=0,vjust=0.5, fontface="bold", family="Tw Cen MT")
gg<-gg + annotate(geom="text", x=2,y=1, label="74 % twitches their tail", color="black", size=6, hjust=0,vjust=0.5, fontface="bold", family="Tw Cen MT")
gg<-gg + annotate(geom="text", x=3,y=1, label="70 % prefer the ground", color="black", size=6, hjust=0,vjust=0.5, fontface="bold", family="Tw Cen MT")
gg<-gg + annotate(geom="text", x=4,y=1, label="63 % are indifferent to human", color="black", size=6, hjust=0,vjust=0.5, fontface="bold", family="Tw Cen MT")
gg<-gg + annotate(geom="text", x=5,y=1, label="66 % kurks", color="black", size=6, hjust=0,vjust=0.5, fontface="bold", family="Tw Cen MT")
gg<-gg + annotate(geom="text", x=6,y=1, label="85 % are adults", color="black", size=6, hjust=0,vjust=0.5, fontface="bold", family="Tw Cen MT")
gg<-gg + annotate(geom="text", x=7,y=1, label="82 % are gray", color="black", size=6, hjust=0,vjust=0.5, fontface="bold", family="Tw Cen MT")
gg<-gg + annotate(geom="text", x=8,y=1, label="37 % do foraging", color="black", size=6, hjust=0,vjust=0.5, fontface="bold", family="Tw Cen MT")
#modifier le thème
gg <- gg +  theme(panel.border = element_rect(color="#394018",size=1, fill=NA),
                    panel.background = element_rect(fill="#394018"),
                    plot.background = element_rect(fill="#394018"),
                    panel.grid.major.x= element_blank(),
                    panel.grid.major.y= element_blank(),
                    panel.grid.minor = element_blank(),
                    axis.line.x = element_blank(),
                    axis.line.y =element_blank(),
                    axis.ticks.x = element_blank(), 
                    axis.ticks.y = element_blank())
#ajouter les titres
gg<-gg + labs(title="New York's squirrels...",
              subtitle = "Here is the list of the main features of Central Park's squirrels\n",
              x=" ", 
              y=" ", 
              caption="\nSOURCE: NYC Squirrel Census   |  DESIGN: Johanie Fournier, agr.")
gg<-gg + theme(  plot.title    =  element_text(size=40, hjust=0,vjust=0.5, family="Tw Cen MT", color="black"),
                 plot.subtitle = element_text(size=16, hjust=0,vjust=0.5, family="Tw Cen MT", color="black"),
                 plot.caption  = element_text(size=10, hjust=1,vjust=0.5, family="Tw Cen MT", color="black"),
                 axis.title.y  = element_blank(),
                 axis.title.x  = element_blank(),
                 axis.text.y   = element_blank(), 
                 axis.text.x   = element_blank())
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-5-1.png" width="100%" style="display: block; margin: auto;" />

