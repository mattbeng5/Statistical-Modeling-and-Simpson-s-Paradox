# Statistical Modelling and Simpson's Paradox

## Table of Contents
- [Project Overview](#project-overview)
- [Data Sources](#data-sources)
- [Tools](#tools)
- [Data Cleaning and Preparation](#data-cleaning-and-preparation)
- [Exploratory Data Analysis](#exploratory-data-analysis)
- [Chi-squared Test](#chi-squared-test)
- [Logistic Regression Model](#logistic-regression-model)
- [Conclusion](#conclusion)



### Project Overview

The objective of this project is to create a logistic regression model to analyze effectiveness of kidney stone treatements. 

In 1986, a group of urologists in London published a research paper in The British Medical Journal that compared the effectiveness of two different methods to remove kidney stones. Treatment A was open surgery (invasive), and treatment B was percutaneous nephrolithotomy (less invasive). When they looked at the results from 700 patients, treatment B had a higher success rate. However, when they only looked at the subgroup of patients different kidney stone sizes, treatment A had a better success rate. What is going on here? This known statistical phenomenon is called Simpon’s paradox. Simpon's paradox occurs when trends appear in subgroups but disappear or reverse when subgroups are combined.

### Data Sources
Kidney Stone Data: The dataset for this analysis is the "kidney_stone_data.csv" file which contains data on two different kidney stone treatments. 

### Tools
- R (tidyverse)

###  Data Cleaning and Preparation
In the initial data preparation phase, we performed following tasks:
1. Data loading and inspection
2. No cleaning and formatting was needed, no missing values

### Exploratory Data Analysis
EDA involved exploring the kidney data to answer the following questions:
- Which treatment has a higher success rate?
- Does kidney stone size effect the success of each treatment?

```R
# Import tidyverse
library(tidyverse)

# Import CSV File
kidney_data <- read_csv("/kaggle/input/kidney-stone-data/kidney_stone_data.csv")

head(kidney_data, 5)
glimpse(kidney_data)
```
There are 3 columns in this dataset. The treatment column represents which treatment is used, either A or B. The stone_size column represents whether the stone size was large or small. The success column represents the success or failure of treatment with 1 being successful and 0 being unsuccesful.

The next step is to determine which treatment has a higher success rate and if the success is impacted by stone size.
```R
 # Compute counts and proportions of successes and failures for each treatment group
treatment_group <-
    kidney_data %>%
    group_by(treatment, success) %>%
    summarize(Count = n()) %>%
    mutate(Proportion = round(Count/sum(Count), 2))

# Compute counts and proportions of successes and failures, grouping by treatment and stone size
treatment_stone_group <-
    kidney_data %>%
    group_by(treatment, stone_size, success) %>%
    summarize(Count = n()) %>%
    mutate(Proportion = round(Count/sum(Count), 2))
```
Not accounting for stone size, both treatments have a high success rate. Treatment A has a 78% success rate and treatment B has a 83% success rate. 

After stratifying by stone size, we can see that for both treatment groups, the smaller stone sizes have a much higher success rate than failure. Using treatment A on a patient with small kidney stones leads to a 93% success rate and using treatment B on a patient with small kidney stones leads to a 87% success rate. Comparing to the overall success rate, we see a much higher jump in treatment A(15 points) than treatment B(5 points).

What is interesting about these two tables is that when just stratifying by treatment group, treatment B has a higher success rate, but when subgrouping by stone size, treatment A has higher success rates for both large and small stones. Lets visualize this to try and get a better understanding.

```R
# Bar plot visualizing the distribution of stone sizes across treatment groups
size_group <- 
    kidney_data %>%
    group_by(treatment, stone_size) %>%
    summarize(Count = n()) %>%
    mutate(Proportion = round(Count/sum(Count), 2))

size_plot <-
    ggplot(size_group, aes(treatment, Count, fill = stone_size)) +
    geom_col(position = position_dodge()) +
    scale_fill_brewer(palette = "Pastel2") +
    labs(title = "Distribution of Stone Sizes by Treatment Group",
            x = "Treatment Group",
            y = "Count", 
            fill = "Stone Size") + theme_minimal()
```
This population was evenly split with 350 patients in each treatment group, but they distribution of kidney stone size is incredibly disproportionate across the two groups. The majority of patients undergoing treatment A(75%) had large kidney stones while the majority of patients undergoing treatment B(77%) had small kidney stones.

Stone size may be a confounding variable in this study, we can assess this with a Chi-squared test. The null hypothesis for this test will be that these variables are not associated while the alternative hypothesis is that these variables do have an association. The significance level will be 0.05.

### Chi-squared Test
```R
# Chi-squared test

chi_test <- chisq.test(kidney_data$treatment, kidney_data$stone_size)
chi_test
```
With the p-value being below the significance level, we will reject the null hypothesis and accept that there is an association between stone size and treatment group. Our next step is to perform a logistic regression to better understand how stone size and treatment type infleunce the success of treatment, we will again be using a significance level of 0.05.

### Logistic Regression Model
```R
# Import broom package 
library(broom)

# Convert variable columns to factors
kidney_data <-
    kidney_data %>%
    mutate(treatment=factor(treatment), stone_size=factor(stone_size))

# Multiple Logistic Regression Model
logistic_model <- 
    glm(success ~ treatment + stone_size, data = kidney_data, family = binomial(link = "logit"))

tidy_model <-
    tidy(logistic_model, exponentiate = TRUE)
```
### Conclusion
-Intercept(Treatment A, Large Stone): The estimate of 2.81 indicates that for the baseline group, the odds of treatment success are about 2.81
-treatmentB(Treatment A vs Treatment B): The estimate of .70 indicates that this group has 30% lower odds of success than the Intercept group. With the p-value being large than 0.05, we can not conclude that changing treatment has a statistically significant impact on success.
-stone_sizesmall(Small vs Large stone): The estimate of 3.53 indicates that regardless of treatment type, patients with small stones have a 3.53 higher odds of success than patients with large stones. The p-value being below 0.05 indicates that this is statistically significant.
In conclusion, treatment A and treatment B do not have a significant effect on whether the removal of kidney stones is effective, it is the size of the kidney stone that has a significant effect on the procedure success. This suggests that the decreased odds of success in treatment B may be due to random variation rather than the treatment.
