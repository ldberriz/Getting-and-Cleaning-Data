Getting and Cleaning-Data
=========================
## Script

### How does the script work?

R script run_analysis.R does the following: 

1.  Merges the training and the test sets to create one data set.

2.  Extracts only the measurements on the mean and standard deviation for each measurement. 

3.  Uses descriptive activity names to name the activities in the data set

4.  Appropriately labels the data set with descriptive variable names. 

5.  From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.

#### Script description

Load the appropriate packages

Use the require function.  If require() retuns false the package has not been installed and should be ibnstalled.  
packages installed are "data.table" and "reshape2".

Create a vector activity_labels with the values of activity levels
```
activity_labels <- read.table("activity_labels.txt")[,2]
```
Create a vector features with the data column names
```
features <- read.table("features.txt")[,2]
```
Extract only the measurements on the mean and standard deviation for each measurement. Create a vector of TRUE / FALSE values that determines which observation to extract.  This will be only the ones that have 'mean' and 'std' in their names.
```
extract_features <- grepl("mean|std", features)
```
Load the data and process X_test & y_test data.
```
X_test <- read.table("./test/X_test.txt")
y_test <- read.table("./test/y_test.txt")
subject_test <- read.table("./test/subject_test.txt")
```

```
names(X_test) = features
```
Extract only the measurements of the mean and standard deviation for each measurement.
```
X_test = X_test[,extract_features]
```
Load activity labels
```
y_test[,2] = activity_labels[y_test[,1]]
names(y_test) = c("Activity_ID", "Activity_Label")
names(subject_test) = "subject"
```
Bind data
```
test_data <- cbind(as.data.table(subject_test), y_test, X_test)
```
Load and process X_train & y_train data.
```
X_train <- read.table("./train/X_train.txt")
y_train <- read.table("./train/y_train.txt")
subject_train <- read.table("./train/subject_train.txt")
names(X_train) = features
```
Extract only the measurements on the mean and standard deviation for each measurement.
```
X_train = X_train[,extract_features]
```
Load activity data
```
y_train[,2] = activity_labels[y_train[,1]]
names(y_train) = c("Activity_ID", "Activity_Label")
names(subject_train) = "subject"
```
Bind data
```
train_data <- cbind(as.data.table(subject_train), y_train, X_train)
```
Merge test and train data
```
data = rbind(test_data, train_data)
id_labels   = c("subject", "Activity_ID", "Activity_Label")
data_labels = setdiff(colnames(data), id_labels)
melt_data      = melt(data, id = id_labels, measure.vars = data_labels)
```
Apply mean function to dataset using dcast function
```
tidy_data   = dcast(melt_data, subject + Activity_Label ~ variable, mean)
write.table(tidy_data, file = "./tidy_data.txt", row.names = FALSE)
```
##Code book 

The tidy_data set contains 180 observations od 81 variables each for a total of 14580 data points.
The first two columns of the data table include the variables 
* subject  as an integer variable which identifies the observation.
* Activity_Label as a Factor w/ 6 levels WALKING
2, WALKING_UPSTAIRS
3, WALKING_DOWNSTAIRS
4, SITTING
5, STANDING
6, LAYING.


The features selected for this database come from the accelerometer and gyroscope 3-axial raw signals tAcc-XYZ and tGyro-XYZ. These time domain signals (prefix 't' to denote time) were captured at a constant rate of 50 Hz. Then they were filtered using a median filter and a 3rd order low pass Butterworth filter with a corner frequency of 20 Hz to remove noise. Similarly, the acceleration signal was then separated into body and gravity acceleration signals (tBodyAcc-XYZ and tGravityAcc-XYZ) using another low pass Butterworth filter with a corner frequency of 0.3 Hz. 

Subsequently, the body linear acceleration and angular velocity were derived in time to obtain Jerk signals (tBodyAccJerk-XYZ and tBodyGyroJerk-XYZ). Also the magnitude of these three-dimensional signals were calculated using the Euclidean norm (tBodyAccMag, tGravityAccMag, tBodyAccJerkMag, tBodyGyroMag, tBodyGyroJerkMag). 

Finally a Fast Fourier Transform (FFT) was applied to some of these signals producing fBodyAcc-XYZ, fBodyAccJerk-XYZ, fBodyGyro-XYZ, fBodyAccJerkMag, fBodyGyroMag, fBodyGyroJerkMag. (Note the 'f' to indicate frequency domain signals). 

These signals were used to estimate variables of the feature vector for each pattern:  
'-XYZ' is used to denote 3-axial signals in the X, Y and Z directions.

tBodyAcc-XYZ
tGravityAcc-XYZ
tBodyAccJerk-XYZ
tBodyGyro-XYZ
tBodyGyroJerk-XYZ
tBodyAccMag
tGravityAccMag
tBodyAccJerkMag
tBodyGyroMag
tBodyGyroJerkMag
fBodyAcc-XYZ
fBodyAccJerk-XYZ
fBodyGyro-XYZ
fBodyAccMag
fBodyAccJerkMag
fBodyGyroMag
fBodyGyroJerkMag

Additional vectors obtained by averaging the signals in a signal window sample. These are used on the angle() variable:

gravityMean
tBodyAccMean
tBodyAccJerkMean
tBodyGyroMean
tBodyGyroJerkMean


The data is then organized using the variable identifier as above and appending the pbservation of interest such as mean or standard deviation.  All these variables are numeric variables.

tBodyAcc-mean()-X              
tBodyAcc-mean()-Y
tBodyAcc-mean()-Z
tBodyAcc-std()-X   
tBodyAcc-std()-Y   
tBodyAcc-std()-Z   
tGravityAcc-mean()-X           
tGravityAcc-mean()-Y           
tGravityAcc-mean()-Z           
tGravityAcc-std()-X            
tGravityAcc-std()-Y            
tGravityAcc-std()-Z            

