Reproducible Research Course Project 1
===============================================
Binu Pundir  

***


This assignment comprises of multiple parts. This is a report that answers all the questions. The entire assignment is compiled in a single R markdown document that can be processed by knitr and be transformed into an HTML file.

Throughout the report we must include the code that was used to generate the output presented. When writing code chunks in the R markdown document, always use `echo = TRUE` so that someone else will be able to read the code.

We can use setoptions to set the global options, `{r setoptions, echo = TRUE}`



### Loading and preprocessing the data
The code chunk below will load the `activity.csv' data file if does not already exist.

```r
if(!file.exists("activity.csv")) {
  
  unzip("activity.zip")
  
}
```
We now create a new variable `df` 

```r
df <-read.csv("activity.csv",sep = ',', na.strings = 'NA')
```
***  

Here is a quick look at the variables in the dataset

```r
names(df)
```

```
## [1] "steps"    "date"     "interval"
```
Some more information about the data


```r
class(df) # class
```

```
## [1] "data.frame"
```

```r
dim(df)   # dimensions
```

```
## [1] 17568     3
```

```r
summary(df) # summary on all the variables
```

```
##      steps                date          interval     
##  Min.   :  0.00   2012-10-01:  288   Min.   :   0.0  
##  1st Qu.:  0.00   2012-10-02:  288   1st Qu.: 588.8  
##  Median :  0.00   2012-10-03:  288   Median :1177.5  
##  Mean   : 37.38   2012-10-04:  288   Mean   :1177.5  
##  3rd Qu.: 12.00   2012-10-05:  288   3rd Qu.:1766.2  
##  Max.   :806.00   2012-10-06:  288   Max.   :2355.0  
##  NA's   :2304     (Other)   :15840
```

### What is the mean total number of steps taken per day?
For this part of the assignment the `missing values` in the dataset are ignored. The missiing values are reported as *NAs*.

1. Calculate the total number of steps taken per day



```r
# Consider data with complete cases only
df1<-df[(complete.cases(df)),]

# Sum the steps by date to get total number of steps taken per day
total_steps_per_day<-aggregate(steps~date, data=df1, sum)
```

2. Histogram of total number of steps taken per day



```r
hist(total_steps_per_day$steps,breaks=30,col=2, main = "Histogram of Total number of steps per day",
     xlab = "Total number of steps per day")
```

![plot of chunk Histogram-1](figure/Histogram-1-1.png)

3. Calculate and report the mean and median of the total number of steps taken per day


```r
# mean and median of total number of steps per day
m<-mean(total_steps_per_day$steps)
m1<-median(total_steps_per_day$steps)
```

The mean and median of total number of steps per day are `10766` and `10765` respectively

***
 





### What is the average daily activity pattern?

1. Make a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)



```r
# aggregate steps to get averaged steps in intervals across all days
avgsteps_interval_allDays<- aggregate(steps~interval, df1, mean)

# Plot the time series data
p<-ggplot(avgsteps_interval_allDays, aes(x=interval, y=steps))+
  geom_line()+theme_bw()+
  labs(x="Time Intervals", y="Average number of steps") + 
  labs(title="Averaged number of steps averaged over all days")
  print(p)
```

![plot of chunk Timeseries](figure/Timeseries-1.png)

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps


```r
# locate the row with maximum number of steps in an interval
  max_interval_row<-which.max(avgsteps_interval_allDays$steps)

  #interval with maximum average number of steps  
intervalWithMaxSteps<-avgsteps_interval_allDays[max_interval_row,]
round(intervalWithMaxSteps)
```

```
##     interval steps
## 104      835   206
```

The interval `835` contains maximum number of steps.

### Imputing missing values

1. Calculate and report the total number of missing values in the dataset 
This can be done in a number of ways, for example using the `is.na()`  or `complete.cases()`

```r
missing_val<-sum(!(complete.cases(df)))
missing_val
```

```
## [1] 2304
```

```r
missing_val<-sum((is.na(df)))
missing_val
```

```
## [1] 2304
```

There are `2304` missing values.

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.


The missing values in `df` are replaced by the mean for that 5-minute interval from the average daily activity pattern computed in `avgsteps_interval_allDays`


```r
# imputate missing data
for (i in 1:nrow(df)){
  if(is.na(df$steps[i])){
    interval_NA<-df$interval[i] # interval with missing data
    row_id<-which(avgsteps_interval_allDays$interval==interval_NA)
    steps_NA<-avgsteps_interval_allDays$steps[row_id] # steps value 
    df$steps[i]<-steps_NA # replace missing data with aggregate data
  }
}

# average steps by date to get total number of steps in a day
avg_imputed<-aggregate(steps~date, df, sum)
```

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps? 


```r
hist(avg_imputed$steps, breaks=30, col=3,main="Histogram of total number of steps taken each day (imputed data)",xlab= "Number of steps taken each day")
```

![plot of chunk Histogram-2](figure/Histogram-2-1.png)

```r
# mean and median of total number of steps taken per day
mean(avg_imputed$steps)
```

```
## [1] 10766.19
```

```r
median(avg_imputed$steps)
```

```
## [1] 10766.19
```

The mean and median values of the 2 datasets are compared below, imputing data in this case seems to have no influence on the mean, the median with imputed data is negligibly higher.

| | Mean | Median| 
|-----|---|-----|
|Data (NAs removed)|10766|10765|
|Data (Imputed) | 10766|10766|

### Are there differences in activity patterns between weekdays and weekends?

Use the dataset with the filled-in missing values for this part.

1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.


```r
library(lubridate)
# Convert date from string to date class
df$date<-ymd(df$date)
# class(df$date)

# Add a new column to df, to indicate the name of the day of the week
df$day_name<-weekdays(df$date)

# Add one more column to classify day as weekday or weekend and initialise to weekday
df$day_type<-c("weekday")

# Now change day_type to weekend if day_name is Saturday or Sunday in df
for (i in 1:nrow(df)){
if(df$day_name[i]=="Saturday" |df$day_name[i]=="Sunday"){
  df$day_type[i]<-"weekend"
 }
}
class(df$day_type)
```

```
## [1] "character"
```

```r
# convert day_type from character to factor as needed
df$day_type<-as.factor(df$day_type)
```

2. Make a panel plot containing a time series plot (i.e. 𝚝𝚢𝚙𝚎 = "𝚕") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.


```r
# aggregate number of steps in an interval over weekdays and weekends
avgsteps_interval_allDays_imputed<-aggregate(steps~interval+day_type, df, mean)

# panel plot for weekdays and weekends
qplot(interval, steps, data=avgsteps_interval_allDays_imputed, geom=c("line"), xlab="Interval", 
ylab="Number of steps", main="") + facet_wrap(~ day_type, ncol=1) + theme_bw()
```

![plot of chunk Timeseries-2](figure/Timeseries-2-1.png)

There are differences in the activity levels during the weekdays and weekends. For instance this person most likely wakes up later on weekends, therefore are peaks in activities at different times.
