---
title: "Reproducible Research Assignment 1"
author: "Mengyi"
output: html_document

---
Loading and preprocessing the data

```{r}
activity <- read.csv("C:/Users/mxu/Desktop/R/activity.csv")
```
#What is mean total number of steps taken per day?
1.Calculate the total number of steps taken per day

```{r}
total_step<-aggregate(activity$steps ,list (activity$date),FUN="sum")
colnames(total_step)<-c("date", "total_step")
```

2.Make a histogram of the total number of steps taken each day
```{r}
pl<-hist(as.numeric(as.character(total_step$total_step)), main="histogram of total steps across days", xlab="total step")
```
3.
Calculate and report the mean and median of the total number of steps taken per day
```{r}
mean(total_step$total_step, na.rm=T)
median(total_step$total_step, na.rm=T)
```

#What is the average daily activity pattern?

1.Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
```{r}
average_interval<-aggregate(activity$steps ,list (activity$interval),FUN="mean",na.rm=T )
colnames(average_interval)<-c("interval", "average_steps")
plot(average_interval$interval,average_interval$average_steps,  type = "l", main="average step for interval across all days")
```


2.Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?
```{r}
plot(average_interval$interval,average_interval$average_steps,  type = "l", main="average step for interval across all days")
max_point<-average_interval[average_interval$average_steps==max(average_interval$average_steps),]
points(max_point, pch = 20, col='red')
```
#Imputing missing values

1.Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)
```{r}
nrow(activity[activity$steps=='NA',])
```

2.Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

The strategy I used is to calculate the daily average steps and applied to the missing values.
```{r}
mean_each_day<-aggregate(activity$steps ,list (activity$date),FUN="mean")
colnames(mean_each_day)<-c("date", "total_step")
mean(mean_each_day$total_step, na.rm=T)  --37.3826
for(i in 1:ncol(mean_each_day)){
  mean_each_day[is.na(mean_each_day[,i]), i] <- mean(mean_each_day[,i], na.rm = TRUE)
}
missing_data<-activity[is.na(activity$steps),]
merge_data<-merge(missing_data,mean_each_day, by='date' )
no_missing_data<-na.omit(activity)
replace_data<-merge_data[,c(4,1,3)]
colnames(replace_data)<-colnames(activity)

```


3.Create a new dataset that is equal to the original dataset but with the missing data filled in.
```{r}
final_data<-rbind(no_missing_data,replace_data)
```


4.Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```{r}
aggre_fal<-aggregate(final_data$steps, list(final_data$date), FUN="sum")
hist(aggre_fal$x, breaks=50, main="Histogram of total steps each day")
mean(aggre_fal$x)--10766.19
median(aggre_fal$x)--10766.19
```

#Are there differences in activity patterns between weekdays and weekends?


1.Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.
```{r}
final_data$date<-as.Date(final_data$date,"%Y-%m-%d")
final_data$weekday<-weekdays(final_data$date)
final_data$weekday<-as.factor(final_data$weekday)


levels(final_data$weekday) <- list(weekday = c("Monday", "Tuesday",
                                               "Wednesday", 
                                             "Thursday", "Friday"),
                                 weekend = c("Saturday", "Sunday"))
avgSteps <- aggregate(final_data$steps, 
                      list(interval = as.numeric(as.character(final_data$interval)), 
                           weekday = final_data$weekday),
                      FUN = "mean")

```


2.Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.

```{r}

library(lattice)
xyplot(avgSteps$x ~ avgSteps$interval | avgSteps$weekday, 
       layout = c(1, 2), type = "l", 
       xlab = "Interval", ylab = "Number of steps")
```
