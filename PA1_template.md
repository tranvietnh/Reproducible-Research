About
-----

This was the first assignment for the Reproducible Research course in
Coursera’s Data Science Specialization track.

The purpose of the project was to answer a series of questions using
data collected from activity monitoring trackers such as FitBit, Nike
Fuelband, or Jawbone Up. In this project, we practiced loading and
processing data, plotting histograms and time series, and imputing
missing values.

Synopsis
--------

It is now possible to collect a large amount of data about personal
movement using activity monitoring devices such as a Fitbit, Nike
Fuelband, or Jawbone Up. These type of devices are part of the
“quantified self” movement - a group of enthusiasts who take
measurements about themselves regularly to improve their health, to find
patterns in their behavior, or because they are tech geeks. But these
data remain under-utilized both because the raw data are hard to obtain
and there is a lack of statistical methods and software for processing
and interpreting the data.

This assignment makes use of data from a personal activity monitoring
device. This device collects data at 5 minute intervals through out the
day. The data consists of two months of data from an anonymous
individual collected during the months of October and November, 2012 and
include the number of steps taken in 5 minute intervals each day.

Data
----

The data for this assignment was downloaded from the course web site:

-   Dataset:
    [Activitymonitoringdata](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip)
    \[52K\]

The variables included in this dataset are:

-   steps: Number of steps taking in a 5-minute interval (missing values
    are coded as NA)

-   date: The date on which the measurement was taken in YYYY-MM-DD
    format

-   interval: Identifier for the 5-minute interval in which measurement
    was taken

The dataset is stored in a comma-separated-value (CSV) file and there
are a total of 17,568 observations in this dataset.

Loading and preprocessing the data
----------------------------------

First, we download the dataset from
“<a href="https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip" class="uri">https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip</a>”

Then we unzip the data and read the activity dataset in our working
directory.

    library(dplyr)
    library(ggplot2)

    fileurl = "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
    download.file(fileurl, "Activity monitoring Data.zip", mode = "wb")
    unzip("Activity monitoring Data.zip", exdir = ".")
    file <- read.csv("activity.csv", sep = ",")

Struture of Dataset:

    str(file)

    ## 'data.frame':    17568 obs. of  3 variables:
    ##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
    ##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...

The date is in factor format, we change it into Date format. We also
filter to eliminate rows having NA

    file$date <- as.Date(file$date)
    file1 <- file %>% filter(complete.cases(file))
    head(file1)

    ##   steps       date interval
    ## 1     0 2012-10-02        0
    ## 2     0 2012-10-02        5
    ## 3     0 2012-10-02       10
    ## 4     0 2012-10-02       15
    ## 5     0 2012-10-02       20
    ## 6     0 2012-10-02       25

What is mean total number of steps taken per day?
-------------------------------------------------

We caculate sum total steps per day

    totalsteps <- file1 %>%
            group_by(date) %>%
            summarise(steps = sum(steps))

We create plot from data

    hist(totalsteps$steps, xlab = "Number of Steps",
            main = "Total of Steps each day", col = "blue")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-6-1.png)

Calculate and report the mean and median of the total number of steps
taken per day

    data.frame(mean = mean(totalsteps$steps), median = median(totalsteps$steps))

    ##       mean median
    ## 1 10766.19  10765

What is the average daily activity pattern?
-------------------------------------------

Calculate average steps for each interval for all days.

    IntervalSteps <-
            file1 %>% group_by(interval) %>% summarise(steps = mean(steps))

We create plot from data

    ggplot(IntervalSteps, aes(x = interval, y = steps)) + geom_line(col = "blue", size = 1) + labs(x = "Intervals", y = "Average of Steps", title = "The average number of Steps by Interval")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-9-1.png)

Find interval with most average steps

    max(IntervalSteps$steps)

    ## [1] 206.1698

    IntervalSteps[which.max(IntervalSteps$steps),1]

    ## # A tibble: 1 x 1
    ##   interval
    ##      <int>
    ## 1      835

Imputing missing values
-----------------------

The number of rows having steps is NA

    sum(is.na(file$steps))

    ## [1] 2304

Create new dataset, with it, NA is filed in

    fileinputed <- file %>% 
            group_by(interval)%>% 
            mutate(steps = replace(steps, is.na(steps), mean(steps, na.rm = TRUE)))

Calculate the total number of steps taken each day with the imputed
data.

    datafull <- fileinputed %>% 
            group_by(date) %>% 
            summarise(steps = sum(steps))

Create plot from data

    hist(datafull$steps, xlab = "Steps", ylab = "Frequency", main = "Histogram of Total Steps per day")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-14-1.png)

    mean(datafull $steps)

    ## [1] 10766.19

    median(datafull$steps)

    ## [1] 10766.19

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

Create new dataset, with it, seperating date to “Weekday” and “Weekend”

    fileinputed1 <- fileinputed %>% 
            mutate(day = ifelse (weekdays(date) == "Sunday"| weekdays(date) == "Saturday", "weekend", "weekday"))

Turn type day into factor class

    fileinputed1$day <- as.factor(fileinputed1$day)

Caculate average steps group by day and interval

    fileavg <- fileinputed1 %>% 
            group_by(day, interval) %>% 
            summarise(steps = mean(steps))

Create plot from data

    ggplot(fileavg, aes(x = interval, y = steps)) + geom_line() + facet_grid(day ~ .) + labs(title = "Weekday and Weekend Avg Steps")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-18-1.png)

Conclusion
==========

There are some differences in average steps between Weekdays and
Weekends. During weekdays, the person is more active at the start of the
day and less active during the day. Meanwhile, during weekends, the
person is less active at start of the day and more active throughout the
day.

This is probably because the person is commuting to work in the morning
and less active during work hours (sitting at desk). During weekends,
the person does not have to prepare for work and therefore less active
in the mornings, but more active during the day as the person is off
from work.
