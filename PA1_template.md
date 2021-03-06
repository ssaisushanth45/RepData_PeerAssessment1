# Reproducible Research: Peer Assessment 1
Sai S Sampathkumar  
15 June 2017  



## Week 2 Reproducible Research Assignment - Coursera

The following in an analysis of data from a personal activity monitoring device like fitbit etc. Data can be downloaded from <https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip>.

The device has collected data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and includes the number of steps taken in 5 minute intervals each day. Hope this helps.

## Loading the data


```r
setwd("C:/Users/ssais/Documents/11. Coursework/1. Data Science Specialization/5. Reproducible Research/2. Data/")
activitydata <- read.csv("./activitydata/activity.csv")
```

## Preprocessing data

* New variable 'dateformatted' added to store 'date' variable as date format for ease of use later (if necessary) using the 'lubridate' package in R. 


```r
library(lubridate)
```

```
## Warning: package 'lubridate' was built under R version 3.3.3
```

```
## 
## Attaching package: 'lubridate'
```

```
## The following object is masked from 'package:base':
## 
##     date
```

```r
activitydata$dateformatted <- ymd(activitydata$date)
```

## What is the mean total number of steps taken per day?

This helps understand the overall pattern of how many steps the user has taken daily on average. Since the data by nature can have outliers (user may record more steps on some days than others), we present both mean and median steps taken per day. 

First, we create a table that has only total steps per day. We already know there are 288 (12*24) five minute intervals in a day. For now, we ignore any missing data the dataset may have; read: we'll deal it later.


```r
library(dplyr)
```

```
## Warning: package 'dplyr' was built under R version 3.3.3
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:lubridate':
## 
##     intersect, setdiff, union
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
Total <- activitydata %>%
        group_by(dateformatted) %>%
        summarize(Total = sum(steps, na.rm = TRUE)) %>%
        arrange(dateformatted)
```


The following is a histogram of total steps taken per day using GGPLOT2:
I have also added vertical dashed lines to show the mean (red) and median (blue).


```r
library(ggplot2)
```

```
## Warning: package 'ggplot2' was built under R version 3.3.3
```

```r
g <- ggplot(Total, aes(x = Total))
g + geom_histogram(binwidth = 2000, colour = "black", fill = "white") +
        xlab("Total # Steps") + ylab("Frequency") +
        ggtitle("Histogram - Total Number of Steps Taken Each Day") +
        theme(axis.text.x = element_text(angle = 90, hjust = 1)) +
        geom_vline(aes(xintercept=mean(Total, na.rm=T)),color="red", linetype="dashed", size=1) + 
        annotate("text", x=7000, y=11, label= "Mean = 9354") +
        geom_vline(aes(xintercept=median(Total, na.rm=T)),color="blue", linetype="dashed", size=1) + 
        annotate("text", x=14000, y=11, label= "Median = 10395")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->


## What is the average daily activity pattern?


To compute the average daily activity pattern, I consolidated the data by time interval (x) and averaged the number of steps taken across all days(y). The data is presented below in a time series plot.


```r
Daily <- activitydata %>%
        group_by(interval) %>%
        summarize(avgsteps = mean(steps, na.rm = TRUE))

