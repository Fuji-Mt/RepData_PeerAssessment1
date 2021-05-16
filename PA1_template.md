---
title: 'Reproducible Research: Peer Assessment 1'
output:
  html_document:
    keep_md: yes
  word_document: default
---


## Loading and preprocessing the data

### Load the data
Load the data from the following [URL](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip) and read the data from it by "read.csv".

```r
filename1 <- "repdata_data_activity.zip"

#checking whether zip file exists or not.
if (!file.exists(filename1)){
  fileURL <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
  download.file(fileURL, filename1)
}  

filename2 <- "activity.csv"

#checking whether zip file is unzip or not
if (!file.exists(filename2)) { 
  unzip(filename2) 
}

#Read the original data  
data <- read.csv(filename2)
head(data)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
## 4    NA 2012-10-01       15
## 5    NA 2012-10-01       20
## 6    NA 2012-10-01       25
```


### Process the original data
Using "dplyr" library, change the data format by "tbl_df". And it is named *data2*. Output is confirmed by "head" function.

```r
library(dplyr)
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
data2 <- tbl_df(data)
```

```
## Warning: `tbl_df()` is deprecated as of dplyr 1.0.0.
## Please use `tibble::as_tibble()` instead.
## This warning is displayed once every 8 hours.
## Call `lifecycle::last_warnings()` to see where this warning was generated.
```

```r
head(data2)
```

```
## # A tibble: 6 x 3
##   steps date       interval
##   <int> <chr>         <int>
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
## 4    NA 2012-10-01       15
## 5    NA 2012-10-01       20
## 6    NA 2012-10-01       25
```

## What is mean total number of steps taken per day?

1. Calculate the total number of steps taken per day.

*data2* is grouped by *date*. And the summation of *steps* per day is calculated and saved into *df.total_steps*. Output is confirmed by "head" function. 


```r
data2_date <- group_by(data2, date)
df.total_steps <- summarise(data2_date, total_steps = sum(steps))
#summarize(data2_date, mean.steps = mean(steps, na.rm = TRUE))
head(df.total_steps)
```

```
## # A tibble: 6 x 2
##   date       total_steps
##   <chr>            <int>
## 1 2012-10-01          NA
## 2 2012-10-02         126
## 3 2012-10-03       11352
## 4 2012-10-04       12116
## 5 2012-10-05       13294
## 6 2012-10-06       15420
```

2. Make a histogram of the total number of steps taken per day.

Using *df.total_steps*, histogram is made.

```r
with(df.total_steps,
         hist(df.total_steps$total_steps,
         xlab = "Total steps taken per day", 
         main = "Total steps taken per day"))
```

![](PA1_template_files/figure-html/makehist-1.png)<!-- -->

```r
# Save png file
dev.copy(png, file = "histogram.png")
```

```
## png 
##   3
```

```r
dev.off()
```

```
## png 
##   2
```

3. Calculate and report the mean and median of the total number of steps taken per day.

The mean and median of the steps taken per day is calculated with *df.total_steps*. And the results are pasted. During this calculation, NA data is removed.

```r
mean.steps <- mean(df.total_steps$total_steps, na.rm = TRUE)
median.steps <- median(df.total_steps$total_steps, na.rm = TRUE)
paste("The mean of the total number of steps taken per day is", mean.steps)
```

```
## [1] "The mean of the total number of steps taken per day is 10766.1886792453"
```

```r
paste("The median of the the total number of steps taken per day is", median.steps)
```

```
## [1] "The median of the the total number of steps taken per day is 10765"
```

## What is the average daily activity pattern?
Make a time series plot of the 5-minute interval (x-axis) for each date.

```r
library(ggplot2)
ggplot(data2, aes(x = interval, y = steps, color = date)) +
  geom_line()
```

```
## Warning: Removed 2304 row(s) containing missing values (geom_path).
```

![](PA1_template_files/figure-html/interval-1.png)<!-- -->

```r
dev.copy(png, file = "interval.png")
```

```
## png 
##   3
```

```r
dev.off()
```

```
## png 
##   2
```

the average number of steps taken, averaged across all days (y-axis).  


```r
data2_interval <- group_by(data2, interval)
df.ave_steps <- summarise(data2_interval, averaged_steps = mean(steps, na.rm = TRUE))
head(df.ave_steps)
```

