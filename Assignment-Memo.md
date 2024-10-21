---
title: "Assignment Memo"
output: 
  html_document:
    keep_md: true
    toc: true
    toc_float: true
---

<style>
body {
text-align: justify}
</style>





# Introduction
This report investigates the performance of different aggregation methods for forecasting competition assessment, using the RCT-A dataset from the HFC competition. I evaluated the five aggregation methods and proposed an improvement based on the best-performing method.\

The dataset was analysed using the **`data.table`** R package, which allows fast and memory efficient handling of data.

# Data Structure

The first-year competition data comes in three main datasets:

- **`rct-a-questions-answers.csv`** dataset contains metadata on the questions, such as dates, taggs, and descriptions. Variables that are important to this assignment are: discover IDs for the questions and answers (for joining of datasets), and the resolved probabilities for the answers (i.e. encoding for the true outcome).

- **`rct-a-daily-forecasts.csv`** dataset contains daily forecast for each performer forecasting method, along with indexes that allow joining this dataset with the other crucial datasets. Variables that are important to this assignment are: date, discover IDs for the questions and answers, external prediction set ID (i.e. the ID that is common to to a predictor that is assigning probabilities to a set of possible answers), and the forecast value itself.

- **`rct-a-prediction-sets.csv`** contains information on prediction sets, along with basic question and answer metadata, forecasted and final probability values, along with indexes that allow joining this dataset with the other datasets. This dataset seems to be redundant, as the important information can be found in the first two datasets.

# Data Cleaning and Preprocessing



To reduce the size of the datasets, only the relevant columns of **`rct-a-questions-answers.csv`** and **`rct-a-daily-forecasts.csv`** were selected. These were:

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

The variables of interest were assessed for the presence of missing values, and these were subsequently removed. Lastly, only the most recent predictions per predictor per day were included in the analysis (although it seems that **`rct-a-daily-forecasts.csv`** dataset already contained only single predictions per predictor per day).










# Aggregation Methods
I aggregated the individual forecasts for each of the question-day pair the using five different methods:

- **Arithmetic Mean:** A simple average of all forecasts.\
\[
\text{Arithmetic Mean}(x) = \frac{1}{n} \sum_{i=1}^{n} x_i
\]
- **Median:** The middle value, which is robust to outliers.\
\[
\text{Median}(x) = 
\begin{cases}
x_{\frac{n+1}{2}} & \text{if } n \text{ is odd} \\
\frac{x_{\frac{n}{2}} + x_{\frac{n}{2} + 1}}{2} & \text{if } n \text{ is even}
\end{cases}
\]
- **Geometric Mean:** A multiplicative average, reducing the influence of extreme forecasts.\
\[
\text{Geometric Mean}(x) = \exp\left(\frac{1}{n} \sum_{i=1}^{n} \log(x_i)\right)
\]
  - In the case where any \( x_i = 0 \), we add a small value \( \epsilon \) to avoid taking the logarithm of zero.
- **Trimmed Mean:** The arithmetic mean after removing the top and bottom 10% of forecasts.\
\[
\text{Trimmed Mean}(x) = \frac{1}{n - 2k} \sum_{i=k+1}^{n-k} x_{(i)}
\]
  - where \( k = \left\lfloor 0.1n \right\rfloor \) is the number of values removed from both the top and bottom of the sorted data.
- **Geometric Mean of Odds:** Converts probabilities to odds before calculating the geometric mean.\
  1. Convert probabilities \( p_i \) to odds:
\[
\text{Odds}(p_i) = \frac{p_i}{1 - p_i}
\]
  2. Compute the geometric mean of the odds:
\[
\text{Geometric Mean of Odds}(p) = \exp\left(\frac{1}{n} \sum_{i=1}^{n} \log\left(\text{Odds}(p_i)\right)\right)
\]
  3. Convert the result back to probabilities:
\[
p = \frac{\text{Geometric Mean of Odds}}{1 + \text{Geometric Mean of Odds}}
\]


Following table shows the aggregated data, using the 5 aggregation method, per day, question, and the possible answers:

