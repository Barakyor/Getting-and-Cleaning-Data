**Getting and Cleaning Data - Week 4: Exercise R**

The purpose of this project is to demonstrate your ability to collect, work with, and clean a data set. The goal is to prepare tidy data that can be used for later analysis. You will be graded by your peers on a series of yes/no questions related to the project. You will be required to submit: 1) a tidy data set as described below, 2) a link to a Github repository with your script for performing the analysis, and 3) a code book that describes the variables, the data, and any transformations or work that you performed to clean up the data called CodeBook.md. You should also include a README.md in the repo with your scripts. This repo explains how all of the scripts work and how they are connected.

One of the most exciting areas in all of data science right now is wearable computing - see for example this article . Companies like Fitbit, Nike, and Jawbone Up are racing to develop the most advanced algorithms to attract new users. The data linked to from the course website represent data collected from the accelerometers from the Samsung Galaxy S smartphone. A full description is available at the site where the data was obtained:

http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones

Here are the data for the project:

https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip

You should create one R script called run_analysis.R that does the following.

Merges the training and the test sets to create one data set.
Extracts only the measurements on the mean and standard deviation for each measurement.
Uses descriptive activity names to name the activities in the data set
Appropriately labels the data set with descriptive variable names.
From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.
Good luck!

_**Code and explanations**

# Load necessary libraries
library(dplyr)

# Set the directory where you want to save the dataset
directory <- "C:/Users/User/Documents/R Training/"

# Create the directory if it doesn't exist
if (!file.exists(directory)) {
  dir.create(directory)
}

# Set the URL for the dataset
url <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"

# Set the file name
file_name <- paste0(directory, "UCI HAR Dataset.zip")

# Download the dataset
download.file(url, file_name, mode = "wb")

# Unzip the dataset
unzip(file_name, exdir = directory)

# Set the working directory to the unzipped dataset
setwd(paste0(directory, "UCI HAR Dataset"))

# Step 2: Read the necessary files
features <- read.table("features.txt", col.names = c("index", "feature"))
activity_labels <- read.table("activity_labels.txt", col.names = c("index", "activity"))
subject_train <- read.table("train/subject_train.txt", col.names = "subject")
X_train <- read.table("train/X_train.txt", col.names = features$feature)
y_train <- read.table("train/y_train.txt", col.names = "activity")
subject_test <- read.table("test/subject_test.txt", col.names = "subject")
X_test <- read.table("test/X_test.txt", col.names = features$feature)
y_test <- read.table("test/y_test.txt", col.names = "activity")

# Step 3: Merge the training and test sets
X <- rbind(X_train, X_test)
y <- rbind(y_train, y_test)
subject <- rbind(subject_train, subject_test)
data <- cbind(subject, y, X)

# Step 4: Extract only the measurements on the mean and standard deviation
mean_std_features <- grep("mean\\(\\)|std\\(\\)", features$feature)
data <- data[, c(1, 2, mean_std_features + 2)]

# Step 5: Use descriptive activity names
data$activity <- factor(data$activity, levels = activity_labels$index, labels = activity_labels$activity)

# Step 6: Label the dataset with descriptive variable names
names(data) <- gsub("\\(|\\)", "", names(data))
names(data) <- tolower(names(data))
names(data) <- gsub("^t", "time", names(data))
names(data) <- gsub("^f", "frequency", names(data))
names(data) <- gsub("acc", "accelerometer", names(data))
names(data) <- gsub("gyro", "gyroscope", names(data))
names(data) <- gsub("mag", "magnitude", names(data))
names(data) <- gsub("bodybody", "body", names(data))

# Step 7: Create a tidy dataset with the average of each variable for each activity and each subject
tidy_data <- aggregate(. ~ subject + activity, data, mean)

# Step 8: Write the tidy dataset to a file
write.table(tidy_data, "tidy_data.txt", row.names = FALSE)
