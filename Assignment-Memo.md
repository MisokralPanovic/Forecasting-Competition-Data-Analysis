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





# Introduction {#introduction}

This report investigates the performance of different aggregation methods for forecasting competition assessment, using the RCT-A dataset from the HFC competition. I evaluated the five aggregation methods and proposed an improvement based on the best-performing method.\

The dataset was analysed using the **`data.table`** R package, which allows fast and memory efficient handling of data.

# Data Structure {#data_structure}

The first-year competition data comes in three main datasets:

- **`rct-a-questions-answers.csv`** dataset contains metadata on the questions, such as dates, taggs, and descriptions. Variables that are important to this assignment are: discover IDs for the questions and answers (for joining of datasets), and the resolved probabilities for the answers (i.e. encoding for the true outcome).

- **`rct-a-daily-forecasts.csv`** dataset contains daily forecast for each performer forecasting method, along with indexes that allow joining this dataset with the other crucial datasets. Variables that are important to this assignment are: date, discover IDs for the questions and answers, external prediction set ID (i.e. the ID that is common to to a predictor that is assigning probabilities to a set of possible answers), and the forecast value itself.

- **`rct-a-prediction-sets.csv`** contains information on prediction sets, along with basic question and answer metadata, forecasted and final probability values, along with indexes that allow joining this dataset with the other datasets. This dataset seems to be redundant, as the important information can be found in the first two datasets.

# Data Cleaning and Preprocessing {#data_cleaning_preprocessing}



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










# Aggregation Methods {#aggregation_methods}
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
<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Day"],"name":[1],"type":["IDate"],"align":["right"]},{"label":["discover question id"],"name":[2],"type":["int"],"align":["right"]},{"label":["discover answer id"],"name":[3],"type":["int"],"align":["right"]},{"label":["Mean"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["Median"],"name":[5],"type":["dbl"],"align":["right"]},{"label":["Geo_Mean"],"name":[6],"type":["dbl"],"align":["right"]},{"label":["Trim_Mean"],"name":[7],"type":["dbl"],"align":["right"]},{"label":["Geo_Mean_Odds"],"name":[8],"type":["dbl"],"align":["right"]}],"data":[{"1":"2018-03-07","2":"177","3":"557","4":"0.20000000","5":"0.20000000","6":"2.000000e-01","7":"0.20000000","8":"2.000000e-01"},{"1":"2018-03-07","2":"177","3":"558","4":"0.20000000","5":"0.20000000","6":"2.000000e-01","7":"0.20000000","8":"2.000000e-01"},{"1":"2018-03-07","2":"177","3":"559","4":"0.20000000","5":"0.20000000","6":"2.000000e-01","7":"0.20000000","8":"2.000000e-01"},{"1":"2018-03-07","2":"177","3":"560","4":"0.20000000","5":"0.20000000","6":"2.000000e-01","7":"0.20000000","8":"2.000000e-01"},{"1":"2018-03-07","2":"177","3":"561","4":"0.20000000","5":"0.20000000","6":"2.000000e-01","7":"0.20000000","8":"2.000000e-01"},{"1":"2018-03-07","2":"178","3":"562","4":"0.19781319","5":"0.20000000","6":"1.886879e-01","7":"0.20000000","8":"1.904701e-01"},{"1":"2018-03-07","2":"178","3":"563","4":"0.19781319","5":"0.20000000","6":"1.886879e-01","7":"0.20000000","8":"1.904701e-01"},{"1":"2018-03-07","2":"178","3":"564","4":"0.19875944","5":"0.20000000","6":"1.981816e-01","7":"0.20000000","8":"1.983119e-01"},{"1":"2018-03-07","2":"178","3":"565","4":"0.19962463","5":"0.20000000","6":"1.995888e-01","7":"0.20000000","8":"1.995975e-01"},{"1":"2018-03-07","2":"178","3":"566","4":"0.20598955","5":"0.20000000","6":"2.029114e-01","7":"0.20000000","8":"2.043580e-01"},{"1":"2018-03-07","2":"179","3":"468","4":"0.50000000","5":"0.50000000","6":"5.000000e-01","7":"0.50000000","8":"5.000000e-01"},{"1":"2018-03-07","2":"180","3":"469","4":"0.14754334","5":"0.20000000","6":"2.726138e-02","7":"0.20000000","8":"3.110157e-02"},{"1":"2018-03-07","2":"180","3":"470","4":"0.18950332","5":"0.20000000","6":"3.873392e-02","7":"0.20000000","8":"4.570137e-02"},{"1":"2018-03-07","2":"180","3":"471","4":"0.37004459","5":"0.20000000","6":"3.029051e-01","7":"0.20000000","8":"7.298552e-01"},{"1":"2018-03-07","2":"180","3":"472","4":"0.15943879","5":"0.20000000","6":"3.184560e-02","7":"0.20000000","8":"3.659521e-02"},{"1":"2018-03-07","2":"180","3":"473","4":"0.13346996","5":"0.20000000","6":"5.809808e-04","7":"0.20000000","8":"6.738064e-04"},{"1":"2018-03-07","2":"181","3":"474","4":"0.07643101","5":"0.04681695","6":"3.653535e-05","7":"0.04681695","8":"3.975081e-05"},{"1":"2018-03-07","2":"181","3":"475","4":"0.10231247","5":"0.16332348","6":"4.822822e-05","7":"0.16332348","8":"5.403929e-05"},{"1":"2018-03-07","2":"181","3":"476","4":"0.61733005","5":"0.56508371","6":"5.048271e-01","7":"0.56508371","8":"9.800570e-01"},{"1":"2018-03-07","2":"181","3":"477","4":"0.12603303","5":"0.20000000","6":"4.654535e-03","7":"0.20000000","8":"5.324583e-03"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