```
##            Day discover question id discover answer id       Mean     Median
##         <IDat>                <int>              <int>      <num>      <num>
##  1: 2018-03-07                  177                557 0.20000000 0.20000000
##  2: 2018-03-07                  177                558 0.20000000 0.20000000
##  3: 2018-03-07                  177                559 0.20000000 0.20000000
##  4: 2018-03-07                  177                560 0.20000000 0.20000000
##  5: 2018-03-07                  177                561 0.20000000 0.20000000
##  6: 2018-03-07                  178                562 0.19781319 0.20000000
##  7: 2018-03-07                  178                563 0.19781319 0.20000000
##  8: 2018-03-07                  178                564 0.19875944 0.20000000
##  9: 2018-03-07                  178                565 0.19962463 0.20000000
## 10: 2018-03-07                  178                566 0.20598955 0.20000000
## 11: 2018-03-07                  179                468 0.50000000 0.50000000
## 12: 2018-03-07                  180                469 0.14754334 0.20000000
## 13: 2018-03-07                  180                470 0.18950332 0.20000000
## 14: 2018-03-07                  180                471 0.37004459 0.20000000
## 15: 2018-03-07                  180                472 0.15943879 0.20000000
## 16: 2018-03-07                  180                473 0.13346996 0.20000000
## 17: 2018-03-07                  181                474 0.07643101 0.04681695
## 18: 2018-03-07                  181                475 0.10231247 0.16332348
## 19: 2018-03-07                  181                476 0.61733005 0.56508371
## 20: 2018-03-07                  181                477 0.12603303 0.20000000
##            Day discover question id discover answer id       Mean     Median
##         Geo_Mean  Trim_Mean Geo_Mean_Odds
##            <num>      <num>         <num>
##  1: 2.000000e-01 0.20000000  2.000000e-01
##  2: 2.000000e-01 0.20000000  2.000000e-01
##  3: 2.000000e-01 0.20000000  2.000000e-01
##  4: 2.000000e-01 0.20000000  2.000000e-01
##  5: 2.000000e-01 0.20000000  2.000000e-01
##  6: 1.886879e-01 0.20000000  1.904701e-01
##  7: 1.886879e-01 0.20000000  1.904701e-01
##  8: 1.981816e-01 0.20000000  1.983119e-01
##  9: 1.995888e-01 0.20000000  1.995975e-01
## 10: 2.029114e-01 0.20000000  2.043580e-01
## 11: 5.000000e-01 0.50000000  5.000000e-01
## 12: 2.726138e-02 0.20000000  3.110157e-02
## 13: 3.873392e-02 0.20000000  4.570137e-02
## 14: 3.029051e-01 0.20000000  7.298552e-01
## 15: 3.184560e-02 0.20000000  3.659521e-02
## 16: 5.809808e-04 0.20000000  6.738064e-04
## 17: 3.653535e-05 0.04681695  3.975081e-05
## 18: 4.822822e-05 0.16332348  5.403929e-05
## 19: 5.048271e-01 0.56508371  9.800570e-01
## 20: 4.654535e-03 0.20000000  5.324583e-03
##         Geo_Mean  Trim_Mean Geo_Mean_Odds
```

# Evaluation of Aggregation Methods
To evaluate the accuracy of each aggregation method, I computed the Brier score, which measures the mean squared error between the aggregated forecast and the actual outcome.


The **Brier score** is a measure of how close the predicted probabilities are to the actual outcomes. It is defined as the mean squared error between the predicted probabilities \( \hat{p}_i \) and the known outcomes \( y_i \), given by the formula:

\[
\text{Brier Score} = \frac{1}{n} \sum_{i=1}^{n} \sum_{j=1}^{r} \left( y_i - \hat{p}_i \right)^2
\]

where:

- \( y_i \) is the actual outcome (0 or 1)
- \( \hat{p}_i \) is the predicted probability for the event
- \( r \) is the number of possible forecast outcomes
- \( n \) is the total number of predictions

**Brier score** ranges from 0 to 1, where low values indicate better predictive capabilities.




# Results
Following table shows the **Brier scores** for each question-day pair per aggregation method used. The final two columns, **`Best_Method`** and **`Ranked_Methods`**, show the best performing method (i.e. method with the lowest **Brier score**) and the order of the method performance, respectively:



