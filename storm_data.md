---
title: "storm_data"
author: "tbaber"
date: "April 18, 2016"
output: html_document
---

```{r setup, include=TRUE}
knitr::opts_chunk$set(echo = TRUE)
```

## Reproducible Research: Peer Assessment2

##Impact of Severe Weather Events on Public Health and Economy in the United States

###Synopsis

In this report, we aim to analyze the impact of different weather events on public health and economy based on the storm database collected from the U.S. National Oceanic and Atmospheric Administration's (NOAA) from 1950 - 2011. We will use the estimates of fatalities, injuries, property and crop damage to decide which types of event are most harmful to the population health and economy. From these data, we found that excessive heat and tornado are most harmful with respect to population health, while flood, drought, and hurricane/typhoon have the greatest economic consequences.

###Data Processing

Download the file from the website and unzip it 

```
if (!file.exists("repdata-data-StormData.csv.bz2")) {
      download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2","repdata-data-StormData.csv.bz2")
}
StormData <- read.csv("repdata-data-StormData.csv.bz2", stringsAsFactors = F)
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

```
StormData$year <- as.numeric(format(as.Date(StormData$BGN_DATE , format = "%m/%d/%Y %H:%M:%S"), "%Y"))

hist(StormData$year, breaks = 30)
```
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

```
agg.fatalities.data <-
        aggregate(
                StormData$FATALITIES, 
                by=list(StormData$EVTYPE), FUN=sum, na.rm=TRUE)
colnames(agg.fatalities.data) = c("event.type", "fatality.total")

fatalities.sorted <- 
    agg.fatalities.data[order(-agg.fatalities.data$fatality.total),] 
top.fatalities <- fatalities.sorted[1:10,]
top.fatalities$event.type <- 
    factor(
        top.fatalities$event.type, levels=top.fatalities$event.type, 
        ordered=TRUE)
```
###Injuries
Aggregate injuries using same method

```
agg.injuries.data <-
        aggregate(
                StormData$INJURIES, 
                by=list(StormData$EVTYPE), FUN=sum, na.rm=TRUE)
colnames(agg.injuries.data) = c("event.type", "injury.total")
injuries.sorted <- agg.injuries.data[order(-agg.injuries.data$injury.total),] 
top.injuries <- injuries.sorted[1:10,]
top.injuries$event.type <- 
    factor(
        top.injuries$event.type, levels=top.injuries$event.type, 
        ordered=TRUE)
```
###Property Damage
Lastly, do the same for Property impact

```
agg.prop.dmg.data <-
        aggregate(
                StormData$PROPDMG, 
                by=list(StormData$EVTYPE), FUN=sum, na.rm=TRUE)
colnames(agg.prop.dmg.data) = c("event.type", "prop.dmg.total")
prop.dmg.sorted <- agg.prop.dmg.data[order(-agg.prop.dmg.data$prop.dmg.total),] 
top.prop.dmg <- prop.dmg.sorted[1:10,]
top.prop.dmg$event.type <- 
    factor(
        top.prop.dmg$event.type, levels=top.prop.dmg$event.type, 
        ordered=TRUE)
```
##Results
###Top 10 causes of fatalities
```
library(ggplot2)
ggplot(data=top.fatalities, aes(x=event.type, y=fatality.total)) + 
    geom_bar(stat="identity") + xlab("Event type") + ylab("Total fatalities") + 
    ggtitle("Fatalities By Event Type") +
    theme(axis.text.x = element_text(angle = 45, hjust = 1))
```
###Top 10 causes of injuries
```
ggplot(data=top.injuries, aes(x=event.type, y=injury.total)) + 
    geom_bar(stat="identity") + xlab("Event type") + ylab("Total injuries") + 
    ggtitle("Injuries By Event Type") +
    theme(axis.text.x = element_text(angle = 45, hjust = 1))
```
###Top 10 causes for property damage
```
ggplot(data=top.prop.dmg, aes(x=event.type, y=prop.dmg.total)) + 
    geom_bar(stat="identity") + xlab("Event type") + 
    ylab("Total property damage") +  ggtitle("Property Damage By Event Type") + 
    theme(axis.text.x = element_text(angle = 45, hjust = 1))
```


