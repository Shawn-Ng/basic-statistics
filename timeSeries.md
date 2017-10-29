# Time Series Cheatsheet

1. Time series Graphics
    - Time plots
    - [Time series patterns](#time-series-patterns)
        1. Trend: Long-term increase or decrease in the data. It does not have to be linear
        2. Level: The level of a series refers to its height on the ordinate axis. A series with a trend will have a changing level, but a series whose level changes may not have a trend
        3. Seasonal: Always of a fixed and known period (quarters of the year, the month, day of the week, or time of day)
        4. Cyclic: Exists when there are rises and falls that are not of a fixed period, average length of cycles is longer than the length of a seasonal pattern
    - Seasonal plots
        - Makes it easier to spot seasonal patterns than normal TS plot
        - `seasonplot(data_ts)`
    - Seasonal subseries plots
        - Emphasises the pattherns within each season. Useful for identifying changes within particular seasons
        - `monthplot(data_ts)`
    - Lag plots
        - Checks whether a TS dataset is random
        - `lag.plot(data_ts, lags=INTEGER, layout=c(), diag=BOOLEAN)`

2. Fundamental Time Series Model
    - [White Noise](#white-noise)
        1. mean function
        2. autocovariance function
    - Random Walk
        - current step = drift + previous step + white noise
        - `y_t = c + y_(t-1) + e_t`
        - constant mean but not variance
    - ACFs
        - `acf(data_ts)`
        - Slow decrease in ACF as lags increase -> trend
        - Regular spikes -> seasonality

3. [Benchmark Forecasting methods](#benchmark-forecasting-methods)
    - Average method
        - Forecasts of all future values are equal to the mean of the historical data
        - `meanf(data_ts, h=VALUE)`
    - Naive method
        - Forecasts of all future values are equal to the most recent observation
        - `naive(data_ts, h=VALUE)`
        - `rwf(data_ts, h=VALUE, drift=T)`
        - rwf: random walk with drift
        - Naive method is potimal when data comes from random walk
    - Seasonal Naive Forecast
        - Forecast to be equal to the last observed value from the same season of the previous year
        - `snaive(data_ts, h=VALUE)`

4. Adjustments and Transformations
    - Calendar
        - Some of the the troughs are cause by different number of days in a month
        - To change from accumulated monthly production to daily production
        - `plot(data_ts/monthdays(data_ts))`
    - Population
        - It is better to track number of units per 1000 people than total number of units
    - Inflation
        - Data affected by value of money are best adjusted before modeling
        - price index = `item price in year x / item price in year x+1 * 100`
    - Transformations
        1. Logarithms: Changes in a log value are relative (percent) changes on the original scale
        2. Box-Cox: Stabilising variance
            - `data_tf <- BoxCox(data_ts, lambda=BoxCox.lambda(data_ts))`
            - `plot(data_tf)`
        3. Back-transforming Forecasts
        4. [Bias Adjustments](#bias-adjustments)

5. Residual Diagnostics
    - Obs value - forecast/fitted value
    - 1 step ahead forecast
    - Residual also known as training error
    - Residual values: `data_fc$residuals`
    - Fitted values: `data_fc$fitted`
    - Residual Properties
        1. Essential
            - Uncorrelated
            - 0 mean
        2. Useful
            - Constant variance
            - Normally distributed
    - Properties not met -> forecast method can be improved
    - Residual correlated -> ARIMA
    - Residual with non 0 mean -> add mean to all forecast
    - `checkresiduals(naive(data_fc),test=FALSE)`
    - Portmanteau Tests
        - Test of whether the first h autocorrelations, are significantly different from what would be expected from a white noise process
        - `res <- residuals(naive(data))`
        1. Box-Pierce test
            - `Box.test(res, lag=10, fitdf=0)`
        2. Ljung-Box test
            - p-value < 0.05 -> not white noise
            - `Box.test(res, lag=10, fitdf=0, type="Lj")`

6. Evaluating Forecast Accuracy
    - Training and Test Sets
        - Training data: Estimate the params of a model
        - Test data: Evaluate model's accuracy
        - Good fit to training data != model forecast well
        - Enough params -> perfect fit, but don't overfit
    - [Error Metrics](#error-metrics)
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
    - [Time Series Cross Validation](#time-series-cross-validation)
        - More sophiscated version of training/test

7. Prediction Intervals
    - 1 step ahead forecast -> forecast dis sd = residuals sd
    - No parameter estimated -> 2 sd identical
    - Parameter estimated -> forecast dis sd > residuals sd
    - Forecast h increase -> Prediction interval increase

8. Time Series Component
    - Time series = Seasonal component & trend-cycle component & remainder component
    1. Additive decomposition: If seasonal component doesn't change with level of time series
        - `$ y_t = S_t + T_t + R_t $`
    2. Multiplicative decompsition:
        - `$ y_t = S_t * T_t * R_t $`

9. Moving Average Filters
    - Estimate the trend-cycle
        1. Linear filter: `diff()`
        2. Moving average of order m (m-MA): averages nearby values
            - Smoothed series capture the main movement of the time series without all of the minor fluctuations
            - Higher order (m) -> smoother curve
            - 2x12-MA for monthly data and 7-MA for daily data
            - m is odd -> m-MA operation is symmetric
                - `ma(data_ts, order=VALUE)`
            - Centred moving average: m is even 
                - Symmetry absent, take 1 obs more from future than past
                - Centred moving average: `2 x m-MA`
                - `ma(data_ts, order=2, centre=FALSE)`
        3. Weighted moving averages: smoother estimates than simple moving averages
            - `x <- 1:10`
            - `y <- ts(x^2)`
            - `stats::filter(y, c(-6/70 , 24/70 , 17/35 , 24/70 , -6/70))`

10. Decomposition Algorithms
    1. Addictive decomposition
        - de-trended series: `y_t - \hatT_t`
        - Calculate the remainder component: `\hatR_t = y_t - \hatT_t - \hatS_t`
        - Constraint seasonal effects to sum to 0
    2. Multiplicative decomposition
        - de-trended series: `y_t / \hatT_t`
        - Calculate the remainder component: `\hatR_t = y_t / (\hatT_t * \hatS_t)`
        - Adjusted seasonal effects so they sum to m -> the average of the seasonal effects is 1;
        - `data_d <- decompose(data_ts, type="multiplicative")`
    3. X11-decomposition
        - `library(seasonal)`
        - `fit <- seas(data_ts, x11="")`
    4. Seasonal and trend (STL) decomposition using loess (locally weighted regression model)
        - Advantage: able to estimate non-linear relationships, handle any type of seasonality, can be made robust to outliers
        - Disadvantage: can only handle additive models (can overcome this by transforming the model first), several parameters
        - `fit <- stl(data_ts, t.window=VALUE, s.window="periodic", robust=BOOLEAN)`

11. Forecasting and Decomposition
    - `fit <- stl(data_ts, t.window=VALUE, s.window="periodic", robust=BOOLEAN)`
    - `fc <- forecast(fit, method="naive")`
    - `plot(fc)`


### Time series patterns
```r
# frequency=1,4,12,52 are annual, quarterly, monthly, weekly respectively
data_ts <- ts(data, start=c(YEAR, QUARTER), frequency=12)

# window(): extract subset of time series
subset <- window(data, start=c(YEAR, QUARTER), end=c(YEAR, QUARTER))

# time series plot
plot(data_ts)

# plotting changes in time series data with diff()
plot(diff(data_ts))
```


### White Noise
A white noise process is one with a mean zero and no correlation between its values at different times. Consists of **uncorrelated** random variables. The variables are not identically distributed

`$E(e_t)=0,  Var(e_t)=\sigma_e^2$`


### Benchmark Forecasting methods
```r
forecast1 <- meanf(data_ts, h=11)
forecast2 <- naive(data_ts, h=11)
forecast3 <- rwf(data_ts, h=11, drift=T)

plot(forecast1, PI=F)
lines(forecast2$mean, col=2)
lines(forecast3$mean, col=3)
legend('topright', col=c(4,2,3), legend=c('Average', 'Naive', 'Drift'))
```


## Bias Adjustments
```r
fc <- rwf(data_ts, drift=T, lambda=0, h=50)
fc2 <- rwf(data_ts, drift=T, lambda=0, h=50, biasadj=T)
plot(fc)
lines(fc2$mean)
```


## Error Metrics
```r
train <- window(data_ts, end=100)
forecast <- meanf(train, h=10)
test <- window(data_ts, start=101)
accuracy(forecast, test)
```


## Time Series Cross Validation
```r
e <- tsCV(data_ts, rwf, drift=TRUE, h=1)
sqrt(mean(e^2, na.rm=TRUE))
sqrt(mean(residuals(rwf(data_ts, drift=TRUE))^2, na.rm=TRUE))
```