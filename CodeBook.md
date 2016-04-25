#Libraries
The libraries used to run the code are 
- data.table
- readr
- plyr
- dplyr

#Variables
- dtXTrain, dtXTest, dtYTrain, dtYTest,  dtFeatures, dtSubjectTrain, dtSubjectTest and dtActivity are data tables used to load the data from the files
- dtX, dtY dtSubject and dtAllData are data tables created in the code to merge the data tables created from the files
- selectedColumns has the columns filtered by mean() and std(), as instructed.
- dtSelectedColumns is the data table obtained when filtering by the columns selected above
- dtWithActivityNames is the data table containing the activities as descriptive names
- dtGroupedWithAverage is the final data table containing the average for the variables selected grouped by subject and activity

#Code
The 5 steps of the script are:

##Step 1. Merges the training and the test sets to create one data set
1. Read the files X_train, y_train, X_test, y_test into data tables. The X files have fixed length so the read_fwf() function is used to specify multiple fields with specified length. 
2. Both X files are merged and their columns names are assigned from another data table that contains all the features. 
3. The Y files are then merged and their column named as "Activity_Number".
4. Subject files are read and merged as data tables.
5. Finally the X, Y and Subject data tables are merged into another data table called dtAllData.
    
##Step 2. Extracts only the measurements on the mean and standard deviation for each measurement.
1. The columns with mean() or std() are then filtered and a new data table is created with the name dtSelectedColumns.

##Step 3. Uses descriptive activity names to name the activities in the data set.
1. The activity_labels file is read and load into the data table dtActivity with 2 columns: "Activity_Number" and "Activity_Name".
2. Merge the data tables dtSelectedColumns and dtActivityTrain by the "Activity_Number"
3. Delete the "Activity_Number" from the data table dtWithActivityNames, as it is no longer needed

##Step 4. Appropriately labels the data set with descriptive variable names.
1. Abbreviatons like Acc, Gyro, etc. are replaced by a more descriptive name.

##Step 5. From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.
1. Apply colMeans on the data table dtGroupedWithAverage, grouped by Activity_Name and Subject_Number
2. Write the data table dtGroupedWithAverage into a TXT file called "tidy_average_data.txt"
