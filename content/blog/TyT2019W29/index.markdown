---
title: "TyT2019W29 - R4DS"
author: Johanie Fournier, agr. 
date: "2019-07-24"
slug: TyT2019W29
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
r4ds_members <- readr::read_csv("https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2019/2019-07-16/r4ds_members.csv")
```

```
## Rows: 678 Columns: 21
```

```
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: ","
## dbl  (20): total_membership, full_members, guests, daily_active_members, dai...
## date  (1): date
```

```
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

## Explore the data


```r
summary(r4ds_members)
```

```
##       date            total_membership  full_members        guests 
##  Min.   :2017-08-27   Min.   :   1.0   Min.   :   1.0   Min.   :0  
##  1st Qu.:2018-02-12   1st Qu.: 978.2   1st Qu.: 978.2   1st Qu.:0  
##  Median :2018-07-31   Median :1605.0   Median :1605.0   Median :0  
##  Mean   :2018-07-31   Mean   :1567.8   Mean   :1567.8   Mean   :0  
##  3rd Qu.:2019-01-16   3rd Qu.:2142.8   3rd Qu.:2142.8   3rd Qu.:0  
##  Max.   :2019-07-05   Max.   :3029.0   Max.   :3029.0   Max.   :0  
##  daily_active_members daily_members_posting_messages weekly_active_members
##  Min.   :  1.00       Min.   :  0.00                 Min.   :  1.0        
##  1st Qu.: 63.00       1st Qu.:  6.00                 1st Qu.:206.0        
##  Median : 88.00       Median : 11.00                 Median :239.0        
##  Mean   : 91.39       Mean   : 13.24                 Mean   :249.7        
##  3rd Qu.:110.00       3rd Qu.: 16.00                 3rd Qu.:307.8        
##  Max.   :258.00       Max.   :111.00                 Max.   :525.0        
##  weekly_members_posting_messages messages_in_public_channels
##  Min.   :  1.00                  Min.   :  0.00             
##  1st Qu.: 35.00                  1st Qu.:  9.25             
##  Median : 48.00                  Median : 19.00             
##  Mean   : 52.16                  Mean   : 28.46             
##  3rd Qu.: 59.00                  3rd Qu.: 35.00             
##  Max.   :278.00                  Max.   :326.00             
##  messages_in_private_channels messages_in_shared_channels messages_in_d_ms
##  Min.   : 0.000               Min.   :0                   Min.   :  0.00  
##  1st Qu.: 0.000               1st Qu.:0                   1st Qu.:  1.00  
##  Median : 0.000               Median :0                   Median :  4.00  
##  Mean   : 1.718               Mean   :0                   Mean   : 13.05  
##  3rd Qu.: 0.000               3rd Qu.:0                   3rd Qu.: 12.00  
##  Max.   :75.000               Max.   :0                   Max.   :227.00  
##  percent_of_messages_public_channels percent_of_messages_private_channels
##  Min.   :0.0000                      Min.   :0.0000                      
##  1st Qu.:0.5840                      1st Qu.:0.0000                      
##  Median :0.8000                      Median :0.0000                      
##  Mean   :0.7248                      Mean   :0.0305                      
##  3rd Qu.:0.9444                      3rd Qu.:0.0000                      
##  Max.   :1.0000                      Max.   :1.0000                      
##  percent_of_messages_d_ms percent_of_views_public_channels
##  Min.   :0.0000           Min.   :0.2726                  
##  1st Qu.:0.0345           1st Qu.:0.9115                  
##  Median :0.1595           Median :0.9519                  
##  Mean   :0.2270           Mean   :0.9285                  
##  3rd Qu.:0.3478           3rd Qu.:0.9744                  
##  Max.   :1.0000           Max.   :1.0000                  
##  percent_of_views_private_channels percent_of_views_d_ms      name  
##  Min.   :0.000000                  Min.   :0.00000       Min.   :0  
##  1st Qu.:0.000000                  1st Qu.:0.02235       1st Qu.:0  
##  Median :0.000000                  Median :0.04170       Median :0  
##  Mean   :0.009773                  Mean   :0.06176       Mean   :0  
##  3rd Qu.:0.006450                  3rd Qu.:0.07433       3rd Qu.:0  
##  Max.   :0.267400                  Max.   :0.72170       Max.   :0  
##  public_channels_single_workspace messages_posted
##  Min.   :10.0                     Min.   :   35  
##  1st Qu.:15.0                     1st Qu.:20543  
##  Median :19.0                     Median :33828  
##  Mean   :17.8                     Mean   :32936  
##  3rd Qu.:21.0                     3rd Qu.:40104  
##  Max.   :27.0                     Max.   :59627
```

## Prepare the data


```r
r4ds<-r4ds_members %>% 
  select('date','total_membership','messages_posted') %>%
  mutate(quarter=quarter(date, with_year = TRUE)) %>% 
  group_by(quarter) %>% 
  summarise(tot_m_1000=sum(total_membership/1000)) %>% 
  filter(!quarter %in% 2019.3)

r4ds_point<-r4ds %>% 
  filter(quarter %in% c(2017.3, 2019.2))

 r4ds_active<-r4ds_members %>% 
  select('date','total_membership','daily_active_members') %>%
  mutate(quarter=quarter(date, with_year = TRUE)) %>% 
  group_by(quarter) %>% 
  summarise(active_1000=sum(daily_active_members/1000)) %>% 
  filter(!quarter %in% 2019.3) 

r4ds_point_active<-r4ds_active %>% 
  filter(quarter %in% c(2017.3, 2019.2))

r4ds<-r4ds_members %>% 
  mutate(daily_message=messages_posted-shift(messages_posted)) %>% 
  filter(daily_message>0 & daily_message<5000) %>% 
  mutate(activity=(daily_message/daily_active_members)) %>% 
    mutate(quarter=quarter(date, with_year = TRUE)) %>% 
  group_by(quarter) %>% 
  summarise(active=sum(activity)) %>% 
  filter(!quarter %in% 2019.3) 

r4ds_point_activity<-r4ds %>% 
  filter(quarter %in% c(2017.3, 2019.2))
```

