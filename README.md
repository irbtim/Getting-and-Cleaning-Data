[Data for the project] (https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip)

*R script does the following*
*1. Merges the training and the test sets to create one data set.*
*2. Extracts only the measurements on the mean and standard deviation for each measurement.*
*3. Uses descriptive activity names to name the activities in the data set*
*4. Appropriately labels the data set with descriptive variable names.*
*5. From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.*


#####1. Merges the training and the test sets to create one data set.

#####Read in the data from files

`features <- read.table('./features.txt',header=FALSE)`

`activityType <- read.table('./activity_labels.txt',header=FALSE)`

`subjectTrain <- read.table('./train/subject_train.txt',header=FALSE)`

`subjectTest <- read.table('./test/subject_test.txt',header=FALSE)`

`xTrain <- read.table('./train/x_train.txt',header=FALSE)`

`xTest <- read.table('./test/x_test.txt',header=FALSE)`

`yTrain <- read.table('./train/y_train.txt',header=FALSE)`

`yTest <- read.table('./test/y_test.txt',header=FALSE)`

#####Assigin column names to the imported data
`colnames(activityType) <- c('activityId','activityType')`

`colnames(subjectTrain) <- "subjectId"`

`colnames(subjectTest) <- "subjectId"`

`colnames(xTrain) <- features[,2]`

`colnames(xTest) <- features[,2]`

`colnames(yTrain) <- "activityId"`

`colnames(yTest) <- "activityId"`

#####Create the final training data set & test data set
`trainingSet <- cbind(yTrain,subjectTrain,xTrain)`

`testSet <- cbind(yTest,subjectTest,xTest)`

#####Combine training set and test set
`dataSet <- rbind(trainingSet,testSet)`

`colNames  = colnames(dataSet)`

#####2. Extracts only the measurements on the mean and standard deviation for each measurement.
#####Create a Vector with TRUE values for the ID, mean() & stddev() columns and FALSE for others
`v<-(grepl("activity..",colNames) | grepl("subject..",colNames) | grepl("-mean..",colNames) & !grepl("-meanFreq..",colNames) & !grepl("mean..-",colNames) | grepl("-std..",colNames) & !grepl("-std()..-",colNames))`

`dataSet <- dataSet[v==TRUE]`

#####3. Uses descriptive activity names to name the activities in the data set.
#####Merge the dataSet with the acitivityType

`dataSet <- merge(dataSet,activityType,by='activityId',all.x=TRUE)`

`colNames  = colnames(dataSet)`

#####4. Appropriately labels the data set with descriptive variable names.
#####Cleaning up names

`for (i in 1:length(colNames))` 

`{` 

`colNames[i] <- gsub("\\()","",colNames[i])`

`colNames[i] <- gsub("-std$","StdDev",colNames[i])`

`colNames[i] <- gsub("-mean","Mean",colNames[i])`

`colNames[i] <- gsub("^(t)","time",colNames[i])`

`colNames[i] <- gsub("^(f)","freq",colNames[i])`

`colNames[i] <- gsub("([Gg]ravity)","Gravity",colNames[i])`

`colNames[i] <- gsub("([Bb]ody[Bb]ody|[Bb]ody)","Body",colNames[i])`

`colNames[i] <- gsub("[Gg]yro","Gyro",colNames[i])`

`colNames[i] <- gsub("AccMag","AccMagnitude",colNames[i])`

`colNames[i] <- gsub("([Bb]odyaccjerkmag)","BodyAccJerkMagnitude",colNames[i])`

`colNames[i] <- gsub("JerkMag","JerkMagnitude",colNames[i])`

`colNames[i] <- gsub("GyroMag","GyroMagnitude",colNames[i])`

`}`

#####Assigning the new column names for the dataSet
`colnames(dataSet) <- colNames`
#####5. From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.

#####Create a new table, noActivity column

`noActivity <- dataSet[,names(dataSet) != 'activityType']`

#####Aggregate the table to include the mean of each variable for each activity and each subject
`aggregateData <- aggregate(noActivity[,names(noActivity) != c('activityId','subjectId')],by=list(activityId=noActivity$activityId,subjectId = noActivity$subjectId),mean)`

#####Merging the aggregateData with activityType to include acitvity names

`aggregateData <- merge(aggregateData,activityType,by='activityId',all.x=TRUE);`

#####Export the aggregateData
 
`write.table(aggregateData, './tidyData.txt',row.names=TRUE,sep='\t')`
