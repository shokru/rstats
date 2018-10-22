Distributions
================

Early illustrations
-------------------

We are interested in prices and carats of diamonds. To get a glimpse of their distribution, it is customary to plot histograms.

``` r
library(tidyverse)
diamonds %>% filter(carat < 2.55) %>% ggplot() + geom_histogram(aes(x = carat), bins = 100)
```

![](Figures/ex1-1.png)

``` r
diamonds %>% filter(carat < 1.1) %>% ggplot() + geom_histogram(aes(x = carat), bins = 200) # Higher precision for small diamonds
```

![](Figures/ex1-2.png)

``` r
diamonds %>% filter(carat < 2.55) %>% ggplot() + geom_histogram(aes(x = price), bins = 30)
```

![](Figures/ex1-3.png)

``` r
diamonds %>% ggplot(aes(x = cut)) + geom_bar(aes(y = (..count..)/sum(..count..))) + ylab("Proportion")
```

![](Figures/ex1-4.png)

Sometimes, it is easier to quantify the distribution of a variable with a few numbers: the mean, the standard deviation, etc. They are usually called the 'descriptive statistics'.

``` r
diamonds %>% summary()
```

    ##      carat               cut        color        clarity     
    ##  Min.   :0.2000   Fair     : 1610   D: 6775   SI1    :13065  
    ##  1st Qu.:0.4000   Good     : 4906   E: 9797   VS2    :12258  
    ##  Median :0.7000   Very Good:12082   F: 9542   SI2    : 9194  
    ##  Mean   :0.7979   Premium  :13791   G:11292   VS1    : 8171  
    ##  3rd Qu.:1.0400   Ideal    :21551   H: 8304   VVS2   : 5066  
    ##  Max.   :5.0100                     I: 5422   VVS1   : 3655  
    ##                                     J: 2808   (Other): 2531  
    ##      depth           table           price             x         
    ##  Min.   :43.00   Min.   :43.00   Min.   :  326   Min.   : 0.000  
    ##  1st Qu.:61.00   1st Qu.:56.00   1st Qu.:  950   1st Qu.: 4.710  
    ##  Median :61.80   Median :57.00   Median : 2401   Median : 5.700  
    ##  Mean   :61.75   Mean   :57.46   Mean   : 3933   Mean   : 5.731  
    ##  3rd Qu.:62.50   3rd Qu.:59.00   3rd Qu.: 5324   3rd Qu.: 6.540  
    ##  Max.   :79.00   Max.   :95.00   Max.   :18823   Max.   :10.740  
    ##                                                                  
    ##        y                z         
    ##  Min.   : 0.000   Min.   : 0.000  
    ##  1st Qu.: 4.720   1st Qu.: 2.910  
    ##  Median : 5.710   Median : 3.530  
    ##  Mean   : 5.735   Mean   : 3.539  
    ##  3rd Qu.: 6.540   3rd Qu.: 4.040  
    ##  Max.   :58.900   Max.   :31.800  
    ## 

``` r
diamonds %>% select(carat, price) %>% apply(2,sd) # Computing the standard deviation over carats & prices
```

    ##        carat        price 
    ##    0.4740112 3989.4397381

``` r
diamonds %>% select(carat, price) %>% filter(carat < 2.1) %>% summary() # This illustrates the sensitivity of the mean (3933 => 3765 for the price) w.r.t. extreme points: the median (2401 => 2357) is much more stable. Outliers have a limited impact on the median.
```

    ##      carat           price      
    ##  Min.   :0.200   Min.   :  326  
    ##  1st Qu.:0.400   1st Qu.:  942  
    ##  Median :0.700   Median : 2357  
    ##  Mean   :0.775   Mean   : 3765  
    ##  3rd Qu.:1.030   3rd Qu.: 5166  
    ##  Max.   :2.090   Max.   :18818

The reason why it is convenient to work with a single figure is that it can easily be computed on many subsamples. It is harder to visually analyse many distributions. Below, we analyse many subcases, i.e., when working with subgroups pertaining to each combination of cut, clarity and color.

``` r
means <- diamonds %>% group_by(cut, clarity, color) %>% 
  summarize(avg_carat = mean(carat), avg_price = mean(price))
means %>% ggplot(aes(x = avg_carat, y = avg_price)) + geom_point(aes(color = clarity, size = cut))
```

![](Figures/group_by-1.png)