```
##            Day discover question id      Mean     Median  Geo_Mean  Trim_Mean
##         <IDat>                <int>     <num>      <num>     <num>      <num>
##  1: 2018-03-07                  179 0.2500000 0.25000000 0.2500000 0.25000000
##  2: 2018-03-08                  179 0.2228512 0.24546198 0.3030460 0.24546198
##  3: 2018-03-09                  179 0.2261973 0.24385379 0.3143616 0.24385379
##  4: 2018-03-10                  179 0.1684095 0.18540150 0.5884959 0.18540150
##  5: 2018-03-11                  179 0.1430425 0.09319259 0.5579004 0.09319259
##  6: 2018-03-12                  179 0.1256116 0.09319259 0.5391778 0.09319259
##  7: 2018-03-13                  179 0.1289142 0.15939347 0.1471616 0.15939347
##  8: 2018-03-14                  179 0.1675982 0.16787305 0.2871592 0.16787305
##  9: 2018-03-15                  179 0.1704827 0.19447678 0.1878166 0.19447678
## 10: 2018-03-16                  179 0.1429946 0.14869955 0.1547969 0.14869955
## 11: 2018-03-17                  179 0.1560561 0.16392004 0.2294760 0.16392004
## 12: 2018-03-18                  179 0.1400550 0.14541115 0.2096699 0.14541115
## 13: 2018-03-19                  179 0.1289182 0.13690000 0.1962357 0.13690000
## 14: 2018-03-20                  179 0.1261416 0.12905148 0.1919909 0.12905148
## 15: 2018-03-21                  179 0.1206404 0.13690000 0.1793264 0.13690000
## 16: 2018-03-22                  179 0.1249560 0.14331848 0.1817539 0.14331848
## 17: 2018-03-23                  179 0.1331086 0.15210000 0.1900316 0.15210000
## 18: 2018-03-24                  179 0.1302167 0.14964689 0.1867844 0.14964689
## 19: 2018-03-25                  179 0.1559943 0.17675763 0.1653350 0.17675763
## 20: 2018-03-26                  179 0.1343479 0.15702722 0.1463788 0.15702722
##            Day discover question id      Mean     Median  Geo_Mean  Trim_Mean
##     Geo_Mean_Odds   Best_Method
##             <num>        <char>
##  1:    0.25000000          Mean
##  2:    0.11156837 Geo_Mean_Odds
##  3:    0.15471319 Geo_Mean_Odds
##  4:    0.36794108          Mean
##  5:    0.23644475        Median
##  6:    0.18181828        Median
##  7:    0.10273736 Geo_Mean_Odds
##  8:    0.20155305          Mean
##  9:    0.16268998 Geo_Mean_Odds
## 10:    0.13784569 Geo_Mean_Odds
## 11:    0.15520474 Geo_Mean_Odds
## 12:    0.10758649 Geo_Mean_Odds
## 13:    0.10661963 Geo_Mean_Odds
## 14:    0.10392824 Geo_Mean_Odds
## 15:    0.09924134 Geo_Mean_Odds
## 16:    0.10536091 Geo_Mean_Odds
## 17:    0.11335986 Geo_Mean_Odds
## 18:    0.10801517 Geo_Mean_Odds
## 19:    0.14863162 Geo_Mean_Odds
## 20:    0.11754518 Geo_Mean_Odds
##     Geo_Mean_Odds   Best_Method
##                                           Ranked_Methods
##                                                   <char>
##  1: Mean > Median > Geo_Mean > Trim_Mean > Geo_Mean_Odds
##  2: Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean
##  3: Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean
##  4: Mean > Median > Trim_Mean > Geo_Mean_Odds > Geo_Mean
##  5: Median > Trim_Mean > Mean > Geo_Mean_Odds > Geo_Mean
##  6: Median > Trim_Mean > Mean > Geo_Mean_Odds > Geo_Mean
##  7: Geo_Mean_Odds > Mean > Geo_Mean > Median > Trim_Mean
##  8: Mean > Median > Trim_Mean > Geo_Mean_Odds > Geo_Mean
##  9: Geo_Mean_Odds > Mean > Geo_Mean > Median > Trim_Mean
## 10: Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean
## 11: Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean
## 12: Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean
## 13: Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean
## 14: Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean
## 15: Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean
## 16: Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean
## 17: Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean
## 18: Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean
## 19: Geo_Mean_Odds > Mean > Geo_Mean > Median > Trim_Mean
## 20: Geo_Mean_Odds > Mean > Geo_Mean > Median > Trim_Mean
##                                           Ranked_Methods
```
\
The  following table shows the ordered summary of the method performance, along with the percentage of question-day pairs in which the method outperformed the rest:

