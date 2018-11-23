Time-series analysis
================

This is the companion notebook to the course on Time-series analysis. The dataset consists of 3 series of indicators characterising the French economy.

``` r
library(tidyverse)
# Then change your working directory
if(!require(lubridate)){install.packages("lubridate")}
library(lubridate)
load("economics.RData")
summary(economics)
```

    ##     Quarter            GDP          Unemployment        CPI        
    ##  Min.   :199001   Min.   :261484   Min.   : 6.80   Min.   : 66.56  
    ##  1st Qu.:199701   1st Qu.:318372   1st Qu.: 8.10   1st Qu.: 77.47  
    ##  Median :200401   Median :421974   Median : 8.90   Median : 85.59  
    ##  Mean   :200365   Mean   :419099   Mean   : 8.91   Mean   : 85.92  
    ##  3rd Qu.:201101   3rd Qu.:511961   3rd Qu.: 9.80   3rd Qu.: 95.67  
    ##  Max.   :201801   Max.   :582503   Max.   :10.40   Max.   :101.72

``` r
economics <- mutate(economics, Year = as.numeric(substr(Quarter,1,4)), Quarter = as.numeric(substr(Quarter,5,6)))
economics <- mutate(economics, Date = make_date(year = Year, month = 3*Quarter, day = 1))
```

First analysis
--------------

The first thing to do is often to have a look at the data:

``` r
eco <- economics %>% select(-Year, -Quarter) %>% gather(key = Variable, value = Value, -Date) 
ggplot(eco, aes(x = Date, y = Value)) + geom_line() + facet_grid(Variable ~ .,  scales = 'free')
```

![](S6_TS_files/figure-markdown_github/plots_1-1.png)

Obviously, these series are not stationary (at least the first two), if only because of their trend which implies that their mean increases with time (the range of the distribution is clearly not constant). One possible solution is to study variations:

``` r
economics$GDP_var <- economics$GDP - lag(economics$GDP)    # Variation in GDP
ggplot(economics, aes(x = Date, y = GDP_var)) + geom_point()
```

