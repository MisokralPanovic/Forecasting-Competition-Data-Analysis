Forecasting Competition Data Analysis
================

# Introduction

This report investigates the performance of different aggregation
methods for forecasting competition assessment, using the RCT-A dataset
from the [HFC competition](https://dataverse.harvard.edu/dataverse/hfc).
I evaluated five aggregation methods, and their performance in correctly
aggregating the predictions from different predictors, and proposed an
improvement based on the best-performing method.  

The dataset was analysed using the **`data.table`** R package, which
allows fast and memory efficient handling of data.

## Downloading the Data

Due to the size of the datasets to be analysed, they can not be hosted
on GitHub. I have hosted them on a Google Drive, and made publicly
accessible. Due to this, the initial step is to download the data with
the help of **`googledrive`** R package.

``` r
# Authenticate with Google Drive
googledrive::drive_auth()

# create key value pairs of dataset names and the corresponding google drive IDs
data_dict_env <- new.env()
data_dict_env[["data/rct-a-daily-forecasts.csv"]] <- "15DlG6rsUrIPcGB0OhLJ55QvmzwTUdUNn"
data_dict_env[["data/rct-a-prediction-sets.csv"]] <- "15JiEmQs1IJMbUyeLcQONUyFRw45GMmms"
data_dict_env[["data/rct-a-questions-answers.csv"]] <- "15LTcptGkzn6DcaCqvXDwaKwwwpWAtPh3"

# download the dataset if they are absent
for (key in ls(data_dict_env)) {
  value <- data_dict_env[[key]]
  
  if (!file.exists(key)) {
    googledrive::drive_download(as_id(value), path = key, overwrite = FALSE)
    message("File downloaded successfully.")
  } else {
    message("File already exists. Download skipped.")
  }
}
```

# Data Structure

The data was read using the **`data.table`** **`fread`** (fast read)
function.

``` r
daily_dt <- data.table::fread("data/rct-a-daily-forecasts.csv")
prediction_dt <- data.table::fread("data/rct-a-prediction-sets.csv")
qa_dt <- data.table::fread("data/rct-a-questions-answers.csv")
```

The first-year competition data comes in three main datasets:

- **`rct-a-questions-answers.csv`** dataset contains metadata on the
  questions, such as dates, taggs, and descriptions. Variables that are
  important to this assignment are: discover IDs for the questions and
  answers (for joining of datasets), and the resolved probabilities for
  the answers (i.e. encoding for the true outcome).

``` r
str(daily_dt) # assess the structure of the data, including if the data was loaded in correct format
```

    ## Classes 'data.table' and 'data.frame':   9251564 obs. of  11 variables:
    ##  $ date                      : POSIXct, format: "2018-03-07 19:00:59" "2018-03-07 19:00:59" ...
    ##  $ discover question id      : int  177 177 177 177 177 177 177 177 177 177 ...
    ##  $ discover answer id        : int  561 559 558 558 559 559 557 561 557 561 ...
    ##  $ site id                   : int  5 5 5 5 5 5 5 5 5 5 ...
    ##  $ external predictor id     : int  78 77 65 76 57 73 56 66 61 65 ...
    ##  $ question id               : int  820 820 820 820 820 820 820 820 820 820 ...
    ##  $ answer id                 : int  2471 2473 2474 2474 2473 2473 2475 2471 2475 2471 ...
    ##  $ external prediction id    : int  6892 6885 6824 6879 6785 6865 6778 6832 6803 6827 ...
    ##  $ external prediction set id: int  2359 2358 2346 2357 2338 2354 2337 2347 2342 2346 ...
    ##  $ created at                : POSIXct, format: "2018-03-07 18:31:21" "2018-03-07 18:31:21" ...
    ##  $ forecast                  : num  0.2 0.2 0.2 0.2 0.2 0.2 0.2 0.2 0.2 0.2 ...
    ##  - attr(*, ".internal.selfref")=<externalptr>

- **`rct-a-daily-forecasts.csv`** dataset contains daily forecast for
  each performer forecasting method, along with indexes that allow
  joining this dataset with the other crucial datasets. Variables that
  are important to this assignment are: date, discover IDs for the
  questions and answers, external prediction set ID (i.e. the ID that is
  common to to a predictor that is assigning probabilities to a set of
  possible answers), and the forecast value itself.

``` r
str(qa_dt) # assess the structure of the data, including if the data was loaded in correct format
```

    ## Classes 'data.table' and 'data.frame':   955 obs. of  38 variables:
    ##  $ discover question id                              : int  5 5 5 5 5 6 7 8 9 9 ...
    ##  $ question name                                     : chr  "How many battle deaths will ACLED record in Sudan in August 2017?" "How many battle deaths will ACLED record in Sudan in August 2017?" "How many battle deaths will ACLED record in Sudan in August 2017?" "How many battle deaths will ACLED record in Sudan in August 2017?" ...
    ##  $ creator id                                        : int  1048 1048 1048 1048 1048 1048 1048 1053 1053 1053 ...
    ##  $ question starts at                                : POSIXct, format: "2017-08-09 16:30:15" "2017-08-09 16:30:15" ...
    ##  $ question ends at                                  : POSIXct, format: "2017-09-01 03:59:15" "2017-09-01 03:59:15" ...
    ##  $ question description                              : chr  "The Armed Conflict Location and Event Dataset (ACLED) summarizes press reports on conflict and protest events i"| __truncated__ "The Armed Conflict Location and Event Dataset (ACLED) summarizes press reports on conflict and protest events i"| __truncated__ "The Armed Conflict Location and Event Dataset (ACLED) summarizes press reports on conflict and protest events i"| __truncated__ "The Armed Conflict Location and Event Dataset (ACLED) summarizes press reports on conflict and protest events i"| __truncated__ ...
    ##  $ question status                                   : chr  "archived" "archived" "archived" "archived" ...
    ##  $ question published at                             : POSIXct, format: "2017-08-09 16:30:29" "2017-08-09 16:30:29" ...
    ##  $ question resolved at                              : POSIXct, format: "2017-09-21 19:23:53" "2017-09-21 19:23:53" ...
    ##  $ question correctness known at                     : POSIXct, format: "2017-09-01 03:59:43" "2017-09-01 03:59:43" ...
    ##  $ use ordinal scoring                               : logi  TRUE TRUE TRUE TRUE TRUE FALSE ...
    ##  $ question created at                               : POSIXct, format: "2017-08-03 13:54:07" "2017-08-03 13:54:07" ...
    ##  $ question tags                                     : chr  "[]" "[]" "[]" "[]" ...
    ##  $ question challenges                               : chr  "[]" "[]" "[]" "[]" ...
    ##  $ discover answer id                                : int  13 14 15 16 17 18 19 20 21 22 ...
    ##  $ answer name                                       : chr  "Less than 7" "Between 7 and 46, inclusive" "More than 46 but less than 122" "Between 122 and 321, inclusive" ...
    ##  $ answer initial probability                        : num  0.2 0.2 0.2 0.2 0.2 0.5 0.5 0.5 0.2 0.2 ...
    ##  $ answer sort order                                 : int  0 1 2 3 4 0 0 0 0 1 ...
    ##  $ answer resolved at                                : POSIXct, format: "2017-09-21 19:23:53" "2017-09-21 19:23:53" ...
    ##  $ answer resolved by user id                        : int  1048 1048 1048 1048 1048 1048 1048 1048 1043 1043 ...
    ##  $ answer resolved probability                       : int  0 1 0 0 0 1 0 0 0 1 ...
    ##  $ answer correctness known at                       : POSIXct, format: "2017-09-01 03:59:43" "2017-09-01 03:59:43" ...
    ##  $ answer created at                                 : POSIXct, format: "2017-08-03 13:54:07" "2017-08-03 13:54:07" ...
    ##  $ clarification 1                                   : logi  NA NA NA NA NA NA ...
    ##  $ clarification 2                                   : logi  NA NA NA NA NA NA ...
    ##  $ clarification 3                                   : chr  "" "" "" "" ...
    ##  $ IFP Generation Method                             : chr  "Data-Driven (R Script)" "Data-Driven (R Script)" "Data-Driven (R Script)" "Data-Driven (R Script)" ...
    ##  $ Country - Primary                                 : chr  "Sudan" "Sudan" "Sudan" "Sudan" ...
    ##  $ Country - Secondary                               : chr  "" "" "" "" ...
    ##  $ Topic                                             : chr  "ACLED battle deaths" "ACLED battle deaths" "ACLED battle deaths" "ACLED battle deaths" ...
    ##  $ Region                                            : chr  "" "" "" "" ...
    ##  $ Non-state violence (Terrorism)                    : chr  "" "" "" "" ...
    ##  $ Domain                                            : chr  "" "" "" "" ...
    ##  $ Created By                                        : chr  "" "" "" "" ...
    ##  $ Deaths                                            : chr  "" "" "" "" ...
    ##  $ RCT                                               : chr  "" "" "" "" ...
    ##  $ Controversial                                     : chr  "" "" "" "" ...
    ##  $ Departure from Status Quo Resolution (Binary Only): chr  "" "" "" "" ...
    ##  - attr(*, ".internal.selfref")=<externalptr>

- **`rct-a-prediction-sets.csv`** contains information on prediction
  sets, along with basic question and answer metadata, forecasted and
  final probability values, along with indexes that allow joining this
  dataset with the other datasets. This dataset seems to be redundant,
  as the important information can be found in the first two datasets.

``` r
str(prediction_dt) # assess the structure of the data, including if the data was loaded in correct format
```

    ## Classes 'data.table' and 'data.frame':   750181 obs. of  33 variables:
    ##  $ prediction set id            : int  340 341 341 341 341 341 342 342 342 342 ...
    ##  $ membership guid              : chr  "52d0b6592281f0d274d93661cf2ef1dd26f55d28" "52d0b6592281f0d274d93661cf2ef1dd26f55d28" "52d0b6592281f0d274d93661cf2ef1dd26f55d28" "52d0b6592281f0d274d93661cf2ef1dd26f55d28" ...
    ##  $ discover question id         : int  182 183 183 183 183 183 183 183 183 183 ...
    ##  $ question id                  : int  807 816 816 816 816 816 816 816 816 816 ...
    ##  $ question name                : chr  "Practice Question: Will there be a new prime minister of the United Kingdom before 1 August 2018?" "Practice Question: What will be the daily closing price of gold on 12 March 2018 in USD?" "Practice Question: What will be the daily closing price of gold on 12 March 2018 in USD?" "Practice Question: What will be the daily closing price of gold on 12 March 2018 in USD?" ...
    ##  $ prediction set created at    : POSIXct, format: "2018-02-28 20:16:37" "2018-02-28 20:18:42" ...
    ##  $ prediction set updated at    : POSIXct, format: "2018-02-28 20:16:37" "2018-02-28 20:18:42" ...
    ##  $ rationale                    : chr  "Given that the negotiation of Brexit is still ongoing (albeit not going particularly well) and there is a coali"| __truncated__ "" "" "" ...
    ##  $ comment id                   : int  391 392 392 392 392 392 393 393 393 393 ...
    ##  $ prediction id                : int  704 705 706 707 708 709 710 711 712 713 ...
    ##  $ answer id                    : int  2410 2451 2452 2453 2454 2455 2451 2452 2453 2454 ...
    ##  $ answer name                  : chr  "Yes" "Less than $1,221" "Between $1,221 and $1,303, inclusive" "More than $1,303 but less than $1,374" ...
    ##  $ filled at                    : POSIXct, format: "2018-02-28 20:16:37" "2018-02-28 20:18:42" ...
    ##  $ created at                   : POSIXct, format: "2018-02-28 20:16:37" "2018-02-28 20:18:42" ...
    ##  $ updated at                   : POSIXct, format: "2018-02-28 20:16:37" "2018-02-28 20:18:42" ...
    ##  $ made after correctness known : logi  FALSE FALSE FALSE FALSE FALSE FALSE ...
    ##  $ forecasted probability       : num  0.05 0.1 0.2 0.3 0.3 0.1 0 0.1 0.8 0.1 ...
    ##  $ starting probability         : num  0.5 0.2 0.2 0.2 0.2 0.2 0.1 0.2 0.3 0.3 ...
    ##  $ final probability            : num  0.05 0.1 0.2 0.3 0.3 0.1 0.05 0.15 0.55 0.2 ...
    ##  $ answer created_at            : POSIXct, format: "2018-02-26 17:05:26" "2018-02-26 17:05:27" ...
    ##  $ answer initial_probability   : num  0.5 0.2 0.2 0.2 0.2 0.2 0.2 0.2 0.2 0.2 ...
    ##  $ answer resolved_at           : POSIXct, format: NA NA ...
    ##  $ answer correctness_known_at  : POSIXct, format: NA NA ...
    ##  $ answer resolved probability  : int  NA NA NA NA NA NA NA NA NA NA ...
    ##  $ answer sort order            : int  0 0 1 2 3 4 0 1 2 3 ...
    ##  $ question ends_at             : POSIXct, format: "2018-03-06 19:00:41" "2018-03-11 18:00:43" ...
    ##  $ question published_at        : POSIXct, format: "2018-02-26 17:05:25" "2018-02-26 17:05:27" ...
    ##  $ question starts_at           : POSIXct, format: "2018-02-26 16:49:41" "2018-02-26 17:00:01" ...
    ##  $ question resolved_at         : POSIXct, format: NA NA ...
    ##  $ question correctness_known_at: POSIXct, format: NA NA ...
    ##  $ question use_ordinal_scoring : logi  FALSE TRUE TRUE TRUE TRUE TRUE ...
    ##  $ site id                      : int  11 11 11 11 11 11 11 11 11 11 ...
    ##  $ site name                    : chr  "HFC Carbon" "HFC Carbon" "HFC Carbon" "HFC Carbon" ...
    ##  - attr(*, ".internal.selfref")=<externalptr>

# Data Cleaning and Preprocessing

To reduce the size of the datasets, only the relevant columns of
**`rct-a-questions-answers.csv`** and **`rct-a-daily-forecasts.csv`**
were selected. These were:

From **`rct-a-daily-forecasts.csv`**:

- **`date`**
- **`discover question id`**
- **`discover answer id`**
- **`forecast`**
- **`created at`**
- **`external prediction set id`**

``` r
# select only columns that will be used in analysis
daily_dt <- daily_dt[, .(date, 
                         `discover question id`, 
                         `discover answer id`,
                         forecast, 
                         `created at`, 
                         `external prediction set id`)]
```

The **`date`** column was converted from **`IDateTime`** format to
**`IDate`** format, for enhanced data clarity.

``` r
# convert datetime to date, to make final summary table nicer
daily_dt[, Day := as.IDate(date)]
```

From **`rct-a-questions-answers.csv`**:

- **`discover question id`**
- **`discover answer id`**
- **`answer resolved probability`**

``` r
# select only columns that will be used in analysis
qa_dt <- qa_dt[, .(`discover question id`, 
                   `discover answer id`,
                   `answer resolved probability`)]
```

The variables of interest were assessed for the presence of missing
values, and these were subsequently removed.

``` r
# drop NA values in important columns (that will be used in the subsequent analysis or preprocessing)
daily_dt_filtered <- daily_dt[complete.cases(daily_dt[, .(
  date, 
  `created at`,
  `discover question id`, 
  `discover answer id`, 
  `external prediction set id`, 
  forecast)])]

qa_dt_filtered <- qa_dt[complete.cases(qa_dt[, .(
  `discover question id`, 
  `discover answer id`, 
  `answer resolved probability`)])]
```

Lastly, only the most recent predictions per predictor per day were
included in the analysis (although it seems that
**`rct-a-daily-forecasts.csv`** dataset already contained only single
predictions per predictor per day).

``` r
# Data cleaning steps
# filter only most recent predictions per prediction on a day
daily_dt_most_recent <- daily_dt_filtered[
  order(-`created at`), # order the data from most recent `created at` time
  .SD[1], # select the first row
  by = .( # group data by:
    date, 
    `discover question id`, 
    `discover answer id`,
    `external prediction set id`)]
```

# Aggregation Methods

I aggregated the individual forecasts for each of the question-day pair
the using five different methods:

- **Arithmetic Mean:** A simple average of all forecasts.  

``` math
\text{Arithmetic Mean}(x) = \frac{1}{n} \sum_{i=1}^{n} x_i
```

- **Median:** The middle value, which is robust to outliers.  

``` math
\text{Median}(x) =
\begin{cases}
x_{\frac{n+1}{2}} & \text{if } n \text{ is odd} \\
\frac{x_{\frac{n}{2}} + x_{\frac{n}{2} + 1}}{2} & \text{if } n \text{ is even}
\end{cases}
```

- **Geometric Mean:** A multiplicative average, reducing the influence
  of extreme forecasts.  

``` math
\text{Geometric Mean}(x) = \exp\left(\frac{1}{n} \sum_{i=1}^{n} \log(x_i)\right)
```

- In the case where any $x_i = 0$, we add a small value $\epsilon$ to
  avoid taking the logarithm of zero.
- **Trimmed Mean:** The arithmetic mean after removing the top and
  bottom 10% of forecasts.  

``` math
\text{Trimmed Mean}(x) = \frac{1}{n - 2k} \sum_{i=k+1}^{n-k} x_{(i)}
```

- where $k = \left\lfloor 0.1n \right\rfloor$ is the number of values
  removed from both the top and bottom of the sorted data.
- **Geometric Mean of Odds:** Converts probabilities to odds before
  calculating the geometric mean.  
  1.  Convert probabilities $p_i$ to odds:

``` math
\text{Odds}(p_i) = \frac{p_i}{1 - p_i}
```

2.  Compute the geometric mean of the odds:

``` math
\text{Geometric Mean of Odds}(p) = \exp\left(\frac{1}{n} \sum_{i=1}^{n} \log\left(\text{Odds}(p_i)\right)\right)
```

3.  Convert the result back to probabilities:

``` math
p = \frac{\text{Geometric Mean of Odds}}{1 + \text{Geometric Mean of Odds}}
```

To do this, I have created two helper functions for calculating the
geometric mean, and geometric mean of odds.

``` r
# create helper functions
geo_mean <- function(x, small_value = 1e-10) {
  x_non_zero <- fifelse(x == 0, small_value, x) # handling 0 values by adding minuscule value (if geometric mean has 0 in the vector the result would be  0)
  return(exp(mean(log(x_non_zero))))
}

geo_mean_odds <- function(x, small_value = 1e-10) {
  x_deextrimised <- fifelse(x == 0, small_value, fifelse(x == 1, 1 - small_value, x)) # handling the presence of 0 and 1 values (would impede the transformation to odds)
  odds <- x_deextrimised/(1-x_deextrimised)
  geo_mean_odds_final <- geo_mean(odds)
  return(geo_mean_odds_final / (1 + geo_mean_odds_final)) # converts back into probabilities
}
```

Following table shows the aggregated data, using the 5 aggregation
method, per day, question, and the possible answers:

``` r
# create the aggregate forecast dataset for each question-date pair using the 5 different methods
aggregated_dt <- daily_dt_most_recent[, .(
  Mean = mean(forecast),
  Median = median(forecast),
  Geo_Mean = geo_mean(forecast),
  Trim_Mean = mean(forecast, trim = 10),
  Geo_Mean_Odds = geo_mean_odds(forecast)
), by = .(Day, `discover question id`, `discover answer id`)][order(Day, `discover question id`, `discover answer id`)] # order the aggregate
aggregated_dt |> head(12) |> kable()
```

| Day | discover question id | discover answer id | Mean | Median | Geo_Mean | Trim_Mean | Geo_Mean_Odds |
|:---|---:|---:|---:|---:|---:|---:|---:|
| 2018-03-07 | 177 | 557 | 0.2000000 | 0.2 | 0.2000000 | 0.2 | 0.2000000 |
| 2018-03-07 | 177 | 558 | 0.2000000 | 0.2 | 0.2000000 | 0.2 | 0.2000000 |
| 2018-03-07 | 177 | 559 | 0.2000000 | 0.2 | 0.2000000 | 0.2 | 0.2000000 |
| 2018-03-07 | 177 | 560 | 0.2000000 | 0.2 | 0.2000000 | 0.2 | 0.2000000 |
| 2018-03-07 | 177 | 561 | 0.2000000 | 0.2 | 0.2000000 | 0.2 | 0.2000000 |
| 2018-03-07 | 178 | 562 | 0.1978132 | 0.2 | 0.1886879 | 0.2 | 0.1904701 |
| 2018-03-07 | 178 | 563 | 0.1978132 | 0.2 | 0.1886879 | 0.2 | 0.1904701 |
| 2018-03-07 | 178 | 564 | 0.1987594 | 0.2 | 0.1981816 | 0.2 | 0.1983119 |
| 2018-03-07 | 178 | 565 | 0.1996246 | 0.2 | 0.1995888 | 0.2 | 0.1995975 |
| 2018-03-07 | 178 | 566 | 0.2059896 | 0.2 | 0.2029114 | 0.2 | 0.2043580 |
| 2018-03-07 | 179 | 468 | 0.5000000 | 0.5 | 0.5000000 | 0.5 | 0.5000000 |
| 2018-03-07 | 180 | 469 | 0.1475433 | 0.2 | 0.0272614 | 0.2 | 0.0311016 |

# Evaluation of Aggregation Methods

To evaluate the accuracy of each aggregation method, I computed the
Brier score, which measures the mean squared error between the
aggregated forecast and the actual outcome.

The **Brier score** is a measure of how close the predicted
probabilities are to the actual outcomes. It is defined as the mean
squared error between the predicted probabilities $\hat{p}_i$ and the
known outcomes $y_i$, given by the formula:

``` math
\text{Brier Score} = \frac{1}{n} \sum_{i=1}^{n} \sum_{j=1}^{r} \left( y_i - \hat{p}_i \right)^2
```

where:

- $y_i$ is the actual outcome (0 or 1)
- $\hat{p}_i$ is the predicted probability for the event
- $r$ is the number of possible forecast outcomes
- $n$ is the total number of predictions

**Brier score** ranges from 0 to 1, where low values indicate better
predictive capabilities.

# Results

To calculate the aggregated barrier score, I have created a function
that does so. It takes as input the predicted probability of event
happening, and the actual probability (i.e. in questions with multiple
answers, it encodes 0 for false answers, and 1 for the correct outcome)

``` r
# define helper function
barrier_score <- function(calculated, known){
  return(mean(sum((known - calculated)^2)))
}
```

To obtain the actual probabilities, the **`qa_dt`** dataset was joined
based on **`discover answer id`** colum to **`aggregated_dt`** dataset.
Since the **`qa_dt`** dataset had already the non-relevant columns
removed, it attached only the **`answer resolved probability`** column.

``` r
# append question metadata (`answer resolved probability` column)
aggregated_metadata_dt <- aggregated_dt[qa_dt, on = .(`discover answer id`), nomatch = 0]
```

Following table shows the **Brier scores** for each question-day pair
per aggregation method used. The final two columns, **`Best_Method`**
and **`Ranked_Methods`**, show the best performing method (i.e. method
with the lowest **Brier score**) and the order of the method
performance, respectively:

``` r
# calculate the barrier scores per day per question
barrier_dt <- aggregated_metadata_dt[, .(
  Mean = barrier_score(Mean, `answer resolved probability`),
  Median = barrier_score(Median, `answer resolved probability`),
  Geo_Mean = barrier_score(Geo_Mean, `answer resolved probability`),
  Trim_Mean = barrier_score(Trim_Mean, `answer resolved probability`),
  Geo_Mean_Odds = barrier_score(Geo_Mean_Odds, `answer resolved probability`)
), by = .(Day, `discover question id`)]
```

``` r
# aggregate table of performance of methods
barrier_dt[, Best_Method := colnames(.SD)[apply(.SD, 1, which.min)], # write the best performing method to Best_Method column
           .SDcols = c("Mean", 
                       "Median", 
                       "Geo_Mean", 
                       "Trim_Mean", 
                       "Geo_Mean_Odds")][, Ranked_Methods := apply(.SD, 1, function(x) {
  method_names <- colnames(.SD)
  ranked_methods <- method_names[order(x)]
  paste(ranked_methods, collapse = " > ")}), 
  .SDcols = c("Mean", "Median", "Geo_Mean", "Trim_Mean", "Geo_Mean_Odds")] # write the order of performance of the methods in Ranked_Methods column
```

| Day | discover question id | Mean | Median | Geo_Mean | Trim_Mean | Geo_Mean_Odds | Best_Method | Ranked_Methods |
|:---|---:|---:|---:|---:|---:|---:|:---|:---|
| 2018-03-07 | 179 | 0.2500000 | 0.2500000 | 0.2500000 | 0.2500000 | 0.2500000 | Mean | Mean \> Median \> Geo_Mean \> Trim_Mean \> Geo_Mean_Odds |
| 2018-03-08 | 179 | 0.2228512 | 0.2454620 | 0.3030460 | 0.2454620 | 0.1115684 | Geo_Mean_Odds | Geo_Mean_Odds \> Mean \> Median \> Trim_Mean \> Geo_Mean |
| 2018-03-09 | 179 | 0.2261973 | 0.2438538 | 0.3143616 | 0.2438538 | 0.1547132 | Geo_Mean_Odds | Geo_Mean_Odds \> Mean \> Median \> Trim_Mean \> Geo_Mean |
| 2018-03-10 | 179 | 0.1684095 | 0.1854015 | 0.5884959 | 0.1854015 | 0.3679411 | Mean | Mean \> Median \> Trim_Mean \> Geo_Mean_Odds \> Geo_Mean |
| 2018-03-11 | 179 | 0.1430425 | 0.0931926 | 0.5579004 | 0.0931926 | 0.2364447 | Median | Median \> Trim_Mean \> Mean \> Geo_Mean_Odds \> Geo_Mean |
| 2018-03-12 | 179 | 0.1256116 | 0.0931926 | 0.5391778 | 0.0931926 | 0.1818183 | Median | Median \> Trim_Mean \> Mean \> Geo_Mean_Odds \> Geo_Mean |
| 2018-03-13 | 179 | 0.1289142 | 0.1593935 | 0.1471616 | 0.1593935 | 0.1027374 | Geo_Mean_Odds | Geo_Mean_Odds \> Mean \> Geo_Mean \> Median \> Trim_Mean |
| 2018-03-14 | 179 | 0.1675982 | 0.1678731 | 0.2871592 | 0.1678731 | 0.2015531 | Mean | Mean \> Median \> Trim_Mean \> Geo_Mean_Odds \> Geo_Mean |
| 2018-03-15 | 179 | 0.1704827 | 0.1944768 | 0.1878166 | 0.1944768 | 0.1626900 | Geo_Mean_Odds | Geo_Mean_Odds \> Mean \> Geo_Mean \> Median \> Trim_Mean |
| 2018-03-16 | 179 | 0.1429946 | 0.1486996 | 0.1547969 | 0.1486996 | 0.1378457 | Geo_Mean_Odds | Geo_Mean_Odds \> Mean \> Median \> Trim_Mean \> Geo_Mean |
| 2018-03-17 | 179 | 0.1560561 | 0.1639200 | 0.2294760 | 0.1639200 | 0.1552047 | Geo_Mean_Odds | Geo_Mean_Odds \> Mean \> Median \> Trim_Mean \> Geo_Mean |
| 2018-03-18 | 179 | 0.1400550 | 0.1454112 | 0.2096699 | 0.1454112 | 0.1075865 | Geo_Mean_Odds | Geo_Mean_Odds \> Mean \> Median \> Trim_Mean \> Geo_Mean |

  
The following table shows the ordered summary of the method performance,
along with the percentage of question-day pairs in which the method
outperformed the rest:

``` r
final_table <- barrier_dt[, .N, by = Best_Method][order(-N)][, Percentage := (N / sum(N)) * 100]
final_table |> head(20) |> kable()
```

| Best_Method   |    N | Percentage |
|:--------------|-----:|-----------:|
| Geo_Mean      | 6396 |   47.78840 |
| Geo_Mean_Odds | 4205 |   31.41811 |
| Median        | 1411 |   10.54244 |
| Mean          | 1372 |   10.25105 |

  
The best performing aggregation method was geometric mean (47.79% of
prediction-day pairs (PDPs)), followed by the geometric mean of odds
(31.42% of PDPs), median (10.54% of PDPs), and the arithmetic mean
(10.25% of PDPs). The trimmed arithmetic mean never outperformed the
other methods. This data suggest that methods that ignore information
from extereme predictions (such as median, mean, and trimmed mean) fail
to capture the true information from aggregate prediction. The geometric
mean and geometric mean of odds appear to compete for the best
prediction method, likely based on the nuances of the structure of the
question and possible answers. Therefore this data suggest that the
nature of question would dictate which aggregate method to use to most
properly assess the aggregate performance of the predictors.

# Improvement on Aggregation Methods

I propose an improvement to the geometric mean of odds by extremising
the odds to penalise under-confidence in forecasters.

The **extremised geometric mean of odds** is calculated in the following
steps:

1.  Convert probabilities $p_i$ to odds:

``` math
\text{Odds}(p_i) = \frac{p_i}{1 - p_i}
```

2.  Compute the geometric mean of the odds:

``` math
\text{Geometric Mean of Odds} = \exp\left(\frac{1}{n} \sum_{i=1}^{n} \log\left(\text{Odds}(p_i)\right)\right)
```

3.  Apply extremisation by raising the geometric mean of odds to the
    power of 2.5:

``` math
\text{Extremised Odds} = \left( \text{Geometric Mean of Odds} \right)^{2.5}
```

4.  Convert the extremised odds back into probabilities:

``` math
p_{\text{extremised}} = \frac{\text{Extremised Odds}}{1 + \text{Extremised Odds}}
```

To use this new aggregaton method, a new helper function was created.

``` r
# New method: Extremised geometric mean of odds
geo_mean_odds_extremised <- function(x, small_value = 1e-10) {
  x_deextrimised <- fifelse(x == 0, small_value, fifelse(x == 1, 1 - small_value, x)) # handling 0 and 1 values
  odds <- x_deextrimised/(1-x_deextrimised)
  geo_mean_odds_final <- geo_mean(odds)^2.5 #penalisation of under-confident experts
  return(geo_mean_odds_final / (1 + geo_mean_odds_final)) # converts back into probabilities
}
```

Prior to the assessment of the improved, best performing method, the
**`rct-a-daily-forecasts.csv`** dataset was filtered to include only the
data from the first day of the competition.

``` r
day1_dt <- daily_dt_most_recent[Day == min(daily_dt_most_recent$Day)]
```

Following table shows the aggregated data, using the 6 aggregation
method (including the improved method), on day 1, per question and the
possible answers:

``` r
aggregated_day1_dt <- day1_dt[, .(
  Mean = mean(forecast),
  Median = median(forecast),
  Geo_Mean = geo_mean(forecast),
  Trim_Mean = mean(forecast, trim = 10),
  Geo_Mean_Odds = geo_mean_odds(forecast),
  Geo_Means_Odds_Extremised = geo_mean_odds_extremised(forecast)
), by = .(`discover question id`, `discover answer id`)][order(`discover question id`, `discover answer id`)]
```

| discover question id | discover answer id | Mean | Median | Geo_Mean | Trim_Mean | Geo_Mean_Odds | Geo_Means_Odds_Extremised |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 177 | 557 | 0.2000000 | 0.2 | 0.2000000 | 0.2 | 0.2000000 | 0.0303030 |
| 177 | 558 | 0.2000000 | 0.2 | 0.2000000 | 0.2 | 0.2000000 | 0.0303030 |
| 177 | 559 | 0.2000000 | 0.2 | 0.2000000 | 0.2 | 0.2000000 | 0.0303030 |
| 177 | 560 | 0.2000000 | 0.2 | 0.2000000 | 0.2 | 0.2000000 | 0.0303030 |
| 177 | 561 | 0.2000000 | 0.2 | 0.2000000 | 0.2 | 0.2000000 | 0.0303030 |
| 178 | 562 | 0.1978132 | 0.2 | 0.1886879 | 0.2 | 0.1904701 | 0.0261503 |
| 178 | 563 | 0.1978132 | 0.2 | 0.1886879 | 0.2 | 0.1904701 | 0.0261503 |
| 178 | 564 | 0.1987594 | 0.2 | 0.1981816 | 0.2 | 0.1983119 | 0.0295351 |
| 178 | 565 | 0.1996246 | 0.2 | 0.1995888 | 0.2 | 0.1995975 | 0.0301186 |
| 178 | 566 | 0.2059896 | 0.2 | 0.2029114 | 0.2 | 0.2043580 | 0.0323521 |
| 179 | 468 | 0.5000000 | 0.5 | 0.5000000 | 0.5 | 0.5000000 | 0.5000000 |
| 180 | 469 | 0.1475433 | 0.2 | 0.0272614 | 0.2 | 0.0311016 | 0.0001846 |

  
Following table shows the **Brier scores** for each question-day pair
per aggregation method used. The final two columns, **`Best_Method`**
and **`Ranked_Methods`**, show the best performing method (i.e. method
with the lowest **Brier score**) and the order of the method
performance, respectively:

``` r
# append question metadata (`answer resolved probability` column)
aggregated_day1_metadata_dt <- aggregated_day1_dt[qa_dt, on = .(`discover answer id`), nomatch = 0]
```

``` r
# calculate the barrier scores per question
barrier_day1_dt <- aggregated_day1_metadata_dt[, .(
  Mean = barrier_score(Mean, `answer resolved probability`),
  Median = barrier_score(Median, `answer resolved probability`),
  Geo_Mean = barrier_score(Geo_Mean, `answer resolved probability`),
  Trim_Mean = barrier_score(Trim_Mean, `answer resolved probability`),
  Geo_Mean_Odds = barrier_score(Geo_Mean_Odds, `answer resolved probability`),
  Geo_Means_Odds_Extremised =  barrier_score(Geo_Means_Odds_Extremised, `answer resolved probability`)
), by = .(`discover question id`)]
barrier_day1_dt
```

``` r
# aggregate table of performance of methods
final_table_new_method <- barrier_day1_dt[, Best_Method := colnames(.SD)[apply(.SD, 1, which.min)],
                                .SDcols = c("Mean", 
                                            "Median", 
                                            "Geo_Mean", 
                                            "Trim_Mean", 
                                            "Geo_Mean_Odds",
                                            "Geo_Means_Odds_Extremised")][, Ranked_Methods := apply(.SD, 1, function(x) {
                                              method_names <- colnames(.SD)
                                              ranked_methods <- method_names[order(x)]
                                              paste(ranked_methods, collapse = " > ")}), 
                                              .SDcols = c("Mean", "Median", "Geo_Mean", 
                                                          "Trim_Mean", "Geo_Mean_Odds", 
                                                          "Geo_Means_Odds_Extremised")] # write the order of performance of the methods in Ranked_Methods column
```

| Day | discover question id | Mean | Median | Geo_Mean | Trim_Mean | Geo_Mean_Odds | Best_Method | Ranked_Methods |
|:---|---:|---:|---:|---:|---:|---:|:---|:---|
| 2018-03-07 | 179 | 0.2500000 | 0.2500000 | 0.2500000 | 0.2500000 | 0.2500000 | Mean | Mean \> Median \> Geo_Mean \> Trim_Mean \> Geo_Mean_Odds |
| 2018-03-08 | 179 | 0.2228512 | 0.2454620 | 0.3030460 | 0.2454620 | 0.1115684 | Geo_Mean_Odds | Geo_Mean_Odds \> Mean \> Median \> Trim_Mean \> Geo_Mean |
| 2018-03-09 | 179 | 0.2261973 | 0.2438538 | 0.3143616 | 0.2438538 | 0.1547132 | Geo_Mean_Odds | Geo_Mean_Odds \> Mean \> Median \> Trim_Mean \> Geo_Mean |
| 2018-03-10 | 179 | 0.1684095 | 0.1854015 | 0.5884959 | 0.1854015 | 0.3679411 | Mean | Mean \> Median \> Trim_Mean \> Geo_Mean_Odds \> Geo_Mean |
| 2018-03-11 | 179 | 0.1430425 | 0.0931926 | 0.5579004 | 0.0931926 | 0.2364447 | Median | Median \> Trim_Mean \> Mean \> Geo_Mean_Odds \> Geo_Mean |
| 2018-03-12 | 179 | 0.1256116 | 0.0931926 | 0.5391778 | 0.0931926 | 0.1818183 | Median | Median \> Trim_Mean \> Mean \> Geo_Mean_Odds \> Geo_Mean |
| 2018-03-13 | 179 | 0.1289142 | 0.1593935 | 0.1471616 | 0.1593935 | 0.1027374 | Geo_Mean_Odds | Geo_Mean_Odds \> Mean \> Geo_Mean \> Median \> Trim_Mean |
| 2018-03-14 | 179 | 0.1675982 | 0.1678731 | 0.2871592 | 0.1678731 | 0.2015531 | Mean | Mean \> Median \> Trim_Mean \> Geo_Mean_Odds \> Geo_Mean |
| 2018-03-15 | 179 | 0.1704827 | 0.1944768 | 0.1878166 | 0.1944768 | 0.1626900 | Geo_Mean_Odds | Geo_Mean_Odds \> Mean \> Geo_Mean \> Median \> Trim_Mean |
| 2018-03-16 | 179 | 0.1429946 | 0.1486996 | 0.1547969 | 0.1486996 | 0.1378457 | Geo_Mean_Odds | Geo_Mean_Odds \> Mean \> Median \> Trim_Mean \> Geo_Mean |
| 2018-03-17 | 179 | 0.1560561 | 0.1639200 | 0.2294760 | 0.1639200 | 0.1552047 | Geo_Mean_Odds | Geo_Mean_Odds \> Mean \> Median \> Trim_Mean \> Geo_Mean |
| 2018-03-18 | 179 | 0.1400550 | 0.1454112 | 0.2096699 | 0.1454112 | 0.1075865 | Geo_Mean_Odds | Geo_Mean_Odds \> Mean \> Median \> Trim_Mean \> Geo_Mean |

  
The following table shows the ordered summary of the method performance,
along with the percentage of question-day pairs in which the method
outperformed the rest:

``` r
final_table_new_method[, .N, by = Best_Method][order(-N)][, Percentage := (N / sum(N)) * 100] |> head(12)|> kable()
```

| Best_Method               |   N | Percentage |
|:--------------------------|----:|-----------:|
| Geo_Means_Odds_Extremised |   9 |  42.857143 |
| Mean                      |   6 |  28.571429 |
| Median                    |   4 |  19.047619 |
| Geo_Mean                  |   1 |   4.761905 |
| Geo_Mean_Odds             |   1 |   4.761905 |

  
The best performing aggregation method was the extremised geometric mean
of odds (42.86% of PDPs), followed by the arithmetic mean (28.57% of
PDPs), median (19.05% of PDPs), the geometric mean (4.76% of PDPs), and
the geometric mean of odds (4.76% of PDPs). The trimmed arithmetic mean
never outperformed the other methods. Evidently, the extremised
geometric mean of odds outperformed the other methods and thus was an
clear improvement in the prediction evaluation. The working principle
behind it is a modification of geometric mean of odds, where the
geometric mean of odds is raised to the power of an extremising
parameter, in this case equal to 2.5. This method is a correction for
forecaster under-confidence. In the present dataset it was able to
outcompete the other methods, however, it is likely that utilising it on
a different dataset, which would contain less forecaster
under-confidence would make it non-optimal.

# Conclusion

The extremised geometric mean of odds provided the best aggregation
performance, suggesting that penalising under-confident predictions can
improve forecasting accuracy. However, the effectiveness of this method
may vary depending on the dataset’s structure and the forecasters’
behaviour.
