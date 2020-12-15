Lilo Heinrich, Tim Novak, and Colin Takeda
12-14-2020

  - [Data Background](#data-background)
  - [Investigation Question](#investigation-question)
  - [Data Tidying](#data-tidying)
  - [Exploratory Data Analysis](#exploratory-data-analysis)
  - [Final Position Order](#final-position-order)
  - [Questions Remaining](#questions-remaining)
  - [Conclusion](#conclusion)

<center>

<h1>

Formula 1 Racing: <br /> Does the car or the driver have the greater
impact?

</h1>

</center>

## Data Background

#### Context

The Formula 1 World Championship has been one of the premier forms of
auto racing around the world since its first season in 1950. The word
“formula” refers to the set of rules that the participants’ cars must
conform to. Each Formula 1 season consists of a series of Grands Prix
races that take place worldwide on racing circuits and closed public
roads. Formula 1 is one of the most popular sports in the world, with
over [450 million TV
viewers](https://digital.hbs.edu/platform-digit/submission/formula-1-speeding-towards-esports-success/)
in 2020.

#### Data Source

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

  - `raceId` & `race_name` The ID number and corresponding name of the
    race
  - `driverId` & `driver_name` The ID number and corresponding driver’s
    name
  - `constructorId` & `constructor_name` The car constructor’s ID number
    and name
  - `positionOrder` The final position order of the race, describing the
    order the racers finished in.
      - Racers that completed the most laps and in the fastest time
        receive the highest positions.
  - `laps` The number of full laps completed
  - `fastestLapSpeed` The speed in km/hr of the racer’s fastest lap of
    the race
  - `statusId` & `status` The status ID number and name.
      - Examples of status name: “Finished”, “Collision”,
        “Disqualified”, or “Engine” (experienced engine problems)
  - `year` The year of the race
  - `circuitId` & `circuit_name` The circuit’s ID number and
    corresponding name

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

  - `total_time` The total time of a race in milliseconds
  - `avg_lap` The average lap time for an individual race in
    milliseconds
  - `circuit_avg_lap` The average lap time for a individual circuit in
    milliseconds
  - `circuit_lap_sd` The standard deviation of the average lap time in
    milliseconds
  - `std_avg_lap` The standardized average lap time \[unitless\]

#### Changes in racing ruleset/vehicle design through the years

This data is for 70 years of races and over that period of time [formula
racing has changed a lot](https://youtu.be/hgLQWIAaCmY). This means that
many factors will change through the years depending on the rule set
that has been put in place. To help mitigate this problem we decided to
filter down to the subset of data which follows the most recent set of
rules for the races. The [most recent significant set of
rules](https://en.wikipedia.org/wiki/Formula_One_engines#2014%E2%80%932021)
dates back to 2014 where the allowed engine specifications were changed.
Thus we filtered our data to only examine the data from 2014 onwards.

#### Potential Problems

  - Some examples of [constructor name
    changes](https://www.reddit.com/r/formula1/comments/1dos3r/i_made_a_diagram_to_show_how_current_f1_teams/)
      - Our dataset does not include the aliases or previous names of
        constructors, so we are unable to account for this
        relationship.  
  - New drivers are skewed because they don’t have as many data points
    yet
      - The more datapoints we have on each driver, the better we can
        characterize their performance. New drivers have fewer known
        datapoints, so their predictions will have a higher level of
        uncertainty.
  - Teams and drivers are correlated, in that the best drivers tend to
    get hired by the best teams
      - This confluence between team and driver may make it difficult to
        separate the impact of driver and constructor.

-----

## Exploratory Data Analysis

#### Standardized Average Lap Time

<img src="Final_Report_files/figure-gfm/finish race-1.png" style="display: block; margin: auto;" />

    ## # A tibble: 1 x 1
    ##   percent_finishrace
    ##                <dbl>
    ## 1               80.4

Final position is heavily reliant on how many laps the driver was able
to complete, and almost a fifth of the time collisions or car troubles
put drivers out of commission before the race is finished. Final
position is not reflective of how well the driver was performing before
an incident happens, so we decided to use average lap time to create a
new metric.

Average lap time is a comprehensive measure of how well a driver
performed in a race because it is informed only on the laps they were
able to complete, unlike final position. The only problem is that it
doesn’t account for the effect of circuit on lap time, so we need to
correct for this difference across circuits.

<img src="Final_Report_files/figure-gfm/average lap time per circuit-1.png" style="display: block; margin: auto;" />

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

<img src="Final_Report_files/figure-gfm/unnamed-chunk-2-1.png" style="display: block; margin: auto;" />

The graph above shows a visual comparison of standardized average lap
time by circuit. The slope of the median is small in magnitude relative
to the range of the standardized average lap time, showing that
standardizing the average lap time successfully minimizes the effect of
circuit.

#### Modeling by Standardized Average Lap Time

First, let’s model the a subset of the data that has completed times to
get a sense of how informative a linear model of **standardized average
lap time** is for our dataset, solely based upon driver, constructor,
and a combination of the two.

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

## Final Position Order

#### Driver and Constructor by Final Position Order

<img src="Final_Report_files/figure-gfm/unnamed-chunk-4-1.png" style="display: block; margin: auto;" />

We can see that when we plot the final position vs the driver of the
vehicle there does seem to be a correlation. in that some drivers tend
to outperform the average and some drivers tend to underperform the
average. If we examine the names, the highly performing racers tend to
be the racers more well renown for their skill, such as [Lewis
Hamilton](https://en.wikipedia.org/wiki/Lewis_Hamilton) and [Nico
Rosberg](https://en.wikipedia.org/wiki/Nico_Rosberg). This suggests that
there is a correlation between the driver performance and the standing
in the race, and that we can see this play out in the relatively linear
relation between the two variables. An interesting relation we can see
in the data are ‘plateaus’ in the median values where there are sets of
drivers with similar performance.

<img src="Final_Report_files/figure-gfm/unnamed-chunk-5-1.png" style="display: block; margin: auto;" />

When we plot the constructor by the final position order, we see
generally that the higher performing constructors are associated with
lower position order, and the lower performing constructors are
associated with higher position order. However this is not a linear
relationship. The highest performing constructors account for most of
the low final position orders and the lowest performing constructors
account for much of the high final position orders, but middle
performing constructors all seem to have similar performance.

Taken together we can see that the driver and constructor graphs are
correlated with the final position order so it is likely that both of
them help account for the final position order, but are not the sole
determining factors. The more linear relationship of driver with final
position order suggests that the driver is slightly more predictive of
the final position order than the less linear vehicle constructor.

#### Modeling by Final Position Order

Next, we will apply a linear model on the entire data set.

    ## Train Fit - Just Driver

    ##   Rsquare 0.3464126

    ##   MSE 23.39755

    ## Train Fit - Just Constructor

    ##   Rsquare 0.3488549

    ##   MSE 23.30775

    ## Train Fit - Driver and Constructor

    ##   Rsquare 0.3817959

    ##   MSE 22.13336

After multiple samples we see that the R-squared values for just driver
and just constructor tend to range from 30% to 37% and a MSE that ranges
from 22 to 24. The just driver and just constructor fits seem to be very
close in R-squared values and MSE to one another and there is no
significant difference between the two.

In comparison, driver and constructor has an R-squared value that tends
to range from 33% to 41% and a MSE of 20 to 23. The driver and
constructor fit tends to always be better than either the just driver or
just constructor fit. Overall, the the range of values for the driver
and constructor fit is promising given the multitude of outside factors
in racing that are not covered by the investigated factors.

#### Predicted vs Actual Final Position Order

<img src="Final_Report_files/figure-gfm/unnamed-chunk-8-1.png" style="display: block; margin: auto;" /><img src="Final_Report_files/figure-gfm/unnamed-chunk-8-2.png" style="display: block; margin: auto;" />

<img src="Final_Report_files/figure-gfm/unnamed-chunk-9-1.png" style="display: block; margin: auto;" />

Above are the visualizations of the actual vs. predicted final position
of our three linear models, with a line through the axis of symmetry
showing where the predicted and actual positions are equal. Comparing
the distance from each datapoint to the line of symmetry, we can see
that there is a weak linear correlation between the predicted and actual
final position in all three models. Interestingly, the shape of the
training data closely resembles the shape of the validation data.

In all of the models, there are very few predicted final positions below
5th place or above 20th place; nearly all of the predicted positions
fall in the middle of the range of possible values. This makes sense,
because a driver or constructor’s performance would have to be either
consistently great or consistently terrible to have achieved a very high
or very low average ranking.

For the model of constructor-only, there are tall vertical
stratifications in predicted final position. This is because the range
of final positions that each constructor achieves is typically wider
than one position value, yet the linear model only assigns a single
coefficient to each constructor, causing all of the datapoints of each
constructor to assume the constructor’s mean position value. This poor
fit is an inherent limitation of reducing the constructors to linear
variables. The same problem applies to considering drivers as linear
variables.

#### Prediction Intervals

<img src="Final_Report_files/figure-gfm/unnamed-chunk-10-1.png" style="display: block; margin: auto;" />

While we are able to compute the prediction intervals on the inputs to
our model, we had a hard time creating an effective visual to compare
the predictions to actual final position in a meaningful way due to the
large number of levels. The graph above shows the position prediction
intervals for each driver-constructor pair, ordered from lowest to
highest predicted value. Graphing the final position on top of the
prediction intervals, we can see that the coverage of the prediction
intervals on the validation data is good, with the majority of
datapoints falling inside the prediction interval.

However, there is a group of high outliers occurring above several of
the driver-constructor pairs with the lowest predicted positions. The
prediction intervals with a lower bound below 0 have much less coverage
of the actual position data, making up most of the outliers seen. If the
predicted value is low, the lower prediction bound will shift into the
negative numbers, but achieving a negative position in the race is not
possible. Prediction intervals are intended for normally distributed
data so the asymmetrical distribution of these driver-constructor pairs
renders them less effective.

The average size of these prediction intervals is approximately 15
positions which is large considering that there are less than 30
possible positions to take in each race. This tells us that the range of
our data is quite wide in comparison to the range of possible values,
unfortunately meaning that the error in our predictions is also
relatively large.

-----

## Questions Remaining

  - Can you visualize rule changes, such as engine specification, on the
    overall trend of the data?
  - Can you look at track changes over the years and effects on lap
    times?
  - How much does starting position have an effect on the final/ending
    position?
  - Using the history of an individual driver’s performances can you
    predict their lap times in future races?

-----

## Conclusion

Our initial driving question sought to answer whether the car or the
driver has a greater impact on overall performance. The answer that we
came to is that both have a sizable impact on performance, and their
individual significance cannot be easily isolated using simple models.
Our models demonstrate that using either driver or constructor on its
own is a significant predictor for final position order. (Which one is a
better approximation relies on the sampling made in the train-test
split). A model using the combination of both variables as a predictor
served as the strongest predictor of final position, however it still
did not sufficiently account for the final position of the vehicles.

If it is desired to have an accurate predictor of the final position of
a Formula 1 race, one should take into account more factors than the two
investigated in this project. However, if one wants to know whether the
driver or the vehicle is more significant in determining results, the
answer is that there is no statistically significant difference between
the two factors’ predictive ability to determine race results.

-----
