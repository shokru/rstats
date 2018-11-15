Monte-Carlo Simulations
================

Early illustrations
-------------------

Generating simple variates

``` r
library(tidyverse)
```

    ## ── Attaching packages ──────────────────────────────────────────────────────────────────────────────────────── tidyverse 1.2.1 ──

    ## ✔ ggplot2 3.0.0     ✔ purrr   0.2.5
    ## ✔ tibble  1.4.2     ✔ dplyr   0.7.6
    ## ✔ tidyr   0.8.1     ✔ stringr 1.3.1
    ## ✔ readr   1.1.1     ✔ forcats 0.3.0

    ## ── Conflicts ─────────────────────────────────────────────────────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

``` r
library(MASS)
```

    ## 
    ## Attaching package: 'MASS'

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     select

``` r
runif(10, min = 2, max = 5)
```

    ##  [1] 3.720882 2.180290 4.027099 3.693903 4.866930 2.988569 2.802279
    ##  [8] 3.008651 2.625428 3.709329

``` r
rnorm(10, mean = 3, sd = 3)
```

    ##  [1]  5.4010479  1.9007447 -0.9359205 -0.4139712  7.4380308  3.2864976
    ##  [7] -1.8789547  6.3897174  2.3718910 -0.3225739

``` r
rgamma(10, shape = 1, rate = 2)
```

    ##  [1] 0.14849052 0.32651799 0.23229809 0.41626306 0.06784637 0.29333849
    ##  [7] 0.03578207 0.54082963 1.18288099 0.77133205

Simulating football outcomes
----------------------------

``` r
countries <- c("France", "Brazil", "Germany", "Vatican")
strength <- c(85, 92, 85, 55)
foot <- data.frame(countries, strength)
```

``` r
n_sim <- 10^3
winner <- c()
for(n in 1:n_sim){   # We resort to a loop for clarity. We recommend the SimDesign package to the interested reader! Or to the map() wrapper
  odds <- foot$strength[1]/(foot$strength[1]+foot$strength[2])
  if(runif(1)<odds){win_1 <- 1} else win_1 <- 2 # First round, first game, win_1 stores the winner
  odds <- foot$strength[3]/(foot$strength[3]+foot$strength[4])
  if(runif(1)<odds){win_2 <- 3} else win_2 <- 4 # First round, second game, win_2 stores the winner
  odds <- foot$strength[win_1]/(foot$strength[win_1]+foot$strength[win_2])
  if(runif(1)<odds){win <- win_1} else win <- win_2 # Second round, win is the final winner
  winner[n] <- win
}
summary(as.factor(winner))
```

    ##   1   2   3   4 
    ## 262 299 296 143

A more detailed approach with offense and defense scores.

``` r
countries <- c("France", "Brazil", "Germany", "Vatican")
defense <- c(0.90, 0.80, 0.86, 0.54)
offense <- c(0.88, 0.94, 0.90, 0.56)
rho <- c(0.9, 0.9, 0.9, 0.3)
foot <- data.frame(countries, defense, offense, rho)
```

``` r
n_sim <- 10^3
winner <- c()
for(n in 1:n_sim){   # We resort to a loop for clarity (again). 
  mu_1 <- c(foot$defense[1], foot$offense[1]) # Obviously, these lines could be automated => see below
  Sig_1 <- rbind(c(1,foot$rho[1]),c(foot$rho[1],1))
  score_1 <- mvrnorm(1, mu = mu_1, Sigma = Sig_1) %>% sum()
  mu_2 <- c(foot$defense[2], foot$offense[2])
  Sig_2 <- rbind(c(1,foot$rho[2]),c(foot$rho[2],1))
  score_2 <- mvrnorm(1, mu = mu_2, Sigma = Sig_2) %>% sum()
  if(score_1 > score_2){win_1 <- 1} else win_1 <- 2 # First round, first game
  mu_3 <- c(foot$defense[3], foot$offense[3])
  Sig_3 <- rbind(c(1,foot$rho[3]),c(foot$rho[3],1))
  score_3 <- mvrnorm(1, mu = mu_3, Sigma = Sig_3) %>% sum()
  mu_4 <- c(foot$defense[4], foot$offense[4])
  Sig_4 <- rbind(c(1,foot$rho[4]),c(foot$rho[4],1))
  score_4 <- mvrnorm(1, mu = mu_4, Sigma = Sig_4) %>% sum()
  if(score_3 > score_4){win_2 <- 3} else win_2 <- 4 # First round, second game
  mu_1 <- c(foot$defense[win_1], foot$offense[win_1]) # Obviously, these lines could be automated => see below
  Sig_1 <- rbind(c(1,foot$rho[win_1]),c(foot$rho[win_1],1))
  score_1 <- mvrnorm(1, mu = mu_1, Sigma = Sig_1) %>% sum()
  mu_2 <- c(foot$defense[win_2], foot$offense[win_2])
  Sig_2 <- rbind(c(1,foot$rho[win_2]),c(foot$rho[win_2],1))
  score_2 <- mvrnorm(1, mu = mu_2, Sigma = Sig_2) %>% sum()
  if(score_1 > score_2){win <- win_1} else win <- win_2 # Second round
  winner[n] <- win
}
summary(as.factor(winner))
```

    ##   1   2   3   4 
    ## 276 268 311 145

``` r
n_sim <- 10^3
v <- 0.1     # The impact of variance
score <- c() # We must initialize the variables we will use later on
win <- c()
winner <- c()
for(n in 1:n_sim){        # The loop over all simulations
  for(i in 1:nrow(foot)){ # Compute the scores in a loop
    mu <- c(foot$defense[i], foot$offense[i])
    Sig <- v*rbind(c(1,foot$rho[i]),c(foot$rho[i],1))
    score[i] <- mvrnorm(1, mu = mu, Sigma = Sig) %>% sum()
  }
  if(score[1] > score[2]){win[1] <- 1} else win[1] <- 2 # First round, first game
  if(score[3] > score[4]){win[2] <- 3} else win[2] <- 4 # First round, second game
  for(i in 1:2){ # Compute the scores in a loop
    mu <- c(foot$defense[win[i]], foot$offense[win[i]])
    Sig <- v*rbind(c(1,foot$rho[win[i]]),c(foot$rho[win[i]],1))
    score[i] <- mvrnorm(1, mu = mu, Sigma = Sig) %>% sum()
  } # Be very careful in the line below, the values 3 & 4 in 'score' have not been erased!
  if(score[1] > score[2]){winner[n] <- win[1]} else winner[n] <- win[2] # First round, first game
}
summary(as.factor(winner))
```

    ##   1   2   3   4 
    ## 285 284 392  39