# Evaluation of Aggregation Methods{#evaluation_aggregation}
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




# Results {#results}
Following table shows the **Brier scores** for each question-day pair per aggregation method used. The final two columns, **`Best_Method`** and **`Ranked_Methods`**, show the best performing method (i.e. method with the lowest **Brier score**) and the order of the method performance, respectively:


<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Day"],"name":[1],"type":["IDate"],"align":["right"]},{"label":["discover question id"],"name":[2],"type":["int"],"align":["right"]},{"label":["Mean"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["Median"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["Geo_Mean"],"name":[5],"type":["dbl"],"align":["right"]},{"label":["Trim_Mean"],"name":[6],"type":["dbl"],"align":["right"]},{"label":["Geo_Mean_Odds"],"name":[7],"type":["dbl"],"align":["right"]},{"label":["Best_Method"],"name":[8],"type":["chr"],"align":["left"]},{"label":["Ranked_Methods"],"name":[9],"type":["chr"],"align":["left"]}],"data":[{"1":"2018-03-07","2":"179","3":"0.2500000","4":"0.25000000","5":"0.2500000","6":"0.25000000","7":"0.25000000","8":"Mean","9":"Mean > Median > Geo_Mean > Trim_Mean > Geo_Mean_Odds"},{"1":"2018-03-08","2":"179","3":"0.2228512","4":"0.24546198","5":"0.3030460","6":"0.24546198","7":"0.11156837","8":"Geo_Mean_Odds","9":"Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean"},{"1":"2018-03-09","2":"179","3":"0.2261973","4":"0.24385379","5":"0.3143616","6":"0.24385379","7":"0.15471319","8":"Geo_Mean_Odds","9":"Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean"},{"1":"2018-03-10","2":"179","3":"0.1684095","4":"0.18540150","5":"0.5884959","6":"0.18540150","7":"0.36794108","8":"Mean","9":"Mean > Median > Trim_Mean > Geo_Mean_Odds > Geo_Mean"},{"1":"2018-03-11","2":"179","3":"0.1430425","4":"0.09319259","5":"0.5579004","6":"0.09319259","7":"0.23644475","8":"Median","9":"Median > Trim_Mean > Mean > Geo_Mean_Odds > Geo_Mean"},{"1":"2018-03-12","2":"179","3":"0.1256116","4":"0.09319259","5":"0.5391778","6":"0.09319259","7":"0.18181828","8":"Median","9":"Median > Trim_Mean > Mean > Geo_Mean_Odds > Geo_Mean"},{"1":"2018-03-13","2":"179","3":"0.1289142","4":"0.15939347","5":"0.1471616","6":"0.15939347","7":"0.10273736","8":"Geo_Mean_Odds","9":"Geo_Mean_Odds > Mean > Geo_Mean > Median > Trim_Mean"},{"1":"2018-03-14","2":"179","3":"0.1675982","4":"0.16787305","5":"0.2871592","6":"0.16787305","7":"0.20155305","8":"Mean","9":"Mean > Median > Trim_Mean > Geo_Mean_Odds > Geo_Mean"},{"1":"2018-03-15","2":"179","3":"0.1704827","4":"0.19447678","5":"0.1878166","6":"0.19447678","7":"0.16268998","8":"Geo_Mean_Odds","9":"Geo_Mean_Odds > Mean > Geo_Mean > Median > Trim_Mean"},{"1":"2018-03-16","2":"179","3":"0.1429946","4":"0.14869955","5":"0.1547969","6":"0.14869955","7":"0.13784569","8":"Geo_Mean_Odds","9":"Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean"},{"1":"2018-03-17","2":"179","3":"0.1560561","4":"0.16392004","5":"0.2294760","6":"0.16392004","7":"0.15520474","8":"Geo_Mean_Odds","9":"Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean"},{"1":"2018-03-18","2":"179","3":"0.1400550","4":"0.14541115","5":"0.2096699","6":"0.14541115","7":"0.10758649","8":"Geo_Mean_Odds","9":"Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean"},{"1":"2018-03-19","2":"179","3":"0.1289182","4":"0.13690000","5":"0.1962357","6":"0.13690000","7":"0.10661963","8":"Geo_Mean_Odds","9":"Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean"},{"1":"2018-03-20","2":"179","3":"0.1261416","4":"0.12905148","5":"0.1919909","6":"0.12905148","7":"0.10392824","8":"Geo_Mean_Odds","9":"Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean"},{"1":"2018-03-21","2":"179","3":"0.1206404","4":"0.13690000","5":"0.1793264","6":"0.13690000","7":"0.09924134","8":"Geo_Mean_Odds","9":"Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean"},{"1":"2018-03-22","2":"179","3":"0.1249560","4":"0.14331848","5":"0.1817539","6":"0.14331848","7":"0.10536091","8":"Geo_Mean_Odds","9":"Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean"},{"1":"2018-03-23","2":"179","3":"0.1331086","4":"0.15210000","5":"0.1900316","6":"0.15210000","7":"0.11335986","8":"Geo_Mean_Odds","9":"Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean"},{"1":"2018-03-24","2":"179","3":"0.1302167","4":"0.14964689","5":"0.1867844","6":"0.14964689","7":"0.10801517","8":"Geo_Mean_Odds","9":"Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean"},{"1":"2018-03-25","2":"179","3":"0.1559943","4":"0.17675763","5":"0.1653350","6":"0.17675763","7":"0.14863162","8":"Geo_Mean_Odds","9":"Geo_Mean_Odds > Mean > Geo_Mean > Median > Trim_Mean"},{"1":"2018-03-26","2":"179","3":"0.1343479","4":"0.15702722","5":"0.1463788","6":"0.15702722","7":"0.11754518","8":"Geo_Mean_Odds","9":"Geo_Mean_Odds > Mean > Geo_Mean > Median > Trim_Mean"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>
\
The  following table shows the ordered summary of the method performance, along with the percentage of question-day pairs in which the method outperformed the rest:
<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Best_Method"],"name":[1],"type":["chr"],"align":["left"]},{"label":["N"],"name":[2],"type":["int"],"align":["right"]},{"label":["Percentage"],"name":[3],"type":["dbl"],"align":["right"]}],"data":[{"1":"Geo_Mean","2":"6396","3":"47.78840"},{"1":"Geo_Mean_Odds","2":"4205","3":"31.41811"},{"1":"Median","2":"1411","3":"10.54244"},{"1":"Mean","2":"1372","3":"10.25105"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>
\
The best performing aggregation method was geometric mean (47.79% of prediction-day pairs (PDPs)), followed by the geometric mean of odds (31.42% of PDPs), median (10.54% of PDPs), and the arithmetic mean (10.25% of PDPs). The trimmed arithmetic mean never outperformed the other methods. This data suggest that methods that ignore information from extereme predictions (such as median, mean, and trimmed mean) fail to capture the true information from aggregate prediction. The geometric mean and geometric mean of odds appear to compete for the best prediction method, likely based on the nuances of the structure of the question and possible answers. Therefore this data suggest that the nature of question would dictate which aggregate method to use to most properly assess the aggregate performance of the predictors.

