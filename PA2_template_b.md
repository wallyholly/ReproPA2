# Damage Analysis of Storms
# Synopsis
Immediately after the title, there should be a synopsis which describes and summarizes your analysis in at most 10 complete sentences.

# Data Processing

```r
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
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
#storm <- read.csv(bzfile(paste("repdata_data_StormData.csv.bz2", sep = "")),sep=",")
storm_sample<-tbl_df(read.csv("storm_sample2.csv"))
head(storm_sample)
```

```
## Source: local data frame [6 x 38]
## 
##        X STATE__          BGN_DATE    BGN_TIME TIME_ZONE COUNTY COUNTYNAME
##    (int)   (int)            (fctr)      (fctr)    (fctr)  (int)     (fctr)
## 1 273753      46 6/20/1996 0:00:00 02:00:00 PM       CST    127      UNION
## 2 786503      72  9/5/2009 0:00:00 06:42:00 AM       AST      4     PRZ004
## 3 814171      29 6/14/2010 0:00:00 09:00:00 AM       CST    127     MARION
## 4 300285      40 6/16/1997 0:00:00 09:14:00 PM       CST     19     CARTER
## 5 253373      12 8/19/1996 0:00:00 02:15:00 PM       EST     21    COLLIER
## 6 607877      12  4/8/2006 0:00:00 06:15:00 PM       EST     79    MADISON
## Variables not shown: STATE (fctr), EVTYPE (fctr), BGN_RANGE (dbl), BGN_AZI
##   (fctr), BGN_LOCATI (fctr), END_DATE (fctr), END_TIME (fctr), COUNTY_END
##   (int), COUNTYENDN (lgl), END_RANGE (dbl), END_AZI (fctr), END_LOCATI
##   (fctr), LENGTH (dbl), WIDTH (int), F (int), MAG (int), FATALITIES (int),
##   INJURIES (int), PROPDMG (dbl), PROPDMGEXP (fctr), CROPDMG (dbl),
##   CROPDMGEXP (fctr), WFO (fctr), STATEOFFIC (fctr), ZONENAMES (fctr),
##   LATITUDE (int), LONGITUDE (int), LATITUDE_E (int), LONGITUDE_ (int),
##   REMARKS (fctr), REFNUM (int)
```


There should be a section titled Data Processing which describes (in words and code) how the data were loaded into R and processed for analysis. In particular, your analysis must start from the raw CSV file containing the data. You cannot do any preprocessing outside the document. If preprocessing is time-consuming you may consider using the cache = TRUE option for certain code chunks.

# Publication
For this assignment you will need to publish your analysis on RPubs.com. If you do not already have an account, then you will have to create a new account. After you have completed writing your analysis in RStudio, you can publish it to RPubs by doing the following:  
    * In RStudio, make sure your R Markdown document (.Rmd) document is loaded in the editor  
    * Click the Knit HTML button in the doc toolbar to preview your document.  
    * In the preview window, click the Publish button.  
Once your document is published to RPubs, you should get a unique URL to that document. Make a note of this URL as you will need it to submit your assignment.

# Questions
Your data analysis must address the following questions:  
1. Across the United States, which types of events (as indicated in the EVTYPE variable) are most harmful with respect to population health?  
2. Across the United States, which types of events have the greatest economic consequences?  
Consider writing your report as if it were to be read by a government or municipal manager who might be responsible for preparing for severe weather events and will need to prioritize resources for different types of events. However, there is no need to make any specific recommendations in your report.
## Question 1
First of all we group the storm data by all events, after that we calculate the sum of all fatalities and injuries by each event type. After that, we filter all events in which fatalities OR injuries occured.

```r
by_events<-group_by(storm_sample,EVTYPE)
person_damage<-summarize(by_events,sum_fat=sum(FATALITIES),sum_inj=sum(INJURIES))
damages<-filter(person_damage,sum_inj>0 | sum_fat>0)
```
