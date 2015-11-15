## Introduction

It is now possible to collect a large amount of data about personal
movement using activity monitoring devices such as a
[Fitbit](http://www.fitbit.com), [Nike
Fuelband](http://www.nike.com/us/en_us/c/nikeplus-fuelband), or
[Jawbone Up](https://jawbone.com/up). These type of devices are part of
the "quantified self" movement -- a group of enthusiasts who take
measurements about themselves regularly to improve their health, to
find patterns in their behavior, or because they are tech geeks. But
these data remain under-utilized both because the raw data are hard to
obtain and there is a lack of statistical methods and software for
processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring
device. This device collects data at 5 minute intervals through out the
day. The data consists of two months of data from an anonymous
individual collected during the months of October and November, 2012
and include the number of steps taken in 5 minute intervals each day.

## Data

The data for this assignment can be downloaded from the course web
site:

* Dataset: [Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip) [52K]

The variables included in this dataset are:

* **steps**: Number of steps taking in a 5-minute interval (missing
    values are coded as `NA`)

* **date**: The date on which the measurement was taken in YYYY-MM-DD
    format

* **interval**: Identifier for the 5-minute interval in which
    measurement was taken




The dataset is stored in a comma-separated-value (CSV) file and there
are a total of 17,568 observations in this
dataset.


## Assignment

This assignment will be described in multiple parts. You will need to
write a report that answers the questions detailed below. Ultimately,
you will need to complete the entire assignment in a **single R
markdown** document that can be processed by **knitr** and be
transformed into an HTML file.

Throughout your report make sure you always include the code that you
used to generate the output you present. When writing code chunks in
the R markdown document, always use `echo = TRUE` so that someone else
will be able to read the code. **This assignment will be evaluated via
peer assessment so it is essential that your peer evaluators be able
to review the code for your analysis**.

For the plotting aspects of this assignment, feel free to use any
plotting system in R (i.e., base, lattice, ggplot2)

Fork/clone the [GitHub repository created for this
assignment](http://github.com/rdpeng/RepData_PeerAssessment1). You
will submit this assignment by pushing your completed files into your
forked repository on GitHub. The assignment submission will consist of
the URL to your GitHub repository and the SHA-1 commit ID for your
repository state.

NOTE: The GitHub repository also contains the dataset for the
assignment so you do not have to download the data separately.



### Loading and preprocessing the data
Here's what I did:
```
library(ggplot2)    #needed for making pretty graph
library(data.table) #needed for working with missing values
options(scipen=999) #disable scientific notation

activity_data <- read.csv('./data/activity.csv')
```
Ok, so now that we've got the data, let me explain what I'm doing with it.

First, I'm going to convert it to a data table, and give it a key for the interval column (*origDT*).  Then I'm going to calculate/aggregate the mean for the interval data (*average_steps_per_int*).  In other words, we'll have for each iterval, accross all days, what the average steps taken were.

Then convert **that** result (a dataframe) to a data table, giving it also a key for the interval column (*avgDT*).  Oh, and I create an intermediate data table  because I need the steps column to be of type double (*intermDT*).
```
origDT<-data.table(activity_data,key=c("interval"))

average_steps_per_int<-aggregate(.~interval,origDT,mean)
avgDT<-data.table(average_steps_per_int,key=c("interval"))
intermDT<-origDT[,steps:=as.double(steps)]
```
Then I join the two data tables together, using the common key between them.  Then I set the steps column from orig table = steps from average table where steps in orig is null. In other words, where there is missing data for a given day & interval, put in the average across all the non-missing days for that interval.

```
outDT<-intermDT[avgDT]
outDT<-outDT[is.na(steps),steps:=i.steps]
outDT<-outDT[,list(steps,date,interval)]
```
Last step of cleanup/tidy/prep: I make a new column that has 2 factors, *weekday* and *weekend* depending on the result of the *weekdays()* function.  Make one last data table to work with.
```
new_col<-factor(weekdays(as.Date(outDT$date)) %in%  c('Saturday','Sunday'),labels=c("weekday","weekend"))
finalDT<-outDT[,day_type:=new_col]

```
Going to process the data and set variables we'll need later on.

```
total_steps_per_day<-aggregate(.~date,activity_data,sum)
mean_steps_per_day<-mean(total_steps_per_day$steps)
median_steps_per_day<-median(total_steps_per_day$steps)

interval_with_most_steps_on_average<-average_steps_per_int[
  average_steps_per_int$steps==max(average_steps_per_int$steps),1]

```
Here we reaggregate with the final datatable, the one that includes the imputed data.
```
new_total_steps_per_day<-aggregate(.~date,finalDT,sum)
new_average_steps_per_day<-mean(new_total_steps_per_day$steps)
new_median_steps_per_day<-median(new_total_steps_per_day$steps)

avg_steps_by_daytype <-aggregate(.~interval+day_type,outDT,mean)
```
