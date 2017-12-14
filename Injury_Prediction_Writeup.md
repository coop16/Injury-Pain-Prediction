<table>
<thead>
<tr class="header">
<th><strong>Primary skills</strong></th>
<th><strong>Primary Programs</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Data visualization</td>
<td><a href="https://public.tableau.com/views/InjuryPrediction/Dashboard1?:embed=y&amp;:display_count=yes">Tableau</a></td>
</tr>
<tr class="even">
<td>Time series modeling</td>
<td>R</td>
</tr>
</tbody>
</table>

Motivation
==========

I have been an athlete my whole life, including playing collegiate
soccer at Whitman College. I have unfortunately suffered from recurring
hip injuries. In the summer of 2017 I played on Seattle's top men's club
ultimate frisbee team. During that time my injury acted up and I decided
to collect data on myself.

<img src="Injury_Prediction_Writeup_files/figure-markdown_strict/unnamed-chunk-1-1.png" alt="Playing soccer for Whitman College"  />
<p class="caption">
Playing soccer for Whitman College
</p>

Objectives
==========

1.  **Visualize and better understand how my activities influence pain
    levels**

2.  **Create a prediction tool for pain levels**

-   *To help me plan my training and participation in team events*

Data
====

I created a Google survey (filled out daily) documenting my daily pain
levels, as well as other factors that may influence my injuries (e.g.
participation in high intensity sports, lifting weights, physical
therapy exercises, hours sitting, hours spent looking at screens, sleep
the previous night, stress levels etc.).

Modeling
========

### Important features of the data considered

-   They are time series data from a single individual (me), with
    suspected autocorrelation structure
-   The sample size is small

### Model choices

Given the above features, I considered time series models, and looked at
ACF and PACF plots to help choose autocorrelation and moving average
parts. I used regression models with ARMA errors, in order to account
for autocorrelation, moving average, as well as adjust for covariates.
Because of the large risk of overfitting the data with such a small
sample size, I only considered a few different sets of adjustment
covariates. The final model (chosen based on the validation criteria
described below) only adjusted for participation in high intensity
sports.

### Validation

Since observations are from a time series I could not use typical
methods of validation such as k-fold cross-validation. Instead, I used a
"rolling origin" approach, which forecasts the *m*th observation with
the previous *m* − 1 observations, and repeats that for all
observations. Since models cannot be fit with only a few observations, a
minimum value of *m* was chosen and only the observations from then on
were actually predicted. The mean squared error of these single
predictions was then used for model validation.

Adressing my Goals
==================

1.  **Visualize and better understand how my activities influence pain
    levels**

