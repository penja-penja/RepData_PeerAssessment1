---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---
opts_chunk$set(echo = TRUE)

## Load required packages
```
library(knitr)
library(dplyr)
library(ggplot2)
```

## 1. Loading and preprocessing the data
### 1.1 Load and unzip data into directory
```
if(!file.exists('./data')) {
  dir.create('./data/')
  
  zipFileUrl <- 'https://d396qusza40orc.cloudfront.net/repdata/data/activity.zip'
  download.file(zipFileUrl, destfile='./data/rawData.zip', method="curl")
  
  unzip('./data/rawData.zip', exdir = './data', unzip = 'internal') 
}
```
### 1.2. Read data
```
activity_data <- read.csv('./data/activity.csv', sep = ',', header = TRUE, stringsAsFactors = FALSE)
```

## 2. What is mean total number of steps taken per day?
### 2.1 Calculate the total number of steps per day
```
activity_by_date <- aggregate(steps ~ date, activity_data, sum)
```

### 2.2 Plot histograph of activty by date
```
colnames(activity_by_date) <- list('date', 'steps')
hist(activity_by_date$steps,
     main = paste('Histogram of daily steps'),
     xlab = 'Steps')
```

### 2.3 Mean steps by day
```
activity_mean_by_day <- aggregate(steps ~ date, activity_by_date, mean)
colnames(activity_mean_by_day) <- list('date', 'mean_steps')
activity_mean_by_day
```

### 2.4 Median steps by day
```
activity_med_by_day <- aggregate(steps ~ date, activity_by_date, median)
colnames(activity_med_by_day) <- list('date', 'median_steps')
activity_med_by_day
```

## 3. What is the average daily activity pattern?
### 3.1 Time seris plot of average steps by interval
```
activity_mean_by_interval <- aggregate(steps ~ interval, activity_data, mean)
colnames(activity_mean_by_interval) <- list('interval', 'mean_steps')
plot(activity_mean_by_interval$interval, 
     activity_mean_by_interval$mean_steps, 
     type = 'l',
     xlab = 'Interval',
     ylab = 'Average steps',
     main = 'Average steps by interval')
```

### 3.2 Interval contain max number of steps
```
activity_mean_by_interval[which.max(activity_mean_by_interval$mean_steps),]
```

## 4. Imputing missing values
### 4.1 Calculate total rows with NA
```
total_na <- sum(is.na(activity_data))
```

### 4.2 Strategy for filling in the missing value
Use mean steps of each interval to replace the missing value

### 4.3 Dataset with missing value filled in with mean step values
```
data_filled <- all_data

for(i in 1:nrow(data_filled)) {
  if (is.na(data_filled$steps[i])) {
    interval <- data_filled[i, 'interval']
    mean_steps <- activity_mean_by_interval[activity_mean_by_interval$interval == interval, 'mean_steps'] 
    data_filled[i, 'steps'] <- mean_steps
  }
}
```

### 4.4 Histogram of total steps per day
```
data_filled_activity_sum <- aggregate(steps ~ date, data_filled , sum)
hist(data_filled_activity_sum$steps,
     main = paste('Histogram of daily steps'),
     xlab = 'Steps')
```

### 4.5 Mean and median comparison between imputed and non imputed data
```
mean(data_filled_activity_sum$steps) - mean(activity_by_date$steps)
median(data_filled_activity_sum$steps) - median(activity_by_date$median)
```

## 5. Are there differences in activity patterns between weekdays and weekends?
### 5.1 Add day_type to the data with na imputed
```
data_filled['day_type'] <- weekdays(as.Date(data_filled$date))
data_filled$day_type[data_filled$day_type %in% c('Saturday','Sunday') ] <- 'weekend'
data_filled$day_type[data_filled$day_type != 'weekend'] <- 'weekday'
```

### 5.2 Plot for weekday and weekend step average
```
data_filled_day_type <- aggregate(steps ~ interval + day_type, data_filled, mean)
qplot(interval, 
      steps, 
      data = data_filled_day_type, 
      type = 'l', 
      geom = c('line'),
      xlab = 'Interval', 
      ylab = 'Average steps', 
      main = '') +
  facet_wrap(~ day_type, ncol = 1)
```