g2 <- ggplot(Daily, aes(x = interval, y = avgsteps))
g2 + geom_line() +
        xlab("5 Minute Time Intervals") + ylab("Avg Steps in All 61 Days") +
        ggtitle("Average Daily Activity Pattern") +
        theme(axis.text.x = element_text(angle = 90, hjust = 1)) +
        annotate("text", x=835, y=200, label= "Max (@835) = 206.17 Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

## Imputing Missing Values

This section has four parts:

* Calculating and reporting the total number of missing values in the dataset (i.e. the total number of rows with NAs) - Shown below as 2304.
* Filling in all of the missing values in the dataset using the mean for that 5-minute interval.
* Creating a new dataset that is equal to the original dataset but with the missing data filled in - Shown in dataset "actdata_fill"
* Making a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. 


```r
# Total # Missing Values
misval <- which((is.na(activitydata$steps)))
print(length(misval))
```

```
## [1] 2304
```

```r
# Create New Dataset actfill_data with NAs filled with mean for that interval
actdata_fill <- activitydata %>%
        group_by(interval) %>%
        mutate(avgsteps = mean(steps, na.rm =TRUE))
for (i in 1:length(actdata_fill$steps)) {
        if (is.na(actdata_fill$steps[i])){
                actdata_fill$steps[i] <- actdata_fill$avgsteps[i]
        }
}

actdata_fill <- as.data.frame(actdata_fill)

# Recalculate Total Steps per Day
Total_fill <- actdata_fill %>%
        group_by(dateformatted) %>%
        summarize(Total = sum(steps, na.rm = TRUE)) %>%
        arrange(dateformatted)

# Histogram (again) for imputed dataset
g <- ggplot(Total_fill, aes(x = Total))
g + geom_histogram(binwidth = 2000, colour = "black", fill = "white") +
        xlab("Total # Steps") + ylab("Frequency") +
        ggtitle("Histogram - Total Number of Steps Taken Each Day - Imputed") +
        theme(axis.text.x = element_text(angle = 90, hjust = 1)) +
        geom_vline(aes(xintercept=mean(Total, na.rm=T)),color="red", linetype="dashed", size=1) + 
        annotate("text", x=7000, y=11, label= "Mean = 10766") +
        geom_vline(aes(xintercept=median(Total, na.rm=T)),color="blue", linetype="dashed", size=1) + 
        annotate("text", x=14000, y=11, label= "Median = 10766")                                
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

Note: Now, with imputed data, we now have median and mean equal to 10766. By imputing the data, Total # Steps (imputed) = 656737.5 compared to Total # Steps (w/NAs) = 570608 -> addition of 86129.5 steps (15% addition to activity - pretty high number for activity missed with NAs)


## Are there differences in activity patterns between weekdays and weekends?

We will use the imputed new data set for this part. Function weekdays() will help in creating a new variable to separate and view the activity patterns by weekdays and weekends.



```r
Daily2 <- actdata_fill %>%
        mutate(dayname=weekdays(dateformatted, abbreviate = TRUE))


for (i in seq(along = Daily2$dayname))
 { if (Daily2$dayname[i] == "Sat") Daily2$daydesc[i] <- "weekend"
else if (Daily2$dayname[i] == "Sun") Daily2$daydesc[i] <- "weekend"
else Daily2$daydesc[i] <- "weekday"}


Daily2 <- Daily2 %>%
        group_by(daydesc, interval) %>%
        summarize(avgsteps = mean(steps, na.rm = TRUE))

g2 <- ggplot()
g2 + geom_line(data = subset(Daily2, daydesc == "weekday"), aes(x = interval, y = avgsteps)) + geom_line(data = subset(Daily2, daydesc == "weekend"), aes(x = interval, y = avgsteps)) + coord_cartesian(ylim = c(0, 250)) +
        facet_grid(daydesc~., scales = "free_y") +
        xlab("5 Minute Time Interval") + ylab("Number of Steps") +
        ggtitle("Average Daily Activity Pattern Weekday vs Weekend") +
        theme(axis.text.x = element_text(angle = 90, hjust = 1)) 
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

Clearly, there is a difference between average steps taken between Weekends and Weekdays, as one might guess. On weekdays, we see four pronounced spikes possibly during morning commute, lunch, tea, evening commute. On weekends, the activity is more uniform and spread throughout the day. The wakeup pattern between the 2 groups is also quite evident without the contribution steps of the early risers during weekdays.

Max (Weekday) = 230.38 Steps vs. Max (Weekend) = 166.64

Thanks for your time and effort looking through this report. Good luck!
