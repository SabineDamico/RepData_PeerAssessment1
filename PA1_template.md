---
title: '#5: Reproducible Research - Project 1'
author: "Sabine D'Amico"
date: "October 26, 2018"
output:
  keep_md: yes
  pdf_document: default
  html_document: default
---

```{r, echo=FALSE}
options(repos = getOption("repos")["CRAN"])
install.packages("chron")
install.packages("tidyverse")
```
```{r, echo=FALSE}
library(chron)
```

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## Loading and preprocessing the data

The activities data was obtained by forking a repository from the following site: https://github.com/rdpeng/RepData_PeerAssessment1
and subsequently downloaded to the "~/R/data" folder


```{r activities, echo=FALSE}
if(!file.exists("~./R/data")){dir.create("~./R/data")}
activities <- read.csv("~./R/data/activity.csv")
summary(activities)
```

## Basic information about the data set
#The total number of steps per day
```{r, echo=FALSE}
totalsteps <-aggregate(steps ~ date, activities,sum)
totalsteps
```

#Histogram of total steps taken per day
Bring up ggplot2
```{r, echo=FALSE}
library(ggplot2)
```
Print histogram
```{r}
plot1a.R <- qplot(totalsteps$steps, geom = "histogram", main = "# of Steps by day", xlab = "Steps",fill=I("coral"),col=I("black"))
plot1a.R
```

#The mean and median total number of steps taken per day 

```{r}
mean(totalsteps$steps, na.rm = TRUE)
median(totalsteps$steps,na.rm = TRUE)
```
#What is the average daily activity pattern?
```{r}
interval <- as.factor(as.character(activities$interval))
mean_int <- as.numeric(tapply(activities$steps, activities$interval, mean, na.rm=TRUE))
mean_int
```
# create a line graph
```{r}
intervalDF <- data.frame(intervalDF = as.numeric(levels(interval)),mean_int)
intervalDF <- intervalDF[order(intervalDF$intervalDF),]
plot1b.R <- plot(intervalDF$intervalDF, intervalDF$mean_int, type = "l", col="red",lwd=2, main = "Average steps per minute", xlab="Time of Day", ylab="Average Steps")
plot1b.R
```


The max value and the interval with the max value, respectively are 
```{r}
round(max(intervalDF$mean_int), digits = 0)
intervalDF$intervalDF[intervalDF$mean_int == max(intervalDF$mean_int)]
```

## Impute missing values
Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)
```{r}
sum(is.na(activities$steps))
```
Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
```{r, echo=FALSE}
activitiesDF <- data.frame(activities)
activitiesreplaceNA <- activitiesDF
activitiesreplaceNA$steps[is.na(activitiesreplaceNA$steps)] <- mean(activitiesDF$steps,na.rm = TRUE) #to replace NA values with the mean of the non-NA values
```

```{r}
newactivitiesDF <- aggregate(steps ~ interval + date ,activitiesreplaceNA, mean)
newtotalsteps <- aggregate(steps ~ date, activitiesreplaceNA, sum)
plot1c.R <- qplot(newtotalsteps$steps, geom = "histogram", main = "# of Steps by day", xlab = "Steps", fill=I("lavender"),col=I("black"))
plot1c.R
```

```{r}
mean(newtotalsteps$steps, na.rm = TRUE)
median(newtotalsteps$steps, na.rm = TRUE)
```
##Are there differences in activity patterns between weekdays and weekends?
```{r}
weekday <- weekdays(as.Date(newactivitiesDF$date))
newactivitiesDFwweekday <- cbind(newactivitiesDF,weekday)
newactivitiesDFwweekendday <- data.frame(newactivitiesDFwweekday)
```
# add weekday or weekend designation to data set
```{r}
weekend <- ifelse(chron::is.weekend(newactivitiesDFwweekendday$date)==TRUE, "Weekend","weekday")
newactivitiesDFwweekendday <- cbind(newactivitiesDFwweekday, weekend)
```
#Make a panel plot containing a time series plot (type="l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis)
```{r}
activtybystepsandweektype <- data.frame(newactivitiesDFwweekendday[,1],newactivitiesDFwweekendday[,3],newactivitiesDFwweekendday[,5])
colnames(activtybystepsandweektype) <- c("interval","steps","weekdaytype")
```
# summarize by average steps and prepare a line graph by average steps
```{r}
activtybystepsandweektype1 <- aggregate(steps ~ interval + weekdaytype ,activtybystepsandweektype, mean)
plot1d.R <- ggplot(data = activtybystepsandweektype1, aes(x = interval, y = steps))+
  facet_grid(weekdaytype ~ .)+
  geom_line(colour="coral", size = 1.5)+ theme_classic()+
  ggtitle("Average steps by weekday/weekend")
plot1d.R
```


## conclusion:There are some differences in activities by weekend and weekday. There is a spike in activity on weekday mornings and little activity during the rest of the day. On weekends, however, there is a later start in activity but the activity continues for most of the day.


























