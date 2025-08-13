---
title: "TyT2019W42 - Show Progress"
author: Johanie Fournier, agr. 
date: "2019-10-17"
slug: TyT2019W42
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
big_epa_cars <- readr::read_csv("https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2019/2019-10-15/big_epa_cars.csv")
```

```
## Rows: 41804 Columns: 83
```

```
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: ","
## chr (22): drive, eng_dscr, fuelType, fuelType1, make, model, mpgData, trany,...
## dbl (59): barrels08, barrelsA08, charge120, charge240, city08, city08U, city...
## lgl  (2): phevBlended, tCharger
```

```
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

## Explore the data


```r
summary(big_epa_cars)
```

```
##    barrels08       barrelsA08        charge120   charge240       
##  Min.   : 0.06   Min.   : 0.0000   Min.   :0   Min.   : 0.00000  
##  1st Qu.:14.33   1st Qu.: 0.0000   1st Qu.:0   1st Qu.: 0.00000  
##  Median :16.48   Median : 0.0000   Median :0   Median : 0.00000  
##  Mean   :17.25   Mean   : 0.2203   Mean   :0   Mean   : 0.04659  
##  3rd Qu.:19.39   3rd Qu.: 0.0000   3rd Qu.:0   3rd Qu.: 0.00000  
##  Max.   :47.09   Max.   :18.3117   Max.   :0   Max.   :13.00000  
##                                                                  
##      city08          city08U           cityA08            cityA08U       
##  Min.   :  6.00   Min.   :  0.000   Min.   :  0.0000   Min.   :  0.0000  
##  1st Qu.: 15.00   1st Qu.:  0.000   1st Qu.:  0.0000   1st Qu.:  0.0000  
##  Median : 17.00   Median :  0.000   Median :  0.0000   Median :  0.0000  
##  Mean   : 18.42   Mean   :  6.223   Mean   :  0.6869   Mean   :  0.5437  
##  3rd Qu.: 21.00   3rd Qu.: 14.817   3rd Qu.:  0.0000   3rd Qu.:  0.0000  
##  Max.   :150.00   Max.   :150.000   Max.   :145.0000   Max.   :145.0835  
##                                                                          
##      cityCD             cityE              cityUF             co2       
##  Min.   :0.000000   Min.   :  0.0000   Min.   :0.00000   Min.   : -1.0  
##  1st Qu.:0.000000   1st Qu.:  0.0000   1st Qu.:0.00000   1st Qu.: -1.0  
##  Median :0.000000   Median :  0.0000   Median :0.00000   Median : -1.0  
##  Mean   :0.000454   Mean   :  0.3436   Mean   :0.00175   Mean   : 92.9  
##  3rd Qu.:0.000000   3rd Qu.:  0.0000   3rd Qu.:0.00000   3rd Qu.: -1.0  
##  Max.   :5.350000   Max.   :122.0000   Max.   :0.92700   Max.   :847.0  
##                                                                         
##       co2A         co2TailpipeAGpm co2TailpipeGpm       comb08      
##  Min.   : -1.000   Min.   :  0.0   Min.   :   0.0   Min.   :  7.00  
##  1st Qu.: -1.000   1st Qu.:  0.0   1st Qu.: 386.4   1st Qu.: 17.00  
##  Median : -1.000   Median :  0.0   Median : 444.4   Median : 20.00  
##  Mean   :  5.945   Mean   : 17.5   Mean   : 465.3   Mean   : 20.67  
##  3rd Qu.: -1.000   3rd Qu.:  0.0   3rd Qu.: 522.8   3rd Qu.: 23.00  
##  Max.   :713.000   Max.   :713.0   Max.   :1269.6   Max.   :136.00  
##                                                                     
##     comb08U           combA08            combA08U            combE         
##  Min.   :  0.000   Min.   :  0.0000   Min.   :  0.0000   Min.   :  0.0000  
##  1st Qu.:  0.000   1st Qu.:  0.0000   1st Qu.:  0.0000   1st Qu.:  0.0000  
##  Median :  0.000   Median :  0.0000   Median :  0.0000   Median :  0.0000  
##  Mean   :  6.945   Mean   :  0.7455   Mean   :  0.5804   Mean   :  0.3505  
##  3rd Qu.: 17.000   3rd Qu.:  0.0000   3rd Qu.:  0.0000   3rd Qu.:  0.0000  
##  Max.   :136.000   Max.   :133.0000   Max.   :133.2662   Max.   :121.0000  
##                                                                            
##    combinedCD         combinedUF         cylinders          displ      
##  Min.   :0.000000   Min.   :0.000000   Min.   : 2.000   Min.   :0.000  
##  1st Qu.:0.000000   1st Qu.:0.000000   1st Qu.: 4.000   1st Qu.:2.200  
##  Median :0.000000   Median :0.000000   Median : 6.000   Median :3.000  
##  Mean   :0.000351   Mean   :0.001729   Mean   : 5.715   Mean   :3.292  
##  3rd Qu.:0.000000   3rd Qu.:0.000000   3rd Qu.: 6.000   3rd Qu.:4.300  
##  Max.   :4.800000   Max.   :0.920000   Max.   :16.000   Max.   :8.400  
##                                        NA's   :212      NA's   :210    
##     drive               engId         eng_dscr            feScore       
##  Length:41804       Min.   :    0   Length:41804       Min.   :-1.0000  
##  Class :character   1st Qu.:    0   Class :character   1st Qu.:-1.0000  
##  Mode  :character   Median :  181   Mode  :character   Median :-1.0000  
##                     Mean   : 8043                      Mean   : 0.4265  
##                     3rd Qu.: 4147                      3rd Qu.:-1.0000  
##                     Max.   :69102                      Max.   :10.0000  
##                                                                         
##    fuelCost08    fuelCostA08        fuelType          fuelType1        
##  Min.   : 500   Min.   :   0.00   Length:41804       Length:41804      
##  1st Qu.:1800   1st Qu.:   0.00   Class :character   Class :character  
##  Median :2200   Median :   0.00   Mode  :character   Mode  :character  
##  Mean   :2241   Mean   :  98.26                                        
##  3rd Qu.:2600   3rd Qu.:   0.00                                        
##  Max.   :7050   Max.   :3950.00                                        
##                                                                        
##     ghgScore         ghgScoreA         highway08        highway08U     
##  Min.   :-1.0000   Min.   :-1.0000   Min.   :  9.00   Min.   :  0.000  
##  1st Qu.:-1.0000   1st Qu.:-1.0000   1st Qu.: 20.00   1st Qu.:  0.000  
##  Median :-1.0000   Median :-1.0000   Median : 24.00   Median :  0.000  
##  Mean   : 0.4248   Mean   :-0.9207   Mean   : 24.56   Mean   :  8.193  
##  3rd Qu.:-1.0000   3rd Qu.:-1.0000   3rd Qu.: 28.00   3rd Qu.: 20.919  
##  Max.   :10.0000   Max.   : 8.0000   Max.   :124.00   Max.   :124.460  
##                                                                        
##    highwayA08        highwayA08U         highwayCD           highwayE       
##  Min.   :  0.0000   Min.   :  0.0000   Min.   :0.000000   Min.   :  0.0000  
##  1st Qu.:  0.0000   1st Qu.:  0.0000   1st Qu.:0.000000   1st Qu.:  0.0000  
##  Median :  0.0000   Median :  0.0000   Median :0.000000   Median :  0.0000  
##  Mean   :  0.8483   Mean   :  0.6502   Mean   :0.000235   Mean   :  0.3593  
##  3rd Qu.:  0.0000   3rd Qu.:  0.0000   3rd Qu.:0.000000   3rd Qu.:  0.0000  
##  Max.   :121.0000   Max.   :121.2005   Max.   :4.060000   Max.   :120.0000  
##                                                                             
##    highwayUF           hlv              hpv               id       
##  Min.   :0.0000   Min.   : 0.000   Min.   :  0.00   Min.   :    1  
##  1st Qu.:0.0000   1st Qu.: 0.000   1st Qu.:  0.00   1st Qu.:10452  
##  Median :0.0000   Median : 0.000   Median :  0.00   Median :20904  
##  Mean   :0.0017   Mean   : 1.996   Mean   : 10.21   Mean   :21023  
##  3rd Qu.:0.0000   3rd Qu.: 0.000   3rd Qu.:  0.00   3rd Qu.:31630  
##  Max.   :0.9100   Max.   :49.000   Max.   :195.00   Max.   :42159  
##                                                                    
##       lv2              lv4             make              model          
##  Min.   : 0.000   Min.   : 0.000   Length:41804       Length:41804      
##  1st Qu.: 0.000   1st Qu.: 0.000   Class :character   Class :character  
##  Median : 0.000   Median : 0.000   Mode  :character   Mode  :character  
##  Mean   : 1.798   Mean   : 6.101                                        
##  3rd Qu.: 0.000   3rd Qu.:13.000                                        
##  Max.   :41.000   Max.   :55.000                                        
##                                                                         
##    mpgData          phevBlended          pv2              pv4        
##  Length:41804       Mode :logical   Min.   :  0.00   Min.   :  0.00  
##  Class :character   FALSE:41697     1st Qu.:  0.00   1st Qu.:  0.00  
##  Mode  :character   TRUE :107       Median :  0.00   Median :  0.00  
##                                     Mean   : 13.48   Mean   : 33.78  
##                                     3rd Qu.:  0.00   3rd Qu.: 91.00  
##                                     Max.   :194.00   Max.   :192.00  
##                                                                      
##      range            rangeCity          rangeCityA           rangeHwy      
##  Min.   :  0.0000   Min.   :  0.0000   Min.   :  0.00000   Min.   :  0.000  
##  1st Qu.:  0.0000   1st Qu.:  0.0000   1st Qu.:  0.00000   1st Qu.:  0.000  
##  Median :  0.0000   Median :  0.0000   Median :  0.00000   Median :  0.000  
##  Mean   :  0.8156   Mean   :  0.7877   Mean   :  0.09447   Mean   :  0.755  
##  3rd Qu.:  0.0000   3rd Qu.:  0.0000   3rd Qu.:  0.00000   3rd Qu.:  0.000  
##  Max.   :370.0000   Max.   :381.5000   Max.   :135.28000   Max.   :355.900  
##                                                                             
##    rangeHwyA            trany               UCity            UCityA        
##  Min.   :  0.00000   Length:41804       Min.   :  0.00   Min.   :  0.0000  
##  1st Qu.:  0.00000   Class :character   1st Qu.: 18.30   1st Qu.:  0.0000  
##  Median :  0.00000   Mode  :character   Median : 21.47   Median :  0.0000  
##  Mean   :  0.08744                      Mean   : 23.27   Mean   :  0.8923  
##  3rd Qu.:  0.00000                      3rd Qu.: 26.00   3rd Qu.:  0.0000  
##  Max.   :114.76000                      Max.   :224.80   Max.   :207.2622  
##                                                                            
##     UHighway        UHighwayA          VClass               year     
##  Min.   :  0.00   Min.   :  0.000   Length:41804       Min.   :1984  
##  1st Qu.: 28.00   1st Qu.:  0.000   Class :character   1st Qu.:1991  
##  Median : 33.33   Median :  0.000   Mode  :character   Median :2003  
##  Mean   : 34.43   Mean   :  1.174                      Mean   :2002  
##  3rd Qu.: 39.00   3rd Qu.:  0.000                      3rd Qu.:2012  
##  Max.   :182.70   Max.   :173.144                      Max.   :2020  
##                                                                      
##   youSaveSpend      guzzler           trans_dscr        tCharger      
##  Min.   :-28000   Length:41804       Length:41804       Mode:logical  
##  1st Qu.: -5750   Class :character   Class :character   TRUE:7213     
##  Median : -3750   Mode  :character   Mode  :character   NA's:34591    
##  Mean   : -3952                                                       
##  3rd Qu.: -1750                                                       
##  Max.   :  4750                                                       
##                                                                       
##    sCharger           atvType           fuelType2            rangeA         
##  Length:41804       Length:41804       Length:41804       Length:41804      
##  Class :character   Class :character   Class :character   Class :character  
##  Mode  :character   Mode  :character   Mode  :character   Mode  :character  
##                                                                             
##                                                                             
##                                                                             
##                                                                             
##    evMotor            mfrCode            c240Dscr           charge240b     
##  Length:41804       Length:41804       Length:41804       Min.   :0.00000  
##  Class :character   Class :character   Class :character   1st Qu.:0.00000  
##  Mode  :character   Mode  :character   Mode  :character   Median :0.00000  
##                                                           Mean   :0.01005  
##                                                           3rd Qu.:0.00000  
##                                                           Max.   :8.50000  
##                                                                            
##   c240bDscr          createdOn          modifiedOn         startStop        
##  Length:41804       Length:41804       Length:41804       Length:41804      
##  Class :character   Class :character   Class :character   Class :character  
##  Mode  :character   Mode  :character   Mode  :character   Mode  :character  
##                                                                             
##                                                                             
##                                                                             
##                                                                             
##     phevCity          phevHwy           phevComb      
##  Min.   : 0.0000   Min.   : 0.0000   Min.   : 0.0000  
##  1st Qu.: 0.0000   1st Qu.: 0.0000   1st Qu.: 0.0000  
##  Median : 0.0000   Median : 0.0000   Median : 0.0000  
##  Mean   : 0.1666   Mean   : 0.1671   Mean   : 0.1661  
##  3rd Qu.: 0.0000   3rd Qu.: 0.0000   3rd Qu.: 0.0000  
##  Max.   :97.0000   Max.   :81.0000   Max.   :88.0000  
## 
```

