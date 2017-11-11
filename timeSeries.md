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

12. Exponential Smoothing
    - A forecasting method is an algorithm that provides a point forecast
    - A statistical model defines a process that generates the data
    - Use of a model allows us to compute prediction intervals
    - For exponential smoothing methods, the type of error does not make any difference - the point forecasts will be the same
    - 5 trend types, local level `l` local growth `b`:
        1. None: `T_h = l`
        2. Addictive: `T_h = l + bh`
        3. Addictive damped: `T_h = l + (\phi + \phi^2 + ... + \phi^h)b`
        4. Multiplicative: `T_h = l*b^h`
        5. Multiplicative damped: `T_h = l*b^(\phi + \phi^2 + ... + \phi^h)`
    1. Simple exponential smoothing (N, N)
        - The forecast is a weighted average of all past observations
        - `fc <- ses(data_ts, h=5, alpha=0.8, initial="simple")`
    2. Holt's linear method (A, N)
        - The forecast is a linear function of h
        - `fc <- holt(data_ts, h=5, alpha=0.8, beta=0, initial="simple")`
    3. Addictive damped method (A_d, N)
        - The damped method introduces a parameter that weakens the trend
        - Short term, forecasts have a trend, long term, they are constant
        - When damping parameter \phi = 1, it becomes Holt's linear method, \phi is rarely < 0.8
        - `fc <- holt(data_ts, h=15, damped=T, phi=0.9)`
    4. Additive Holt-Winters method (A, A)
        - Addictive method: seasonal variations roughly constant
        - `fc <- hw(data_ts, seasonal="addictive")`
    5. Multiplicative Holt-Winters method (A, M)
        - Multiplicative method: seasonal variations change proportionally to the level
        - `fc <- hw(data_ts, seasonal="multiplicative")`
    6. Exponential trend method (M, N)
    7. Multiplicative damped trend method (M_d, A)

13. State Space Models
    - For each forecast methods, there are 2 state space models: additive and multiplicative errors
    - Point forecasts will be same for both, but different intervals
    - `(Error, Trend, Seasonality)`
    - The models with multiplicative error/trend/seasonality are numerically unstable when the data values contain
zeros or negative values
    - The residuals are the estimates of the innovation errors. For the state space models with additive errors, these are identical to the one-step forecast errors. For models with multiplicative errors, these two quantities are not the same
    - `model <- ets(data_ts, "ETS")`
    - `RESIDUALS <- residuals(model)`
    - `fc_errors <- residuals(model, type="response")`
    - In any state space model, the initial state `x0` and the parameters are unknown
    - We need to estimate:
        - Smoothing parameters, `alpha, beta` for ETS model
        - Initial state: `x0`
        - Innovations variance: `sigma^2`
    - Some approaches to estimate parameters:
        - Likelihood function: `fit_mle <- ets(data_ts, "ANN")`
        - Log-likelihood function
        - Minimising the one-step MSE, MAE or some other error metric: `fit_mae <- ets(data_ts, "ANN", opt.crit="mae")`
        - Minimise the residual variance

14. Prediction Intervals
    - `fit1 <- ets(data_ts, "AAA")`
    - 95% interval: `fc1_aaa <- forecast(fit1, h=12, level-0.95)`
    - Simulation for 95% interval: `fc2_aaa <- forecast(fit1, h=12, level-0.95)`

15. Model Selection: to choose the best model
    - Potential problems:
        - Test set is too small to draw reliable conclusions
        - Diffcult to decide which error metric to use
    - Steps:
        - Split the time series into a training and test set
        - Fit each model using the training set (via MLE)
        - Assess the forecast accuracy of each model using the test set
        - Choose the model with the lowest forecast accuracy
        - Refit the chosen model to the full time series and use these new parameters for forecasting future observations
    - Or use cross validation to get the best model
    - Or use penalized likelihood method
        - The model with the highest likelihood is chosen as the best one. However, the likelihood is penalised for the number of parameters used. A model with more parameters is penalised more than one with fewer parameters
        - Akaike Information Criteria (AIC)
        - Bayes Information Criteria (BIC)
        - `fit <- ets(data_ts)`
        - `fc <- forecast(fit, h=12, level=90)`

16. Stationary
    - A weakly stationary time series is a finite variance process such that mean is a constant and `cov(y_s,y_t)` depends on `s` and `t` only through `s-t`
    - A stationary time series (e.g. WN) is (a) roughly horizontal (b) constant variance (c) no predictable pattern in long terms
    - Identify non-stationary series:
        - Timeplot shows mean and variance are not constant
        - ACF plots
            - Stationary series ACF drops to 0 quickly, while non-stationary series decreases slowly
    - Transformations: stabilize the variance of a time series
    - Differencing: stabilizes the mean of a time series by removing changes in the level of a time series
        - `plot(data_ts)`
        - `acf(data_ts)`
        - `plot(diff(data_ts))`
        - `acf(diff(data_ts))`
    - Seasonal difference is the difference between an observation and the corresponding observation from the previous year
        - `data_log <- log(data_ts)`
        - `data_logdiff <- diff(data_log, 12)`
        - `plot(data_logdiff, main="Differenced Log - Transform")`
    - Unit root testing: determine the required order of differencing
        1. Augmented Dickey Fuller (ADF) test: H0 is that the data are non-stationary and non-seasonal
            - small p-value -> stationary
            - `library(tseries)`
            - `adf.test(data_ts)`
        2. [Kwiatkowski-Phillips-Schmidt-Shin (KPSS) test](#kwiatkowski-phillips-schmidt-shin-kpss-test): H0 is that the data are stationary and non-seasonal
            - small p-value -> non-stationary
    - Backward shift operator: describe the process of differencing

17. ARIMA model
    - Autoregressive AR(p) model
        - Multiple linear regression with lagged values of `y_t` as predictors, `e_t` is white noise and its variance will only change the scale of the series, not the patterns
    - Moving average MA(q) model
        - Each value of `y_t` as a weighted moving average of the past few forecast errors, `e_t` is white noise and its variance will only change the scale of the series, not the patterns
        - MA(q) VS m-MA smoothing
            - m-MA smoothing is used for estimating the trend-cycle of past values. It is a linear filter
            - MA(q) model is used for forecasting future values
    - Autoregressive moving average (ARMA) model: combination of AR(p) and MA(q) models
        - Predictors: lagged values `y_t` and lagged errors
    - ARIMA (p,q,d) model = ARMA combine with differencing
        - `fit <- auto.arima(uschange[,1], max.P=0, max.Q=0, D=0)`
        - `plot(forecast(fit,h=10), include=80)`
        - Higher the value of d, the more rapidly the forecast intervals increase in size
        - d = 0, long term forecast sd goes to sd
    - Partial autocorrelation function (PACF): meaures relationship between `y_t` and `y_t-k` when time lag effects removed
        - `tsdisplay(data_ts[,1])`
        - `auto.arima(data_ts[,1],seasonal=FALSE, stepwise=TRUE, approximation=FALSE)`


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


### Kwiatkowski-Phillips-Schmidt-Shin (KPSS) test
```r
ns <- nsdiffs (x) # Check seasonal and apply
if(ns > 0)
    xstar <- diff (x,lag= frequency (x), differences =ns)
else
    xstar <- x
nd <- ndiffs ( xstar ) # Check first order and apply
if(nd > 0)
    xstar <- diff (xstar , differences =nd)
```