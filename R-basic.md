# R summary

### 1. Arithmetic

```r
1 + 2 = 3
1 - 2 = -1
1 * 2 = 2
1 / 2 = 0.5
1 ^ 2 = 1
1 %% 2 = 1
```

### Descriptive Statistics

```r
mean(x)
median(x)

summary(x)
str(x)

max(x)
min(x)

var(x)
sd(x)
IQR(x)
quantile(x)

CV <- function(x) {
  sd(x)/mean(x)
}
CV(x)
```

### Central moment
```r
skew <- function(x) {
  n <- length(x)
  m2 <- mean((x-mean(x))^2)
  m3 <- mean((x-mean(x))^3)
  sk <- sqrt(n*(n-1))/(n-2) * m3/m2^(3/2)
  return(sk)
}

kurt <- function(x) {
  n <- length(x)
  m2 <- mean((x-mean(x))^2)
  m4 <- mean((x-mean(x))^4)
  kurt <- (n-1)/((n-2)*(n-3)) * ((n+1)*m4/m2^2 - 3*(n-1))
  return(kurt)
}
```

### Plot
```r
# Histogram
hist(x, freq=FALSE, main = paste("Histogram of return"), xlab = "return", ylab="frequency", axes = TRUE)

xpt <- seq(-10,100,0.1)
ypt <- dnorm(seq(-10,100,0.1), mean(x), sd(x))
lines(xpt,ypt)

hist(x, freq=TRUE, main = paste("Histogram of return"), xlab = "return", ylab="frequency", axes = TRUE)

# Normal curve imposed on the histogram
aypt <- ypt*length(x) *10

# length(x) *10 is the area of the histogram
lines(xpt,aypt)

# QQplot
qqnorm(x)
qqline(x)

# Stemplot
stem(x)

# Boxplot
boxplot(x)
```