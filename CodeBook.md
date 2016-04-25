#Libraries


#Variables


#Code



#################
#Library imports#
#################
library(data.table)
install.packages("readr")
library(readr)
library(plyr)
library(dplyr)

##################################################################
#1. Merges the training and the test sets to create one data set.#
##################################################################

#Read the x files for train and test, with 651 variables of fixed lenght (16) into a data table
dtXTrain <- data.table(read_fwf("./X_train.txt", fwf_widths(c(rep(16,561)))))
dtXTest <- data.table(read_fwf("./X_test.txt", fwf_widths(c(rep(16,561)))))

#Merge the x files
dtX <- rbind(dtXTrain, dtXTest) 
    
#Read the features file into a data table and assign the variable names to the columns
dtFeatures <- data.table(read.csv("./features.txt", sep = " ", header = FALSE))
names(dtFeatures) <- c("Column","Feature")

#Assign the variable names extracted from dtFeatures to dtX
names(dtX) <- as.character(dtFeatures$Feature)

#Read the y files with the activity number into a data table
dtYTrain <- data.table(read.csv("./y_train.txt", header = FALSE))
dtYTest <- data.table(read.csv("./y_test.txt", header = FALSE))

#Merge the y files
dtY <- rbind(dtYTrain, dtYTest) 

#Assign the variable name Activity_Number to the column
names(dtY) <- "Activity_Number"

#Read the subject files with the subject number into a data table
dtSubjectTrain <- data.table(read.csv("./subject_train.txt", header = FALSE))
dtSubjectTest <- data.table(read.csv("./subject_test.txt", header = FALSE))

#Merge the y files
dtSubject <- rbind(dtSubjectTrain, dtSubjectTest) 

#Assign the variable name Subject_Number to the column
names(dtSubject) <- "Subject_Number"

#Merge x_train, y_file and subject_train files and their columns into a one
dtAllData <- cbind(dtX, dtY, dtSubject)

############################################################################################
#2. Extracts only the measurements on the mean and standard deviation for each measurement.#
############################################################################################

#Select columns with mean or std included in the name
selectedColumns <- grep("\\bmean()\\b|\\bstd()\\b|Activity_Number|Subject_Number", names(dtAllData))
dtSelectedColumns <- select(dtAllData, selectedColumns)

############################################################################
#3. Uses descriptive activity names to name the activities in the data set.#
############################################################################

#Read the activity_labels file with 2 variables into a data table
dtActivity <- data.table(read.csv("./activity_labels.txt", sep = " ", header = FALSE))

#Assign the variable names Activity_Number and Activity_Name to the 2 columns
names(dtActivity) <- c("Activity_Number","Activity_Name")

#Merge the dtSelectedColumns with the dtActivityTrain to provide the activity names for any activity number
dtWithActivityNames <- merge(dtSelectedColumns, dtActivity, by = "Activity_Number")

#Delete the activity number as it is no longer needed
dtWithActivityNames[, Activity_Number:=NULL]

#######################################################################
#4. Appropriately labels the data set with descriptive variable names.#
#######################################################################

names(dtWithActivityNames) <- sub("^t","Time ",names(dtWithActivityNames))
names(dtWithActivityNames) <- sub("^f","Frequency ",names(dtWithActivityNames))
names(dtWithActivityNames) <- gsub("Body","Body ",names(dtWithActivityNames))
names(dtWithActivityNames) <- sub("Acc","Acceleration ",names(dtWithActivityNames))
names(dtWithActivityNames) <- sub("Gravity","Gravity ",names(dtWithActivityNames))
names(dtWithActivityNames) <- sub("Gyro","Gyroscope ",names(dtWithActivityNames))
names(dtWithActivityNames) <- sub("Jerk","Jerk ",names(dtWithActivityNames))
names(dtWithActivityNames) <- sub("Mag","Magnitude ",names(dtWithActivityNames))
names(dtWithActivityNames) <- sub("-mean\\(\\)","Mean",names(dtWithActivityNames))
names(dtWithActivityNames) <- sub("-std\\(\\)","Standard Deviation",names(dtWithActivityNames))
names(dtWithActivityNames) <- sub("-X"," (X)",names(dtWithActivityNames))
names(dtWithActivityNames) <- sub("-Y"," (Y)",names(dtWithActivityNames))
names(dtWithActivityNames) <- sub("-Z"," (Z)",names(dtWithActivityNames))

#5. From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.

#Apply colMeans on the data table, grouped by Activity_Name and Subject_Number
dtGroupedWithAverage <- ddply(dtWithActivityNames, .(Activity_Name, Subject_Number), function(x) colMeans(x[, 1:66]))

#Write the data table into a TXT file
write.table(dtGroupedWithAverage, "tidy_average_data.txt", row.name=FALSE)