```
##      Best_Method     N Percentage
##           <char> <int>      <num>
## 1:      Geo_Mean  6396   47.78840
## 2: Geo_Mean_Odds  4205   31.41811
## 3:        Median  1411   10.54244
## 4:          Mean  1372   10.25105
```
\
The best performing aggregation method was geometric mean (47.79% of prediction-day pairs (PDPs)), followed by the geometric mean of odds (31.42% of PDPs), median (10.54% of PDPs), and the arithmetic mean (10.25% of PDPs). The trimmed arithmetic mean never outperformed the other methods. This data suggest that methods that ignore information from extereme predictions (such as median, mean, and trimmed mean) fail to capture the true information from aggregate prediction. The geometric mean and geometric mean of odds appear to compete for the best prediction method, likely based on the nuances of the structure of the question and possible answers. Therefore this data suggest that the nature of question would dictate which aggregate method to use to most properly assess the aggregate performance of the predictors.

# Improvement on Aggregation Methods
I propose an improvement to the geometric mean of odds by extremising the odds to penalise under-confidence in forecasters.

The **extremised geometric mean of odds** is calculated in the following steps:

1. Convert probabilities \( p_i \) to odds:

\[
\text{Odds}(p_i) = \frac{p_i}{1 - p_i}
\]

2. Compute the geometric mean of the odds:

\[
\text{Geometric Mean of Odds} = \exp\left(\frac{1}{n} \sum_{i=1}^{n} \log\left(\text{Odds}(p_i)\right)\right)
\]

3. Apply extremisation by raising the geometric mean of odds to the power of 2.5:

\[
\text{Extremised Odds} = \left( \text{Geometric Mean of Odds} \right)^{2.5}
\]

4. Convert the extremised odds back into probabilities:

\[
p_{\text{extremised}} = \frac{\text{Extremised Odds}}{1 + \text{Extremised Odds}}
\]

Prior to the assessment of the improved, best performing method, the **`rct-a-daily-forecasts.csv`** dataset was filtered to include only the data from the first day of the competition.






Following table shows the aggregated data, using the 6 aggregation method (including the improved method), on day 1, per question and the possible answers:

```
##     discover question id discover answer id       Mean     Median     Geo_Mean
##                    <int>              <int>      <num>      <num>        <num>
##  1:                  177                557 0.20000000 0.20000000 2.000000e-01
##  2:                  177                558 0.20000000 0.20000000 2.000000e-01
##  3:                  177                559 0.20000000 0.20000000 2.000000e-01
##  4:                  177                560 0.20000000 0.20000000 2.000000e-01
##  5:                  177                561 0.20000000 0.20000000 2.000000e-01
##  6:                  178                562 0.19781319 0.20000000 1.886879e-01
##  7:                  178                563 0.19781319 0.20000000 1.886879e-01
##  8:                  178                564 0.19875944 0.20000000 1.981816e-01
##  9:                  178                565 0.19962463 0.20000000 1.995888e-01
## 10:                  178                566 0.20598955 0.20000000 2.029114e-01
## 11:                  179                468 0.50000000 0.50000000 5.000000e-01
## 12:                  180                469 0.14754334 0.20000000 2.726138e-02
## 13:                  180                470 0.18950332 0.20000000 3.873392e-02
## 14:                  180                471 0.37004459 0.20000000 3.029051e-01
## 15:                  180                472 0.15943879 0.20000000 3.184560e-02
## 16:                  180                473 0.13346996 0.20000000 5.809808e-04
## 17:                  181                474 0.07643101 0.04681695 3.653535e-05
## 18:                  181                475 0.10231247 0.16332348 4.822822e-05
## 19:                  181                476 0.61733005 0.56508371 5.048271e-01
## 20:                  181                477 0.12603303 0.20000000 4.654535e-03
##     discover question id discover answer id       Mean     Median     Geo_Mean
##      Trim_Mean Geo_Mean_Odds Geo_Means_Odds_Extremised
##          <num>         <num>                     <num>
##  1: 0.20000000  2.000000e-01              3.030303e-02
##  2: 0.20000000  2.000000e-01              3.030303e-02
##  3: 0.20000000  2.000000e-01              3.030303e-02
##  4: 0.20000000  2.000000e-01              3.030303e-02
##  5: 0.20000000  2.000000e-01              3.030303e-02
##  6: 0.20000000  1.904701e-01              2.615029e-02
##  7: 0.20000000  1.904701e-01              2.615029e-02
##  8: 0.20000000  1.983119e-01              2.953508e-02
##  9: 0.20000000  1.995975e-01              3.011863e-02
## 10: 0.20000000  2.043580e-01              3.235208e-02
## 11: 0.50000000  5.000000e-01              5.000000e-01
## 12: 0.20000000  3.110157e-02              1.845780e-04
## 13: 0.20000000  4.570137e-02              5.016426e-04
## 14: 0.20000000  7.298552e-01              9.230637e-01
## 15: 0.20000000  3.659521e-02              2.811359e-04
## 16: 0.20000000  6.738064e-04              1.180510e-08
## 17: 0.04681695  3.975081e-05              9.963415e-12
## 18: 0.16332348  5.403929e-05              2.147003e-11
## 19: 0.56508371  9.800570e-01              9.999409e-01
## 20: 0.20000000  5.324583e-03              2.096572e-06
##      Trim_Mean Geo_Mean_Odds Geo_Means_Odds_Extremised
```


\
Following table shows the **Brier scores** for each question-day pair per aggregation method used. The final two columns, **`Best_Method`** and **`Ranked_Methods`**, show the best performing method (i.e. method with the lowest **Brier score**) and the order of the method performance, respectively:




