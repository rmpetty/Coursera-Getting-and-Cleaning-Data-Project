Cleaning Data Week 4 Final Project
Ruthie Macha Petty
October 1, 2017

# OBJECTIVE
The README that explains the analysis files is clear and understandable.

Data Sources Used in this project (all needed files were placed in one folder for processing):
https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip

1.	subject_test (2947 rows; 1 column) - the individuals who performed each activity (1 to 30)
2.	subject_train (7352 rows; 1 column) - the individuals who performed each activity (1 to 30)
3.	Y_test (2947 rows; 1 column) - test labels:  these are the activities
4.	Y_train (7352 rows; 1 column) - labels:  these are the activities
5.	X_test (2947 rows; 561 columns) - test set
6.	X_train (7352 rows; 561 columns) - test set
7.	Activities (6 rows, 2 columns) - translates the activity code to a description
8.	Features (561 rows, 2 columms) - includes the column names for each of the 561 types of values

# Credits 
1. Script to load multiple packages
  https://gist.github.com/RobertMyles/96665be044540e9ac668ee697dac6db7
2. General project advice from David Hood
  https://thoughtfulbloke.wordpress.com/2015/09/09/getting-and-cleaning-the-assignment/

# Required packages
Packages <- c("dplyr", "readr", "tidyr", "stats")

# Read Data
Test and train data were tidied separately, then merged.
Pull in data from the following files, named as shown:

  subjectTest <- read.table("subject_test.txt")
  subjectTrain <- read.table("subject_train.txt")
  testLabels <- read.table("Y_test.txt")
  trainLabels <- read.table("Y_train.txt")
  testSet <- read.table("X_test.txt")
  trainSet <- read.table("X_train.txt")
  features <- read.table("features.txt")
  activityLabels <- read.table("activity_labels.txt")

Data processed as follows:

# Tidy the subject data
 1. Change the column name in the subject files to "subject."
names(subjectTest) <- ("subject")
names(subjectTrain) <- ("subject")
 2. Add a column called "source" with a value of "test" or "train"
  Creating a reference column that could be used once data is merged
subjectTest <- mutate(subjectTest, source = "test")
subjectTrain <- mutate(subjectTrain, source = "train")

# Tidy the activity labels (Y)
 1. Change the column name to "activity"
names(testLabels) <- ("activity")
names(trainLabels) <- ("activity")
 2. Match each activity number to its description 
names(activityLabels) <- c("ID", "DESC")
testLabels <- activityLabels$DESC[match(testLabels$activity, activityLabels$ID)]
trainLabels <- activityLabels$DESC[match(trainLabels$activity, activityLabels$ID)]

# Tidy the data sets (X)
 1. Add features as column headers
f <- features$V2
names(testSet) <- f
names(trainSet) <- f
 2. Extract all "mean" and "std" measures columns (regardless of case)
 Returns 86 columns
extract <- grep("[Mm]ean|[Ss]td", names(testSet))
eTest <- testSet[extract]
eTrain <- trainSet[extract]

# Join all columns (subject + activity + data sets)
bindTest <- cbind(subjectTest, testLabels)
bindTrain <- cbind(subjectTrain, trainLabels)
 Tidy column names before adding data sets
names(bindTest) <- c("subject", "source", "activity")
names(bindTrain) <- c("subject", "source", "activity")
Test <- cbind(bindTest, eTest)
Train <- cbind(bindTrain, eTrain)

# Join all rows (Test and Train)
mergedData <- rbind(Test, Train)

# Create a second, independent tidy data set with the average of each variable for each activity and each subject.
finalSummary <-
  mergedData %>%
  group_by(subject, activity) %>%
  summarise_if(is.numeric, mean, na.rm = TRUE)

# Output audit data
dim(Test)
dim(Train)
dim(mergedData)
dim(finalSummary)
mergedData[1:5,1:5]
finalSummary[1:5,1:5]

# Results
 > dim(Test)
 [1] 2947   89
 >   dim(Train)
 [1] 7352   89
 >   dim(mergedData)
 [1] 10299    89
 >   dim(finalSummary)
 [1] 180  88
 >   mergedData[1:5,1:5]
 subject source activity tBodyAcc-mean()-X tBodyAcc-mean()-Y
 1       2   test STANDING         0.2571778       -0.02328523
 2       2   test STANDING         0.2860267       -0.01316336
 3       2   test STANDING         0.2754848       -0.02605042
 4       2   test STANDING         0.2702982       -0.03261387
 5       2   test STANDING         0.2748330       -0.02784779
 >   finalSummary[1:5,1:5]
  A tibble: 5 x 5
  Groups:   subject [1]
 subject           activity `tBodyAcc-mean()-X` `tBodyAcc-mean()-Y`
 <int>             <fctr>               <dbl>               <dbl>
   1       1             LAYING           0.2215982        -0.040513953
 2       1            SITTING           0.2612376        -0.001308288
 3       1           STANDING           0.2789176        -0.016137590
 4       1            WALKING           0.2773308        -0.017383819
 5       1 WALKING_DOWNSTAIRS           0.2891883        -0.009918505

# Submission instructions
Please upload the tidy data set created in step 5 of the instructions. 
Please upload your data set as a txt file created with write.table() using row.name=FALSE

setwd("~/GitHub/Coursera-Getting-and-Cleaning-Data-Project")
write.table(finalSummary, file="step5tidydatarmp.txt", sep = "\t", row.names=FALSE)
