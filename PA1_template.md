# The analysis of the activity data

2024/11/06

By Satoshi Ohnishi

### 1. Read the activity data

Load the necessary libraries and read the activity data.

    library(dplyr)

    ## 
    ## 次のパッケージを付け加えます: 'dplyr'

    ## 以下のオブジェクトは 'package:stats' からマスクされています:
    ## 
    ##     filter, lag

    ## 以下のオブジェクトは 'package:base' からマスクされています:
    ## 
    ##     intersect, setdiff, setequal, union

    library(ggplot2)

    # read activity.csv
    activity <- read.csv("activity.csv")
    head(activity)

    ##   steps       date interval
    ## 1    NA 2012-10-01        0
    ## 2    NA 2012-10-01        5
    ## 3    NA 2012-10-01       10
    ## 4    NA 2012-10-01       15
    ## 5    NA 2012-10-01       20
    ## 6    NA 2012-10-01       25

### 2. Caliculate total number of steps taken per day

Now, Let us calculate the total number of steps taken per day.

    total_steps <- aggregate(steps ~ date, data = activity, sum)
    total_steps

    ##          date steps
    ## 1  2012-10-02   126
    ## 2  2012-10-03 11352
    ## 3  2012-10-04 12116
    ## 4  2012-10-05 13294
    ## 5  2012-10-06 15420
    ## 6  2012-10-07 11015
    ## 7  2012-10-09 12811
    ## 8  2012-10-10  9900
    ## 9  2012-10-11 10304
    ## 10 2012-10-12 17382
    ## 11 2012-10-13 12426
    ## 12 2012-10-14 15098
    ## 13 2012-10-15 10139
    ## 14 2012-10-16 15084
    ## 15 2012-10-17 13452
    ## 16 2012-10-18 10056
    ## 17 2012-10-19 11829
    ## 18 2012-10-20 10395
    ## 19 2012-10-21  8821
    ## 20 2012-10-22 13460
    ## 21 2012-10-23  8918
    ## 22 2012-10-24  8355
    ## 23 2012-10-25  2492
    ## 24 2012-10-26  6778
    ## 25 2012-10-27 10119
    ## 26 2012-10-28 11458
    ## 27 2012-10-29  5018
    ## 28 2012-10-30  9819
    ## 29 2012-10-31 15414
    ## 30 2012-11-02 10600
    ## 31 2012-11-03 10571
    ## 32 2012-11-05 10439
    ## 33 2012-11-06  8334
    ## 34 2012-11-07 12883
    ## 35 2012-11-08  3219
    ## 36 2012-11-11 12608
    ## 37 2012-11-12 10765
    ## 38 2012-11-13  7336
    ## 39 2012-11-15    41
    ## 40 2012-11-16  5441
    ## 41 2012-11-17 14339
    ## 42 2012-11-18 15110
    ## 43 2012-11-19  8841
    ## 44 2012-11-20  4472
    ## 45 2012-11-21 12787
    ## 46 2012-11-22 20427
    ## 47 2012-11-23 21194
    ## 48 2012-11-24 14478
    ## 49 2012-11-25 11834
    ## 50 2012-11-26 11162
    ## 51 2012-11-27 13646
    ## 52 2012-11-28 10183
    ## 53 2012-11-29  7047

### 3. make histogram of the total number of steps taken each day

How distributed the total number of steps taken each day is?

Let us make a histogram of the total number of steps taken each day.

    hist(total_steps$steps,
         main = "Total number of steps taken each day", 
         xlab = "Number of steps", 
         ylab = "Frequency", col = "skyblue")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-2-1.png)

    # save the plot into 'figure' directory
    dev.copy(png, file = "figure/histogram1.png")

    ## png 
    ##   3

### 4. Calculate the mean and median of the total number of steps taken per day

What is the mean and median of the total number of steps taken per day?

    #Calculate the mean and median of the total number of steps taken per day
    mean_steps <- mean(total_steps$steps, na.rm = TRUE)
    median_steps <- median(total_steps$steps, na.rm = TRUE)

    # print mean
    print(mean_steps)

    ## [1] 10766.19

    # print median
    print(median_steps)

    ## [1] 10765

### 5. Time series plot of the average number of steps taken

