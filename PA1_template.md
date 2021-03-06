---
title: "Reproducible Research: Peer Assessment 1"
author: Karri S.
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data



```r
f <- unz('activity.zip',
         filename='activity.csv')
d <- read.table(f, header=T, sep=",", stringsAsFactors=F)

library(ggplot2)
library(dplyr)
```


## What is mean total number of steps taken per day?

Histogram of the total number of daily steps.


```r
dd <- d %>% group_by(date) %>% summarise(steps=sum(steps))
ggplot(dd, aes(x=steps)) + geom_histogram()
```

```
## stat_bin: binwidth defaulted to range/30. Use 'binwidth = x' to adjust this.
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2-1.png) 


Mean and median number of steps taken each day


```r
c(mean(dd$steps, na.rm=T), median(dd$steps, na.rm=T))
```

```
## [1] 10766.19 10765.00
```


## What is the average daily activity pattern?

Average number of steps taken during the day.


```r
da <- d %>% group_by(interval) %>% summarise(steps=mean(steps, na.rm=T))
ggplot(da, aes(x=interval, y=steps)) + geom_line()
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png) 

Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
da$interval[da$steps == max(da$steps)]
```

```
## [1] 835
```

## Imputing missing values

Total number of missing values.


```r
sum(is.na(d$steps))
```

```
## [1] 2304
```

Impute missing data: use values averaged over days which were calculated in the previous step.  


```r
dm <- d
dm$steps[is.na(dm$steps)] = rep(da$steps, length(dm$steps)/length(da$steps))[is.na(dm$steps)]
```

Histogram of the total number of daily steps with imputed data.


```r
dd2 <- dm %>% group_by(date) %>% summarise(steps=sum(steps))
ggplot(dd2, aes(x=steps)) + geom_histogram()
```

```
## stat_bin: binwidth defaulted to range/30. Use 'binwidth = x' to adjust this.
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8-1.png) 

Mean and median number of steps with imputed data.


```r
c(mean(dd2$steps, na.rm=T), median(dd2$steps, na.rm=T))
```

```
## [1] 10766.19 10766.19
```

## Are there differences in activity patterns between weekdays and weekends?

Difference between weekday and weekend activity. Note that the `weekdays` function is locale dependent so we change locale to 'English' which seems to work on Windows at least...


```r
is.weekend <- function(day) {
    wkday <- weekdays(as.Date(day), T)
    wkday %in% c("Sat", "Sun")
}


Sys.setlocale('LC_TIME', 'English')
```

```
## [1] "English_United States.1252"
```

```r
dm$weekday <- rep("weekday", length(dm$steps))
dm$weekday[is.weekend(dm$date)] <- "weekend"
dm.weekday <- as.factor(dm$weekday)

xx <- dm %>% group_by(weekday, interval) %>% summarise(steps=mean(steps, na.rm=T))

ggplot(xx, aes(x=interval, y=steps)) + geom_line() + facet_wrap(~ weekday, ncol=1)
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10-1.png) 


