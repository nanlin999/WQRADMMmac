# WQRADMM for macOS users
## Instructions

R package for distributed quantile regression in longitudinal big data based on ADMM.

It can be used to reproduce the simulation studies in the following paper:

**Ye Fan, Nan Lin and Liqun Yu**. *Distributed Quantile Regression for Longitudinal Big Data.*

Two main functions are included: **WQRADMM( )** and **paraWQRADMM( )**, repectively designed for non-distributed and distributed environments.

## Installation requirements
```
Install Xcode before installing this package. 
```
## Code for installing WQRADMM 
```
install.packages("devtools")
library(devtools)
install_github("https://github.com/nanlin999/WQRADMMmac", force = TRUE)
library(WQRADMM)
```

## Code for testing WQRADMM

```
###function for generating the correlation matrix (AR(1) or exchangeable)
gcov = function(p, rho, type){
  if(type == "exchangeable"){
    cov = matrix(rho, p, p)
    diag(cov) = rep(1, p)
  }
  else{
    cov = diag(p)
    for(i in 1:p){
      for(j in 1:p){
        if(i < j) cov[i,j] = rho^{j-i}
        else cov[i,j] = cov[j,i]
      }
    }
  }
  cov
}

###generate synthetic data (heteroscedastic model with d = 0.75*p under Student's t error)
N = 10000
p = 100
n = 10
rep = rep(n, N)
nsum = sum(rep)
d = 0.75*p
rho_X = 0.5
rho_e = 0.5
tau = 0.75
set.seed(999)
X = matrix(rnorm(nsum*p), nsum, p)
cov_X = gcov(p, rho_X, "ar1")
X = X%*%chol(cov_X)
for(i in 1:d){
  X[,i] = pnorm(X[,i])
}
beta = rnorm(p)
cov_e = gcov(n, rho_e, "ar1")
e = matrix(rt(N*n, 3), N, n)
e = as.vector(t(e%*%chol(cov_e)))
sigma = 0.5
e = sigma*e
Y = X%*%beta+apply(X[,1:d]*e/d, 1, sum)
beta_true = c(quantile(e/d, tau)+beta[1:d], beta[(d+1):p])

###calculate the traditional quantile regression estimator
CQR = WQRADMM::WQRADMM(X, Y, rep, tau, FALSE, "CQR")
beta_CQR = CQR$Estimation_CQR
AE_CQR = sum(abs(beta_CQR-beta_true))
Iteration_CQR = CQR$Iteration_CQR
Time_CQR = CQR$Time_CQR

###calculate the weighted quantile regression estimator
WQR = WQRADMM::WQRADMM(X, Y, rep, tau, FALSE, "WQR")
beta_WQR = WQR$Estimation_WQR
AE_WQR = sum(abs(beta_WQR-beta_true))
Iteration_WQR = WQR$Iteration_WQR
Time_WQR = WQR$Time_WQR
Time_total = WQR$Time_total

###output the results
AE_WQR     ##estimation error
Time_WQR   ##running time for solving WQR problem  
Time_total ##total computational time
```
