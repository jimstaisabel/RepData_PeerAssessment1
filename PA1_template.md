# REPRODUCIBLE RESEARCH: Peer Assessment 1

Author: Jim Sta. Isabel

## Loading and preprocessing the data

    setwd("/Users/jimstaisabel/Documents/R projects/ReproducibleResearch-Coursera/RepData_PeerAssessment1")
    activity <- read.csv("activity.csv")
    activity$date <- as.Date(activity$date, "%Y-%m-%d")

## Mean total number of steps per day

#### Histogram of total number of steps per day

    steps_per_day <- tapply(activity$steps, activity$date, sum, na.rm = TRUE)

    library(ggplot2)
    qplot(steps_per_day, xlab = "Mean number of steps per day")

    ## `stat_bin()` using `bins = 30`. Pick better value `binwidth`.

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-82-1.png)

#### Mean and median number of daily steps

    c(Mean = mean(steps_per_day, na.rm = TRUE), Median = median(steps_per_day, na.rm = TRUE))

    ##     Mean   Median 
    ##  9354.23 10395.00

## Average daily activity pattern

    mean_steps_interval <- aggregate(activity$steps, by = list(activity$interval), FUN = mean, na.rm = TRUE)
    names(mean_steps_interval) <- c("interval", "totalsteps")

    ggplot(mean_steps_interval, aes(x = interval, y = totalsteps)) + geom_line() +
        labs(x = "5-Minute Interval", y = "Mean Daily Number of Steps")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-84-1.png)

#### Interval with highest mean number of steps

    mean_steps_interval[which.max(mean_steps_interval$totalsteps), ]

    ##     interval totalsteps
    ## 104      835   206.1698

Interval 835 has the highest mean daily number of steps taken, with an
average of 206 steps.

## Inputting missing values

#### Number of missing values in the steps variable

    table(is.na(activity$steps))

    ## 
    ## FALSE  TRUE 
    ## 15264  2304

#### Filling in the missing data

The NA values will be imputed by replacing them with the median number
of steps for the interval.

    library(dplyr)

    activity2 <- activity %>%
      group_by(interval) %>%
      mutate(steps = ifelse(is.na(steps),
                            median(steps, na.rm = TRUE),
                            steps)) %>%
      ungroup()

#### Histogram of total number of steps per day using the new dataset with imputed values

    steps_per_day2 <- tapply(activity2$steps, activity2$date, sum, na.rm = TRUE)

    qplot(steps_per_day2, xlab = "Mean number of steps per day")

    ## `stat_bin()` using `bins = 30`. Pick better value `binwidth`.

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-88-1.png)

#### Mean and median number of daily steps using the new dataset with imputed values

    c(Mean = mean(steps_per_day2), Median = median(steps_per_day2))

    ##      Mean    Median 
    ##  9503.869 10395.000

With the imputed data to replace missing values, there are less zero
values, so the mean is now higher (9354 vs 9503). The median is the
same.

## Activity patterns between weekdays and weekends

#### Creating a weekday/weekend variable

    activity2$dayofweek <- weekdays(activity2$date)
    weekdays <- c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday")
    activity2$dayofweek <- ifelse(activity2$dayofweek %in% weekdays, "weekday", "weekend")

#### Activity pattern on weekdays vs on weekends

    mean_steps_interval2 <- aggregate(activity2$steps, by = list(activity2$interval, activity2$dayofweek), FUN = mean)
    names(mean_steps_interval2) <- c("interval", "dayofweek", "totalsteps")

    ggplot(mean_steps_interval2, aes(x = interval, y = totalsteps)) +
        geom_line() +
        facet_wrap(dayofweek ~., nrow = 2, ncol = 1) +
        labs(x = "5-Minute Interval", y = "Mean Daily Number of Steps")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-91-1.png)