```
##            Day discover question id      Mean     Median  Geo_Mean  Trim_Mean
##         <IDat>                <int>     <num>      <num>     <num>      <num>
##  1: 2018-03-07                  179 0.2500000 0.25000000 0.2500000 0.25000000
##  2: 2018-03-08                  179 0.2228512 0.24546198 0.3030460 0.24546198
##  3: 2018-03-09                  179 0.2261973 0.24385379 0.3143616 0.24385379
##  4: 2018-03-10                  179 0.1684095 0.18540150 0.5884959 0.18540150
##  5: 2018-03-11                  179 0.1430425 0.09319259 0.5579004 0.09319259
##  6: 2018-03-12                  179 0.1256116 0.09319259 0.5391778 0.09319259
##  7: 2018-03-13                  179 0.1289142 0.15939347 0.1471616 0.15939347
##  8: 2018-03-14                  179 0.1675982 0.16787305 0.2871592 0.16787305
##  9: 2018-03-15                  179 0.1704827 0.19447678 0.1878166 0.19447678
## 10: 2018-03-16                  179 0.1429946 0.14869955 0.1547969 0.14869955
## 11: 2018-03-17                  179 0.1560561 0.16392004 0.2294760 0.16392004
## 12: 2018-03-18                  179 0.1400550 0.14541115 0.2096699 0.14541115
## 13: 2018-03-19                  179 0.1289182 0.13690000 0.1962357 0.13690000
## 14: 2018-03-20                  179 0.1261416 0.12905148 0.1919909 0.12905148
## 15: 2018-03-21                  179 0.1206404 0.13690000 0.1793264 0.13690000
## 16: 2018-03-22                  179 0.1249560 0.14331848 0.1817539 0.14331848
## 17: 2018-03-23                  179 0.1331086 0.15210000 0.1900316 0.15210000
## 18: 2018-03-24                  179 0.1302167 0.14964689 0.1867844 0.14964689
## 19: 2018-03-25                  179 0.1559943 0.17675763 0.1653350 0.17675763
## 20: 2018-03-26                  179 0.1343479 0.15702722 0.1463788 0.15702722
##            Day discover question id      Mean     Median  Geo_Mean  Trim_Mean
##     Geo_Mean_Odds   Best_Method
##             <num>        <char>
##  1:    0.25000000          Mean
##  2:    0.11156837 Geo_Mean_Odds
##  3:    0.15471319 Geo_Mean_Odds
##  4:    0.36794108          Mean
##  5:    0.23644475        Median
##  6:    0.18181828        Median
##  7:    0.10273736 Geo_Mean_Odds
##  8:    0.20155305          Mean
##  9:    0.16268998 Geo_Mean_Odds
## 10:    0.13784569 Geo_Mean_Odds
## 11:    0.15520474 Geo_Mean_Odds
## 12:    0.10758649 Geo_Mean_Odds
## 13:    0.10661963 Geo_Mean_Odds
## 14:    0.10392824 Geo_Mean_Odds
## 15:    0.09924134 Geo_Mean_Odds
## 16:    0.10536091 Geo_Mean_Odds
## 17:    0.11335986 Geo_Mean_Odds
## 18:    0.10801517 Geo_Mean_Odds
## 19:    0.14863162 Geo_Mean_Odds
## 20:    0.11754518 Geo_Mean_Odds
##     Geo_Mean_Odds   Best_Method
##                                           Ranked_Methods
##                                                   <char>
##  1: Mean > Median > Geo_Mean > Trim_Mean > Geo_Mean_Odds
##  2: Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean
##  3: Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean
##  4: Mean > Median > Trim_Mean > Geo_Mean_Odds > Geo_Mean
##  5: Median > Trim_Mean > Mean > Geo_Mean_Odds > Geo_Mean
##  6: Median > Trim_Mean > Mean > Geo_Mean_Odds > Geo_Mean
##  7: Geo_Mean_Odds > Mean > Geo_Mean > Median > Trim_Mean
##  8: Mean > Median > Trim_Mean > Geo_Mean_Odds > Geo_Mean
##  9: Geo_Mean_Odds > Mean > Geo_Mean > Median > Trim_Mean
## 10: Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean
## 11: Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean
## 12: Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean
## 13: Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean
## 14: Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean
## 15: Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean
## 16: Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean
## 17: Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean
## 18: Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean
## 19: Geo_Mean_Odds > Mean > Geo_Mean > Median > Trim_Mean
## 20: Geo_Mean_Odds > Mean > Geo_Mean > Median > Trim_Mean
##                                           Ranked_Methods
```

\
The  following table shows the ordered summary of the method performance, along with the percentage of question-day pairs in which the method outperformed the rest:

```
##                  Best_Method     N Percentage
##                       <char> <int>      <num>
## 1: Geo_Means_Odds_Extremised     9  42.857143
## 2:                      Mean     6  28.571429
## 3:                    Median     4  19.047619
## 4:                  Geo_Mean     1   4.761905
## 5:             Geo_Mean_Odds     1   4.761905
```
\
The best performing aggregation method was the extremised geometric mean of odds (42.86% of PDPs), followed by the arithmetic mean (28.57% of PDPs), median (19.05% of PDPs), the geometric mean (4.76% of PDPs), and the geometric mean of odds (4.76% of PDPs). The trimmed arithmetic mean never outperformed the other methods. Evidently, the extremised geometric mean of odds outperformed the other methods and thus was an clear improvement in the prediction evaluation. The working principle behind it is a modification of geometric mean of odds, where the geometric mean of odds is raised to the power of an extremising parameter, in this case equal to 2.5. This method is a correction for forecaster under-confidence. In the present dataset it was able to outcompete the other methods, however, it is likely that utilising it on a different dataset, which would contain less forecaster under-confidence would make it non-optimal.

# Conclusion
The extremised geometric mean of odds provided the best aggregation performance, suggesting that penalising under-confident predictions can improve forecasting accuracy. However, the effectiveness of this method may vary depending on the dataset's structure and the forecasters' behaviour.
