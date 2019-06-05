[Home](https://mgcodesandstats.github.io/) |
[Portfolio](https://mgcodesandstats.github.io/portfolio/) |
[Terms and Conditions](https://mgcodesandstats.github.io/terms/) |
[E-mail me](mailto:contact@michaeljgrogan.com) |
[LinkedIn](https://www.linkedin.com/in/michaeljgrogan/)

# Variance-Covariance Matrix: Stock Price Analysis in R (corpcor, covmat)

The purpose of a variance-covariance matrix is to illustrate the variance of a particular variable (diagonals) while covariance illustrates the covariances between the exhaustive combinations of variables.

## Why do we use variance-covariance matrices?

A variance-covariance matrix is particularly useful when it comes to analysing the volatility between elements of a group of data. For instance, a variance-covariance matrix has particular applications when it comes to analysing portfolio returns.

If several assets with a high covariance are included in a portfolio, then this represents high risk. This indicates that several assets with high volatility move together, which is what investors would typically want to avoid.

On the other hand, selecting assets that show negative covariance allows for greater diversification of the portfolio, since the volatility of the assets do not tend to move together.

You might find the following Investopedia article useful in gaining more information on the financial aspect.
 
## Variance-Covariance Matrix generation in R

First, we install our libraries.
```
#Install Packages
install.packages("corpcor")
install.packages("tseries")
install.packages("quantmod")
```
We then load the corpcor, tseries and quantmod libraries into the environment.
```
#Load libraries
library(corpcor)
library(tseries)
```
For the purpose of this variance-covariance matrix, let us stream stock data with quantmod to analyse the following five stocks:

- AAPL
- CBS
- EFX
- GOOGL
- VZ

Now, we need to:

- Download the relevant stock prices for the past 30 days
- Convert into daily returns by obtaining the log and daily difference for each stock to account for compounding

```
> #Quantmod
> aapl = getSymbols("AAPL",type="xts")
> cbs = getSymbols("CBS",type="xts")
> efx = getSymbols("EFX",type="xts")
> googl = getSymbols("GOOGL",type="xts")
> vz = getSymbols("VZ",type="xts")
```

Now, we store these within data frames and obtain the differences of the logs:

```
> #Store within dataframes
> aapl=data.frame(tail(AAPL$AAPL.Close,30))
> cbs=data.frame(tail(CBS$CBS.Close,30))
> efx=data.frame(tail(EFX$EFX.Close,30))
> googl=data.frame(tail(GOOGL$GOOGL.Close,30))
> vz=data.frame(tail(VZ$VZ.Close,30))

> #Logs and differences
> aapl<-diff(log(aapl$AAPL.Close),1)
> cbs<-diff(log(cbs$CBS.Close),1)
> efx<-diff(log(efx$EFX.Close),1)
> googl<-diff(log(googl$GOOGL.Close),1)
> vz<-diff(log(vz$VZ.Close),1)
```

We then combine each stock into the one data frame named portfolio:
```
> portfolio <- data.frame(aapl, cbs, efx, googl, vz)
```
Now, let us generate a **5 x 5** covariance matrix:
```
> #Range Names
> range.names = c("AAPL", "CBS", "EFX", "GOOGL", "VZ")
> names(portfolio) = range.names
> covmatrix = matrix(c(cov(portfolio)),
+                 nrow=5, ncol=5)
> dimnames(covmatrix) = list(range.names, range.names)
> 
> #Covariance matrix
> covmatrix
               AAPL           CBS          EFX         GOOGL            VZ
AAPL   5.821228e-05 -4.240955e-06 3.545054e-05  6.099968e-05  4.733450e-07
CBS   -4.240955e-06  2.778983e-04 4.656228e-05  1.597556e-05 -2.651646e-05
EFX    3.545054e-05  4.656228e-05 2.230373e-04  2.608622e-05  2.325623e-05
GOOGL  6.099968e-05  1.597556e-05 2.608622e-05  1.613921e-04 -7.482914e-06
VZ     4.733450e-07 -2.651646e-05 2.325623e-05 -7.482914e-06  1.313256e-04
```
The readings above indicate the extent to which the volatility in prices move together. For instance, we can see that the volatility between AAPL and VZ is significantly less than that between AAPL and GOOGL, since the latter are both technology stocks.

## Shrinkage estimate of covariance

It is often the case that if we have too few observations in our data (which we arguably do given we are using 30 days of data), then our covariance estimates will be unreliable.

To remedy this, we can use what is called a shrinkage estimate of covariance. This works by ensuring that our covariance matrix is positive definite, i.e. one where all eigenvalues are real and positive.

When we shrink the covariance matrix, we end up with the following:
```
> #Shrinkage estimate of covariance
> cov.shrink(covmatrix)
Estimating optimal shrinkage intensity lambda.var (variance vector): 1 

Estimating optimal shrinkage intensity lambda (correlation matrix): 0.8531 

               AAPL           CBS           EFX         GOOGL            VZ
AAPL   4.388321e-09 -3.435343e-10  4.139977e-11  5.105568e-10 -2.569910e-10
CBS   -3.435343e-10  4.388321e-09  1.561082e-11 -1.523166e-10 -3.626202e-10
EFX    4.139977e-11  1.561082e-11  4.388321e-09 -1.550852e-10 -5.283354e-11
GOOGL  5.105568e-10 -1.523166e-10 -1.550852e-10  4.388321e-09 -3.192974e-10
VZ    -2.569910e-10 -3.626202e-10 -5.283354e-11 -3.192974e-10  4.388321e-09
attr(,"lambda")
[1] 0.853093
attr(,"lambda.estimated")
[1] TRUE
attr(,"class")
[1] "shrinkage"
attr(,"lambda.var")
[1] 1
attr(,"lambda.var.estimated")
[1] TRUE
```
## Correlation Matrix

Once we have ensured that our covariance matrix is positive definite, we can convert it into a correlation matrix - i.e. determine the extent to which returns move together rather than volatility.
```
> #Transform covariance to correlation matrix
> cov2cor(covmatrix)
              AAPL         CBS       EFX       GOOGL           VZ
AAPL   1.000000000 -0.03334367 0.3111192  0.62933100  0.005413721
CBS   -0.033343673  1.00000000 0.1870262  0.07543487 -0.138802692
EFX    0.311119193  0.18702623 1.0000000  0.13749322  0.135886379
GOOGL  0.629331001  0.07543487 0.1374932  1.00000000 -0.051399037
VZ     0.005413721 -0.13880269 0.1358864 -0.05139904  1.000000000
```
## Conclusion

In this example, we have seen how to:

- Stream stock prices with quantmod
- Construct a covariance matrix with quantmod
- Shrink a covariance matrix to ensure it is positive definite
- Convert a covariance matrix into a correlation matrix
