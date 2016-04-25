##Libraries
The libraries used to run the code are 
- data.table
- readr
- plyr
- dplyr

##Variables
- dtXTrain, dtXTest, dtYTrain, dtYTest,  dtFeatures, dtSubjectTrain, dtSubjectTest and dtActivity are data tables used to load the data from the files
- dtX, dtY dtSubject and dtAllData are data tables created in the code to merge the data tables created from the files
- selectedColumns has the columns filtered by mean() and std(), as instructed.
- dtSelectedColumns is the data table obtained when filtering by the columns selected above
- dtWithActivityNames is the data table containing the activities as descriptive names
- dtGroupedWithAverage is the final data table containing the average for the variables selected grouped by subject and activity

##Code
The 5 steps of the script are:

#Step 1. Merges the training and the test sets to create one data set
1. Read the files X_train, y_train, X_test, y_test into data tables. The X files have fixed length so the read_fwf() function is used to specify multiple fields with specified length. 
2. Both X files are merged and their columns names are assigned from another data table that contains all the features. 
3. The Y files are then merged and their column named as "Activity_Number".
4. Subject files are read and merged as data tables
5. Finally the X, Y and Subject data tables are merged into another data table called dtAllData
    
#Step 2. Extracts only the measurements on the mean and standard deviation for each measurement.

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