``` r
means %>% ggplot(aes(x = avg_carat, y = avg_price)) + geom_point(aes(color = cut, size = clarity))
```

![](Figures/group_by-2.png)

The above plots reveal different clusters! Typically, they show the impact of the clarity on the size of the diamonds (and hence on their price). Or is it the other way around?

For the sake of illustration, we plot a series of histograms that characterize the price distribution over various subsets of diamonds. The x-axis and y-axis are common across all histograms to ease comparison.

``` r
diamonds %>% ggplot(aes(x = price)) + 
    geom_histogram(bins = 10) + 
    facet_grid(cut ~ color) +
    theme(text = element_text(size=12))
```

![](Figures/facets-1.png)

### Convergence from histogram (discrete) to density (continuous)

``` r
x <- rnorm(10^6) %>% data.frame()
colnames(x) <- "simulation"
x %>% ggplot(aes(x = simulation)) + geom_histogram(bins = 20)
```

![](Figures/convergence-1.png)

``` r
x %>% ggplot(aes(x = simulation)) + geom_histogram(bins = 100)
```

![](Figures/convergence-2.png)

``` r
x %>% ggplot(aes(x = simulation)) + geom_histogram(bins = 1000)
```

![](Figures/convergence-3.png)

Parametric families
-------------------

For modelling purposes, it is convenient to specify simple functions that characterize distributions. An exhaustive list of probability distributions is provided here: <https://en.wikipedia.org/wiki/List_of_probability_distributions>

### Continuous distributions over the real line

The most famous of them is the Gaussian (normal) distribution, which was just used above.

``` r
ggplot(data.frame(x = c(-6, 9)), aes(x)) + 
  stat_function(fun = dnorm, args = list(mean = 0, sd = 1), aes(color = "m = 0.0, s = 1.0")) +
  stat_function(fun = dnorm, args = list(mean = 0.5, sd = 1.5), aes(color = "m = 0.5, s = 1.5")) +
  stat_function(fun = dnorm, args = list(mean = 1, sd = 2), aes(color = "m = 1.0, s = 2.0")) +
  stat_function(fun = dnorm, args = list(mean = 1.5, sd = 2.5), aes(color = "m = 1.5, s = 2.5")) +
  scale_colour_manual("Legend", values = c("#B266FF", "#6666FF", "#66B2FF", "#00FF80"))
```

![](Figures/norm_dens-1.png)

The coding of many RGB colors can be found here: <https://www.rapidtables.com/web/color/RGB_Color.html>
The Gaussian distribution has very light tails: extreme points are very rare. Its heavy-tailed *cousin* is the Student distribution.

``` r
ggplot(data.frame(x = c(-7, 7)), aes(x)) + 
  stat_function(fun = dt, args = list(df = 0.5), aes(color = "df = 0.5 (Student)")) +
  stat_function(fun = dt, args = list(df = 1.0), aes(color = "df = 1.0 (Student)")) + 
  stat_function(fun = dt, args = list(df = 2.0), aes(color = "df = 2.0 (Student)")) + 
  stat_function(fun = dnorm, args = list(mean = 0, sd = 2), aes(color = "m = 0, s = 2.0 (Gaussian)")) 
```

![](Figures/t_dens-1.png)

``` r
ggplot(data.frame(x = c(-9, -3)), aes(x)) + 
  stat_function(fun = dt, args = list(df = 0.5), aes(color = "df = 0.5 (Student)")) +
  stat_function(fun = dt, args = list(df = 1.0), aes(color = "df = 1.0 (Student)")) + 
  stat_function(fun = dt, args = list(df = 2.0), aes(color = "df = 2.0 (Student)")) + 
  stat_function(fun = dnorm, args = list(mean = 0, sd = 2), aes(color = "m = 0, s = 2.0 (Gaussian)")) 
```

![](Figures/t_dens-2.png)

On the second graph, we have zoomed on the left tail to see the difference.

### Positive continuous distributions

The gamma distribution is one archetypal such distribution. It nests the exponential family as a special case.

