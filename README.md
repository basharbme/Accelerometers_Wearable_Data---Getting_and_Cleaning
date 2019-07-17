

#### We are loading the "dplyr" library (piping and its select function will be used)
    library(dplyr)
#### We are downloading the zip file
    filename <- "projectfiles_UCI_HAR_Dataset"
    zipURL <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
    download.file(zipURL, filename)
#### We are checking if folder exists and unzipping it
    if (!dir.exists(filename)){unzip(filename)}
#### We are checking names of the directories available in our woring directory in order to localise the one concerning us (e.g. "UCI HAR Dataset")
    list.dirs(getwd())
---
### 1.
#### We are Loading all necessary files creating corresponding data frames:
##### the complete list of variables of each feature vector
    features <- read.table("UCI HAR Dataset/features.txt", col.names = c("n","functions"))
##### - the complete list of activities and the corresponding class labels
    activities <- read.table("UCI HAR Dataset/activity_labels.txt", col.names = c("code", "activity"))
##### - the measurements of both sets (test and subject) for axes X and Y
    x_test <- read.table("UCI HAR Dataset/test/X_test.txt", col.names = features$functions)
    y_test <- read.table("UCI HAR Dataset/test/y_test.txt", col.names = "code")
    x_train <- read.table("UCI HAR Dataset/train/X_train.txt", col.names = features$functions)
    y_train <- read.table("UCI HAR Dataset/train/y_train.txt", col.names = "code")
##### - the subjects ids corresponding to each row of measurements for the training and the test sets
    subject_test <- read.table("UCI HAR Dataset/test/subject_test.txt", col.names = "subject")
    subject_train <- read.table("UCI HAR Dataset/train/subject_train.txt", col.names = "subject")

#### We are merging the rows of the training and the test sets for axe X
    x <- rbind(x_train, x_test)
#### We are merging the rows of the training and the test sets for axe Y
    y <- rbind(y_train, y_test)
#### We are merging the rows of subjects ids
    subject <- rbind(subject_train, subject_test)
#### We are merging the columns of subjects ids and measurements for axe X and Y for both sets (train, test) creating one unique data set.
	unique_data <- cbind(subject, x, y)
---
### 2.
#### We are extracting only the mean and standard deviation variable for each measurement, that is to say we select only the columns whose name contains the sequences of letters "mean" or "std"
	tidy_data1 <- unique_data %>% select(subject, code, contains("mean"), 	contains("std"))
---
### 3.
#### We are changing the names of the activities.
#### We are replacing the codes with descriptions according to the key data frame "activities"
	tidy_data1$code <- activities[tidy_data1$code, 2]
---
### 4.
#### We are replacing the labels of the variables of the data set with descriptive variable names (according to the description of the researchers)
	names(tidy_data1)[2] = "Subject"
	names(tidy_data1)[2] = "Activity"
	names(tidy_data1)<-gsub("^f", "Frequency", names(tidy_data1))
	names(tidy_data1)<-gsub("^t", "Time", names(tidy_data1))
	names(tidy_data1)<-gsub("-freq()", "Frequency", names(tidy_data1), ignore.case = TRUE)
	names(tidy_data1)<-gsub("-mean()", "Mean", names(tidy_data1), ignore.case = TRUE)
	names(tidy_data1)<-gsub("-std()", "STandardDeviation",names(tidy_data1), ignore.case = TRUE)
	names(tidy_data1)<-gsub("Acc", "Accelerometer", names(tidy_data1))
	names(tidy_data1)<-gsub("angle", "Angle", names(tidy_data1))
	names(tidy_data1)<-gsub("BodyBody", "Body", names(tidy_data1))
	names(tidy_data1)<-gsub("gravity", "Gravity", names(tidy_data1))
	names(tidy_data1)<-gsub("Gyro", "Gyroscope", names(tidy_data1))
	names(tidy_data1)<-gsub("Mag", "Magnitude", names(tidy_data1))
	names(tidy_data1)<-gsub("tBody", "TimeBody", names(tidy_data1))
	
#### We are replacing the double dots with one, and we are deleting the dots at the end of the variable names (when existed)	
	names(tidy_data1)<-gsub("..", ".", names(tidy_data1), fixed=TRUE)
	names(tidy_data1)<-gsub("\\.$", "", names(tidy_data1))

---
### 5.
#### We are creating a new data set grouping the measurements of the previous one by subject and activity and calculating the average values for each activity and each subject
	Data <- tidy_data1 %>%
	        group_by(Subject, Activity) %>%
	        summarise_all(funs(mean))
#### We are assigning the final data frame into a correspnding .txt file)
	write.table(Data, "Data.txt")