![](S6_TS_files/figure-markdown_github/variations-1.png) Clearly, this is better. Indeed, it seems that the variations in the French GDP display a somewhat coherent pattern. Nevertheless, the classical transform used to \`stationarise' series consists in taking returns, i.e., relative variations:

``` r
economics$GDP_ret <- economics$GDP / lag(economics$GDP) - 1 # Return in GDP
ggplot(economics, aes(x = Date, y = GDP_ret)) + geom_bar(stat = "identity") 
```

![](S6_TS_files/figure-markdown_github/returns-1.png)

The original process can be reconstructed from returns:
$$X\_t=X\_0\\prod\_{s=1}^t(1+r\_s).$$

Below, we show an example that starts from random returns and the recomposes the integrated process.

``` r
r <- runif(12, min = -0.5, max = 0.5)
x <- cumprod(1+r)
time <- rep(1:12,2)
value <- c(r,x)
type <- c(rep("r",12), rep("x",12))
data.frame(time,value,type) %>% ggplot(aes(x = time, y = value)) + geom_line() +facet_grid(type~., scales = "free")
```

![](S6_TS_files/figure-markdown_github/reconstruction-1.png) When returns are negative (resp. positive), the process x decreases (resp. increases).

Classical processes
-------------------

### The random walk

The most typical random process. Increasing or decreasing by one with probability 1/2.

``` r
x <- 0 # A first example, step-by-step. The process starts at zero.
nb_steps <- 15
for(n in 2:nb_steps){
  x[n] <- x[n-1] + 2 * rbinom(1, 1, 0.5) - 1
}
data.frame(date = 1:nb_steps, x) %>% ggplot(aes(x = date, y = x)) + geom_line()
```

![](S6_TS_files/figure-markdown_github/RW_1-1.png)

``` r
x <- 0 # A quicker, neater way to proceed.
nb_steps <- 80
x <- (2*rbinom(nb_steps, 1, 0.5) - 1) %>% cumsum()
df <- data.frame(time = 1:nb_steps, x)
ggplot(df, aes(x = time, y = x)) + geom_line()
```

![](S6_TS_files/figure-markdown_github/RW_2-1.png)

### Autoregressive processes

Autoregressive processes add memory to the trajectory: increments are no longer independent. Below, we build a trajectory of one such process.

``` r
a <- 1
b <- 0.5
x <- 1 # Initial value.
nb_steps <- 20
for(n in 2:nb_steps){
  x[n] <- a + b * x[n-1] + rnorm(1)
}
data.frame(date = 1:nb_steps, x) %>% ggplot(aes(x = date, y = x)) + geom_line()
```

![](S6_TS_files/figure-markdown_github/AR(1)-1.png)

Let's use a dedicated function.

``` r
Nb_points <- 100
AR <- arima.sim(list(ar = c(0.9)), Nb_points)
Time <- 1:Nb_points
AR %>% cbind(Time) %>% data.frame() %>% ggplot(aes(x = Time, y = AR)) + geom_line()
```

    ## Don't know how to automatically pick scale for object of type ts. Defaulting to continuous.

![](S6_TS_files/figure-markdown_github/AR(1)%20v2-1.png)

### Autocorrelogram

The autocorrelogram computes the correlation between lagged values of the series.

``` r
acf(economics$GDP_ret, na.action = na.pass)
```

![](S6_TS_files/figure-markdown_github/acf-1.png)

``` r
acf(AR)
```

![](S6_TS_files/figure-markdown_github/acf-2.png) We see autocorrelation in the French economic growth. This probably comes from so-called economic cycles (periods of growth followed by recessions).

### Estimation

For a given series, one may want to estimate the corresponding AR parameters (if that seems relevant).

``` r
ar(economics$GDP_ret, na.action = na.pass, order.max = 1)   # Autoregressive coefficient
```

    ## 
    ## Call:
    ## ar(x = economics$GDP_ret, order.max = 1, na.action = na.pass)
    ## 
    ## Coefficients:
    ##      1  
    ## 0.5829  
    ## 
    ## Order selected 1  sigma^2 estimated as  1.896e-05

``` r
series <- economics$GDP_ret[2: nrow(economics)]             # Taking out the NA term
ar(series, order.max = 1, method = "ols")                   # Classical least-squares method
```

    ## 
    ## Call:
    ## ar(x = series, order.max = 1, method = "ols")
    ## 
    ## Coefficients:
    ##      1  
    ## 0.5833  
    ## 
    ## Intercept: -4.826e-05 (0.0004093) 
    ## 
    ## Order selected 1  sigma^2 estimated as  1.86e-05

``` r
ar(series, order.max = 1, method = "mle")                   # Max likelihood
```

    ## 
    ## Call:
    ## ar(x = series, order.max = 1, method = "mle")
    ## 
    ## Coefficients:
    ##      1  
    ## 0.5819  
    ## 
    ## Order selected 1  sigma^2 estimated as  1.856e-05

``` r
rho <- cor(economics$GDP_ret, lag(economics$GDP_ret), use = "complete")
rho                                                         # Moment-based value
```

    ## [1] 0.5850411

Once the model is estimated, we can forecast the future value.

### Forecast

``` r
ar_est <- ar(series, order.max = 1, method = "mle") 
predict(ar_est)
```

    ## $pred
    ## Time Series:
    ## Start = 113 
    ## End = 113 
    ## Frequency = 1 
    ## [1] 0.006385801
    ## 
    ## $se
    ## Time Series:
    ## Start = 113 
    ## End = 113 
    ## Frequency = 1 
    ## [1] 0.004307705

``` r
0.583*series[112] + mean(series)*(1-0.583) # Manual check!
```

    ## [1] 0.006384121

We compare it to the conditional mean: pretty close.

Economic Indicators: Multivariate analysis
------------------------------------------

### First steps

Often, variables inside a sample are expected to be related.

``` r
economics$Unemp_ret <- economics$Unemployment / lag(economics$Unemployment) - 1
cor(economics$GDP, economics$Unemployment)
```

    ## [1] 0.02307862

``` r
cor(economics$GDP_ret, economics$Unemp_ret, use = "complete")
```

    ## [1] -0.5047352

``` r
cor(lag(economics$GDP_ret), economics$Unemp_ret, use = "complete")
```

    ## [1] -0.4970082

``` r
data <- economics %>% select(Date, GDP_ret, Unemp_ret) %>% gather(key = Indicator, value = Value, -Date)
data %>% ggplot(aes(x = Date, y = Value)) + 
  geom_bar(stat = "identity") + facet_grid(Indicator~., scales = "free") +
  theme(axis.text.x = element_text(angle = 90))
