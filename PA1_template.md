---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data
### Show any code that is needed to
1. Load the data (i.e. read.csv())
2. Process/transform the data (if necessary) into a format suitable for your analysis

```r
## below code for 1 and 2
unzip(zipfile="./activity.zip")
dataset <- read.csv("./activity.csv", sep=",")
```

## What is mean total number of steps taken per day?
### For this part of the assignment, you can ignore the missing values in the dataset.
1. Calculate the total number of steps taken per day

```r
totalSteps <- tapply(dataset$steps, dataset$date, sum)
```
2. If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day

```r
hist(totalSteps,xlab="total number of steps taken each day", main="Histogram of the total number of steps", breaks = 20)
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

3. Calculate and report the mean and median of the total number of steps taken per day

```r
mean(totalSteps,na.rm=TRUE)
```

```
## [1] 10766.19
```

```r
median(totalSteps,na.rm=TRUE)
```

```
## [1] 10765
```


## What is the average daily activity pattern?
1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
avgSteps <- tapply(dataset$steps, dataset$interval, mean, na.rm=TRUE)
interval = dataset$interval[1:288]
plot(interval, avgSteps, type="l",xlab="5-minute interval",ylab="Average number of steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
interval[avgSteps==max(avgSteps)]
```

```
## [1] 835
```

## Imputing missing values
### Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.
1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
sum(is.na(dataset$steps))
```

```
## [1] 2304
```

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
## below code for 2 and 3
dataset2 <- dataset
### replace missing values using the mean for that 5-minute interval:
for ( i in 1:17568) {
	if (is.na(dataset$steps[i])) { 
		j <-  i %% 288
		if (j != 0) {
			dataset2$steps[i] <- avgSteps[j]  ## j cannot be 0
			} else dataset2$steps[i] <- avgSteps[288]
		}
	}
print("Take a look at the top a few lines of the new dataset")
```

```
## [1] "Take a look at the top a few lines of the new dataset"
```

```r
head(dataset2) 
```

```
##       steps       date interval
## 1 1.7169811 2012-10-01        0
## 2 0.3396226 2012-10-01        5
## 3 0.1320755 2012-10-01       10
## 4 0.1509434 2012-10-01       15
## 5 0.0754717 2012-10-01       20
## 6 2.0943396 2012-10-01       25
```

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
totalSteps2 <- tapply(dataset2$steps, dataset2$date, sum)
hist(totalSteps2,xlab="total number of steps taken each day", main="Histogram of the total number of steps w/o NA", breaks = 20)
```

![](PA1_template_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

```r
mean(totalSteps2,na.rm=TRUE)
```

```
## [1] 10766.19
```

```r
median(totalSteps2,na.rm=TRUE)
```

```
## [1] 10766.19
```
#### To answer the questions, it depends on the strategy for filling in all of the missing values. After imputing missing data, the values differ from the estimates from the first part of the assignment. The impact of imputing missing data on the estimates of the total daily number of steps is the frequency of total number of steps taken each day between 10000 to 11000 increased.

## Are there differences in activity patterns between weekdays and weekends?
### For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.
1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```r
wkday <- c("Monday","Tuesday","Wednesday","Thursday","Friday")
factor <- weekdays(as.Date(dataset2$date)) %in% wkday
wkdayORwkend <- vector()
for (i in 1:17568) {
if (factor[i]) {
wkdayORwkend[i] <- "weekday"
} else wkdayORwkend[i] <- "weekend"
}
dataset2new <- data.frame(dataset2,wkdayORwkend)
print("Take a look at the top a few lines of the new dataset")
```

```
## [1] "Take a look at the top a few lines of the new dataset"
```

```r
head(dataset2new)
```

```
##       steps       date interval wkdayORwkend
## 1 1.7169811 2012-10-01        0      weekday
## 2 0.3396226 2012-10-01        5      weekday
## 3 0.1320755 2012-10-01       10      weekday
## 4 0.1509434 2012-10-01       15      weekday
## 5 0.0754717 2012-10-01       20      weekday
## 6 2.0943396 2012-10-01       25      weekday
```

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.

```r
dataBywkdays <- aggregate(steps ~ wkdayORwkend+interval, dataset2new, mean)
library(lattice)
xyplot(steps~interval | wkdayORwkend, data=dataBywkdays, layout=c(1,2), type="l", xlab="Interval", ylab="Number of steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png)<!-- -->
