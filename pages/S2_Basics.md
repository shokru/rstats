R Basics
================

-   [1 - GETTING STARTED](#getting-started)
    -   [Working directory](#working-directory)
    -   [Packages](#packages)
    -   [Importing data](#importing-data)
-   [2 - CREATING DATA](#creating-data)
    -   [Simple sequences](#simple-sequences)
    -   [Dataframes](#dataframes)
    -   [Dimensions](#dimensions)
    -   [Boolean (TRUE/FALSE) data](#boolean-truefalse-data)
-   [3 - HANDLING DATA IN PURE R](#handling-data-in-pure-r)
    -   [Extracting data](#extracting-data)
    -   [Writing / Replacing values](#writing-replacing-values)
    -   [Seeing data](#seeing-data)
-   [4 - LOOPS: 1st step towards programming](#loops-1st-step-towards-programming)
-   [5 - MISC. FUNCTIONS](#misc.-functions)
    -   [Statistics](#statistics)
    -   [Changing modes](#changing-modes)
    -   [The apply() function](#the-apply-function)
    -   [User-specified functions](#user-specified-functions)
    -   [Missing data](#missing-data)
-   [Exercises](#exercises)
    -   [Handling vectors & matrices](#handling-vectors-matrices)
    -   [Filtering in base R is tedious!](#filtering-in-base-r-is-tedious)
    -   [The importance of modes](#the-importance-of-modes)
    -   [Functions](#functions)

1 - GETTING STARTED
-------------------

### Working directory

The very first step after launching RStudio is to specify the working directory. You can do it directly in the Files pane (using the blue wheel), or via the setwd() function. The getwd() yields the location of the current working directory.

``` r
getwd()
```

    ## [1] "/Users/coqueret/Documents/IT/Cours/RStats/Git"

### Packages

Working with packages requires:
1 - an installation, only once (basically it downloads the code/files on your computer)
2 - an activation, each time you start RStudio

``` r
if(!require(openxlsx)) {install.packages("openxlsx") } # Only installs if missing
if(!require(readxl)) {install.packages("readxl") }     # Only installs if missing
# The hashtag is used for comments: the program does not read the line
library(openxlsx)
library(readxl)
library(tidyverse)
```

### Importing data

This is usually done directly in the user interface, or with packages like *openxlsx* or *readxl* (to import Excel files) with the function read.xlsx() or read\_excel(). The basic case looks like that: test\_data &lt;- read.xlsx("MyFile.xlsx")
OR
test\_data &lt;- read\_excel("MyFile.xlsx").
This stores your data into the test\_data variable. This assumes that the Excel file "MyFile.xlsx" exists in your working directory.

``` r
anes <- read.xlsx("anes.xlsx")
anes <- read_excel("anes.xlsx") # Same, with another packages
```

2 - CREATING DATA
-----------------

### Simple sequences

You can create data from scratch, using the colon operator for instance. Or the seq() function for sequences.

``` r
1:10 # All integers from 1 to 10
```

    ##  [1]  1  2  3  4  5  6  7  8  9 10

``` r
3:17 # All integers from 3 to 17
```

    ##  [1]  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17

``` r
seq(1,2,0.1) # The syntax is: seq(begin, end, step size)
```

    ##  [1] 1.0 1.1 1.2 1.3 1.4 1.5 1.6 1.7 1.8 1.9 2.0

A very important function: the c() function concatenates and encapsulates numbers (or text):

``` r
c(2,5,7)
```

    ## [1] 2 5 7

``` r
c(1:6,12:20)
```

    ##  [1]  1  2  3  4  5  6 12 13 14 15 16 17 18 19 20

``` r
c("R", " is ", "awesome")
```

    ## [1] "R"       " is "    "awesome"

Another way to concatenate data is to use row-bind and column-bind functions rbind() and cbind().

``` r
rbind(c(2,5,7),c(3,1,8)) # Binding rows
```

    ##      [,1] [,2] [,3]
    ## [1,]    2    5    7
    ## [2,]    3    1    8

``` r
cbind(c(2,5,7),c(3,1,8)) # Binding columns
```

    ##      [,1] [,2]
    ## [1,]    2    3
    ## [2,]    5    1
    ## [3,]    7    8

You can also fill in matrices: the syntax is matrix(DATA, nrow = r, ncol = c) where the length of DATA must be equal to r\*c! If one dimension is omitted, R will do the division.

``` r
m <- matrix(1:20, nrow = 4) 
m2 <- matrix(1:20, nrow = 4, byrow = T) # A matrix can be filled by rows or by columns
m
```

    ##      [,1] [,2] [,3] [,4] [,5]
    ## [1,]    1    5    9   13   17
    ## [2,]    2    6   10   14   18
    ## [3,]    3    7   11   15   19
    ## [4,]    4    8   12   16   20

``` r
m2
```

    ##      [,1] [,2] [,3] [,4] [,5]
    ## [1,]    1    2    3    4    5
    ## [2,]    6    7    8    9   10
    ## [3,]   11   12   13   14   15
    ## [4,]   16   17   18   19   20

``` r
t(m)        # t() is used for transposing; it works for vectors too! By default, vectors are columns
```

    ##      [,1] [,2] [,3] [,4]
    ## [1,]    1    2    3    4
    ## [2,]    5    6    7    8
    ## [3,]    9   10   11   12
    ## [4,]   13   14   15   16
    ## [5,]   17   18   19   20

``` r
m*m2        # This is term-by-term multiplication
```

    ##      [,1] [,2] [,3] [,4] [,5]
    ## [1,]    1   10   27   52   85
    ## [2,]   12   42   80  126  180
    ## [3,]   33   84  143  210  285
    ## [4,]   64  136  216  304  400

``` r
m%*%t(m2)   # This is matrix multiplication
```

    ##      [,1] [,2] [,3] [,4]
    ## [1,]  175  400  625  850
    ## [2,]  190  440  690  940
    ## [3,]  205  480  755 1030
    ## [4,]  220  520  820 1120

R is great to generate random data.

``` r
runif(15) # uniform distribution: 15 samples
```

    ##  [1] 0.83897283 0.79429790 0.52407071 0.23659196 0.95202209 0.10142787
    ##  [7] 0.84494161 0.12705258 0.45811340 0.06277804 0.23890984 0.38186320
    ## [13] 0.11732788 0.33101132 0.77597068

``` r
rnorm(10) # Gaussian distribution (parameters could be specified), 10 samples 
```

    ##  [1] -1.2248451 -0.8313384  0.8510247  2.5983142  1.4981450 -0.7406631
    ##  [7] -1.3440975  1.6551370  1.3776806 -1.4457856

An overview of all distributions available across R packages can be found here: <https://cran.r-project.org/web/views/Distributions.html>

### Dataframes

Datasets often mix text and numbers. R can do that too, with data frames (the modern version of dataframe is the tibble). Let's create one with the data.frame() function. We use the round() function which rounds up numbers.

``` r
nb_gender <- 7                                              # Number of people of each gender
Gender <- rep(c("Male"),nb_gender)                          # nb_gender men in total
Weight <- rnorm(nb_gender, mean = 70, sd = 8) %>% round()   # in kilos
Height <- rnorm(nb_gender, mean = 178, sd = 10) %>% round() # in cm
Age <- rnorm(nb_gender, mean = 40, sd = 7)  %>% round()  
data <- data.frame(Gender,Weight,Height,Age)                # data with only men
Gender <- rep(c("Female"),nb_gender)                        # nb_gender women in total
Weight <-  rnorm(nb_gender, 60, sd = 8)  %>% round()        # in kilos
Height <-  rnorm(nb_gender, 167, sd = 10)  %>% round()      # in cm
Age <- rnorm(nb_gender, mean = 40, sd = 7)  %>% round()  
data <- rbind(data, data.frame(Gender,Weight,Height,Age))   # grouping women with men
data
```

    ##    Gender Weight Height Age
    ## 1    Male     63    159  52
    ## 2    Male     90    187  36
    ## 3    Male     70    187  44
    ## 4    Male     61    166  39
    ## 5    Male     88    179  44
    ## 6    Male     68    181  38
    ## 7    Male     71    167  44
    ## 8  Female     57    169  30
    ## 9  Female     60    167  34
    ## 10 Female     49    158  30
    ## 11 Female     61    160  43
    ## 12 Female     54    179  40
    ## 13 Female     73    170  37
    ## 14 Female     69    161  37

You can use rownames() or colnames() to get or set the names of rows or columns: colnames(data).

### Dimensions

You can obtain the dimension of a matrix or data frame with the dim() function: dim(data). (Nb rows and nb columns). Each dimension can be obtained separately with nrow() and ncol() For vectors, the number of elements can be found with the length() function.

``` r
dim(data) 
```

    ## [1] 14  4

``` r
nrow(data)   # Number of rows
```

    ## [1] 14

``` r
ncol(data)   # Number of columns
```

    ## [1] 4

``` r
length(3:35) # Length of a vector
```

    ## [1] 33

### Boolean (TRUE/FALSE) data

In R, it is usefulto perform tests. For instance, given the sequence 1:12, we want to know which values are strictly greater than 6. The simple command 1:12&gt;6 will provide the answer: the statement is false for the first six elements (1 to 6) and true for the last six (7 to 12).

``` r
1:12>6
```

    ##  [1] FALSE FALSE FALSE FALSE FALSE FALSE  TRUE  TRUE  TRUE  TRUE  TRUE
    ## [12]  TRUE

3 - HANDLING DATA IN PURE R
---------------------------

### Extracting data

Accessing the values of a variable can be done with the square brackets \[\] thanks to indexing. For instance, the value in the third row and second column of data is data\[3,2\].
When columns have names, it is possible to use it to isolate a particular column with the dollar ( **$** ) operator:

``` r
data$Age # The Age column of data
```

    ##  [1] 52 36 44 39 44 38 44 30 34 30 43 40 37 37

Another way to proceed is to omit to specify the row numbers: since Height is the third column of data, then the result is the same with data\[,3\]. This give you all of the third column. Likewise, data\[3,\] will return all of the third row.

``` r
data[,3] # Third column
```

    ##  [1] 159 187 187 166 179 181 167 169 167 158 160 179 170 161

``` r
data[3,] # Third row
```

    ##   Gender Weight Height Age
    ## 3   Male     70    187  44

You can extract data with boolean vectors! For instance, if we want to select the people who are older than 42 years old: simple!

``` r
data$Age>42
```

    ##  [1]  TRUE FALSE  TRUE FALSE  TRUE FALSE  TRUE FALSE FALSE FALSE  TRUE
    ## [12] FALSE FALSE FALSE

will provide the corresponding indices. To extract the data, you just need to select the right rows and all columns:

``` r
data[data$Age > 42, ]                   # Mind the comma: we keep all columns!
```

    ##    Gender Weight Height Age
    ## 1    Male     63    159  52
    ## 3    Male     70    187  44
    ## 5    Male     88    179  44
    ## 7    Male     71    167  44
    ## 11 Female     61    160  43

``` r
data[data$Age > 42 & data$Weight >70, ] # the & operator allows to add sorting criteria
```

    ##   Gender Weight Height Age
    ## 5   Male     88    179  44
    ## 7   Male     71    167  44

Only the TRUE rows are kept. As we will see, the filter() function of the tidyverse does just that.

### Writing / Replacing values

Writing on data frames, vectors, or matrices can be done with the arrow operator:

``` r
data[3,2] <- 99
data[c(7,9),3] <- 166        # Replace 2 cells at a time! Seventh and ninth row on the third column.
data[c(6,8),3] <- c(199,176) # Same, but with 2 different values.
data                         # CHECK where the new values are!
```

    ##    Gender Weight Height Age
    ## 1    Male     63    159  52
    ## 2    Male     90    187  36
    ## 3    Male     99    187  44
    ## 4    Male     61    166  39
    ## 5    Male     88    179  44
    ## 6    Male     68    199  38
    ## 7    Male     71    166  44
    ## 8  Female     57    176  30
    ## 9  Female     60    166  34
    ## 10 Female     49    158  30
    ## 11 Female     61    160  43
    ## 12 Female     54    179  40
    ## 13 Female     73    170  37
    ## 14 Female     69    161  37

### Seeing data

Unlike in Excel, the data is not directly shown in R. You have to ask for it! To see the content of a variable, you have to type its name and press ENTER. Sometimes, the content of the variable is huge and cannot properly be displayed.
The head() function shows the first 6 lines and the tail() function shows the last 6 lines.

``` r
head(data, 8) # First 8 rows (default number of rows is 6)
```

    ##   Gender Weight Height Age
    ## 1   Male     63    159  52
    ## 2   Male     90    187  36
    ## 3   Male     99    187  44
    ## 4   Male     61    166  39
    ## 5   Male     88    179  44
    ## 6   Male     68    199  38
    ## 7   Male     71    166  44
    ## 8 Female     57    176  30

``` r
tail(data)    # Last 6 rows
```

    ##    Gender Weight Height Age
    ## 9  Female     60    166  34
    ## 10 Female     49    158  30
    ## 11 Female     61    160  43
    ## 12 Female     54    179  40
    ## 13 Female     73    170  37
    ## 14 Female     69    161  37

If you want to see the different possible values of a factor (categorical variables), you can use the levels() function.

``` r
levels(diamonds$clarity)
```

    ## [1] "I1"   "SI2"  "SI1"  "VS2"  "VS1"  "VVS2" "VVS1" "IF"

Though honestly, you would get the information using the summary() function as well - because there aren't too many of them.

4 - LOOPS: 1st step towards programming
---------------------------------------

There are usually several types of loops, but we will focus on the **for** loop. Its structure is simple: the idea is to repeat a task a finite number of times. This allows to automate the changes in a variable. For instance, the Fibonacci sequence:

``` r
nb <- 20    # Number of desired numbers
Fib <- 1    # First value
Fib[2] <- 1 # Second value
for(k in 3:nb){
   Fib[k] <- Fib[k-1] + Fib[k-2] # New value equals the sum of the 2 previous ones
}
Fib # Show the sequence
```

    ##  [1]    1    1    2    3    5    8   13   21   34   55   89  144  233  377
    ## [15]  610  987 1597 2584 4181 6765

**GOING FURTHER**: loops can take time. To speed things up, have a look at the map() family of functions. The apply functions are also cool (see below).

5 - MISC. FUNCTIONS
-------------------

### Statistics

Below, we present a few useful functions.

``` r
sqrt(5)                             # Square root
```

    ## [1] 2.236068

``` r
mean(c(2:6, 8:43))                  # Average value
```

    ## [1] 22.87805

``` r
sd(c(2:6, 8:43))                    # Standard deviation, use var() for the variance
```

    ## [1] 12.17004

``` r
v <- rnorm(18)                      # We generate a random vector
min(v)                              # Minimum
```

    ## [1] -2.369415

``` r
max(v)                              # Maximum
```

    ## [1] 1.613747

``` r
v                                   # A look at the whole vector
```

    ##  [1] -1.3901215  0.2143833  0.1238296 -2.3694148  1.2232405  1.6137474
    ##  [7]  1.2195476 -0.5027623 -0.1421806  0.2070257  0.5525198  0.5272807
    ## [13]  0.1125104  0.9756675  0.4112256  0.7606094  0.6218447 -0.6624647

``` r
which(v == max(v))                  # which() returns the index of the items that are TRUE inside a Boolean expression
```

    ## [1] 6

### Changing modes

Usual modes for variables are:
- logical (Boolean, TRUE or FALE), **NOTE**: they are also converted into number: FALSE = 0, TRUE = 1
- numeric (numbers),
- character (text),
- factor (unordered category) and
- ordered factor (ordered category)
It is sometimes possible to switch from one to another. One counter-example is: translating a charater into a number. For a factor, the levels is the values that can be taken by the variable. See levels(). Some examples below.

``` r
c("3", "8", "7")                        # Numbers viewed as text 
```

    ## [1] "3" "8" "7"

``` r
c("3", "8", "7") %>% as.numeric()       # Change the above into *true* numbers
```

    ## [1] 3 8 7

``` r
c(3,4,6) %>% as.character()             # The opposite: change fields into characters
```

    ## [1] "3" "4" "6"

``` r
data$Age %>% as.factor() %>% summary()  
```

    ## 30 34 36 37 38 39 40 43 44 52 
    ##  2  1  1  2  1  1  1  1  3  1

``` r
# as.factor() transforms the fields into catagories, the final step computes the number of elements in each catergory
fac <- rep(c(1,2,3),2) %>% as.factor()  # We create a simple factor and change its values below
fac <- fac %>% recode_factor(`1` = "Low", `2`= "Medium", `3`= "High", .ordered = TRUE)
```

**GOING FURTHER**:
- lists in R are very flexible structures that can embed different types of modes. - arrays are like matrices, but are allowed to have higher dimensions (&gt;2). - tibbles are the dataframes 2.0 stemming from the tidyverse.

### The apply() function

When given a rectangular dataset with numbers only, it is often useful to *apply* the same function to all rows or all columns. Normally, this would require a *for* loop. Fortunately, alternative solutions exist. For simple functions, like sums and means, dedicated functions have been created.

``` r
data_num <- select(data, -Gender)  # Take out gender because it's not a number
colMeans(data_num)                 # Mean of all columns
```

    ##    Weight    Height       Age 
    ##  68.78571 172.35714  39.14286

``` r
rowSums(data_num)                  # Sum of all rows: not very meaningful
```

    ##  [1] 274 313 330 266 311 305 281 263 260 237 264 273 280 267

For more general functions, apply is the way to go. You need to specify the dimension across which the function is computed.

``` r
apply(data_num, 2, mean) # The syntax is: apply(data, dimension, function), dimension 2 = column
```

    ##    Weight    Height       Age 
    ##  68.78571 172.35714  39.14286

``` r
apply(data_num, 1, sum)  # Dimension 1 = row
```

    ##  [1] 274 313 330 266 311 305 281 263 260 237 264 273 280 267

``` r
apply(data_num, 2, sd)   # Computing the standard deviation of each column
```

    ##   Weight   Height      Age 
    ## 14.50824 12.50604  5.98533

Note that you can use apply() on arrays with more than two dimensions.

### User-specified functions

R let's you create your own functions! Below, we create *f*(*x*)=(1 + *x*<sup>2</sup>)<sup>−1</sup> and plot it.

``` r
dens = function(x){
  return(1/(1+x^2))
}
dens(3)
```

    ## [1] 0.1

``` r
ggplot(data.frame(x = c(-4,4)), aes(x = x)) + stat_function(fun = dens)
```

![](S2_Basics_files/figure-markdown_github/fun-1.png)

### Missing data

This is a common problem in data science. Here, we only aim to locate missing data - using the is.na() function. Dealing with absent values is out of our scope.

``` r
data[3,4] <- NA         # NA is often the default for missing data points
is.na(data)             # This is the brute-force method. Ok for small datasets
```

    ##       Gender Weight Height   Age
    ##  [1,]  FALSE  FALSE  FALSE FALSE
    ##  [2,]  FALSE  FALSE  FALSE FALSE
    ##  [3,]  FALSE  FALSE  FALSE  TRUE
    ##  [4,]  FALSE  FALSE  FALSE FALSE
    ##  [5,]  FALSE  FALSE  FALSE FALSE
    ##  [6,]  FALSE  FALSE  FALSE FALSE
    ##  [7,]  FALSE  FALSE  FALSE FALSE
    ##  [8,]  FALSE  FALSE  FALSE FALSE
    ##  [9,]  FALSE  FALSE  FALSE FALSE
    ## [10,]  FALSE  FALSE  FALSE FALSE
    ## [11,]  FALSE  FALSE  FALSE FALSE
    ## [12,]  FALSE  FALSE  FALSE FALSE
    ## [13,]  FALSE  FALSE  FALSE FALSE
    ## [14,]  FALSE  FALSE  FALSE FALSE

``` r
is.na(data) %>% rowSums # Locating rows
```

    ##  [1] 0 0 1 0 0 0 0 0 0 0 0 0 0 0

``` r
is.na(data) %>% colSums # Locating columns
```

    ## Gender Weight Height    Age 
    ##      0      0      0      1

Exercises
---------

### Handling vectors & matrices

1.  Set your working directory and load the tidyverse package
2.  What are **two** simple ways to create the series {0.1, 0.2, 0.3, ..., 9.9, 10}?
3.  Same question for {2,4,6, ..., 96, 98, 100}.
4.  What is the simplest way to build a 10-by-10 matrix corresponding to the classical multiplication table? i.e. such that on the sixth column and seventh row (or vice-versa), you get 42. Replace its first value by the string "One" and see what happens.
5.  Fill in a 10-by-10 matrix with random numbers and compute its column sums

### Filtering in base R is tedious!

1.  Load the ANES database ("anes.RData")
2.  Using only base R commands (and not the filter() function), find the men in the dataset aged 87 years old.
3.  Again without reosrting to filter(), determine the party affiliation of the people older than 70 who have three or more children
4.  Likewise, compute the proportion of party affiliation of respondents younger than 25; same question for those older than 60. Any comments?

### The importance of modes

1.  Load the ANES database in Excel format ("anes.xlsx") - make sure you have the file at the right place before!
2.  Look at the summary: what happened?
3.  Change the modes into factors, and, possibly, ordered factors when need be.
    **CONCLUSION**: data preparation can be time-consuming. But it is imperative.

### Functions

The purpose of the questions below is to manipulate functions with integrated loops.
1) Create a function with one argument, n, that returns the values of j^j for j = 1...n. Test it on n = 10 and check! Anyway we could to this faster?
