DataProcessingDocument
================

***Lilo’s work 12/8***

``` r
# throw out extraneous/irrelevant columns
df_results_trim <- 
  df_results %>%
  select(-c(time, position, positionText, points, grid, number))

# select which attributes to keep about each driver
df_drivers_trim <-
  df_drivers %>%
  unite(driver_name, c(forename, surname), sep = " ") %>% # driver's full name
  select(driverId, driver_name, driver_nationality=nationality)
df_results_plusdrivers <- left_join(df_results_trim, df_drivers_trim, by = "driverId")

# select which attributes to keep about each constructor
df_constructors_trim <-
  df_constructors %>%
  select(constructorId, constructor_name=name, constructor_nationality=nationality)
df_results_plusconstructors <- left_join(df_results_plusdrivers, df_constructors_trim, by = "constructorId")

# select which attributes to keep about each race
df_races_trim <-
  df_races %>%
  select(raceId, year, round, circuitId, circuitName=name)
df_results_plusraces <- left_join(df_results_plusconstructors, df_races_trim, by = "raceId")

# get status from statusId
df_results_plusstatus <- left_join(df_results_plusraces, df_status, by = "statusId")

# turn all \\N into NAs
df_clean <-
  df_results_plusstatus %>%
  mutate_all(na_if, "\\N")
df_clean
```

    ## # A tibble: 24,900 x 21
    ##    resultId raceId driverId constructorId positionOrder  laps milliseconds
    ##       <dbl>  <dbl>    <dbl>         <dbl>         <dbl> <dbl> <chr>       
    ##  1        1     18        1             1             1    58 5690616     
    ##  2        2     18        2             2             2    58 5696094     
    ##  3        3     18        3             3             3    58 5698779     
    ##  4        4     18        4             4             4    58 5707797     
    ##  5        5     18        5             1             5    58 5708630     
    ##  6        6     18        6             3             6    57 <NA>        
    ##  7        7     18        7             5             7    55 <NA>        
    ##  8        8     18        8             6             8    53 <NA>        
    ##  9        9     18        9             2             9    47 <NA>        
    ## 10       10     18       10             7            10    43 <NA>        
    ## # ... with 24,890 more rows, and 14 more variables: fastestLap <chr>,
    ## #   rank <chr>, fastestLapTime <chr>, fastestLapSpeed <chr>, statusId <dbl>,
    ## #   driver_name <chr>, driver_nationality <chr>, constructor_name <chr>,
    ## #   constructor_nationality <chr>, year <dbl>, round <dbl>, circuitId <dbl>,
    ## #   circuitName <chr>, status <chr>

``` r
df_multi_drivers <-
  df_clean %>%
  
  # get one datapoint of each pair of driver and constructor
  group_by(driver_name, constructor_name) %>%
  filter(row_number() == 1) %>%
  
  # get how many constructors that each driver has driven for
  group_by(driver_name) %>%
  mutate(driver_numconstr = n()) %>%
  
  # keep drivers that drove for more than one constructor
  filter(driver_numconstr > 1, row_number() == 1) %>%
  select(driver_name) %>% 
  arrange(driver_name)
df_multi_drivers
```

    ## # A tibble: 499 x 1
    ## # Groups:   driver_name [499]
    ##    driver_name         
    ##    <chr>               
    ##  1 Adrian Sutil        
    ##  2 Aguri Suzuki        
    ##  3 Al Herman           
    ##  4 Al Keller           
    ##  5 Alain Prost         
    ##  6 Alan Jones          
    ##  7 Alan Rees           
    ##  8 Alberto Ascari      
    ##  9 Alberto Colombo     
    ## 10 Alessandro de Tomaso
    ## # ... with 489 more rows

