Lilo Heinrich, Tim Novak, and Colin Takeda
12-14-2020

  - [Data Background](#data-background)
  - [Investigation Question](#investigation-question)
  - [Data Tidying](#data-tidying)
  - [Exploratory Data Analysis](#exploratory-data-analysis)
  - [Final Position](#final-position)
  - [Conclusion](#conclusion)
  - [Rubrics](#rubrics)

<center>

<h1>

Formula 1 Racing: <br /> Does the car or the driver have the greater
impact?

</h1>

</center>

## Data Background

#### Context

The Formula 1 World Championship has been one of the premier forms of
auto racing around the world since its inaugural season in 1950. The
word “formula” refers to the set of rules to which all participants’
cars must conform. A Formula 1 season consists of a series of Grands
Prix races which take place worldwide on circuits and closed public
roads.

#### Source

The [Formula 1 World
Championships](https://www.kaggle.com/rohanrao/formula-1-world-championship-1950-2020)
dataset consists of all information on the Formula 1 races, drivers,
constructors, qualifying, circuits, lap times, pit stops, and
championships for every season from 1950 to 2020.

This dataset was published by Rohan Rao, a Data Scientist who goes by
the name [Vopani](https://www.kaggle.com/rohanrao) on Kaggle. (Fun fact:
he is also the reigning National Sudoku Champion of India.) He compiled
this dataset using the [Ergast Developer API](http://ergast.com/mrd/),
an experimental web service that provides a historical record of motor
racing data for non-commercial purposes. The API provides data
specifically for the Formula One series, from the beginning of the world
championships in 1950 up to the present season.

The Ergast API website does not give any details about their data
collection procedures, but their data is very thorough and complete.
Additionally, Formula 1 world championships are internationally
televised and race results are publicly accessible information [(see
official Formula 1 website)](https://www.formula1.com/en/results.html),
so there is no reason to doubt the accuracy of the data reported.

-----

## Investigation Question

#### Does the car or the driver have the greater impact?

Unlike most sports, Formula 1 racing is heavily reliant on the
performance capability of their equipment, which is their cars that are
created by the constructors. To what extent do the constructor and
driver predict race performance, and which one has the greater
predictive capability?

-----

## Data Tidying

The data came in many separate files, including `results.csv`,
`drivers.csv`, `constructors.csv`, `circuits.csv`, `races.csv`,
`status.csv`, and `laptimes.csv`. Our first step was to join all of
these data frames together by their relevant ID numbers and replace the
missing values with `NA`.

In our dataset we kept these columns:

  - resultId
  - raceId
  - driverId
  - constructorId
  - positionOrder
  - laps
  - fastestLapSpeed
  - statusId
  - driver\_name
  - constructor\_name
  - year
  - round
  - circuitId
  - race\_name
  - status
  - circuit\_name

#### Time data

The column `milliseconds` reports the race completion time for all of
the drivers that were able to finish the race. However, drivers are
often unable to complete the full race due to collisions, car
breakdowns, or other problems, leaving many missing values for total
race time. We discarded `milliseconds` and created a second dataset
where we supplemented information from `laptimes.csv` to compute total
race time even when race status was not finished. The lap times were
understandably missing when the number of laps completed was 0, so we
removed these observations. Overall, we still only have lap times for
9,233 of our 24,900 though.

In this time-filtered dataset we added these columns:

  - total\_time
  - avg\_lap
  - circuit\_avg\_lap
  - circuit\_lap\_sd
  - std\_avg\_lap

#### Potential Problems

  - Some examples of [constructor name
    changes](https://www.reddit.com/r/formula1/comments/1dos3r/i_made_a_diagram_to_show_how_current_f1_teams/)

  - Rules change year to year and can significantly affect performance

  - New drivers are skewed because they don’t have as many data points
    yet

  - This data is for 70 years of races and [formula racing has changed a
    lot](https://youtu.be/hgLQWIAaCmY)
    
      - Consider filtering on year \>= 2000

-----

## Exploratory Data Analysis

#### Standardized Average Lap Time

![](Final_Report_files/figure-gfm/finish%20race-1.png)<!-- -->

As shown in the graph above, final position is heavily reliant on how
many laps the driver was able to complete. And almost a third of the
time, collisions or car troubles put drivers out of commission before
the race is finished. These final standings are not reflective of how
well the driver was doing before that point, so we decided to use
average lap time to create a more comprehensive performance metric.

Average lap time is a comprehensive measure of how well a driver
performed in a race because it is informed only on the laps they were
able to complete, unlike final position. The only problem is that it
doesn’t account for the effect of circuit on lap time, so we need to
correct for this difference across circuits.

![](Final_Report_files/figure-gfm/average%20lap%20time%20per%20circuit-1.png)<!-- -->

The average lap time varies across circuits due to the differences in
track length and shape, so we need a way to compare average lap time
across circuits. Dividing average lap time by average lap time per
circuit doesn’t work because it doesn’t account for the range of average
lap time on each circuit. To correct for the impact of circuit on
average lap time, we standardized by circuit:

\[\mu = \sum_{i}^{n} \frac{x_i}{n}\]

\[\sigma = \sqrt{\sum_{i}^{n} \frac{(x_i - \mu)^2}{n}}\]

\[z = \frac{x-\mu}{\sigma}\]

where \(x\) is the data, \(\mu\) is the mean, \(\sigma\) is the standard
deviation, and \(z\) is the standard score of \(x\)

#### Standardized Average Lap Time by Circuit

![](Final_Report_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

The graph above shows a visual comparison of standardized average lap
time by circuit. The slope of the median is small in magnitude relative
to the range of the standardized average lap time, showing that
standardizing the average lap time successfully minimizes the effect of
circuit.

#### Modeling by Standard Average Lap Time

First, let’s model the a subset of the data that has completed times to
get a sense of how informative a linear model of **standarderized
average lap time** is for our dataset, solely based upon driver,
constructor, and a combination of the two.

``` r
f_driv_sal <-
  df_timedata %>%
  lm(
    data = .,
    formula = std_avg_lap ~ as.factor(driverId)
  )

f_cons_sal <- 
  df_timedata %>% 
  lm(
    data = .,
    formula = std_avg_lap ~ as.factor(constructorId)
  )

f_drivcons_sal <- 
  df_timedata %>% 
  lm(
    data = .,
    formula = std_avg_lap ~ as.factor(driverId) + as.factor(constructorId)
  )
```

    ## Subset Fit - Just Driver

    ##   Rsquare 0.05762464

    ##   MSE 1.133674

    ## Subset Fit - Just Constructor

    ##   Rsquare 0.04821027

    ##   MSE 1.145

    ## Subset Fit - Driver and Constructor

    ##   Rsquare 0.08991716

    ##   MSE 1.094826

Starting with looking at the mean square error (MSE) we can compare the
different fits against one another. Between fits, the error is lowest
with both *driver and constructor*. The “goodness of fit” is best with
both factors involved, which may imply that both are informative towards
standard average lap time. However, the difference is quite small
between MSEs, so the predictive capabilities of both still seem minute.
The order of best to worst fit, solely based upon MSE, is **driver and
constructor, just driver, and just constructor.** These results may
imply that driver is a better predictor of outcome than constructor, but
this is not necessarily the case.

Looking next at our R-square value we see that our models *does not*
encapsulate much of the variance of the data. We see the fraction of the
variance of the data ranges from 4.8% to 8.9%. The order of best to
worst fit, solely based upon R-square, is **driver and constructor, just
driver, and finally just constructor.**

These models should be taken with a grain or more of salt as we are
using the entire subset of data set to create them, so they are
extremely optimistic with the fit and do not cover all observations
available. Also, as we see from our R-square values, using driver and/or
constructor does not seem very fruitful for modeling standardized
average lap time. While this was a useful metric for comparing across
circuits, it does not seem to be as useful for modeling. Instead, we
should explore other variables to indicate performance.

## Final Position

#### Modelling by Final Position

Next, we will model the entire data set

``` r
df_data_with_rows <- tibble::rowid_to_column(df_data, "ID")

df_validate_pos <-
  df_data_with_rows %>%
  group_by(driverId, constructorId) %>% 
  slice_sample(prop = 0.5) %>% 
  ungroup()

df_train_pos <-
  anti_join(
    df_data_with_rows,
    df_validate_pos,
    by = "ID"
  )

df_train_pos
```

    ## # A tibble: 1,405 x 17
    ##       ID resultId raceId driverId constructorId positionOrder  laps
    ##    <int>    <dbl>  <dbl>    <dbl>         <dbl>         <dbl> <dbl>
    ##  1     2    22131    900      825             1             2    57
    ##  2     3    22132    900       18             1             3    57
    ##  3     5    22134    900      822             3             5    57
    ##  4     6    22135    900      807            10             6    57
    ##  5     7    22136    900        8             6             7    57
    ##  6     8    22137    900      818             5             8    57
    ##  7     9    22138    900      826             5             9    57
    ##  8    10    22139    900      815            10            10    57
    ##  9    11    22140    900       16            15            11    56
    ## 10    13    22142    900      820           206            13    55
    ## # … with 1,395 more rows, and 10 more variables: fastestLapSpeed <dbl>,
    ## #   statusId <dbl>, driver_name <chr>, constructor_name <chr>, year <dbl>,
    ## #   round <dbl>, circuitId <dbl>, race_name <chr>, status <chr>,
    ## #   circuit_name <chr>

``` r
df_validate_pos
```

    ## # A tibble: 1,362 x 17
    ##       ID resultId raceId driverId constructorId positionOrder  laps
    ##    <int>    <dbl>  <dbl>    <dbl>         <dbl>         <dbl> <dbl>
    ##  1   727    22858    943        1           131             2    71
    ##  2  1848    23983    999        1           131             1    67
    ##  3  1596    23730    986        1           131             9    70
    ##  4  1289    23420    971        1           131             2    57
    ##  5  2528    24666   1033        1           131             1    70
    ##  6   962    23093    956        1           131             1    71
    ##  7   526    22656    932        1           131             1    70
    ##  8    23    22152    901        1           131             1    56
    ##  9  2469    24606   1030        1           131             1    55
    ## 10  1140    23271    964        1           131             3    53
    ## # … with 1,352 more rows, and 10 more variables: fastestLapSpeed <dbl>,
    ## #   statusId <dbl>, driver_name <chr>, constructor_name <chr>, year <dbl>,
    ## #   round <dbl>, circuitId <dbl>, race_name <chr>, status <chr>,
    ## #   circuit_name <chr>

``` r
f_driv_pos <-
  df_train_pos %>%
  lm(
    data = .,
    formula = positionOrder ~ as.factor(driverId)
  )

f_cons_pos <- 
  df_train_pos %>% 
  lm(
    data = .,
    formula = positionOrder ~ as.factor(constructorId)
  )

f_drivcons_pos <- 
  df_train_pos %>% 
  lm(
    data = .,
    formula = positionOrder ~ as.factor(driverId) + as.factor(constructorId)
  )
```

    ## Train Fit - Just Driver

    ##   Rsquare 0.3722665

    ##   MSE 21.53272

    ## Train Fit - Just Constructor

    ##   Rsquare 0.3899165

    ##   MSE 20.9296

    ## Train Fit - Driver and Constructor

    ##   Rsquare 0.4102879

    ##   MSE 20.22789

#### Prediction Interval

    ## Warning in predict.lm(model, data): prediction from a rank-deficient fit may be
    ## misleading

    ## # A tibble: 1,362 x 21
    ##       ID resultId raceId driverId constructorId positionOrder  laps
    ##    <int>    <dbl>  <dbl>    <dbl>         <dbl>         <dbl> <dbl>
    ##  1   727    22858    943        1           131             2    71
    ##  2  1848    23983    999        1           131             1    67
    ##  3  1596    23730    986        1           131             9    70
    ##  4  1289    23420    971        1           131             2    57
    ##  5  2528    24666   1033        1           131             1    70
    ##  6   962    23093    956        1           131             1    71
    ##  7   526    22656    932        1           131             1    70
    ##  8    23    22152    901        1           131             1    56
    ##  9  2469    24606   1030        1           131             1    55
    ## 10  1140    23271    964        1           131             3    53
    ## # … with 1,352 more rows, and 14 more variables: fastestLapSpeed <dbl>,
    ## #   statusId <dbl>, driver_name <chr>, constructor_name <chr>, year <dbl>,
    ## #   round <dbl>, circuitId <dbl>, race_name <chr>, status <chr>,
    ## #   circuit_name <chr>, pred <dbl>, pred_driv <dbl>, pred_cons <dbl>,
    ## #   pred_drivcons <dbl>

![](Final_Report_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

    ## # A tibble: 1,405 x 20
    ##       ID resultId raceId driverId constructorId positionOrder  laps
    ##    <int>    <dbl>  <dbl>    <dbl>         <dbl>         <dbl> <dbl>
    ##  1     2    22131    900      825             1             2    57
    ##  2     3    22132    900       18             1             3    57
    ##  3     5    22134    900      822             3             5    57
    ##  4     6    22135    900      807            10             6    57
    ##  5     7    22136    900        8             6             7    57
    ##  6     8    22137    900      818             5             8    57
    ##  7     9    22138    900      826             5             9    57
    ##  8    10    22139    900      815            10            10    57
    ##  9    11    22140    900       16            15            11    56
    ## 10    13    22142    900      820           206            13    55
    ## # … with 1,395 more rows, and 13 more variables: fastestLapSpeed <dbl>,
    ## #   statusId <dbl>, driver_name <chr>, constructor_name <chr>, year <dbl>,
    ## #   round <dbl>, circuitId <dbl>, race_name <chr>, status <chr>,
    ## #   circuit_name <chr>, pi_fit <dbl>, pi_lwr <dbl>, pi_upr <dbl>

![](Final_Report_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

If possible given your data, report all estimates with confidence /
prediction / tolerance intervals. If not possible, clearly explain why
it is not possible to provide intervals and document what sources of
uncertainty are not quantified.

-----

## Conclusion

testing testing testing

-----

## Rubrics

Questions to answer:

  - What question did you set out to answer?
  - What data did you find to help answer that question?
  - What is the relevant background on your question?
  - What level of (quantified) certainty do you have in your results?
  - What conclusions did you come to?
  - What questions do you have remaining?
  - Make sure your report contains at least one presentation-quality
    figure

Observed:

  - (The usual stuff)
  - Must provide background
  - Must posit a question

Supported:

  - (The usual stuff)
  - Some analysis must support answering question

Assessed:

  - (The usual stuff)
  - All estimates must be provided with some quantification of
    uncertainty (e.g. confidence / prediction / tolerance intervals), OR
    a justification for why producing an interval is not possible and
    documentation for sources of uncertainty not accounted for.

Styled:

  - (The usual stuff)
  - Report must contain at least one presentation-quality figure

> > > > > > > 0186163de513b72013cb76d44c3cda1f8b393891
