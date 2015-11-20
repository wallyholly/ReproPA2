# Damage Analysis of Storms
# Synopsis
This document presents an analysis of weather event types and their negative impacts on economic factors (property and crop damages) and personal health (number of fatalities and number of injuries). It is analyzed which weather events have the largest impacts on each, health and economy.

# Data Processing

```r
library(plyr)
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
## 
## The following objects are masked from 'package:plyr':
## 
##     arrange, count, desc, failwith, id, mutate, rename, summarise,
##     summarize
## 
## The following objects are masked from 'package:stats':
## 
##     filter, lag
## 
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
setwd ("c:/r_dat/repro_pa2")
storm <- read.csv(bzfile(paste("repdata_data_StormData.csv.bz2", sep = "")),sep=",")
#storm<-tbl_df(read.csv("storm_sample.csv"))
```
## Data preparation for the results of question 1
We group the storm data by EVTYPE and calculate the sum of all fatalities and injuries by each event type for the grouped stormdata with the summarize function. After that, we filter all events in which fatalities OR injuries occured. Then we sort the data by the sum of fatalities and then by the sum of injuries.

```r
by_events<-group_by(storm,EVTYPE)
person_damage<-summarize(by_events,sum_fat=sum(FATALITIES),sum_inj=sum(INJURIES))
pdamages<-filter(person_damage,sum_inj>0 | sum_fat>0)
```
## Data preparation for the results of question 2
For the property damage values and the crop damage values, the units of measurements (PROPDMGEXP, CROPDMGEXP) are replaced by numeric values in the storm data set. After that the values in PROPDMG and CROPDMG are multiplied by the numeric values of the units of measurement. We then group the processed data by the weather event variable EVTYPE. After that the amounts of damages for both types (property and crop) are calculated (sum_precondam and sum_crecondam). The data are filtered, sorted in the last step.

```r
storm$PROPDMGEXP <- gsub("\\-|\\+|\\?|0", "1", storm$PROPDMGEXP)
storm$PROPDMGEXP <- gsub("h|H|2","100", storm$PROPDMGEXP)
storm$PROPDMGEXP <- gsub("k|K|3","1000", storm$PROPDMGEXP)
storm$PROPDMGEXP <- gsub("4","10000", storm$PROPDMGEXP)
storm$PROPDMGEXP <- gsub("5","100000", storm$PROPDMGEXP)
storm$PROPDMGEXP <- gsub("m|M|6","1000000", storm$PROPDMGEXP)
storm$PROPDMGEXP <- gsub("7","10000000", storm$PROPDMGEXP)
storm$PROPDMGEXP <- gsub("8","100000000", storm$PROPDMGEXP)
storm$PROPDMGEXP <- gsub("b|B|9","1000000000", storm$PROPDMGEXP)
storm$PROPDMGEXP <- as.numeric(storm$PROPDMGEXP)

storm$CROPDMGEXP <- gsub("\\-|\\+|\\?|0", "1", storm$CROPDMGEXP)
storm$CROPDMGEXP <- gsub("h|H|2","100", storm$CROPDMGEXP)
storm$CROPDMGEXP <- gsub("k|K|3","1000", storm$CROPDMGEXP)
storm$CROPDMGEXP <- gsub("4","10000", storm$CROPDMGEXP)
storm$CROPDMGEXP <- gsub("5","100000", storm$CROPDMGEXP)
storm$CROPDMGEXP <- gsub("m|M|6","1000000", storm$CROPDMGEXP)
storm$CROPDMGEXP <- gsub("7","10000000", storm$CROPDMGEXP)
storm$CROPDMGEXP <- gsub("8","100000000", storm$CROPDMGEXP)
storm$CROPDMGEXP <- gsub("b|B|9","1000000000", storm$CROPDMGEXP)
storm$CROPDMGEXP <- as.numeric(storm$CROPDMGEXP)

storm$PRECONDAM <- storm$PROPDMG * storm$PROPDMGEXP
storm$CRECONDAM <- storm$CROPDMG * storm$CROPDMGEXP
storm$ECONDAM <- storm$PRECONDAM + storm$CRECONDAM

by_events_econ<-group_by(storm,EVTYPE)
econ_damage<-summarize(by_events_econ, sum_econdam=sum(ECONDAM))
edamages<-filter(econ_damage,sum_econdam>0)
edamages<-arrange(edamages, desc(sum_econdam))
edamages$sum_econdam<-edamages$sum_econdam/1000000000
```

