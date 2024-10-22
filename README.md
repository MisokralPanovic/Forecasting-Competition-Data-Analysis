Assignment Memo
================

# Introduction

This report investigates the performance of different aggregation
methods for forecasting competition assessment, using the RCT-A dataset
from the HFC competition. I evaluated the five aggregation methods and
proposed an improvement based on the best-performing method.  

The dataset was analysed using the **`data.table`** R package, which
allows fast and memory efficient handling of data.

# Data Structure

The first-year competition data comes in three main datasets:

- **`rct-a-questions-answers.csv`** dataset contains metadata on the
  questions, such as dates, tags, and descriptions. Variables that are
  important to this assignment are: discover IDs for the questions and
  answers (for joining of datasets), and the resolved probabilities for
  the answers (i.e. encoding for the true outcome).

- **`rct-a-daily-forecasts.csv`** dataset contains daily forecast for
  each performer forecasting method, along with indexes that allow
  joining this dataset with the other crucial datasets. Variables that
  are important to this assignment are: date, discover IDs for the
  questions and answers, external prediction set ID (i.e. the ID that is
  common to to a predictor that is assigning probabilities to a set of
  possible answers), and the forecast value itself.

- **`rct-a-prediction-sets.csv`** contains information on prediction
  sets, along with basic question and answer metadata, forecasted and
  final probability values, along with indexes that allow joining this
  dataset with the other datasets. This dataset seems to be redundant,
  as the important information can be found in the first two datasets.

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

From **`rct-a-questions-answers.csv`**:

- **`discover question id`**
- **`discover answer id`**
- **`answer resolved probability`**

The variables of interest were assessed for the presence of missing
values, and these were subsequently removed. Lastly, only the most
recent predictions per predictor per day were included in the analysis
(although it seems that **`rct-a-daily-forecasts.csv`** dataset already
contained only single predictions per predictor per day).

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

Following table shows the aggregated data, using the 5 aggregation
method, per day, question, and the possible answers:

| Day | discover question id | discover answer id | Mean | Median | Geo_Mean | Trim_Mean | Geo_Mean_Odds |
|:---|---:|---:|---:|---:|---:|---:|---:|
| 2018-03-07 | 177 | 557 | 0.2000000 | 0.2000000 | 0.2000000 | 0.2000000 | 0.2000000 |
| 2018-03-07 | 177 | 558 | 0.2000000 | 0.2000000 | 0.2000000 | 0.2000000 | 0.2000000 |
| 2018-03-07 | 177 | 559 | 0.2000000 | 0.2000000 | 0.2000000 | 0.2000000 | 0.2000000 |
| 2018-03-07 | 177 | 560 | 0.2000000 | 0.2000000 | 0.2000000 | 0.2000000 | 0.2000000 |
| 2018-03-07 | 177 | 561 | 0.2000000 | 0.2000000 | 0.2000000 | 0.2000000 | 0.2000000 |
| 2018-03-07 | 178 | 562 | 0.1978132 | 0.2000000 | 0.1886879 | 0.2000000 | 0.1904701 |
| 2018-03-07 | 178 | 563 | 0.1978132 | 0.2000000 | 0.1886879 | 0.2000000 | 0.1904701 |
| 2018-03-07 | 178 | 564 | 0.1987594 | 0.2000000 | 0.1981816 | 0.2000000 | 0.1983119 |
| 2018-03-07 | 178 | 565 | 0.1996246 | 0.2000000 | 0.1995888 | 0.2000000 | 0.1995975 |
| 2018-03-07 | 178 | 566 | 0.2059896 | 0.2000000 | 0.2029114 | 0.2000000 | 0.2043580 |
| 2018-03-07 | 179 | 468 | 0.5000000 | 0.5000000 | 0.5000000 | 0.5000000 | 0.5000000 |
| 2018-03-07 | 180 | 469 | 0.1475433 | 0.2000000 | 0.0272614 | 0.2000000 | 0.0311016 |
| 2018-03-07 | 180 | 470 | 0.1895033 | 0.2000000 | 0.0387339 | 0.2000000 | 0.0457014 |
| 2018-03-07 | 180 | 471 | 0.3700446 | 0.2000000 | 0.3029051 | 0.2000000 | 0.7298552 |
| 2018-03-07 | 180 | 472 | 0.1594388 | 0.2000000 | 0.0318456 | 0.2000000 | 0.0365952 |
| 2018-03-07 | 180 | 473 | 0.1334700 | 0.2000000 | 0.0005810 | 0.2000000 | 0.0006738 |
| 2018-03-07 | 181 | 474 | 0.0764310 | 0.0468170 | 0.0000365 | 0.0468170 | 0.0000398 |
| 2018-03-07 | 181 | 475 | 0.1023125 | 0.1633235 | 0.0000482 | 0.1633235 | 0.0000540 |
| 2018-03-07 | 181 | 476 | 0.6173301 | 0.5650837 | 0.5048271 | 0.5650837 | 0.9800570 |
| 2018-03-07 | 181 | 477 | 0.1260330 | 0.2000000 | 0.0046545 | 0.2000000 | 0.0053246 |

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