``` r
# get all of the races "multi drivers" drove
df_results_multi_drivers <- inner_join(df_clean, df_multi_drivers, by = "driver_name")
df_results_multi_drivers %>% 
  arrange(driver_name)
```

    ## # A tibble: 23,059 x 21
    ##    resultId raceId driverId constructorId positionOrder  laps milliseconds
    ##       <dbl>  <dbl>    <dbl>         <dbl>         <dbl> <dbl> <chr>       
    ##  1       16     18       16            10            16     8 <NA>        
    ##  2       42     19       16            10            20     5 <NA>        
    ##  3       63     20       16            10            19    56 <NA>        
    ##  4       87     21       16            10            21     0 <NA>        
    ##  5      104     22       16            10            16    57 <NA>        
    ##  6      123     23       16            10            15    67 <NA>        
    ##  7      148     24       16            10            20    13 <NA>        
    ##  8      167     25       16            10            19    69 <NA>        
    ##  9      186     26       16            10            18    10 <NA>        
    ## 10      203     27       16            10            15    67 5550362     
    ## # ... with 23,049 more rows, and 14 more variables: fastestLap <chr>,
    ## #   rank <chr>, fastestLapTime <chr>, fastestLapSpeed <chr>, statusId <dbl>,
    ## #   driver_name <chr>, driver_nationality <chr>, constructor_name <chr>,
    ## #   constructor_nationality <chr>, year <dbl>, round <dbl>, circuitId <dbl>,
    ## #   circuitName <chr>, status <chr>

``` r
#model <-
#  df_clean %>%
#  lm(
#    formula = positionOrder ~ driver_name
    #formula = positionOrder ~ constructor_name
    #formula = positionOrder ~ constructor_name + driver_name
#  )
#rsquare(model, df_clean)

#model2 <-
#  df_results_multi_drivers %>%
#  lm(
#    formula = positionOrder ~ driver_name
    #formula = positionOrder ~ constructor_name
   #formula = positionOrder ~ constructor_name + driver_name
#  )
#rsquare(model2, df_results_multi_drivers)
```

Fitting a linear model to predict final standing for all the datapoints,
driver and constructor have an rsquare of 0.30 and 0.25 respectively.
When I add both variables together in the same model, the rsquare
increases to 0.36, showing a great amount of overlapping information
between driver and constructor and giving a “rank-deficient fit” error.

I separated out only the datapoints about drivers who drove for multiple
constructors, and it turns out that over 23000 of the approximately
25000 datapoints were driven by “multi-drivers” as I’m calling them.
Again fitting a linear model to predict final standing, driver and
constructor have an rsquare of 0.26 and 0.24 respectively, showing a
slight decrease in prediction capability but continuing the trend that
constructor is slightly less accurate of a predictor than driver. When I
add both as linear variables together, the rsquare increases to 0.32,
still showing a great amount of overlapping information between driver
and constructor.

Thinking about why or why not a linear prediction model of final
standing makes sense:

  - Final standing (positionOrder) is a discrete positive integer.
  - It can be greatly affected by status (Collision, etc.) which may be
    random or unpredictable. Maybe that could justify the relatively low
    r-square correlation that we’re seeing from this model?
  - What if I add nationality into the prediction? Do some countries’
    drivers or constructors actually do better overall?
  - Pit stop times also affect the overall time and therefore the final
    standing. Perhaps we could take that into account?
  - Standing can also be greatly affected by starting position in the
    race, because it gives an advantage. Is there a variable that tells
    us starting position?
  - If we care about using driver and constructor as linear variables,
    then they must not be integer ID numbers because the model would
    likely interpret integers as a continuous variable even though both
    of these are in fact discrete, which is why I fit by name and not
    ID.

***Colin’s Work 12/8***