## Prepare the data


```r
top_20<-big_epa_cars %>% 
  group_by(make) %>% 
  summarise(count=n()) %>% 
  top_n(20) %>% 
  arrange(desc(count))
```

```
## Selecting by count
```

```r
#type de carburant
type<-big_epa_cars %>% 
  select(fuelType) %>% 
  unique()

data<-big_epa_cars %>% 
  left_join(top_20) %>% 
  filter(cityA08==0, highwayA08==0, fuelType==c("Regular","Premium","Diesel","Midgrade")) %>% 
  mutate(categorie=NA) %>% 
  mutate(categorie=ifelse(VClass=="Compact Cars", "Cars", categorie)) %>% 
  mutate(categorie=ifelse(VClass=="Large Cars", "Cars", categorie)) %>% 
  mutate(categorie=ifelse(VClass=="Midsize Cars", "Cars", categorie)) %>% 
  mutate(categorie=ifelse(VClass=="Standard Pickup Trucks 2WD", "Pickup Trucks", categorie)) %>% 
  mutate(categorie=ifelse(VClass=="Standard Pickup Trucks 4WD", "Pickup Trucks", categorie)) %>% 
  filter(!is.na(categorie)) %>% 
  mutate(mpg = highway08 * .45 + city08 * .55) %>% 
  mutate(l100=235.22/mpg) %>% 
  select(make, model,year, mpg, l100, VClass, categorie, fuelType) %>% 
  filter(year<=2019, year>=2000)
```