# Improvement on Aggregation Methods {#improvement_aggregation}
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
<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["discover question id"],"name":[1],"type":["int"],"align":["right"]},{"label":["discover answer id"],"name":[2],"type":["int"],"align":["right"]},{"label":["Mean"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["Median"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["Geo_Mean"],"name":[5],"type":["dbl"],"align":["right"]},{"label":["Trim_Mean"],"name":[6],"type":["dbl"],"align":["right"]},{"label":["Geo_Mean_Odds"],"name":[7],"type":["dbl"],"align":["right"]},{"label":["Geo_Means_Odds_Extremised"],"name":[8],"type":["dbl"],"align":["right"]}],"data":[{"1":"177","2":"557","3":"0.20000000","4":"0.20000000","5":"2.000000e-01","6":"0.20000000","7":"2.000000e-01","8":"3.030303e-02"},{"1":"177","2":"558","3":"0.20000000","4":"0.20000000","5":"2.000000e-01","6":"0.20000000","7":"2.000000e-01","8":"3.030303e-02"},{"1":"177","2":"559","3":"0.20000000","4":"0.20000000","5":"2.000000e-01","6":"0.20000000","7":"2.000000e-01","8":"3.030303e-02"},{"1":"177","2":"560","3":"0.20000000","4":"0.20000000","5":"2.000000e-01","6":"0.20000000","7":"2.000000e-01","8":"3.030303e-02"},{"1":"177","2":"561","3":"0.20000000","4":"0.20000000","5":"2.000000e-01","6":"0.20000000","7":"2.000000e-01","8":"3.030303e-02"},{"1":"178","2":"562","3":"0.19781319","4":"0.20000000","5":"1.886879e-01","6":"0.20000000","7":"1.904701e-01","8":"2.615029e-02"},{"1":"178","2":"563","3":"0.19781319","4":"0.20000000","5":"1.886879e-01","6":"0.20000000","7":"1.904701e-01","8":"2.615029e-02"},{"1":"178","2":"564","3":"0.19875944","4":"0.20000000","5":"1.981816e-01","6":"0.20000000","7":"1.983119e-01","8":"2.953508e-02"},{"1":"178","2":"565","3":"0.19962463","4":"0.20000000","5":"1.995888e-01","6":"0.20000000","7":"1.995975e-01","8":"3.011863e-02"},{"1":"178","2":"566","3":"0.20598955","4":"0.20000000","5":"2.029114e-01","6":"0.20000000","7":"2.043580e-01","8":"3.235208e-02"},{"1":"179","2":"468","3":"0.50000000","4":"0.50000000","5":"5.000000e-01","6":"0.50000000","7":"5.000000e-01","8":"5.000000e-01"},{"1":"180","2":"469","3":"0.14754334","4":"0.20000000","5":"2.726138e-02","6":"0.20000000","7":"3.110157e-02","8":"1.845780e-04"},{"1":"180","2":"470","3":"0.18950332","4":"0.20000000","5":"3.873392e-02","6":"0.20000000","7":"4.570137e-02","8":"5.016426e-04"},{"1":"180","2":"471","3":"0.37004459","4":"0.20000000","5":"3.029051e-01","6":"0.20000000","7":"7.298552e-01","8":"9.230637e-01"},{"1":"180","2":"472","3":"0.15943879","4":"0.20000000","5":"3.184560e-02","6":"0.20000000","7":"3.659521e-02","8":"2.811359e-04"},{"1":"180","2":"473","3":"0.13346996","4":"0.20000000","5":"5.809808e-04","6":"0.20000000","7":"6.738064e-04","8":"1.180510e-08"},{"1":"181","2":"474","3":"0.07643101","4":"0.04681695","5":"3.653535e-05","6":"0.04681695","7":"3.975081e-05","8":"9.963415e-12"},{"1":"181","2":"475","3":"0.10231247","4":"0.16332348","5":"4.822822e-05","6":"0.16332348","7":"5.403929e-05","8":"2.147003e-11"},{"1":"181","2":"476","3":"0.61733005","4":"0.56508371","5":"5.048271e-01","6":"0.56508371","7":"9.800570e-01","8":"9.999409e-01"},{"1":"181","2":"477","3":"0.12603303","4":"0.20000000","5":"4.654535e-03","6":"0.20000000","7":"5.324583e-03","8":"2.096572e-06"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>


\
Following table shows the **Brier scores** for each question-day pair per aggregation method used. The final two columns, **`Best_Method`** and **`Ranked_Methods`**, show the best performing method (i.e. method with the lowest **Brier score**) and the order of the method performance, respectively:



<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Day"],"name":[1],"type":["IDate"],"align":["right"]},{"label":["discover question id"],"name":[2],"type":["int"],"align":["right"]},{"label":["Mean"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["Median"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["Geo_Mean"],"name":[5],"type":["dbl"],"align":["right"]},{"label":["Trim_Mean"],"name":[6],"type":["dbl"],"align":["right"]},{"label":["Geo_Mean_Odds"],"name":[7],"type":["dbl"],"align":["right"]},{"label":["Best_Method"],"name":[8],"type":["chr"],"align":["left"]},{"label":["Ranked_Methods"],"name":[9],"type":["chr"],"align":["left"]}],"data":[{"1":"2018-03-07","2":"179","3":"0.2500000","4":"0.25000000","5":"0.2500000","6":"0.25000000","7":"0.25000000","8":"Mean","9":"Mean > Median > Geo_Mean > Trim_Mean > Geo_Mean_Odds"},{"1":"2018-03-08","2":"179","3":"0.2228512","4":"0.24546198","5":"0.3030460","6":"0.24546198","7":"0.11156837","8":"Geo_Mean_Odds","9":"Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean"},{"1":"2018-03-09","2":"179","3":"0.2261973","4":"0.24385379","5":"0.3143616","6":"0.24385379","7":"0.15471319","8":"Geo_Mean_Odds","9":"Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean"},{"1":"2018-03-10","2":"179","3":"0.1684095","4":"0.18540150","5":"0.5884959","6":"0.18540150","7":"0.36794108","8":"Mean","9":"Mean > Median > Trim_Mean > Geo_Mean_Odds > Geo_Mean"},{"1":"2018-03-11","2":"179","3":"0.1430425","4":"0.09319259","5":"0.5579004","6":"0.09319259","7":"0.23644475","8":"Median","9":"Median > Trim_Mean > Mean > Geo_Mean_Odds > Geo_Mean"},{"1":"2018-03-12","2":"179","3":"0.1256116","4":"0.09319259","5":"0.5391778","6":"0.09319259","7":"0.18181828","8":"Median","9":"Median > Trim_Mean > Mean > Geo_Mean_Odds > Geo_Mean"},{"1":"2018-03-13","2":"179","3":"0.1289142","4":"0.15939347","5":"0.1471616","6":"0.15939347","7":"0.10273736","8":"Geo_Mean_Odds","9":"Geo_Mean_Odds > Mean > Geo_Mean > Median > Trim_Mean"},{"1":"2018-03-14","2":"179","3":"0.1675982","4":"0.16787305","5":"0.2871592","6":"0.16787305","7":"0.20155305","8":"Mean","9":"Mean > Median > Trim_Mean > Geo_Mean_Odds > Geo_Mean"},{"1":"2018-03-15","2":"179","3":"0.1704827","4":"0.19447678","5":"0.1878166","6":"0.19447678","7":"0.16268998","8":"Geo_Mean_Odds","9":"Geo_Mean_Odds > Mean > Geo_Mean > Median > Trim_Mean"},{"1":"2018-03-16","2":"179","3":"0.1429946","4":"0.14869955","5":"0.1547969","6":"0.14869955","7":"0.13784569","8":"Geo_Mean_Odds","9":"Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean"},{"1":"2018-03-17","2":"179","3":"0.1560561","4":"0.16392004","5":"0.2294760","6":"0.16392004","7":"0.15520474","8":"Geo_Mean_Odds","9":"Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean"},{"1":"2018-03-18","2":"179","3":"0.1400550","4":"0.14541115","5":"0.2096699","6":"0.14541115","7":"0.10758649","8":"Geo_Mean_Odds","9":"Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean"},{"1":"2018-03-19","2":"179","3":"0.1289182","4":"0.13690000","5":"0.1962357","6":"0.13690000","7":"0.10661963","8":"Geo_Mean_Odds","9":"Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean"},{"1":"2018-03-20","2":"179","3":"0.1261416","4":"0.12905148","5":"0.1919909","6":"0.12905148","7":"0.10392824","8":"Geo_Mean_Odds","9":"Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean"},{"1":"2018-03-21","2":"179","3":"0.1206404","4":"0.13690000","5":"0.1793264","6":"0.13690000","7":"0.09924134","8":"Geo_Mean_Odds","9":"Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean"},{"1":"2018-03-22","2":"179","3":"0.1249560","4":"0.14331848","5":"0.1817539","6":"0.14331848","7":"0.10536091","8":"Geo_Mean_Odds","9":"Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean"},{"1":"2018-03-23","2":"179","3":"0.1331086","4":"0.15210000","5":"0.1900316","6":"0.15210000","7":"0.11335986","8":"Geo_Mean_Odds","9":"Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean"},{"1":"2018-03-24","2":"179","3":"0.1302167","4":"0.14964689","5":"0.1867844","6":"0.14964689","7":"0.10801517","8":"Geo_Mean_Odds","9":"Geo_Mean_Odds > Mean > Median > Trim_Mean > Geo_Mean"},{"1":"2018-03-25","2":"179","3":"0.1559943","4":"0.17675763","5":"0.1653350","6":"0.17675763","7":"0.14863162","8":"Geo_Mean_Odds","9":"Geo_Mean_Odds > Mean > Geo_Mean > Median > Trim_Mean"},{"1":"2018-03-26","2":"179","3":"0.1343479","4":"0.15702722","5":"0.1463788","6":"0.15702722","7":"0.11754518","8":"Geo_Mean_Odds","9":"Geo_Mean_Odds > Mean > Geo_Mean > Median > Trim_Mean"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

\
The  following table shows the ordered summary of the method performance, along with the percentage of question-day pairs in which the method outperformed the rest:
<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["Best_Method"],"name":[1],"type":["chr"],"align":["left"]},{"label":["N"],"name":[2],"type":["int"],"align":["right"]},{"label":["Percentage"],"name":[3],"type":["dbl"],"align":["right"]}],"data":[{"1":"Geo_Means_Odds_Extremised","2":"9","3":"42.857143"},{"1":"Mean","2":"6","3":"28.571429"},{"1":"Median","2":"4","3":"19.047619"},{"1":"Geo_Mean","2":"1","3":"4.761905"},{"1":"Geo_Mean_Odds","2":"1","3":"4.761905"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>
\
The best performing aggregation method was the extremised geometric mean of odds (42.86% of PDPs), followed by the arithmetic mean (28.57% of PDPs), median (19.05% of PDPs), the geometric mean (4.76% of PDPs), and the geometric mean of odds (4.76% of PDPs). The trimmed arithmetic mean never outperformed the other methods. Evidently, the extremised geometric mean of odds outperformed the other methods and thus was an clear improvement in the prediction evaluation. The working principle behind it is a modification of geometric mean of odds, where the geometric mean of odds is raised to the power of an extremising parameter, in this case equal to 2.5. This method is a correction for forecaster under-confidence. In the present dataset it was able to outcompete the other methods, however, it is likely that utilising it on a different dataset, which would contain less forecaster under-confidence would make it non-optimal.

# Conclusion {#conclusion}
The extremised geometric mean of odds provided the best aggregation performance, suggesting that penalising under-confident predictions can improve forecasting accuracy. However, the effectiveness of this method may vary depending on the dataset's structure and the forecasters' behaviour.
