
<!-- README.md is generated from README.Rmd. Please edit that file -->

# IntegrativeFPM

The goal of *IntegrativeFPM* is to implement integrative analyses for
the finite population mean parameters combining a non-probability sample
with a probability sample which provides high-dimensional representative
covariate information of the target population.

Two datasets

  - The non-probability sample contains observations on (X,Y); however,
    the sampling mechanism is unknown.

  - The probability sample with sampling weights represents the finite
    population; however, it contains observations only on X.

The function *IntegrativeFPM* implements a two-step approach for
integrating the probability sample and the nonprobability sample for
finite population inference.

  - Step 1 selects important variables in the sampling score model and
    the outcome model by penalization.

  - Step 2 conducts a doubly robust inference for the finite population
    mean of interest. In this step, the nuisance model parameters are
    refitted based on the selected variables by minimizing the
    asymptotic squared bias of the doubly robust estimator. This
    estimating strategy mitigates the possible first-step selection
    error and renders the doubly robust estimator root-n consistent if
    either the sampling probability or the outcome model is correctly
    specified.

## Installation with `devtools`:

``` r
devtools::install_github("shuyang1987/IntegrativeFPM")
```

### Main Paper: Yang, Kim, and Song (2019)

Yang, S., Kim, J.K., and Song, R. (2019). Doubly Robust Inference when
Combining Probability and Non-probability Samples with High-dimensional
Data.

## Example

This is an example for illustration.

``` r

set.seed(1234)

## population size

N <- 10000

## x is a p-dimensional covariate

p <- 50
x <- matrix( rnorm(N*p,0,1),N,p)

## y is a continuous outcome 

beta0 <- c(1,1,1,1,1,rep(0,p-4))
y <- cbind(1,x)%*%beta0 + rnorm(N,0,1)
true <- mean(y)

## y2 is a binary outcome

ly2 <- (cbind(1,x)%*%beta0)
ply <- exp(ly2)/(1+exp(ly2))
y2 <- rbinom(N,1,ply)
true2 <- mean(y2)

## A.set is a prob sample: SRS
## sampling probability into A is known when estimation

nAexp <- 1000
probA <- rep(nAexp/N,N)
A.index <- rbinom(N,size = 1,prob = probA)
A.loc <- which(A.index == 1)
nA <- sum(A.index == 1)
sw.A <- 1/probA[A.loc]
x.A <- x[A.loc,]
y.A <- rep(NA,nA) # y is not observed in Sample A
y2.A <- rep(NA,nA)

## B.set is a nonprob sample
## sampling probability into B is unknown when estimation

nBexp <- 2000
alpha0 <- c(-2,1,1,1,1,rep(0,p-4))
probB <- (1+exp(-cbind(1,x)%*%alpha0))^(-1) 
B.index <- rbinom(N,size = 1,prob = probB)
B.loc <- which(B.index == 1)
nB <- sum(B.index)
x.B <- x[B.loc,]
y.B <- y[B.loc]
y2.B <- y2[B.loc]

## combined dataset

y.AB <- c(y.A,y.B)
y2.AB <- c(y2.A,y2.B)
x.AB <- rbind(x.A,x.B)
deltaB <- c(rep(0,nA),rep(1,nB))
sw <- c(sw.A,rep(1,nB))

## specify tuning parameters

lambda_a <- 1.25
lambda_b <- 0.25
lambda2_b <- 0.02

true
#> [1] 1.000452
IntegrativeFPM::IntegrativeFPM(y=y.AB, x=x.AB, deltaB, sw, family="gaussian",lambda_a, cv_a=0, lambda_b, cv_b=0)
#> $est
#> [1] 1.057844
#> 
#> $se
#> [1] 0.09016443
#> 
#> $alpha.selected
#> [1] 1 2 3 4
#> 
#> $beta.selected
#> [1] 1 2 3 4

true2
#> [1] 0.6551
IntegrativeFPM::IntegrativeFPM(y=y2.AB, x=x.AB, deltaB, sw, family="binomial",lambda_a, cv_a=0, lambda2_b, cv_b=0)
#> $est
#> [1] 0.6618466
#> 
#> $se
#> [1] 0.02876472
#> 
#> $alpha.selected
#> [1] 1 2 3 4
#> 
#> $beta.selected
#> [1] 1 2 3 4
```
