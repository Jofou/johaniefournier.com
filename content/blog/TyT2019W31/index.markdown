---
title: "TyT2019W31 - Video Games"
author: Johanie Fournier, agr. 
date: "2019-08-01"
slug: TyT2019W31
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
video_games <- readr::read_csv("https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2019/2019-07-30/video_games.csv")
```

```
## Rows: 26688 Columns: 10
```

```
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: ","
## chr (5): game, release_date, owners, developer, publisher
## dbl (5): number, price, average_playtime, median_playtime, metascore
```

```
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

## Explore the data


```r
summary(video_games)
```

```
##      number         game           release_date           price        
##  Min.   :   1   Length:26688       Length:26688       Min.   :  0.490  
##  1st Qu.: 821   Class :character   Class :character   1st Qu.:  2.990  
##  Median :2356   Mode  :character   Mode  :character   Median :  5.990  
##  Mean   :2904                                         Mean   :  8.947  
##  3rd Qu.:4523                                         3rd Qu.:  9.990  
##  Max.   :8846                                         Max.   :595.990  
##                                                       NA's   :3095     
##     owners           developer          publisher         average_playtime  
##  Length:26688       Length:26688       Length:26688       Min.   :   0.000  
##  Class :character   Class :character   Class :character   1st Qu.:   0.000  
##  Mode  :character   Mode  :character   Mode  :character   Median :   0.000  
##                                                           Mean   :   9.057  
##                                                           3rd Qu.:   0.000  
##                                                           Max.   :5670.000  
##                                                           NA's   :9         
##  median_playtime     metascore    
##  Min.   :   0.00   Min.   :20.00  
##  1st Qu.:   0.00   1st Qu.:66.00  
##  Median :   0.00   Median :73.00  
##  Mean   :   5.16   Mean   :71.89  
##  3rd Qu.:   0.00   3rd Qu.:80.00  
##  Max.   :3293.00   Max.   :98.00  
##  NA's   :12        NA's   :23838
```

## Prepare the data


```r
#SCORE
#moyenne globale
moyenne_globale_score <- video_games %>% 
  filter(metascore>=45) %>% 
  summarise(score_moy_globale=mean(metascore, na.rm=TRUE))

#moyenne annuelle
moyenne_annee_score <- video_games %>% 
  mutate(release_date = ifelse(release_date == "8 Oct, 2014", "Oct 8, 2014", release_date),
         date = mdy(release_date),
         annee = year(date))  %>%
 filter(metascore>=45) %>% 
  group_by(annee)%>%
  summarise(score_moy_annee=mean(metascore, na.rm=TRUE))

#fichier de travail
games_score <- video_games %>% 
  mutate(release_date = ifelse(release_date == "8 Oct, 2014", "Oct 8, 2014", release_date),
         date = mdy(release_date),
         annee = year(date))%>%
  filter(metascore>=45) %>% 
  group_by(annee, metascore) %>% 
  summarise(moy_annee=mean(metascore)) %>% 
  right_join(moyenne_annee_score, by="annee") %>%
  mutate(score_moy_global=moyenne_globale_score$score_moy_globale)

#PRIX
#moyenne prix
med_globale_prix <- video_games %>% 
  summarise(prix_med_globale=mean(price, na.rm=TRUE))

#median annuel
med_annee_prix <- video_games %>% 
  mutate(release_date = ifelse(release_date == "8 Oct, 2014", "Oct 8, 2014", release_date),
         date = mdy(release_date),
         annee = year(date))  %>%
  group_by(annee)%>%
  summarise(prix_med_annee=mean(price, na.rm=TRUE))

#fichier de travail
games_prix <- video_games %>% 
 filter(price<=20) %>%
  mutate(release_date = ifelse(release_date == "8 Oct, 2014", "Oct 8, 2014", release_date),
         date = mdy(release_date),
         annee = year(date))  %>%
    group_by(annee, price) %>% 
  summarise(moy_annee=mean(price)) %>% 
  right_join(med_annee_prix, by="annee") %>%
  mutate(prix_med_globale=med_globale_prix$prix_med_globale)
```

## Visualize the data


