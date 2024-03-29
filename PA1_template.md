---
title: "PA1_template.Rmd"
author: "Matthew Lerner"
date: "Saturday, September 13, 2014"
output: html_document
---
# Loading and preprocessing the data

## 1. Load the data 
We do this using the R Code below.


```r
activity <- read.csv("~/activity.csv")
```

## 2. Process/transform the data (if necessary) into a format suitable for your analysis
The Data is already in a suitable format.  The data will be manipulated when necessary, based on the task at hand

# What is mean total number of steps taken per day?

## 1. Make a histogram of the total number of steps taken each day
We use the aggregate function to create a new dataset, that sums the steps based on date.  Then we use the hist function to plot the sums in a histogram.


```r
z <- aggregate(steps ~ date, data=activity , FUN=sum)
hist(z$steps)
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2.png) 

## 2. Calculate and report the mean and median total number of steps taken per day
The summary function on z$steps will give us this information. 

```r
summary(z$steps)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##      41    8840   10800   10800   13300   21200
```

# What is the average daily activity pattern?

## 1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis).
We again use the aggregate function to create a new dataset, this time using the mean function to find the average steps for each interval. We then plot the averages for each interval as a time series. 

```r
y <- aggregate(steps ~ interval, data=activity , FUN=mean)
plot(steps~interval, data=y, type = "l")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 

## 2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?
We look in our aggregate subset, y, to find the max value of steps.  We then subset y to give the exact row that the max is in, thus indicating what interval the max belongs to. 

```r
y[y$steps==max(y$steps),]
```

```
##     interval steps
## 104      835 206.2
```

## Imputing missing values
# Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

# 1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)
This fairly straight-forward. The is.na function will, go through a vector and a create a new logical vector, indicate which data points are na ("TRUE") and which ones are not ("FALSE"). We then sum this logical vector (1 for "TRUE", 0 for "FALSE").

```r
sum(is.na(activity$steps))
```

```
## [1] 2304
```

# 2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
We will fill the NAs with the closest known value, using the "na.locf" function from the "zoo" package. 

# 3. Create a new dataset that is equal to the original dataset but with the missing data filled in.
We first load the zoo package. Then we create a new "activity" dataset, "activity2", so we do not mess with our original while manipulating the data. We then use the "na.locf" which goes through the vector, top to bottom, and replaces na values with the last non-na value.  If there were no non-na before the nas, then the value will be unchanged.  To fix this we run the vector we just had back through the na.locf in reverse, bottom to top, specifying fromLast = TRUE, within the function.

```r
library(zoo)
activity2 <- activity
activity2$steps <-na.locf(na.locf(activity2$steps,na.rm=FALSE),fromLast = TRUE, na.rm=FALSE)
```

# 4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day.  What is the impact of imputing missing data on the estimates of the total daily number of steps?
We use the aggregate function to create a new dataset, that sums the steps based on date.  Then we use the hist function to plot the sums in a histogram.aggregate based on date. We also use the summary function to get the mean and median.

```r
x <- aggregate(steps ~ date, data=activity2 , FUN=sum)
hist(x$steps)
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8.png) 

```r
summary(x$steps)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##       0    6780   10400    9350   12800   21200
```

## Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?
Comparing to our original values (shown below), there are many more days with smaller amounts of total steps, and the mean and median are obviously quite large.  Therefore, the missing data can have a large effect on the estimates. 

```r
hist(z$steps)
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9.png) 

```r
summary(z$steps)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##      41    8840   10800   10800   13300   21200
```


## Are there differences in activity patterns between weekdays and weekends?

# For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

# 1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.
For this we will use the "isWeekend"" function in the "timeDate" package.  We first load the package using the library function. We then reformat the dates to timeDate format, using the "as.timeDate" function. Then we create a new vector named "Weekday", which identifys the weekend dates, using the is "isWeekend"" function, and insert it into the "activity" data frame. Then we sub out, within that vector, the "TRUE"'s for "Weekend", and the "FALSE"s for"Weekday."

```r
library(timeDate)
activity$date<-as.timeDate(activity$date)
activity$weekday<-isWeekend(activity$date)
activity$weekday <- gsub("TRUE", "Weekend",activity$weekday)
activity$weekday <- gsub("FALSE", "Weekday",activity$weekday)
```

We use the aggregate function to create 2 new datasets, one for weekdays and one for weekends, using the mean function to find the average steps for each interva. We then make 2 plots. 


```r
w <- aggregate(steps ~ interval, data=activity[activity$weekday=="Weekday", ] , FUN=mean)
v <- aggregate(steps ~ interval, data=activity[activity$weekday=="Weekend", ] , FUN=mean)
plot(steps~interval, data=w, type = "l")
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-111.png) 

```r
plot(steps~interval, data=v, type = "l")
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-112.png) 
