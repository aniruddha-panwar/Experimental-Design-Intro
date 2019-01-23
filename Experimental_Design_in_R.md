---
title: "Experimental Design"
author: "Ani Panwar"
date: "January 22, 2019"
output: 
  html_document:
    keep_md: TRUE
    highlight: haddock
    number_sections: yes
    theme: cerulean
    toc: TRUE
---




## Steps of Experiment

* Planning - Formulate hypothesis/question(s); dependent variable; independent variables; population of interest
* Design - choosing design (logit regression, factorial design)
* Analysis

Randomization, Replication and Blocking to keep bias low and asses variability of outcome.

Randomization helps spread any variability that exists naturally between subjects evenly across groups.
Replication repeats experiment to fully asses variability. Need to repeat an experiment with an adequate number of subjects to achieve an acceptable statistical power.
Blocking helps control variability by making treatment groups more alike. Inside the groups the differences will be minimal; Across groups the differences will be larger.  
  
Dataset - `ToothGrowth`  
Has results of study that examined the effect of three different doses of Vitamin C on the length of the odontoplasts cells (responsible for tooth growth) in 60 guinea pigs.  
`len` variable is the outcome variable corresponding to the tooth length.  
Suppose its known that the average length of the guinea pigs odontoplasts is 18 micrometers, lets conduct a t-test to check that the mean of `len` is not equal to 18.


```r
# Load dataset as a dataframe
data(ToothGrowth)


t.test(x=ToothGrowth$len, alternative = "two.sided", mu = 18)
```

```
## 
## 	One Sample t-test
## 
## data:  ToothGrowth$len
## t = 0.82361, df = 59, p-value = 0.4135
## alternative hypothesis: true mean is not equal to 18
## 95 percent confidence interval:
##  16.83731 20.78936
## sample estimates:
## mean of x 
##  18.81333
```
In the above two sided t-test, the null hypothesis is that the mean of `len` is not equal to 18. Given the high p-value we can say that we fail to reject the Null hypothesis. Thus there is evidence to believe that the mean length of the odontoplasts is 18 micometers.  

### Randomization

In the experiment that yielded the ToothGrowth dataset, guinea pigs were randomized to receive Vitamin C either through orange juice or ascorbic acid, indicated in the dataset by the `supp` variable. It's natural to wonder if there is a difference in tooth length by supplement type - a question that a t-test can also answer!


```r
# t-test to determine if there is a difference in toothlength based on supplement
ToothGrowth_ttest <- t.test(len~supp, data =  ToothGrowth)

#Load Broom
library(broom)

# Tidy ToothGrowth_ttest
tidy(ToothGrowth_ttest)
```

```
## # A tibble: 1 x 10
##   estimate estimate1 estimate2 statistic p.value parameter conf.low
##      <dbl>     <dbl>     <dbl>     <dbl>   <dbl>     <dbl>    <dbl>
## 1     3.70      20.7      17.0      1.92  0.0606      55.3   -0.171
## # ... with 3 more variables: conf.high <dbl>, method <chr>,
## #   alternative <chr>
```
  
Given the p-value of around 0.06 (above 0.05; can't reject null hypo), there seems to be no evidence to support the alternate hypothesis that there's a difference in mean tooth length by supplement type.  
  
### Replication

`count()` takes group variables as parameter. 

```r
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
ToothGrowth %>% count(supp,dose)
```

```
## # A tibble: 6 x 3
##   supp   dose     n
##   <fct> <dbl> <int>
## 1 OJ      0.5    10
## 2 OJ      1      10
## 3 OJ      2      10
## 4 VC      0.5    10
## 5 VC      1      10
## 6 VC      2      10
```
  
In the above results we can see that the sample size for each combo of dose and supplement is 10; which is less but is deemed sufficient for this experiment.  
  
### Blocking  

Though this is not true, suppose the supplement type is actually a nuisance factor we'd like to control for by blocking, and we're actually only interested in the effect of the dose of Vitamin C on guinea pig tooth growth. If we block by supplement type, we create groups that are more similar, in that they'll have the same supplement type, allowing us to examine only the effect of dose on tooth length.

We'll use the `aov()` function to examine this. `aov()` creates a linear regression model by calling `lm()` and examining results with `anova()` all in one function call. To use `aov()`, we'll still need functional notation, as with the randomization exercise, but this time the formula should be `len ~ dose + supp` to indicate we've blocked by supplement type.


```r
library(ggplot2)
# Check effect of dos eon len visually
ggplot(ToothGrowth, aes(x = factor(dose), y = len)) + geom_boxplot()
```

![](Experimental_Design_in_R_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

```r
# Create ToothGrowth_aov
ToothGrowth_aov <- aov(len ~ factor(dose) + supp, data = ToothGrowth)

# Examine ToothGrowth_aov with summary()
summary(ToothGrowth_aov)
```

```
##              Df Sum Sq Mean Sq F value   Pr(>F)    
## factor(dose)  2 2426.4  1213.2   82.81  < 2e-16 ***
## supp          1  205.4   205.4   14.02 0.000429 ***
## Residuals    56  820.4    14.7                     
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```
  
Small p value of `dose` seems to suggest a driving effect on `len`.  
  
## Hypothesis Testing
  
### One Sided vs. Two Sided Tests

### Power & Sample Size Calculations

  