Now, let us look at the time series plot of the average number of steps
taken, averaged across all days.

    # Make time series plot of the 5-minute interval and the average number of steps taken, averaged across all days
    ggplot(data = activity, aes(x = interval, y = steps)) +
      geom_line(stat = "summary", fun.y = "mean", col = "darkgreen") +
      labs(title = "Time series plot of the average number of steps taken",
           x = "Interval",
           y = "Average number of steps taken")

    ## Warning in geom_line(stat = "summary", fun.y = "mean", col = "darkgreen"):
    ## Ignoring unknown parameters: `fun.y`

    ## Warning: Removed 2304 rows containing non-finite outside the scale range
    ## (`stat_summary()`).

    ## No summary function supplied, defaulting to `mean_se()`

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-4-1.png)

    # Save the plot into 'figure' directory
    dev.copy(png, file = "figure/timeseries.png")

    ## png 
    ##   4

### 6. The maximum number of steps taken in 5-minute interval

Which 5-minute interval, on average across all the days in the dataset,
contains the maximum number of steps?

    # Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?
    max_interval <- activity %>% 
      group_by(interval) %>% 
      summarise(mean_steps = mean(steps, na.rm = TRUE)) %>% 
      filter(mean_steps == max(mean_steps))

    print(max_interval)

    ## # A tibble: 1 × 2
    ##   interval mean_steps
    ##      <int>      <dbl>
    ## 1      835       206.

### 7. Imputing missing values and make histogram of the total number of steps taken each day after missing values are filled in

I think that missing values should be filled in with the 0 value.

Because if fill in with the mean or median value, the distribution of
the data becomes bigger than the original data.

The monitors may be sleeping or

Therefore, I will fill in the missing values with 0.

    # Fill in the missing values with 0
    activity_imputed <- activity[is.na(activity$steps), "steps"] <- 0
    # Calculate the total number of steps taken per day
    total_steps_imputed <- aggregate(steps ~ date, data = activity, sum)
      
    hist(total_steps_imputed$steps,
         main = "Total number of steps taken each day", 
         xlab = "Number of steps", 
         ylab = "Frequency", col = "orange")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-6-1.png)

    # save the plot into 'figure' directory
    dev.copy(png, file = "figure/histogram2.png")

    ## png 
    ##   5

    # Calculate the mean and median of the total number of steps taken per day
    mean_steps_imputed <- mean(total_steps_imputed$steps, na.rm = TRUE)
    median_steps_imputed <- median(total_steps_imputed$steps, na.rm = TRUE)
    print(mean_steps_imputed)

    ## [1] 9354.23

    print(median_steps_imputed)

    ## [1] 10395

### 8. Are there differences in activity patterns between weekdays and weekends?

How different are the activity patterns between weekdays and weekends?

Let us make a panel plot containing a time series plot of the 5-minute
interval and the average number of steps taken, averaged across all
weekday days or weekend days.

    # Create new column "weekday" sunday and saturday is weekend, other days are weekday
    activity$date <- as.Date(activity$date)
    activity$weekday <- weekdays(activity$date)
    activity$weekday <- ifelse(activity$weekday == "土曜日" | activity$weekday == "日曜日", "weekend", "weekday")

    # Make 2 panel plot containing a time series plot of the 5-minute interval and the average number of steps taken, averaged across all weekday days or weekend days
    # weekday color is red, weekend color is blue
    ggplot(data = activity, aes(x = interval, y = steps)) +
      geom_line(stat = "summary", fun.y = "mean", color = "orangered") +
      facet_wrap(~ weekday, ncol = 1, nrow = 2) +
      labs(title = "Average number of steps taken by 5-minute interval",
           x = "Interval",
           y = "Average number of steps") +
      theme_minimal()

    ## Warning in geom_line(stat = "summary", fun.y = "mean", color = "orangered"):
    ## Ignoring unknown parameters: `fun.y`

    ## No summary function supplied, defaulting to `mean_se()`
    ## No summary function supplied, defaulting to `mean_se()`

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-7-1.png)

    # Save the plot into 'figure' directory
    dev.copy(png, file = "figure/panelplot.png")

    ## png 
    ##   6

### 9. Conclusion

In the weekday, the number of steps taken is high in the morning.

I think that the handling of missing values is important.

In this analysis, I filled in the missing values with 0.

But it may be better to fill in with the mean or median value.

I believe that the handling of missing values changes the distribution
of the data and results and conclusions may change.
