# Bellabeat-case-study

## Author

Krishna Pareek

## Date

July 18, 2024



## Overview

Bellabeat is a high-tech manufacturer specializing in health-focused products designed for women. Although a successful small company, Bellabeat has potential for significant growth in the global smart device market. Urška Sršen, co-founder and Chief Creative Officer, aims to leverage smart device fitness data to uncover new business opportunities.

## Key Stakeholders

- **Urška Sršen**: Co-founder and Chief Creative Officer of Bellabeat
- **Sando Mur**: Mathematician and co-founder of Bellabeat

## Key Products

- **Bellabeat App**: Provides comprehensive health data related to activity, sleep, stress, menstrual cycle, and mindfulness to help users make healthier decisions.
- **Leaf**: A wellness tracker that can be worn as a bracelet, necklace, or clip, tracking activity, sleep, and stress.
- **Time**: A wellness tracker resembling a classic timepiece, tracking similar metrics as the Leaf.
- **Spring**: A smart water bottle that tracks daily water intake and hydration levels.
- **Bellabeat Membership**: Offers 24/7 access to personalized guidance on nutrition, activity, sleep, health, beauty, and mindfulness based on individual goals.

## Data Analysis Process

The analysis follows a structured six-step process:

1. **Ask**
2. **Prepare**
3. **Process**
4. **Analyze**
5. **Share**
6. **Act**

### Ask

**Stakeholders:**
- Urška Sršen
- Sando Mur
- Bellabeat Marketing Analytics Team

**Business Questions:**
1. What are the trends in smart device usage?
2. How do these trends apply to Bellabeat customers?
3. How can these trends influence Bellabeat’s marketing strategy?

### Prepare

