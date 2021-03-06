**Analysis of Personal Movement Using Activity Monitoring Devices**
=====================================================================

### **Introduction**

It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These type of devices are part of the "quantified self" movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

### **Loading and preprocessing the data**

#### 1. Load the data

```r
Sys.setlocale("LC_TIME", "English")
```

```
## [1] "English_United States.1252"
```

```r
setwd("C:/Users/m.hernandez.moreno/Desktop/Curso Data Science/05. Reproducible research/Project1")
data <- read.csv("activity.csv", header = TRUE, sep = ",", na.strings = "NA")
```

#### 2. Process/transform the data into a format suitable for the analysis


```r
summary(data)
```

```
##      steps                date          interval     
##  Min.   :  0.00   2012-10-01:  288   Min.   :   0.0  
##  1st Qu.:  0.00   2012-10-02:  288   1st Qu.: 588.8  
##  Median :  0.00   2012-10-03:  288   Median :1177.5  
##  Mean   : 37.38   2012-10-04:  288   Mean   :1177.5  
##  3rd Qu.: 12.00   2012-10-05:  288   3rd Qu.:1766.2  
##  Max.   :806.00   2012-10-06:  288   Max.   :2355.0  
##  NA's   :2304     (Other)   :15840
```

```r
str(data)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

Converting "date" variable to date class:

```r
data$date <- as.Date(data$date, format = "%Y-%m-%d")
str(data)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Date, format: "2012-10-01" "2012-10-01" ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```


### **What is mean total number of steps taken per day?**

#### 1. Calculate the total number of steps taken per day

Subsitting the dataset to ignore missing values:

```r
data_no_NA <- data[with(data, { !(is.na(steps)) } ), ]
```

Total number of steps taken each day:

```r
steps_day <- aggregate(steps ~ date, data_no_NA, sum)
```

#### 2. Make a histogram of the total number of steps taken each day

```r
hist(steps_day$steps, col="grey", xlab = "Number of Steps", main= "Total Steps Taken Each Day")
```

![plot1](figures/plot1.png)

#### 3. Calculate the mean and median of the total number of steps taken per day

The mean of the total number of steps taken per day is:

```r
mean(steps_day$steps)
```

```
## [1] 10766.19
```
and the median is:

```r
median(steps_day$steps)
```

```
## [1] 10765
```

### **What is the average daily activity pattern?**

#### 1. Make a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

Calculating average steps per interval for all days:

```r
avg_steps_interval <- aggregate(steps ~ interval, data, mean)
```

Plotting the time series:

```r
plot(avg_steps_interval$interval, avg_steps_interval$steps, type='l', col=1, main="Average Steps per Day by Interval", xlab="Time Intervals", ylab="Average number of steps")
```

![plot2](figures/plot2.png)

#### 2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

Identifying the interval index which has the highest average steps:

```r
interval_max <- which.max(avg_steps_interval$steps)
interval_max_i <- avg_steps_interval[interval_max, ]$interval
interval_max_s <- round(avg_steps_interval[interval_max, ]$step)
```
**The 835-th 5-minute interval contains the maximum number of steps (206).**


### **Imputing missing values**

#### 1. Calculate and report the total number of missing values in the dataset.

```r
NA_steps <- sum(is.na(as.character(data$steps)))
NA_interval <- sum(is.na(as.character(data$interval)))
NA_date <- sum(is.na(as.character(data$date)))
NA_total <- sum(!complete.cases(data))
```

**There are 2304 missing steps values, 0 missing interval values and 0 missing date values, so the total number of missing values is 2304.**


#### 2. Devise a strategy for filling in all of the missing values in the dataset.

Using mean for the day compute missing values:

```r
StepsAverage <- aggregate(steps ~ interval, data, mean)
fillNA <- numeric()
for (i in 1:nrow(data)) {
    obs <- data[i, ]
    if (is.na(obs$steps)) {
        steps <- subset(StepsAverage, interval == obs$interval)$steps
    } else {
        steps <- obs$steps
    }
    fillNA <- c(fillNA, steps)
}
```

#### 3. Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
data2 <- data
data2$steps <- fillNA
```

#### 4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day.


```r
steps_day2 <- aggregate(steps ~ date, data2, sum)
hist(steps_day2$steps, main = paste("Total Steps Taken Each Day"), col="red", xlab="Number of Steps")
#Create Histogram to show difference. 
hist(steps_day$steps, main = paste("Total of Steps Taken Each Day"), col="grey", xlab="Number of Steps", add=T)
legend("topright", c("New dataset", "Dataset"), col=c("red", "grey"), lwd=10)
```

![plot3](figures/plot3.png)

The mean of the total number of steps taken per day is:

```r
mean(steps_day2$steps)
```

```
## [1] 10766.19
```
and the median is:

```r
median(steps_day2$steps)
```

```
## [1] 10766.19
```

**Do these values differ from the estimates from the first part of the assignment?**

The mean is the same but the median have a small variance:


```r
median1 <- median(steps_day$steps)
median2 <- median(steps_day2$steps)
median2 - median1
```

```
## [1] 1.188679
```

**What is the impact of imputing missing data on the estimates of the total daily number of steps?**

The impact has the biggest effect on the 10000 - 150000 step interval and changes frequency from 27.5 to 35.


### **Are there differences in activity patterns between weekdays and weekends?**

#### 1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.



```r
data2$day <- as.factor(ifelse(weekdays(data2$date) %in% c("Saturday","Sunday"),"Weekend", "Weekday"))
```


#### 2. Make a panel plot containing a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

Calculating the average number of steps taken, averaged across all weekday days or weekend days: 

```r
avg_steps_interval_d <- aggregate(steps ~ interval + day, data2, mean)
```


Plotting the time series:

```r
library(lattice)

xyplot(steps ~ interval|day,data=avg_steps_interval_d, main="Average Steps per Day by Interval",xlab="Interval", ylab="Steps",layout=c(1,2), type="l")
```

![plot4](figures/plot4.png)