Visual findings are presented in this interactive **[Tableau
Dashboard](https://public.tableau.com/views/InjuryPrediction/Dashboard1?:embed=y&:display_count=yes)**.

1.  **Create a prediction tool for pain levels**

In the dashboard I also implemented forecasting of the next day's pain
(based on the model descrived above) with input for whether I play
sports the next day. During the season, I used these forecasts (in
addition to other considerations) to help plan my training and
participation in sports for the following days.

-   *Note: I actually used a slightly different Tableau dashboard
    created on Tableau desktop, which allows for R scripts integrated
    into calculated fields. Therefore each night I could just refresh
    the connection to the data source and automatically have the
    new forecast. However, R integration is not available on Tableau
    Public and so I had to run forecasts in my local R server and export
    data to Tableau Public in order to create this publically
    available dashboard.*

<iframe align = "center" width = "1020" height = "850" src="https://public.tableau.com/views/InjuryPrediction/Dashboard1?:showVizHome=no&:embed=true"/>
### Daily Pain Levels and Forecasting (Top Plot)

-   **Pain levels typically increase from playing sports
    (no surprise!)**: seen by how the spikes in pain levels usually
    occur with participation in sports. *(Note: the spikes without
    playing sports on Sept. 6th and 20th both occured because of
    receiving a dry needling treatment)*

### Lag Plot (Bottom Plot)

-   **Interpreting the plot:** The lag plot on the bottom left of the
    dashboard shows the relationship between pain levels one
    day (y-axis) compared to the previous day (x-axis), with coloring
    based on sports participation in those two days. With this plot, if
    a point is below the diagonal line, that means that pain was worse
    the previous day and improves the second day. A point above the
    diagonal means pain increases the second day.
-   **Participation in sports typically makes pain worse and rest
    typically reduces pain (same conclusion as first plot)**. This is
    seen by how the when sports are played the second day (<span
    style="color:orange">orange</span> or <span
    style="color:red">red</span>), the points are almost always above
    the diagonal line, and when sports are not played (<span
    style="color:green">green</span> and <span
    style="color:blue">blue</span>) the points are typically below the
    diagnoal line.
-   **However, pain from playing sports may linger for one or more
    days**. This is seen by how the <span
    style="color:green">green</span> points (playing sports the first
    day but not the second), are more frequently above/closer to the
    diagonal line than the <span style="color:blue">blue</span> points
    (playing sports neither day).

Key Takeaways
=============

So how did I actually use this information?

The main insight it provided was that I should think about my activities
multiple days ahead of time and try and avoid high intensity sports
multiple days in a row if I can. This was far from ground breaking
insight, but that was expected given that I have suffered from similar
issues for years and already have extensive knowledge of the issue. More
than anything having actual data gave me support for what I know, and
kept me from ignoring those facts. On a day where I really wanted to
play, but knew I probably shouldn't for my health, I now had an
additional voice telling me to be cautious.

Annotated R Code for Time Series Modeling
=========================================

### Data Management

Because I created my own Google survey, the resulting data required
almost no cleaning.

    # packages for importing data, time series and table formatting
    library(readr)
    library(forecast)
    library(xtable)

    # load dataset
    setwd("C:/Users/Cooper/OneDrive/Documents/Data Science Injury Prediction Project")
    injury <- read_csv("InjurySurveyResponses_until_10_23_17.csv")

    # sample size
    n <- dim(injury)[1]

    # create time series object for pain levels
    ts <- ts(injury$AveragePainLevels)

### Descriptives

The plot below gives the same line plot I created in Tableau.

    # Pain History Plot
    plot(ts, ylab = "Pain Level", ylim = c(0, 10), xlab = "Day number", main = "Daily Pain History")

![](Injury_Prediction_Writeup_files/figure-markdown_strict/unnamed-chunk-4-1.png)

We now consider descriptive plots used to determine the time series
model specification. A plot of the Sample Autocorrelation Function (ACF)
and Partial Autocorrelation Function (PACF) are provided.

    # ACF and PACF plots
    library(astsa)
    acf2(ts)

![](Injury_Prediction_Writeup_files/figure-markdown_strict/unnamed-chunk-5-1.png)

The spike at the 1st lag in the ACF plot suggests the moving average
component of our time series model should be 1. The spikes at the 1st
and 2nd lags in the PACF plot suggest that the autoregressive component
should be 2.

### Adjustment Covariates

We consider three sets of adjustment covariates. The sets were chosen
based on my knowledge of what have seemed most influential to my pain
levels in the past. With such a small sample size I wanted to be
cautious of overfitting the data (even with cross-validation being used)
and so limited the number of adjustment covariates sets I tried.

    # Minimal adjustment (just whether played high intensity sports)
    x1 <- as.data.frame(cbind(sports = injury$HighIntensitySports))

    # Moderate adjustment (add tissue work, lower body lifting)
    x2 <- as.data.frame(cbind(injury$HighIntensitySports, injury$TissueWork, injury$LowerBodyLifting))

    # Most adjustment (add Stretching, PT exercises)
    x3 <- as.data.frame(cbind(injury$HighIntensitySports, injury$TissueWork, injury$LowerBodyLifting, 
        injury$Stretching, injury$PTexercises))

### Model Validation

I used a cross-validation method appropriate for time series: "forecast
evaluation with a rolling origin". Essentially I chose a minimum number
of observations to fit the model, *k*, then use those *k* observations
to predict the (*k* + 1)th observation, then use the first (*k* + 1)
observations to predict the (*k* + 2)th observation and so on. After all
points have been predicted (besides the first *k* observations of
course), we compare the predictions to the actual pain levels and
calculate the MSE of these predictions.

    # Framework for 'rolling origin' cross-validation
    h <- 1  #number of points to predict
    k <- 10  #minimum number of points used to fit model

    # matrix to fill in CV results
    cv.mat <- matrix(rep(NA, 4 * (n - k)), ncol = 4)

    # model unadjusted for covariates
    for (i in k:(n - 1)) {
        # define vector for first i days
        y.short <- ts[1:i]
        # fit model based on first i days
        fit.short <- arima(as.matrix(y.short), order = c(2, 0, 1), method = "ML")
        # predict (i+1)th day
        next.forecast <- forecast(object = fit.short, h = h)
        # record error between observed and predicted pain on (i+1)th day
        cv.mat[i - k + 1, 1] <- (next.forecast$mean - ts[i + 1])
    }


    # minimal adjustment
    for (i in k:(n - 1)) {
        # define vectors for first i days
        y.short <- ts[1:i]
        X.short <- x1[1:i, ]
        # fit model based on first i days
        fit.short <- arima(as.matrix(y.short), order = c(2, 0, 1), xreg = cbind(X.short), 
            method = "ML")
        # predict (i+1)th day
        next.forecast <- forecast(object = fit.short, h = h, xreg = x1[i + 1, ])
        # record error between observed and predicted pain on (i+1)th day
        cv.mat[i - k + 1, 2] <- (next.forecast$mean - ts[i + 1])
    }

    # medium adjustment
    for (i in k:(n - 1)) {
        # define vectors for first i days
        y.short <- ts[1:i]
        X.short <- x2[1:i, ]
        # fit model based on first i days
        fit.short <- arima(as.matrix(y.short), order = c(2, 0, 1), xreg = cbind(X.short), 
            method = "ML")
        next.forecast <- forecast(object = fit.short, h = h, xreg = x2[i + 1, ])
        # record error between observed and predicted pain on (i+1)th day
        cv.mat[i - k + 1, 3] <- (next.forecast$mean - ts[i + 1])
    }

    # most adjustment
    for (i in k:(n - 1)) {
        # define vectors for first i days
        y.short <- ts[1:i]
        X.short <- x3[1:i, ]
        # fit model based on first i days
        fit.short <- arima(as.matrix(y.short), order = c(2, 0, 1), xreg = cbind(X.short), 
            method = "ML")
        # predict (i+1)th day
        next.forecast <- forecast(object = fit.short, h = h, xreg = x3[i + 1, ])
        # record error between observed and predicted pain on (i+1)th day
        cv.mat[i - k + 1, 4] <- (next.forecast$mean - ts[i + 1])
    }

    # Cross validation MSE
    cv.mse <- apply(cv.mat^2, 2, mean)

    # Print table
    tab <- rbind(round(cv.mse, digits = 2))
    colnames(tab) <- c("unadjusted", "minimal adjustment", "medium adjustment", 
        "most adjustment")
    row.names(tab) <- c("CV MSE")
    xtab <- xtable(tab)
    print(xtab, type = "html")

<!-- html table generated in R 3.4.2 by xtable 1.8-2 package -->
<!-- Thu Dec 14 15:06:32 2017 -->
<table border="1">
<tr>
<th>
</th>
<th>
unadjusted
</th>
<th>
minimal adjustment
</th>
<th>
medium adjustment
</th>
<th>
most adjustment
</th>
</tr>
<tr>
<td align="right">
CV MSE
</td>
<td align="right">
3.33
</td>
<td align="right">
2.76
</td>
<td align="right">
3.54
</td>
<td align="right">
6.43
</td>
</tr>
</table>
The model only adjusted for high intensity sports performs best. With
such a small sample size, it is reasonable that we cannot adjust for
many variables without overfitting the data. We can look closer at the
output for this model:

    # final model
    final.fit <- arima(as.matrix(ts), order = c(2, 0, 1), xreg = x1)
    final.fit

    ## 
    ## Call:
    ## arima(x = as.matrix(ts), order = c(2, 0, 1), xreg = x1)
    ## 
    ## Coefficients:
    ##          ar1      ar2     ma1  intercept  sports
    ##       0.2842  -0.3117  0.3687     5.1168  1.1490
    ## s.e.  0.2360   0.1673  0.2355     0.2667  0.2809
    ## 
    ## sigma^2 estimated as 1.967:  log likelihood = -98.73,  aic = 209.47

With this model the estimated effect of playing sports is a 1.15 higher
pain level on average.

### Forecasting

 *"Prediction? ... Pain."* -Clubber Lang

Finally I forecast pain levels for the next day. I integrated similar
scripts in Tableau to automatically forecast the next day's pain with
the input of either playing sports or not. Multiple days could also be
predicted, but I would recomment only one or two days as the model does
not posess the complexity or training data to predict very far out.

    # if play sports the next day
    newx1 <- c(1)
    forecast(object = final.fit, h = 1, xreg = newx1)

    ##    Point Forecast    Lo 80    Hi 80    Lo 95    Hi 95
    ## 57       5.717597 3.920367 7.514827 2.968971 8.466223

    # if don't play sports the next day
    newx2 <- c(0)
    forecast(object = final.fit, h = 1, xreg = newx2)

    ##    Point Forecast    Lo 80    Hi 80    Lo 95    Hi 95
    ## 57       4.568568 2.771338 6.365798 1.819942 7.317194

We see that the predicted pain levels are higher if I play sports. The
uncertainty of the model is highlighted by the very wide confidence
intervals. We can also make a forecast plot similar to as is done in the
Tableau Dashboard. Here we show the forecast and confidence intervals
for if I played sports the next day.

    # plot forecast for if do play sports
    plot(forecast(object = final.fit, h = 1, xreg = newx1))

![](Injury_Prediction_Writeup_files/figure-markdown_strict/unnamed-chunk-10-1.png)
