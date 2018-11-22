R Basics
================

1 - GETTING STARTED
-------------------

### Working directory

The very first step after launching RStudio is to specify the working directory. You can do it directly in the *Files* pane, or via the setwd() function.

``` r
getwd()
```

    ## [1] "/Users/coqueret/Documents/IT/Cours/RStats/Git"

### Packages

Working with packages requires:  
1 - an installation, only once (basically it downloads the code on your computer)  
2 - an activation, each time you start RStudio  

``` r
# install.packages(c("openxlsx", "readxl")) # The hashtag is used for comments: the program does not read the content to the right of the hashtag.
library(openxlsx)
library(readxl)
library(tidyverse)
```

### Importing data

This is usually done directly in the user interface, or with packages like *openxlsx* or *readxl* (to import Excel files) with the function read.xlsx() or read\_excel(). The basic case:  

test\_data &lt;- read.xlsx("MyFile.xlsx")  

or  

test\_data &lt;- read\_excel("MyFile.xlsx")  


This stores your data into the test\_data variable. This assumes that the Excel file "MyFile.xlsx" exists in your working directory.

``` r
anes <- read.xlsx("anes.xlsx")
anes <- read_excel("anes.xlsx") # Same thing, but with another package
```

2 - CREATING DATA
-----------------

### Simple sequences

You can create data from scratch, using the colon operator for instance. Or the seq() function for sequences

``` r
1:10
```

    ##  [1]  1  2  3  4  5  6  7  8  9 10

``` r
3:17
```

    ##  [1]  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17

``` r
seq(1,2,0.1) # The syntax is: seq(begin, end, interval)
```

    ##  [1] 1.0 1.1 1.2 1.3 1.4 1.5 1.6 1.7 1.8 1.9 2.0

More generally, the c() function concatenates and encapsulates numbers (or text):

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

Another way to replicate data is to use row-bind and column-bind functions rbind() and cbind().

``` r
rbind(c(2,5,7),c(3,1,8)) 
```

    ##      [,1] [,2] [,3]
    ## [1,]    2    5    7
    ## [2,]    3    1    8

``` r
cbind(c(2,5,7),c(3,1,8)) 
```

    ##      [,1] [,2]
    ## [1,]    2    3
    ## [2,]    5    1
    ## [3,]    7    8

You can also fill in matrices:

``` r
m <- matrix(1:20, nrow = 4) 
m2 <- matrix(1:20, nrow = 4, byrow = T)
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
t(m) # t() is used for transposing; it works for vectors too! By default, vectors are columns
```

    ##      [,1] [,2] [,3] [,4]
    ## [1,]    1    2    3    4
    ## [2,]    5    6    7    8
    ## [3,]    9   10   11   12
    ## [4,]   13   14   15   16
    ## [5,]   17   18   19   20

``` r
m*m2 # This is term-by-term multiplication
```

    ##      [,1] [,2] [,3] [,4] [,5]
    ## [1,]    1   10   27   52   85
    ## [2,]   12   42   80  126  180
    ## [3,]   33   84  143  210  285
    ## [4,]   64  136  216  304  400

``` r
m%*%t(m2) # This is matrix multiplication
```

    ##      [,1] [,2] [,3] [,4]
    ## [1,]  175  400  625  850
    ## [2,]  190  440  690  940
    ## [3,]  205  480  755 1030
    ## [4,]  220  520  820 1120

R is great to generate random data.

``` r
runif(10) # uniform distribution
```

    ##  [1] 0.50140577 0.75051722 0.92951625 0.25878493 0.73824195 0.10229592
    ##  [7] 0.41775010 0.69210135 0.05917485 0.34108482

``` r
rnorm(10) # Gaussian distribution (parameters could be specified) 
```

    ##  [1] -0.9489200  1.3704752 -0.1174206 -0.6787922 -0.8680675 -1.5932713
    ##  [7] -1.2754354 -0.1479403  0.5422086  1.0044849

### Dataframes

Datasets often mix text and numbers. R can do that too, with data frames. Let's create one with the data.frame() function. We use the round() function which rounds up numbers.

