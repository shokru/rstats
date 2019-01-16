Monte-Carlo Simulations
================

-   [Early illustrations](#early-illustrations)
-   [Simulating football outcomes](#simulating-football-outcomes)
    -   [Simple case](#simple-case)
    -   [Complex case](#complex-case)
-   [Exercises](#exercises)
    -   [Approximating Pi](#approximating-pi)
    -   [Musical predictions](#musical-predictions)
    -   [Generating artificial data](#generating-artificial-data)
    -   [Back to Pi](#back-to-pi)
    -   [**BONUS**](#bonus)

Early illustrations
-------------------

Generating simple variates.

``` r
library(tidyverse)
library(MASS)
n_samples <- 15
runif(n_samples, min = 2, max = 5)
```

    ##  [1] 3.695901 2.236514 4.148905 3.883262 4.837644 2.906620 2.193842
    ##  [8] 2.695147 3.827567 3.540240 4.433619 3.907684 3.878504 4.704410
    ## [15] 3.799516

``` r
rnorm(n_samples, mean = 3, sd = 3)
```

    ##  [1]  5.68236147  5.75137084  5.94510482  1.52249631  0.03597672
    ##  [6]  4.21212076  1.98164918  2.11097525 -3.47132364  2.08572467
    ## [11]  1.48024120  6.54456993  3.21276727  0.08223739 -2.64724897

``` r
rgamma(n_samples, shape = 1, rate = 2)
```

    ##  [1] 0.92390427 0.35745154 1.24741955 0.04705248 2.70214297 0.12578382
    ##  [7] 0.06614143 0.10200150 0.10274576 1.11306465 0.24376501 0.03305231
    ## [13] 1.08441967 0.29794456 0.89678411

Simulating football outcomes
----------------------------

### Simple case

Four teams compete. Their strengths (or scores, out of 100) are:

| France | Brazil | Germany | Vatican |
|--------|--------|---------|---------|
| 85     | 92     | 85      | 55      |

``` r
countries <- c("France", "Brazil", "Germany", "Vatican")
strength <- c(85, 92, 85, 55)
foot <- data.frame(countries, strength)
```

First round: France-Brazil and Germany-Vatican. The winners play the final. During a game, the probability Team 1 wins over Team 2 is *s*<sub>1</sub>/(*s*<sub>1</sub> + *s*<sub>2</sub>), where the *s*<sub>*i*</sub> are the scores. The purpose is to compute the probability of winning the tournament for each team.

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
summary(as.factor(winner))/n_sim
```

    ##     1     2     3     4 
    ## 0.279 0.291 0.271 0.159

### Complex case

A more detailed approach with offense and defense scores:

|         | France | Brazil | Germany | Vatican |
|---------|--------|--------|---------|---------|
| Defense | 0.90   | 0.80   | 0.86    | 0.54    |
| Offense | 0.88   | 0.94   | 0.90    | 0.56    |
| rho     | 0.90   | 0.90   | 0.90    | 0.30    |

``` r
countries <- c("France", "Brazil", "Germany", "Vatican")
defense <- c(0.90, 0.80, 0.86, 0.54)
offense <- c(0.88, 0.94, 0.90, 0.56)
rho <- c(0.9, 0.9, 0.9, 0.3)
foot <- data.frame(countries, defense, offense, rho)
```

The sequence is the same. The outcome of the matches are determined as follows. For each team, defense and offense are simulated as a Gaussian variable with unit variance and mean equal to the scores given in the table above. The correlation between the two outcomes is rho.

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
summary(as.factor(winner))/n_sim
```

    ##     1     2     3     4 
    ## 0.281 0.281 0.291 0.147

Below, a more compact version in which an additional parameter v is added to reduce randomness. The probabilities are more concentrated when variance is smaller.

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
summary(as.factor(winner))/n_sim
```

    ##     1     2     3     4 
    ## 0.288 0.274 0.396 0.042

Exercises
---------

### Approximating Pi

The equation for the unit circle is: y^2 + x^2 = 1. The surface of a circle with unit radius is equal to Pi. With this in mind, can you compute a Monte-Carlo based estimation of Pi?
How many simulations would you need to reach with a high level of confidence the number 3.14159?

### Musical predictions

The imaginary sovereign city-state of Atlantis is voting for the best singer of all time. There are 6 finalists and their relative popularity is:

**Elvis Presley**: 78%
**Madonna**: 72%
**John Lennon**: 67%
**Freddie Mercury**: 64%
**Beyoncé**: 62%
**Bob Marley**: 60%

Source: Atlantis Daily News.
The vote takes place in two rounds (there is a run-off). After the first round, the best two contenders face each other to win the title.

In order to simulate who will win, we proceed as follows: for each candidate, an exponential law is drawn with parameter equal to his/her popularity. The two candidates with **smallest** values win. Same procedure for the final round. What are the winning probabilities for each candidate? What happens if we change the parameter of the exponential distribution and we multiply the popularity by 100?

### Generating artificial data

We want to generate profiles for 100 imaginary male athletes. We want their age (years), height (cm), weight (kg) and their best running time on 100m (s).
The ranges of variables are included in: (18-30) for age, (165-194) for height (65-90) for weight and (10.1-11.0) for running time.
Choose a simulation protocol that can smartly match these ranges. The round() functions cuts decimals. Any idea how to introduce some correlation between height and weight?

### Back to Pi

We consider the first exercise. The point here is to visually grasp the convergence of the estimator as the number of simulations increases. Create a dataframe that stores the values obtained for the following 12 values of number of simulations: 3^(1:12). Then, plot the result, as well as a vertical line at 3.14.

### **BONUS**

Use Monte-Carlo simulations to compute the surface of an equilateral triangle with unit side length.
A bit of help: if you draw bivariate uniform samples on (-1,1), the equation is: max(-2y, y-sqrt(3)x, y+sqrt(3)x) &lt; sqrt(4/3) .
The true value is sqrt(3)/4, which is approximately equal to 0.433.
