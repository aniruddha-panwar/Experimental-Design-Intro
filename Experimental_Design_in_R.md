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
  
Data set - `ToothGrowth`  
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
In the above **two sided t-test**, the null hypothesis is that the mean of `len` is not equal to 18. Given the high p-value (0.4135 ; at significance level of 0.05) we can say that we fail to reject the Null hypothesis. Thus there is evidence to believe that the mean length of the odontoplasts is 18 micrometers.  

### Randomization

In the experiment that yielded the ToothGrowth data set, guinea pigs were randomized to receive Vitamin C either through orange juice or ascorbic acid, indicated in the data set by the `supp` variable. It's natural to wonder if there is a difference in tooth length by supplement type - a question that a t-test can also answer!


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
  
Is the question in mind. There are two types of hypothesis - Null and Alternate hypo.  

**NULL HYPOTHESIS** - on the `ToothGrowth` data set would imply that there is no effect of Vit C. dosage or administration type on the length of the odontoplasts.

* there is no change
* no difference between groups
* mean, median, or observed statistic = a definite number

**ALTERNATIVE HYPOTHESIS** - 

* there is a change
* difference between groups
* mean, median, observed statistic is >, < (*one sided test*) or != a number (*two sided test*)  
  
Terms to remember -
* *Effect Size* - standardized difference of a statistic of two groups

### One Sided vs. Two Sided Tests

Recall in the first exercise that we tested to see if the mean of the guinea pigs' teeth in `ToothGrowth` data set was not equal to 18 micrometers. That was an example of a two sided test: it looked to see if the mean of `len` is some other number on either side of 18.  
  
We can also conduct a one sided test, explicitly checking to see if the mean is less than or greater than 18. Whether to use a one or two sided test usually follows from your research question. Want to know if an intervention causes longer tooth growth? One sided, greater than. Want to know if a drug causes the test group to lose more weight? One sided, less than. Simply want to know if there's a difference in test scores between two groups of students? Two sided test.  
  

```r
# Test to see if mean length of odontoplasts is less than 18 - One sided test
t.test(x=ToothGrowth$len, alternative = "less", mu = 18)
```

```
## 
## 	One Sample t-test
## 
## data:  ToothGrowth$len
## t = 0.82361, df = 59, p-value = 0.7933
## alternative hypothesis: true mean is less than 18
## 95 percent confidence interval:
##      -Inf 20.46358
## sample estimates:
## mean of x 
##  18.81333
```

```r
# Test to see if mean length of odontoplasts is less than 18 - One sided test
t.test(x=ToothGrowth$len, alternative = "greater", mu = 18)
```

```
## 
## 	One Sample t-test
## 
## data:  ToothGrowth$len
## t = 0.82361, df = 59, p-value = 0.2067
## alternative hypothesis: true mean is greater than 18
## 95 percent confidence interval:
##  17.16309      Inf
## sample estimates:
## mean of x 
##  18.81333
```
  
In both of the above one-sided tests the high p-value points to insufficient force to reject the Null hypothesis that the mean `len` is 18 micrometers.

### Power & Sample Size Calculations

**POWER** is the probability that the test correctly rejects the NULL hypo given the alternative hypo is TRUE.  
Rule of thumb in statistics is to have 80% power which needs adequate sample size to achieve.  
**SAMPLE SIZE** - as this increases, power increases; because we now have a better description of data.  
  
  
The `pwr` package can help calculate test power (can be achieved via other packages and base as well). Basically if two of *power, effect size* or *sample size* are known, the third can be calculated.
  A call to any `pwr.*()` function returns an object of class `power.htest`, which can then be manipulated in the same way as many different R objects.  
 
 

```r
library(pwr)
```

```
## Warning: package 'pwr' was built under R version 3.5.2
```

```r
# Calculate power for an example

pwr.t.test(n = 100, # number in each group
           d = 0.35, # effect size
           sig.level = 0.10, # significance level
           type = "two.sample", 
           alternative = "two.sided",
           power = NULL) # Leave the value to be caculated as NULL
```

```
## 
##      Two-sample t test power calculation 
## 
##               n = 100
##               d = 0.35
##       sig.level = 0.1
##           power = 0.7943532
##     alternative = two.sided
## 
## NOTE: n is number in *each* group
```

```r
# Calculate sample size for an example

pwr.t.test(n = NULL, 
           d = 0.25, 
           sig.level = 0.05, 
           type = "one.sample", alternative = "greater", 
           power = 0.8)
```

```
## 
##      One-sample t test power calculation 
## 
##               n = 100.2877
##               d = 0.25
##       sig.level = 0.05
##           power = 0.8
##     alternative = greater
```
  In the first example Power is calculated at 79.44%, given 100 sample size in each group, effect size of 0.35 and significance level of 0.10.  
  In the second example above the sample size is calculated at 100.29 given Power of 80%, effect size of 0.25 and significance level of 0.05.  
    
  
##  Single and Multiple Factor Experiments


### ANOVA  
  
Allows  us to test across >2 groups. For e.g. given 3 groups of sample size 100 each, ANOVA can be used to determine if the sample mean is different. The catch here however is that though the ANOVA might suggest a diiferent mean for one group over others, it will not specify the group with the different mean.
  Two ways to perform anova - 

```r
set.seed(1)
x = 1:40
y = rnorm(x)
dat <- data.frame(x=x,y=y)

# First way to ANOVA is to build model (lm/glm etc.) and then call anova()
glm.fit <- glm(y~x,data = dat)

anova(glm.fit)
```

```
## Analysis of Deviance Table
## 
## Model: gaussian, link: identity
## 
## Response: y
## 
## Terms added sequentially (first to last)
## 
## 
##      Df Deviance Resid. Df Resid. Dev
## NULL                    39     30.661
## x     1 0.075499        38     30.585
```

```r
# Second way to anova is to call aov; which builds an lm in itself
aov(y~x,data=dat)
```

```
## Call:
##    aov(formula = y ~ x, data = dat)
## 
## Terms:
##                         x Residuals
## Sum of Squares   0.075499 30.585457
## Deg. of Freedom         1        38
## 
## Residual standard error: 0.8971513
## Estimated effects may be unbalanced
```

### 