```

    ## Warning: Removed 2 rows containing missing values (position_stack).

![](S6_TS_files/figure-markdown_github/multi-1.png)

``` r
economics %>% ggplot(aes(x = GDP_ret, y = Unemp_ret)) + geom_point() + geom_smooth(method = "lm")
```

    ## Warning: Removed 1 rows containing non-finite values (stat_smooth).

    ## Warning: Removed 1 rows containing missing values (geom_point).

![](S6_TS_files/figure-markdown_github/multi-2.png) Raw GDP and unemployment appear unrelated. But variations in both variables are negatively correlated: when GDP increases, unemployment decreases. The last correlation show that even the previous value of change in growth is correlated to current change in unemployment.

### Estimation

Below, we estimate a vector autoregression model on GDP and unemployment growths.

``` r
if(!require(vars)){install.packages("vars")}
library(vars)
library(tidyverse)
# FIRST, we prepare the data
var_data <- economics %>% dplyr::select(GDP_ret, Unemp_ret) # MASS package overrides select() function
train_data <- var_data[2:112,] # Fitting the model on all dates but the last
test_data <- var_data[113,]    # Testing the model on the last date
VAR_est <- VAR(train_data, p = 2)
VAR_est
```

    ## 
    ## VAR Estimation Results:
    ## ======================= 
    ## 
    ## Estimated coefficients for equation GDP_ret: 
    ## ============================================ 
    ## Call:
    ## GDP_ret = GDP_ret.l1 + Unemp_ret.l1 + GDP_ret.l2 + Unemp_ret.l2 + const 
    ## 
    ##   GDP_ret.l1 Unemp_ret.l1   GDP_ret.l2 Unemp_ret.l2        const 
    ##  0.479483246 -0.015960002  0.191987786  0.023239009  0.002365023 
    ## 
    ## 
    ## Estimated coefficients for equation Unemp_ret: 
    ## ============================================== 
    ## Call:
    ## Unemp_ret = GDP_ret.l1 + Unemp_ret.l1 + GDP_ret.l2 + Unemp_ret.l2 + const 
    ## 
    ##   GDP_ret.l1 Unemp_ret.l1   GDP_ret.l2 Unemp_ret.l2        const 
    ##  -1.62836886   0.26023443  -0.35321054   0.04992616   0.01502423

### Forecast

Finally, the forecast.

``` r
predict(VAR_est, n.ahead = 1)
```

    ## $GDP_ret
    ##                     fcst       lower      upper          CI
    ## GDP_ret.fcst 0.009536154 0.001051062 0.01802125 0.008485093
    ## 
    ## $Unemp_ret
    ##                       fcst       lower      upper         CI
    ## Unemp_ret.fcst -0.02055491 -0.06647668 0.02536686 0.04592177

``` r
test_data
```

    ## # A tibble: 1 x 2
    ##   GDP_ret Unemp_ret
    ##     <dbl>     <dbl>
    ## 1 0.00581    0.0349

The quality of the forecast in this particular exemple is not impressive...
