# Reproducible Research - Assessment 1
F. Novak  
Tuesday, March 3, 2015  
## Introduction
This report describes the analyses and results associated with the Reproducible Research course, and in particular the first assessment (project). The data used for this class described personal movement, and was collected for several days at 5-minute intervals using activity monitoring devices.

The source of the data was a website:
https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip


The data was downloaded on February 13,2015, at 8:03PM, Eastern Standard Time.

## Opening and Checking Data
The dataset was downloaded to local hard drive and opened using read.csv without any special options, as follows.


```r
file <- "./activity.csv"
d <- read.csv(file)
names(d)
```

```
## [1] "steps"    "date"     "interval"
```
The date column was processed to ensure it would be useable as a date.

```r
library(lubridate)
d$date <- as.Date(d$date)
```

## Preliminary Analysis - Ignoring NA
The assignment informed us there are NAs, but required some analyses without accounting for the NA values.  This section provides the code and analyses.

### Total Steps Per Day
One part of the assignment is to find the total number of steps per day, which was found using some dplyr commands.

```r
library(dplyr)
spd <- summarize(group_by(d, date),sum(steps,na.rm=T))
names(spd) <- c("date","total_steps")
```
The results are shown in a histogram on a histogram.

```r
hist(spd$total_steps, main = "Total # of Steps in a Day"
     , xlab = "steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png) 

The mean and median number of steps per day were calculated.

```r
mean(spd$total_steps)
```

```
## [1] 9354.23
```

```r
median(spd$total_steps)
```

```
## [1] 10395
```

### Average Daily Activity Pattern
The next part of the assignment is to find mean steps per interval (spi), across all days, still ignoring NAs.


```r
spi <- summarize(group_by(d, interval), mean(steps,na.rm=T))
names(spi) <- c("interval","mean_steps")
```
The result is plotted using ggplot.

```r
library(ggplot2)
g1 <- ggplot(spi,aes(interval,mean_steps))
g1 + geom_point(size = 3) + 
  labs(title = "Means # of steps at each interval") +
  labs(x = "interval", y="mean")
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png) 

The assignment asks which 5-minute interval has the maximum number of steps, on average across all the days in the dataset, 


```r
spi$interval[spi$mean_steps == max(spi$mean_steps)]
```

```
## [1] 835
```

## Cleaning Data - Imputing Missing Values
The data contains a number of missing values (NA) in the "steps" column.  The number of NAs was determined, as a fraction of the total and as an absolute number.

```r
m_s <- is.na(d$steps)
mean(m_s) 
```

```
## [1] 0.1311475
```

```r
sum(m_s) 
```

```
## [1] 2304
```

The assignment requires a new data set with imputed values that replace the NA values.  The assignment allows that the strategy can be simple.

The entire set of NA was printed, and it was found that the NAs appear in large sets consecutive intervals.  What was not found is NAs scattered here and there throughout the data.  Therefore, the strategy that was adopted was to replace each NA with the mean value that was calculated previously for the interval. 

Other options were considered, but judged to be less attractive.  One was to replace the value with the mean for the whole day.  However, there is a pretty wide distribution (see earlier time plot), depending on the interval.  For example, the mean values during the early morning intervals are zero or nearly zero.  Replacing NAs that appear in those intervals would, in all probability, not be close to correct.  Another option that would have been attractive, if the NAs were scattered here and there, would be to interpolate between the adjacent intervals.  However, with many consecutive NA values, this woud not work.

Thus, the approach was to use the previously calculated mean value for each interval, and use those to replace the NAs.

```r
d_new <-  d

for(i in 1:nrow(d_new)) {
  if(m_s[i]) {
    d_new$steps[i] <- spi$mean_steps[spi$interval == d_new$interval[i]]
  }
  else {
    d_new$steps[i] <- d$steps[i]
  }
}
```

The results were spot-checked.  First, it was confirmed that there are no NAs in the new dataset.

```r
m_s_new <- is.na(d_new$steps)
mean(m_s_new)
```

```
## [1] 0
```

