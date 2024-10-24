---
title: 'TyT2024W21 - EDA:Carbon Majors Emissions Data'
author: Johanie Fournier, agr.
date: "2024-10-24"
slug: TyT2024W21_EDA
categories:
  - rstats
  - tidymodels
  - tidytuesday
tags:
  - eda
  - tidytuesday
summary: "This week we're exploring historical emissions data from Carbon Majors. They have complied a database of emissions data going back to 1854. In tihs first part, I start with some exploratory data analysis."
editor_options: 
  chunk_output_type: inline
---

<script src="index_files/libs/htmlwidgets-1.5.4/htmlwidgets.js"></script>
<link href="index_files/libs/datatables-css-0.0.0/datatables-crosstalk.css" rel="stylesheet" />
<script src="index_files/libs/datatables-binding-0.19/datatables.js"></script>
<script src="index_files/libs/jquery-3.6.0/jquery-3.6.0.min.js"></script>
<link href="index_files/libs/dt-core-1.10.20/css/jquery.dataTables.min.css" rel="stylesheet" />
<link href="index_files/libs/dt-core-1.10.20/css/jquery.dataTables.extra.css" rel="stylesheet" />
<script src="index_files/libs/dt-core-1.10.20/js/jquery.dataTables.min.js"></script>
<link href="index_files/libs/crosstalk-1.1.1/css/crosstalk.css" rel="stylesheet" />
<script src="index_files/libs/crosstalk-1.1.1/js/crosstalk.min.js"></script>
<script src="index_files/libs/plotly-binding-4.10.0/plotly.js"></script>
<script src="index_files/libs/typedarray-0.1/typedarray.min.js"></script>
<link href="index_files/libs/plotly-htmlwidgets-css-2.5.1/plotly-htmlwidgets.css" rel="stylesheet" />
<script src="index_files/libs/plotly-main-2.5.1/plotly-latest.min.js"></script>


I have worked extensively with spatial data over the past two years, so I decided to select suitable [`#TidyTuesday` dataset](https://github.com/rfordatascience/tidytuesday) and document what I have learned so far."

My latest contribution to the [`#TidyTuesday`](https://github.com/rfordatascience/tidytuesday) project featuring a recent dataset on carbon major emissions. The dataset is a compilation of emissions data from 1854 to 2019.

## Goal

The overall goal of this blog series is to predict carbon emissions over time and space.

In this first part, the goal is to do some Exploratory Data Analysis (EDA) to look at the data set and summarize the main characteristics. To do so, I will look at the data structure, anomalies, outliers and relationships.

## Get the data

Let's start by reading in the data:

``` r
emissions <- readr::read_csv('https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2024/2024-05-21/emissions.csv')

library(skimr)
library(PerformanceAnalytics)

skim(emissions)
```

<table style="width: auto;" class="table table-condensed">
<caption>
Data summary
</caption>
<tbody>
<tr>
<td style="text-align:left;">
Name
</td>
<td style="text-align:left;">
emissions
</td>
</tr>
<tr>
<td style="text-align:left;">
Number of rows
</td>
<td style="text-align:left;">
12551
</td>
</tr>
<tr>
<td style="text-align:left;">
Number of columns
</td>
<td style="text-align:left;">
7
</td>
</tr>
<tr>
<td style="text-align:left;">
\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_
</td>
<td style="text-align:left;">
</td>
</tr>
<tr>
<td style="text-align:left;">
Column type frequency:
</td>
<td style="text-align:left;">
</td>
</tr>
<tr>
<td style="text-align:left;">
character
</td>
<td style="text-align:left;">
4
</td>
</tr>
<tr>
<td style="text-align:left;">
numeric
</td>
<td style="text-align:left;">
3
</td>
</tr>
<tr>
<td style="text-align:left;">
\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_
</td>
<td style="text-align:left;">
</td>
</tr>
<tr>
<td style="text-align:left;">
Group variables
</td>
<td style="text-align:left;">
None
</td>
</tr>
</tbody>
</table>

**Variable type: character**

<table>
<thead>
<tr>
<th style="text-align:left;">
skim_variable
</th>
<th style="text-align:right;">
n_missing
</th>
<th style="text-align:right;">
complete_rate
</th>
<th style="text-align:right;">
min
</th>
<th style="text-align:right;">
max
</th>
<th style="text-align:right;">
empty
</th>
<th style="text-align:right;">
n_unique
</th>
<th style="text-align:right;">
whitespace
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
parent_entity
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
2
</td>
<td style="text-align:right;">
39
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
122
</td>
<td style="text-align:right;">
0
</td>
</tr>
<tr>
<td style="text-align:left;">
parent_type
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
12
</td>
<td style="text-align:right;">
22
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:right;">
0
</td>
</tr>
<tr>
<td style="text-align:left;">
commodity
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
6
</td>
<td style="text-align:right;">
19
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
9
</td>
<td style="text-align:right;">
0
</td>
</tr>
<tr>
<td style="text-align:left;">
production_unit
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
6
</td>
<td style="text-align:right;">
18
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
4
</td>
<td style="text-align:right;">
0
</td>
</tr>
</tbody>
</table>

**Variable type: numeric**

<table>
<thead>
<tr>
<th style="text-align:left;">
skim_variable
</th>
<th style="text-align:right;">
n_missing
</th>
<th style="text-align:right;">
complete_rate
</th>
<th style="text-align:right;">
mean
</th>
<th style="text-align:right;">
sd
</th>
<th style="text-align:right;">
p0
</th>
<th style="text-align:right;">
p25
</th>
<th style="text-align:right;">
p50
</th>
<th style="text-align:right;">
p75
</th>
<th style="text-align:right;">
p100
</th>
<th style="text-align:left;">
hist
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
year
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
1987.15
</td>
<td style="text-align:right;">
29.20
</td>
<td style="text-align:right;">
1854
</td>
<td style="text-align:right;">
1973.00
</td>
<td style="text-align:right;">
1994.00
</td>
<td style="text-align:right;">
2009.00
</td>
<td style="text-align:right;">
2022.00
</td>
<td style="text-align:left;">
▁▁▁▅▇
</td>
</tr>
<tr>
<td style="text-align:left;">
production_value
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
412.71
</td>
<td style="text-align:right;">
1357.57
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
10.60
</td>
<td style="text-align:right;">
63.20
</td>
<td style="text-align:right;">
320.66
</td>
<td style="text-align:right;">
27192.00
</td>
<td style="text-align:left;">
▇▁▁▁▁
</td>
</tr>
<tr>
<td style="text-align:left;">
total_emissions_MtCO2e
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
113.22
</td>
<td style="text-align:right;">
329.81
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
8.79
</td>
<td style="text-align:right;">
33.06
</td>
<td style="text-align:right;">
102.15
</td>
<td style="text-align:right;">
8646.91
</td>
<td style="text-align:left;">
▇▁▁▁▁
</td>
</tr>
</tbody>
</table>

``` r
chart.Correlation(select_if(emissions, is.numeric))
```

<img src="index.markdown_strict_files/figure-markdown_strict/unnamed-chunk-2-1.png" width="1260" />

So, we have a temporal dataset because their's a *year* column, 3 classifications columns (*parent_entity*, *parent_type*, *commodity*) and our variable of interest *total_emission_MtCO2e*.

## Trend over time

Is there a general trend over time?

``` r
sum_emissions_year<-emissions  |> 
  group_by(year) |>  
  summarise(sum=sum(total_emissions_MtCO2e)) |>  
  ungroup()

ggplot(data=sum_emissions_year, aes(x=year, y=sum))+
  geom_line()
```

<img src="index.markdown_strict_files/figure-markdown_strict/unnamed-chunk-4-1.png" width="1260" />

We can see a clear augmentation of carbon emissions over time.

The ultimate goal for this blog series will be to predict over time and space the carbon emission and visualize the result. To achieve that, we first need to understand more the relationship between *parent_entity* and *total_emission_MtCO2e*.

## Space trend

``` r
sum_emissions_entity<-emissions  |> 
  group_by(parent_entity) |>  
  summarise(sum=sum(total_emissions_MtCO2e)) |>  
  ungroup() |> 
  arrange(desc(sum))

DT::datatable(sum_emissions_entity) |> 
  DT::formatRound(columns=c("sum"), digits=0)
```

<div id="htmlwidget-f60512fad21e01a42060" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-f60512fad21e01a42060">{"x":{"filter":"none","vertical":false,"data":[["1","2","3","4","5","6","7","8","9","10","11","12","13","14","15","16","17","18","19","20","21","22","23","24","25","26","27","28","29","30","31","32","33","34","35","36","37","38","39","40","41","42","43","44","45","46","47","48","49","50","51","52","53","54","55","56","57","58","59","60","61","62","63","64","65","66","67","68","69","70","71","72","73","74","75","76","77","78","79","80","81","82","83","84","85","86","87","88","89","90","91","92","93","94","95","96","97","98","99","100","101","102","103","104","105","106","107","108","109","110","111","112","113","114","115","116","117","118","119","120","121","122"],["China (Coal)","Former Soviet Union","Saudi Aramco","Chevron","ExxonMobil","Gazprom","National Iranian Oil Co.","BP","Shell","Coal India","Poland","Pemex","Russian Federation","China (Cement)","ConocoPhillips","British Coal Corporation","CNPC","Peabody Coal Group","TotalEnergies","Abu Dhabi National Oil Company","Petroleos de Venezuela","Kuwait Petroleum Corp.","Iraq National Oil Company","Sonatrach","Rosneft","Occidental Petroleum","BHP","Petrobras","CONSOL Energy","Nigerian National Petroleum Corp.","Czechoslovakia","Petronas","Eni","QatarEnergy","Pertamina","Anglo American","Libya National Oil Corp.","Arch Resources","Lukoil","Kazakhstan","Equinor","RWE","Rio Tinto","Glencore","Alpha Metallurgical Resources","ONGC India","Sasol","Ukraine","Surgutneftegas","Repsol","Petroleum Development Oman","Sinopec","Egyptian General Petroleum","TurkmenGaz","Petoro","CNOOC","North Korea","Marathon Oil","Bumi Resources","Devon Energy","Singareni Collieries","Sonangol","Holcim Group","Novatek","Ecopetrol","Suncor Energy","Hess Corporation","Ovintiv","Czech Republic","Canadian Natural Resources","Cyprus AMAX Minerals","Westmoreland Mining","BASF","American Consolidated Natural Resources","Exxaro Resources Ltd","Bapco Energies","Adaro Energy","YPF","Cenovus Energy","APA Corporation","Banpu","PetroEcuador","EOG Resources","Alliance Resource Partners","Kiewit Mining Group","Heidelberg Materials","North American Coal","Chesapeake Energy","Syrian Petroleum","Cloud Peak","Vistra","Teck Resources","Inpex","Naftogaz","Coterra Energy","PTTEP","OMV Group","EQT Corporation","Southwestern Energy","Woodside Energy","UK Coal","Cemex","Santos","Pioneer Natural Resources","Murphy Oil","Orlen","Antero","Taiheiyo Cement","Continental Resources","Tourmaline Oil","Whitehaven Coal","Navajo Transitional Energy Company","Wolverine Fuels","Seriti Resources","Obsidian Energy","Vale","SM Energy","Adani Enterprises","CNX Resources","CRH","Tullow Oil","Slovakia"],[276458.02492082,135112.668813454,68831.5371038049,57897.8526780308,55105.0951438195,50686.9725960528,43111.6933371727,42530.4193021717,40674.0595286655,29391.301962912,28749.881872077,25496.9702223447,23412.4597853956,23161.2674748,20222.2587592593,19745.3614159457,18950.9099374694,17735.4272607568,17583.5566389311,17383.1781780909,16900.9964886558,15921.8100817819,15188.3319625943,14954.6969185315,14294.5995626711,12907.1666995437,11042.1107132738,10799.3014774493,10490.1353121505,10243.4346425407,9618.47096479113,9130.3042923771,9074.62598421214,8404.87718545639,8269.72842642096,8162.76285199174,8146.45093179807,7969.29320263312,7835.30435399672,7768.91312157044,7738.84445852112,7584.75041806049,6767.00067503288,6329.25433231514,6127.22336061584,5917.36393716968,4991.74738532649,4969.16296710561,4734.52335389813,4584.29006915196,4386.74015392261,4374.18031699524,4318.01143317843,4222.7028687985,4173.51889785532,4147.45195702434,4103.6786217367,3804.40360061962,3762.01223959853,3296.65217966089,3290.96068860636,3281.23149402648,3172.61488629005,3096.41678676808,3095.75610781481,3072.27555015738,3026.03861025751,2992.80130871679,2737.35005584155,2640.05036633372,2568.88336291029,2339.07321609418,2313.43445405336,2239.72239693733,2159.90141611748,2127.18114419001,2068.44584656087,2038.97607788709,1965.29890484058,1963.91632421968,1942.73452903939,1922.40210164233,1806.07862580342,1777.07079047006,1688.6139943611,1683.69769631625,1643.72803770716,1608.84172667945,1578.27098510082,1476.15740538291,1393.53784597781,1307.51870577066,1256.04352675101,1252.49025479955,1183.72510380333,1079.56774370571,1014.16171050331,1000.97785430615,981.974828995937,918.253426767465,881.959476578908,867.4431883275,836.503784597426,825.675499865858,765.018468149223,720.309583008861,606.199782562199,580.217273699,455.424224997692,449.588109759621,428.392375723516,389.715756839308,384.717350233232,361.397036552038,355.74103477328,317.151646348455,316.173342181209,316.050246230678,227.074903356675,216.74008502,210.986423185838,104.499507804202]],"container":"<table class=\"display\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>parent_entity<\/th>\n      <th>sum<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"columnDefs":[{"targets":2,"render":"function(data, type, row, meta) {\n    return type !== 'display' ? data : DTWidget.formatRound(data, 0, 3, \",\", \".\");\n  }"},{"className":"dt-right","targets":2},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false}},"evals":["options.columnDefs.0.render"],"jsHooks":[]}</script>

We have a clear indication that country does not produce the same amount of carbon.

## Spatio-temporal trend

Can we link the spatial trend to the temporal trend? Let's find out by looking at the top 10 countries with the highest emissions.

``` r
top10_entity<-sum_emissions_entity |> 
  top_n(6, sum) |> 
  select(parent_entity)

emissions_top10<-emissions |> 
  filter(parent_entity %in% top10_entity$parent_entity) 

plot_data<-emissions_top10 |> 
  group_by(parent_entity, year) |>
  summarize(sum=sum(total_emissions_MtCO2e)) |>
  ungroup() |>
  mutate(date=as.Date(as.character(year), "%Y"),
         parent_entity_fct=as.factor(parent_entity)) |>
  select(parent_entity_fct, date, sum) |>
  pad_by_time(date, .by = "year")

plot_data |> 
    group_by(parent_entity_fct) |> 
    plot_time_series(
        .date_var    = date,
        .value       = sum,
        .interactive = TRUE,
        .facet_ncol  = 2,
        .facet_scales = "free",
        .plotly_slider = TRUE
    )
```

