---
title: 'Getting and Cleaning Data: Peer Assessment'
output: html_document
---
## Introduction
One of the most exciting areas in all of data science right now is wearable computing - see for example [this article](http://www.insideactivitytracking.com/data-science-activity-tracking-and-the-battle-for-the-worlds-top-sports-brand/). Companies like Fitbit, Nike, and Jawbone Up are racing to develop the most advanced algorithms to attract new users. The data linked to from the course website represent data collected from the accelerometers from the Samsung Galaxy S smartphone. A full description is available at the site where the data was obtained:
http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones

Here are the data for the project:
https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip

## Tasks
Create one R script called `run_analysis.R` that does the following:

1. Merges the training and the test sets to create one data set.

2. Extracts only the measurements on the mean and standard deviation for each measurement. 

3. Uses descriptive activity names to name the activities in the data set

4. Appropriately labels the data set with descriptive variable names. 

5. From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.

## Implementation
#### Download and unzip data
Although not explicitly stated, this obviously is a prerequisite. We check if the data is already there, otherwise download and unzip it. In the last step capture URL and timestamp in a `downloaded.txt` file.

```r
if (!file.exists("data")) { # only download if not already done before
  dir.create("data")
  url <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
  download.file(url = url, destfile = "data/data.zip", method = "curl")
  unzip(zipfile = "data/data.zip")
  # write a little note when and from where the file was downloaded
  cat(paste("File data.zip downloaded", date(), "from", url), file = "data /downloaded.txt")
}
```
#### Merge the training and the test sets to create one data set
In the first steps we just read all required files, both metadata and data files.

```r
# go to where the data is
setwd("UCI HAR Dataset")
# Read the metadata
activity_labels <- read.table(file="activity_labels.txt")
features <- read.table(file="features.txt")
# Read the "train" files
train_x <- read.table(file="train/X_train.txt")
train_y <- read.table(file="train/y_train.txt")
train_subject <- read.table(file="train/subject_train.txt")
# Read the "test" files
test_x <- read.table(file="test/X_test.txt")
test_y <- read.table(file="test/y_test.txt")
test_subject <- read.table(file="test/subject_test.txt")
```
Then we merge the individual sets, appropriately label the comlumns and create the full data set.

```r
# Merge train and test files
x <- rbind(train_x, test_x)
y <- rbind(train_y, test_y)
subject <- rbind(train_subject, test_subject)
# Rename the columns
names(x) <- features[,2]
names(y) <- "Activity"
names(subject) <- "Subject"
# Create the full data set
data <- cbind(subject, y, x)
```
#### Extract only the measurements on the mean and standard deviation for each measurement
We use a simple `grep()` with regular expression to find the matching columns. All other columns can be dropped.

```r
matches <- grep("Activity|Subject|mean\\(\\)|std\\(\\)", colnames(data))
# Drop all other columns
data <- data[, matches]
```
#### From the data set, create a second, independent tidy data set with the average of each variable for each activity and each subject
We use the `aggregate(..., mean)` function to calculate the averages.

```r
tidy <- aggregate(data, by = list(data$Activity, data$Subject), mean)
tidy <- tidy[, -c(1,2)] # drop "Group.1" & "Group.2" columns
```

#### Use descriptive activity names to name the activities in the data set
The descriptive names can be obtained from the `activity_labels` metadata.

```r
tidy$Activity <- as.factor(tidy$Activity)
levels(tidy$Activity) <- activity_labels[,2]
tidy$Activity <- as.character(tidy$Activity)
```

#### Write a tidy.txt file
In the last step the tidy data is writen to a plain text file.

```r
write.table(tidy, file="tidy.txt", row.names=FALSE)
```
Cheers, that's it!
