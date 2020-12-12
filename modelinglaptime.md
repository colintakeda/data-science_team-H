Modeling Avg Lap Time for Racers and Constructors
================

  - [EDA](#eda)
  - [Historgrams](#historgrams)
      - [Modeling](#modeling)
  - [Creating data frames](#creating-data-frames)
  - [Visualizing Std Average Lap
    Time](#visualizing-std-average-lap-time)
  - [Creating Model](#creating-model)

### EDA

## Historgrams

``` r
df_avglaptime %>% 
  filter(avg_lap <= 200000) %>% 
  ggplot(aes(avg_lap)) + 
  geom_histogram(bins = 50) +
  labs(
    title = "Average Lap Time Histogram", 
    x = "Average Lap Time (ms)"
  ) + 
  theme_minimal()
```

![](modelinglaptime_files/figure-gfm/variable%20histgrams-1.png)<!-- -->

``` r
df_avglaptime %>% 
  ggplot(aes(circuit_avg_lap)) + 
  geom_histogram(bins = 60) + 
  labs(
    title = "Circuit Average Lap Time Histogram", 
    x = "Average Circuit Lap Time (ms)"
  ) + 
  theme_minimal()
```

![](modelinglaptime_files/figure-gfm/variable%20histgrams-2.png)<!-- -->

``` r
df_avglaptime %>% 
  mutate(
    year = as.factor(year), 
    circuit_name = fct_reorder(circuit_name, circuit_avg_lap)) %>% 
  ggplot(aes(circuit_name, circuit_avg_lap)) + 
  geom_jitter(aes(color = year), alpha = 1/14) +
  coord_flip() + 
  labs(title = "Circuit Average Lap for GPs containing Drivers Who Have Driven For Multiple Teams") +
  theme_minimal() +
  theme(plot.title = element_text(size = 9))
```

![](modelinglaptime_files/figure-gfm/circuit%20average%20lap-1.png)<!-- -->
***Observations***

  - Same Grands Prix can have vastly different times, most likely due to
    different courses under the same GP name
      - See United States GP and French GP
  - Also the circuit average lap time seems to diverge onto one time and
    doesn’t seem to diverge much
  - `SO MUCH data` that it is hard to visualize the number of
    observations for a single GP
      - What is the best way to visualize these observations??

### Modeling

## Creating data frames

## Visualizing Std Average Lap Time

``` r
df_stdavglap %>% 
  ggplot(aes(std_avg_lap)) +
  geom_histogram(bins = 50)
```

![](modelinglaptime_files/figure-gfm/unnamed-chunk-1-1.png)<!-- -->

## Creating Model

``` r
fullfit_driver <-
  df_stdavglap %>%
  lm(
    data = .,
    formula = std_avg_lap ~ as.factor(driverId)
  )

print("Full Fit - Just Driver")
```

    ## [1] "Full Fit - Just Driver"

``` r
cat("Rsquare", rsquare(fullfit_driver, df_stdavglap), "\n")
```

    ## Rsquare 0.05748965

``` r
cat("MSE", mse(fullfit_driver, df_stdavglap), "\n")
```

    ## MSE 0.9387334

``` r
fullfit_constructor <- 
  df_stdavglap %>% 
  lm(
    data = .,
    formula = std_avg_lap ~ as.factor(constructorId)
  )

print("Full Fit - Just Constructor")
```

    ## [1] "Full Fit - Just Constructor"

``` r
cat("Rsquare", rsquare(fullfit_constructor, df_stdavglap), "\n")
```

    ## Rsquare 0.04633339

``` r
cat("MSE", mse(fullfit_constructor, df_stdavglap), "\n")
```

    ## MSE 0.9498449

``` r
fullfit_driver_constructor <- 
  df_stdavglap %>% 
  lm(
    data = .,
    formula = std_avg_lap ~ as.factor(driverId) + as.factor(constructorId)
  )

print("Full Fit - Driver and Constructor")
```

    ## [1] "Full Fit - Driver and Constructor"

``` r
cat("Rsquare", rsquare(fullfit_driver_constructor, df_stdavglap), "\n")
```

    ## Rsquare 0.08119187

``` r
cat("MSE", mse(fullfit_driver_constructor, df_stdavglap), "\n")
```

    ## MSE 0.9151261

***Observations***

  - Looking at our mean square error first we can compare our different
    models against one another
      - Between models, the error is lowest with both `driver and
        constructor` which is a good sign that both are informative
        towards lap time
  - The order of best model to worst, solely based upon MSE, is driver &
    constructor, just driver, and finally just constructor
      - These results may imply that driver is a better predictor of
        outcome than constructor, but I will argue why not next
  - Looking next at our rsquared value we see that our models do **not**
    have very good coverage of the data
  - We are looking at each model only covering around `5.7% to 8.1%` of
    the data, which calls into validity the comparisons made for our MSE
  - The order of best model to worst, solely based upon rsquare, is
    driver & constructor, just driver, and finally just constructor
  - So overall, the overall standings seem to follow the exact same
    trend seen in both MSE and rsquare
  - These models should be taken with a grain or more of salt as we are
    using the entire data set to create these linear models
  - Next, we’ll see how constructor and driver stacks up against a
    training and validation data set

<!-- end list -->

``` r
#Setting seed for temporary repeatability
#set.seed("101")

#Getting number of observations to feed into n for df_train
number_obs <- df_stdavglap %>% 
  group_by(driverId, circuitId) %>% 
  summarize(n = n()) %>% 
  pull(n)
```

    ## `summarise()` regrouping output by 'driverId' (override with `.groups` argument)

``` r
#Sampling by 
df_train <-
  df_stdavglap %>%
  group_by(driverId, circuitId) %>% 
  slice_sample(n = 2) %>% 
  ungroup()

df_validate <-
  anti_join(
    df_stdavglap,
    df_train,
    by = "ID"
  )

df_train %>% arrange(ID)
```

    ## # A tibble: 4,210 x 22
    ##       ID resultId raceId driverId constructorId positionOrder  laps
    ##    <int>    <dbl>  <dbl>    <dbl>         <dbl>         <dbl> <dbl>
    ##  1     2        2     18        2             2             2    58
    ##  2     6        6     18        6             3             6    57
    ##  3     7        7     18        7             5             7    55
    ##  4     9        9     18        9             2             9    47
    ##  5    10       10     18       10             7            10    43
    ##  6    11       11     18       11             8            11    32
    ##  7    12       12     18       12             4            12    30
    ##  8    14       14     18       14             9            14    25
    ##  9    16       16     18       16            10            16     8
    ## 10    19       24     19        9             2             2    56
    ## # … with 4,200 more rows, and 15 more variables: fastestLapSpeed <dbl>,
    ## #   statusId <dbl>, driver_name <chr>, constructor_name <chr>, year <dbl>,
    ## #   round <dbl>, circuitId <dbl>, race_name <chr>, status <chr>,
    ## #   circuit_name <chr>, total_time <dbl>, avg_lap <dbl>, circuit_avg_lap <dbl>,
    ## #   circuit_lap_sd <dbl>, std_avg_lap <dbl>

``` r
df_validate %>% arrange(ID)
```

    ## # A tibble: 5,023 x 22
    ##       ID resultId raceId driverId constructorId positionOrder  laps
    ##    <int>    <dbl>  <dbl>    <dbl>         <dbl>         <dbl> <dbl>
    ##  1     1        1     18        1             1             1    58
    ##  2     3        3     18        3             3             3    58
    ##  3     4        4     18        4             4             4    58
    ##  4     5        5     18        5             1             5    58
    ##  5     8        8     18        8             6             8    53
    ##  6    13       13     18       13             6            13    29
    ##  7    15       15     18       15             7            15    19
    ##  8    17       22     18       22            11            22    58
    ##  9    18       23     19        8             6             1    56
    ## 10    20       25     19        5             1             3    56
    ## # … with 5,013 more rows, and 15 more variables: fastestLapSpeed <dbl>,
    ## #   statusId <dbl>, driver_name <chr>, constructor_name <chr>, year <dbl>,
    ## #   round <dbl>, circuitId <dbl>, race_name <chr>, status <chr>,
    ## #   circuit_name <chr>, total_time <dbl>, avg_lap <dbl>, circuit_avg_lap <dbl>,
    ## #   circuit_lap_sd <dbl>, std_avg_lap <dbl>

``` r
trainfit_driver <-
  df_train %>%
  lm(
    data = .,
    formula = std_avg_lap ~ as.factor(driverId)
  )

print("Train Fit - Just Driver")
```

    ## [1] "Train Fit - Just Driver"

``` r
cat("Rsquare", rsquare(trainfit_driver, df_validate), "\n")
```

    ## Rsquare 0.00199221

``` r
cat("MSE", mse(trainfit_driver, df_validate), "\n")
```

    ## MSE 0.9838992

``` r
trainfit_constructor <- 
  df_train %>% 
  lm(
    data = .,
    formula = std_avg_lap ~ as.factor(constructorId)
  )

print("Train Fit - Just Constructor")
```

    ## [1] "Train Fit - Just Constructor"

``` r
cat("Rsquare", rsquare(trainfit_constructor, df_validate), "\n")
```

    ## Rsquare 0.01529398

``` r
cat("MSE", mse(trainfit_constructor, df_validate), "\n")
```

    ## MSE 0.9683658

``` r
trainfit_driver_constructor <- 
  df_train %>% 
  lm(
    data = .,
    formula = std_avg_lap ~ as.factor(driverId) + as.factor(constructorId)
  )

print("Train Fit - Driver and Constructor")
```

    ## [1] "Train Fit - Driver and Constructor"

``` r
cat("Rsquare", rsquare(trainfit_driver_constructor, df_validate), "\n")
```

    ## Rsquare 0.01700952

``` r
cat("MSE", mse(trainfit_driver_constructor, df_validate), "\n")
```

    ## MSE 0.970778

***Observations***

``` r
driver_levels <- df_stdavglap %>% 
  mutate(driverId = as.factor(driverId)) %>% 
  pull(driverId) %>% 
  levels()
  
driver_levels
```

    ##   [1] "1"   "2"   "3"   "4"   "5"   "6"   "7"   "8"   "9"   "10"  "11"  "12" 
    ##  [13] "13"  "14"  "15"  "16"  "17"  "18"  "19"  "20"  "21"  "22"  "23"  "24" 
    ##  [25] "25"  "26"  "27"  "28"  "29"  "30"  "31"  "32"  "33"  "34"  "35"  "36" 
    ##  [37] "37"  "38"  "39"  "40"  "41"  "42"  "43"  "44"  "45"  "46"  "47"  "48" 
    ##  [49] "49"  "50"  "51"  "52"  "53"  "54"  "55"  "56"  "57"  "58"  "59"  "60" 
    ##  [61] "61"  "62"  "63"  "64"  "65"  "66"  "67"  "68"  "69"  "70"  "71"  "72" 
    ##  [73] "73"  "74"  "75"  "76"  "77"  "78"  "79"  "81"  "82"  "83"  "84"  "85" 
    ##  [85] "86"  "153" "154" "155" "807" "808" "810" "811" "812" "813" "814" "815"
    ##  [97] "816" "817" "818" "819" "820" "821" "822" "823" "824" "825" "826" "827"
    ## [109] "828" "829" "830" "831" "832" "833" "834" "835" "836" "837" "838" "839"
    ## [121] "840" "841" "842" "843" "844" "845" "846" "847" "848" "849"

``` r
df_stdavglap
```

    ## # A tibble: 9,233 x 22
    ##       ID resultId raceId driverId constructorId positionOrder  laps
    ##    <int>    <dbl>  <dbl>    <dbl>         <dbl>         <dbl> <dbl>
    ##  1     1        1     18        1             1             1    58
    ##  2     2        2     18        2             2             2    58
    ##  3     3        3     18        3             3             3    58
    ##  4     4        4     18        4             4             4    58
    ##  5     5        5     18        5             1             5    58
    ##  6     6        6     18        6             3             6    57
    ##  7     7        7     18        7             5             7    55
    ##  8     8        8     18        8             6             8    53
    ##  9     9        9     18        9             2             9    47
    ## 10    10       10     18       10             7            10    43
    ## # … with 9,223 more rows, and 15 more variables: fastestLapSpeed <dbl>,
    ## #   statusId <dbl>, driver_name <chr>, constructor_name <chr>, year <dbl>,
    ## #   round <dbl>, circuitId <dbl>, race_name <chr>, status <chr>,
    ## #   circuit_name <chr>, total_time <dbl>, avg_lap <dbl>, circuit_avg_lap <dbl>,
    ## #   circuit_lap_sd <dbl>, std_avg_lap <dbl>

``` r
df_test <- 
  df_stdavglap %>% 
  select(driverId, constructorId) %>% 
  mutate(driverId = as.factor(driverId),
         constructorId = as.factor(constructorId))

tibble(df_test) %>% 
  add_predictions(trainfit_driver, var = "sal_pred-d") %>%
  add_predictions(trainfit_constructor, var = "sal_pred-c") %>%
  add_predictions(trainfit_constructor, var = "sal_pred-d+c") %>%  
  pivot_longer(
    names_to = c(".value", "model"),
    names_sep = "-",
    cols = matches("sal")
  ) %>% 
  ggplot(aes(driverId, sal_pred, color = model)) +
  geom_line(aes(group = model), alpha = .5) +
  geom_point(alpha = .5) +
  theme_minimal() +
  theme(
    aspect.ratio= 2/3,
    axis.text.x = element_text(angle = 270, vjust = 0.5, hjust = 0) 
    )
```

![](modelinglaptime_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->
