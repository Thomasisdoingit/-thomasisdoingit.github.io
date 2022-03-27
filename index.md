---
title: "Capstone Case Study"
author: "Thomas"
date: '2022-03-24'
output: html_document
---



## Business task 


The business task we will examine is the following:

##### *How do annual members and casual riders use Cyclistic services differently?*

## Prepare

Using the data from Cyclistic (Cyclistic is a fictional company the data used comes from Motivate International Inc. under this [licence](https://ride.divvybikes.com/data-license-agreement)).

The data is downloaded and will be treated locally using RStudio.
We will be using data from 2019 because the monthly data from 2021 is incomplete. Furthermore we could suppose measures taken during the pandemic have changed the use patterns in a way that is not representative of long term use. The data is supposed reliable and original for this case study, it is comprehensive and still relatively current.

The data includes:  

 * The duration of each trip  
 * Start and end points  
 * The status of the client: member or casual rider  
 * Their birth year and gender  

This should allow us to compare demographics between member and casual riders, as well as the differences in usage of the service (routes, travel time etc).

## Process

* Tools
For this analysis I have chosen to use Rstudio over spreadsheets because of the large size of the dataset.

I could have used SQL but I believe if we have the possibility to treat the data locally it's better to use it.

Furthermore I would like to sharpen my R skills.

* Data integrity 

First I loaded each csv file:


``` {r, eval=F, echo=T}
library(readr)
library(dplyr)
library(tidyr)
library(janitor)

Quarter_1_trips <- read.csv(file='Divvy_Trips_2019_Q1.csv')
#I have a look at it
str(Quarter_1_trips)
View(Quarter_1_trips)

#loading each quarterly file and checking for data integrity
Quarter_2_trips <- read.csv(file='Divvy_Trips_2019_Q2.csv')
#I have a look at it
str(Quarter_2_trips)
View(Quarter_2_trips)
```  

Here we realized the column names for Q2 are inconsistent with the rest of the data set, so we decided to change them using the following:
```{r, eval=FALSE, echo=TRUE}
#list Q1 variable names
ls(Quarter_1_trips)

#replace the bad column names with the ones consistent with the rest of the dataset.
names(Quarter_2_trips) <- c("trip_id", "start_time","end_time", "bikeid","tripduration","from_station_id","from_station_name","to_station_id","to_station_name","usertype","gender","birthyear")
View(Quarter_2_trips)
```

After correcting the column names we can merge all the data frames together:
```{r, eval=F, echo= T}
Semester_1 <- full_join(Quarter_1_trips, Quarter_2_trips)
glimpse(Semester_1)

Semester_2<- full_join(Quarter_3_trips,Quarter_4_trips)
glimpse(Semester_2)
 
Yearly_trips <- full_join(Semester_1,Semester_2)
glimpse(Yearly_trips)
```
We can start checking for duplicates or empty cells.
```{r, eval= F, echo=TRUE}
Yearly_trips %>%  get_dupes(trip_id)
```

Then we added a column with the day of the week calculated using `start_time` and `wday()`.  

```{r, eval= F, echo=TRUE}

# create a temp frame for dates
library(lubridate)
date_trips <- Yearly_trips %>%  select(trip_id, start_time)
View(date_trips)

View(date_trips)
#join date_trips and Yearly_trips to populate day_of_week
Yearly_trips_day <- left_join(Yearly_trips, date_trips)
glimpse(Yearly_trips_day)


```

## Analysis 

First we start by filtering data depending on the status of the client: member or casual: 
```{r, eval= F, echo=TRUE}
# Create casual table 
Yearly_casuals <-  filter(Yearly_clean, usertype == "Customer")
glimpse(Yearly_casuals)

#compare total and casual trips
nrow(Yearly_clean) - nrow(Yearly_casuals)

#Create suscriber/ member table
Yearly_members <-  filter(Yearly_clean, usertype == "Subscriber")
glimpse(Yearly_members)

```
When doing `summary(Yearly_casuals)` and `summary(Yearly_members)` I realize a lot of my variable are in the wrong types.  

I tried using this as shown [here] (https://cran.r-project.org/web/packages/hablar/vignettes/convert.html)  
  
  
```{r, eval= F, echo=TRUE}
install.packages("hablar")
library(hablar)
Yearly_casuals %>%  type.convert(num("tripduration"),as_date("start_time"), as_date("end_time"))

summary(Yearly_casuals)
summary(Yearly_members)
```  

But I get an error:
![the error] (https://github.com/Thomasisdoingit/thomasisdoingit.github.io/blob/db5197072053a6f2d2a53b86d18d264cda80ffc7/wrong%20data%20type.png)
