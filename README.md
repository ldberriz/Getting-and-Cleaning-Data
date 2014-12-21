Getting-and-Cleaning-Data
=========================
## Script

### How does the script work?

R script run_analysis.R does the following: 

* Merges the training and the test sets to create one data set.

* Extracts only the measurements on the mean and standard deviation for each measurement. 

* Uses descriptive activity names to name the activities in the data set

* Appropriately labels the data set with descriptive variable names. 

* From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.

### Script description

Load the appropriate packages

Use the require function.  If require() retuns false the package has not been installed and should be ibnstalled.  
packages installed are "data.table" and "reshape2".

Create a vector with the values of activity levels
...
  activity_labels <- read.table("activity_labels.txt")[,2]
...
Create a vector with the data column names
...
  features <- read.table("features.txt")[,2]
...
Extract only the measurements on the mean and standard deviation for each measurement. Create a vector of TRUE / FALSE that determines which observation to extract.  This will be only the ones that have 'mean' and 'std' in their names.
extract_features <- grepl("mean|std", features)

Load the data and process X_test & y_test data.
...
  X_test <- read.table("./test/X_test.txt")

  y_test <- read.table("./test/y_test.txt")

  subject_test <- read.table("./test/subject_test.txt")

  names(X_test) = features
...
Extract only the measurements of the mean and standard deviation for each measurement.
...
  X_test = X_test[,extract_features]
...
Load activity labels
...
  y_test[,2] = activity_labels[y_test[,1]]

  names(y_test) = c("Activity_ID", "Activity_Label")

  names(subject_test) = "subject"
...
Bind data
...
test_data <- cbind(as.data.table(subject_test), y_test, X_test)
...
Load and process X_train & y_train data.
...
X_train <- read.table("./train/X_train.txt")

y_train <- read.table("./train/y_train.txt")

subject_train <- read.table("./train/subject_train.txt")

names(X_train) = features
...
Extract only the measurements on the mean and standard deviation for each measurement.
...
X_train = X_train[,extract_features]
...
Load activity data
...
y_train[,2] = activity_labels[y_train[,1]]

names(y_train) = c("Activity_ID", "Activity_Label")

names(subject_train) = "subject"
...

Bind data
...
train_data <- cbind(as.data.table(subject_train), y_train, X_train)
...
Merge test and train data
...
data = rbind(test_data, train_data)

id_labels   = c("subject", "Activity_ID", "Activity_Label")

data_labels = setdiff(colnames(data), id_labels)

melt_data      = melt(data, id = id_labels, measure.vars = data_labels)
...
Apply mean function to dataset using dcast function
...
tidy_data   = dcast(melt_data, subject + Activity_Label ~ variable, mean)

write.table(tidy_data, file = "./tidy_data.txt", row.names = FALSE)
...
##Code book 

The code book

For almost any data set, the measurements you calculate will need to be described in more detail than you will sneak into the spreadsheet. The code book contains this information. At minimum it should contain:

Information about the variables (including units!) in the data set not contained in the tidy data
Information about the summary choices you made
Information about the experimental study design you used
In our genomics example, the analyst would want to know what the unit of measurement for each clinical/demographic variable is (age in years, treatment by name/dose, level of diagnosis and how heterogeneous). They would also want to know how you picked the exons you used for summarizing the genomic data (UCSC/Ensembl, etc.). They would also want to know any other information about how you did the data collection/study design. For example, are these the first 20 patients that walked into the clinic? Are they 20 highly selected patients by some characteristic like age? Are they randomized to treatments?

A common format for this document is a Word file. There should be a section called "Study design" that has a thorough description of how you collected the data. There is a section called "Code book" that describes each variable and its units.

How to code variables

When you put variables into a spreadsheet there are several main categories you will run into depending on their data type:

Continuous
Ordinal
Categorical
Missing
Censored
Continuous variables are anything measured on a quantitative scale that could be any fractional number. An example would be something like weight measured in kg. Ordinal data are data that have a fixed, small (< 100) number of levels but are ordered. This could be for example survey responses where the choices are: poor, fair, good. Categorical data are data where there are multiple categories, but they aren't ordered. One example would be sex: male or female. Missing data are data that are missing and you don't know the mechanism. You should code missing values as NA. Censored data/make up/ throw away missing observations.

In general, try to avoid coding categorical or ordinal variables as numbers. When you enter the value for sex in the tidy data, it should be "male" or "female". The ordinal values in the data set should be "poor", "fair", and "good" not 1, 2 ,3. This will avoid potential mixups about which direction effects go and will help identify coding errors.

Always encode every piece of information about your observations using text. For example, if you are storing data in Excel and use a form of colored text or cell background formatting to indicate information about an observation ("red variable entries were observed in experiment 1.") then this information will not be exported (and will be lost!) when the data is exported as raw text. Every piece of data should be encoded as actual text that can be exported.

The instruction list/script

You may have heard this before, but reproducibility is kind of a big deal in computational science. That means, when you submit your paper, the reviewers and the rest of the world should be able to exactly replicate the analyses from raw data all the way to final results. If you are trying to be efficient, you will likely perform some summarization/data analysis steps before the data can be considered tidy.

The ideal thing for you to do when performing summarization is to create a computer script (in R, Python, or something else) that takes the raw data as input and produces the tidy data you are sharing as output. You can try running your script a couple of times and see if the code produces the same output.

In many cases, the person who collected the data has incentive to make it tidy for a statistician to speed the process of collaboration. They may not know how to code in a scripting language. In that case, what you should provide the statistician is something called pseudocode. It should look something like:

Step 1 - take the raw file, run version 3.1.2 of summarize software with parameters a=1, b=2, c=3
Step 2 - run the software separately for each sample
Step 3 - take column three of outputfile.out for each sample and that is the corresponding row in the output data set
You should also include information about which system (Mac/Windows/Linux) you used the software on and whether you tried it more than once to confirm it gave the same results. Ideally, you will run this by a fellow student/labmate to confirm that they can obtain the same output file you did.

What you should expect from the analyst
When you turn over a properly tidied data set it dramatically decreases the workload on the statistician. So hopefully they will get back to you much sooner. But most careful statisticians will check your recipe, ask questions about steps you performed, and try to confirm that they can obtain the same tidy data that you did with, at minimum, spot checks.

You should then expect from the statistician:

An analysis script that performs each of the analyses (not just instructions)
The exact computer code they used to run the analysis
All output files/figures they generated.
This is the information you will use in the supplement to establish reproducibility and precision of your results. Each of the steps in the analysis should be clearly explained and you should ask questions when you don't understand what the analyst did. It is the responsibility of both the statistician and the scientist to understand the statistical analysis. You may not be able to perform the exact analyses without the statistician's code, but you should be able to explain why the statistician performed each step to a labmate/your principal investigator.