``` r
df_results %>% 
  arrange(desc(raceId), driverId) %>% 
  glimpse()
```

    ## Rows: 24,900
    ## Columns: 18
    ## $ resultId        <dbl> 24886, 24900, 24888, 24903, 24887, 24895, 24899, 24...
    ## $ raceId          <dbl> 1044, 1044, 1044, 1044, 1044, 1044, 1044, 1044, 104...
    ## $ driverId        <dbl> 1, 8, 20, 154, 815, 817, 822, 825, 826, 830, 832, 8...
    ## $ constructorId   <dbl> 131, 51, 6, 210, 211, 4, 131, 210, 213, 9, 1, 4, 21...
    ## $ number          <dbl> 44, 7, 5, 8, 11, 3, 77, 20, 26, 33, 55, 31, 18, 99,...
    ## $ grid            <dbl> 6, 8, 11, 17, 3, 5, 9, 13, 16, 2, 15, 7, 1, 10, 19,...
    ## $ position        <chr> "1", "15", "3", "\\N", "2", "10", "14", "17", "12",...
    ## $ positionText    <chr> "1", "15", "3", "R", "2", "10", "14", "17", "12", "...
    ## $ positionOrder   <dbl> 1, 15, 3, 18, 2, 10, 14, 17, 12, 6, 5, 11, 9, 20, 1...
    ## $ points          <dbl> 25, 0, 15, 0, 18, 1, 0, 0, 0, 8, 10, 0, 2, 0, 0, 12...
    ## $ laps            <dbl> 58, 57, 58, 49, 58, 58, 57, 55, 57, 58, 58, 57, 58,...
    ## $ time            <chr> "1:42:19.313", "\\N", "+31.960", "\\N", "+31.633", ...
    ## $ milliseconds    <chr> "6139313", "\\N", "6171273", "\\N", "6170946", "623...
    ## $ fastestLap      <chr> "56", "57", "53", "38", "50", "54", "57", "45", "53...
    ## $ rank            <chr> "6", "9", "8", "18", "12", "13", "2", "15", "17", "...
    ## $ fastestLapTime  <chr> "1:39.413", "1:39.743", "1:39.662", "1:43.281", "1:...
    ## $ fastestLapSpeed <chr> "193.302", "192.663", "192.819", "186.063", "191.41...
    ## $ statusId        <dbl> 1, 11, 1, 130, 1, 1, 11, 54, 11, 1, 1, 11, 1, 6, 11...

``` r
df_laptimes %>%
  arrange(desc(raceId), driverId) %>% 
  glimpse()
```

    ## Rows: 487,314
    ## Columns: 6
    ## $ raceId       <dbl> 1044, 1044, 1044, 1044, 1044, 1044, 1044, 1044, 1044, ...
    ## $ driverId     <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
    ## $ lap          <dbl> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16,...
    ## $ position     <dbl> 6, 6, 6, 6, 6, 6, 6, 6, 8, 8, 6, 6, 5, 5, 6, 6, 6, 5, ...
    ## $ time         <time> 02:11:00, 01:59:00, 01:56:00, 01:55:00, 01:54:00, 01:...
    ## $ milliseconds <dbl> 131904, 119881, 116659, 115018, 114724, 113283, 112562...

``` r
df_pitstops %>% 
  glimpse()
```

    ## Rows: 7,911
    ## Columns: 7
    ## $ raceId       <dbl> 841, 841, 841, 841, 841, 841, 841, 841, 841, 841, 841,...
    ## $ driverId     <dbl> 153, 30, 17, 4, 13, 22, 20, 814, 816, 67, 2, 1, 808, 3...
    ## $ stop         <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
    ## $ lap          <dbl> 1, 1, 11, 12, 13, 13, 14, 14, 14, 15, 15, 16, 16, 16, ...
    ## $ time         <time> 17:05:23, 17:05:52, 17:20:48, 17:22:34, 17:24:10, 17:...
    ## $ duration     <chr> "26.898", "25.021", "23.426", "23.251", "23.842", "23....
    ## $ milliseconds <dbl> 26898, 25021, 23426, 23251, 23842, 23643, 22603, 24863...

``` r
pits <- df_pitstops %>% 
  filter(driverId == 1, raceId == 841) %>% 
  pull(lap)

df_laptimes %>% 
  filter(driverId == 1, raceId == 841) %>% 
  ggplot(aes(lap, milliseconds  / 1000 / 60)) + 
  geom_vline(xintercept = pits, linetype = 2, color = "grey") +
  geom_line() +
  geom_point(color = "blue") + 
  labs(
    x = "Lap Number",
    y = "Lap Time (Minutes)"
  )
```

![](dataProcessingDocument_files/figure-gfm/lap%20and%20laptimes%20visualization%20for%20specific%20race%20and%20racer-1.png)<!-- -->

``` r
df_laptimes %>% 
  filter(raceId == 841) %>% 
  mutate(driverId = as.factor(driverId))%>% 
  ggplot(aes(lap, milliseconds / 1000 / 60)) +
  geom_line(aes(color = driverId)) + 
  ylim(1.5, 2.5) + 
  labs(
    x = "Lap Number",
    y = "Lap Time (Minutes)"
  )
```

    ## Warning: Removed 1 row(s) containing missing values (geom_path).

![](dataProcessingDocument_files/figure-gfm/lap%20number%20&%20laptimes%20for%20all%20racers%20in%20a%20given%20race-1.png)<!-- -->