Addtionally, some rows from the original dataset were selected that were NA, and some other rows were selected that were not NA. For each of the selected rows, the original and imputed values were printed.  The expectation was that if the orignal was NA, it was replaced with a numeric value. In the cases when the original value was not NA, the expectation was that the values would be the same.

```r
i_na <- c(287,2100,9000,9978)
i_not_na <- c(353,1794,5017,9769)
na_check <- cbind(i_na,m_s[i_na],d$steps[i_na],d_new$steps[i_na])
not_na_check <- cbind(i_not_na,m_s[i_not_na],d$steps[i_not_na],
                      d_new$steps[i_not_na])
na_check; not_na_check
```

```
##      i_na                
## [1,]  287 1 NA  0.2264151
## [2,] 2100 1 NA 49.0377358
## [3,] 9000 1 NA 44.4905660
## [4,] 9978 1 NA 47.7547170
```

```
##      i_not_na      
## [1,]      353 0 0 0
## [2,]     1794 0 0 0
## [3,]     5017 0 0 0
## [4,]     9769 0 0 0
```

The results are acceptable.  The new values of total steps per day is calculated and provided in a new histogram.


```r
spd_new <- summarize(group_by(d_new, date),sum(steps,na.rm=T))
names(spd_new) <- c("date","total_steps")

hist(spd_new$total_steps, main = "Total # of Steps in a Day,
    with NA replaced", 
    xlab = "steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-13-1.png) 

Finally, the new mean and median of total steps per day were calculated.

```r
mean(spd_new$total_steps)
```

```
## [1] 10766.19
```

```r
median(spd_new$total_steps)
```

```
## [1] 10766.19
```

From this table, and by comparing the 
To see the effect of imputation more readily, the original and new values are tablulated.

```r
x1 <- mean(spd$total_steps)
x2 <- median(spd$total_steps)
y1 <- mean(spd_new$total_steps)
y2 <- median(spd_new$total_steps)

Original <- c(x1,x2); New <- c(y1,y2)
Diff <- c(y1-x1,y2-x2)

table <- cbind(New,Original,Diff)
row.names(table) <- c("Mean","Median")

table
```

```
##             New Original      Diff
## Mean   10766.19  9354.23 1411.9592
## Median 10766.19 10395.00  371.1887
```

What is apparent is that the total number of steps per day tends to be larger.  This is what we expect because many of the non-zero imputed values replace what are effectively zero in the original dataset.  Therefore, there is a shift away from zero on the histogram.  We conclude that the effect of NA is a bias toward zero.

Something that was surprising was that in the new data, the mean and median values were the same.

### Activity Patterns on Weekdays and Weekends
In the final part of the assignment, a new factor variable is created to indicate the behavior patterns on weekdays and weekends.  The new dataset, the one with imputed values that replace the NAs, is used.

First, the new factor is created.

```r
dow <- weekdays(d_new$date)
type_day <- factor(dow)
levels(type_day) <- list(Weekend = c("Saturday","Sunday"),
                    Weekday = c("Monday","Tuesday",
                    "Wednesday","Thursday","Friday"))

d_new <- cbind(d_new,type_day)
```

Then, the results are summarized using dplyr and plotted using ggplot, as before.


```r
spi_new <- summarize(group_by(d_new, type_day, interval),mean(steps))
names(spi_new) <- c("type_day","interval","mean_steps")

g1 <- ggplot(spi_new,aes(interval,mean_steps))
g1 + geom_line(color = "blue") + 
    labs(title = "Mean # of steps at each interval") +
    labs(x = "interval", y="mean") + 
    facet_grid(~ type_day ~ .)
```

![](PA1_template_files/figure-html/unnamed-chunk-17-1.png) 

What is apparent is the person being measured is more active on weekends than on weekdays.  One exception is at around 8AM, when the person takes more steps on weekdays.

### Conclusion
Each part of the assignment in Assessment 1 was completed.  The original data, including the NAs, were analyzed.  The effect of imputing NA with the means was investigated, and the behavior on weekends was compared to that of weekdays.

This report provides the results and the code in sufficient detail that the analysis is reproducible.