## Visualize the data


```r
#Graphique 
gg1<-ggplot(data=r4ds, aes(x = quarter, y=tot_m_1000))
gg1<-gg1 + geom_step(linetype=5, color="#A9A9A9", size=2.5)
gg1<-gg1 + geom_step(data=r4ds_active, aes(x = quarter, y=active_1000),linetype=5, color="#A9A9A9", size=2.5)
gg1<-gg1 +  geom_rect(data=r4ds,
            mapping=aes(xmin=2018.1,xmax=2018.4,ymin=0,ymax=Inf),
            fill='#01A7C2',alpha=0.05)
gg1<-gg1 + geom_point(data=r4ds_point,
                    mapping=(aes(x=quarter,y=tot_m_1000)), 
                    color="#A9A9A9", size=5)
gg1<-gg1 + geom_point(data=r4ds_point_active,
                    mapping=(aes(x=quarter,y=active_1000)), 
                    color="#A9A9A9", size=5)
#ajuster les axes 
gg1<-gg1 + scale_x_yearqtr(breaks = seq(from = min(r4ds$quarter), to = max(r4ds$quarter), by = 0.25),
                  format = "%Y-%q")
gg1<-gg1 + scale_y_continuous(breaks=seq(0,300,50), limits = c(0, 300))
#modifier la légende
gg1<-gg1 + theme(legend.position="none")
#modifier le thème
gg1<-gg1 +theme(panel.border = element_blank(),
              panel.background = element_blank(),
              plot.background = element_blank(),
              panel.grid.major.y= element_blank(),
              panel.grid.major.x= element_blank(),
              panel.grid.minor = element_blank(),
              axis.line.x = element_line(color="#A9A9A9"),
              axis.line.y = element_line(color="#A9A9A9"),
              axis.ticks= element_blank())
#ajouter les titres
gg1<-gg1 + labs(title="",
              subtitle=" ",
              y="Members (x1000)", 
              x=" ")
gg1<-gg1 + theme(plot.title    = element_text(hjust=0,size=15, color="#A9A9A9", face="bold"),
               plot.subtitle = element_text(hjust=0,size=12, color="#A9A9A9"),
               axis.title.y  = element_text(hjust=1,size=12, color="#A9A9A9", angle=90),
               axis.title.x  = element_blank(),
               axis.text.y   = element_text(hjust=0.5, size=10, color="#A9A9A9"), 
               axis.text.x   = element_text(hjust=0.5, size=10, color="#A9A9A9"))
#ajouter les étiquettes
gg1<-gg1 + annotate(geom="text", x=2019.2,y=270, label="Total", color="#A9A9A9", size=5, hjust=1,vjust=0, fontface="bold")
gg1<-gg1 + annotate(geom="text", x=2019.2,y=18, label="Active", color="#A9A9A9", size=5, hjust=1
                    ,vjust=0, fontface="bold")


gg2<-ggplot(data=r4ds, aes(x = quarter, y=active))
gg2<-gg2 + geom_step(linetype=5, color="#A9A9A9", size=2.5)
gg2<-gg2 +  geom_rect(data=r4ds,
            mapping=aes(xmin=2018.1,xmax=2018.4,ymin=0,ymax=Inf),
            fill='#01A7C2',alpha=0.05)
gg2<-gg2 + geom_point(data=r4ds_point_activity,
                    mapping=(aes(x=quarter,y=active)), 
                    color="#A9A9A9", size=5)
#ajuster les axes 
gg2<-gg2 + scale_x_yearqtr(breaks = seq(from = min(r4ds$quarter), to = max(r4ds$quarter), by = 0.25),
                  format = "%Y-%q")
gg2<-gg2 + scale_y_continuous(breaks=seq(0,100,25), limits = c(0, 100))
#modifier la légende
gg2<-gg2 + theme(legend.position="none")
#modifier le thème
gg2<-gg2 +theme(panel.border = element_blank(),
              panel.background = element_blank(),
              plot.background = element_blank(),
              panel.grid.major.y= element_blank(),
              panel.grid.major.x= element_blank(),
              panel.grid.minor = element_blank(),
              axis.line.x = element_line(color="#A9A9A9"),
              axis.line.y = element_line(color="#A9A9A9"),
              axis.ticks= element_blank())
#ajouter les titres
gg2<-gg2 + labs(title="",
              subtitle=" ",
              y="Daily messages/acive member", 
              x=" ")
gg2<-gg2 + theme(plot.title    = element_text(hjust=0,size=15, color="#A9A9A9", face="bold"),
               plot.subtitle = element_text(hjust=0,size=12, color="#A9A9A9"),
               axis.title.y  = element_text(hjust=1,size=12, color="#A9A9A9", angle=90),
               axis.title.x  = element_blank(),
               axis.text.y   = element_text(hjust=0.5, size=10, color="#A9A9A9"), 
               axis.text.x   = element_text(hjust=0.5, size=10, color="#A9A9A9"))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-5-1.png" width="100%" style="display: block; margin: auto;" />

