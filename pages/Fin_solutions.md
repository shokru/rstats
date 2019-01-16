Financial Data Science Exercises
================

-   [DATA](#data)
-   [PART I: GRAPHS](#part-i-graphs)
-   [PART II: DATA HANDLING](#part-ii-data-handling)
    -   [Simple filters](#simple-filters)
    -   [Modifying data: normalised prices](#modifying-data-normalised-prices)
    -   [Modifying data: computing returns (1)](#modifying-data-computing-returns-1)
    -   [Modifying data: computing returns (2)](#modifying-data-computing-returns-2)
-   [PART III: PIVOT TABLES](#part-iii-pivot-tables)
-   [PART IV: FACTOR ANALYSIS](#part-iv-factor-analysis)
-   [PART V: LOOPS](#part-v-loops)

NOTE: the tasks are sequential. Consequently, PART II must be done after PART I, etc.

DATA
----

The data consists of monhtly financial information pertaining to 30 large US firms. They are characterised by their ticker:

| A - F                 | G - M                   | O - Z                 |
|-----------------------|-------------------------|-----------------------|
| AAPL (Apple)          | GE (General Electric)   | ORCL (Oracle)         |
| BA (Boeing)           | HD (Home Depot)         | PFE (Pfizer)          |
| BAC (Bank of America) | IBM                     | PG (Procter & Gamble) |
| C (Citigroup)         | INTC (Intel)            | T (AT&T)              |
| CSCO (Cisco)          | JNJ (Johnson & Johnson) | UNH (United Health)   |
| CVS (CVS Health)      | JPM (JP Morgan)         | UPS                   |
| CVX (Chevron)         | K (Kellogg)             | VZ (Verizon)          |
| D (Dominion Energy)   | MCK (McKesson)          | WFC (Wells Fargo)     |
| DIS (Disney)          | MRK (Merck)             | WMT (Walmart)         |
| F (Ford)              | MSFT (Microsoft)        | XOM (Exxon)           |

There are 3 attributes: closing price (Close), market capitalisation in M$ (Mkt\_Cap) and price-to-book ratio (P2B). Finally, the time range is 2000-2018.

------------------------------------------------------------------------

PART I: GRAPHS
--------------

1.  Set the working directory, load the data ('data.RData') and the tidyverse package.
2.  First, take a look at the data using the head() function and then the summary() function.
3.  Plot (with a line) the price of the Microsoft (MSFT) share through time.
4.  Plot (with a line) the price of Apple (AAPL), IBM and Microsoft with one color for each company.
    Obviously, there is a scale problem: we'll try to solve it in the next section.

``` r
# Load the data in the bottom right pane
library(tidyverse)
load("data.RData")
data <- data %>% arrange(Tick, Date) # Arranges/orders the data
head(data)                           # Shows the first 6 lines
```

    ##   Tick       Date  Close  Mkt_Cap    P2B
    ## 1 AAPL 2000-01-03 3.9978 17999.44 4.2324
    ## 2 AAPL 2000-02-01 3.5804 16162.71 3.7905
    ## 3 AAPL 2000-03-01 4.6540 21009.50 4.9272
    ## 4 AAPL 2000-04-03 4.7612 21493.17 5.3351
    ## 5 AAPL 2000-05-01 4.4397 20042.16 4.9749
    ## 6 AAPL 2000-06-01 3.1830 14504.53 3.5668

``` r
summary(data)                        # Descriptive statistics
```

    ##       Tick           Date                Close            Mkt_Cap      
    ##  AAPL   : 221   Min.   :2000-01-03   Min.   :  1.004   Min.   :  4491  
    ##  BA     : 221   1st Qu.:2004-08-02   1st Qu.: 28.050   1st Qu.: 63498  
    ##  BAC    : 221   Median :2009-03-02   Median : 42.535   Median :128864  
    ##  C      : 221   Mean   :2009-03-02   Mean   : 57.519   Mean   :145819  
    ##  CSCO   : 221   3rd Qu.:2013-10-01   3rd Qu.: 66.045   3rd Qu.:198487  
    ##  CVS    : 221   Max.   :2018-05-01   Max.   :557.000   Max.   :887952  
    ##  (Other):5304                                                          
    ##       P2B           
    ##  Min.   :   0.0922  
    ##  1st Qu.:   2.0783  
    ##  Median :   3.1054  
    ##  Mean   :   6.6966  
    ##  3rd Qu.:   5.2977  
    ##  Max.   :1259.1554  
    ## 

``` r
data %>% filter(Tick == "MSFT") %>% ggplot(aes(x = Date, y = Close)) + geom_line() # First plot
```

![](Fin_solutions_files/figure-markdown_github/UP%20TO%20YOU!%201-1.png)

``` r
data %>% filter(Tick == "MSFT" | Tick == "IBM" | Tick == "AAPL") %>% ggplot(aes(x = Date, y = Close)) + geom_line(aes(color = Tick))
```

![](Fin_solutions_files/figure-markdown_github/UP%20TO%20YOU!%201-2.png)

``` r
# data %>% filter(Tick %in% c("MSFT", "IBM", "AAPL")) %>% ggplot(aes(x = Date, y = Close)) + geom_line(aes(color = Tick)) # Alternative solution
```

------------------------------------------------------------------------

PART II: DATA HANDLING
----------------------

### Simple filters

1.  Filter the data to keep only MSFT figures.
2.  Filter the data to keep only year 2018.
3.  Do both at the same time

``` r
# filter(data, Tick == "MSFT")      # Point 1)
# filter(data, Date > "2017-12-31") # Point 2)
filter(data, Tick == "MSFT" & Date > "2017-12-31")
```

    ##   Tick       Date Close  Mkt_Cap    P2B
    ## 1 MSFT 2018-01-01 85.54 659906.0 8.4110
    ## 2 MSFT 2018-02-01 94.26 725782.5 9.2684
    ## 3 MSFT 2018-03-01 92.85 714925.8 9.1298
    ## 4 MSFT 2018-04-02 88.52 681585.7 8.7040
    ## 5 MSFT 2018-05-01 95.00 729903.8 9.2196

### Modifying data: normalised prices

*The aim of this subsection is to add a new column to the data dataframe. This column would be equal to prices normalised to one on the first date. This makes graphical comparison across firms easier. This could be done with a loop (see comments below). We proceed otherwise for pedagogical purposes: use the tidyverse!*

1.  Create a matrix with prices only (call it prices): first, select the *Tick*, *Date* and *Close* columns. Second, use the spread() function. Finally, remove the Date column with select().
2.  Normalise each series by its first value; this way they all start by one. Hint: you can create a function and apply it on the matrix using apply(). Call this new matrix tmp.
3.  Create a date vector with all dates.
4.  Concatenate (using cbind()) the date vector back on the tmp matrix.
5.  Using the gather() function, which is the inverse of spread(), put the data back into its original form! Hint: call the new column NClose, for normalised Close. Also, arrange it by Tick and Date.
6.  Finally, create a new NClose column in the full dataset and plot (line) the price of Apple, IBM and Microsoft (MSFT) with one color for each company.

``` r
prices <- data %>% select(Tick, Date, Close) %>% spread(key = Tick, value = Close) %>% select(-Date) # Point 1)
normalize <- function(v){return(v/v[1])} # Normalises a vector by its first value
tmp <- prices %>% apply(2,normalize)     # That's point 2) of the exercises
date <- filter(data, Tick == "AAPL") %>% select(Date)                         # Point 3)
tmp <- cbind(date,tmp)                                                        # Point 4)
tmp <- gather(tmp, key = Tick, value = NClose, -Date) %>% arrange(Tick, Date) # Point 5)
data$NClose <- tmp$NClose                                                     # Point 6)
data %>% filter(Tick == "MSFT" | Tick == "IBM" | Tick == "AAPL") %>% ggplot(aes(x = Date, y = NClose)) + geom_line(aes(color = Tick))
```

![](Fin_solutions_files/figure-markdown_github/UP%20TO%20YOU!%203-1.png)

``` r
# Alternative solution below, using a loop:
prices2 <- data %>% select(Tick, Date, Close) %>% spread(key = Tick, value = Close) 
for(i in 2:31){ # Don't touch the first column! 
    prices2[,i] <- prices2[,i] / prices2[1,i]
}
tmp <- gather(prices2, key = Tick, value = NClose, -Date) %>% arrange(Tick, Date) # Point 5)
```

### Modifying data: computing returns (1)

1.  Going further: using the same process and the price matrix, add an important column to the dataset, namely the monthly returns of each stock (hint: combine the apply() and the lag() functions). In the process, keep the matrix of returns in a tmp variable.
2.  Using tmp, compute the covariance matrix of the first 14 stocks (in alphabetical order). Because of the NAs, you will need the option (argument) use = "complete.obs" in the cor() function.

``` r
tmp <- prices / apply(prices, 2, lag) - 1 # Computes the returns
tmp2 <- cbind(date,tmp)                   # Binds the dates
tmp2 <- gather(tmp2, key = Tick, value = Return, -Date) %>% arrange(Tick, Date)
data$Return <- tmp2$Return

# The short version below: 
data$Return <- data$Close / lag(data$Close) - 1 # Compute returns at once, but the first return of all stocks is wrong!
data[data$Date == "2000-01-03",]$Return <- NA   # Remove the false returns!


# Below, we show how to do this using a loop, but this is clearly not optimal!
# data2 <- c() 
# ticks <- levels(data$Tick)
# for(i in 1:length(ticks)){
#     tmp <- data %>% filter(Tick == ticks[i]) %>% mutate(Return = Close/lag(Close) - 1)
#     data2 <- rbind(data2, tmp) 
# }    

C <- cor(tmp[,1:14], use = "complete.obs") %>% round(2) %>% data.frame()
print(C)
```

    ##      AAPL   BA  BAC    C CSCO  CVS  CVX    D  DIS    F   GE   HD  IBM INTC
    ## AAPL 1.00 0.18 0.14 0.20 0.38 0.05 0.25 0.01 0.33 0.14 0.29 0.24 0.46 0.56
    ## BA   0.18 1.00 0.36 0.48 0.35 0.33 0.44 0.31 0.54 0.34 0.48 0.35 0.20 0.31
    ## BAC  0.14 0.36 1.00 0.84 0.28 0.24 0.33 0.16 0.46 0.39 0.57 0.45 0.28 0.29
    ## C    0.20 0.48 0.84 1.00 0.38 0.25 0.42 0.23 0.53 0.36 0.63 0.46 0.35 0.38
    ## CSCO 0.38 0.35 0.28 0.38 1.00 0.12 0.25 0.03 0.47 0.35 0.43 0.39 0.58 0.64
    ## CVS  0.05 0.33 0.24 0.25 0.12 1.00 0.34 0.28 0.33 0.27 0.34 0.38 0.17 0.15
    ## CVX  0.25 0.44 0.33 0.42 0.25 0.34 1.00 0.43 0.43 0.25 0.45 0.34 0.32 0.26
    ## D    0.01 0.31 0.16 0.23 0.03 0.28 0.43 1.00 0.19 0.11 0.24 0.12 0.07 0.07
    ## DIS  0.33 0.54 0.46 0.53 0.47 0.33 0.43 0.19 1.00 0.40 0.56 0.51 0.44 0.52
    ## F    0.14 0.34 0.39 0.36 0.35 0.27 0.25 0.11 0.40 1.00 0.39 0.35 0.22 0.25
    ## GE   0.29 0.48 0.57 0.63 0.43 0.34 0.45 0.24 0.56 0.39 1.00 0.45 0.48 0.37
    ## HD   0.24 0.35 0.45 0.46 0.39 0.38 0.34 0.12 0.51 0.35 0.45 1.00 0.44 0.37
    ## IBM  0.46 0.20 0.28 0.35 0.58 0.17 0.32 0.07 0.44 0.22 0.48 0.44 1.00 0.56
    ## INTC 0.56 0.31 0.29 0.38 0.64 0.15 0.26 0.07 0.52 0.25 0.37 0.37 0.56 1.00

``` r
if(!require(ggcorrplot)){install.packages("ggcorrplot")} # Below, we visualise the correlation matrix using the ggcorrplot package
```

    ## Loading required package: ggcorrplot

``` r
library(ggcorrplot)
ggcorrplot(C, hc.order = TRUE, type = "upper", outline.col = "white") +
    theme(text = element_text(size=8), axis.text.x = element_text(angle=90, hjust=1))
```

![](Fin_solutions_files/figure-markdown_github/UP%20TO%20YOU!%204-1.png)

Visually, the largest correlation is between Citigroup and Bank of America, which highlights the importance of the sector for asset co-movements.

### Modifying data: computing returns (2)

There exists a much simpler way to proceed! The power of group\_by()...

``` r
data %>% group_by(Tick) %>% mutate(return = Close/lag(Close) - 1)
```

    ## # A tibble: 6,630 x 8
    ## # Groups:   Tick [30]
    ##    Tick  Date       Close Mkt_Cap   P2B NClose   Return   return
    ##    <fct> <date>     <dbl>   <dbl> <dbl>  <dbl>    <dbl>    <dbl>
    ##  1 AAPL  2000-01-03  4.00  17999.  4.23  1      NA       NA     
    ##  2 AAPL  2000-02-01  3.58  16163.  3.79  0.896  -0.104   -0.104 
    ##  3 AAPL  2000-03-01  4.65  21010.  4.93  1.16    0.300    0.300 
    ##  4 AAPL  2000-04-03  4.76  21493.  5.34  1.19    0.0230   0.0230
    ##  5 AAPL  2000-05-01  4.44  20042.  4.97  1.11   -0.0675  -0.0675
    ##  6 AAPL  2000-06-01  3.18  14505.  3.57  0.796  -0.283   -0.283 
    ##  7 AAPL  2000-07-03  3.81  17353.  4.30  0.953   0.196    0.196 
    ##  8 AAPL  2000-08-01  3.52  16025.  3.98  0.881  -0.0750  -0.0750
    ##  9 AAPL  2000-09-01  4.53  20615.  5.12  1.13    0.286    0.286 
    ## 10 AAPL  2000-10-02  1.73   7881.  2.02  0.433  -0.618   -0.618 
    ## # ... with 6,620 more rows

Thanks to the grouping, the formula inside mutate() is applied sequentially stock-by-stock!

------------------------------------------------------------------------

PART III: PIVOT TABLES
----------------------

1.  First, we are (again) going to augment the database by adding a Year column. The code is:

``` r
if(!require(lubridate)){install.packages("lubridate")} 
# This is a package that handles dates effectively. SKIP this step if already installed.
library(lubridate)
data <- data %>% mutate(Year = year(Date)) # Adding a Year column for yearly statistics.
```

1.  Group the data by company and by year and create a new variable *Avg\_Cap* which computes the average market capitalisation.
2.  Plot (line) the corresponding results: x-axis is Year, y-axis is Avg\_Cap with one color for each firm. =&gt; Hard to read. Beyond 10 firms, the graph becomes less insightful. Try to do the same with a barplot.
3.  Pick one year (e.g., 2005) and show the average capitalization of firms in decreasing order.
4.  From data, create a pivot table (call it pt) that computes the average capitalization, average P2B and average return for each firm. Hint: for returns, there are some NAs, so use the argument na.rm = T in the mean().

``` r
data %>% group_by(Tick, Year) %>% 
    summarise(Avg_Cap = mean(Mkt_Cap)) %>% # This is point 1)
    ggplot(aes(x=Year, y =Avg_Cap)) + geom_line(aes(color = Tick))                  # Point 2)
```

![](Fin_solutions_files/figure-markdown_github/UP%20TO%20YOU!%205-1.png)

``` r
data %>% group_by(Tick, Year) %>% 
    summarise(Avg_Cap = mean(Mkt_Cap)) %>% 
    ggplot(aes(x=Year, y =Avg_Cap)) + geom_bar(aes(fill = Tick), stat = "identity") # Point 2)
```

![](Fin_solutions_files/figure-markdown_github/UP%20TO%20YOU!%205-2.png)

``` r
data %>% group_by(Tick, Year) %>% 
    summarise(Avg_Cap = mean(Mkt_Cap)) %>% 
    filter(Year == 2005) %>% 
    arrange(desc(Avg_Cap))                                  # Point 3)
```

    ## # A tibble: 30 x 3
    ## # Groups:   Tick [30]
    ##    Tick   Year Avg_Cap
    ##    <fct> <dbl>   <dbl>
    ##  1 GE     2005 371958.
    ##  2 XOM    2005 369876.
    ##  3 MSFT   2005 279008.
    ##  4 C      2005 240445.
    ##  5 WMT    2005 204644.
    ##  6 JNJ    2005 192385.
    ##  7 PFE    2005 189760.
    ##  8 BAC    2005 181023.
    ##  9 INTC   2005 152857.
    ## 10 PG     2005 137125.
    ## # ... with 20 more rows

``` r
pt <- data %>% group_by(Tick) %>% 
    summarise(Avg_Cap = mean(Mkt_Cap), Avg_P2B = mean(P2B), Avg_Ret = mean(Return, na.rm = T)) # Point 4: beware of NA problems!
```

------------------------------------------------------------------------

PART IV: FACTOR ANALYSIS
------------------------

Research in financial economics has shown that firms' characteristics are likely to drive profitability. We try to investigate this idea below.
1) Using data, plot (scatter plot, i.e., with points) y = returns versus x = Mkt\_Cap. 2) Same exercise but with pt (pivot table from PART III), x = Avg\_Cap and y = Avg\_Ret. 3) Same exercise as 2), but after removing AAPL from the sample 4) From pt, plot x = Avg\_P2B and y = Avg\_ret. Comment?

``` r
data %>% ggplot(aes(x = Mkt_Cap, y = Return)) + geom_point() + stat_smooth()  # Point 1)
```

![](Fin_solutions_files/figure-markdown_github/UP%20TO%20YOU!%206-1.png)

``` r
pt %>% ggplot(aes(x = Avg_Cap, y = Avg_Ret)) + geom_point() + stat_smooth() + # Point 2)
    geom_text(aes(label = Tick)) # You can add this line to see the firm names.
```

![](Fin_solutions_files/figure-markdown_github/UP%20TO%20YOU!%206-2.png)

``` r
pt %>% ggplot(aes(x = Avg_Cap, y = Avg_Ret)) + geom_point() + stat_smooth(method = "lm") # lm is for 'linear model', hence the straight line.
```

![](Fin_solutions_files/figure-markdown_github/UP%20TO%20YOU!%206-3.png)

``` r
pt %>% filter(Tick != "AAPL") %>% ggplot(aes(x = Avg_Cap, y = Avg_Ret)) + # Point 3): we remove AAPL
    geom_point() + stat_smooth(method = "lm")
```

![](Fin_solutions_files/figure-markdown_github/UP%20TO%20YOU!%206-4.png)

``` r
pt %>% ggplot(aes(x = Avg_P2B, y = Avg_Ret)) + geom_point() + stat_smooth(method = "lm") +
    geom_text(aes(label = Tick))
```

![](Fin_solutions_files/figure-markdown_github/UP%20TO%20YOU!%206-5.png)

In the last graph, there is an outlier for the average P2B: error in the data? Further investigation would show that in 2017, the book value of Boeing was very small and hence its P2B ratio was extremely high.
In the above graphs, we see the mild negative relationship between firm size and average return. Though our sample is much too small and our study not rigourous, this resembles the so-called size effect according to which small firms are more profitable than large firms (though not in financial *bad times*).

------------------------------------------------------------------------

PART V: LOOPS
-------------

Create a loop over all dates that computes the aggregate market capitalisation of all 30 firms at each date (call it Agg\_Cap). Plot the corresponing series. (Note: this is a very inefficient way to do this.)

``` r
dates <- data$Date %>% 
    as.factor() %>% 
    levels() %>%
    as.Date()       # Creating a vector of dates.
dates <- data %>% 
    filter(Tick == "AAPL") %>% 
    select(Date) %>% 
    as.matrix() %>% # This is to get rid of the dataframe structure to avoid column numbers 
    as.Date()       # Similar though not exactly the same result

Agg_Cap <- 0
for(t in 1:length(dates)){
    Agg_Cap[t] <- data %>%
        filter(Date == dates[t]) %>%
        select(Mkt_Cap) %>%
        sum()
}
data.frame(dates, Agg_Cap) %>% ggplot(aes(x = dates, y = Agg_Cap)) + geom_line()
```

![](Fin_solutions_files/figure-markdown_github/UP%20TO%20YOU!%207-1.png)