``` r
nb_gender <- 7                                            # Number of people of each gender
Gender <- rep(c("Male"),nb_gender)                        # nb_gender men in total
Weight <-  round(70 + rnorm(nb_gender)*5)                 # in kilos
Height <-  round(176 + rnorm(nb_gender)*10)               # in cm
Age <- round(40 + rnorm(nb_gender)*6)  
data <- data.frame(Gender,Weight,Height,Age)              # data with only men
Gender <- rep(c("Female"),nb_gender)                      # nb_gender women in total
Weight <-  round(58 + rnorm(nb_gender)*5)                 # in kilos
Height <-  round(167 + rnorm(nb_gender)*10)               # in cm
Age <- round(40 + rnorm(nb_gender)*6)  
data <- rbind(data, data.frame(Gender,Weight,Height,Age)) # grouping women with men
data
```

    ##    Gender Weight Height Age
    ## 1    Male     74    183  45
    ## 2    Male     75    169  52
    ## 3    Male     69    183  31
    ## 4    Male     70    167  33
    ## 5    Male     69    173  40
    ## 6    Male     72    190  31
    ## 7    Male     70    173  46
    ## 8  Female     59    177  34
    ## 9  Female     56    173  44
    ## 10 Female     61    177  50
    ## 11 Female     50    152  34
    ## 12 Female     58    156  36
    ## 13 Female     59    172  45
    ## 14 Female     58    145  33

You can use rownames() or colnames() to get or set the names of rows or columns: colnames(data).

### Dimensions

You can obtain the dimension of a matrix or data frame with the dim() function: dim(data). (Nb rows and nb columns). Each dimension can be obtained separately with nrow() and ncol() For vectors, the number of elements can be found with the length() function.

``` r
dim(data) 
```

    ## [1] 14  4

``` r
nrow(data) # Number of rows
```

    ## [1] 14

``` r
ncol(data) # Number of columns
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
data$Age
```

    ##  [1] 45 52 31 33 40 31 46 34 44 50 34 36 45 33

Another way to proceed is to omit to specify the row numbers: since Height is the third column of data, then the result is the same with data\[,3\]. This give you all of the third column. Likewise, data\[3,\] will return all of the third row.

``` r
data[,3]
```

    ##  [1] 183 169 183 167 173 190 173 177 173 177 152 156 172 145

``` r
data[3,]
```

    ##   Gender Weight Height Age
    ## 3   Male     69    183  31

You can extract data with boolean vectors! For instance, if we want to select the people who are older than 42 years old: simple!

``` r
data$Age>42
```

    ##  [1]  TRUE  TRUE FALSE FALSE FALSE FALSE  TRUE FALSE  TRUE  TRUE FALSE
    ## [12] FALSE  TRUE FALSE

will provide the corresponding indices. To extract the data, you just need to select the right rows and all columns:

``` r
data[data$Age > 42, ] # Mind the last column!
```

    ##    Gender Weight Height Age
    ## 1    Male     74    183  45
    ## 2    Male     75    169  52
    ## 7    Male     70    173  46
    ## 9  Female     56    173  44
    ## 10 Female     61    177  50
    ## 13 Female     59    172  45

``` r
data[data$Age > 42 & data$Weight >70, ] # the & operator allows to add sorting criteria
```

    ##   Gender Weight Height Age
    ## 1   Male     74    183  45
    ## 2   Male     75    169  52

Only the TRUE rows are kept. As we will see, the filter() function of the tidyverse does just that.

### Writing / Replacing values

Writing on data frames, vectors, or matrices can be done with the arrow operator:

``` r
data[3,2] <- 99
data[c(7,9),3] <- 166 # Replace 2 cells at a time! Seventh and ninth row on the third column.
data[c(7,9),3] <- c(199,166) # Same, but with 2 different values.
data # CHECK where the new values are!
```

    ##    Gender Weight Height Age
    ## 1    Male     74    183  45
    ## 2    Male     75    169  52
    ## 3    Male     99    183  31
    ## 4    Male     70    167  33
    ## 5    Male     69    173  40
    ## 6    Male     72    190  31
    ## 7    Male     70    199  46
    ## 8  Female     59    177  34
    ## 9  Female     56    166  44
    ## 10 Female     61    177  50
    ## 11 Female     50    152  34
    ## 12 Female     58    156  36
    ## 13 Female     59    172  45
    ## 14 Female     58    145  33

### Seeing data

Unlike in Excel, the data is not directly shown in R. You have to ask for it! To see the content of a variable, you have to type its name and press ENTER. Sometimes, the content of the variable is huge and cannot properly be displayed.
The head() function shows the first 6 lines and the tail() function shows the last 6 lines.

``` r
head(data)
```

    ##   Gender Weight Height Age
    ## 1   Male     74    183  45
    ## 2   Male     75    169  52
    ## 3   Male     99    183  31
    ## 4   Male     70    167  33
    ## 5   Male     69    173  40
    ## 6   Male     72    190  31

``` r
tail(data)
```

    ##    Gender Weight Height Age
    ## 9  Female     56    166  44
    ## 10 Female     61    177  50
    ## 11 Female     50    152  34
    ## 12 Female     58    156  36
    ## 13 Female     59    172  45
    ## 14 Female     58    145  33