Following table shows the **Brier scores** for each question-day pair
per aggregation method used. The final two columns, **`Best_Method`**
and **`Ranked_Methods`**, show the best performing method (i.e. method
with the lowest **Brier score**) and the order of the method
performance, respectively:

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
| 2018-03-19 | 179 | 0.1289182 | 0.1369000 | 0.1962357 | 0.1369000 | 0.1066196 | Geo_Mean_Odds | Geo_Mean_Odds \> Mean \> Median \> Trim_Mean \> Geo_Mean |
| 2018-03-20 | 179 | 0.1261416 | 0.1290515 | 0.1919909 | 0.1290515 | 0.1039282 | Geo_Mean_Odds | Geo_Mean_Odds \> Mean \> Median \> Trim_Mean \> Geo_Mean |
| 2018-03-21 | 179 | 0.1206404 | 0.1369000 | 0.1793264 | 0.1369000 | 0.0992413 | Geo_Mean_Odds | Geo_Mean_Odds \> Mean \> Median \> Trim_Mean \> Geo_Mean |
| 2018-03-22 | 179 | 0.1249560 | 0.1433185 | 0.1817539 | 0.1433185 | 0.1053609 | Geo_Mean_Odds | Geo_Mean_Odds \> Mean \> Median \> Trim_Mean \> Geo_Mean |
| 2018-03-23 | 179 | 0.1331086 | 0.1521000 | 0.1900316 | 0.1521000 | 0.1133599 | Geo_Mean_Odds | Geo_Mean_Odds \> Mean \> Median \> Trim_Mean \> Geo_Mean |
| 2018-03-24 | 179 | 0.1302167 | 0.1496469 | 0.1867844 | 0.1496469 | 0.1080152 | Geo_Mean_Odds | Geo_Mean_Odds \> Mean \> Median \> Trim_Mean \> Geo_Mean |
| 2018-03-25 | 179 | 0.1559943 | 0.1767576 | 0.1653350 | 0.1767576 | 0.1486316 | Geo_Mean_Odds | Geo_Mean_Odds \> Mean \> Geo_Mean \> Median \> Trim_Mean |
| 2018-03-26 | 179 | 0.1343479 | 0.1570272 | 0.1463788 | 0.1570272 | 0.1175452 | Geo_Mean_Odds | Geo_Mean_Odds \> Mean \> Geo_Mean \> Median \> Trim_Mean |

  
The following table shows the ordered summary of the method performance,
along with the percentage of question-day pairs in which the method
outperformed the rest:

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

Prior to the assessment of the improved, best performing method, the
**`rct-a-daily-forecasts.csv`** dataset was filtered to include only the
data from the first day of the competition.

Following table shows the aggregated data, using the 6 aggregation
method (including the improved method), on day 1, per question and the
possible answers:

| discover question id | discover answer id | Mean | Median | Geo_Mean | Trim_Mean | Geo_Mean_Odds | Geo_Means_Odds_Extremised |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 177 | 557 | 0.2000000 | 0.2000000 | 0.2000000 | 0.2000000 | 0.2000000 | 0.0303030 |
| 177 | 558 | 0.2000000 | 0.2000000 | 0.2000000 | 0.2000000 | 0.2000000 | 0.0303030 |
| 177 | 559 | 0.2000000 | 0.2000000 | 0.2000000 | 0.2000000 | 0.2000000 | 0.0303030 |
| 177 | 560 | 0.2000000 | 0.2000000 | 0.2000000 | 0.2000000 | 0.2000000 | 0.0303030 |
| 177 | 561 | 0.2000000 | 0.2000000 | 0.2000000 | 0.2000000 | 0.2000000 | 0.0303030 |
| 178 | 562 | 0.1978132 | 0.2000000 | 0.1886879 | 0.2000000 | 0.1904701 | 0.0261503 |
| 178 | 563 | 0.1978132 | 0.2000000 | 0.1886879 | 0.2000000 | 0.1904701 | 0.0261503 |
| 178 | 564 | 0.1987594 | 0.2000000 | 0.1981816 | 0.2000000 | 0.1983119 | 0.0295351 |
| 178 | 565 | 0.1996246 | 0.2000000 | 0.1995888 | 0.2000000 | 0.1995975 | 0.0301186 |
| 178 | 566 | 0.2059896 | 0.2000000 | 0.2029114 | 0.2000000 | 0.2043580 | 0.0323521 |
| 179 | 468 | 0.5000000 | 0.5000000 | 0.5000000 | 0.5000000 | 0.5000000 | 0.5000000 |
| 180 | 469 | 0.1475433 | 0.2000000 | 0.0272614 | 0.2000000 | 0.0311016 | 0.0001846 |
| 180 | 470 | 0.1895033 | 0.2000000 | 0.0387339 | 0.2000000 | 0.0457014 | 0.0005016 |
| 180 | 471 | 0.3700446 | 0.2000000 | 0.3029051 | 0.2000000 | 0.7298552 | 0.9230637 |
| 180 | 472 | 0.1594388 | 0.2000000 | 0.0318456 | 0.2000000 | 0.0365952 | 0.0002811 |
| 180 | 473 | 0.1334700 | 0.2000000 | 0.0005810 | 0.2000000 | 0.0006738 | 0.0000000 |
| 181 | 474 | 0.0764310 | 0.0468170 | 0.0000365 | 0.0468170 | 0.0000398 | 0.0000000 |
| 181 | 475 | 0.1023125 | 0.1633235 | 0.0000482 | 0.1633235 | 0.0000540 | 0.0000000 |
| 181 | 476 | 0.6173301 | 0.5650837 | 0.5048271 | 0.5650837 | 0.9800570 | 0.9999409 |
| 181 | 477 | 0.1260330 | 0.2000000 | 0.0046545 | 0.2000000 | 0.0053246 | 0.0000021 |

  
Following table shows the **Brier scores** for each question-day pair
per aggregation method used. The final two columns, **`Best_Method`**
and **`Ranked_Methods`**, show the best performing method (i.e. method
with the lowest **Brier score**) and the order of the method
performance, respectively:

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
| 2018-03-19 | 179 | 0.1289182 | 0.1369000 | 0.1962357 | 0.1369000 | 0.1066196 | Geo_Mean_Odds | Geo_Mean_Odds \> Mean \> Median \> Trim_Mean \> Geo_Mean |
| 2018-03-20 | 179 | 0.1261416 | 0.1290515 | 0.1919909 | 0.1290515 | 0.1039282 | Geo_Mean_Odds | Geo_Mean_Odds \> Mean \> Median \> Trim_Mean \> Geo_Mean |
| 2018-03-21 | 179 | 0.1206404 | 0.1369000 | 0.1793264 | 0.1369000 | 0.0992413 | Geo_Mean_Odds | Geo_Mean_Odds \> Mean \> Median \> Trim_Mean \> Geo_Mean |
| 2018-03-22 | 179 | 0.1249560 | 0.1433185 | 0.1817539 | 0.1433185 | 0.1053609 | Geo_Mean_Odds | Geo_Mean_Odds \> Mean \> Median \> Trim_Mean \> Geo_Mean |
| 2018-03-23 | 179 | 0.1331086 | 0.1521000 | 0.1900316 | 0.1521000 | 0.1133599 | Geo_Mean_Odds | Geo_Mean_Odds \> Mean \> Median \> Trim_Mean \> Geo_Mean |
| 2018-03-24 | 179 | 0.1302167 | 0.1496469 | 0.1867844 | 0.1496469 | 0.1080152 | Geo_Mean_Odds | Geo_Mean_Odds \> Mean \> Median \> Trim_Mean \> Geo_Mean |
| 2018-03-25 | 179 | 0.1559943 | 0.1767576 | 0.1653350 | 0.1767576 | 0.1486316 | Geo_Mean_Odds | Geo_Mean_Odds \> Mean \> Geo_Mean \> Median \> Trim_Mean |
| 2018-03-26 | 179 | 0.1343479 | 0.1570272 | 0.1463788 | 0.1570272 | 0.1175452 | Geo_Mean_Odds | Geo_Mean_Odds \> Mean \> Geo_Mean \> Median \> Trim_Mean |

  
The following table shows the ordered summary of the method performance,
along with the percentage of question-day pairs in which the method
outperformed the rest:

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
