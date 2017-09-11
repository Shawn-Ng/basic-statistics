# Forecasting Using R

https://www.datacamp.com/courses/forecasting-using-r

Disclaimer: All credits belong to the **Rob J. Hyndman**


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
# same as, %>% piper operator
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


## 4. Forecasting with ARIMA models


## 5. Advanced meth2d5


