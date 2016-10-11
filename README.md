# Johns-Hopkins-R-Programming
Here are works for Johns Hopkins R Programming course

Including scripts for the following assignments:

## Programming Assignment 1

### Introduction

For this first programming assignment you will write three functions that are meant to interact with dataset that accompanies this assignment. The dataset is contained in a zip file specdata.zip that you can download from the Coursera web site.

### Data

The zip file contains 332 comma-separated-value (CSV) files containing pollution monitoring data for fine particulate matter (PM) air pollution at 332 locations in the United States. Each file contains data from a single monitor and the ID number for each monitor is contained in the file name. For example, data for monitor 200 is contained in the file "200.csv". Each file contains three variables:

* Date: the date of the observation in YYYY-MM-DD format (year-month-day)

* sulfate: the level of sulfate PM in the air on that date (measured in micrograms per cubic meter)

* nitrate: the level of nitrate PM in the air on that date (measured in micrograms per cubic meter)

For this programming assignment you will need to unzip this file and create the directory 'specdata'. Once you have unzipped the zip file, do not make any modifications to the files in the 'specdata' directory. In each file you'll notice that there are many days where either sulfate or nitrate (or both) are missing (coded as NA). This is common with air pollution monitoring data in the United States.

### Part 1

Write a function named 'pollutantmean' that calculates the mean of a pollutant (sulfate or nitrate) across a specified list of monitors. The function 'pollutantmean' takes three arguments: 'directory', 'pollutant', and 'id'. Given a vector monitor ID numbers, 'pollutantmean' reads that monitors' particulate matter data from the directory specified in the 'directory' argument and returns the mean of the pollutant across all of the monitors, ignoring any missing values coded as NA. A prototype of the function is as follows:

```
pollutantmean <- function(directory, pollutant, id = 1:332) {
        ## 'directory' is a character vector of length 1 indicating
        ## the location of the CSV files

        ## 'pollutant' is a character vector of length 1 indicating
        ## the name of the pollutant for which we will calculate the
        ## mean; either "sulfate" or "nitrate".

        ## 'id' is an integer vector indicating the monitor ID numbers
        ## to be used

        ## Return the mean of the pollutant across all monitors list
        ## in the 'id' vector (ignoring NA values)
}
```

Example output:

```
source("pollutantmean.R")
pollutantmean("specdata", "sulfate", 1:10)
## [1] 4.064
pollutantmean("specdata", "nitrate", 70:72)
## [1] 1.706
pollutantmean("specdata", "nitrate", 23)
## [1] 1.281
```

### Part 2

Write a function that reads a directory full of files and reports the number of completely observed cases in each data file. The function should return a data frame where the first column is the name of the file and the second column is the number of complete cases. A prototype of this function follows:

```
complete <- function(directory, id = 1:332) {
        ## 'directory' is a character vector of length 1 indicating
        ## the location of the CSV files

        ## 'id' is an integer vector indicating the monitor ID numbers
        ## to be used
        
        ## Return a data frame of the form:
        ## id nobs
        ## 1  117
        ## 2  1041
        ## ...
        ## where 'id' is the monitor ID number and 'nobs' is the
        ## number of complete cases
}
```

Example output:

```
source("complete.R")
complete("specdata", 1)
##   id nobs
## 1  1  117
complete("specdata", c(2, 4, 8, 10, 12))
##   id nobs
## 1  2 1041
## 2  4  474
## 3  8  192
## 4 10  148
## 5 12   96
complete("specdata", 30:25)
##   id nobs
## 1 30  932
## 2 29  711
## 3 28  475
## 4 27  338
## 5 26  586
## 6 25  463
complete("specdata", 3)
##   id nobs
## 1  3  243
```

### Part 3

Write a function that takes a directory of data files and a threshold for complete cases and calculates the correlation between sulfate and nitrate for monitor locations where the number of completely observed cases (on all variables) is greater than the threshold. The function should return a vector of correlations for the monitors that meet the threshold requirement. If no monitors meet the threshold requirement, then the function should return a numeric vector of length 0. A prototype of this function follows:

```
corr <- function(directory, threshold = 0) {
        ## 'directory' is a character vector of length 1 indicating
        ## the location of the CSV files

        ## 'threshold' is a numeric vector of length 1 indicating the
        ## number of completely observed observations (on all
        ## variables) required to compute the correlation between
        ## nitrate and sulfate; the default is 0

        ## Return a numeric vector of correlations
}
```

For this function you will need to use the `cor` function in R which calculates the correlation between two vectors. Please read the help page for this function via `?cor` and make sure that you know how to use it.

Example output:

