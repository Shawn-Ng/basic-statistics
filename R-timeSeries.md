# Time Series Analysis

Editor: Shawn Ng<br>
Content Author: **David S. Matteson**<br>
[Site](https://www.datacamp.com/courses/introduction-to-time-series-analysis)<br>

1. [Exploratory time series data analysis](#1-exploratory-time-series-data-analysis)
    - [Sampling frequency](#sampling-frequency)
    - [Missing values](#missing-values)
    - [Time series](#time-series)
2. [Predicting the future](#2-predicting-the-future)
    - [Trend](#trend)
    - [White noise (WN) model](#white-noise-wn-model)
    - [Random walk (RW) model](#random-walk-rw-model)
    - [Stationary](#stationary)
3. [Correlation analysis and the autocorrelation function](#3-correlation-analysis-and-the-autocorrelation-function)
    - [Finanical time series](#financial-ts)
    - [Autocorrelation](#autocorrelation)
4. [Autoregression (AR) model](#4-autoregression-ar-model)
    - Autocorrelations can be estimated at many lags to better assess how a time series relates to its past
    - Each observation is regressed on the previous observation
5. [Moving average (MA) model](#5-moving-average-ma-model)
    - [AR vs MA models](#ar-vs-ma-models)





## 1. Exploratory time series data analysis
### Sampling frequency
```r
start(ts_data)
end(ts_data)

# time creates the vector of times at which a time series was sampled
time(ts_data)

# deltat the time interval between observations
deltat(ts_data)

# frequency returns the number of samples per unit time
frequency(ts_data)

# cycle gives the positions in the cycle of each observation
cycle(ts_data)
```

### Missing values
```r
mean(data, na.rm=TRUE)
```

### Time series
```r
ts(data, freq=TIME)
is.ts()

ts.plot(ts_data, col)
legend('topleft', legendLocation, colnames(ts_data))
```





## 2. Predicting the future
### Trend
Common types of trends

- Linear: persistent growth or decay over time
    - removing trends in level by differencing
        - `diff(ts_data, s)`: remove linear trend using seasonal difference transformation
        - remove periodic trends

- Rapid growth/decay
    - linearize using `log()`

- Variance: increasing variability over time
    - removing trend in variability using log transformation
        - log series: linearize linear trend
        - Only use for +ve linear trend
        - It slightly shrinks observations that are greater than one towards zero, while greatly shrinking very large observations. This property can stabilize variability when a series exhibits increasing variability over time

- Periodic or seasonal patterns
    - `diff(ts_data, s)`: removing seasonal trends with seasonal differencing

```r
# removing trend in variability using log transformation
data_log <- log(ts_data)
ts.plot(data_log)

# removing trends in level by differencing
data_diff <- diff(ts_data)
ts.plot(data_diff)

# removing seasonal trends with seasonal differencing
data_seasonal <- diff(ts_data, lag=4)
ts.plot(data_seasonal)
```

### White noise (WN) model
1. Fixed, constant mean
2. Fixed, constant variance
3. No correlation over time
4. Display no obvious pattern

```r
# White model has arima(0,0,0) model, n=100 obs
white_noise <- arima.sim(model = list(order=c(0,0,0)), n=100, mean=VALUE, sd=VALUE)
ts.plot(white_noise)

# Estimate the white noise model
arima(ts_data, order=c(0,0,0))
```

### Random walk (RW) model
RW model is not stationary and exhibits very strong persistence. Its sample autocovariance function (ACF) also decays to zero very slowly, meaning past values have a long lasting impact on current values.

1. No specified mean/variance
2. Strong dependence over time -> little covariance
3. Its changes/increments are white noise

```r
# RW has order=c(0,1,0)
rw <- arima.sim(model = list(order=c(0,1,0)), n=100)
ts.plot(rw)

# Generate a RW model with a drift uing arima.sim
rw_drift <- arima.sim(model = list(order = c(0, 1, 0)), n = 100, mean = 1)

# 1st diff
rw_diff <- diff(rw)
ts.plot(rw_diff)

# fit WN to differenced RW data
model_wn <- arima(diff(rw), order=c(0,0,0))

# intercept
int_wn <- model_wn$coef

# Use abline(0, ...) to add time trend to the figure
abline(0, int_wn)
```

### Stationary
This means that the autocorrelation for any particular lag is the same regardless of where we are in time. __Mean, variance, covariance constant for all t__.

- Are parsimonious
- Has distributional stability over time

__Observed time series__:

- Fluctuate randomly
- Behave similarly from 1 time period to the next
- Can modeled with __fewer params__

```r
# Convert WN data to RW, vice-versa
random_walk <- cumsum(white_noise)
white_noise <- cumsum(random_walk)
```





## 3. Correlation analysis and the autocorrelation function
### Financial ts
```r
# colMeans(): calculate the sample mean for each column
colMeans(ts_data)

# calculate sample var/hist/qqnorm for each index
apply(ts_data, FUN=var/hist/qqnorm)

# to make a scatterplot matrix of the indices in ts_data
pairs(ts_data)

# sample covariances and correlations
cov(ts_data); cor(ts_data)
```

- Correlation is standandized of covariance
- Correlation is always between -1 to 1

`cor(A, B) = cov(A, B) / (sd(A) * sd(B))`

### Autocorrelation
```r
cor(ts_data)

# lag-i autocorrelation
acf(ts_data, lag.max=10, plot=TRUE)
```





## 4. Autoregression (AR) model
Autoregressive (AR) recursion:
Today = Constant + slope * yesterday + Noise

Mean centered version:
(Today - Mean) = Slope * (Yesterday - Mean) + Noise

- WN & RW model are special cases
- RW model is a special case of AR modoel, where slope params = 1
- Persistence is defined by a high correlation between an observation and its lag
- Anti-persistence is defined by a large amount of variation between an observation and its lag
- Strong persistence = very little convariance

```r
# AR model with 0.5 slope
x <- arima.sim(model=list(ar=0.5), n=100)

# ACF estimate the autocorrelation function
acf(ts_data)

# Fitting AR model
AR <- arima(ts_data, order=c(1,0,0))
ts.plot(ts_data)
AR_fitted <- AR - residuals(AR)
points(AR_fitted)

# Predict 1 to 10 steps ahead
predict(ts_data, n.ahead=10)
```

### Simple forecasts from an estimated AR model
```r
# Fit an AR model to ts_data
AR_fit <- arima(ts_data, order = c(1,0,0))
print(AR_fit)

# Use predict() to make a 1-step forecast
predict_AR <- predict(AR_fit)

# Obtain the 1-step forecast using $pred[1]
predict_AR$pred[1]

# Use predict to make 1-step through 10-step forecasts
predict(AR_fit, n.ahead = 10)

# Run to plot the ts_data series plus the forecast and 95% prediction intervals
ts.plot(ts_data, xlim = c(1871, 1980))
AR_forecast <- predict(AR_fit, n.ahead = 10)$pred
AR_forecast_se <- predict(AR_fit, n.ahead = 10)$se
points(AR_forecast, type = "l", col = 2)
points(AR_forecast - 2*AR_forecast_se, type = "l", col = 2, lty = 2)
points(AR_forecast + 2*AR_forecast_se, type = "l", col = 2, lty = 2)
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