```
## # A tibble: 6 x 2
##   interval averaged_steps
##      <int>          <dbl>
## 1        0         1.72  
## 2        5         0.340 
## 3       10         0.132 
## 4       15         0.151 
## 5       20         0.0755
## 6       25         2.09
```


To find which 5-minute interval contains maximum number of steps, data is arranged by the averaged_steps. Then, the interval which appears in the first row is what this question is requesting. 


```r
df <- arrange(df.ave_steps, desc(averaged_steps))
paste("5-minute interval which contains the maximum number of steps across all the days in the dataset is",
      as.numeric(df[1, 1]))
```

```
## [1] "5-minute interval which contains the maximum number of steps across all the days in the dataset is 835"
```

## Imputing missing values
Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
paste("The total number of rows with NAs is", sum(is.na(data2$steps)))
```

```
## [1] "The total number of rows with NAs is 2304"
```
Strategy for filling in all missing values in the dataset is to use the mean for all day.

Create a new dataset that is equal to the original dataset but with the missing data filled in as *data2_filled*.
And missing value was replaced by 37 (same value as mean step through all day)

```r
data2_filled <- data2
#mutate(data2, steps = recode(steps, NA = )
#a <- mean(data2$steps, na.rm = TRUE)
data2_filled[is.na(data2$steps), 1] <- 37
# mean(data2$steps, na.rm = TRUE)
#<- data2[is.na(data2$steps), mean(data2$steps, na.rm = TRUE)]
```

Make a histogram with *data2_filled* and calculate mean and median total numner of steps.

```r
data2_filled_date <- group_by(data2_filled, date)
df2.total_steps <- summarise(data2_filled_date, total_steps = sum(steps))

with(df2.total_steps,
         hist(total_steps,
         xlab = "Total steps taken per day", 
         main = "Total steps taken per day"))
```

![](PA1_template_files/figure-html/makehist2-1.png)<!-- -->

```r
# Save png file
dev.copy(png, file = "histogram2.png")
```

```
## png 
##   3
```

```r
dev.off()
```

```
## png 
##   2
```

```r
mean.steps2 <- mean(df2.total_steps$total_steps, na.rm = TRUE)
median.steps2 <- median(df2.total_steps$total_steps, na.rm = TRUE)
paste("The mean of the total number of steps taken per day after filled in is", mean.steps2)
```

```
## [1] "The mean of the total number of steps taken per day after filled in is 10751.737704918"
```

```r
paste("The median of the the total number of steps taken per day after filled in is", median.steps2)
```

```
## [1] "The median of the the total number of steps taken per day after filled in is 10656"
```
Mean value is 1.002 times larger than the the first part of the assignment.
Median value  is 1.02 times larger than the the first part of the assignment.

## Are there differences in activity patterns between weekdays and weekends?
Create *category2* variable which shows each date is weekday or weekend using wday function.

```r
library(lubridate)
```

```
## 
## Attaching package: 'lubridate'
```

```
## The following objects are masked from 'package:base':
## 
##     date, intersect, setdiff, union
```

```r
date_POSIX <- as.POSIXct(strptime(data2_filled$date, "%Y-%m-%d"))
data3 <- mutate(data2_filled, date_POSIX = date_POSIX)
data3 <- mutate(data3, category = wday(date_POSIX))
data3 <- mutate(data3, category2 = case_when(
  category == 1 ~ "weekend",
  category == 7 ~ "weekend", 
  TRUE ~ "weekday"))
```

Make a panel plot containing a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

```r
data3_group <- group_by(data3, interval, category2)
data3_sum1 <- summarise(data3_group, Number_of_steps = mean(steps))
```

```
## `summarise()` has grouped output by 'interval'. You can override using the `.groups` argument.
```

```r
#head(data3_sum1)

library(ggplot2)
ggplot(data3_sum1, aes(x = interval, y = Number_of_steps)) +
  geom_line() +
  facet_grid(rows = vars(category2))
```

![](PA1_template_files/figure-html/panelplot-1.png)<!-- -->

```r
dev.copy(png, file = "panelplot.png")
```

```
## png 
##   3
```

```r
dev.off()
```

```
## png 
##   2
```


