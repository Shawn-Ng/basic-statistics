# Time Series Cheatsheet

1. Time series Graphics
    1. Time plots
    2. [Time series patterns](#time-series-patterns)
    3. Seasonal plots
        * Makes it easier to spot seasonal patterns than normal TS plot
        * `seasonplot(timeSeriesData)`
    4. Seasonal subseries plots
        * Emphasises the pattherns within each season. Useful for identifying changes within particular seasons
        * `monthplot(timeSeriesData)`
    5. lag plots
        * Checks whether a TS dataset is random
        * `lag.plot(timeSeriesData, lags=INTEGER, layout=c(), diag=BOOLEAN)`
2. Fundamental Time Series Model
    1. [White Noise](#white-noise)
        * mean function
        * autocovariance function
    2. Random Walk
        * current step = drift + previous step + white noise
        * `y_t = c + y_(t-1) + e_t`
        * constant mean but not variance
    3. ACFs
        * `acf(timeSeriesData)`
        * Slow decrease in ACF as lags increase -> trend
        * Regular spikes -> seasonality


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
