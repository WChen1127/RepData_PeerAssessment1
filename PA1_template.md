Peer Assignment 1: Activity Monitor Data Analysis
================================================= 


## Introduction
It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a [Fitbit][1], [Nike Fuelband][2], or [Jawbone Up][3]. These type of devices are part of the “quantified self” movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

[1]: http://www.fitbit.com/                             "Fitbit"
[2]: http://www.nike.com/us/en_us/c/nikeplus-fuelband/  "Nike Fuelband"
[3]: https://jawbone.com/up/                            "Jawbone Up"


## Data
The data for this assignment can be downloaded from the course web site:

- Dataset: [Activity monitoring data][4] [52K] 

The variables included in this dataset are:

- **steps**: Number of steps taking in a 5-minute interval (missing values are coded as NA)

- **date**: The date on which the measurement was taken in YYYY-MM-DD format

- **interval**: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.

[4]: https://class.coursera.org/repdata-003/human_grading/view/courses/972142/assessments/3/submissions/                                       "Activity monitoring data"


## Assignment
This assignment will be described in multiple parts. I will write a report that answers the questions detailed below. Ultimately, I will complete the entire assignment in a **single R markdown** document that can be processed by **knitr** and be transformed into an HTML file.

Throughout my report make sure you always include the code that I used to generate the output you present. When writing code chunks in the R markdown document, always use echo = TRUE so that someone else will be able to read the code. **This assignment will be evaluated via peer assessment so it is essential that my peer evaluators be able to review the code for my analysis.**

For the plotting aspects of this assignment, feel free to use any plotting system in R (i.e., base, lattice, ggplot2)

Fork/clone the [GitHub repository created for this assignment][5]. I will submit this assignment by pushing my completed files into my forked repository on GitHub. The assignment submission will consist of the URL to your GitHub repository and the SHA-1 commit ID for your repository state.

NOTE: The GitHub repository also contains the dataset for the assignment so I do not have to download the data separately.

[5]: https://github.com/rdpeng/RepData_PeerAssessment1 "GitHub repository created for this assignment"

### Loading and preprocessing the data
Show any code that is needed to  

1. Load the data (i.e. read.csv())  

2. Process/transform the data (if necessary) into a format suitable for your analysis.



```r
data <- read.csv("activity.csv")
```

### What is mean total number of steps taken per day?
For this part of the assignment, you can ignore the missing values in the dataset.  

1. Make a histogram of the total number of steps taken each day

2. Calculate and report the **mean** and **median** total number of steps taken per day



```r
# clean up the data, removing all "NA"
data_nna <- subset(data, data$steps != "NA")
library(reshape)
```

```
## Loading required package: plyr
## 
## Attaching package: 'reshape'
## 
## The following objects are masked from 'package:plyr':
## 
##     rename, round_any
```

```r
# reshape the data based on "date"
mdata <- melt(data_nna, id = "date")
daydata <- cast(mdata, date ~ variable, sum)
hist(daydata$steps, 
     main = "Histogram of the daily total steps",
     xlab = "daily total steps")
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2.png) 

```r
# calculate the mean and median
summ_step <- summary(daydata$steps)
summ_step[['Mean']]
```

```
## [1] 10800
```

```r
summ_step[["Median"]]
```

```
## [1] 10800
```


### What is the average daily activity pattern?

1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
# reshape the data based on "interval"
mdata2 <- melt(data_nna[, -2], id = "interval")
intdata <- cast(mdata2, interval ~ variable, mean)
plot(x = intdata$interval, y = intdata$steps,
     type = "l", main = "Average steps in each interval",
     xlab = "interval", ylab = "average steps")
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3.png) 

```r
# the location of the maximum steps
ind <- which.max(intdata$steps)
intdata$interval[ind]
```

```
## [1] 835
```

### Imputing missing values

Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
# the total number of "NA"s
summ_steps <- summary(data$steps)
summ_steps[["NA's"]]
```

```
## [1] 2304
```

```r
# locate all "NA"s
ind_na <- which(is.na(data$steps)) 

# define each "NA" with the mean for that 5-minute interval
for(i in ind_na){
  ival <- data$interval[i]
  data$steps[i] <- intdata$steps[intdata$interval == ival]
}

# reshape the data based on the date
mdata3 <- melt(data, id = "date")
daydata2 <- cast(mdata3, date ~ variable, sum)
hist(daydata2$steps, 
     main = "Histogram of the daily total steps",
     xlab = "daily total steps")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 

```r
# calculate the mean and median
summ_step2 <- summary(daydata2$steps)
summ_step2[['Mean']]
```

```
## [1] 10800
```

```r
summ_step2[["Median"]]
```

```
## [1] 10800
```

Note: both the **mean** and **median** are not changed, since I impute the missing data with the  mean for that 5-minute interval. Under such strategy, the **mean** is definitely remained while the "median" may change. 

### Are there differences in activity patterns between weekdays and weekends?

For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). The plot should look something like the following, which was creating using simulated data:


```r
# a function to convert "date" to "weekdays"
weekday_f <- function(date){
  wds <- weekdays(as.POSIXct(date))
  wd <- ifelse(wds == "Saturday" | wds == "Sunday", "weekend", "weekday")
  return(wd)
}
data$weekdays <- lapply(data$date, weekday_f)

# subset of "weekday"
data_wkd <- subset(data, data$weekdays == "weekday")
mdata_wkd <- melt(data_wkd[,-c(2, 4)], id = "interval")
intdata_wkd <- cast(mdata_wkd, interval ~ variable, mean)
intdata_wkd$weekday <- "weekday"

# subset of "weekend"
data_wkend <- subset(data, data$weekdays == "weekend")
mdata_wkend <- melt(data_wkend[,-c(2, 4)], id = "interval")
intdata_wkend<- cast(mdata_wkend, interval ~ variable, mean)
intdata_wkend$weekday <- "weekend"

# aggregate "weekday" and "weekend" subsets
aggdata <- rbind(intdata_wkd, intdata_wkend)

library(lattice)
xyplot(steps ~ interval | weekday, data=aggdata, type="l")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5.png) 




