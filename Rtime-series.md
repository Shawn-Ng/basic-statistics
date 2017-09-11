# Time Series Analysis

Editor: Shawn Ng<br>
Content Author: **David S. Matteson**<br>
Site: https://www.datacamp.com/courses/introduction-to-time-series-analysis<br>

1. [Exploratory time series data analysis](#1-exploratory-time-series-data-analysis)
2. [Predicting the future](#2-predicting-the-future)
    1. [Trend](#trend)
    2. [White noise (WN) model](#white-noise-wn-model)
    3. [Random walk (RW) model](#random-walk-rw-model)
    4. [Stationary](#stationary)
3. [Correlation analysis and the autocorrelation function](#3-correlation-analysis-and-the-autocorrelation-function)
    1. [Finanical time series](#financial-ts)
    2. [Autocorrelation](#autocorrelation)
4. [Autoregression (AR) model](#4-autoregression-ar-model)
5. [Moving average (MA) model](#5-moving-average-ma-model)
    1. [AR vs MA models](#ar-vs-ma-models)


## 1. Exploratory time series data analysis

### Sampling frequency
```r
> data
     Jan Feb Mar Apr May Jun Jul Aug Sep Oct Nov Dec
1949 xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx
1950 xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx
1951 xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx
1952 xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx
1953 xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx
1954 xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx
1955 xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx
1956 xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx
1957 xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx
1958 xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx
1959 xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx
1960 xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx xxx

Relevant functions:
start(), end(), time(), deltat(), frequency(), cycle()
```

### Missing values
```r
mean(data, na.rm=TRUE)
```

### Time series
```r
ts(data, freq=ytimeline)
is.ts()

ts.plot(data, col:)
legend(legendLocation, colnames(data))
```





## 2. Predicting the future

### Trend

Common types of trends

- persistent growth or decay over time
  - removing trends in level by differencing
    - diff series: remove linear trend
    - diff(x, s): seasonal difference transformation
    - remove periodic trends

- rapid growth/decay

- increasing variability over time
  * removing trend in variability using log transformation
    - log series: linearize linear trend
    - only use for +ve linear trend
    - It slightly shrinks observations that are greater than one towards zero, while greatly shrinking very large observations. This property can stabilize variability when a series exhibits increasing variability over time

- periodic or seasonal patterns
  * removing seasonal trends with seasonal differencing

```r
# removing trend in variability using log transformation
data <- log(data)
ts.plot(data)

# removing trends in level by differencing
dataE <- diff(data)
ts.plot(dataE)
length(data)
length(dataE)

# removing seasonal trends with seasonal differencing
dataE <- diff(data, lag=4)
ts.plot(dataE)
length(data)
length(dataE)
```


### White noise (WN) model
1. Fixed, constant mean
2. Fixed, constant variance
3. No correlation over time
4. Display no obvious pattern

```r
# White model has arima(0,0,0) model, n=100 obs
white_noise <- arima.sim(model = list(order=c(0,0,0)), n=100, mean=value, sd=value)
ts.plot(white_noise)

# Estimate WN model
arima(timeSeries, order=c(0,0,0))
```


### Random walk (RW) model
1. No specified mean/variance
2. Strong dependence over time -> little covariance
3. Its changes/increments are white noise

```r
# RW has order=c(0,1,0)
random_walk <- arima.sim(model = list(order=c(0,1,0)), n=100)
ts.plot(random_walk)

# 1st diff
random_walk_diff <- diff(random_walk)
ts.plot(random_walk_diff)

# fit WN to differenced RW data
model_wn <- arima(diff(random_walk), order=c(0,0,0))
# intercept
model_wn$coef
```


### Stationary
- Are parsimonious
- Has distributional stability over time

**Observed time series**:

- Fluctuate randomly
- Behave similarly from 1 time period to the next
- Can modeled with fewer params

```r
# Convert WN data to RW
random_walk <- cumsum(white_noise)
```





## 3. Correlation analysis and the autocorrelation function

### Financial ts
```r
# colMeans(): calculate the sample mean for each column
colMeans()

# calculate sample var/hist/qqnorm for each index
apply(data, FUN=var/hist/qqnorm)

# to make a scatterplot matrix of the indices in data
pairs(data)

# sample covariances and correlations
cov(); cor()
```

- Correlation is standandized of covariance
- Correlation is always between -1 to 1

`cor(A, B) = cov(A, B) / (sd(A) * sd(B))`

### Autocorrelation
```r
cor()

# lag-i autocorrelation
acf(data, lag.max)
```





## 4. Autoregression (AR) model
Autoregressive (AR) recursion:
Today = Constant + slope * yesterday + Noise

Mean centered version:
(Today - Mean) = Slope * (Yesterday - Mean) + Noise

- WN & RW model are special cases
- RW model is a special case of AR modoel, where slope params = 1
- Strong persistence = very little convariance

```r
# AR model with 0.5 slope
x <- arima.sim(model=list(ar=0.5), n=100)

# ACF estimate the autocorrelation function
acf(data)

# Fitting AR model
AR <- arima(data, order=c(1,0,0))
ts.plot(data)
AR_fitted <- AR - residuals(AR)
points(AR_fitted)

# Predict 1 to 10 steps ahead
predict(data, n.ahead=10)
```




## 5. Moving average (MA) model
- WN model is a special case
- Shows short-run dependence.

```r
# Generate MA model with slope 0.5
x <- arima.sim(model=list(ma = 0.5), n=100)
acf(x)

# Fit the MA model to x
MA <- arima(x, order=c(0,0,1))
MA_fit <- x - resid(MA)
points(MA_fit, type = "l", col = 2, lty = 2)

# Simple forecasts from an estimated MA model
predict_MA <-predict(MA, n.ahead=1)
MA_forecasts <- predict(MA, n.ahead = 10)$pred
MA_forecast_se <- predict(MA, n.ahead = 10)$se
points(MA_forecasts, type = "l", col = 2)
points(MA_forecasts - 2*MA_forecast_se, type = "l", col = 2, lty = 2)
points(MA_forecasts + 2*MA_forecast_se, type = "l", col = 2, lty = 2)
```

### AR vs MA models
To determine model fit (for AR & MA), you can measure the Akaike information criterion (AIC) and Bayesian information criterion (BIC)

```r
# small value -> better fit
AIC(AR)
BIC(AR)
```
