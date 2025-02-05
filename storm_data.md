# storm_data
tbaber  
April 18, 2016  


```{r setup, include=TRUE
knitr::opts_chunk$set(echo = TRUE)
library(ggplot2)
library(plyr)
}
```

## Reproducible Research: Peer Assessment2

##Impact of Severe Weather Events on Public Health and Economy in the United States

###Synopsis

In this report, we aim to analyze the impact of different weather events on public health and economy based on the storm database collected from the U.S. National Oceanic and Atmospheric Administration's (NOAA) from 1950 - 2011. We will use the estimates of fatalities, injuries, property and crop damage to decide which types of event are most harmful to the population health and economy. From these data, we found that excessive heat and tornado are most harmful with respect to population health, while flood, drought, and hurricane/typhoon have the greatest economic consequences.

###Data Processing

Download the file from the website and unzip it 


```r
if (!file.exists("repdata-data-StormData.csv.bz2")) {
      download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2","repdata-data-StormData.csv.bz2")
}
StormData <- read.csv("repdata-data-StormData.csv.bz2", stringsAsFactors = F)
library(ggplot2)
library(plyr)
```
Check the names of the variables in the file
```
dim(StormData)
```
```
names(StormData)

```

Get the first 6 rows

```
head(StormData)
```
Very little data is available for analysis between 1950 and 1990 as evident from the following histogram


```r
StormData$year <- as.numeric(format(as.Date(StormData$BGN_DATE , format = "%m/%d/%Y %H:%M:%S"), "%Y"))

hist(StormData$year, breaks = 30)
```

![](storm_data_files/figure-html/unnamed-chunk-2-1.png)
Therefore, we will use the data from 1990 onwards for more complete records
```
storm <- StormData[StormData$year >=1995, ]
dim(storm)
```
Now there are 681500 rows and 38 columns 

##Impact on Public Health


We will get the first 15 most severe events and check their impact on public health by looking at number of fatalities and injuries caused by these events
FATALITIES and INJURIES are the 2 columns we will aggregate to determine the impact on Public Health

###Fatalities


```r
fatalities = ddply(StormData, .(EVTYPE), summarize, sum = sum(FATALITIES))
fatalities = fatalities[order(fatalities$sum, decreasing = TRUE), ]
head(fatalities, 5)
```

```
##             EVTYPE  sum
## 834        TORNADO 5633
## 130 EXCESSIVE HEAT 1903
## 153    FLASH FLOOD  978
## 275           HEAT  937
## 464      LIGHTNING  816
```


```r
library(ggplot2)
ggplot(fatalities[1:6, ], aes(EVTYPE, sum, fill=EVTYPE,alpha=0.3)) + geom_bar(stat = "identity") + 
  xlab("Event Type") + ylab("Number of Fatalities") + ggtitle("Fatalities by Event type") 
```

![](storm_data_files/figure-html/unnamed-chunk-4-1.png)

###Injuries


```r
library(plyr)
injuries = ddply(StormData, .(EVTYPE), summarize, sum.injuries = sum(INJURIES,na.rm=TRUE))
injuries = injuries[order(injuries$sum.injuries, decreasing = TRUE), ]
```
Look at the 5 most harmful eather events

```r
head(injuries,5)
```

```
##             EVTYPE sum.injuries
## 834        TORNADO        91346
## 856      TSTM WIND         6957
## 170          FLOOD         6789
## 130 EXCESSIVE HEAT         6525
## 464      LIGHTNING         5230
```

```r
ggplot(injuries[1:6, ], aes(EVTYPE, sum.injuries, fill = EVTYPE,alpha=0.5)) + geom_bar(stat = "identity") + 
  xlab("Event Type") + ylab("Number of Injuries") + ggtitle("Injuries by Event type")
```

![](storm_data_files/figure-html/unnamed-chunk-7-1.png)

###Property Damage

First look at the various exponents of PROPDMGEXP

```r
unique(StormData$PROPDMGEXP)
```

```
##  [1] "K" "M" ""  "B" "m" "+" "0" "5" "6" "?" "4" "2" "3" "h" "7" "H" "-"
## [18] "1" "8"
```
Convert the letters to the corresponding exponents and recalculate the total property damage

```r
StormData$pd <- 0
StormData[StormData$PROPDMGEXP == "H", ]$pd <- 
  StormData[StormData$PROPDMGEXP == "H", ]$PROPDMG * 10^2

StormData[StormData$PROPDMGEXP == "K", ]$pd <- 
  StormData[StormData$PROPDMGEXP == "K", ]$PROPDMG * 10^3

StormData[StormData$PROPDMGEXP == "M", ]$pd <- 
  StormData[StormData$PROPDMGEXP == "M", ]$PROPDMG * 10^6

StormData[StormData$PROPDMGEXP == "B", ]$pd <- 
  StormData[StormData$PROPDMGEXP == "B", ]$PROPDMG * 10^9

# Converting the H, K, M, B into units to be able to calculate Crop Damage
StormData$cd <- 0

StormData[StormData$CROPDMGEXP == "H", ]$cd <- 
  StormData[StormData$CROPDMGEXP == "H", ]$CROPDMG * 10^2

StormData[StormData$CROPDMGEXP == "K", ]$cd <- 
  StormData[StormData$CROPDMGEXP == "K", ]$CROPDMG * 10^3

StormData[StormData$CROPDMGEXP == "M", ]$cd <- 
  StormData[StormData$CROPDMGEXP == "M", ]$CROPDMG * 10^6

StormData[StormData$CROPDMGEXP == "B", ]$cd <- 
  StormData[StormData$CROPDMGEXP == "B", ]$CROPDMG * 10^9
```
Now we make a new dataset of property damage arranged according to events type and look at the first 6 major events in terms of economic loss



```r
dam <- aggregate(pd + cd ~ EVTYPE, data = StormData, sum)
names(dam) <- c("EVTYPE", "TDAMAGE")
dam <- dam[order(-dam$TDAMAGE), ][1:10, ]
dam$EVTYPE <- factor(dam$EVTYPE, levels = dam$EVTYPE)
```
Plotting the total damage by event type

```r
ggplot(dam, aes(x = EVTYPE, y = TDAMAGE)) + 
    geom_bar(stat = "identity", fill ="red" ) + 
    theme(axis.text.x = element_text(angle = 90, hjust = 1)) + 
    xlab("EVENT TYPE") + ylab("DAMAGES (US$)") +
  ggtitle("Property damage by Top 10 Weather Events")
```

![](storm_data_files/figure-html/unnamed-chunk-11-1.png)

###Results

It is evident from this exploratory analysis presented here that flood brings about the most economic loss to US population while tornado is is responsible to most deaths and injuries. If agriculture is the dominating economic factor, then drought may be the most harful factor for the economy. In regards to human health, more priorities naturally will go towards addressing tornado as it claims most lives and causes injuries.