# Results
## Results Q1
In order to analyse Q1, we first print the TOP 10 harmful events by their number of fatalities and injuries! After that we present a panel bar plot to illustrate the results graphically.

```r
pdamages_fat<-head(arrange(pdamages, desc(sum_fat)),10)
pdamages_inj<-head(arrange(pdamages, desc(sum_inj)),10)

print(pdamages_fat)
```

```
## Source: local data frame [10 x 3]
## 
##            EVTYPE sum_fat sum_inj
##            (fctr)   (dbl)   (dbl)
## 1         TORNADO    5633   91346
## 2  EXCESSIVE HEAT    1903    6525
## 3     FLASH FLOOD     978    1777
## 4            HEAT     937    2100
## 5       LIGHTNING     816    5230
## 6       TSTM WIND     504    6957
## 7           FLOOD     470    6789
## 8     RIP CURRENT     368     232
## 9       HIGH WIND     248    1137
## 10      AVALANCHE     224     170
```

```r
print(pdamages_inj)
```

```
## Source: local data frame [10 x 3]
## 
##               EVTYPE sum_fat sum_inj
##               (fctr)   (dbl)   (dbl)
## 1            TORNADO    5633   91346
## 2          TSTM WIND     504    6957
## 3              FLOOD     470    6789
## 4     EXCESSIVE HEAT    1903    6525
## 5          LIGHTNING     816    5230
## 6               HEAT     937    2100
## 7          ICE STORM      89    1975
## 8        FLASH FLOOD     978    1777
## 9  THUNDERSTORM WIND     133    1488
## 10              HAIL      15    1361
```

```r
par(mfrow=c(1,2))
barplot(pdamages_fat$sum_fat,names.arg=pdamages_fat$EVTYPE,cex.names=0.5,las=2)
barplot(pdamages_inj$sum_inj,names.arg=pdamages_inj$EVTYPE,cex.names=0.5,las=2)
```

![](PA2_template_b_files/figure-html/unnamed-chunk-4-1.png) 


## Results Q2
In order to analyse Q2, we first print the TOP 10 harmful events by their ecnomomic impact! After that we present a bar plot to illustrate the results graphically.

```r
edamages<-head(edamages,20)
print(edamages)
```

```
## Source: local data frame [20 x 2]
## 
##                           EVTYPE sum_econdam
##                           (fctr)       (dbl)
## 1     TORNADOES, TSTM WIND, HAIL   1.6025000
## 2                HIGH WINDS/COLD   0.1175000
## 3      HURRICANE OPAL/HIGH WINDS   0.1100000
## 4        WINTER STORM HIGH WINDS   0.0650000
## 5           Heavy Rain/High Surf   0.0150000
## 6                LAKESHORE FLOOD   0.0075400
## 7         HIGH WINDS HEAVY RAINS   0.0075100
## 8                   FOREST FIRES   0.0055000
## 9           FLASH FLOODING/FLOOD   0.0019250
## 10 HEAVY SNOW/HIGH WINDS & FLOOD   0.0015200
## 11                  Frost/Freeze   0.0011000
## 12         TROPICAL STORM GORDON   0.0010000
## 13         DUST STORM/HIGH WINDS   0.0005500
## 14      MARINE THUNDERSTORM WIND   0.0004864
## 15         ASTRONOMICAL LOW TIDE   0.0003200
## 16                     GLAZE ICE   0.0003053
## 17             HEAT WAVE DROUGHT   0.0002500
## 18             THUNDERSTORM HAIL   0.0000550
## 19            HIGH WIND AND SEAS   0.0000500
## 20             WILD/FOREST FIRES   0.0000320
```

```r
barplot(edamages$sum_econdam, names.arg=edamages$EVTYPE,cex.names=0.5,las=2)
```

![](PA2_template_b_files/figure-html/unnamed-chunk-5-1.png) 
