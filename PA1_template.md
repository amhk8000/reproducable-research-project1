Coursera Reproduciable Research Project 1
=========================================

Introduction
------------

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

The data for this assignment can be downloaded from the course web site:

Dataset: Activity monitoring data \[52K\]

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations.

Variables included in this dataset are:

**steps:** Number of steps taking in a 5-minute interval (missing values are coded as NA).

**date:** The date on which the measurement was taken in YYYY-MM-DD format.

**interval:** Identifier for the 5-minute interval in which measurement was taken

Assignment
----------

This assignment involves creating an R markdown document that can be processed with Knitr to be displayed in HTML format. The document will answer question about the Activity monitoring data by loading and processing data. the document will contains the answers to the questions and also the code used to load the data.

Q1. Loading and preprocessing the data
--------------------------------------

Load the data

``` r
activity <- read.csv("activity.csv", header = TRUE)
```

Format date column to date type and save as a data frame.

``` r
activity$date <- as.Date(activity$date, "%Y-%m-%d")
activity <- as.data.frame(activity)
```

Q2. What is mean total number of steps taken per day?
-----------------------------------------------------

Calculate the total number of steps taken per day (Ignore missing values).

``` r
stepsperday <- tapply(activity$steps, activity$date, sum)
```

Make a histogram of the frequency of total number of steps per day

``` r
hist(stepsperday, 
     col = "red",
     xlab = "Daily Total Steps", 
     ylab = "Frequency ",
     main = "Histogram: Frequency - Daily Total Steps")
```

![](PA1_template_files/figure-markdown_github/unnamed-chunk-4-1.png)

Calculate and report the mean and median number of steps taken each day

``` r
meansteps <- mean(stepsperday, na.rm = TRUE)
mediansteps <- median(stepsperday, na.rm = TRUE)
```

so the mean is 1.076618910^{4} and median is 10765 steps.

Q3. What is the average daily activity pattern?
-----------------------------------------------

Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

Average number of steps taken per interval

``` r
avgstepsinterval <- tapply(activity$steps, activity$interval, mean, na.rm = TRUE)
```

Time series plot of the average number of steps taken

``` r
plot(as.numeric(names(avgstepsinterval)), 
     avgstepsinterval, 
     xlab = "Interval", 
     ylab = "Steps", 
     main = "Average Daily Steps Pattern", 
     type = "l")
```

![](PA1_template_files/figure-markdown_github/unnamed-chunk-7-1.png)

Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

``` r
maxavgInterval <- names(sort(avgstepsinterval, decreasing = TRUE)[1])
maxavgSteps <- sort(avgstepsinterval, decreasing = TRUE)[1]
```

The 5-minuter interval at 835 contains the maximum number of steps at 206.1698113.

Q4. Imputing missing values
---------------------------

There are a number of days/intervals where there are missing values (coded as *NA*). The presence of missing days may introduce bias into some calculations or summaries of the data.

### Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with *NAs*)

``` r
NA.vals <- sum(is.na(activity$steps))
```

The total number of missing values are 2304

### A strategy used for filling in all of the missing values.

The mean for the 5-minute interval is used to fill missing values.

### Create a new dataset that is equal to the original dataset but with the missing data filled in.

Fill missing values with the mean for the 5-minute interval

``` r
nasteps <- is.na(activity$steps)
meaninterval <- tapply(activity$steps, activity$interval, mean, na.rm=TRUE, simplify = TRUE)

activityimputed <- data.frame(activity)
activityimputed$steps[nasteps] <- meaninterval[(as.character(activityimputed$interval[nasteps]))]
```

### Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day.

create a new dataset and find total steps taken with the missing data filled in

``` r
totalstepsperday <- tapply(activityimputed$steps, activityimputed$date, sum)
```

Make a histogram of the total number of steps taken each day

``` r
hist(totalstepsperday, 
     col = "red",
     xlab = "Number of Steps", 
     ylab  = "Total Daily steps",
     main = "Histogram: Steps per Day (Imputed data)"
    )
```

![](PA1_template_files/figure-markdown_github/unnamed-chunk-12-1.png)

Calculate the mean and median of dataset with the missing values filled in.

``` r
meanimputed <- mean(totalstepsperday, na.rm = TRUE)
medianimputed <- median(totalstepsperday, na.rm = TRUE)
```

### Do the imputed values differ from the estimates from the first part of the assignment?

\`\`\` For the imputer data mean is 1.076618910^{4} and median is 1.076618910^{4}

### What is the impact of imputing missing data on the estimates of the total daily ?

Comparing original and imputed datasets, mean stays same as for median there is some difference but with the imputed median closer to the mean.Using the mean to imputer missing values resulted with higher frequency counts around the mean.

Q5. Plot and compare the average number of steps taken per 5-minute interval across weekdays and weekends
---------------------------------------------------------------------------------------------------------

Create a new variable with two levels, weekday and weekend based on the day of the week using the dataset with filled-in missing values

``` r
activityimputed$day <- ifelse(weekdays(as.Date(activityimputed$date)) %in% c("Saturday","Sunday"), "weekend", "weekday")
```

Calculate the average number of steps taken, for weekdays

``` r
library(dplyr)
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
stepsall <-  activityimputed[, c('interval','day','steps')]

stepsgroup <-stepsall %>%  filter(day == 'weekday')  %>% 
              group_by(day, interval) %>%                            # multiple group columns
            summarise(steps = mean(steps))
stepsweekday <-   tapply(stepsgroup$steps, stepsgroup$interval, mean, na.rm = TRUE)
```

Time series plot of the average number of steps taken for weekdays

``` r
plot(as.numeric(names(stepsweekday)), 
     stepsweekday, 
     xlab = "Interval", 
     ylab = "Average Steps", 
     main = "Activity Pattern (Weekdays)", 
     type = "l")
```

![](PA1_template_files/figure-markdown_github/unnamed-chunk-16-1.png)

Calculate the total number of imputed steps taken for weekends

``` r
stepsgroup <-stepsall %>%  filter(day == 'weekend')  %>% 
              group_by(day, interval) %>%                            # multiple group columns
            summarise(steps = mean(steps))
stepsweekend <-   tapply(stepsgroup$steps, stepsgroup$interval, mean, na.rm = TRUE)
```

Time series plot of the average number of steps taken for weekends

``` r
plot(as.numeric(names(stepsweekend )), 
     stepsweekend, 
     xlab = "Interval", 
     ylab = "Average Steps", 
     main = "Activity Pattern (Weekends)", 
     type = "l")
```

![](PA1_template_files/figure-markdown_github/unnamed-chunk-18-1.png)

### Are there differences in activity patterns between weekdays and weekends?

Weekday activities are done less frequently than weekends.
