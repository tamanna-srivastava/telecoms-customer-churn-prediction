# Telecom Customer Churn Prediction

This project aims at predicting which customers are more likely to churn.

**The key finding:** Customers who regularly exceed their call allowances are more likely to leave. The highest-risk segment churns at around 79%, against a base rate of 49%.

---

### Business Problem

BangorTelco is a fictional telecoms provider facing a major churn problem. Almost half of its customers (49%) are churning and switching to other network providers. This project aims at identifying which of its customers are likely to churn so that retention efforts can be focused towards them.

**The business question:** Which customers are most likely to leave?

---

### Data

20,000 customer records, 14 variables, each record represents a unique customer.

| Variable | Description |
|---|---|
| `CUSTOMERID` | Unique identifier |
| `YEAR_BIRTH` | Year of birth, converted to AGE where needed |
| `COLLEGE` | Whether the customer attended college |
| `INCOME` | Annual income (£) |
| `OVERAGE` | Average monthly charge above the plan allowance (£) |
| `LEFTOVER` | Average share of plan minutes left unused each month (%) |
| `HOUSE` | Estimated house value (£) |
| `HANDSET_PRICE` | Price of the customer's handset (£) |
| `OVER_15MINS_CALLS_PER_MONTH` | Number of calls longer than 15 minutes per month |
| `AVERAGE_CALL_DURATION` | Average call length (minutes) |
| `REPORTED_SATISFACTION` | Self-reported satisfaction level |
| `REPORTED_USAGE_LEVEL` | Self-reported usage level |
| `CONSIDERING_CHANGE_OF_PLAN` | Self-reported intent to change plan |
| `LEAVE` | Target variable: LEAVE or STAY |

---

### Data Cleaning

- No duplicate records or missing values.
- Two kinds of data errors were found - birth years before 1924 and negative overage values.
- 19,996 records remained after cleaning.
- Skewness and kurtosis were checked across the numeric variables. None of the variables were strongly skewed, and the distributions sit slightly flatter than a normal distribution. This was checked for the purpose of descriptive analysis and none of the methods employed have a normality requirement.

---

### Methodology

**Decision Tree** - This lays down the rules to predict whether a customer will churn or not based on certain attributes. The tree finds the most important distinguishing factors and gives the likelihood of churning for each group it creates. Retention efforts can then be targeted at the groups the tree flags as the riskiest.

Eight variants were built, varying the train/test ratio (70:30 and 80:20), the sampling method (simple random and stratified) and the package and control parameters (`tree` and `rpart`). Out-of-sample accuracy ranged from 66.89% to 71.23%. These were built to assess sample-wise variation in the results.

The tree used here is not the highest scoring one. The highest scoring tree reached 71.23% but has fourteen terminal nodes and a plot that cannot be read. The one selected scores 68.68% with seven nodes. Those two were fitted on different splits, so part of the gap between them is split variation rather than a real difference in quality, but some accuracy was given up either way to get a set of rules a marketing team can actually apply.

**Logistic regression** - This predicts the probability of a customer churning rather than a yes or no label. The business can then set its own threshold on those probabilities. For example, it can target the riskiest 20% rather than everyone above 50%.

**K-Nearest Neighbours** - This predicts whether a customer is likely to churn based on their similarity to customers who have already left. While logistic regression assumes the relationship between the predictors and churn is linear, KNN assumes nothing and simply looks at similar customers. So, a KNN model can show patterns within the customers that a linear model may miss.

**K-Means Clustering** - This checks for natural groupings within the customers. It is an unsupervised method and never sees the churn label, so it works as an independent check on the results from the supervised models. The project finds these groupings and then checks how churn varies within them.

PCA is used to plot the KNN results and the clusters in two dimensions. The models themselves were fitted on the eight standardised variables, not on the principal components.

---

### Findings and Results

| Model | Split | Accuracy | Sensitivity | Specificity |
|---|---|---|---|---|
| Decision tree (rpart, 7 nodes) | Stratified 80:20 | 68.68% | 82.07% | 55.69% |
| Logistic regression | Random 80:20 | 64.85% | 61.24% | 68.51% |
| KNN (k = 41) | Stratified 70:30 | 68.27% | 69.16% | 67.41% |

Sensitivity here means the share of actual leavers the model caught. Specificity means the share of actual stayers it correctly left alone.

**These three models were fitted on different test sets, so the accuracies are not directly comparable to one another.** Some of the difference between them comes from the splits rather than from the models themselves. Cross-validation over identical folds would give a cleaner comparison.



