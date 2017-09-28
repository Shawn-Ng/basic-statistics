# Time Series Cheatsheet

1. Time series Graphics
    1. Time plots
    2. [Time series patterns](#time-series-patterns)
    3. Seasonal plots
        - Makes it easier to spot seasonal patterns than normal TS plot
        - `seasonplot(timeSeriesData)`
    4. Seasonal subseries plots
        - Emphasises the pattherns within each season. Useful for identifying changes within particular seasons
        - `monthplot(timeSeriesData)`
    5. lag plots
        - Checks whether a TS dataset is random
        - `lag.plot(timeSeriesData, lags=INTEGER, layout=c(), diag=BOOLEAN)`
2. Fundamental Time Series Model
    1. [White Noise](#white-noise)
        1. mean function
        2. autocovariance function
    2. Random Walk
        - current step = drift + previous step + white noise
        - `y_t = c + y_(t-1) + e_t`
        - constant mean but not variance
    3. ACFs
        - `acf(timeSeriesData)`
        - Slow decrease in ACF as lags increase -> trend
        - Regular spikes -> seasonality
3. [Benchmark Forecasting methods](#benchmark-forecasting-methods)
    1. Average method
        - Forecasts of all future values are equal to the mean of the historical data
        - `meanf(timeSeriesData, horizon)`
    2. Naive method
        - Forecasts of all future values are equal to the most recent observation
        - `naive(timeSeriesData, horizon)`
        - `rwf(timeSeriesData, horizon, drift=T)`
        - rwf: random walk with drift
        - Naive method is potimal when data comes from random walk
    3. Seasonal Naive Forecast
        - Forecast to be equal to the last observed value from the same season of the previous year
        - `snaive(timeSeriesData, horizon)`
4. Adjustments and Transformations
    1. Calendar
        - Some of the the troughs are cause by different number of days in a month
        - To change from accumulated monthly production to daily production
        - `plot(data/monthdays(data))`
    2. Population
        - It is better to track number of units per 1000 people than total number of units
    3. Inflation
        - Data affected by value of money are best adjusted before modeling
        - price index = `item price in year x / item price in year x+1 * 100`
    4. Transformations
        1. Logarithms: Changes in a log value are relative (percent) changes on the original scale
        2. Box-Cox: Stabilising variance
            - `tData <- BoxCox(timeSeriesData, lambda=BoxCox.lambda(timeSeriesData))`
            - `plot(tData)`
        3. Back-transforming Forecasts
        4. [Bias Adjustments](#bias-adjustments)
5. Residual Diagnostics
    - Obs value - forecast/fitted value
    - 1 step ahead forecast
    - Residual also known as training error
    - Residual values: `forecastData$residuals`
    - Fitted values: `forecastData$fitted`
    1. Residual Properties
        1. Essential
            - Uncorrelated
            - 0 mean
        2. Useful
            - Constant variance
            - Normally distributed
    - Properties not met -> forecast method can be improved
    - Residual correlated -> ARIMA
    - Residual with non 0 mean -> add mean to all forecast
    - `checkresiduals(naive(forecastData),test=FALSE)`
    2. Portmanteau Tests
        - Test of whether the first h autocorrelations, are significantly different from what would be expected from a white noise process
        - `res <- residuals(naive(data))`
        1. Box-Pierce test
            - `Box.test(res, lag=10, fitdf=0)`
        2. Ljung-Box test
            - p-value < 0.05 -> not white noise
            - `Box.test(res, lag=10, fitdf=0, type="Lj")`
6. Evaluating Forecast Accuracy
    1. Training and Test Sets
        - Training data: Estimate the params of a model
        - Test data: Evaluate model's accuracy
        - Good fit to training data != model forecast well
        - Enough params -> perfect fit, but don't overfit
    2. [Error Metrics](#error-metrics)
        - Forecast error: diff(obs value - forecast)
        - Scale dependent error
            1. Mean Absolute Error (MAE), low MAE -> optimal forecast of median
            2. Root Mean Square Error (RMSE), low RMSE -> optimal forecast of mean
        - Scale independent error
            1. Mean Absolute Percentage Error (MAPE)
                - If forecast ~ 0 -> extreme percentage error
                - Heavier penalty on -ve than +ve error
            2. Symmetric MAPE (sMAPE)
                - Range from -200% to 200%
                - Computationally unstable, could be negative
            3. Scaled Error
                - better than naive forecast -> <1
                - worse than naive forecast -> >1
    3. [Time Series Cross Validation](#time-series-cross-validation)
        - More sophiscated version of training/test
7. Prediction Intervals
    - 1 step ahead forecast -> forecast dis sd = residuals sd
    - No parameter estimated -> 2 sd identical
    - Parameter estimated -> forecast dis sd > residuals sd
    - Forecast horizon increase -> Prediction interval increase





### Time series patterns
```r
# frequency=1,4,12,52 are annual, quarterly, monthly, weekly respectively
timeSeriesData <- ts(data, start=YEAR, frequency=12)

# window(): extract subset of time series
subset <- window(data, start=YEAR, end=YEAR)

# time series plot
plot(timeSeriesData)

# plotting changes in time series data with diff()
plot(diff(timeSeriesData))
```


### White Noise
A white noise process is one with a mean zero and no correlation between its values at different times. Consists of **uncorrelated** random variables. The variables are not identically distributed

`$E(e_t)=0,  Var(e_t)=\sigma_e^2$`


### Benchmark Forecasting methods
```r
forecast1 <- meanf(timeSeriesData, h=11)
forecast2 <- naive(timeSeriesData, h=11)
forecast3 <- rwf(timeSeriesData, h=11, drift=T)

plot(forecast1, PI=F)
lines(forecast2$mean, col=2)
lines(forecast3$mean, col=3)
legend('topright', col=c(4,2,3), legend=c('Average', 'Naive', 'Drift'))
```


## Bias Adjustments
```r
fc <- rwf(timeSeriesData, drift=T, lambda=0, h=50)
fc2 <- rwf(timeSeriesData, drift=T, lambda=0, h=50, biasadj=T)
plot(fc)
lines(fc2$mean)
```


## Error Metrics
```r
train <- window(timeSeriesData, end=100)
forecast <- meanf(train, h=10)
test <- window(timeSeriesData, start=101)
accuracy(forecast, test)
```


## Time Series Cross Validation
```r
e <- tsCV(timeSeriesData, rwf, drift=TRUE, h=1)
sqrt(mean(e^2, na.rm=TRUE))
sqrt(mean(residuals(rwf(timeSeriesData, drift=TRUE))^2, na.rm=TRUE))
```