If you want to see the different possible values of a factor (categorical variables), you can use the levels() function.

``` r
levels(diamonds$clarity)
```

    ## [1] "I1"   "SI2"  "SI1"  "VS2"  "VS1"  "VVS2" "VVS1" "IF"

Though honestly, you would get the information using the summary() function as well.

### Simple plots

The base function is plot() and once a graph is create, lines can be added, via lines(). Many options are available but beyond the scope of this document. Note that these functions are seldom used by the R community: ggplot() is preferred.

``` r
plot(data$Weight, ylim = c(10,100)) # ylim gives the y-axis limits
lines(data$Age, col = "red")
```

![](https://github.com/shokru/rstats/blob/master/Figures/plot-1.png)

4 - LOOPS: 1st step towards programming
---------------------------------------

There are usually several types of loops, but we will focus on the **for** loop. Its structure is simple: the idea is to repeat a task a finite number of times. This allows to automate the changes in a variable. For instance, the Fibonacci sequence:

``` r
nb <- 20  # Number of desired numbers
Fib <- 1
Fib[2] <- 1
for(k in 3:nb){
   Fib[k] <- Fib[k-1] + Fib[k-2]
}
Fib # Show the sequence
```

    ##  [1]    1    1    2    3    5    8   13   21   34   55   89  144  233  377
    ## [15]  610  987 1597 2584 4181 6765

**GOING FURTHER**: loops can take time. To speed things up, have a look at the map() family of functions.

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

    ## [1] -2.155806

``` r
max(v)                              # Maximum
```

    ## [1] 1.629156

``` r
v                                   # A look at the whole vector
```

    ##  [1]  0.96484121  1.13981624  0.05451418 -0.30256997 -0.06545961
    ##  [6]  1.25121121 -0.76047872  0.77354277 -1.48912772 -2.15580596
    ## [11] -0.21940015  0.76470194  1.62915583  0.32854717  1.29587053
    ## [16] -0.19802395 -1.36218192  0.21746843

``` r
which(v == max(v))                  # which() returns the index of the items that are TRUE inside a Boolean expression
```

    ## [1] 13

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
data$Age %>% as.factor() %>% summary()  # as.factor() transforms the fields into catagories, the final step computes the number of elements in each catergory
```

    ## 31 33 34 36 40 44 45 46 50 52 
    ##  2  2  2  1  1  1  2  1  1  1

``` r
fac <- rep(c(1,2,3),2) %>% as.factor()  # We create a simple factor and change its values below
fac <- fac %>% recode_factor(`1` = "Low", `2`= "Medium", `3`= "High", .ordered = TRUE)
```

**GOING FURTHER**:
- lists in R are very flexible structures that can embed different types of modes. - arrays are like matrices, but are allowed to have higher dimensions (&gt;2). - tibbles are the dataframes 2.0 stemming from the tidyverse.

### The apply() function

When given a rectangular dataset with numbers only, it is often useful to *apply* the same function to all rows or all columns. Normally, this would require a *for* loop. Fortunately, alternative solutions exist. For simple functions, like sums and means, dedicated functions have been created.

``` r
data_num <- select(data, -Gender)  # Take out gender because it's not a number
colMeans(data_num)
```

    ##    Weight    Height       Age 
    ##  66.42857 172.07143  39.57143

``` r
rowSums(data_num)
```

    ##  [1] 302 296 313 270 282 293 315 270 266 288 236 250 276 236

For more general functions, apply is the way to go. You need to specify the dimension across which the function is computed.

``` r
apply(data_num, 2, mean) # The syntax is: apply(data, dimension, function), dimension 2 = column
```

    ##    Weight    Height       Age 
    ##  66.42857 172.07143  39.57143

``` r
apply(data_num, 1, sum)  # Dimension 1 = row
```

    ##  [1] 302 296 313 270 282 293 315 270 266 288 236 250 276 236

``` r
apply(data_num, 2, sd)   # Computing the standard deviation of each column
```

    ##    Weight    Height       Age 
    ## 12.138396 14.678421  7.292929

Note that you can use apply() on arrays with more than two dimensions.

### User-specified functions

R let's you create your own functions!

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

![](https://github.com/shokru/rstats/blob/master/Figures/fun-1.png)

### Missing data

This is a common problem in data science. Here, we only aim to locate missing data - using the is.na() function. Dealing with absent values is out of our scope.

``` r
data[3,4] <- NA # NA is often the default for missing data points
is.na(data) # This is the brute-force method. Ok for small datasets
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