**Decision tree -** House value splits the base first, at £605,000. Below that threshold, the next split is overage. **Customers overcharged £109 or more per month have only a 21% chance of staying.** That is 22% of the entire customer base churning at about 79%, and it is the highest-risk segment in the model. Within the customers paying less than £109 in overage, those with 25% or more of their minutes left unused each month have a 39% chance of staying. Above £605,000 in house value the split is on income, and the direction is counterintuitive. Customers earning £100,000 or more have only a 42% chance of staying, while those earning under £100,000 form the most loyal segment in the model at 82%.

**Logistic regression -** Accuracy 64.85%, AUC 0.703. All six predictors were statistically significant, but overage and house value are far more important than the rest. An AUC of 0.703 means that if you pick one customer who left and one who stayed at random, the model gives the higher churn probability to the actual leaver about 70% of the time.

**KNN -** Accuracy 68.27%, and the most balanced sensitivity and specificity of the three models. It beats logistic regression on both accuracy and sensitivity. Sensitivity is the metric that matters for this business question, since the objective is catching leavers.

**Clustering -** Three clusters, chosen using the elbow method.

| Cluster | Size | Churn rate | Profile |
|---|---|---|---|
| 1 | 7,099 | 38.8% | Lower income, cheaper handsets, low overage. Largest and most loyal. |
| 2 | 4,068 | 50.7% | Higher income, more expensive handsets, low overage. Sits at the average. |
| 3 | 4,831 | 63.4% | Overage above roughly £130 per month, frequent long calls, spans all income levels. |

Overall churn rate across the training set is 49.3%. The clustering model never sees the LEAVE column, yet the segment it isolates on usage behaviour alone turns out to have the highest churn rate in the dataset.

The full model output, the node-by-node reading of the tree, the diagnostics and the comparison of all eight trees are in the knitted report.

---

### The one insight

Three different methods provide the same insight. The decision tree makes overage its strongest split. Logistic regression gives overage the strongest statistical signal of any predictor. K-means isolates a high-overage segment that churns at 63.4%.

**Customers who consistently exceed their allowance are the ones who leave.** This shows that the customers with high overcharge are among the riskiest churners. This indicates that this set of customers might be on the wrong plan and require a targeted retention strategy designed for them.

---

### Recommendations

1. **Review the price plans of high-overage customers first.** Cluster 3 is 30% of the customer base and churns at 63.4%. It spans all income levels, so it cannot be targeted by income. These customers can be found by usage data.
2. **Move high-leftover customers to smaller plans.** Customers with 25% or more of their minutes unused each month churn at around 61%. They are paying for capacity they do not use.
3. **Do not assume affluent customers are entirely loyal.** High house value combined with income above £100,000 is a churn risk, not a loyalty signal. This segment needs a different retention approach, likely premium benefits rather than price cuts.


### Limitations

- **The relationship between overage and churn is associative, not causal.** A price plan review should be tested on a sample before being rolled out to the whole segment.
- **The clusters are imposed, not discovered.** The elbow plot is shallow and the cluster boundaries in PCA space are sharp straight lines with no gaps, which indicates k-means is dividing a continuous spread of customers rather than recovering groups that exist naturally.
- **The three models used different train/test splits,** so their accuracies cannot be compared directly.
- **The self-reported fields were not used in the final models.** REPORTED_SATISFACTION, REPORTED_USAGE_LEVEL and CONSIDERING_CHANGE_OF_PLAN are categorical and were excluded from the distance-based and regression models.

---

---

### Repository structure

```
.
├── README.md                          This file
├── Telecom_Customer_Prediction.Rmd    Full analysis source
├── Telecom_Customer_Prediction.pdf    Knitted report with all output
└── data/
    └── Telecom_Customers.csv          Source dataset
```

### Running the analysis

Open `Telecom_Customer_Prediction.Rmd` in RStudio and knit. The data path is relative, so the `data/` folder must stay where it is.

Packages used: `tidyverse`, `caret`, `rpart`, `rpart.plot`, `tree`, `class`, `cluster`, `pROC`, `ggplot2`, `moments`, `corrplot`, `MASS`.

All random operations are seeded with `set.seed(100)`, and age is calculated against a hardcoded year, so the results reproduce exactly regardless of when the file is knitted.

---

### Notes on this version

This project began as an MSc assignment and was reworked before publishing. The changes include the following among others:

- Replaced a hardcoded database connection with a relative CSV path, so the analysis runs anywhere.
- Corrected the train/test sampling, which had been drawing with replacement.
- Fixed a data leakage problem in the KNN and clustering tasks. The test data had been scaled using its own means and standard deviations instead of the training set's, which lets information from the test set affect the transformation.
- Stopped describing the AUC figure as an accuracy figure.

---

**Author:** Tamanna Srivastava
**Tools:** R, R Markdown
**LinkedIn:** [tamanna-srivastava](https://www.linkedin.com/in/tamanna-srivastava-/)
