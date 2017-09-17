# Forecasting Using R

Editor: Shawn Ng<br>
Content Author: **Rob J. Hyndman**<br>
Site: https://www.datacamp.com/courses/forecasting-using-r<br>

1. [Exploring and visualizing time series in R](#1-exploring-and-visualizing-time-series-in-r)
2. [Benchmark methods and forecast accuracy](#2-benchmark-methods-and-forecast-accuracy)
3. [Exponential smoothing](#3-exponential-smoothing)
4. [Forecasting with ARIMA models](#4-forecasting-with-arima-models)
5. [Advanced methods](#5-advanced-methods)


## 1. Exploring and visualizing time series in R
```r
autoplot(data)

# To find outlier
which.max(data)

# No. of obs per unit time
frequency(data)

ggseasonplot(data)
ggseasonplot(data, polar=T)

# Restrict the data to start in year
dataE <- window(data, start=year)
ggsubseriesplot(dataE)

# Autocorrelation of non-seasonal time series
autoplot(data)
gglagplot(data)
ggAcf(data)

# Ljung-Box test confirms the randomness of a series; 
# p-value > 0.05 suggests that the data are not significantly different from white noise
Box.test(diff(data), lag = 10, type = "Ljung")
```





## 2. Benchmark methods and forecast accuracy
Forecast is the mean or median of simulated futures of a time series<br>
Naive forecast: use the most recent observation

```r
data_naive <- naive(data, h=20)
data_seasonal_naive <- snaive(data, h = 16)

autoplot(data)
summary(data)
```

Fitted value: 1-step forecast of all previous observations, not true forecast as all params are estimated<br>
Residuals: 1-step forecast error, diff bet fitted value and obs

```r
# p-value >= 0.05 -> white noise
checkresiduals(naive(data))
# same as, %>% pipe operator
data %>% naive() %>% checkresiduals()

checkresiduals(snaive(data))
# same as
data %>% snaive() %>% checkresiduals()
```

Forecast error: diff bet. observed value and its multi-steps forecast in the test set
* It is not residuals
* Errors on training vs test set

Measures of forecast accuracy
1. Mean abs error (MAE)
2. Mean sq error (MSE)
3. Mean abs percentage error (MAPE)
4. Mean abs scaled error (MASE)

Training set: is a data set that is used to discover possible relationships<br>
Test set: is a data set that is used to verify the strength of these potential relationships

We are interested to see the forecast accuracy of test set

```r
## Forecast accuracy of non-seasonal methods
# assume data has 1108 data points
training <- subset.ts(data, end=1000)

# naive forecast, simple forcasting functions, h=1108-1000
naiveFC <- naive(training, h=108)
# meanf(): gives forecasts equal to the mean of all observations
meanFC <- meanf(training, h=108)

# smaller root mean squared error (RMSE) -> better accuracy
accuracy(naiveFC, data)
accuracy(meanFC, data)

## Forecast accuracyy of seasonal methods
# Create three training series omitting the last 1, 2, and 3 years
train1 <- window(vn[, "colZ"], end = c(2014, 4))
train2 <- window(vn[, "colZ"], end = c(2013, 4))
train3 <- window(vn[, "colZ"], end = c(2012, 4))

# Produce forecasts using snaive()
fc1 <- snaive(train1, h = 4)
fc2 <- snaive(train2, h = 4)
fc3 <- snaive(train3, h = 4)

# Use accuracy() to compare the MAPE of each series
accuracy(fc1, vn[, "colZ"])["Test set", "MAPE"]
accuracy(fc2, vn[, "colZ"])["Test set", "MAPE"]
accuracy(fc3, vn[, "colZ"])["Test set", "MAPE"]
```

Good forecasting model?
* Small residuals only means that the model fits the training data well, not that it produces good forecasts.
* A good model forecasts has low RMSE on the test set and has white noise residuals

```r
## time series cross-validation: tsCV()
# Compute cross-validated errors for up to 8 steps ahead
e <- matrix(NA_real_, nrow = 1000, ncol = 8)
for (h in 1:8)
  e[, h] <- tsCV(data, forecastfunction = naive, h = h)

# Compute the MSE values and remove missing values
mse <- colMeans(e^2, na.rm = TRUE)
```





## 3. Exponential smoothing
```r
# ses(): simple exponential smoothing. The parameters are estimated using least squares estimation.
# h: horizon, years
fc <- ses(timeSeriesData, h=10)

# smoothing parameters, alpha=0.3457 -> 34.57% of emphasis on latest data
summary(fc)

# Add 1 step forecast for training data
autoplot(fc) + autolayer(fitted(fc))


# SES VS NAIVE
# x is the number of obs reserved for testing
train <- subset.ts(timeSeriesData, end=length(timeSeriesData)-x)

fcSes <- naive(train, h=x)
fcNaive <- ses(train, h=x)
accuracy(fcSes, timeSeriesData)
accuracy(fcNaive, timeSeriesData)


# Holt's trend methods
fcholt <- holt(timeSeriesData, h=x)
summary(fcholt)
autoplot(fcholt)
checkresiduals(fcholt)

# Holt with monthly data
# 1 year -> h=1*12
fc <- holt(timeSeriesData, seasonal="multiplicative", h=12)


# Fit ETS model to data
fitdata <- ets(timeSeriesData)
checkresiduals(fitdata)
# p value > 0.05 => model fails the Ljung-Box test
autoplot(forecast(fitdata))


# ETS VS seasonal naive
# Function to return ETS forecasts
fets <- function(y, h) {
  forecast(ets(y), h = h)
}

# Apply tsCV() for both methods
e1 <- tsCV(timeSeriesData, fets, h = 4)
e2 <- tsCV(timeSeriesData, snaive, h = 4)

# Compute MSE of resulting errors
mean(e1^2, na.rm=TRUE)
mean(e2^2, na.rm=TRUE)


# ETS doesn't always work
# Use ets() to model the lynx series
fit <- ets(timeSeriesData)

# Use summary() to look at model and parameters
summary(fit)

# Plot 20-year forecasts of the series
fit %>% forecast(h=20) %>% autoplot()
```

## 4. Forecasting with ARIMA models
```r
# 1. Box-Cox transformations
# Try four values of lambda in Box-Cox transformations
timeSeriesData %>% BoxCox(lambda = 0.0) %>% autoplot()
timeSeriesData %>% BoxCox(lambda = 0.1) %>% autoplot()
timeSeriesData %>% BoxCox(lambda = 0.2) %>% autoplot()
timeSeriesData %>% BoxCox(lambda = 0.3) %>% autoplot()

# Compare with BoxCox.lambda()
BoxCox.lambda(timeSeriesData)
```

Differencing is a way of making a time series stationary; this means that you remove any systematic patterns such as trend and seasonality from the data. A white noise series is considered a special case of a stationary time series.
```r
# 2. Non-seasonal differencing for stationarity
# Plot the rate
autoplot(timeSeriesData)

# Plot the differenced rate
autoplot(diff(timeSeriesData))

# Plot the ACF of the differenced rate
ggAcf(diff(timeSeriesData))
```

With seasonal data, differences are often taken between observations in the same season of consecutive years, rather than in consecutive periods. For example, with quarterly data, one would take the difference between Q1 in one year and Q1 in the previous year. This is called seasonal differencing.

Sometimes you need to apply both seasonal differences and lag-1 differences to the same series, thus, calculating the differences in the differences
```r
# Plot the data
autoplot(timeSeriesData)

# Take logs and seasonal differences of timeSeriesData
difflogData <- diff(log(timeSeriesData), lag = 12)

# Plot difflogData
autoplot(difflogData)

# Take another difference and plot
ddifflogData <- diff(difflogData)
autoplot(ddifflogData)

# Plot ACF of ddifflogData
ggAcf(ddifflogData)
```

`auto.arima()` function will select an appropriate autoregressive integrated moving average (ARIMA) model given a time series
```r
# Fit an automatic ARIMA model to the timeSeriesData series
fit <- auto.arima(timeSeriesData)

# Check that the residuals look like white noise
checkresiduals(fit)
summary(fit)

# Plot forecasts of fit
fit %>% forecast(h = 10) %>% autoplot()
```

### Forecasting with ARIMA models
The `Arima()` function can be used to select a specific ARIMA model. Its first argument, order, is set to a vector that specifies the values of `p`, `d` and `q`. The second argument, `include.constant`, is a booolean that determines if the constant cc, or drift, should be included
```r
# Plot forecasts from an ARIMA(0,1,1) model with no drift
timeSeriesData %>% Arima(order = c(0,1,1), include.constant = FALSE) %>% forecast() %>% autoplot()

# Plot forecasts from an ARIMA(2,1,3) model with drift
timeSeriesData %>% Arima(order = c(2,1,3), include.constant = TRUE) %>% forecast() %>% autoplot()

# Plot forecasts from an ARIMA(0,0,1) model with a constant
timeSeriesData %>% Arima(order = c(0,0,1), include.constant = TRUE) %>% forecast() %>% autoplot()

# Plot forecasts from an ARIMA(0,2,1) model with no constant
timeSeriesData %>% Arima(order = c(0,2,1), include.constant = FALSE) %>% forecast() %>% autoplot()
```

### Comparing auto.arima() and ets() on non-seasonal data
The AICc statistic is useful for selecting between models in the same class. For example, you can use it to select an ETS model or to select an ARIMA model. However, you cannot use it to compare ETS and ARIMA models because they are in different model classes.
```r
# Set up forecast functions for ETS and ARIMA models
fets <- function(x, h) {
  forecast(ets(x), h = h)
}
farima <- function(x, h) {
  forecast(auto.arima(x), h=h)
}

# Compute CV errors for ETS as e1
e1 <- tsCV(timeSeriesData, fets, 1)

# Compute CV errors for ARIMA as e2
e2 <- tsCV(timeSeriesData, farima, 1)

# Find MSE of each model class
mean(e1^2, na.rm=TRUE)
mean(e2^2, na.rm=TRUE)

# Plot 10-year forecasts using the best model class
timeSeriesData %>% farima(h=10) %>% autoplot()
```

### Automatic ARIMA models for seasonal time series
`auto.arima()` function also works with seasonal data. Note that setting `lambda = 0` in the `auto.arima()` function - applying a log transformation - means that the model will be fitted to the transformed data, and that the forecasts will be back-transformed onto the original scale
```r
# Check that the logged timeSeriesData have stable variance
timeSeriesData %>% log() %>% autoplot()
fit <- auto.arima(timeSeriesData, lambda=0)
summary(fit)
# Plot 2-year forecasts
fit %>% forecast(h=24) %>% autoplot()
```

The `auto.arima()` function needs to estimate a lot of different models, and various short-cuts are used to try to make the function as fast as possible. This can cause a model to be returned which does not actually have the smallest AICc value. To make `auto.arima()` work harder to find a good model, add the optional argument `stepwise = FALSE` to look at a much larger collection of models
```r
auto.arima(timeSeriesData, stepwise = FALSE)
```

### Comparing auto.arima() and ets() on seasonal data
Because the series is very long, you can afford to use a training and test set rather than time series cross-validation. This is much faster.
```r
# Use 20 years of the timeSeriesData beginning in 1988
train <- window(timeSeriesData, start = c(1988,1), end = c(2007,4))

# Fit an ARIMA and an ETS model to the training data
fit1 <- auto.arima(train)
fit2 <- ets(train)

# Check that both models have white noise residuals
checkresiduals(fit1)
checkresiduals(fit2)

# Produce forecasts for each model
fc1 <- forecast(fit1, h = 25)
fc2 <- forecast(fit2, h = 25)

# Use accuracy() to find better model based on RMSE
accuracy(fc1, timeSeriesData)
accuracy(fc2, timeSeriesData)
```

## 5. Advanced methods


