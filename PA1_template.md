---
title: 'Reproducible Research: Peer Assessment 1'
author: "Govind Singhal"
date: "Sunday, September 13, 2015"
output: html_document
---



##Loading and preprocessing the data

Show any code that is needed to

1.Load the data (i.e. `read.csv()`)

2.Process/transform the data (if necessary) into a format suitable for your analysis

```{r}
setwd("C:/aaaaaaaaaaaaa/coursera/Data science/5.Reproducial Research/repdata-data-activity")

library(plyr)
library(lattice)
library(ggplot2)
library(data.table)
library(grid)
library(scales)
library(httr)
library(dplyr)
activity <- read.csv("activity.csv")

```

##What is mean total number of steps taken per day?

For this part of the assignment, you can ignore the missing values in the dataset.

1.Calculate the total number of steps taken per day

2.Calculate and report the mean and median of the total number of steps taken per day

```{r}
activity$fechayhora <- as.POSIXct(with(activity, paste(date, paste(interval %/% 100, interval %% 100, sep=":"))),
    format="%Y-%m-%d %H:%M",tz="")

pasospordia <- setNames(aggregate(steps~as.Date(date), activity, sum, na.rm = TRUE), c("date","steps"))

xaxis <- seq(1, nrow(pasospordia), by = 6)

okscale <- list(x = list(rot = 45, cex = 1.0, labels = format(pasospordia$date, "%d-%b-%Y")[xaxis], at = xaxis))

barchart(date ~ steps, data = pasospordia, main = "steps per day", ylab = "steps", xlab = "date", scales = okscale, horizontal = F)
paste("mean:", mean(pasospordia$steps))
paste("median:", median(pasospordia$steps))
```
##What is the average daily activity pattern?

1.Make a time series plot (i.e. `type = "l"`) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

2.Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```{r}
seriedetiempo <- aggregate(steps ~ interval, data = activity, FUN = mean)
plot(seriedetiempo, type = "l", axes = F, xlab = "Time of the day", 
    ylab = "Average across all days provided a time", main = "Average number of steps taken", 
    col = "red")
axis(1,at=c(seq(0,2400,100),835), label = paste(c(seq(0,24,1),8),c(rep(":00",25),":40"),sep=""), pos = 0)
axis(2, at=c(seq(0,210,30),206.2), label = c(seq(0,210,30),206.2), pos = 0)
maximo <- which.max(seriedetiempo$steps)
segments(832, 0, 832, 206.2, col = "blue", lty = "dashed")
text(835,200, "max average of steps: (832,206.2)", col = "blue", adj = c(-.1, -.1))
segments(0, 206.2, 832, 206.2, col = "blue", lty = "dashed")
seriedetiempo [maximo, ]
#which.max(seriedetiempo$steps)/12
paste(835, "is equivalent to 8.667 hours, this gives the maximum is reached at 8:40 am")
```

##Imputing missing values

Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

1.Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with `NA`s)

2.Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

3.Create a new dataset that is equal to the original dataset but with the missing data filled in.

4.Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```{r}
paste("missing observations:", sum(is.na(activity$steps)))

"missing observations can be replaced with the sample mean instead of replacing them by zero (the 1st won't bias the estimates while the 2nd will)"

datosnuevos <- activity
datosnuevos[is.na(activity$steps), ]$steps <- mean(activity$steps)

datosnuevos$fechayhora <- as.POSIXct(with(datosnuevos, paste(date, paste(interval %/% 100, interval %% 100, sep=":"))),
    format="%Y-%m-%d %H:%M",tz="")

pasospordia2 <- setNames(aggregate(steps~as.Date(date), datosnuevos, sum, na.rm = TRUE), c("date","steps"))

xaxis <- seq(1, nrow(pasospordia2), by = 6)

okscale2 <- list(x = list(rot = 45, cex = 1.0, labels = format(pasospordia2$date, "%d-%b-%Y")[xaxis], at = xaxis))

barchart(date ~ steps, data = pasospordia2, main = "steps per day", ylab = "steps", xlab = "date", scales = okscale2, horizontal = F)

paste("mean:", mean(pasospordia2$steps))
paste("median:", median(pasospordia2$steps))

paste("means difference:", mean(pasospordia2$steps)-mean(pasospordia$steps))
paste("medians difference:", median(pasospordia2$steps)-median(pasospordia$steps))
```

##Are there differences in activity patterns between weekdays and weekends?

For this part the `weekdays()` function may be of some help here. Use the dataset with the filled-in missing values for this part.

1.Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

2.Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.

```{r}
str(datosnuevos)
datosnuevos$date <- as.Date(datosnuevos$date, "%Y-%m-%d")
datosnuevos$day <- weekdays(datosnuevos$date)
datosnuevos$tipodia <- c("weekday")
for (i in 1:nrow(datosnuevos)){
  if (datosnuevos$day[i] == "Saturday" || datosnuevos$day[i] == "Sunday"){
    datosnuevos$tipodia[i] <- "weekend"
  }
}
datosnuevos$tipodia <- as.factor(datosnuevos$tipodia)
semana_o_finde <- aggregate(steps ~ interval+tipodia, datosnuevos, mean)
qplot(interval, steps, data=semana_o_finde, geom=c("line"), xlab="5-min intervals", 
      ylab="steps mean", main="") + facet_wrap(~ tipodia, ncol=1)
```