```
source("corr.R")
source("complete.R")
cr <- corr("specdata", 150)
head(cr)
## [1] -0.01896 -0.14051 -0.04390 -0.06816 -0.12351 -0.07589
summary(cr)
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
## -0.2110 -0.0500  0.0946  0.1250  0.2680  0.7630
cr <- corr("specdata", 400)
head(cr)
## [1] -0.01896 -0.04390 -0.06816 -0.07589  0.76313 -0.15783
summary(cr)
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
## -0.1760 -0.0311  0.1000  0.1400  0.2680  0.7630
cr <- corr("specdata", 5000)
summary(cr)
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
## 
length(cr)
## [1] 0
cr <- corr("specdata")
summary(cr)
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
## -1.0000 -0.0528  0.1070  0.1370  0.2780  1.0000
length(cr)
## [1] 323
```

## Programming Assignment 2

### Introduction

This second programming assignment will require you to write an R
function that is able to cache potentially time-consuming computations.
For example, taking the mean of a numeric vector is typically a fast
operation. However, for a very long vector, it may take too long to
compute the mean, especially if it has to be computed repeatedly (e.g.
in a loop). If the contents of a vector are not changing, it may make
sense to cache the value of the mean so that when we need it again, it
can be looked up in the cache rather than recomputed. In this
Programming Assignment you will take advantage of the scoping rules of
the R language and how they can be manipulated to preserve state inside
of an R object.

### Example: Caching the Mean of a Vector

In this example we introduce the `<<-` operator which can be used to
assign a value to an object in an environment that is different from the
current environment. Below are two functions that are used to create a
special object that stores a numeric vector and caches its mean.

The first function, `makeVector` creates a special "vector", which is
really a list containing a function to

1.  set the value of the vector
2.  get the value of the vector
3.  set the value of the mean
4.  get the value of the mean

<!-- -->

    makeVector <- function(x = numeric()) {
            m <- NULL
            set <- function(y) {
                    x <<- y
                    m <<- NULL
            }
            get <- function() x
            setmean <- function(mean) m <<- mean
            getmean <- function() m
            list(set = set, get = get,
                 setmean = setmean,
                 getmean = getmean)
    }

The following function calculates the mean of the special "vector"
created with the above function. However, it first checks to see if the
mean has already been calculated. If so, it `get`s the mean from the
cache and skips the computation. Otherwise, it calculates the mean of
the data and sets the value of the mean in the cache via the `setmean`
function.

    cachemean <- function(x, ...) {
            m <- x$getmean()
            if(!is.null(m)) {
                    message("getting cached data")
                    return(m)
            }
            data <- x$get()
            m <- mean(data, ...)
            x$setmean(m)
            m
    }

### Assignment: Caching the Inverse of a Matrix

Matrix inversion is usually a costly computation and there may be some
benefit to caching the inverse of a matrix rather than computing it
repeatedly (there are also alternatives to matrix inversion that we will
not discuss here). Your assignment is to write a pair of functions that
cache the inverse of a matrix.

Write the following functions:

1.  `makeCacheMatrix`: This function creates a special "matrix" object
    that can cache its inverse.
2.  `cacheSolve`: This function computes the inverse of the special
    "matrix" returned by `makeCacheMatrix` above. If the inverse has
    already been calculated (and the matrix has not changed), then
    `cacheSolve` should retrieve the inverse from the cache.

Computing the inverse of a square matrix can be done with the `solve`
function in R. For example, if `X` is a square invertible matrix, then
`solve(X)` returns its inverse.

For this assignment, assume that the matrix supplied is always
invertible.

## Programming Assignment 3

### Introduction