<div id="htmlwidget-f60512fad21e01a42060" style="width:1260px;height:900px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-f60512fad21e01a42060">{"x":{"data":[{"x":["1884-10-24","1885-10-24","1886-10-24","1887-10-24","1888-10-24","1889-10-24","1890-10-24","1891-10-24","1892-10-24","1893-10-24","1894-10-24","1895-10-24","1896-10-24","1897-10-24","1898-10-24","1899-10-24","1900-10-24","1901-10-24","1902-10-24","1903-10-24","1904-10-24","1905-10-24","1906-10-24","1907-10-24","1908-10-24","1909-10-24","1910-10-24","1911-10-24","1912-10-24","1913-10-24","1914-10-24","1915-10-24","1916-10-24","1917-10-24","1918-10-24","1919-10-24","1920-10-24","1921-10-24","1922-10-24","1923-10-24","1924-10-24","1925-10-24","1926-10-24","1927-10-24","1928-10-24","1929-10-24","1930-10-24","1931-10-24","1932-10-24","1933-10-24","1934-10-24","1935-10-24","1936-10-24","1937-10-24","1938-10-24","1939-10-24","1940-10-24","1941-10-24","1942-10-24","1943-10-24","1944-10-24","1945-10-24","1946-10-24","1947-10-24","1948-10-24","1949-10-24","1950-10-24","1951-10-24","1952-10-24","1953-10-24","1954-10-24","1955-10-24","1956-10-24","1957-10-24","1958-10-24","1959-10-24","1960-10-24","1961-10-24","1962-10-24","1963-10-24","1964-10-24","1965-10-24","1966-10-24","1967-10-24","1968-10-24","1969-10-24","1970-10-24","1971-10-24","1972-10-24","1973-10-24","1974-10-24","1975-10-24","1976-10-24","1977-10-24","1978-10-24","1979-10-24","1980-10-24","1981-10-24","1982-10-24","1983-10-24","1984-10-24","1985-10-24","1986-10-24","1987-10-24","1988-10-24","1989-10-24","1990-10-24","1991-10-24","1992-10-24","1993-10-24","1994-10-24","1995-10-24","1996-10-24","1997-10-24","1998-10-24","1999-10-24","2000-10-24","2001-10-24","2002-10-24","2003-10-24","2004-10-24","2005-10-24","2006-10-24","2007-10-24","2008-10-24","2009-10-24","2010-10-24","2011-10-24","2012-10-24","2013-10-24","2014-10-24","2015-10-24","2016-10-24","2017-10-24","2018-10-24","2019-10-24","2020-10-24","2021-10-24","2022-10-24"],"y":[0.0348306472662587,0.072901899705787,0.119392244019391,0.169804872019963,0.227202571025111,0.337990525929783,0.432461361421588,0.597168836232185,0.721457201096719,0.851922320150531,0.969225076457885,1.01456405885941,1.16877223763333,1.39547607628416,1.51415195529082,1.52113574063546,2.2476007543866,2.76227679148905,3.13401830289136,3.43474135700313,3.78993087815129,4.42920422448903,5.15323473212225,5.72571704840302,6.00841760105247,6.91031581536233,7.97094570132958,8.49209222625067,8.93148006301763,8.81479754230173,9.02517576257504,10.4111272471434,11.692318882514,13.1508134608869,14.5265885264622,16.9764765904784,19.4847414490789,19.1000848590584,21.3410705494571,28.3327540597607,36.0209313844411,36.5945014780643,41.2831748935054,42.9580340750076,46.6100055388618,54.6142123580948,55.3139404037285,52.9440551358716,63.5453958134918,79.1553839741943,95.8501622788674,103.07798652214,112.464065625208,131.679066561786,118.899051153409,134.052619573414,125.977233135419,139.079615049972,108.912161554802,141.952442263903,180.343601480124,194.480322758231,212.076170080446,237.468191301191,246.817588520836,226.568144179117,274.351524875869,323.246538455711,347.634878220674,364.681916737829,377.366126841569,419.479653723412,460.050482701059,474.166466223162,468.296205333669,493.484114232342,512.079682037819,549.134537816439,605.841367341941,664.935609203305,726.381550827145,795.473401869029,857.161756372444,927.308376215806,1027.04995666306,1088.30319248966,1205.94266905118,1281.53642460109,1373.14480848295,1397.34605826699,1240.97328272489,1133.22135406215,995.901771171342,1043.22688475864,1005.18703646362,1042.4432842548,853.248862535727,576.210179590584,551.467173119028,583.187955669586,642.274469672109,669.354205749303,668.830554078507,694.298718927466,708.460673482002,719.659275156863,708.055519662238,729.940252021928,711.662610924076,711.29632432986,737.571269265222,688.286368573562,693.252760368798,695.126251392596,688.39135350672,687.159959476027,694.133946617405,684.270419124787,664.134232576572,656.037967108446,663.251061421962,648.045894179428,678.404613815938,678.034498413834,653.152704338434,666.475691104697,675.729072846675,686.695602663964,645.86205956938,635.406450073202,603.776992423384,621.075467331777,613.66380889592,603.963205810944,580.045165020283,597.218981986108,567.277078427857,560.29977199011,563.322369412809],"text":["date: 1884-10-24<br />.value: 3.483065e-02","date: 1885-10-24<br />.value: 7.290190e-02","date: 1886-10-24<br />.value: 1.193922e-01","date: 1887-10-24<br />.value: 1.698049e-01","date: 1888-10-24<br />.value: 2.272026e-01","date: 1889-10-24<br />.value: 3.379905e-01","date: 1890-10-24<br />.value: 4.324614e-01","date: 1891-10-24<br />.value: 5.971688e-01","date: 1892-10-24<br />.value: 7.214572e-01","date: 1893-10-24<br />.value: 8.519223e-01","date: 1894-10-24<br />.value: 9.692251e-01","date: 1895-10-24<br />.value: 1.014564e+00","date: 1896-10-24<br />.value: 1.168772e+00","date: 1897-10-24<br />.value: 1.395476e+00","date: 1898-10-24<br />.value: 1.514152e+00","date: 1899-10-24<br />.value: 1.521136e+00","date: 1900-10-24<br />.value: 2.247601e+00","date: 1901-10-24<br />.value: 2.762277e+00","date: 1902-10-24<br />.value: 3.134018e+00","date: 1903-10-24<br />.value: 3.434741e+00","date: 1904-10-24<br />.value: 3.789931e+00","date: 1905-10-24<br />.value: 4.429204e+00","date: 1906-10-24<br />.value: 5.153235e+00","date: 1907-10-24<br />.value: 5.725717e+00","date: 1908-10-24<br />.value: 6.008418e+00","date: 1909-10-24<br />.value: 6.910316e+00","date: 1910-10-24<br />.value: 7.970946e+00","date: 1911-10-24<br />.value: 8.492092e+00","date: 1912-10-24<br />.value: 8.931480e+00","date: 1913-10-24<br />.value: 8.814798e+00","date: 1914-10-24<br />.value: 9.025176e+00","date: 1915-10-24<br />.value: 1.041113e+01","date: 1916-10-24<br />.value: 1.169232e+01","date: 1917-10-24<br />.value: 1.315081e+01","date: 1918-10-24<br />.value: 1.452659e+01","date: 1919-10-24<br />.value: 1.697648e+01","date: 1920-10-24<br />.value: 1.948474e+01","date: 1921-10-24<br />.value: 1.910008e+01","date: 1922-10-24<br />.value: 2.134107e+01","date: 1923-10-24<br />.value: 2.833275e+01","date: 1924-10-24<br />.value: 3.602093e+01","date: 1925-10-24<br />.value: 3.659450e+01","date: 1926-10-24<br />.value: 4.128317e+01","date: 1927-10-24<br />.value: 4.295803e+01","date: 1928-10-24<br />.value: 4.661001e+01","date: 1929-10-24<br />.value: 5.461421e+01","date: 1930-10-24<br />.value: 5.531394e+01","date: 1931-10-24<br />.value: 5.294406e+01","date: 1932-10-24<br />.value: 6.354540e+01","date: 1933-10-24<br />.value: 7.915538e+01","date: 1934-10-24<br />.value: 9.585016e+01","date: 1935-10-24<br />.value: 1.030780e+02","date: 1936-10-24<br />.value: 1.124641e+02","date: 1937-10-24<br />.value: 1.316791e+02","date: 1938-10-24<br />.value: 1.188991e+02","date: 1939-10-24<br />.value: 1.340526e+02","date: 1940-10-24<br />.value: 1.259772e+02","date: 1941-10-24<br />.value: 1.390796e+02","date: 1942-10-24<br />.value: 1.089122e+02","date: 1943-10-24<br />.value: 1.419524e+02","date: 1944-10-24<br />.value: 1.803436e+02","date: 1945-10-24<br />.value: 1.944803e+02","date: 1946-10-24<br />.value: 2.120762e+02","date: 1947-10-24<br />.value: 2.374682e+02","date: 1948-10-24<br />.value: 2.468176e+02","date: 1949-10-24<br />.value: 2.265681e+02","date: 1950-10-24<br />.value: 2.743515e+02","date: 1951-10-24<br />.value: 3.232465e+02","date: 1952-10-24<br />.value: 3.476349e+02","date: 1953-10-24<br />.value: 3.646819e+02","date: 1954-10-24<br />.value: 3.773661e+02","date: 1955-10-24<br />.value: 4.194797e+02","date: 1956-10-24<br />.value: 4.600505e+02","date: 1957-10-24<br />.value: 4.741665e+02","date: 1958-10-24<br />.value: 4.682962e+02","date: 1959-10-24<br />.value: 4.934841e+02","date: 1960-10-24<br />.value: 5.120797e+02","date: 1961-10-24<br />.value: 5.491345e+02","date: 1962-10-24<br />.value: 6.058414e+02","date: 1963-10-24<br />.value: 6.649356e+02","date: 1964-10-24<br />.value: 7.263816e+02","date: 1965-10-24<br />.value: 7.954734e+02","date: 1966-10-24<br />.value: 8.571618e+02","date: 1967-10-24<br />.value: 9.273084e+02","date: 1968-10-24<br />.value: 1.027050e+03","date: 1969-10-24<br />.value: 1.088303e+03","date: 1970-10-24<br />.value: 1.205943e+03","date: 1971-10-24<br />.value: 1.281536e+03","date: 1972-10-24<br />.value: 1.373145e+03","date: 1973-10-24<br />.value: 1.397346e+03","date: 1974-10-24<br />.value: 1.240973e+03","date: 1975-10-24<br />.value: 1.133221e+03","date: 1976-10-24<br />.value: 9.959018e+02","date: 1977-10-24<br />.value: 1.043227e+03","date: 1978-10-24<br />.value: 1.005187e+03","date: 1979-10-24<br />.value: 1.042443e+03","date: 1980-10-24<br />.value: 8.532489e+02","date: 1981-10-24<br />.value: 5.762102e+02","date: 1982-10-24<br />.value: 5.514672e+02","date: 1983-10-24<br />.value: 5.831880e+02","date: 1984-10-24<br />.value: 6.422745e+02","date: 1985-10-24<br />.value: 6.693542e+02","date: 1986-10-24<br />.value: 6.688306e+02","date: 1987-10-24<br />.value: 6.942987e+02","date: 1988-10-24<br />.value: 7.084607e+02","date: 1989-10-24<br />.value: 7.196593e+02","date: 1990-10-24<br />.value: 7.080555e+02","date: 1991-10-24<br />.value: 7.299403e+02","date: 1992-10-24<br />.value: 7.116626e+02","date: 1993-10-24<br />.value: 7.112963e+02","date: 1994-10-24<br />.value: 7.375713e+02","date: 1995-10-24<br />.value: 6.882864e+02","date: 1996-10-24<br />.value: 6.932528e+02","date: 1997-10-24<br />.value: 6.951263e+02","date: 1998-10-24<br />.value: 6.883914e+02","date: 1999-10-24<br />.value: 6.871600e+02","date: 2000-10-24<br />.value: 6.941339e+02","date: 2001-10-24<br />.value: 6.842704e+02","date: 2002-10-24<br />.value: 6.641342e+02","date: 2003-10-24<br />.value: 6.560380e+02","date: 2004-10-24<br />.value: 6.632511e+02","date: 2005-10-24<br />.value: 6.480459e+02","date: 2006-10-24<br />.value: 6.784046e+02","date: 2007-10-24<br />.value: 6.780345e+02","date: 2008-10-24<br />.value: 6.531527e+02","date: 2009-10-24<br />.value: 6.664757e+02","date: 2010-10-24<br />.value: 6.757291e+02","date: 2011-10-24<br />.value: 6.866956e+02","date: 2012-10-24<br />.value: 6.458621e+02","date: 2013-10-24<br />.value: 6.354065e+02","date: 2014-10-24<br />.value: 6.037770e+02","date: 2015-10-24<br />.value: 6.210755e+02","date: 2016-10-24<br />.value: 6.136638e+02","date: 2017-10-24<br />.value: 6.039632e+02","date: 2018-10-24<br />.value: 5.800452e+02","date: 2019-10-24<br />.value: 5.972190e+02","date: 2020-10-24<br />.value: 5.672771e+02","date: 2021-10-24<br />.value: 5.602998e+02","date: 2022-10-24<br />.value: 5.633224e+02"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(44,62,80,1)","dash":"solid"},"hoveron":"points","showlegend":false,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":["1900-10-24","1901-10-24","1902-10-24","1903-10-24","1904-10-24","1905-10-24","1906-10-24","1907-10-24","1908-10-24","1909-10-24","1910-10-24","1911-10-24","1912-10-24","1913-10-24","1914-10-24","1915-10-24","1916-10-24","1917-10-24","1918-10-24","1919-10-24","1920-10-24","1921-10-24","1922-10-24","1923-10-24","1924-10-24","1925-10-24","1926-10-24","1927-10-24","1928-10-24","1929-10-24","1930-10-24","1931-10-24","1932-10-24","1933-10-24","1934-10-24","1935-10-24","1936-10-24","1937-10-24","1938-10-24","1939-10-24","1940-10-24","1941-10-24","1942-10-24","1943-10-24","1944-10-24","1945-10-24","1946-10-24","1947-10-24","1948-10-24","1949-10-24","1950-10-24","1951-10-24","1952-10-24","1953-10-24","1954-10-24","1955-10-24","1956-10-24","1957-10-24","1958-10-24","1959-10-24","1960-10-24","1961-10-24","1962-10-24","1963-10-24","1964-10-24","1965-10-24","1966-10-24","1967-10-24","1968-10-24","1969-10-24","1970-10-24","1971-10-24","1972-10-24","1973-10-24","1974-10-24","1975-10-24","1976-10-24","1977-10-24","1978-10-24","1979-10-24","1980-10-24","1981-10-24","1982-10-24","1983-10-24","1984-10-24","1985-10-24","1986-10-24","1987-10-24","1988-10-24","1989-10-24","1990-10-24","1991-10-24"],"y":[39.7528009979273,38.5105259687102,37.2682509351872,36.0259759016643,34.7837008751613,33.5414258416384,32.299150811082,31.0568757816124,29.8146007451718,28.5723257146155,27.3300506851459,26.0877756545896,24.8455006210666,23.6032255888829,22.3609505610407,21.1186755275178,19.8764004983006,18.6341254674918,17.3918504339689,16.1495754047518,14.9073003712288,23.3428447479092,31.7783891146519,40.2139334913323,48.6494778472184,57.085022237217,65.5205666017016,73.9561109616411,82.3916553532671,90.8271997132066,99.2627441061718,125.22132315233,175.568245592219,216.307412974509,276.657133996287,270.746389396912,314.22104638092,315.204928203567,330.166888688225,409.887404285549,489.607919936667,551.645892587933,613.683865480947,345.51643896479,315.289402902056,364.24994643774,400.012560038238,434.796260914761,509.332762785739,615.46460126646,717.155200223985,741.824747173158,796.308894946073,935.777077661274,1073.3350908059,1166.16668342196,1324.66488130379,1501.82046273599,1657.47618825286,1743.08739307935,1825.7276775497,1899.67741487805,2015.87273054667,2157.38001650869,2311.39935431015,2462.51917483345,2566.87889311353,2679.96618172574,2761.37599951609,2671.6054053708,2746.39889053739,2882.01135922187,3114.165642873,3212.19316042618,3361.11547531109,3577.91766919181,3773.05018627959,3946.59292467458,4139.35754869594,4273.32284480511,4692.15386258758,4761.94827983937,4899.98344952702,4996.6849650294,5104.22727726802,5244.30829648439,5474.76566526985,5603.60173647077,5768.69073258056,3657.28896350588,3473.05088677596,3139.28009559471],"text":["date: 1900-10-24<br />.value: 3.975280e+01","date: 1901-10-24<br />.value: 3.851053e+01","date: 1902-10-24<br />.value: 3.726825e+01","date: 1903-10-24<br />.value: 3.602598e+01","date: 1904-10-24<br />.value: 3.478370e+01","date: 1905-10-24<br />.value: 3.354143e+01","date: 1906-10-24<br />.value: 3.229915e+01","date: 1907-10-24<br />.value: 3.105688e+01","date: 1908-10-24<br />.value: 2.981460e+01","date: 1909-10-24<br />.value: 2.857233e+01","date: 1910-10-24<br />.value: 2.733005e+01","date: 1911-10-24<br />.value: 2.608778e+01","date: 1912-10-24<br />.value: 2.484550e+01","date: 1913-10-24<br />.value: 2.360323e+01","date: 1914-10-24<br />.value: 2.236095e+01","date: 1915-10-24<br />.value: 2.111868e+01","date: 1916-10-24<br />.value: 1.987640e+01","date: 1917-10-24<br />.value: 1.863413e+01","date: 1918-10-24<br />.value: 1.739185e+01","date: 1919-10-24<br />.value: 1.614958e+01","date: 1920-10-24<br />.value: 1.490730e+01","date: 1921-10-24<br />.value: 2.334284e+01","date: 1922-10-24<br />.value: 3.177839e+01","date: 1923-10-24<br />.value: 4.021393e+01","date: 1924-10-24<br />.value: 4.864948e+01","date: 1925-10-24<br />.value: 5.708502e+01","date: 1926-10-24<br />.value: 6.552057e+01","date: 1927-10-24<br />.value: 7.395611e+01","date: 1928-10-24<br />.value: 8.239166e+01","date: 1929-10-24<br />.value: 9.082720e+01","date: 1930-10-24<br />.value: 9.926274e+01","date: 1931-10-24<br />.value: 1.252213e+02","date: 1932-10-24<br />.value: 1.755682e+02","date: 1933-10-24<br />.value: 2.163074e+02","date: 1934-10-24<br />.value: 2.766571e+02","date: 1935-10-24<br />.value: 2.707464e+02","date: 1936-10-24<br />.value: 3.142210e+02","date: 1937-10-24<br />.value: 3.152049e+02","date: 1938-10-24<br />.value: 3.301669e+02","date: 1939-10-24<br />.value: 4.098874e+02","date: 1940-10-24<br />.value: 4.896079e+02","date: 1941-10-24<br />.value: 5.516459e+02","date: 1942-10-24<br />.value: 6.136839e+02","date: 1943-10-24<br />.value: 3.455164e+02","date: 1944-10-24<br />.value: 3.152894e+02","date: 1945-10-24<br />.value: 3.642499e+02","date: 1946-10-24<br />.value: 4.000126e+02","date: 1947-10-24<br />.value: 4.347963e+02","date: 1948-10-24<br />.value: 5.093328e+02","date: 1949-10-24<br />.value: 6.154646e+02","date: 1950-10-24<br />.value: 7.171552e+02","date: 1951-10-24<br />.value: 7.418247e+02","date: 1952-10-24<br />.value: 7.963089e+02","date: 1953-10-24<br />.value: 9.357771e+02","date: 1954-10-24<br />.value: 1.073335e+03","date: 1955-10-24<br />.value: 1.166167e+03","date: 1956-10-24<br />.value: 1.324665e+03","date: 1957-10-24<br />.value: 1.501820e+03","date: 1958-10-24<br />.value: 1.657476e+03","date: 1959-10-24<br />.value: 1.743087e+03","date: 1960-10-24<br />.value: 1.825728e+03","date: 1961-10-24<br />.value: 1.899677e+03","date: 1962-10-24<br />.value: 2.015873e+03","date: 1963-10-24<br />.value: 2.157380e+03","date: 1964-10-24<br />.value: 2.311399e+03","date: 1965-10-24<br />.value: 2.462519e+03","date: 1966-10-24<br />.value: 2.566879e+03","date: 1967-10-24<br />.value: 2.679966e+03","date: 1968-10-24<br />.value: 2.761376e+03","date: 1969-10-24<br />.value: 2.671605e+03","date: 1970-10-24<br />.value: 2.746399e+03","date: 1971-10-24<br />.value: 2.882011e+03","date: 1972-10-24<br />.value: 3.114166e+03","date: 1973-10-24<br />.value: 3.212193e+03","date: 1974-10-24<br />.value: 3.361115e+03","date: 1975-10-24<br />.value: 3.577918e+03","date: 1976-10-24<br />.value: 3.773050e+03","date: 1977-10-24<br />.value: 3.946593e+03","date: 1978-10-24<br />.value: 4.139358e+03","date: 1979-10-24<br />.value: 4.273323e+03","date: 1980-10-24<br />.value: 4.692154e+03","date: 1981-10-24<br />.value: 4.761948e+03","date: 1982-10-24<br />.value: 4.899983e+03","date: 1983-10-24<br />.value: 4.996685e+03","date: 1984-10-24<br />.value: 5.104227e+03","date: 1985-10-24<br />.value: 5.244308e+03","date: 1986-10-24<br />.value: 5.474766e+03","date: 1987-10-24<br />.value: 5.603602e+03","date: 1988-10-24<br />.value: 5.768691e+03","date: 1989-10-24<br />.value: 3.657289e+03","date: 1990-10-24<br />.value: 3.473051e+03","date: 1991-10-24<br />.value: 3.139280e+03"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(44,62,80,1)","dash":"solid"},"hoveron":"points","showlegend":false,"xaxis":"x2","yaxis":"y2","hoverinfo":"text","frame":null},{"x":["1912-10-24","1913-10-24","1914-10-24","1915-10-24","1916-10-24","1917-10-24","1918-10-24","1919-10-24","1920-10-24","1921-10-24","1922-10-24","1923-10-24","1924-10-24","1925-10-24","1926-10-24","1927-10-24","1928-10-24","1929-10-24","1930-10-24","1931-10-24","1932-10-24","1933-10-24","1934-10-24","1935-10-24","1936-10-24","1937-10-24","1938-10-24","1939-10-24","1940-10-24","1941-10-24","1942-10-24","1943-10-24","1944-10-24","1945-10-24","1946-10-24","1947-10-24","1948-10-24","1949-10-24","1950-10-24","1951-10-24","1952-10-24","1953-10-24","1954-10-24","1955-10-24","1956-10-24","1957-10-24","1958-10-24","1959-10-24","1960-10-24","1961-10-24","1962-10-24","1963-10-24","1964-10-24","1965-10-24","1966-10-24","1967-10-24","1968-10-24","1969-10-24","1970-10-24","1971-10-24","1972-10-24","1973-10-24","1974-10-24","1975-10-24","1976-10-24","1977-10-24","1978-10-24","1979-10-24","1980-10-24","1981-10-24","1982-10-24","1983-10-24","1984-10-24","1985-10-24","1986-10-24","1987-10-24","1988-10-24","1989-10-24","1990-10-24","1991-10-24","1992-10-24","1993-10-24","1994-10-24","1995-10-24","1996-10-24","1997-10-24","1998-10-24","1999-10-24","2000-10-24","2001-10-24","2002-10-24","2003-10-24","2004-10-24","2005-10-24","2006-10-24","2007-10-24","2008-10-24","2009-10-24","2010-10-24","2011-10-24","2012-10-24","2013-10-24","2014-10-24","2015-10-24","2016-10-24","2017-10-24","2018-10-24","2019-10-24","2020-10-24","2021-10-24","2022-10-24"],"y":[2.2730340121878,2.17876291736654,2.59874941790899,1.7706292922529,1.56252026754441,1.32865338092096,1.36131767723189,1.333293560529,1.59226989620708,1.70083319085234,3.21599579169756,6.94275787827169,5.94760245250734,5.7393331190843,21.0406861110755,21.7796591986184,22.067572549312,25.9165938557113,40.7788483909627,38.1816177237185,39.470809353332,39.8877620937993,41.8042870761337,49.4151897866163,54.2358746846553,87.0384477779076,100.851592136792,105.471519462007,121.271875012179,125.40936784313,117.372602748562,138.052530971693,169.182197608888,185.17556022012,199.041010169761,213.881188812857,243.64389569845,245.846390218993,294.756829288764,351.31659515308,374.878475314094,399.63450777586,425.256282397352,479.883090380239,523.67740159769,580.349097301616,611.609620067042,654.42029628823,723.259991444453,790.889318191887,863.385760043486,917.278492299719,974.192010545228,1087.58779311954,1206.87437394915,1325.11362329058,1441.5717168031,1563.66196952089,1706.77787653338,1740.23981208104,1863.47481723901,1924.05956727354,1898.89733354987,1474.25184959576,1509.30138602881,1465.89703483692,1366.47581377308,1367.10263443224,1278.96829695075,1220.28108267061,926.001230996761,796.61921919565,681.184386246695,578.927068387984,568.000609545554,557.385741769209,524.878755518325,482.312512460535,485.254076010609,484.542921141084,480.353055806105,480.466557719879,530.034706764828,506.242319849207,528.283003323597,543.176527539351,574.194293254596,547.876787593111,524.361761402559,534.370213092144,509.045828808085,487.012198311748,476.38116660784,420.398493772561,455.549455068518,449.382691421596,437.082440449096,458.69748800054,464.065930454871,442.839256575428,432.937479906509,431.961209787587,431.970492103092,448.605410811831,454.176836157949,469.39813851214,496.153936281432,516.609297088778,465.84514084166,469.132720344242,454.453737010673],"text":["date: 1912-10-24<br />.value: 2.273034e+00","date: 1913-10-24<br />.value: 2.178763e+00","date: 1914-10-24<br />.value: 2.598749e+00","date: 1915-10-24<br />.value: 1.770629e+00","date: 1916-10-24<br />.value: 1.562520e+00","date: 1917-10-24<br />.value: 1.328653e+00","date: 1918-10-24<br />.value: 1.361318e+00","date: 1919-10-24<br />.value: 1.333294e+00","date: 1920-10-24<br />.value: 1.592270e+00","date: 1921-10-24<br />.value: 1.700833e+00","date: 1922-10-24<br />.value: 3.215996e+00","date: 1923-10-24<br />.value: 6.942758e+00","date: 1924-10-24<br />.value: 5.947602e+00","date: 1925-10-24<br />.value: 5.739333e+00","date: 1926-10-24<br />.value: 2.104069e+01","date: 1927-10-24<br />.value: 2.177966e+01","date: 1928-10-24<br />.value: 2.206757e+01","date: 1929-10-24<br />.value: 2.591659e+01","date: 1930-10-24<br />.value: 4.077885e+01","date: 1931-10-24<br />.value: 3.818162e+01","date: 1932-10-24<br />.value: 3.947081e+01","date: 1933-10-24<br />.value: 3.988776e+01","date: 1934-10-24<br />.value: 4.180429e+01","date: 1935-10-24<br />.value: 4.941519e+01","date: 1936-10-24<br />.value: 5.423587e+01","date: 1937-10-24<br />.value: 8.703845e+01","date: 1938-10-24<br />.value: 1.008516e+02","date: 1939-10-24<br />.value: 1.054715e+02","date: 1940-10-24<br />.value: 1.212719e+02","date: 1941-10-24<br />.value: 1.254094e+02","date: 1942-10-24<br />.value: 1.173726e+02","date: 1943-10-24<br />.value: 1.380525e+02","date: 1944-10-24<br />.value: 1.691822e+02","date: 1945-10-24<br />.value: 1.851756e+02","date: 1946-10-24<br />.value: 1.990410e+02","date: 1947-10-24<br />.value: 2.138812e+02","date: 1948-10-24<br />.value: 2.436439e+02","date: 1949-10-24<br />.value: 2.458464e+02","date: 1950-10-24<br />.value: 2.947568e+02","date: 1951-10-24<br />.value: 3.513166e+02","date: 1952-10-24<br />.value: 3.748785e+02","date: 1953-10-24<br />.value: 3.996345e+02","date: 1954-10-24<br />.value: 4.252563e+02","date: 1955-10-24<br />.value: 4.798831e+02","date: 1956-10-24<br />.value: 5.236774e+02","date: 1957-10-24<br />.value: 5.803491e+02","date: 1958-10-24<br />.value: 6.116096e+02","date: 1959-10-24<br />.value: 6.544203e+02","date: 1960-10-24<br />.value: 7.232600e+02","date: 1961-10-24<br />.value: 7.908893e+02","date: 1962-10-24<br />.value: 8.633858e+02","date: 1963-10-24<br />.value: 9.172785e+02","date: 1964-10-24<br />.value: 9.741920e+02","date: 1965-10-24<br />.value: 1.087588e+03","date: 1966-10-24<br />.value: 1.206874e+03","date: 1967-10-24<br />.value: 1.325114e+03","date: 1968-10-24<br />.value: 1.441572e+03","date: 1969-10-24<br />.value: 1.563662e+03","date: 1970-10-24<br />.value: 1.706778e+03","date: 1971-10-24<br />.value: 1.740240e+03","date: 1972-10-24<br />.value: 1.863475e+03","date: 1973-10-24<br />.value: 1.924060e+03","date: 1974-10-24<br />.value: 1.898897e+03","date: 1975-10-24<br />.value: 1.474252e+03","date: 1976-10-24<br />.value: 1.509301e+03","date: 1977-10-24<br />.value: 1.465897e+03","date: 1978-10-24<br />.value: 1.366476e+03","date: 1979-10-24<br />.value: 1.367103e+03","date: 1980-10-24<br />.value: 1.278968e+03","date: 1981-10-24<br />.value: 1.220281e+03","date: 1982-10-24<br />.value: 9.260012e+02","date: 1983-10-24<br />.value: 7.966192e+02","date: 1984-10-24<br />.value: 6.811844e+02","date: 1985-10-24<br />.value: 5.789271e+02","date: 1986-10-24<br />.value: 5.680006e+02","date: 1987-10-24<br />.value: 5.573857e+02","date: 1988-10-24<br />.value: 5.248788e+02","date: 1989-10-24<br />.value: 4.823125e+02","date: 1990-10-24<br />.value: 4.852541e+02","date: 1991-10-24<br />.value: 4.845429e+02","date: 1992-10-24<br />.value: 4.803531e+02","date: 1993-10-24<br />.value: 4.804666e+02","date: 1994-10-24<br />.value: 5.300347e+02","date: 1995-10-24<br />.value: 5.062423e+02","date: 1996-10-24<br />.value: 5.282830e+02","date: 1997-10-24<br />.value: 5.431765e+02","date: 1998-10-24<br />.value: 5.741943e+02","date: 1999-10-24<br />.value: 5.478768e+02","date: 2000-10-24<br />.value: 5.243618e+02","date: 2001-10-24<br />.value: 5.343702e+02","date: 2002-10-24<br />.value: 5.090458e+02","date: 2003-10-24<br />.value: 4.870122e+02","date: 2004-10-24<br />.value: 4.763812e+02","date: 2005-10-24<br />.value: 4.203985e+02","date: 2006-10-24<br />.value: 4.555495e+02","date: 2007-10-24<br />.value: 4.493827e+02","date: 2008-10-24<br />.value: 4.370824e+02","date: 2009-10-24<br />.value: 4.586975e+02","date: 2010-10-24<br />.value: 4.640659e+02","date: 2011-10-24<br />.value: 4.428393e+02","date: 2012-10-24<br />.value: 4.329375e+02","date: 2013-10-24<br />.value: 4.319612e+02","date: 2014-10-24<br />.value: 4.319705e+02","date: 2015-10-24<br />.value: 4.486054e+02","date: 2016-10-24<br />.value: 4.541768e+02","date: 2017-10-24<br />.value: 4.693981e+02","date: 2018-10-24<br />.value: 4.961539e+02","date: 2019-10-24<br />.value: 5.166093e+02","date: 2020-10-24<br />.value: 4.658451e+02","date: 2021-10-24<br />.value: 4.691327e+02","date: 2022-10-24<br />.value: 4.544537e+02"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(44,62,80,1)","dash":"solid"},"hoveron":"points","showlegend":false,"xaxis":"x3","yaxis":"y3","hoverinfo":"text","frame":null},{"x":["1938-10-24","1939-10-24","1940-10-24","1941-10-24","1942-10-24","1943-10-24","1944-10-24","1945-10-24","1946-10-24","1947-10-24","1948-10-24","1949-10-24","1950-10-24","1951-10-24","1952-10-24","1953-10-24","1954-10-24","1955-10-24","1956-10-24","1957-10-24","1958-10-24","1959-10-24","1960-10-24","1961-10-24","1962-10-24","1963-10-24","1964-10-24","1965-10-24","1966-10-24","1967-10-24","1968-10-24","1969-10-24","1970-10-24","1971-10-24","1972-10-24","1973-10-24","1974-10-24","1975-10-24","1976-10-24","1977-10-24","1978-10-24","1979-10-24","1980-10-24","1981-10-24","1982-10-24","1983-10-24","1984-10-24","1985-10-24","1986-10-24","1987-10-24","1988-10-24","1989-10-24","1990-10-24","1991-10-24","1992-10-24","1993-10-24","1994-10-24","1995-10-24","1996-10-24","1997-10-24","1998-10-24","1999-10-24","2000-10-24","2001-10-24","2002-10-24","2003-10-24","2004-10-24","2005-10-24","2006-10-24","2007-10-24","2008-10-24","2009-10-24","2010-10-24","2011-10-24","2012-10-24","2013-10-24","2014-10-24","2015-10-24","2016-10-24","2017-10-24","2018-10-24","2019-10-24","2020-10-24","2021-10-24","2022-10-24"],"y":[0.0197395058953151,0.156879224630646,0.202379782664089,0.171873273553148,0.180665927401346,0.194132258200919,0.310823443940291,0.849835416519266,2.39042489362503,3.58308504226417,5.69666149145129,6.94216489553904,7.95747886301275,11.0845341037921,12.0375448869142,12.2940932633672,13.8712837721818,14.0468697303539,14.393173756499,14.4409809622668,14.7745579120776,15.9443944431976,18.2028058648193,20.5538689442304,23.062796377392,25.9412108895416,27.9929052716827,32.5590239750153,39.0223935595961,42.4690607168702,46.8025289068647,49.6080431696521,57.9900843615139,72.1299981495095,91.1677309150163,287.684493625171,738.570488090137,617.310531304208,1254.80007921136,1355.97798999274,1235.30692108627,1420.70120274509,1492.26784788929,1507.12899967456,1014.81048990887,715.212393670649,669.311791122459,540.874329757563,791.765583438344,685.114842168767,845.401318315427,847.883691788565,1071.16666815412,1342.17934352217,1374.858116768,1334.59891819158,1349.0431649642,1350.9611743953,1368.1340628694,1353.89867673815,1393.99192180773,1283.95210268444,1376.69001500848,1360.07112030716,1279.49240809662,1507.41649381985,1552.36886386991,1689.01044391863,1673.83358389971,1599.01398691477,1675.01519062768,1542.80181957054,1578.49562180938,1726.16915998109,1819.92205410137,1685.53814981657,1713.47145881372,1854.59613712096,2009.79644401376,1893.67139713296,1930.59554395672,1897.6610902916,1784.33397517894,1777.46138910926,1962.15758460958],"text":["date: 1938-10-24<br />.value: 1.973951e-02","date: 1939-10-24<br />.value: 1.568792e-01","date: 1940-10-24<br />.value: 2.023798e-01","date: 1941-10-24<br />.value: 1.718733e-01","date: 1942-10-24<br />.value: 1.806659e-01","date: 1943-10-24<br />.value: 1.941323e-01","date: 1944-10-24<br />.value: 3.108234e-01","date: 1945-10-24<br />.value: 8.498354e-01","date: 1946-10-24<br />.value: 2.390425e+00","date: 1947-10-24<br />.value: 3.583085e+00","date: 1948-10-24<br />.value: 5.696661e+00","date: 1949-10-24<br />.value: 6.942165e+00","date: 1950-10-24<br />.value: 7.957479e+00","date: 1951-10-24<br />.value: 1.108453e+01","date: 1952-10-24<br />.value: 1.203754e+01","date: 1953-10-24<br />.value: 1.229409e+01","date: 1954-10-24<br />.value: 1.387128e+01","date: 1955-10-24<br />.value: 1.404687e+01","date: 1956-10-24<br />.value: 1.439317e+01","date: 1957-10-24<br />.value: 1.444098e+01","date: 1958-10-24<br />.value: 1.477456e+01","date: 1959-10-24<br />.value: 1.594439e+01","date: 1960-10-24<br />.value: 1.820281e+01","date: 1961-10-24<br />.value: 2.055387e+01","date: 1962-10-24<br />.value: 2.306280e+01","date: 1963-10-24<br />.value: 2.594121e+01","date: 1964-10-24<br />.value: 2.799291e+01","date: 1965-10-24<br />.value: 3.255902e+01","date: 1966-10-24<br />.value: 3.902239e+01","date: 1967-10-24<br />.value: 4.246906e+01","date: 1968-10-24<br />.value: 4.680253e+01","date: 1969-10-24<br />.value: 4.960804e+01","date: 1970-10-24<br />.value: 5.799008e+01","date: 1971-10-24<br />.value: 7.213000e+01","date: 1972-10-24<br />.value: 9.116773e+01","date: 1973-10-24<br />.value: 2.876845e+02","date: 1974-10-24<br />.value: 7.385705e+02","date: 1975-10-24<br />.value: 6.173105e+02","date: 1976-10-24<br />.value: 1.254800e+03","date: 1977-10-24<br />.value: 1.355978e+03","date: 1978-10-24<br />.value: 1.235307e+03","date: 1979-10-24<br />.value: 1.420701e+03","date: 1980-10-24<br />.value: 1.492268e+03","date: 1981-10-24<br />.value: 1.507129e+03","date: 1982-10-24<br />.value: 1.014810e+03","date: 1983-10-24<br />.value: 7.152124e+02","date: 1984-10-24<br />.value: 6.693118e+02","date: 1985-10-24<br />.value: 5.408743e+02","date: 1986-10-24<br />.value: 7.917656e+02","date: 1987-10-24<br />.value: 6.851148e+02","date: 1988-10-24<br />.value: 8.454013e+02","date: 1989-10-24<br />.value: 8.478837e+02","date: 1990-10-24<br />.value: 1.071167e+03","date: 1991-10-24<br />.value: 1.342179e+03","date: 1992-10-24<br />.value: 1.374858e+03","date: 1993-10-24<br />.value: 1.334599e+03","date: 1994-10-24<br />.value: 1.349043e+03","date: 1995-10-24<br />.value: 1.350961e+03","date: 1996-10-24<br />.value: 1.368134e+03","date: 1997-10-24<br />.value: 1.353899e+03","date: 1998-10-24<br />.value: 1.393992e+03","date: 1999-10-24<br />.value: 1.283952e+03","date: 2000-10-24<br />.value: 1.376690e+03","date: 2001-10-24<br />.value: 1.360071e+03","date: 2002-10-24<br />.value: 1.279492e+03","date: 2003-10-24<br />.value: 1.507416e+03","date: 2004-10-24<br />.value: 1.552369e+03","date: 2005-10-24<br />.value: 1.689010e+03","date: 2006-10-24<br />.value: 1.673834e+03","date: 2007-10-24<br />.value: 1.599014e+03","date: 2008-10-24<br />.value: 1.675015e+03","date: 2009-10-24<br />.value: 1.542802e+03","date: 2010-10-24<br />.value: 1.578496e+03","date: 2011-10-24<br />.value: 1.726169e+03","date: 2012-10-24<br />.value: 1.819922e+03","date: 2013-10-24<br />.value: 1.685538e+03","date: 2014-10-24<br />.value: 1.713471e+03","date: 2015-10-24<br />.value: 1.854596e+03","date: 2016-10-24<br />.value: 2.009796e+03","date: 2017-10-24<br />.value: 1.893671e+03","date: 2018-10-24<br />.value: 1.930596e+03","date: 2019-10-24<br />.value: 1.897661e+03","date: 2020-10-24<br />.value: 1.784334e+03","date: 2021-10-24<br />.value: 1.777461e+03","date: 2022-10-24<br />.value: 1.962158e+03"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(44,62,80,1)","dash":"solid"},"hoveron":"points","showlegend":false,"xaxis":"x4","yaxis":"y4","hoverinfo":"text","frame":null},{"x":["1945-10-24","1946-10-24","1947-10-24","1948-10-24","1949-10-24","1950-10-24","1951-10-24","1952-10-24","1953-10-24","1954-10-24","1955-10-24","1956-10-24","1957-10-24","1958-10-24","1959-10-24","1960-10-24","1961-10-24","1962-10-24","1963-10-24","1964-10-24","1965-10-24","1966-10-24","1967-10-24","1968-10-24","1969-10-24","1970-10-24","1971-10-24","1972-10-24","1973-10-24","1974-10-24","1975-10-24","1976-10-24","1977-10-24","1978-10-24","1979-10-24","1980-10-24","1981-10-24","1982-10-24","1983-10-24","1984-10-24","1985-10-24","1986-10-24","1987-10-24","1988-10-24","1989-10-24","1990-10-24","1991-10-24","1992-10-24","1993-10-24","1994-10-24","1995-10-24","1996-10-24","1997-10-24","1998-10-24","1999-10-24","2000-10-24","2001-10-24","2002-10-24","2003-10-24","2004-10-24","2005-10-24","2006-10-24","2007-10-24","2008-10-24","2009-10-24","2010-10-24","2011-10-24","2012-10-24","2013-10-24","2014-10-24","2015-10-24","2016-10-24","2017-10-24","2018-10-24","2019-10-24","2020-10-24","2021-10-24","2022-10-24"],"y":[16.228095030136,39.6686767381187,63.1092584544963,86.5498401464467,109.539641447042,132.529442746058,155.519244043736,178.50904530829,297.281547343305,416.054049454302,534.826551377988,653.59905333134,354.313408143366,891.144057436733,1009.91655939008,1128.68906161269,674.759765159383,674.759765159383,736.101561908556,785.174999227577,809.711718022349,883.321874162612,613.417968163985,808.700069088019,975.580561805197,1259.33482038998,1267.9054961402,1288.58504273842,1276.58658732779,1345.18389185208,1398.29361956072,1438.12898246547,1487.54838760565,1671.46336283781,1713.3524491422,1677.29900313599,1681.32902155114,1802.20252173941,1932.59660204496,2134.63845615564,2359.26464101471,2418.11913578831,2510.21451954226,2650.26441494324,2851.17029109173,2919.19375357697,2932.66321047896,3014.53478908354,3114.14926685652,3325.49532442076,3494.99951464243,3691.81857353082,3584.44428209246,3523.76140330471,3473.78106283762,3743.81336195276,3980.04787178811,4193.37838273228,4962.86316819439,5741.04006859665,6397.02669535968,6950.36969783699,7464.69703177791,7852.86114816505,8426.11998783509,9272.9450543655,10181.6938453573,10670.4151773709,10749.3761208227,10477.8154348874,10133.2925559505,9221.23635989209,9511.52711080173,9967.35098287703,10365.8550895362,10515.8543690961,11120.7375920092,12290.3795710213],"text":["date: 1945-10-24<br />.value: 1.622810e+01","date: 1946-10-24<br />.value: 3.966868e+01","date: 1947-10-24<br />.value: 6.310926e+01","date: 1948-10-24<br />.value: 8.654984e+01","date: 1949-10-24<br />.value: 1.095396e+02","date: 1950-10-24<br />.value: 1.325294e+02","date: 1951-10-24<br />.value: 1.555192e+02","date: 1952-10-24<br />.value: 1.785090e+02","date: 1953-10-24<br />.value: 2.972815e+02","date: 1954-10-24<br />.value: 4.160540e+02","date: 1955-10-24<br />.value: 5.348266e+02","date: 1956-10-24<br />.value: 6.535991e+02","date: 1957-10-24<br />.value: 3.543134e+02","date: 1958-10-24<br />.value: 8.911441e+02","date: 1959-10-24<br />.value: 1.009917e+03","date: 1960-10-24<br />.value: 1.128689e+03","date: 1961-10-24<br />.value: 6.747598e+02","date: 1962-10-24<br />.value: 6.747598e+02","date: 1963-10-24<br />.value: 7.361016e+02","date: 1964-10-24<br />.value: 7.851750e+02","date: 1965-10-24<br />.value: 8.097117e+02","date: 1966-10-24<br />.value: 8.833219e+02","date: 1967-10-24<br />.value: 6.134180e+02","date: 1968-10-24<br />.value: 8.087001e+02","date: 1969-10-24<br />.value: 9.755806e+02","date: 1970-10-24<br />.value: 1.259335e+03","date: 1971-10-24<br />.value: 1.267905e+03","date: 1972-10-24<br />.value: 1.288585e+03","date: 1973-10-24<br />.value: 1.276587e+03","date: 1974-10-24<br />.value: 1.345184e+03","date: 1975-10-24<br />.value: 1.398294e+03","date: 1976-10-24<br />.value: 1.438129e+03","date: 1977-10-24<br />.value: 1.487548e+03","date: 1978-10-24<br />.value: 1.671463e+03","date: 1979-10-24<br />.value: 1.713352e+03","date: 1980-10-24<br />.value: 1.677299e+03","date: 1981-10-24<br />.value: 1.681329e+03","date: 1982-10-24<br />.value: 1.802203e+03","date: 1983-10-24<br />.value: 1.932597e+03","date: 1984-10-24<br />.value: 2.134638e+03","date: 1985-10-24<br />.value: 2.359265e+03","date: 1986-10-24<br />.value: 2.418119e+03","date: 1987-10-24<br />.value: 2.510215e+03","date: 1988-10-24<br />.value: 2.650264e+03","date: 1989-10-24<br />.value: 2.851170e+03","date: 1990-10-24<br />.value: 2.919194e+03","date: 1991-10-24<br />.value: 2.932663e+03","date: 1992-10-24<br />.value: 3.014535e+03","date: 1993-10-24<br />.value: 3.114149e+03","date: 1994-10-24<br />.value: 3.325495e+03","date: 1995-10-24<br />.value: 3.495000e+03","date: 1996-10-24<br />.value: 3.691819e+03","date: 1997-10-24<br />.value: 3.584444e+03","date: 1998-10-24<br />.value: 3.523761e+03","date: 1999-10-24<br />.value: 3.473781e+03","date: 2000-10-24<br />.value: 3.743813e+03","date: 2001-10-24<br />.value: 3.980048e+03","date: 2002-10-24<br />.value: 4.193378e+03","date: 2003-10-24<br />.value: 4.962863e+03","date: 2004-10-24<br />.value: 5.741040e+03","date: 2005-10-24<br />.value: 6.397027e+03","date: 2006-10-24<br />.value: 6.950370e+03","date: 2007-10-24<br />.value: 7.464697e+03","date: 2008-10-24<br />.value: 7.852861e+03","date: 2009-10-24<br />.value: 8.426120e+03","date: 2010-10-24<br />.value: 9.272945e+03","date: 2011-10-24<br />.value: 1.018169e+04","date: 2012-10-24<br />.value: 1.067042e+04","date: 2013-10-24<br />.value: 1.074938e+04","date: 2014-10-24<br />.value: 1.047782e+04","date: 2015-10-24<br />.value: 1.013329e+04","date: 2016-10-24<br />.value: 9.221236e+03","date: 2017-10-24<br />.value: 9.511527e+03","date: 2018-10-24<br />.value: 9.967351e+03","date: 2019-10-24<br />.value: 1.036586e+04","date: 2020-10-24<br />.value: 1.051585e+04","date: 2021-10-24<br />.value: 1.112074e+04","date: 2022-10-24<br />.value: 1.229038e+04"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(44,62,80,1)","dash":"solid"},"hoveron":"points","showlegend":false,"xaxis":"x5","yaxis":"y5","hoverinfo":"text","frame":null},{"x":["1989-10-24","1990-10-24","1991-10-24","1992-10-24","1993-10-24","1994-10-24","1995-10-24","1996-10-24","1997-10-24","1998-10-24","1999-10-24","2000-10-24","2001-10-24","2002-10-24","2003-10-24","2004-10-24","2005-10-24","2006-10-24","2007-10-24","2008-10-24","2009-10-24","2010-10-24","2011-10-24","2012-10-24","2013-10-24","2014-10-24","2015-10-24","2016-10-24","2017-10-24","2018-10-24","2019-10-24","2020-10-24","2021-10-24","2022-10-24"],"y":[1693.43217716262,1733.86996832489,1724.29157491297,1654.52092917355,1595.87384115673,1569.26262924412,1556.64842144902,1483.16858261764,1422.60646648182,1484.1623290074,1483.75801622585,1429.12795860994,1411.40587006425,1456.13377555047,1521.08842481353,1563.32372372097,1551.87579687463,1563.77664132029,1544.43600191045,1540.45813838504,1309.85995366535,1435.72890780055,1450.58747692011,1388.62394567277,1423.98238899303,1314.81169802048,1253.45639354235,1265.60800613004,1403.84442044742,1605.8684966794,1614.38345370428,1484.22081706453,1498.24889316881,1254.52647723756],"text":["date: 1989-10-24<br />.value: 1.693432e+03","date: 1990-10-24<br />.value: 1.733870e+03","date: 1991-10-24<br />.value: 1.724292e+03","date: 1992-10-24<br />.value: 1.654521e+03","date: 1993-10-24<br />.value: 1.595874e+03","date: 1994-10-24<br />.value: 1.569263e+03","date: 1995-10-24<br />.value: 1.556648e+03","date: 1996-10-24<br />.value: 1.483169e+03","date: 1997-10-24<br />.value: 1.422606e+03","date: 1998-10-24<br />.value: 1.484162e+03","date: 1999-10-24<br />.value: 1.483758e+03","date: 2000-10-24<br />.value: 1.429128e+03","date: 2001-10-24<br />.value: 1.411406e+03","date: 2002-10-24<br />.value: 1.456134e+03","date: 2003-10-24<br />.value: 1.521088e+03","date: 2004-10-24<br />.value: 1.563324e+03","date: 2005-10-24<br />.value: 1.551876e+03","date: 2006-10-24<br />.value: 1.563777e+03","date: 2007-10-24<br />.value: 1.544436e+03","date: 2008-10-24<br />.value: 1.540458e+03","date: 2009-10-24<br />.value: 1.309860e+03","date: 2010-10-24<br />.value: 1.435729e+03","date: 2011-10-24<br />.value: 1.450587e+03","date: 2012-10-24<br />.value: 1.388624e+03","date: 2013-10-24<br />.value: 1.423982e+03","date: 2014-10-24<br />.value: 1.314812e+03","date: 2015-10-24<br />.value: 1.253456e+03","date: 2016-10-24<br />.value: 1.265608e+03","date: 2017-10-24<br />.value: 1.403844e+03","date: 2018-10-24<br />.value: 1.605868e+03","date: 2019-10-24<br />.value: 1.614383e+03","date: 2020-10-24<br />.value: 1.484221e+03","date: 2021-10-24<br />.value: 1.498249e+03","date: 2022-10-24<br />.value: 1.254526e+03"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(44,62,80,1)","dash":"solid"},"hoveron":"points","showlegend":false,"xaxis":"x6","yaxis":"y6","hoverinfo":"text","frame":null},{"x":["1884-10-24","1885-10-24","1886-10-24","1887-10-24","1888-10-24","1889-10-24","1890-10-24","1891-10-24","1892-10-24","1893-10-24","1894-10-24","1895-10-24","1896-10-24","1897-10-24","1898-10-24","1899-10-24","1900-10-24","1901-10-24","1902-10-24","1903-10-24","1904-10-24","1905-10-24","1906-10-24","1907-10-24","1908-10-24","1909-10-24","1910-10-24","1911-10-24","1912-10-24","1913-10-24","1914-10-24","1915-10-24","1916-10-24","1917-10-24","1918-10-24","1919-10-24","1920-10-24","1921-10-24","1922-10-24","1923-10-24","1924-10-24","1925-10-24","1926-10-24","1927-10-24","1928-10-24","1929-10-24","1930-10-24","1931-10-24","1932-10-24","1933-10-24","1934-10-24","1935-10-24","1936-10-24","1937-10-24","1938-10-24","1939-10-24","1940-10-24","1941-10-24","1942-10-24","1943-10-24","1944-10-24","1945-10-24","1946-10-24","1947-10-24","1948-10-24","1949-10-24","1950-10-24","1951-10-24","1952-10-24","1953-10-24","1954-10-24","1955-10-24","1956-10-24","1957-10-24","1958-10-24","1959-10-24","1960-10-24","1961-10-24","1962-10-24","1963-10-24","1964-10-24","1965-10-24","1966-10-24","1967-10-24","1968-10-24","1969-10-24","1970-10-24","1971-10-24","1972-10-24","1973-10-24","1974-10-24","1975-10-24","1976-10-24","1977-10-24","1978-10-24","1979-10-24","1980-10-24","1981-10-24","1982-10-24","1983-10-24","1984-10-24","1985-10-24","1986-10-24","1987-10-24","1988-10-24","1989-10-24","1990-10-24","1991-10-24","1992-10-24","1993-10-24","1994-10-24","1995-10-24","1996-10-24","1997-10-24","1998-10-24","1999-10-24","2000-10-24","2001-10-24","2002-10-24","2003-10-24","2004-10-24","2005-10-24","2006-10-24","2007-10-24","2008-10-24","2009-10-24","2010-10-24","2011-10-24","2012-10-24","2013-10-24","2014-10-24","2015-10-24","2016-10-24","2017-10-24","2018-10-24","2019-10-24","2020-10-24","2021-10-24","2022-10-24"],"y":[73.4873030286515,64.1948915509056,55.3328479651647,46.8950120219339,38.8752234717186,31.267322065024,24.0651475523553,17.2625396842178,10.8533382111166,4.83138288355704,-0.809486547955649,-6.07543033291626,-10.9726087208195,-15.5071819611602,-19.6853103034331,-23.513153997133,-26.9968732917546,-30.1426284367927,-32.9725906549223,-35.4943668124321,-37.6922706242368,-39.5506158052513,-41.0537160703904,-42.185885134569,-42.9314367127019,-43.274684519704,-43.19994227049,-42.691523679975,-41.7337424630736,-40.3109123347008,-38.4073470097714,-36.0073602032002,-33.0952656299021,-29.6553770047919,-25.6720080427846,-21.4583698155288,-17.2943909576545,-13.1144615285477,-8.85297158759453,-4.4443111941811,0.177129592306458,5.07696071248205,10.3207921069595,15.9742337163528,22.1028954812758,28.7723873423423,36.0483192401662,43.9963011153614,52.6819429085419,62.1708545603214,72.5286460113139,83.8209272021333,96.1133080733934,110.283427019497,126.962951761134,145.848536228349,166.636834351189,189.024500059699,212.708187283923,237.384549953908,262.750241999698,288.501917351339,314.336229938876,339.949833692354,365.03938254182,389.301530417318,412.432931248893,434.130238966592,454.090107500458,472.009190780539,489.626888586063,508.724485923018,529.061464561588,550.397306271953,572.491492824296,595.103505988798,617.992827535641,640.918939235009,663.641322857081,685.919460172041,707.51283295007,728.18092296135,747.683211976063,765.779181764392,782.228314096517,796.790090742621,809.223993472887,819.289504057495,827.528420128849,834.682799279214,840.812513218564,845.977433656872,850.237432304112,853.652380870257,856.282151065281,858.186614599158,859.425643181861,860.059108523364,860.14688233364,859.748836322663,858.924842200407,857.734771676845,856.238496461951,854.495888265698,852.56681879806,850.204804246398,847.132138046937,843.361857782949,838.907001037706,833.780605394481,827.995708436545,821.56534774717,814.502560909629,806.820385507194,798.531859123136,789.650019340728,780.187903743242,770.15854991395,759.574995436124,748.450277893036,736.797434867958,724.629503944162,711.960113683401,698.789691754621,685.111474401887,670.918697869262,656.204598400812,640.962412240599,625.185375632688,608.866724821144,591.999696050031,574.577525563412,556.593449605352,538.040704419915,518.912526251166,499.202151343168,478.902815939985,458.007756285683,436.510208624324],"text":["date: 1884-10-24<br />.value_smooth:    73.4873030","date: 1885-10-24<br />.value_smooth:    64.1948916","date: 1886-10-24<br />.value_smooth:    55.3328480","date: 1887-10-24<br />.value_smooth:    46.8950120","date: 1888-10-24<br />.value_smooth:    38.8752235","date: 1889-10-24<br />.value_smooth:    31.2673221","date: 1890-10-24<br />.value_smooth:    24.0651476","date: 1891-10-24<br />.value_smooth:    17.2625397","date: 1892-10-24<br />.value_smooth:    10.8533382","date: 1893-10-24<br />.value_smooth:     4.8313829","date: 1894-10-24<br />.value_smooth:    -0.8094865","date: 1895-10-24<br />.value_smooth:    -6.0754303","date: 1896-10-24<br />.value_smooth:   -10.9726087","date: 1897-10-24<br />.value_smooth:   -15.5071820","date: 1898-10-24<br />.value_smooth:   -19.6853103","date: 1899-10-24<br />.value_smooth:   -23.5131540","date: 1900-10-24<br />.value_smooth:   -26.9968733","date: 1901-10-24<br />.value_smooth:   -30.1426284","date: 1902-10-24<br />.value_smooth:   -32.9725907","date: 1903-10-24<br />.value_smooth:   -35.4943668","date: 1904-10-24<br />.value_smooth:   -37.6922706","date: 1905-10-24<br />.value_smooth:   -39.5506158","date: 1906-10-24<br />.value_smooth:   -41.0537161","date: 1907-10-24<br />.value_smooth:   -42.1858851","date: 1908-10-24<br />.value_smooth:   -42.9314367","date: 1909-10-24<br />.value_smooth:   -43.2746845","date: 1910-10-24<br />.value_smooth:   -43.1999423","date: 1911-10-24<br />.value_smooth:   -42.6915237","date: 1912-10-24<br />.value_smooth:   -41.7337425","date: 1913-10-24<br />.value_smooth:   -40.3109123","date: 1914-10-24<br />.value_smooth:   -38.4073470","date: 1915-10-24<br />.value_smooth:   -36.0073602","date: 1916-10-24<br />.value_smooth:   -33.0952656","date: 1917-10-24<br />.value_smooth:   -29.6553770","date: 1918-10-24<br />.value_smooth:   -25.6720080","date: 1919-10-24<br />.value_smooth:   -21.4583698","date: 1920-10-24<br />.value_smooth:   -17.2943910","date: 1921-10-24<br />.value_smooth:   -13.1144615","date: 1922-10-24<br />.value_smooth:    -8.8529716","date: 1923-10-24<br />.value_smooth:    -4.4443112","date: 1924-10-24<br />.value_smooth:     0.1771296","date: 1925-10-24<br />.value_smooth:     5.0769607","date: 1926-10-24<br />.value_smooth:    10.3207921","date: 1927-10-24<br />.value_smooth:    15.9742337","date: 1928-10-24<br />.value_smooth:    22.1028955","date: 1929-10-24<br />.value_smooth:    28.7723873","date: 1930-10-24<br />.value_smooth:    36.0483192","date: 1931-10-24<br />.value_smooth:    43.9963011","date: 1932-10-24<br />.value_smooth:    52.6819429","date: 1933-10-24<br />.value_smooth:    62.1708546","date: 1934-10-24<br />.value_smooth:    72.5286460","date: 1935-10-24<br />.value_smooth:    83.8209272","date: 1936-10-24<br />.value_smooth:    96.1133081","date: 1937-10-24<br />.value_smooth:   110.2834270","date: 1938-10-24<br />.value_smooth:   126.9629518","date: 1939-10-24<br />.value_smooth:   145.8485362","date: 1940-10-24<br />.value_smooth:   166.6368344","date: 1941-10-24<br />.value_smooth:   189.0245001","date: 1942-10-24<br />.value_smooth:   212.7081873","date: 1943-10-24<br />.value_smooth:   237.3845500","date: 1944-10-24<br />.value_smooth:   262.7502420","date: 1945-10-24<br />.value_smooth:   288.5019174","date: 1946-10-24<br />.value_smooth:   314.3362299","date: 1947-10-24<br />.value_smooth:   339.9498337","date: 1948-10-24<br />.value_smooth:   365.0393825","date: 1949-10-24<br />.value_smooth:   389.3015304","date: 1950-10-24<br />.value_smooth:   412.4329312","date: 1951-10-24<br />.value_smooth:   434.1302390","date: 1952-10-24<br />.value_smooth:   454.0901075","date: 1953-10-24<br />.value_smooth:   472.0091908","date: 1954-10-24<br />.value_smooth:   489.6268886","date: 1955-10-24<br />.value_smooth:   508.7244859","date: 1956-10-24<br />.value_smooth:   529.0614646","date: 1957-10-24<br />.value_smooth:   550.3973063","date: 1958-10-24<br />.value_smooth:   572.4914928","date: 1959-10-24<br />.value_smooth:   595.1035060","date: 1960-10-24<br />.value_smooth:   617.9928275","date: 1961-10-24<br />.value_smooth:   640.9189392","date: 1962-10-24<br />.value_smooth:   663.6413229","date: 1963-10-24<br />.value_smooth:   685.9194602","date: 1964-10-24<br />.value_smooth:   707.5128330","date: 1965-10-24<br />.value_smooth:   728.1809230","date: 1966-10-24<br />.value_smooth:   747.6832120","date: 1967-10-24<br />.value_smooth:   765.7791818","date: 1968-10-24<br />.value_smooth:   782.2283141","date: 1969-10-24<br />.value_smooth:   796.7900907","date: 1970-10-24<br />.value_smooth:   809.2239935","date: 1971-10-24<br />.value_smooth:   819.2895041","date: 1972-10-24<br />.value_smooth:   827.5284201","date: 1973-10-24<br />.value_smooth:   834.6827993","date: 1974-10-24<br />.value_smooth:   840.8125132","date: 1975-10-24<br />.value_smooth:   845.9774337","date: 1976-10-24<br />.value_smooth:   850.2374323","date: 1977-10-24<br />.value_smooth:   853.6523809","date: 1978-10-24<br />.value_smooth:   856.2821511","date: 1979-10-24<br />.value_smooth:   858.1866146","date: 1980-10-24<br />.value_smooth:   859.4256432","date: 1981-10-24<br />.value_smooth:   860.0591085","date: 1982-10-24<br />.value_smooth:   860.1468823","date: 1983-10-24<br />.value_smooth:   859.7488363","date: 1984-10-24<br />.value_smooth:   858.9248422","date: 1985-10-24<br />.value_smooth:   857.7347717","date: 1986-10-24<br />.value_smooth:   856.2384965","date: 1987-10-24<br />.value_smooth:   854.4958883","date: 1988-10-24<br />.value_smooth:   852.5668188","date: 1989-10-24<br />.value_smooth:   850.2048042","date: 1990-10-24<br />.value_smooth:   847.1321380","date: 1991-10-24<br />.value_smooth:   843.3618578","date: 1992-10-24<br />.value_smooth:   838.9070010","date: 1993-10-24<br />.value_smooth:   833.7806054","date: 1994-10-24<br />.value_smooth:   827.9957084","date: 1995-10-24<br />.value_smooth:   821.5653477","date: 1996-10-24<br />.value_smooth:   814.5025609","date: 1997-10-24<br />.value_smooth:   806.8203855","date: 1998-10-24<br />.value_smooth:   798.5318591","date: 1999-10-24<br />.value_smooth:   789.6500193","date: 2000-10-24<br />.value_smooth:   780.1879037","date: 2001-10-24<br />.value_smooth:   770.1585499","date: 2002-10-24<br />.value_smooth:   759.5749954","date: 2003-10-24<br />.value_smooth:   748.4502779","date: 2004-10-24<br />.value_smooth:   736.7974349","date: 2005-10-24<br />.value_smooth:   724.6295039","date: 2006-10-24<br />.value_smooth:   711.9601137","date: 2007-10-24<br />.value_smooth:   698.7896918","date: 2008-10-24<br />.value_smooth:   685.1114744","date: 2009-10-24<br />.value_smooth:   670.9186979","date: 2010-10-24<br />.value_smooth:   656.2045984","date: 2011-10-24<br />.value_smooth:   640.9624122","date: 2012-10-24<br />.value_smooth:   625.1853756","date: 2013-10-24<br />.value_smooth:   608.8667248","date: 2014-10-24<br />.value_smooth:   591.9996961","date: 2015-10-24<br />.value_smooth:   574.5775256","date: 2016-10-24<br />.value_smooth:   556.5934496","date: 2017-10-24<br />.value_smooth:   538.0407044","date: 2018-10-24<br />.value_smooth:   518.9125263","date: 2019-10-24<br />.value_smooth:   499.2021513","date: 2020-10-24<br />.value_smooth:   478.9028159","date: 2021-10-24<br />.value_smooth:   458.0077563","date: 2022-10-24<br />.value_smooth:   436.5102086"],"type":"scatter","mode":"lines","line":{"width":3.77952755905512,"color":"rgba(51,102,255,1)","dash":"solid"},"hoveron":"points","showlegend":false,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":["1900-10-24","1901-10-24","1902-10-24","1903-10-24","1904-10-24","1905-10-24","1906-10-24","1907-10-24","1908-10-24","1909-10-24","1910-10-24","1911-10-24","1912-10-24","1913-10-24","1914-10-24","1915-10-24","1916-10-24","1917-10-24","1918-10-24","1919-10-24","1920-10-24","1921-10-24","1922-10-24","1923-10-24","1924-10-24","1925-10-24","1926-10-24","1927-10-24","1928-10-24","1929-10-24","1930-10-24","1931-10-24","1932-10-24","1933-10-24","1934-10-24","1935-10-24","1936-10-24","1937-10-24","1938-10-24","1939-10-24","1940-10-24","1941-10-24","1942-10-24","1943-10-24","1944-10-24","1945-10-24","1946-10-24","1947-10-24","1948-10-24","1949-10-24","1950-10-24","1951-10-24","1952-10-24","1953-10-24","1954-10-24","1955-10-24","1956-10-24","1957-10-24","1958-10-24","1959-10-24","1960-10-24","1961-10-24","1962-10-24","1963-10-24","1964-10-24","1965-10-24","1966-10-24","1967-10-24","1968-10-24","1969-10-24","1970-10-24","1971-10-24","1972-10-24","1973-10-24","1974-10-24","1975-10-24","1976-10-24","1977-10-24","1978-10-24","1979-10-24","1980-10-24","1981-10-24","1982-10-24","1983-10-24","1984-10-24","1985-10-24","1986-10-24","1987-10-24","1988-10-24","1989-10-24","1990-10-24","1991-10-24"],"y":[104.516094872438,88.5019888669616,73.7154449893484,60.136011372659,47.7432361499544,36.5166674542957,26.4358534187438,17.4803421763599,9.62968186020491,2.86342060333993,-2.83889346117403,-7.49771220027593,-11.1859426770806,-13.9239604847642,-15.6534202853575,-16.3159767408913,-15.8532845133962,-14.2069982649033,-11.3187726574431,-7.13026235304653,-1.58312201374434,5.38099369843268,13.8204301214538,22.9105832159397,31.9718998185977,41.2803015815267,51.1117101568255,61.742047196593,73.4472343529281,86.5031932779296,101.185845623696,117.771113042327,136.534917185922,157.753179706577,181.701822256394,206.272586316632,229.73357376573,252.925246851551,276.688067821959,301.86249892482,329.289002407998,359.808040519357,394.26007550676,433.485569618074,478.324985101161,529.618784203886,585.645182228891,644.217157423228,705.477316451375,769.568265977814,836.632612667023,906.812963183482,980.251924191673,1057.09210235607,1137.47610434116,1221.54653681142,1309.44600643133,1401.31711986537,1500.50607241326,1609.05400439406,1725.14594000129,1846.96690342845,1972.70191886904,2100.53601051658,2228.65420256458,2355.24151920653,2478.48298463594,2596.56362304633,2707.6684586312,2815.5364006462,2924.91690994422,3035.51083811465,3147.01903674685,3259.14235743019,3371.58165175406,3484.03777130783,3596.21156768087,3707.80389246256,3818.51559724227,3928.04753360938,4036.10055315325,4142.73115431421,4248.28073864381,4352.87751239833,4456.64968183402,4559.72545320715,4662.23303277398,4764.30062679078,4866.05644151381,4967.62868319932,5069.14555810358,5170.73527248287],"text":["date: 1900-10-24<br />.value_smooth:   104.5160949","date: 1901-10-24<br />.value_smooth:    88.5019889","date: 1902-10-24<br />.value_smooth:    73.7154450","date: 1903-10-24<br />.value_smooth:    60.1360114","date: 1904-10-24<br />.value_smooth:    47.7432361","date: 1905-10-24<br />.value_smooth:    36.5166675","date: 1906-10-24<br />.value_smooth:    26.4358534","date: 1907-10-24<br />.value_smooth:    17.4803422","date: 1908-10-24<br />.value_smooth:     9.6296819","date: 1909-10-24<br />.value_smooth:     2.8634206","date: 1910-10-24<br />.value_smooth:    -2.8388935","date: 1911-10-24<br />.value_smooth:    -7.4977122","date: 1912-10-24<br />.value_smooth:   -11.1859427","date: 1913-10-24<br />.value_smooth:   -13.9239605","date: 1914-10-24<br />.value_smooth:   -15.6534203","date: 1915-10-24<br />.value_smooth:   -16.3159767","date: 1916-10-24<br />.value_smooth:   -15.8532845","date: 1917-10-24<br />.value_smooth:   -14.2069983","date: 1918-10-24<br />.value_smooth:   -11.3187727","date: 1919-10-24<br />.value_smooth:    -7.1302624","date: 1920-10-24<br />.value_smooth:    -1.5831220","date: 1921-10-24<br />.value_smooth:     5.3809937","date: 1922-10-24<br />.value_smooth:    13.8204301","date: 1923-10-24<br />.value_smooth:    22.9105832","date: 1924-10-24<br />.value_smooth:    31.9718998","date: 1925-10-24<br />.value_smooth:    41.2803016","date: 1926-10-24<br />.value_smooth:    51.1117102","date: 1927-10-24<br />.value_smooth:    61.7420472","date: 1928-10-24<br />.value_smooth:    73.4472344","date: 1929-10-24<br />.value_smooth:    86.5031933","date: 1930-10-24<br />.value_smooth:   101.1858456","date: 1931-10-24<br />.value_smooth:   117.7711130","date: 1932-10-24<br />.value_smooth:   136.5349172","date: 1933-10-24<br />.value_smooth:   157.7531797","date: 1934-10-24<br />.value_smooth:   181.7018223","date: 1935-10-24<br />.value_smooth:   206.2725863","date: 1936-10-24<br />.value_smooth:   229.7335738","date: 1937-10-24<br />.value_smooth:   252.9252469","date: 1938-10-24<br />.value_smooth:   276.6880678","date: 1939-10-24<br />.value_smooth:   301.8624989","date: 1940-10-24<br />.value_smooth:   329.2890024","date: 1941-10-24<br />.value_smooth:   359.8080405","date: 1942-10-24<br />.value_smooth:   394.2600755","date: 1943-10-24<br />.value_smooth:   433.4855696","date: 1944-10-24<br />.value_smooth:   478.3249851","date: 1945-10-24<br />.value_smooth:   529.6187842","date: 1946-10-24<br />.value_smooth:   585.6451822","date: 1947-10-24<br />.value_smooth:   644.2171574","date: 1948-10-24<br />.value_smooth:   705.4773165","date: 1949-10-24<br />.value_smooth:   769.5682660","date: 1950-10-24<br />.value_smooth:   836.6326127","date: 1951-10-24<br />.value_smooth:   906.8129632","date: 1952-10-24<br />.value_smooth:   980.2519242","date: 1953-10-24<br />.value_smooth:  1057.0921024","date: 1954-10-24<br />.value_smooth:  1137.4761043","date: 1955-10-24<br />.value_smooth:  1221.5465368","date: 1956-10-24<br />.value_smooth:  1309.4460064","date: 1957-10-24<br />.value_smooth:  1401.3171199","date: 1958-10-24<br />.value_smooth:  1500.5060724","date: 1959-10-24<br />.value_smooth:  1609.0540044","date: 1960-10-24<br />.value_smooth:  1725.1459400","date: 1961-10-24<br />.value_smooth:  1846.9669034","date: 1962-10-24<br />.value_smooth:  1972.7019189","date: 1963-10-24<br />.value_smooth:  2100.5360105","date: 1964-10-24<br />.value_smooth:  2228.6542026","date: 1965-10-24<br />.value_smooth:  2355.2415192","date: 1966-10-24<br />.value_smooth:  2478.4829846","date: 1967-10-24<br />.value_smooth:  2596.5636230","date: 1968-10-24<br />.value_smooth:  2707.6684586","date: 1969-10-24<br />.value_smooth:  2815.5364006","date: 1970-10-24<br />.value_smooth:  2924.9169099","date: 1971-10-24<br />.value_smooth:  3035.5108381","date: 1972-10-24<br />.value_smooth:  3147.0190367","date: 1973-10-24<br />.value_smooth:  3259.1423574","date: 1974-10-24<br />.value_smooth:  3371.5816518","date: 1975-10-24<br />.value_smooth:  3484.0377713","date: 1976-10-24<br />.value_smooth:  3596.2115677","date: 1977-10-24<br />.value_smooth:  3707.8038925","date: 1978-10-24<br />.value_smooth:  3818.5155972","date: 1979-10-24<br />.value_smooth:  3928.0475336","date: 1980-10-24<br />.value_smooth:  4036.1005532","date: 1981-10-24<br />.value_smooth:  4142.7311543","date: 1982-10-24<br />.value_smooth:  4248.2807386","date: 1983-10-24<br />.value_smooth:  4352.8775124","date: 1984-10-24<br />.value_smooth:  4456.6496818","date: 1985-10-24<br />.value_smooth:  4559.7254532","date: 1986-10-24<br />.value_smooth:  4662.2330328","date: 1987-10-24<br />.value_smooth:  4764.3006268","date: 1988-10-24<br />.value_smooth:  4866.0564415","date: 1989-10-24<br />.value_smooth:  4967.6286832","date: 1990-10-24<br />.value_smooth:  5069.1455581","date: 1991-10-24<br />.value_smooth:  5170.7352725"],"type":"scatter","mode":"lines","line":{"width":3.77952755905512,"color":"rgba(51,102,255,1)","dash":"solid"},"hoveron":"points","showlegend":false,"xaxis":"x2","yaxis":"y2","hoverinfo":"text","frame":null},{"x":["1912-10-24","1913-10-24","1914-10-24","1915-10-24","1916-10-24","1917-10-24","1918-10-24","1919-10-24","1920-10-24","1921-10-24","1922-10-24","1923-10-24","1924-10-24","1925-10-24","1926-10-24","1927-10-24","1928-10-24","1929-10-24","1930-10-24","1931-10-24","1932-10-24","1933-10-24","1934-10-24","1935-10-24","1936-10-24","1937-10-24","1938-10-24","1939-10-24","1940-10-24","1941-10-24","1942-10-24","1943-10-24","1944-10-24","1945-10-24","1946-10-24","1947-10-24","1948-10-24","1949-10-24","1950-10-24","1951-10-24","1952-10-24","1953-10-24","1954-10-24","1955-10-24","1956-10-24","1957-10-24","1958-10-24","1959-10-24","1960-10-24","1961-10-24","1962-10-24","1963-10-24","1964-10-24","1965-10-24","1966-10-24","1967-10-24","1968-10-24","1969-10-24","1970-10-24","1971-10-24","1972-10-24","1973-10-24","1974-10-24","1975-10-24","1976-10-24","1977-10-24","1978-10-24","1979-10-24","1980-10-24","1981-10-24","1982-10-24","1983-10-24","1984-10-24","1985-10-24","1986-10-24","1987-10-24","1988-10-24","1989-10-24","1990-10-24","1991-10-24","1992-10-24","1993-10-24","1994-10-24","1995-10-24","1996-10-24","1997-10-24","1998-10-24","1999-10-24","2000-10-24","2001-10-24","2002-10-24","2003-10-24","2004-10-24","2005-10-24","2006-10-24","2007-10-24","2008-10-24","2009-10-24","2010-10-24","2011-10-24","2012-10-24","2013-10-24","2014-10-24","2015-10-24","2016-10-24","2017-10-24","2018-10-24","2019-10-24","2020-10-24","2021-10-24","2022-10-24"],"y":[36.8006423760324,25.7509568030723,15.7211101807169,6.73602110020676,-1.179391847218,-8.00021007031703,-13.70151497785,-18.2583879785767,-21.6459104812568,-23.83916389465,-24.8132296275159,-24.5431890886144,-23.0041236867051,-20.1711148305477,-15.9394612532705,-10.2627296112058,-3.20340319792009,5.17603469302055,14.8131007680499,25.6453117336016,37.6101842961096,50.6452351620076,64.6879810377293,79.6759386297086,95.5466246443792,112.237555788175,129.686248767529,147.830220288877,167.693804545176,190.1549292699,214.931501480254,241.741428193442,270.302616426669,300.33297319714,331.55040552206,363.672820418633,396.418124904064,429.504225995558,462.64903071032,495.570446065554,527.986379078464,559.614736766257,594.280905405347,635.172412270852,681.050375878023,730.675914742113,782.810147378374,836.21419230206,889.649168028423,941.876193072715,991.65638595019,1037.7508651761,1078.92074926569,1113.92715673423,1141.53120609696,1160.49401586913,1172.74244299575,1181.23930382666,1186.29965598437,1188.23855709142,1187.37106477033,1184.01223664362,1178.47713033381,1171.08080346342,1162.13831365499,1151.96471853103,1140.87507571406,1129.1844428266,1117.20787749119,1105.26043733034,1091.21683147336,1073.14792222605,1051.66299697568,1027.37134310953,1000.88224801487,972.80499907897,943.74888368911,914.323189232561,885.137203096595,856.800212668487,829.92150533551,805.110368484937,782.976089504042,764.127955780098,746.60757663086,728.154160439659,708.923393192138,689.07096087394,668.752549470706,648.12384496808,627.340533351703,606.558300607217,585.932832720266,565.61981567649,545.774935461533,526.553878061037,508.112329460643,490.605975645996,474.009549056173,458.144953974302,442.938634152091,428.31703334125,414.206595293491,400.533763760521,387.224982494052,374.206695245793,361.405345767453,348.747377810744,336.159235127375,323.567361469055,310.898200587494],"text":["date: 1912-10-24<br />.value_smooth:    36.8006424","date: 1913-10-24<br />.value_smooth:    25.7509568","date: 1914-10-24<br />.value_smooth:    15.7211102","date: 1915-10-24<br />.value_smooth:     6.7360211","date: 1916-10-24<br />.value_smooth:    -1.1793918","date: 1917-10-24<br />.value_smooth:    -8.0002101","date: 1918-10-24<br />.value_smooth:   -13.7015150","date: 1919-10-24<br />.value_smooth:   -18.2583880","date: 1920-10-24<br />.value_smooth:   -21.6459105","date: 1921-10-24<br />.value_smooth:   -23.8391639","date: 1922-10-24<br />.value_smooth:   -24.8132296","date: 1923-10-24<br />.value_smooth:   -24.5431891","date: 1924-10-24<br />.value_smooth:   -23.0041237","date: 1925-10-24<br />.value_smooth:   -20.1711148","date: 1926-10-24<br />.value_smooth:   -15.9394613","date: 1927-10-24<br />.value_smooth:   -10.2627296","date: 1928-10-24<br />.value_smooth:    -3.2034032","date: 1929-10-24<br />.value_smooth:     5.1760347","date: 1930-10-24<br />.value_smooth:    14.8131008","date: 1931-10-24<br />.value_smooth:    25.6453117","date: 1932-10-24<br />.value_smooth:    37.6101843","date: 1933-10-24<br />.value_smooth:    50.6452352","date: 1934-10-24<br />.value_smooth:    64.6879810","date: 1935-10-24<br />.value_smooth:    79.6759386","date: 1936-10-24<br />.value_smooth:    95.5466246","date: 1937-10-24<br />.value_smooth:   112.2375558","date: 1938-10-24<br />.value_smooth:   129.6862488","date: 1939-10-24<br />.value_smooth:   147.8302203","date: 1940-10-24<br />.value_smooth:   167.6938045","date: 1941-10-24<br />.value_smooth:   190.1549293","date: 1942-10-24<br />.value_smooth:   214.9315015","date: 1943-10-24<br />.value_smooth:   241.7414282","date: 1944-10-24<br />.value_smooth:   270.3026164","date: 1945-10-24<br />.value_smooth:   300.3329732","date: 1946-10-24<br />.value_smooth:   331.5504055","date: 1947-10-24<br />.value_smooth:   363.6728204","date: 1948-10-24<br />.value_smooth:   396.4181249","date: 1949-10-24<br />.value_smooth:   429.5042260","date: 1950-10-24<br />.value_smooth:   462.6490307","date: 1951-10-24<br />.value_smooth:   495.5704461","date: 1952-10-24<br />.value_smooth:   527.9863791","date: 1953-10-24<br />.value_smooth:   559.6147368","date: 1954-10-24<br />.value_smooth:   594.2809054","date: 1955-10-24<br />.value_smooth:   635.1724123","date: 1956-10-24<br />.value_smooth:   681.0503759","date: 1957-10-24<br />.value_smooth:   730.6759147","date: 1958-10-24<br />.value_smooth:   782.8101474","date: 1959-10-24<br />.value_smooth:   836.2141923","date: 1960-10-24<br />.value_smooth:   889.6491680","date: 1961-10-24<br />.value_smooth:   941.8761931","date: 1962-10-24<br />.value_smooth:   991.6563860","date: 1963-10-24<br />.value_smooth:  1037.7508652","date: 1964-10-24<br />.value_smooth:  1078.9207493","date: 1965-10-24<br />.value_smooth:  1113.9271567","date: 1966-10-24<br />.value_smooth:  1141.5312061","date: 1967-10-24<br />.value_smooth:  1160.4940159","date: 1968-10-24<br />.value_smooth:  1172.7424430","date: 1969-10-24<br />.value_smooth:  1181.2393038","date: 1970-10-24<br />.value_smooth:  1186.2996560","date: 1971-10-24<br />.value_smooth:  1188.2385571","date: 1972-10-24<br />.value_smooth:  1187.3710648","date: 1973-10-24<br />.value_smooth:  1184.0122366","date: 1974-10-24<br />.value_smooth:  1178.4771303","date: 1975-10-24<br />.value_smooth:  1171.0808035","date: 1976-10-24<br />.value_smooth:  1162.1383137","date: 1977-10-24<br />.value_smooth:  1151.9647185","date: 1978-10-24<br />.value_smooth:  1140.8750757","date: 1979-10-24<br />.value_smooth:  1129.1844428","date: 1980-10-24<br />.value_smooth:  1117.2078775","date: 1981-10-24<br />.value_smooth:  1105.2604373","date: 1982-10-24<br />.value_smooth:  1091.2168315","date: 1983-10-24<br />.value_smooth:  1073.1479222","date: 1984-10-24<br />.value_smooth:  1051.6629970","date: 1985-10-24<br />.value_smooth:  1027.3713431","date: 1986-10-24<br />.value_smooth:  1000.8822480","date: 1987-10-24<br />.value_smooth:   972.8049991","date: 1988-10-24<br />.value_smooth:   943.7488837","date: 1989-10-24<br />.value_smooth:   914.3231892","date: 1990-10-24<br />.value_smooth:   885.1372031","date: 1991-10-24<br />.value_smooth:   856.8002127","date: 1992-10-24<br />.value_smooth:   829.9215053","date: 1993-10-24<br />.value_smooth:   805.1103685","date: 1994-10-24<br />.value_smooth:   782.9760895","date: 1995-10-24<br />.value_smooth:   764.1279558","date: 1996-10-24<br />.value_smooth:   746.6075766","date: 1997-10-24<br />.value_smooth:   728.1541604","date: 1998-10-24<br />.value_smooth:   708.9233932","date: 1999-10-24<br />.value_smooth:   689.0709609","date: 2000-10-24<br />.value_smooth:   668.7525495","date: 2001-10-24<br />.value_smooth:   648.1238450","date: 2002-10-24<br />.value_smooth:   627.3405334","date: 2003-10-24<br />.value_smooth:   606.5583006","date: 2004-10-24<br />.value_smooth:   585.9328327","date: 2005-10-24<br />.value_smooth:   565.6198157","date: 2006-10-24<br />.value_smooth:   545.7749355","date: 2007-10-24<br />.value_smooth:   526.5538781","date: 2008-10-24<br />.value_smooth:   508.1123295","date: 2009-10-24<br />.value_smooth:   490.6059756","date: 2010-10-24<br />.value_smooth:   474.0095491","date: 2011-10-24<br />.value_smooth:   458.1449540","date: 2012-10-24<br />.value_smooth:   442.9386342","date: 2013-10-24<br />.value_smooth:   428.3170333","date: 2014-10-24<br />.value_smooth:   414.2065953","date: 2015-10-24<br />.value_smooth:   400.5337638","date: 2016-10-24<br />.value_smooth:   387.2249825","date: 2017-10-24<br />.value_smooth:   374.2066952","date: 2018-10-24<br />.value_smooth:   361.4053458","date: 2019-10-24<br />.value_smooth:   348.7473778","date: 2020-10-24<br />.value_smooth:   336.1592351","date: 2021-10-24<br />.value_smooth:   323.5673615","date: 2022-10-24<br />.value_smooth:   310.8982006"],"type":"scatter","mode":"lines","line":{"width":3.77952755905512,"color":"rgba(51,102,255,1)","dash":"solid"},"hoveron":"points","showlegend":false,"xaxis":"x3","yaxis":"y3","hoverinfo":"text","frame":null},{"x":["1938-10-24","1939-10-24","1940-10-24","1941-10-24","1942-10-24","1943-10-24","1944-10-24","1945-10-24","1946-10-24","1947-10-24","1948-10-24","1949-10-24","1950-10-24","1951-10-24","1952-10-24","1953-10-24","1954-10-24","1955-10-24","1956-10-24","1957-10-24","1958-10-24","1959-10-24","1960-10-24","1961-10-24","1962-10-24","1963-10-24","1964-10-24","1965-10-24","1966-10-24","1967-10-24","1968-10-24","1969-10-24","1970-10-24","1971-10-24","1972-10-24","1973-10-24","1974-10-24","1975-10-24","1976-10-24","1977-10-24","1978-10-24","1979-10-24","1980-10-24","1981-10-24","1982-10-24","1983-10-24","1984-10-24","1985-10-24","1986-10-24","1987-10-24","1988-10-24","1989-10-24","1990-10-24","1991-10-24","1992-10-24","1993-10-24","1994-10-24","1995-10-24","1996-10-24","1997-10-24","1998-10-24","1999-10-24","2000-10-24","2001-10-24","2002-10-24","2003-10-24","2004-10-24","2005-10-24","2006-10-24","2007-10-24","2008-10-24","2009-10-24","2010-10-24","2011-10-24","2012-10-24","2013-10-24","2014-10-24","2015-10-24","2016-10-24","2017-10-24","2018-10-24","2019-10-24","2020-10-24","2021-10-24","2022-10-24"],"y":[72.0147453464205,51.5137776694759,32.7744558942617,15.8360784891033,0.737943922326306,-12.4806493377439,-23.7804028227819,-33.1220180644622,-40.4661965944593,-45.7736399444479,-49.0050496461023,-50.0006308020886,-48.6993752963861,-45.2101648326394,-39.6418811144931,-32.103405845592,-22.7036207295805,-11.5514074701034,1.24435222919468,15.5747766646691,31.3309841326753,48.4040929295686,67.9515786384053,90.9244247576516,116.904011456778,145.471718905256,176.208927272555,208.697016728146,242.5173674415,277.251359582088,312.480373319379,347.785788822844,382.748986261955,420.295844809634,462.841885158027,509.202949755445,558.194881050195,608.633521490587,659.334713524928,709.114299601527,756.788122168694,801.172023674736,841.081846567962,877.886072912951,913.76332349617,948.744648848762,982.861099501868,1016.14372598663,1048.62357883419,1080.33170857569,1111.29916574227,1141.55700086507,1171.13626447523,1200.0680071039,1227.97272802695,1254.61512503558,1280.24254643395,1305.1023405262,1329.4418556165,1353.50844000898,1377.5494420078,1401.81220991711,1426.54409204106,1451.9924366838,1477.67472519328,1502.97120305599,1527.92336478387,1552.57270488886,1576.9607178829,1601.12889827792,1625.11874058588,1648.9717393187,1672.72938898833,1696.43318410671,1720.12461918578,1743.7700159297,1767.29935835513,1790.69963444275,1813.95783217322,1837.06093952722,1859.99594448543,1882.74983502852,1905.30959913717,1927.66222479205,1949.79469997384],"text":["date: 1938-10-24<br />.value_smooth:    72.0147453","date: 1939-10-24<br />.value_smooth:    51.5137777","date: 1940-10-24<br />.value_smooth:    32.7744559","date: 1941-10-24<br />.value_smooth:    15.8360785","date: 1942-10-24<br />.value_smooth:     0.7379439","date: 1943-10-24<br />.value_smooth:   -12.4806493","date: 1944-10-24<br />.value_smooth:   -23.7804028","date: 1945-10-24<br />.value_smooth:   -33.1220181","date: 1946-10-24<br />.value_smooth:   -40.4661966","date: 1947-10-24<br />.value_smooth:   -45.7736399","date: 1948-10-24<br />.value_smooth:   -49.0050496","date: 1949-10-24<br />.value_smooth:   -50.0006308","date: 1950-10-24<br />.value_smooth:   -48.6993753","date: 1951-10-24<br />.value_smooth:   -45.2101648","date: 1952-10-24<br />.value_smooth:   -39.6418811","date: 1953-10-24<br />.value_smooth:   -32.1034058","date: 1954-10-24<br />.value_smooth:   -22.7036207","date: 1955-10-24<br />.value_smooth:   -11.5514075","date: 1956-10-24<br />.value_smooth:     1.2443522","date: 1957-10-24<br />.value_smooth:    15.5747767","date: 1958-10-24<br />.value_smooth:    31.3309841","date: 1959-10-24<br />.value_smooth:    48.4040929","date: 1960-10-24<br />.value_smooth:    67.9515786","date: 1961-10-24<br />.value_smooth:    90.9244248","date: 1962-10-24<br />.value_smooth:   116.9040115","date: 1963-10-24<br />.value_smooth:   145.4717189","date: 1964-10-24<br />.value_smooth:   176.2089273","date: 1965-10-24<br />.value_smooth:   208.6970167","date: 1966-10-24<br />.value_smooth:   242.5173674","date: 1967-10-24<br />.value_smooth:   277.2513596","date: 1968-10-24<br />.value_smooth:   312.4803733","date: 1969-10-24<br />.value_smooth:   347.7857888","date: 1970-10-24<br />.value_smooth:   382.7489863","date: 1971-10-24<br />.value_smooth:   420.2958448","date: 1972-10-24<br />.value_smooth:   462.8418852","date: 1973-10-24<br />.value_smooth:   509.2029498","date: 1974-10-24<br />.value_smooth:   558.1948811","date: 1975-10-24<br />.value_smooth:   608.6335215","date: 1976-10-24<br />.value_smooth:   659.3347135","date: 1977-10-24<br />.value_smooth:   709.1142996","date: 1978-10-24<br />.value_smooth:   756.7881222","date: 1979-10-24<br />.value_smooth:   801.1720237","date: 1980-10-24<br />.value_smooth:   841.0818466","date: 1981-10-24<br />.value_smooth:   877.8860729","date: 1982-10-24<br />.value_smooth:   913.7633235","date: 1983-10-24<br />.value_smooth:   948.7446488","date: 1984-10-24<br />.value_smooth:   982.8610995","date: 1985-10-24<br />.value_smooth:  1016.1437260","date: 1986-10-24<br />.value_smooth:  1048.6235788","date: 1987-10-24<br />.value_smooth:  1080.3317086","date: 1988-10-24<br />.value_smooth:  1111.2991657","date: 1989-10-24<br />.value_smooth:  1141.5570009","date: 1990-10-24<br />.value_smooth:  1171.1362645","date: 1991-10-24<br />.value_smooth:  1200.0680071","date: 1992-10-24<br />.value_smooth:  1227.9727280","date: 1993-10-24<br />.value_smooth:  1254.6151250","date: 1994-10-24<br />.value_smooth:  1280.2425464","date: 1995-10-24<br />.value_smooth:  1305.1023405","date: 1996-10-24<br />.value_smooth:  1329.4418556","date: 1997-10-24<br />.value_smooth:  1353.5084400","date: 1998-10-24<br />.value_smooth:  1377.5494420","date: 1999-10-24<br />.value_smooth:  1401.8122099","date: 2000-10-24<br />.value_smooth:  1426.5440920","date: 2001-10-24<br />.value_smooth:  1451.9924367","date: 2002-10-24<br />.value_smooth:  1477.6747252","date: 2003-10-24<br />.value_smooth:  1502.9712031","date: 2004-10-24<br />.value_smooth:  1527.9233648","date: 2005-10-24<br />.value_smooth:  1552.5727049","date: 2006-10-24<br />.value_smooth:  1576.9607179","date: 2007-10-24<br />.value_smooth:  1601.1288983","date: 2008-10-24<br />.value_smooth:  1625.1187406","date: 2009-10-24<br />.value_smooth:  1648.9717393","date: 2010-10-24<br />.value_smooth:  1672.7293890","date: 2011-10-24<br />.value_smooth:  1696.4331841","date: 2012-10-24<br />.value_smooth:  1720.1246192","date: 2013-10-24<br />.value_smooth:  1743.7700159","date: 2014-10-24<br />.value_smooth:  1767.2993584","date: 2015-10-24<br />.value_smooth:  1790.6996344","date: 2016-10-24<br />.value_smooth:  1813.9578322","date: 2017-10-24<br />.value_smooth:  1837.0609395","date: 2018-10-24<br />.value_smooth:  1859.9959445","date: 2019-10-24<br />.value_smooth:  1882.7498350","date: 2020-10-24<br />.value_smooth:  1905.3095991","date: 2021-10-24<br />.value_smooth:  1927.6622248","date: 2022-10-24<br />.value_smooth:  1949.7947000"],"type":"scatter","mode":"lines","line":{"width":3.77952755905512,"color":"rgba(51,102,255,1)","dash":"solid"},"hoveron":"points","showlegend":false,"xaxis":"x4","yaxis":"y4","hoverinfo":"text","frame":null},{"x":["1945-10-24","1946-10-24","1947-10-24","1948-10-24","1949-10-24","1950-10-24","1951-10-24","1952-10-24","1953-10-24","1954-10-24","1955-10-24","1956-10-24","1957-10-24","1958-10-24","1959-10-24","1960-10-24","1961-10-24","1962-10-24","1963-10-24","1964-10-24","1965-10-24","1966-10-24","1967-10-24","1968-10-24","1969-10-24","1970-10-24","1971-10-24","1972-10-24","1973-10-24","1974-10-24","1975-10-24","1976-10-24","1977-10-24","1978-10-24","1979-10-24","1980-10-24","1981-10-24","1982-10-24","1983-10-24","1984-10-24","1985-10-24","1986-10-24","1987-10-24","1988-10-24","1989-10-24","1990-10-24","1991-10-24","1992-10-24","1993-10-24","1994-10-24","1995-10-24","1996-10-24","1997-10-24","1998-10-24","1999-10-24","2000-10-24","2001-10-24","2002-10-24","2003-10-24","2004-10-24","2005-10-24","2006-10-24","2007-10-24","2008-10-24","2009-10-24","2010-10-24","2011-10-24","2012-10-24","2013-10-24","2014-10-24","2015-10-24","2016-10-24","2017-10-24","2018-10-24","2019-10-24","2020-10-24","2021-10-24","2022-10-24"],"y":[75.9267554630569,102.141066152276,129.822603552521,158.905823678477,189.325182544826,221.015136166252,253.91014055744,287.944651733072,323.053125707833,359.170018496406,396.231880759493,434.298356494732,473.488541722207,513.921532462004,555.716424734208,598.992314558905,643.86829795618,690.463470946118,738.896929548805,789.287769784326,839.863789555914,889.253491663546,938.177029472422,987.354556347743,1037.50622565471,1089.35219075852,1143.61260502438,1201.00762181748,1262.25739450304,1328.08207644624,1395.0771352332,1460.347939614,1525.37949469974,1591.65680560154,1660.66487743049,1733.88871529773,1812.81332431434,1898.92370959145,1993.70487624017,2085.82029037374,2166.4532295716,2240.86841614415,2314.33057240177,2392.10442065488,2479.45468321385,2581.64608238909,2703.943340491,2851.61117982997,3029.91432271639,3236.88698955025,3464.97818603514,3711.07481243695,3972.06376902155,4244.8319560548,4526.26627380257,4813.25362253074,5102.68090250518,5391.43501399175,5676.40285725633,5962.80589586785,6257.80811836083,6561.21021244943,6872.81286584786,7192.4167662703,7519.82260143094,7854.83105904398,8197.2428268236,8546.85859248399,8903.47904373934,9266.72975888759,9636.4734446594,10012.866481418,10396.0652495268,10786.226129349,11183.5055012479,11588.0597455868,12000.045242729,12419.6183730378],"text":["date: 1945-10-24<br />.value_smooth:    75.9267555","date: 1946-10-24<br />.value_smooth:   102.1410662","date: 1947-10-24<br />.value_smooth:   129.8226036","date: 1948-10-24<br />.value_smooth:   158.9058237","date: 1949-10-24<br />.value_smooth:   189.3251825","date: 1950-10-24<br />.value_smooth:   221.0151362","date: 1951-10-24<br />.value_smooth:   253.9101406","date: 1952-10-24<br />.value_smooth:   287.9446517","date: 1953-10-24<br />.value_smooth:   323.0531257","date: 1954-10-24<br />.value_smooth:   359.1700185","date: 1955-10-24<br />.value_smooth:   396.2318808","date: 1956-10-24<br />.value_smooth:   434.2983565","date: 1957-10-24<br />.value_smooth:   473.4885417","date: 1958-10-24<br />.value_smooth:   513.9215325","date: 1959-10-24<br />.value_smooth:   555.7164247","date: 1960-10-24<br />.value_smooth:   598.9923146","date: 1961-10-24<br />.value_smooth:   643.8682980","date: 1962-10-24<br />.value_smooth:   690.4634709","date: 1963-10-24<br />.value_smooth:   738.8969295","date: 1964-10-24<br />.value_smooth:   789.2877698","date: 1965-10-24<br />.value_smooth:   839.8637896","date: 1966-10-24<br />.value_smooth:   889.2534917","date: 1967-10-24<br />.value_smooth:   938.1770295","date: 1968-10-24<br />.value_smooth:   987.3545563","date: 1969-10-24<br />.value_smooth:  1037.5062257","date: 1970-10-24<br />.value_smooth:  1089.3521908","date: 1971-10-24<br />.value_smooth:  1143.6126050","date: 1972-10-24<br />.value_smooth:  1201.0076218","date: 1973-10-24<br />.value_smooth:  1262.2573945","date: 1974-10-24<br />.value_smooth:  1328.0820764","date: 1975-10-24<br />.value_smooth:  1395.0771352","date: 1976-10-24<br />.value_smooth:  1460.3479396","date: 1977-10-24<br />.value_smooth:  1525.3794947","date: 1978-10-24<br />.value_smooth:  1591.6568056","date: 1979-10-24<br />.value_smooth:  1660.6648774","date: 1980-10-24<br />.value_smooth:  1733.8887153","date: 1981-10-24<br />.value_smooth:  1812.8133243","date: 1982-10-24<br />.value_smooth:  1898.9237096","date: 1983-10-24<br />.value_smooth:  1993.7048762","date: 1984-10-24<br />.value_smooth:  2085.8202904","date: 1985-10-24<br />.value_smooth:  2166.4532296","date: 1986-10-24<br />.value_smooth:  2240.8684161","date: 1987-10-24<br />.value_smooth:  2314.3305724","date: 1988-10-24<br />.value_smooth:  2392.1044207","date: 1989-10-24<br />.value_smooth:  2479.4546832","date: 1990-10-24<br />.value_smooth:  2581.6460824","date: 1991-10-24<br />.value_smooth:  2703.9433405","date: 1992-10-24<br />.value_smooth:  2851.6111798","date: 1993-10-24<br />.value_smooth:  3029.9143227","date: 1994-10-24<br />.value_smooth:  3236.8869896","date: 1995-10-24<br />.value_smooth:  3464.9781860","date: 1996-10-24<br />.value_smooth:  3711.0748124","date: 1997-10-24<br />.value_smooth:  3972.0637690","date: 1998-10-24<br />.value_smooth:  4244.8319561","date: 1999-10-24<br />.value_smooth:  4526.2662738","date: 2000-10-24<br />.value_smooth:  4813.2536225","date: 2001-10-24<br />.value_smooth:  5102.6809025","date: 2002-10-24<br />.value_smooth:  5391.4350140","date: 2003-10-24<br />.value_smooth:  5676.4028573","date: 2004-10-24<br />.value_smooth:  5962.8058959","date: 2005-10-24<br />.value_smooth:  6257.8081184","date: 2006-10-24<br />.value_smooth:  6561.2102124","date: 2007-10-24<br />.value_smooth:  6872.8128658","date: 2008-10-24<br />.value_smooth:  7192.4167663","date: 2009-10-24<br />.value_smooth:  7519.8226014","date: 2010-10-24<br />.value_smooth:  7854.8310590","date: 2011-10-24<br />.value_smooth:  8197.2428268","date: 2012-10-24<br />.value_smooth:  8546.8585925","date: 2013-10-24<br />.value_smooth:  8903.4790437","date: 2014-10-24<br />.value_smooth:  9266.7297589","date: 2015-10-24<br />.value_smooth:  9636.4734447","date: 2016-10-24<br />.value_smooth: 10012.8664814","date: 2017-10-24<br />.value_smooth: 10396.0652495","date: 2018-10-24<br />.value_smooth: 10786.2261293","date: 2019-10-24<br />.value_smooth: 11183.5055012","date: 2020-10-24<br />.value_smooth: 11588.0597456","date: 2021-10-24<br />.value_smooth: 12000.0452427","date: 2022-10-24<br />.value_smooth: 12419.6183730"],"type":"scatter","mode":"lines","line":{"width":3.77952755905512,"color":"rgba(51,102,255,1)","dash":"solid"},"hoveron":"points","showlegend":false,"xaxis":"x5","yaxis":"y5","hoverinfo":"text","frame":null},{"x":["1989-10-24","1990-10-24","1991-10-24","1992-10-24","1993-10-24","1994-10-24","1995-10-24","1996-10-24","1997-10-24","1998-10-24","1999-10-24","2000-10-24","2001-10-24","2002-10-24","2003-10-24","2004-10-24","2005-10-24","2006-10-24","2007-10-24","2008-10-24","2009-10-24","2010-10-24","2011-10-24","2012-10-24","2013-10-24","2014-10-24","2015-10-24","2016-10-24","2017-10-24","2018-10-24","2019-10-24","2020-10-24","2021-10-24","2022-10-24"],"y":[1750.33099774673,1706.33891247843,1665.9912139599,1629.47912546813,1596.99387028012,1568.95945566753,1545.17077308169,1524.87537206917,1507.3208021765,1494.4538911056,1487.14067117482,1483.00749885538,1479.6807306185,1482.7731517374,1493.92349164916,1504.79702307086,1507.05901871955,1497.18745095891,1480.06364386857,1459.95789773951,1441.14051286273,1427.88178952921,1419.71991523189,1413.43457867736,1409.15833197709,1407.02372724251,1406.44736595797,1407.11773274917,1409.57520443962,1414.36015785286,1421.63589398151,1431.11880291976,1442.67889514346,1456.18618112848],"text":["date: 1989-10-24<br />.value_smooth:  1750.3309977","date: 1990-10-24<br />.value_smooth:  1706.3389125","date: 1991-10-24<br />.value_smooth:  1665.9912140","date: 1992-10-24<br />.value_smooth:  1629.4791255","date: 1993-10-24<br />.value_smooth:  1596.9938703","date: 1994-10-24<br />.value_smooth:  1568.9594557","date: 1995-10-24<br />.value_smooth:  1545.1707731","date: 1996-10-24<br />.value_smooth:  1524.8753721","date: 1997-10-24<br />.value_smooth:  1507.3208022","date: 1998-10-24<br />.value_smooth:  1494.4538911","date: 1999-10-24<br />.value_smooth:  1487.1406712","date: 2000-10-24<br />.value_smooth:  1483.0074989","date: 2001-10-24<br />.value_smooth:  1479.6807306","date: 2002-10-24<br />.value_smooth:  1482.7731517","date: 2003-10-24<br />.value_smooth:  1493.9234916","date: 2004-10-24<br />.value_smooth:  1504.7970231","date: 2005-10-24<br />.value_smooth:  1507.0590187","date: 2006-10-24<br />.value_smooth:  1497.1874510","date: 2007-10-24<br />.value_smooth:  1480.0636439","date: 2008-10-24<br />.value_smooth:  1459.9578977","date: 2009-10-24<br />.value_smooth:  1441.1405129","date: 2010-10-24<br />.value_smooth:  1427.8817895","date: 2011-10-24<br />.value_smooth:  1419.7199152","date: 2012-10-24<br />.value_smooth:  1413.4345787","date: 2013-10-24<br />.value_smooth:  1409.1583320","date: 2014-10-24<br />.value_smooth:  1407.0237272","date: 2015-10-24<br />.value_smooth:  1406.4473660","date: 2016-10-24<br />.value_smooth:  1407.1177327","date: 2017-10-24<br />.value_smooth:  1409.5752044","date: 2018-10-24<br />.value_smooth:  1414.3601579","date: 2019-10-24<br />.value_smooth:  1421.6358940","date: 2020-10-24<br />.value_smooth:  1431.1188029","date: 2021-10-24<br />.value_smooth:  1442.6788951","date: 2022-10-24<br />.value_smooth:  1456.1861811"],"type":"scatter","mode":"lines","line":{"width":3.77952755905512,"color":"rgba(51,102,255,1)","dash":"solid"},"hoveron":"points","showlegend":false,"xaxis":"x6","yaxis":"y6","hoverinfo":"text","frame":null}],"layout":{"margin":{"t":55.4520547945205,"r":7.30593607305936,"b":25.5707762557078,"l":34.337899543379},"plot_bgcolor":"rgba(255,255,255,1)","paper_bgcolor":"rgba(255,255,255,1)","font":{"color":"rgba(44,62,80,1)","family":"","size":14.6118721461187},"title":{"text":"Time Series Plot","font":{"color":"rgba(44,62,80,1)","family":"","size":17.5342465753425},"x":0,"xref":"paper"},"xaxis":{"domain":[0,0.414330058057811],"automargin":true,"type":"date","autorange":true,"range":["1877-11-29","2029-09-17"],"tickmode":"auto","ticktext":["1900","1950","2000"],"tickvals":[-25567,-7305,10957],"categoryorder":"array","categoryarray":["1900","1950","2000"],"nticks":null,"ticks":"outside","tickcolor":"rgba(204,204,204,1)","ticklen":3.65296803652968,"tickwidth":0,"showticklabels":true,"tickfont":{"color":"rgba(44,62,80,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(204,204,204,1)","gridwidth":0,"zeroline":false,"anchor":"y","title":"","hoverformat":".2f","rangeslider":{"type":"date"}},"yaxis":{"domain":[0.774428025263965,1],"automargin":true,"type":"linear","autorange":true,"range":[-115.305721659038,1469.37709540632],"tickmode":"auto","ticktext":["0","500","1000"],"tickvals":[0,500,1000],"categoryorder":"array","categoryarray":["0","500","1000"],"nticks":null,"ticks":"outside","tickcolor":"rgba(204,204,204,1)","ticklen":3.65296803652968,"tickwidth":0,"showticklabels":true,"tickfont":{"color":"rgba(44,62,80,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(204,204,204,1)","gridwidth":0,"zeroline":false,"anchor":"x","title":"","hoverformat":".2f"},"shapes":[{"type":"rect","fillcolor":"transparent","line":{"color":"rgba(44,62,80,1)","width":0,"linetype":"solid"},"yref":"paper","xref":"paper","x0":0,"x1":0.414330058057811,"y0":0.774428025263965,"y1":1},{"type":"rect","fillcolor":"rgba(44,62,80,1)","line":{"color":"rgba(44,62,80,1)","width":0,"linetype":"solid"},"yref":"paper","xref":"paper","x0":0,"x1":0.414330058057811,"y0":0,"y1":24.9730178497302,"yanchor":1,"ysizemode":"pixel"},{"type":"rect","fillcolor":"transparent","line":{"color":"rgba(44,62,80,1)","width":0,"linetype":"solid"},"yref":"paper","xref":"paper","x0":0.585669941942189,"x1":1,"y0":0.774428025263965,"y1":1},{"type":"rect","fillcolor":"rgba(44,62,80,1)","line":{"color":"rgba(44,62,80,1)","width":0,"linetype":"solid"},"yref":"paper","xref":"paper","x0":0.585669941942189,"x1":1,"y0":0,"y1":24.9730178497302,"yanchor":1,"ysizemode":"pixel"},{"type":"rect","fillcolor":"transparent","line":{"color":"rgba(44,62,80,1)","width":0,"linetype":"solid"},"yref":"paper","xref":"paper","x0":0,"x1":0.414330058057811,"y0":0.441094691930632,"y1":0.558905308069368},{"type":"rect","fillcolor":"rgba(44,62,80,1)","line":{"color":"rgba(44,62,80,1)","width":0,"linetype":"solid"},"yref":"paper","xref":"paper","x0":0,"x1":0.414330058057811,"y0":0,"y1":24.9730178497302,"yanchor":0.558905308069368,"ysizemode":"pixel"},{"type":"rect","fillcolor":"transparent","line":{"color":"rgba(44,62,80,1)","width":0,"linetype":"solid"},"yref":"paper","xref":"paper","x0":0.585669941942189,"x1":1,"y0":0.441094691930632,"y1":0.558905308069368},{"type":"rect","fillcolor":"rgba(44,62,80,1)","line":{"color":"rgba(44,62,80,1)","width":0,"linetype":"solid"},"yref":"paper","xref":"paper","x0":0.585669941942189,"x1":1,"y0":0,"y1":24.9730178497302,"yanchor":0.558905308069368,"ysizemode":"pixel"},{"type":"rect","fillcolor":"transparent","line":{"color":"rgba(44,62,80,1)","width":0,"linetype":"solid"},"yref":"paper","xref":"paper","x0":0,"x1":0.414330058057811,"y0":0,"y1":0.225571974736035},{"type":"rect","fillcolor":"rgba(44,62,80,1)","line":{"color":"rgba(44,62,80,1)","width":0,"linetype":"solid"},"yref":"paper","xref":"paper","x0":0,"x1":0.414330058057811,"y0":0,"y1":24.9730178497302,"yanchor":0.225571974736035,"ysizemode":"pixel"},{"type":"rect","fillcolor":"transparent","line":{"color":"rgba(44,62,80,1)","width":0,"linetype":"solid"},"yref":"paper","xref":"paper","x0":0.585669941942189,"x1":1,"y0":0,"y1":0.225571974736035},{"type":"rect","fillcolor":"rgba(44,62,80,1)","line":{"color":"rgba(44,62,80,1)","width":0,"linetype":"solid"},"yref":"paper","xref":"paper","x0":0.585669941942189,"x1":1,"y0":0,"y1":24.9730178497302,"yanchor":0.225571974736035,"ysizemode":"pixel"}],"annotations":[{"text":"ExxonMobil","x":0.207165029028906,"y":1,"showarrow":false,"ax":0,"ay":0,"font":{"color":"rgba(255,255,255,1)","family":"","size":11.689497716895},"xref":"paper","yref":"paper","textangle":-0,"xanchor":"center","yanchor":"bottom"},{"text":"Former Soviet Union","x":0.792834970971094,"y":1,"showarrow":false,"ax":0,"ay":0,"font":{"color":"rgba(255,255,255,1)","family":"","size":11.689497716895},"xref":"paper","yref":"paper","textangle":-0,"xanchor":"center","yanchor":"bottom"},{"text":"Chevron","x":0.207165029028906,"y":0.558905308069368,"showarrow":false,"ax":0,"ay":0,"font":{"color":"rgba(255,255,255,1)","family":"","size":11.689497716895},"xref":"paper","yref":"paper","textangle":-0,"xanchor":"center","yanchor":"bottom"},{"text":"Saudi Aramco","x":0.792834970971094,"y":0.558905308069368,"showarrow":false,"ax":0,"ay":0,"font":{"color":"rgba(255,255,255,1)","family":"","size":11.689497716895},"xref":"paper","yref":"paper","textangle":-0,"xanchor":"center","yanchor":"bottom"},{"text":"China (Coal)","x":0.207165029028906,"y":0.225571974736035,"showarrow":false,"ax":0,"ay":0,"font":{"color":"rgba(255,255,255,1)","family":"","size":11.689497716895},"xref":"paper","yref":"paper","textangle":-0,"xanchor":"center","yanchor":"bottom"},{"text":"Gazprom","x":0.792834970971094,"y":0.225571974736035,"showarrow":false,"ax":0,"ay":0,"font":{"color":"rgba(255,255,255,1)","family":"","size":11.689497716895},"xref":"paper","yref":"paper","textangle":-0,"xanchor":"center","yanchor":"bottom"}],"xaxis2":{"type":"date","autorange":true,"range":["1896-04-05","1996-05-11"],"tickmode":"auto","ticktext":["1900","1920","1940","1960","1980"],"tickvals":[-25567,-18263,-10958,-3653,3652],"categoryorder":"array","categoryarray":["1900","1920","1940","1960","1980"],"nticks":null,"ticks":"outside","tickcolor":"rgba(204,204,204,1)","ticklen":3.65296803652968,"tickwidth":0,"showticklabels":true,"tickfont":{"color":"rgba(44,62,80,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"domain":[0.585669941942189,1],"gridcolor":"rgba(204,204,204,1)","gridwidth":0,"zeroline":false,"anchor":"y2","title":"","hoverformat":".2f"},"yaxis2":{"type":"linear","autorange":true,"range":[-305.566312206964,6057.94106804664],"tickmode":"auto","ticktext":["0","2000","4000","6000"],"tickvals":[0,2000,4000,6000],"categoryorder":"array","categoryarray":["0","2000","4000","6000"],"nticks":null,"ticks":"outside","tickcolor":"rgba(204,204,204,1)","ticklen":3.65296803652968,"tickwidth":0,"showticklabels":true,"tickfont":{"color":"rgba(44,62,80,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"domain":[0.774428025263965,1],"gridcolor":"rgba(204,204,204,1)","gridwidth":0,"zeroline":false,"anchor":"x2","title":"","hoverformat":".2f"},"xaxis3":{"type":"date","autorange":true,"range":["1907-04-25","2028-04-23"],"tickmode":"auto","ticktext":["1920","1940","1960","1980","2000","2020"],"tickvals":[-18263,-10958,-3653,3652,10957,18262],"categoryorder":"array","categoryarray":["1920","1940","1960","1980","2000","2020"],"nticks":null,"ticks":"outside","tickcolor":"rgba(204,204,204,1)","ticklen":3.65296803652968,"tickwidth":0,"showticklabels":true,"tickfont":{"color":"rgba(44,62,80,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"domain":[0,0.414330058057811],"gridcolor":"rgba(204,204,204,1)","gridwidth":0,"zeroline":false,"anchor":"y3","title":"","hoverformat":".2f"},"yaxis3":{"type":"linear","autorange":true,"range":[-122.256869472569,2021.50320711859],"tickmode":"auto","ticktext":["0","500","1000","1500","2000"],"tickvals":[0,500,1000,1500,2000],"categoryorder":"array","categoryarray":["0","500","1000","1500","2000"],"nticks":null,"ticks":"outside","tickcolor":"rgba(204,204,204,1)","ticklen":3.65296803652968,"tickwidth":0,"showticklabels":true,"tickfont":{"color":"rgba(44,62,80,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"domain":[0.441094691930632,0.558905308069368],"gridcolor":"rgba(204,204,204,1)","gridwidth":0,"zeroline":false,"anchor":"x3","title":"","hoverformat":".2f"},"xaxis4":{"type":"date","autorange":true,"range":["1934-08-11","2027-01-05"],"tickmode":"auto","ticktext":["1940","1960","1980","2000","2020"],"tickvals":[-10958,-3653,3652,10957,18262],"categoryorder":"array","categoryarray":["1940","1960","1980","2000","2020"],"nticks":null,"ticks":"outside","tickcolor":"rgba(204,204,204,1)","ticklen":3.65296803652968,"tickwidth":0,"showticklabels":true,"tickfont":{"color":"rgba(44,62,80,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"domain":[0.585669941942189,1],"gridcolor":"rgba(204,204,204,1)","gridwidth":0,"zeroline":false,"anchor":"y4","title":"","hoverformat":".2f"},"yaxis4":{"type":"linear","autorange":true,"range":[-152.990484542881,2112.78629775456],"tickmode":"auto","ticktext":["0","500","1000","1500","2000"],"tickvals":[0,500,1000,1500,2000],"categoryorder":"array","categoryarray":["0","500","1000","1500","2000"],"nticks":null,"ticks":"outside","tickcolor":"rgba(204,204,204,1)","ticklen":3.65296803652968,"tickwidth":0,"showticklabels":true,"tickfont":{"color":"rgba(44,62,80,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"domain":[0.441094691930632,0.558905308069368],"gridcolor":"rgba(204,204,204,1)","gridwidth":0,"zeroline":false,"anchor":"x4","title":"","hoverformat":".2f"},"xaxis5":{"type":"date","autorange":true,"range":["1941-12-17","2026-08-30"],"tickmode":"auto","ticktext":["1960","1980","2000","2020"],"tickvals":[-3653,3652,10957,18262],"categoryorder":"array","categoryarray":["1960","1980","2000","2020"],"nticks":null,"ticks":"outside","tickcolor":"rgba(204,204,204,1)","ticklen":3.65296803652968,"tickwidth":0,"showticklabels":true,"tickfont":{"color":"rgba(44,62,80,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"domain":[0,0.414330058057811],"gridcolor":"rgba(204,204,204,1)","gridwidth":0,"zeroline":false,"anchor":"y5","title":"","hoverformat":".2f"},"yaxis5":{"type":"linear","autorange":true,"range":[-603.941418870245,13039.7878869381],"tickmode":"auto","ticktext":["0","4000","8000","12000"],"tickvals":[0,4000,8000,12000],"categoryorder":"array","categoryarray":["0","4000","8000","12000"],"nticks":null,"ticks":"outside","tickcolor":"rgba(204,204,204,1)","ticklen":3.65296803652968,"tickwidth":0,"showticklabels":true,"tickfont":{"color":"rgba(44,62,80,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"domain":[0,0.225571974736035],"gridcolor":"rgba(204,204,204,1)","gridwidth":0,"zeroline":false,"anchor":"x5","title":"","hoverformat":".2f"},"xaxis6":{"type":"date","autorange":true,"range":["1988-02-29","2024-06-17"],"tickmode":"auto","ticktext":["1990","2000","2010","2020"],"tickvals":[7305,10957,14610,18262],"categoryorder":"array","categoryarray":["1990","2000","2010","2020"],"nticks":null,"ticks":"outside","tickcolor":"rgba(204,204,204,1)","ticklen":3.65296803652968,"tickwidth":0,"showticklabels":true,"tickfont":{"color":"rgba(44,62,80,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"domain":[0.585669941942189,1],"gridcolor":"rgba(204,204,204,1)","gridwidth":0,"zeroline":false,"anchor":"y6","title":"","hoverformat":".2f"},"yaxis6":{"type":"linear","autorange":true,"range":[1228.61266333213,1775.17472795695],"tickmode":"auto","ticktext":["1300","1400","1500","1600","1700"],"tickvals":[1300,1400,1500,1600,1700],"categoryorder":"array","categoryarray":["1300","1400","1500","1600","1700"],"nticks":null,"ticks":"outside","tickcolor":"rgba(204,204,204,1)","ticklen":3.65296803652968,"tickwidth":0,"showticklabels":true,"tickfont":{"color":"rgba(44,62,80,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"domain":[0,0.225571974736035],"gridcolor":"rgba(204,204,204,1)","gridwidth":0,"zeroline":false,"anchor":"x6","title":"","hoverformat":".2f"},"showlegend":false,"legend":{"bgcolor":"rgba(255,255,255,1)","bordercolor":"transparent","borderwidth":0,"font":{"color":"rgba(44,62,80,1)","family":"","size":11.689497716895}},"hovermode":"closest","barmode":"relative"},"config":{"doubleClick":"reset","modeBarButtonsToAdd":["hoverclosest","hovercompare"],"showSendToCloud":false},"source":"A","attrs":{"6597457b1269":{"x":{},"y":{},"type":"scatter"},"659712f5cf22":{"x":{},"y":{}}},"cur_data":"6597457b1269","visdat":{"6597457b1269":["function (y) ","x"],"659712f5cf22":["function (y) ","x"]},"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.2,"selected":{"opacity":1},"debounce":0},"shinyEvents":["plotly_hover","plotly_click","plotly_selected","plotly_relayout","plotly_brushed","plotly_brushing","plotly_clickannotation","plotly_doubleclick","plotly_deselect","plotly_afterplot","plotly_sunburstclick"],"base_url":"https://plot.ly"},"evals":[],"jsHooks":[]}</script>

Each *parent_entity* has its own trend over time.

## Anomalies and outliers

``` r
library(anomalize)

plot_data |> 
    group_by(parent_entity_fct) |> 
    time_decompose(sum) |> 
    anomalize(remainder) |>
  plot_anomalies(size_dots = 1, ncol = 2)
```

<img src="index.markdown_strict_files/figure-markdown_strict/unnamed-chunk-10-1.png" width="1260" />

``` r
plot_data |> 
    filter(parent_entity_fct=="Saudi Aramco") |> 
    time_decompose(sum) |> 
    anomalize(remainder) |>
  plot_anomaly_decomposition()
```

<img src="index.markdown_strict_files/figure-markdown_strict/unnamed-chunk-12-1.png" width="1260" />

So for simplicity, I will replace the anomalies detected by the trend for all the data. All the subsequent analysis will be done with the corrected data for the top 50 countries

``` r
top50_entity<-sum_emissions_entity |> 
  top_n(50, sum) |> 
  select(parent_entity)


final_data<-emissions |> 
  filter(parent_entity %in% top50_entity$parent_entity) |>
  group_by(parent_entity, year) |>
  summarize(sum=sum(total_emissions_MtCO2e)) |>
  ungroup() |>
  mutate(date=as.Date(as.character(year), "%Y"),
         parent_entity_fct=as.factor(parent_entity)) |>
  select(parent_entity_fct, date, sum) |>
  filter(parent_entity_fct %ni% c("Seriti Resources", "CNX Resources", "Navajo Transitional Energy Company"))|> 
  pad_by_time(date, 
              .by = "year", 
              .pad_value = NA) |> 
    group_by(parent_entity_fct) |> 
    time_decompose(sum) |> 
    anomalize(remainder) |> 
  mutate(observed=case_when(anomaly=="Yes" ~ trend,
                            TRUE ~ observed)) |> 
  select(parent_entity_fct, date, observed)
```

## Conclusion

In this first part, we have explored the dataset and identified the main characteristics. We have seen that the carbon emissions have increased over time and that the top 50 countries have different trends. We have also identified some anomalies and outliers that have been correct for the work to come in the next part.

<a href = "https://johaniefournier.aweb.page/p/4b2b1e24-af09-488d-8ff6-7b46ce61e367"> ![](petit.png)

## Session Info

``` r
sessionInfo()
```

    R version 4.1.1 (2021-08-10)
    Platform: x86_64-apple-darwin17.0 (64-bit)
    Running under: macOS Big Sur 10.16

    Matrix products: default
    BLAS:   /Library/Frameworks/R.framework/Versions/4.1/Resources/lib/libRblas.0.dylib
    LAPACK: /Library/Frameworks/R.framework/Versions/4.1/Resources/lib/libRlapack.dylib

    locale:
    [1] en_US.UTF-8/en_US.UTF-8/en_US.UTF-8/C/en_US.UTF-8/en_US.UTF-8

    attached base packages:
    [1] stats     graphics  grDevices datasets  utils     methods   base     

    other attached packages:
     [1] anomalize_0.3.0            PerformanceAnalytics_2.0.4
     [3] xts_0.12.1                 zoo_1.8-12                
     [5] jofou.lib_0.0.0.9000       reticulate_1.37.0         
     [7] tidytuesdayR_1.0.2         tictoc_1.2.1              
     [9] terra_1.6-17               sf_1.0-5                  
    [11] pins_1.0.1.9000            fs_1.5.2                  
    [13] timetk_2.6.1               yardstick_1.2.0           
    [15] workflowsets_0.1.0         workflows_0.2.4           
    [17] tune_0.1.6                 rsample_0.1.0             
    [19] parsnip_1.1.1              modeldata_0.1.1           
    [21] infer_1.0.0                dials_0.0.10              
    [23] scales_1.2.1               broom_1.0.4               
    [25] tidymodels_0.1.4           recipes_0.1.17            
    [27] doFuture_0.12.0            future_1.22.1             
    [29] foreach_1.5.1              skimr_2.1.5               
    [31] forcats_1.0.0              stringr_1.5.0             
    [33] dplyr_1.1.2                purrr_1.0.1               
    [35] readr_2.1.4                tidyr_1.3.0               
    [37] tibble_3.2.1               ggplot2_3.4.2             
    [39] tidyverse_2.0.0            lubridate_1.9.2           
    [41] kableExtra_1.3.4.9000      inspectdf_0.0.11          
    [43] openxlsx_4.2.4             knitr_1.36                

    loaded via a namespace (and not attached):
      [1] readxl_1.4.2       backports_1.4.1    systemfonts_1.0.3 
      [4] lazyeval_0.2.2     repr_1.1.7         splines_4.1.1     
      [7] crosstalk_1.1.1    listenv_0.8.0      usethis_2.0.1     
     [10] digest_0.6.29      htmltools_0.5.8.1  fansi_0.5.0       
     [13] magrittr_2.0.3     tzdb_0.1.2         globals_0.14.0    
     [16] ggfittext_0.9.1    gower_0.2.2        vroom_1.6.0       
     [19] svglite_2.0.0      hardhat_1.3.0      timechange_0.1.1  
     [22] tseries_0.10-48    forecast_8.15      prettyunits_1.1.1 
     [25] colorspace_2.0-2   rvest_1.0.3        rappdirs_0.3.3    
     [28] xfun_0.39          crayon_1.4.2       jsonlite_1.8.4    
     [31] survival_3.2-11    iterators_1.0.13   glue_1.6.2        
     [34] gtable_0.3.0       ipred_0.9-12       webshot_0.5.2     
     [37] future.apply_1.8.1 quantmod_0.4.18    padr_0.6.0        
     [40] DBI_1.1.1          Rcpp_1.0.13        viridisLite_0.4.0 
     [43] progress_1.2.2     units_0.7-2        GPfit_1.0-8       
     [46] bit_4.0.4          proxy_0.4-26       tibbletime_0.1.8  
     [49] DT_0.19            lava_1.6.10        prodlim_2019.11.13
     [52] htmlwidgets_1.5.4  httr_1.4.6         farver_2.1.0      
     [55] pkgconfig_2.0.3    sass_0.4.0         nnet_7.3-16       
     [58] utf8_1.2.2         labeling_0.4.2     tidyselect_1.2.0  
     [61] rlang_1.1.1        DiceDesign_1.9     munsell_0.5.0     
     [64] cellranger_1.1.0   tools_4.1.1        cli_3.6.1         
     [67] sweep_0.2.5        generics_0.1.3     evaluate_0.14     
     [70] fastmap_1.2.0      yaml_2.2.1         bit64_4.0.5       
     [73] zip_2.2.0          nlme_3.1-152       xml2_1.3.4        
     [76] compiler_4.1.1     rstudioapi_0.14    plotly_4.10.0     
     [79] curl_5.2.3         png_0.1-7          e1071_1.7-9       
     [82] lhs_1.1.3          bslib_0.3.1        stringi_1.7.5     
     [85] highr_0.9          lattice_0.20-44    Matrix_1.3-4      
     [88] classInt_0.4-3     urca_1.3-0         vctrs_0.6.5       
     [91] pillar_1.9.0       lifecycle_1.0.3    furrr_0.2.3       
     [94] lmtest_0.9-38      jquerylib_0.1.4    data.table_1.14.2 
     [97] R6_2.5.1           renv_1.0.7         KernSmooth_2.23-20
    [100] parallelly_1.28.1  codetools_0.2-18   assertthat_0.2.1  
    [103] MASS_7.3-54        withr_2.5.0        fracdiff_1.5-1    
    [106] parallel_4.1.1     hms_1.1.3          quadprog_1.5-8    
    [109] grid_4.1.1         rpart_4.1-15       timeDate_3043.102 
    [112] class_7.3-19       rmarkdown_2.25     TTR_0.24.2        
    [115] base64enc_0.1-3   
