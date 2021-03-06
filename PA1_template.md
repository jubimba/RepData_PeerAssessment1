# Activity Monitoring Data Analysis
Julia Fischer  
August 13, 2017  

## Introduction
This analysis uses 2 months of anonymous data from a personal activity monitoring device. The device collects data at 5 minute intervals throughout the day. The data was collected during October and November, 2012. The analysis done here will be explained step by step, so you can reproduce it.

## Data
The data for this assignment was downloaded from the course web site:

* *Dataset*: [Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip) [52K]

The variables included in this dataset are:

* *steps*: Number of steps taking in a 5-minute interval (missing values are coded as NA)  

* *date*: The date on which the measurement was taken in YYYY-MM-DD format  

* *interval*: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.

### Load the set 
Download the data, unzip it, read the .csv file, and print the file name; we are assuming that the zip file contains only one .csv file.


```r
download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip", 
              destfile = "./activity.zip")
unzip("./activity.zip", exdir = "./data")
files <- list.files("./data/")
filename <- paste("./data", files[1], sep="/")
actdata <- read.csv(filename)
print(filename)
```

```
## [1] "./data/activity.csv"
```


### Load required R packages 


```r
library("dplyr")
```

```
## Warning: package 'dplyr' was built under R version 3.4.1
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
library("ggplot2")
```

```
## Warning: package 'ggplot2' was built under R version 3.4.1
```


### Mean of total number of steps taken per day
For this part of the analysis, the missing values in the dataset were ignored.
First, a histogram of the total number of steps taken each day is drawn, second, the mean and median total number of steps taken per day is shown.

1. Calculate the number of steps per day and show it in a histogram


```r
stepsperday <- aggregate(steps ~ date, actdata, sum, na.rm=TRUE)
hist(stepsperday$steps, breaks=20, col=rgb(0.2,0.8,0.5,0.5) , border=F , main="Histogram of Steps per Day", xlab="Steps per Day", xlim=c(0,25000))
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

2. Calculate mean and median of the total number of steps taken per day

```r
stepssummary <- summary(stepsperday)
stepsmean <- stepssummary[4,2]
stepsmedian <- stepssummary[3,2]
```

- **Mean   :10766  **
- **Median :10765  **

### Average daily activity pattern

Time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

1. Create a dataframe with the average number of steps taken per every 5-minutes interval, across all days.

```r
stepsinterval <- aggregate(actdata$steps, by = list(interval = actdata$interval),FUN=mean, na.rm=TRUE)
colnames(stepsinterval) <- c("interval", "stepsavg")
```
  
  
2. Show it in a plot


```r
plot(stepsinterval, type="l", lwd=2, bty="n", xlim=c(0,2500), col=rgb(0.2,0.4,0.6,0.8), main="Average daily activity pattern", xlab="Interval", ylab="Average number of steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

3. Determine the interval with the maximum number of steps


```r
maxinterval <- stepsinterval[which.max(stepsinterval$steps),]
```

The interval which contains the maximum number of steps is **835** with **206.1698113** steps.


### Imputing missing values

Now we will calculate mean and median but with filling in data for missing observations. 

1. Calculate the total number of missing values in the dataset (i.e. the total number of rows with NAs)


```r
sumna <- sum(is.na(actdata))
```
The total number of missing values is **2304**.

2. The strategy to fill in the NAs will be to use the 5-minutes interval mean. To do that, we create an array with the means of the 5-minutes intervals. 


```r
stepsinterval_array <- array(data = as.vector(stepsinterval$stepsavg), dimnames = list(as.list(stepsinterval$interval)))
```

3. Now we create a new dataframe and substitute the NAs of the new dataframe with the respective 5-minutes mean.


```r
actdata_nona <- actdata
nas <- is.na(actdata_nona$steps)

actdata_nona$steps[nas] <- stepsinterval_array[as.character(actdata_nona$interval[nas])]
```

4. We now make a histogram of the total number of steps taken each day and calculate and report the mean and median total number of steps taken per day. 


```r
stepsperday_nona <- aggregate(steps ~ date, actdata_nona, sum)
hist(stepsperday_nona$steps, breaks=20, col=rgb(0.2,0.8,0.5,0.5) , border=F , main="Histogram of Steps per Day (missing data filled by mean)", xlab="Steps per Day", xlim=c(0,25000))
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png)<!-- -->

2. Calculate mean and median of the total number of steps taken per day

```r
stepssummary_nona <- summary(stepsperday_nona)
stepsmean_nona <- stepssummary_nona[4,2]
stepsmedian_nona <- stepssummary_nona[3,2]
```

- **Mean   :10766  **
- **Median :10766  **

The mean and the median are now exactly the same. 

### Differences in activity patterns between weekdays and weekends

We will now analyse the differences in activity between weekdays and weekends. For this we will use the weekdays() function and the dataset with the filled-in missing values. 

1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day. For this, we will create an array showing the day type, and then add the day type to our dataset. 


```r
pxdate <- strptime(actdata_nona$date,"%Y-%m-%d")
nameofday <- weekdays(pxdate)
typeofday <- replicate(length(nameofday),'Weekday')
typeofday[grep('Saturday',nameofday)] <- 'Weekend'
typeofday[grep('Sunday',nameofday)] <- 'Weekend'
actdata_nona$typeofday <- typeofday
```


2. Now we make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). 


```r
actdata_nona_typeofday <- actdata_nona %>% group_by(interval, typeofday) %>% summarise(steps = mean(steps))
```



```r
ggplot(actdata_nona_typeofday, aes(x=interval, y=steps)) + facet_wrap(~ typeofday, nrow=2, ncol=1) +
        labs(x="Interval", y="Number of steps") + geom_line(color="#0072B2") + theme_bw()
```

![](PA1_template_files/figure-html/unnamed-chunk-15-1.png)<!-- -->

One can see from the plots, that the weekend has the highest peak, assuming that the person is doing some weekend sports. During the week the activity is more evenly distributed, probably due to a daily work routine.