``` r
ggplot(data.frame(x = c(0, 6)), aes(x)) + 
  stat_function(fun = dgamma, args = list(rate = 1, shape = 1), aes(color = "a = 1, b = 1")) +
  stat_function(fun = dgamma, args = list(rate = 1, shape = 2), aes(color = "a = 2, b = 1")) +
  stat_function(fun = dgamma, args = list(rate = 2, shape = 1), aes(color = "a = 1, b = 2")) +
  stat_function(fun = dgamma, args = list(rate = 2, shape = 2), aes(color = "a = 2, b = 2")) +
  scale_colour_manual("Legend", values = c("#FF3333", "#FF9933", "#66B2FF", "#00FF80"))
```

![](Figures/gamma_dens-1.png)

``` r
ggplot(data.frame(x = c(0, 1)), aes(x)) + 
  stat_function(fun = dbeta, args = list(shape1 = 1, shape2 = 3), aes(color = "a = 1, b = 3")) +
  stat_function(fun = dbeta, args = list(shape1 = 1, shape2 = 5), aes(color = "a = 1, b = 5")) +
  stat_function(fun = dbeta, args = list(shape1 = 2, shape2 = 2), aes(color = "a = 2, b = 2")) +
  stat_function(fun = dbeta, args = list(shape1 = 2, shape2 = 6), aes(color = "a = 2, b = 6")) +
  scale_colour_manual("Legend", values = c("#FF3333", "#FF9933", "#66B2FF", "#00FF80"))
```

![](Figures/beta_dens-1.png)

Other examples include the Pareto and the lognormal distributions.
Another class of continuous distributions is that of laws with bounded support. The uniform and the beta laws belong to this class.

### Discrete distributions

The most classical ones are the Poisson (infinite support) and binomial distributions (finite support).

``` r
ggplot(data.frame(x = c(0,8)), aes(x)) + 
  stat_function(aes(color = "lambda = 1", size = 1.3), geom = "point", n = 9, fun = dpois, args = list(lambda = 1)) +
  stat_function(aes(color = "lambda = 2", size = 1.2), geom = "point", n = 9, fun = dpois, args = list(lambda = 2)) +
  stat_function(aes(color = "lambda = 3", size = 1.1), geom = "point", n = 9, fun = dpois, args = list(lambda = 3))
```

![](Figures/discrete-1.png)

Other usual families: hypergeometric, beta-binomial (finite support), negative binomial (infinite support).

One last graph to show the shape of cumulative distribution functions.

``` r
ggplot(data.frame(x = c(-3,5)), aes(x)) + 
  stat_function(fun = pnorm, args = list(mean = 0, sd = 1), aes(color = "GAUSSIAN, m = 0.0, s = 1.0")) +
  stat_function(aes(color = "POISSON, lambda = 2", size = 1.2), geom = "point", n = 9, fun = ppois, args = list(lambda = 2)) 
```

![](Figures/cdf-1.png)

### Strange mixes

Densities can have strange shapes.

``` r
dens = function(x){(dnorm(x,2,3) + dt(x,2))/2 } # DENSITY NOT EQUAL TO THAT OF THE SUM OF THE TWO CORRESPONDING RVs!!!
dens_2 = function(x){(dnorm(x,-2,0.8) + dnorm(x,1,1.2))/2 }
ggplot(data.frame(x = c(-6, 9)), aes(x)) + stat_function(fun = dens, aes(color = "mix_1")) +
  stat_function(fun = dens_2, aes(color = "mix_2"))
```

![](Figures/mix-1.png)

Correlations
------------

Very often, variables come in groups. Studying their univariate properties is of course useful, but understanding how they interact is sometimes even more insightful. Because we expect the size of a diamond to have a predominant impact on its price, the variables price and carat should be strongly positively correlated.

``` r
cor(diamonds$carat, diamonds$price)
```

    ## [1] 0.9215913

A stylized sample is visually clear.

``` r
library(MASS)
```

    ## 
    ## Attaching package: 'MASS'

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     select

``` r
m <- c(0,0)
rho <- 0.9
sig <- rbind(c(1,rho), c(rho,1))
z <- mvrnorm(1000, mu = m, Sigma = sig) %>% data.frame()
ggplot(z, aes(x = X1, y = X2)) + geom_point()
```

![](Figures/cor_simulated-1.png)

Let's see what happens with diamonds!

``` r
diamonds %>% 
    ggplot(aes(x = carat, y = price, color = clarity)) + 
    geom_point() +
    facet_grid(color ~ cut) +
    theme(text = element_text(size=12))
```

![](Figures/cor_diamonds-1.png)

Obviously, the correlation is positive. The slopes a probably non-linear and depend both on clarity and color.
