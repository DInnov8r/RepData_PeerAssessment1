### Reproducible Research: Peer Assessment 1 - July 20, 2014

#### Introduction
The following report analyzes data from a personal activity monitoring device such as a [Fitbit](http://www.fitbit.com/), [Nike Fuelband](http://www.nike.com/us/en_us/c/nikeplus-fuelband), or [Jawbone Up](https://jawbone.com/up) to answer several questions listed below. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

#### Data
The data for this assignment can be downloaded from the course web site:

Dataset: [Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip) [52K]
The variables included in this dataset are:

-**steps**: Number of steps taking in a 5-minute interval (missing values are coded as _NA_)

-**date**: The date on which the measurement was taken in YYYY-MM-DD format

-**interval**: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.


#### Loading and preprocessing the data


```r
activity <- read.csv("activity.csv", colClasses = c("numeric", "character", "numeric"))
suppressMessages(require(lattice))
activity["date"] <- as.Date(activity$date, "%Y-%m-%d")
```

#### What is mean total number of steps taken per day?

```r
steps_per_day <- aggregate(steps ~ date, data = activity, sum, na.rm = TRUE)
mean_steps_per_day <- mean(steps_per_day$steps)
median_steps_per_day <- median(steps_per_day$steps)

hist(steps_per_day$steps, main = "Frequency of Steps Taken Per Day",
               xlab = "Daily Step Count Range",
               ylab="Number of Days Step Count Range is Reached", col = "green", xlim=c(0,25000), ylim=c(0,20), breaks=10)
```

<img src="./PA1_template_files/figure-html/histsteps.png" title="plot of chunk histsteps" alt="plot of chunk histsteps" style="display: block; margin: auto;" />

The **mean** steps taken per day is **10766**.  The **median** steps taken per day is **10765**.

The histogram shows 36 days where the step count is greater than 10,000 steps per day.  This is consistent with the number of business days during the months of October and November when the data was collected.

#### What is the average daily activity pattern?

```r
time_series <- tapply(activity$steps, activity$interval %% 100 / 5 + activity$interval %/% 100 * 12 + 1, mean, na.rm = TRUE)

max_interval <- which.max(time_series)
hour_of_day_start <- (max_interval * 5) %/% 60
minute_of_hour_start <- (max_interval * 5) %% 60

hour_of_day_end <- ((max_interval + 1) * 5) %/% 60
minute_of_hour_end <- ((max_interval + 1) * 5) %% 60

am_pm_start <- "AM"
am_pm_end <- "AM"

if(hour_of_day_start > 12) {
  hour_of_day_start <- hour_of_day_start - 12
  am_pm_start <- "PM"
}

if(hour_of_day_end > 12) {
  hour_of_day_end <- hour_of_day_end - 12
  am_pm_end <- "PM"
}


plot(row.names(time_series), time_series, type = "l", xlab = "5-minute Interval (288/Day)", 
    ylab = "Daily Average (steps)", main = "Average Number of Steps During 5-minute Time Intervals In a 24-hour Day", 
    col = "blue")
```

<img src="./PA1_template_files/figure-html/activitypattern.png" title="plot of chunk activitypattern" alt="plot of chunk activitypattern" style="display: block; margin: auto;" />

The 5-minute interval with the **maximum** number of steps (on average) is interval **104**,
or **between 8:40****AM and 8:45****AM**.  This may be when person from whom the data was collected exercises and/or walks to his or her place of work.

**Local maxima** occur at intervals **147**, **191**, and **226** or **12:15-12:20pm**, **3:55-4:00pm**, and **6:50-6:55pm**.  These may be times when the person takes a lunch break, takes an afternoon break, and returns home.

#### Inputing missing values

```r
NA_count <- sum(is.na(activity))
```

There are **2304** missing values in the dataset.

Missing (_NA_) values for daily 5-minute intervals are filled with the interval average across all dates.  This is done using a series of dcasts to reshape the data to a wide format with rowMeans to fill NA values.  The resulting dataset is reshaped to a long format for subsequent analysis.


```r
suppressMessages(require(reshape))
suppressMessages(require(reshape2))
a <- dcast(activity, interval ~ date, value.var="steps", fill=0)
r <- dcast(activity, interval ~ date, value.var = "steps", fill = rowMeans(a, na.rm = TRUE))
r2 <- reshape(r, direction = "long", varying=list(names(r)[2:length(names(r))]),
                               v.names=c("steps"), timevar="date", idvar=c("interval"),
                               times=names(r)[2:length(names(r))], new.row.names=1:dim(activity)[1])

steps_per_day2 <- aggregate(steps ~ date, data = r2, sum, na.rm = TRUE)
hist(steps_per_day2$steps, main = "Frequency of Steps Taken Per Day", xlab = "Daily Step Count Range",
          ylab="Number of Days Step Count Range is Reached", col = "green", xlim=c(0,25000), ylim=c(0,20), breaks=10)
```

<img src="./PA1_template_files/figure-html/unnamed-chunk-2.png" title="plot of chunk unnamed-chunk-2" alt="plot of chunk unnamed-chunk-2" style="display: block; margin: auto;" />

```r
mean_steps_per_day <- mean(steps_per_day2$steps)
median_steps_per_day <- median(steps_per_day2$steps)
```

The mean steps taken per day is **11279**.  The median steps taken per day is **11458**.

The histogram shows an increase in the number of days over 10,000 steps.  The mean and median also increase.  Perhaps this person didn't wear their device during the Thanksgiving holiday.

#### Are there differences in activity patterns between weekdays and weekends?


```r
r2["date"] <- as.Date(r2$date, "%Y-%m-%d")
r2[(weekdays(r2$date) %in% c("Saturday", "Sunday")), "TypeOfDay"] <- "Weekend"
r2[!(weekdays(r2$date) %in% c("Saturday", "Sunday")), "TypeOfDay"] <- "Weekday"

steps <- aggregate(steps ~ interval + TypeOfDay, data = r2, mean)
names(steps) <- c("Interval", "TypeOfDay", "Steps")
steps$Interval <- steps$Interval %% 100 / 5 + steps$Interval %/% 100 * 12 + 1

splot <- xyplot(Steps ~ Interval | TypeOfDay, steps, type = "l", layout = c(1, 2), 
                xlab = "5-minute Interval (288 per Day)", ylab = "Daily Average (steps)")
update(splot,
       main="Comparison of Average Number of Steps During 5-minute\nTime Intervals In a 24-hour Day\nfor Weekend Days Versus Weekdays")
```

<img src="./PA1_template_files/figure-html/unnamed-chunk-3.png" title="plot of chunk unnamed-chunk-3" alt="plot of chunk unnamed-chunk-3" style="display: block; margin: auto;" />

Weekday steps show a large peak around **8:40AM** followed by 4 smaller peaks around lunch time, afternoon break time, and supper time.  Step data appears more uniform throughout weekend days and have smaller peaks.