The data for this analysis is sourced from the [FitBit Fitness Tracker Data](https://www.kaggle.com/datasets). It includes personal fitness tracker data from thirty Fitbit users, including activity, heart rate, and sleep monitoring.

**Data Integrity and Credibility:**
- **Reliable**: LOW — Small sample size
- **Original**: LOW — Third-party source
- **Comprehensive**: LOW — Limited demographic details
- **Current**: LOW — Data from 2016
- **Cited**: LOW — Source not explicitly identified

**Limitations:**
- Small sample size
- Limited data on user demographics and health conditions

### Process

```r
# Install necessary packages
install.packages(c("tidyverse", "skimr", "here", "janitor", "ggplot2", "lubridate", "dplyr", "sqldf", "plotrix"))

# Load libraries
library(tidyverse)
library(skimr)
library(here)
library(janitor)
library(ggplot2)
library(lubridate)
library(dplyr)
library(sqldf)
library(plotrix)

# Set the working directory
setwd("C:/Users/Hp-D/Desktop/bellabeat case study/mturkfitbit_export_4.12.16-5.12.16/Fitabase Data 4.12.16-5.12.16")

# Load data
daily_activity <- read.csv("dailyActivity_merged.csv")
sleep_day <- read.csv("sleepDay_merged.csv")
weight <- read.csv("weightLogInfo_merged.csv")
hourly_step <- read.csv("hourlySteps_merged.csv")

# Check data
head(daily_activity)
head(sleep_day)
head(weight)
head(hourly_step)

# Check for missing values and duplicates
sum(is.na(daily_activity))
sum(is.na(sleep_day))
sum(is.na(weight))
sum(duplicated(daily_activity))
sum(duplicated(sleep_day))
sum(duplicated(weight))
sum(duplicated(hourly_step))

# Clean the data
weight_info <- weight %>%
  select(Id, Date, WeightKg, WeightPounds, BMI, IsManualReport, LogId)

sleep_day <- sleep_day %>%
  distinct()

# Separate date and time
sleep_day_new <- sleep_day %>%
  separate(SleepDay, c("date", "time"), " ")

weight_info_new <- weight_info %>%
  separate(Date, c("date", "time"), " ")

# Find unique rows
n_distinct(daily_activity$Id)
n_distinct(sleep_day_new$Id)
n_distinct(weight_info_new$Id)
n_distinct(hourly_step$Id)
```
# Analyze
## Change Data Format and Summary:

### Convert date columns to Date type and check data structure:

```r
daily_activity$ActivityDate = as.Date(daily_activity$ActivityDate, "%m/%d/%Y") 
glimpse(daily_activity)

sleep_day_new$date = as.Date(sleep_day_new$date, "%m/%d/%y")
glimpse(sleep_day_new)

weight_info_new$date = as.Date(weight_info_new$date, "%m/%d/%y")
glimpse(weight_info_new)
```

### Summary statistics for datasets:

```r
daily_activity %>% 
  select(TotalSteps, TotalDistance, VeryActiveMinutes, FairlyActiveMinutes, LightlyActiveMinutes,
         Calories) %>% summary()

sleep_day_new %>% select(TotalSleepRecords, TotalMinutesAsleep, TotalTimeInBed) %>% summary()

weight_info_new %>% select(WeightKg, WeightPounds, BMI) %>% summary()
```

### Add a new column for weekdays and merge datasets:

```r
daily_activity <- daily_activity %>% mutate(Weekday = weekdays(as.Date(ActivityDate, "%m/%d/%Y")))
```
### merge data
```r
merge1 <- merge(sleep_day_new, weight_info_new, by = "Id", all = TRUE)
merge_data <- merge(merge1, daily_activity, by = "Id", all = TRUE)
```
# Order from Monday to Sunday for plotting
```r
merge_data$Weekday <- factor(merge_data$Weekday, levels = c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"))

# Save merged data
write_csv(merge_data, "merge_data.csv")
```
# Check for NA and duplicates in merged data
```r
sum(is.na(merge_data))
sum(duplicated(merge_data))
n_distinct(merge_data$Id)
```
### Add a new column for weekdays and merge datasets:

```r
daily_activity <- daily_activity %>% mutate(Weekday = weekdays(as.Date(ActivityDate, "%m/%d/%Y")))

merge1 <- merge(sleep_day_new, weight_info_new, by = "Id", all = TRUE)
merge_data <- merge(merge1, daily_activity, by = "Id", all = TRUE)
```
# Order from Monday to Sunday for plotting
```r
merge_data$Weekday <- factor(merge_data$Weekday, levels = c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"))

# Save merged data
write_csv(merge_data, "merge_data.csv")

# Check for NA and duplicates in merged data
sum(is.na(merge_data))
sum(duplicated(merge_data))
n_distinct(merge_data$Id)
```


# Share
 ## visualization
### Total Active Minutes per Weekday:
```r
# Calculate total active minutes
daily_activity$total_active_minitus <- daily_activity$VeryActiveMinutes + daily_activity$FairlyActiveMinutes + daily_activity$LightlyActiveMinutes

# Plot Total Active Minutes per Weekday
ggplot(daily_activity, aes(x= Weekday, y = total_active_minitus)) +
  geom_bar(stat = "identity", fill = "green") +
  labs(x = "Weekday", y = "Total Active Minutes", title = "Total Active Minutes per Weekday")
```
This bar chart shows the total active minutes for each weekday, highlighting that users are most active on the middle days of the week.
### Relation Between Sleep and Awake Time:

![active minitus per week](https://github.com/user-attachments/assets/d27ebadd-3a1f-4a7d-b118-b89c620cb56c)


# Calculate awake time in bed
```r
sleep_day_new$awake_time_in_bed <- sleep_day_new$TotalTimeInBed - sleep_day_new$TotalMinutesAsleep

# Plot relationship between Total Minutes Asleep and Awake Time in Bed
ggplot(sleep_day_new, aes(x=TotalMinutesAsleep, y= awake_time_in_bed)) +
  geom_point(color = "blue") +
  geom_smooth(color = "orange") +
  labs(x = "Total Minutes Asleep", y = "Awake Time in Bed", title = "Relation Between Sleep and Awake Time in Bed")
```
This scatter plot explores the relationship between the total minutes of sleep and the time spent awake in bed.

![sleep and time in bed](https://github.com/user-attachments/assets/81c63ae2-2764-4999-a432-4fa1fb661886)

 as we see  spending 300 to 450 minutes (5 to 7.5 hours) in bed report better sleep quality with fewer awakenings, aligning closely with the CDC's recommendation of at least 7 hours of sleep per night;

### Hourly Steps:

```
# Convert ActivityHour to POSIXct and extract hour
hourly_step$ActivityHour <- as.POSIXct(hourly_step$ActivityHour, format="%m/%d/%Y %I:%M:%S %p")
hourly_step$Hour <- format(hourly_step$ActivityHour, format= "%H")

# Plot hourly steps
ggplot(data=hourly_step, aes(x=Hour, y=StepTotal, fill=Hour)) +
  geom_bar(stat="identity") +
  labs(title="Hourly Steps")
```
This bar chart shows the total number of steps taken in each hour of the day.
![hourly steps](https://github.com/user-attachments/assets/fafa4a81-aea4-4386-8061-ba7f0414de42)

as we see people are more active during he time of 5 to 7 pm and take more steps

### Percentage of Active Minutes by Activity Level:

```r
# Calculate total minutes for each activity level
Sedentary <- sum(daily_activity$SedentaryMinutes)
Lightly <- sum(daily_activity$LightlyActiveMinutes)
Fairly <- sum(daily_activity$FairlyActiveMinutes)
Active <- sum(daily_activity$VeryActiveMinutes)

activity_minutes <- c(Sedentary, Lightly, Fairly, Active)

# Calculate the percentage of active minutes for each level
activity_percent <- round(activity_minutes / sum(activity_minutes) * 100, 1)

# Create a pie chart
par(bg= "white")
legend_labels <- c("Sedentary", "Lightly Active", "Fairly Active", "Very Active")
pie3D(activity_percent, labels=paste0(activity_percent, "%"),
      main="Percentage of Active Minutes by Activity Level", 
      col=c("darkred", "purple", "yellow", "green"),
      border="lightgrey", labelcex = 0.7)
legend("topright", legend_labels, cex=0.9,
       fill=c("darkred", "purple", "yellow", "green"), x.intersp = 1, y.intersp = 1)
```
![pie](https://github.com/user-attachments/assets/c6e2af83-4012-44c6-b268-4efc283f87cf)

the pie chart clearly show that 81.3% time people stay sedentary and spent  only 2.8% time in activity.