```
## Joining, by = "make"
```

```r
mean_all<-data %>% 
  group_by(year, categorie) %>% 
  summarise(moy_l100=mean(l100), ecart=sd(l100))
```

```
## `summarise()` has grouped output by 'year'. You can override using the `.groups` argument.
```

```r
model_ford<-data %>% 
  filter(make=="Ford") %>% 
  group_by(year,categorie) %>% 
  summarise(moy_l100=mean(l100), 
            count=n())  %>% 
  filter(year<=2019, year>=2000)
```

```
## `summarise()` has grouped output by 'year'. You can override using the `.groups` argument.
```

## Visualize the data


```r
#Graphique
gg<-ggplot()
gg <- gg + geom_jitter(data=data, aes(x=year, y=l100),size=1, color="#8597A0", size=2.5, alpha = 0.25, width = 0.20)
gg <- gg + geom_point(data=mean_all, aes(x=year, y=moy_l100),fill="#6D7C83",color="#6D7C83", shape=21, size=3, stroke=1)
gg <- gg + geom_errorbar(data=mean_all, aes(x=year, ymax = moy_l100 + ecart, ymin = moy_l100 - ecart), color = "#6D7C83", width=.2)
gg <- gg + geom_line(data=model_ford, aes(x=year, y=moy_l100),color="#003379", size=1.5)
gg<-gg + facet_wrap(categorie~., dir = "v")
gg<-gg + annotate("rect", 
                   xmin=2014-0.5,
                   xmax=2016+0.5,
                   ymin=-Inf, 
                   ymax=Inf,
                   fill="#003379", alpha=0.2)
#ajuster les axes
gg<-gg + scale_y_continuous(breaks=seq(5,20,5), limits=c(5, 20))
gg<-gg + scale_y_reverse()
gg<-gg + scale_x_continuous(breaks=seq(2000,2020,5), limits=c(1999, 2020), expand=c(0,0.2))

#modifier le thème
gg <- gg +  theme(panel.border = element_blank(),
                    panel.background = element_blank(),
                    plot.background = element_blank(),
                    panel.grid.major.x= element_blank(),
                    panel.grid.major.y= element_line(size=0.3, linetype="dashed",color="#A9A9A9"),
                    panel.grid.minor = element_blank(),
                    axis.line.x = element_line(size=0.5, color="#A9A9A9"),
                    axis.line.y =element_blank(),
                    axis.ticks.x = element_line(size=0.5, color="#A9A9A9"), 
                    axis.ticks.y = element_blank(), 
                    strip.background =element_blank())
#ajouter les titres
gg<-gg + labs(title="<span style='color:#003379'>**FORD**</span> is losing its lead!",
              subtitle = "\nIn 2015, Ford was able to design vehicles that had better gas mileage (liters per 100 km) than the average of other vehicle brands. This advantage is not\nas marked in the last 3 years. Only the results of gasoline vehicles are shown.\n",
              x=" ", 
              y="Gas Mileage (Litres per 100 km)", 
              caption="\nSOURCE: www.fueleconomy.gov   |  DESIGN: Johanie Fournier, agr.")
gg<-gg + theme(  plot.title    =  element_markdown(lineheight = 1.1,size=31, hjust=0,vjust=0.5, face="bold", color="#404040"),
                 plot.subtitle = element_text(size=14, hjust=0,family="Tw Cen MT", color="#8B8B8B"),
                 plot.caption  = element_text(size=10, hjust=1,vjust=0.5, family="Tw Cen MT", color="#8B8B8B"),
                 axis.title.y  = element_text(size=12, hjust=1,vjust=0.5, family="Tw Cen MT", color="#8B8B8B", angle=90),
                 axis.title.x  = element_blank(),
                 axis.text.y   = element_text(size=12, hjust=0.5,vjust=0.5, family="Tw Cen MT", color="#8B8B8B"), 
                 axis.text.x   = element_text(size=12, hjust=0.5,vjust=0.5, family="Tw Cen MT", color="#8B8B8B"),
                 strip.text = element_text(size=12, hjust=0,vjust=1, family="Tw Cen MT", color="#8B8B8B"))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-5-1.png" width="100%" style="display: block; margin: auto;" />

