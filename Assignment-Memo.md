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







<!-- Add comments to this chunk!!!! -->


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
{"columns":[{"label":["discover question id"],"name":[1],"type":["int"],"align":["right"]},{"label":["discover answer id"],"name":[2],"type":["int"],"align":["right"]},{"label":["Mean"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["Median"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["Geo_Mean"],"name":[5],"type":["dbl"],"align":["right"]},{"label":["Trim_Mean"],"name":[6],"type":["dbl"],"align":["right"]},{"label":["Geo_Mean_Odds"],"name":[7],"type":["dbl"],"align":["right"]},{"label":["Geo_Means_Odds_Extremised"],"name":[8],"type":["dbl"],"align":["right"]}],"data":[{"1":"177","2":"557","3":"0.20000000","4":"0.20000000","5":"2.000000e-01","6":"0.20000000","7":"2.000000e-01","8":"3.030303e-02"},{"1":"177","2":"558","3":"0.20000000","4":"0.20000000","5":"2.000000e-01","6":"0.20000000","7":"2.000000e-01","8":"3.030303e-02"},{"1":"177","2":"559","3":"0.20000000","4":"0.20000000","5":"2.000000e-01","6":"0.20000000","7":"2.000000e-01","8":"3.030303e-02"},{"1":"177","2":"560","3":"0.20000000","4":"0.20000000","5":"2.000000e-01","6":"0.20000000","7":"2.000000e-01","8":"3.030303e-02"},{"1":"177","2":"561","3":"0.20000000","4":"0.20000000","5":"2.000000e-01","6":"0.20000000","7":"2.000000e-01","8":"3.030303e-02"},{"1":"178","2":"562","3":"0.19781319","4":"0.20000000","5":"1.886879e-01","6":"0.20000000","7":"1.904701e-01","8":"2.615029e-02"},{"1":"178","2":"563","3":"0.19781319","4":"0.20000000","5":"1.886879e-01","6":"0.20000000","7":"1.904701e-01","8":"2.615029e-02"},{"1":"178","2":"564","3":"0.19875944","4":"0.20000000","5":"1.981816e-01","6":"0.20000000","7":"1.983119e-01","8":"2.953508e-02"},{"1":"178","2":"565","3":"0.19962463","4":"0.20000000","5":"1.995888e-01","6":"0.20000000","7":"1.995975e-01","8":"3.011863e-02"},{"1":"178","2":"566","3":"0.20598955","4":"0.20000000","5":"2.029114e-01","6":"0.20000000","7":"2.043580e-01","8":"3.235208e-02"},{"1":"179","2":"468","3":"0.50000000","4":"0.50000000","5":"5.000000e-01","6":"0.50000000","7":"5.000000e-01","8":"5.000000e-01"},{"1":"180","2":"469","3":"0.14754334","4":"0.20000000","5":"2.726138e-02","6":"0.20000000","7":"3.110157e-02","8":"1.845780e-04"},{"1":"180","2":"470","3":"0.18950332","4":"0.20000000","5":"3.873392e-02","6":"0.20000000","7":"4.570137e-02","8":"5.016426e-04"},{"1":"180","2":"471","3":"0.37004459","4":"0.20000000","5":"3.029051e-01","6":"0.20000000","7":"7.298552e-01","8":"9.230637e-01"},{"1":"180","2":"472","3":"0.15943879","4":"0.20000000","5":"3.184560e-02","6":"0.20000000","7":"3.659521e-02","8":"2.811359e-04"},{"1":"180","2":"473","3":"0.13346996","4":"0.20000000","5":"5.809808e-04","6":"0.20000000","7":"6.738064e-04","8":"1.180510e-08"},{"1":"181","2":"474","3":"0.07643101","4":"0.04681695","5":"3.653535e-05","6":"0.04681695","7":"3.975081e-05","8":"9.963415e-12"},{"1":"181","2":"475","3":"0.10231247","4":"0.16332348","5":"4.822822e-05","6":"0.16332348","7":"5.403929e-05","8":"2.147003e-11"},{"1":"181","2":"476","3":"0.61733005","4":"0.56508371","5":"5.048271e-01","6":"0.56508371","7":"9.800570e-01","8":"9.999409e-01"},{"1":"181","2":"477","3":"0.12603303","4":"0.20000000","5":"4.654535e-03","6":"0.20000000","7":"5.324583e-03","8":"2.096572e-06"},{"1":"181","2":"478","3":"0.07789344","4":"0.04050957","5":"2.071504e-04","6":"0.04050957","7":"2.256644e-04","8":"7.654251e-10"},{"1":"184","2":"485","3":"0.15597515","4":"0.20000000","5":"1.022342e-02","6":"0.20000000","7":"1.200652e-02","8":"1.627986e-05"},{"1":"184","2":"486","3":"0.21482067","4":"0.20000000","5":"1.626626e-01","6":"0.20000000","7":"1.727444e-01","8":"1.953628e-02"},{"1":"184","2":"487","3":"0.16711660","4":"0.20000000","5":"1.106950e-02","6":"0.20000000","7":"1.315611e-02","8":"2.052053e-05"},{"1":"184","2":"488","3":"0.16775891","4":"0.20000000","5":"3.259371e-02","6":"0.20000000","7":"3.779298e-02","8":"3.056495e-04"},{"1":"184","2":"489","3":"0.29432868","4":"0.20000000","5":"2.646013e-01","6":"0.20000000","7":"2.836489e-01","8":"8.979967e-02"},{"1":"185","2":"490","3":"0.09123619","4":"0.06643407","5":"4.236629e-05","6":"0.06643407","7":"4.687318e-05","8":"1.504395e-11"},{"1":"185","2":"491","3":"0.66642607","4":"0.81766008","5":"5.410487e-01","6":"0.81766008","7":"9.867337e-01","8":"9.999790e-01"},{"1":"185","2":"492","3":"0.09963858","4":"0.09332315","5":"3.769912e-03","6":"0.09332315","7":"4.185428e-03","8":"1.145257e-06"},{"1":"185","2":"493","3":"0.07342517","4":"0.01683593","5":"3.226902e-05","6":"0.01683593","7":"3.500016e-05","8":"7.247913e-12"},{"1":"185","2":"494","3":"0.06927399","4":"0.00574676","5":"2.620990e-05","6":"0.00574676","7":"2.830721e-05","8":"4.263569e-12"},{"1":"187","2":"500","3":"0.13346177","4":"0.20000000","5":"5.781956e-04","6":"0.20000000","7":"6.705728e-04","8":"1.166388e-08"},{"1":"187","2":"501","3":"0.18618513","4":"0.20000000","5":"3.813584e-02","6":"0.20000000","7":"4.484645e-02","8":"4.774522e-04"},{"1":"187","2":"502","3":"0.36070619","4":"0.20000000","5":"2.982537e-01","6":"0.20000000","7":"7.220867e-01","8":"9.158376e-01"},{"1":"187","2":"503","3":"0.18618513","4":"0.20000000","5":"3.813584e-02","6":"0.20000000","7":"4.484645e-02","8":"4.774522e-04"},{"1":"187","2":"504","3":"0.13346177","4":"0.20000000","5":"5.781956e-04","6":"0.20000000","7":"6.705728e-04","8":"1.166388e-08"},{"1":"188","2":"505","3":"0.06640199","4":"0.00000000","5":"1.920322e-06","6":"0.00000000","7":"2.067879e-06","8":"6.149151e-15"},{"1":"188","2":"506","3":"0.18210448","4":"0.20000000","5":"6.675406e-03","6":"0.20000000","7":"8.197918e-03","8":"6.211460e-06"},{"1":"188","2":"507","3":"0.56334330","4":"0.54637197","5":"4.573375e-01","6":"0.54637197","7":"9.724617e-01","8":"9.998651e-01"},{"1":"188","2":"508","3":"0.11673038","4":"0.13493833","5":"7.982377e-04","6":"0.13493833","7":"9.157897e-04","8":"2.543807e-08"},{"1":"188","2":"509","3":"0.07141984","4":"0.01291678","5":"1.721159e-04","6":"0.01291678","7":"1.862535e-04","8":"4.736566e-10"},{"1":"192","2":"525","3":"0.20000000","4":"0.20000000","5":"2.000000e-01","6":"0.20000000","7":"2.000000e-01","8":"3.030303e-02"},{"1":"192","2":"526","3":"0.20000000","4":"0.20000000","5":"2.000000e-01","6":"0.20000000","7":"2.000000e-01","8":"3.030303e-02"},{"1":"192","2":"527","3":"0.20000000","4":"0.20000000","5":"2.000000e-01","6":"0.20000000","7":"2.000000e-01","8":"3.030303e-02"},{"1":"192","2":"528","3":"0.20000000","4":"0.20000000","5":"2.000000e-01","6":"0.20000000","7":"2.000000e-01","8":"3.030303e-02"},{"1":"192","2":"529","3":"0.20000000","4":"0.20000000","5":"2.000000e-01","6":"0.20000000","7":"2.000000e-01","8":"3.030303e-02"},{"1":"193","2":"530","3":"0.73266245","4":"0.91173088","5":"6.613738e-01","6":"0.91173088","7":"9.894828e-01","8":"9.999884e-01"},{"1":"193","2":"531","3":"0.14894525","4":"0.07402332","5":"5.099478e-03","6":"0.07402332","7":"6.034830e-03","8":"2.872325e-06"},{"1":"193","2":"532","3":"0.11839230","4":"0.01424580","5":"2.196508e-03","6":"0.01424580","7":"2.526482e-03","8":"3.228769e-07"},{"1":"194","2":"533","3":"0.11976527","4":"0.03150587","5":"3.832181e-03","6":"0.03150587","7":"4.391559e-03","8":"1.292185e-06"},{"1":"194","2":"534","3":"0.10395143","4":"0.02032715","5":"9.048727e-04","6":"0.02032715","7":"1.640916e-03","8":"1.095214e-07"},{"1":"194","2":"535","3":"0.62602705","4":"0.85296040","5":"3.161745e-01","6":"0.85296040","7":"7.052758e-01","8":"8.985642e-01"},{"1":"194","2":"536","3":"0.08360709","4":"0.03826272","5":"5.487460e-04","6":"0.03826272","7":"6.010880e-04","8":"8.871519e-09"},{"1":"194","2":"537","3":"0.06664915","4":"0.00000000","5":"2.014453e-06","6":"0.00000000","7":"2.169783e-06","8":"6.934949e-15"},{"1":"195","2":"538","3":"0.20000000","4":"0.20000000","5":"2.000000e-01","6":"0.20000000","7":"2.000000e-01","8":"3.030303e-02"},{"1":"195","2":"539","3":"0.20000000","4":"0.20000000","5":"2.000000e-01","6":"0.20000000","7":"2.000000e-01","8":"3.030303e-02"},{"1":"195","2":"540","3":"0.20000000","4":"0.20000000","5":"2.000000e-01","6":"0.20000000","7":"2.000000e-01","8":"3.030303e-02"},{"1":"195","2":"541","3":"0.20000000","4":"0.20000000","5":"2.000000e-01","6":"0.20000000","7":"2.000000e-01","8":"3.030303e-02"},{"1":"195","2":"542","3":"0.20000000","4":"0.20000000","5":"2.000000e-01","6":"0.20000000","7":"2.000000e-01","8":"3.030303e-02"},{"1":"196","2":"543","3":"0.17631372","4":"0.00818365","5":"1.615728e-03","6":"0.00818365","7":"2.061068e-03","8":"1.938522e-07"},{"1":"197","2":"544","3":"0.19675799","4":"0.06901614","5":"4.340909e-03","6":"0.06901614","7":"5.655606e-03","8":"2.439803e-06"},{"1":"198","2":"545","3":"0.50000000","4":"0.50000000","5":"5.000000e-01","6":"0.50000000","7":"5.000000e-01","8":"5.000000e-01"},{"1":"199","2":"546","3":"0.17046553","4":"0.00961636","5":"5.698651e-04","6":"0.00961636","7":"7.202172e-04","8":"1.394572e-08"},{"1":"200","2":"567","3":"0.16775160","4":"0.00215612","5":"8.904320e-05","6":"0.00215612","7":"1.122216e-04","8":"1.334483e-10"},{"1":"201","2":"568","3":"0.16924167","4":"0.00611904","5":"1.607227e-03","6":"0.00611904","7":"2.024747e-03","8":"1.854076e-07"},{"1":"202","2":"569","3":"0.18655841","4":"0.04953366","5":"3.282462e-03","6":"0.04953366","7":"4.203972e-03","8":"1.158039e-06"},{"1":"203","2":"570","3":"0.33638223","4":"0.50000000","5":"3.388556e-02","6":"0.50000000","7":"5.119307e-02","8":"6.757559e-04"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>


\
Following table shows the **Brier scores** for each question-day pair per aggregation method used. The final two columns, **`Best_Method`** and **`Ranked_Methods`**, show the best performing method (i.e. method with the lowest **Brier score**) and the order of the method performance, respectively:
<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["discover question id"],"name":[1],"type":["int"],"align":["right"]},{"label":["Mean"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["Median"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["Geo_Mean"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["Trim_Mean"],"name":[5],"type":["dbl"],"align":["right"]},{"label":["Geo_Mean_Odds"],"name":[6],"type":["dbl"],"align":["right"]},{"label":["Geo_Means_Odds_Extremised"],"name":[7],"type":["dbl"],"align":["right"]}],"data":[{"1":"179","2":"0.25000000","3":"2.500000e-01","4":"2.500000e-01","5":"2.500000e-01","6":"2.500000e-01","7":"2.500000e-01"},{"1":"180","2":"0.94276182","3":"8.000000e-01","4":"1.040487e+00","5":"8.000000e-01","6":"1.474881e+00","7":"1.851678e+00"},{"1":"181","2":"1.26649561","3":"1.296193e+00","4":"1.254799e+00","5":"1.296193e+00","6":"1.960461e+00","7":"1.999882e+00"},{"1":"184","2":"0.78353521","3":"8.000000e-01","4":"7.724371e-01","5":"8.000000e-01","6":"7.665541e-01","7":"9.693732e-01"},{"1":"185","2":"0.13971359","3":"4.668702e-02","4":"2.106505e-01","5":"4.668702e-02","6":"1.935160e-04","7":"4.405766e-10"},{"1":"187","2":"0.86269260","3":"8.000000e-01","4":"1.015593e+00","5":"8.000000e-01","6":"1.435740e+00","7":"1.837804e+00"},{"1":"188","2":"1.14019296","3":"1.087021e+00","4":"1.207606e+00","5":"1.087021e+00","6":"1.943918e+00","7":"1.999730e+00"},{"1":"192","2":"0.80000000","3":"8.000000e-01","4":"8.000000e-01","5":"8.000000e-01","6":"8.000000e-01","7":"9.439853e-01"},{"1":"193","2":"1.27510518","3":"1.688889e+00","4":"1.427247e+00","5":"1.688889e+00","6":"1.967049e+00","7":"1.999971e+00"},{"1":"194","2":"0.17643764","3":"2.449049e-02","4":"4.676331e-01","5":"2.449049e-02","6":"8.688468e-02","7":"1.028921e-02"},{"1":"195","2":"0.80000000","3":"8.000000e-01","4":"8.000000e-01","5":"8.000000e-01","6":"8.000000e-01","7":"9.439853e-01"},{"1":"196","2":"0.03108653","3":"6.697213e-05","4":"2.610576e-06","5":"6.697213e-05","6":"4.248001e-06","7":"3.757869e-14"},{"1":"197","2":"0.03871371","3":"4.763228e-03","4":"1.884349e-05","5":"4.763228e-03","6":"3.198588e-05","7":"5.952638e-12"},{"1":"198","2":"0.25000000","3":"2.500000e-01","4":"2.500000e-01","5":"2.500000e-01","6":"2.500000e-01","7":"2.500000e-01"},{"1":"199","2":"0.02905850","3":"9.247438e-05","4":"3.247462e-07","5":"9.247438e-05","6":"5.187128e-07","7":"1.944831e-16"},{"1":"177","2":"0.80000000","3":"8.000000e-01","4":"8.000000e-01","5":"8.000000e-01","6":"8.000000e-01","7":"9.439853e-01"},{"1":"178","2":"0.80442075","3":"8.000000e-01","4":"8.141152e-01","5":"8.000000e-01","6":"8.125465e-01","7":"9.518932e-01"},{"1":"200","2":"0.02814060","3":"4.648853e-06","4":"7.928692e-09","5":"4.648853e-06","6":"1.259369e-08","7":"1.780844e-20"},{"1":"201","2":"0.02864274","3":"3.744265e-05","4":"2.583178e-06","5":"3.744265e-05","6":"4.099601e-06","7":"3.437596e-14"},{"1":"202","2":"0.03480404","3":"2.453583e-03","4":"1.077456e-05","5":"2.453583e-03","6":"1.767338e-05","7":"1.341054e-12"},{"1":"203","2":"0.11315300","3":"2.500000e-01","4":"1.148231e-03","5":"2.500000e-01","6":"2.620731e-03","7":"4.566460e-07"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
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