Download the file ProgAssignment3-data.zip file containing the data for Programming Assignment 3 from the Coursera web site. Unzip the file in a directory that will serve as your working directory. When you start up R make sure to change your working directory to the directory where you unzipped the data.
The data for this assignment come from the Hospital Compare web site (http://hospitalcompare.hhs.gov) run by the U.S. Department of Health and Human Services. The purpose of the web site is to provide data and information about the quality of care at over 4,000 Medicare-certified hospitals in the U.S. This dataset es- sentially covers all major U.S. hospitals. This dataset is used for a variety of purposes, including determining whether hospitals should be fined for not providing high quality care to patients (see http://goo.gl/jAXFX for some background on this particular topic).
The Hospital Compare web site contains a lot of data and we will only look at a small subset for this assignment. The zip file for this assignment contains three files
• outcome-of-care-measures.csv: Contains information about 30-day mortality and readmission rates for heart attacks, heart failure, and pneumonia for over 4,000 hospitals.
• hospital-data.csv: Contains information about each hospital.
• Hospital_Revised_Flatfiles.pdf: Descriptions of the variables in each file (i.e the code book).
A description of the variables in each of the files is in the included PDF file named Hospital_Revised_Flatfiles.pdf. This document contains information about many other files that are not included with this programming assignment. You will want to focus on the variables for Number 19 (“Outcome of Care Measures.csv”) and Number 11 (“Hospital Data.csv”). You may find it useful to print out this document (at least the pages for
Tables 19 and 11) to have next to you while you work on this assignment. In particular, the numbers of the variables for each table indicate column indices in each table (i.e. “Hospital Name” is column 2 in the outcome-of-care-measures.csv file).

### 1 Plot the 30-day mortality rates for heart attack

Read the outcome data into R via the read.csv function and look at the first few rows.

There are many columns in this dataset. You can see how many by typing ncol(outcome) (you can see the number of rows with the nrow function). In addition, you can see the names of each column by typing names(outcome) (the names are also in the PDF document.
To make a simple histogram of the 30-day death rates from heart attack (column 11 in the outcome dataset), run

###  2 Finding the best hospital in a state
Write a function called best that take two arguments: the 2-character abbreviated name of a state and an outcome name. The function reads the outcome-of-care-measures.csv file and returns a character vector with the name of the hospital that has the best (i.e. lowest) 30-day mortality for the specified outcome in that state. The hospital name is the name provided in the Hospital.Name variable. The outcomes can be one of “heart attack”, “heart failure”, or “pneumonia”. Hospitals that do not have data on a particular outcome should be excluded from the set of hospitals when deciding the rankings.
Handling ties. If there is a tie for the best hospital for a given outcome, then the hospital names should be sorted in alphabetical order and the first hospital in that set should be chosen (i.e. if hospitals “b”, “c”, and “f” are tied for best, then hospital “b” should be returned).
The function should use the following template.

best <- function(state, outcome) {
        ## Read outcome data
}

Check that state and outcome are valid

Return hospital name in that state with lowest 30-day death rate

The function should check the validity of its arguments. If an invalid state value is passed to best, the function should throw an error via the stop function with the exact message “invalid state”. If an invalid outcome value is passed to best, the function should throw an error via the stop function with the exact message “invalid outcome”.

Save your code for this function to a file named best.R.

Use the submit script provided to submit your solution to this part. There are 3 tests that need to be passed
for this part of the assignment.

### 3 Ranking hospitals by outcome in a state

Write a function called rankhospital that takes three arguments: the 2-character abbreviated name of a state (state), an outcome (outcome), and the ranking of a hospital in that state for that outcome (num). The function reads the outcome-of-care-measures.csv file and returns a character vector with the name of the hospital that has the ranking specified by the num argument. For example, the call
rankhospital("MD", "heart failure", 5)

would return a character vector containing the name of the hospital with the 5th lowest 30-day death rate for heart failure. The num argument can take values “best”, “worst”, or an integer indicating the ranking (smaller numbers are better). If the number given by num is larger than the number of hospitals in that state, then the function should return NA. Hospitals that do not have data on a particular outcome should be excluded from the set of hospitals when deciding the rankings.
Handling ties. It may occur that multiple hospitals have the same 30-day mortality rate for a given cause of death. In those cases ties should be broken by using the hospital name. For example, in Texas (“TX”), the hospitals with lowest 30-day mortality rate for heart failure are shown here.

Check that state and outcome are valid

Return hospital name in that state with the given rank 30-day death rate

The function should check the validity of its arguments. If an invalid state value is passed to best, the function should throw an error via the stop function with the exact message “invalid state”. If an invalid outcome value is passed to best, the function should throw an error via the stop function with the exact message “invalid outcome”.

Save your code for this function to a file named rankhospital.R.

Use the submit script provided to submit your solution to this part. There are 4 tests that need to be passed for this part of the assignment.

### 4 Ranking hospitals in all states

Write a function called rankall that takes two arguments: an outcome name (outcome) and a hospital rank- ing (num). The function reads the outcome-of-care-measures.csv file and returns a 2-column data frame containing the hospital in each state that has the ranking specified in num. For example the function call rankall("heart attack", "best") would return a data frame containing the names of the hospitals that are the best in their respective states for 30-day heart attack death rates. The function should return a value for every state (some may be NA). The first column in the data frame is named hospital, which contains the hospital name, and the second column is named state, which contains the 2-character abbreviation for the state name. Hospitals that do not have data on a particular outcome should be excluded from the set of hospitals when deciding the rankings.
Handling ties. The rankall function should handle ties in the 30-day mortality rates in the same way that the rankhospital function handles ties.

The function should use the following template.

rankall <- function(outcome, num = "best") {
        ## Read outcome data
}

Check that state and outcome are valid

For each state, find the hospital of the given rank

Return a data frame with the hospital names and the (abbreviated) state name

NOTE: For the purpose of this part of the assignment (and for efficiency), your function should NOT call the rankhospital function from the previous section.
The function should check the validity of its arguments. If an invalid outcome value is passed to rankall, the function should throw an error via the stop function with the exact message “invalid outcome”. The num variable can take values “best”, “worst”, or an integer indicating the ranking (smaller numbers are better). If the number given by num is larger than the number of hospitals in that state, then the function should return NA.

Save your code for this function to a file named rankall.R.