```r
#Graphique score
set.seed(123)
gg1<-ggplot(games_score, aes(x=annee, y=metascore, fill=annee))
gg1 <- gg1 + geom_jitter(color="#8597A0", size=5, alpha = 0.25, width = 0.20)
gg1 <- gg1 + geom_hline(aes(yintercept = score_moy_global), color = "#6D7C83", size = 0.5) 
gg1 <- gg1 + geom_segment(aes(x = annee, xend = annee,y = score_moy_global, yend = score_moy_annee), size = 0.5, color='#6D7C83')
gg1 <- gg1 + geom_point(mapping=aes(x=annee, y=score_moy_annee, fill=annee), fill="#386FA4",color="#6D7C83", shape=21, size=7, stroke=1)
#retirer la légende
gg1 <- gg1 + theme(legend.position = "none")
#ajuster les axes 
gg1 <- gg1 + scale_y_continuous(breaks=seq(40,100,20), limits = c(40, 100))
gg1 <- gg1 + scale_x_continuous(breaks=seq(2004,2018,1), limits = c(2003, 2018))
#modifier le thème
gg1 <- gg1 +  theme(panel.border = element_blank(),
                    panel.background = element_blank(),
                    plot.background = element_blank(),
                    panel.grid.major.y= element_line(size=0.1,linetype="dotted", color="#6D7C83"),
                    panel.grid.major.x= element_blank(),
                    panel.grid.minor = element_blank(),
                    axis.line.x = element_blank(),
                    axis.line.y = element_blank(),
                    axis.ticks.y = element_blank(),
                    axis.ticks.x = element_blank())
#ajouter les étiquettes
gg1<-gg1 + annotate(geom="text", x=2003,y=74, label="Mean=72", color="#6D7C83", size=4, hjust=0.5,vjust=0, fontface="bold")
#ajouter les titres
gg1<-gg1 + labs(title="Evolution of Video Games",
              subtitle="\nThe score has not changed much in recent years. As for the price, it is clear that there are more and more video\n games that are available at lower prices.",
              y="Metascore", 
              x=" ")
gg1<-gg1 + theme(plot.title    = element_text(hjust=0,size=36, color="#6D7C83", face="bold", family="Arial Rounded MT Bold"),
                 plot.subtitle = element_text(hjust=0,size=12, color="#6D7C83", family="Arial Rounded MT Bold"),
                 axis.title.y  = element_text(hjust=1, vjust=0, size=12, color="#6D7C83", face="bold"),
                 axis.title.x  = element_blank(),
                 axis.text.y   = element_text(hjust=0.5, vjust=0, size=12, color="#6D7C83", face="bold"), 
                 axis.text.x   = element_blank())

#Graphique Prix
set.seed(123)
gg2<-ggplot(games_prix, aes(x=annee, y=price, fill=annee))
gg2 <- gg2 + geom_jitter(color="#8597A0", size=6, alpha = 0.25, width = 0.20)
gg2 <- gg2 + geom_hline(aes(yintercept = prix_med_globale), color = "#6D7C83", size = 0.5) 
gg2 <- gg2 + geom_segment(aes(x = annee, xend = annee,y = prix_med_globale, yend = prix_med_annee), size = 0.5, color='#6D7C83')
gg2 <- gg2 + geom_point(mapping=aes(x=annee, y=prix_med_annee, fill=annee), fill="#386FA4",color="#6D7C83", shape=21, size=8.5, stroke=1)
#retirer la légende
gg2 <- gg2 + theme(legend.position = "none")
#ajuster les axes 
gg2 <- gg2 + scale_y_continuous(breaks=seq(0,20,5), limits = c(0, 20))
gg2 <- gg2 + scale_x_continuous(breaks=seq(2004,2018,1), limits = c(2003, 2018))
#modifier le thème
gg2 <- gg2 +  theme(panel.border = element_blank(),
                    panel.background = element_blank(),
                    plot.background = element_blank(),
                    panel.grid.major.y= element_line(size=0.1,linetype="dotted", color="#6D7C83"),
                    panel.grid.major.x= element_blank(),
                    panel.grid.minor = element_blank(),
                    axis.line.x = element_blank(),
                    axis.line.y = element_blank(),
                    axis.ticks.y = element_blank(),
                    axis.ticks.x = element_blank())
#ajouter les étiquettes
gg2<-gg2 + annotate(geom="text", x=2003,y=9.5, label="Mean=9", color="#6D7C83", size=5, hjust=0.5,vjust=0, fontface="bold")
#ajouter les titres
gg2<-gg2 + labs(title=" ",
              subtitle=" ",
              y="Price ($US)", 
              x=" ")
gg2<-gg2 + theme(plot.title    = element_blank(),
                 plot.subtitle = element_blank(),
                 axis.title.y  = element_text(hjust=1, vjust=0, size=14, color="#6D7C83", face="bold"),
                 axis.title.x  = element_blank(),
                 axis.text.y   = element_text(hjust=0.5, vjust=0, size=14, color="#6D7C83", face="bold"), 
                 axis.text.x   = element_text(hjust=0.5, vjust=0, size=14, color="#6D7C83", face="bold"))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-5-1.png" width="100%" style="display: block; margin: auto;" />

