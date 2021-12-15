---
title: Mediation Lab
author: 'Dina Arch & Karen Nylund-Gibson'
date: '2021-12-14'
slug: mediation-lab
categories: Labs
tags: 
- r markdown
- mediation
- data analysis
subtitle: 'An example of mediation with multiple mediators for a linear models course'
summary: ''
authors: []
lastmod: '2021-12-14T19:52:21-08:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---





```r
library(haven) #read_sav()
library(mediation) # mediate() (Tingley, Yamamoto, Hirose, Keele, & Imai, 2014)
library(gvlma) # gvlma()
library(kableExtra) #kable()
library(corrr) #correlate()
library(psych) #mediate()
library(tidyverse)
```

# Mediation in R

This exampes comes from a tutorial paper on mediation found [here](https://www.tqmp.org/RegularArticles/vol13-3/p148/p148.pdf).
This is an open source paper that shared the data set used in this example.
The following is code and interpretation are based on this paper.
Note that they use a different package to estimate the model and you will see that our estimates are very close, though not exact.

Kane, L.
& Ashbaugh, A. R.
(2017).
Simple and parallel mediation: A tutorial exploring anxiety sensitivity, sensation seeking, and gender.
*The Quantitative Methods for Psychology*, 13(3), 148--165.
<doi:10.20982/tqmp.13.3.p148>

## Read in data

Downloaded from the journal [website](https://www.tqmp.org/RegularArticles/vol13-3/p148/index.html).


```r
tqmp <- read_sav("p148.sav")
```

+---------------+---------------------------------------------+--------------------------------------------+
| Variable Name | Description                                 | Notes                                      |
+===============+=============================================+============================================+
| Gender        | Gender                                      | 0 = female, 1= male                        |
+---------------+---------------------------------------------+--------------------------------------------+
| ASI.TOT       | Anxiety sensitivity index total scale score | higher scores mean more anxiety            |
+---------------+---------------------------------------------+--------------------------------------------+
| ASI.PHY       | ASI-3 physical concerns subscale            | higher values means more concerns          |
+---------------+---------------------------------------------+--------------------------------------------+
| ASI.SOC       | ASI-3 social concerns subscale              | higher values means more concerns          |
+---------------+---------------------------------------------+--------------------------------------------+
| ASI.COG       | ASI-3 cognitive concerns subscale           | higher values means more concerns          |
+---------------+---------------------------------------------+--------------------------------------------+
| SSs           | UPPS-P sensation seeking subscale           | higher values means more sensation seeking |
+---------------+---------------------------------------------+--------------------------------------------+
| NUs           | UPPS-P negative urgency subscale            | higher values means more negative urgency  |
+---------------+---------------------------------------------+--------------------------------------------+

## Correlation

Just to explore the variables and their relationships we look at a correlation table.


```r
#Pretty correlation table:
tab_02 = tqmp%>% 
  correlate() %>%
  shave(upper = TRUE) %>%
  fashion(decimals = 2, na_print = "—") 
tab_02 %>% 
  kable()
```

<table>
 <thead>
  <tr>
   <th style="text-align:left;"> term </th>
   <th style="text-align:left;"> Gender </th>
   <th style="text-align:left;"> ASI.TOT </th>
   <th style="text-align:left;"> ASI.PHY </th>
   <th style="text-align:left;"> ASI.SOC </th>
   <th style="text-align:left;"> ASI.COG </th>
   <th style="text-align:left;"> SSs </th>
   <th style="text-align:left;"> NUs </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Gender </td>
   <td style="text-align:left;"> — </td>
   <td style="text-align:left;"> — </td>
   <td style="text-align:left;"> — </td>
   <td style="text-align:left;"> — </td>
   <td style="text-align:left;"> — </td>
   <td style="text-align:left;"> — </td>
   <td style="text-align:left;"> — </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ASI.TOT </td>
   <td style="text-align:left;"> -.12 </td>
   <td style="text-align:left;"> — </td>
   <td style="text-align:left;"> — </td>
   <td style="text-align:left;"> — </td>
   <td style="text-align:left;"> — </td>
   <td style="text-align:left;"> — </td>
   <td style="text-align:left;"> — </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ASI.PHY </td>
   <td style="text-align:left;"> -.17 </td>
   <td style="text-align:left;"> .89 </td>
   <td style="text-align:left;"> — </td>
   <td style="text-align:left;"> — </td>
   <td style="text-align:left;"> — </td>
   <td style="text-align:left;"> — </td>
   <td style="text-align:left;"> — </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ASI.SOC </td>
   <td style="text-align:left;"> -.09 </td>
   <td style="text-align:left;"> .80 </td>
   <td style="text-align:left;"> .54 </td>
   <td style="text-align:left;"> — </td>
   <td style="text-align:left;"> — </td>
   <td style="text-align:left;"> — </td>
   <td style="text-align:left;"> — </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ASI.COG </td>
   <td style="text-align:left;"> -.05 </td>
   <td style="text-align:left;"> .90 </td>
   <td style="text-align:left;"> .75 </td>
   <td style="text-align:left;"> .57 </td>
   <td style="text-align:left;"> — </td>
   <td style="text-align:left;"> — </td>
   <td style="text-align:left;"> — </td>
  </tr>
  <tr>
   <td style="text-align:left;"> SSs </td>
   <td style="text-align:left;"> .19 </td>
   <td style="text-align:left;"> -.23 </td>
   <td style="text-align:left;"> -.27 </td>
   <td style="text-align:left;"> -.17 </td>
   <td style="text-align:left;"> -.14 </td>
   <td style="text-align:left;"> — </td>
   <td style="text-align:left;"> — </td>
  </tr>
  <tr>
   <td style="text-align:left;"> NUs </td>
   <td style="text-align:left;"> .02 </td>
   <td style="text-align:left;"> .38 </td>
   <td style="text-align:left;"> .30 </td>
   <td style="text-align:left;"> .30 </td>
   <td style="text-align:left;"> .39 </td>
   <td style="text-align:left;"> .13 </td>
   <td style="text-align:left;"> — </td>
  </tr>
</tbody>
</table>

# Simple Mediation Model

This example is from the paper directly.

-   Independent Variable (X): Gender.

-   Dependent Variable (Y): Sensation seeking (SSs)

-   Mediator (M): Anxiety sensitivity (ASI.TOT)

Figure 1.
(from paper)

![](simple_med.JPG)

## Testing Statistical Assumptions

Before we dive into analyzing the mediation model, we first check the statistical assumptions.
Please follow along on page 151 in the tutorial paper to see how they interpreted the assumptions for this mediation model.

#### Linearity, Homoscedasticity, Normality of estimation error

*Linearity*:"To examine these criteria for a simple mediation, you need to plot residuals against predicted values in four regressions: X predicting Y (c), X predicting M (a), M predicting Y (b), and X and M predicting Y (combined linearity of b and c')." (pg. 151)

*Homoscedasticity*: "To check homoscedasticity, return to the same plot that we created to examine linearity, but this time look for consistency in vertical range across the X axis. In other words, see if the data spreads on the Y axis consistently and equally throughout the plot, resembling a rectangle." (pg. 152)

*Normality of estimation error*: "To examine this assumption, we can create a Q-Q plot with the residuals we saved from the regression" (pg. 152)

In R, **plot 1** creates a plot of the residuals against the predicted values and **plot 2** creates a Q-Q plot of the residuals.


```r
# X predicting Y (path c) 
fit_c <- lm(SSs ~ Gender, data=tqmp) 
plot(fit_c, c(1,2))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-1.png" width="672" /><img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-2.png" width="672" />

```r
# X predicting M (path a)
fit_a <- lm(ASI.TOT ~ Gender, data=tqmp) 
plot(fit_a, c(1,2))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-3.png" width="672" /><img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-4.png" width="672" />

```r
# M predicting Y (path b)
fit_b <- lm(SSs ~ ASI.TOT, data=tqmp) 
plot(fit_b, c(1,2))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-5.png" width="672" /><img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-6.png" width="672" />

```r
# X and M predicting Y (b and c')
fit_cb <- lm(SSs ~  Gender + ASI.TOT, data=tqmp) 
plot(fit_cb, c(1,2))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-7.png" width="672" /><img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-8.png" width="672" />

## Simple Mediation R Code

**Important note!** There are **two** `mediate()` functions used in this lab.
One comes from the `psych` package and the other comes from the `mediation` package.
You must identify the package in the code like shown below to specify which `mediate()` function you're using.
For simple mediation with one mediator, we use [`mediation::mediate()`](https://ademos.people.uic.edu/Chapter14.html#24_method_2:_the_mediation_pacakge_method).
For multiple mediators, we use [`psych::mediate()`](http://personality-project.org/r/psych/HowTo/mediation.pdf) shown in the next example.

Recall the steps to test for a mediation effect using one mediator (See slide 15 in lecture):


```r
# Model 1: M regressed on X, path a
fit_a <-  lm(ASI.TOT ~ Gender, data=tqmp) 
summary(fit_a)
```

```
## 
## Call:
## lm(formula = ASI.TOT ~ Gender, data = tqmp)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -24.135 -11.343  -3.135  10.657  47.449 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)   26.135      1.236  21.151   <2e-16 ***
## Gender        -3.584      1.750  -2.048   0.0415 *  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 15.03 on 293 degrees of freedom
## Multiple R-squared:  0.01411,	Adjusted R-squared:  0.01074 
## F-statistic: 4.192 on 1 and 293 DF,  p-value: 0.0415
```

```r
# Model 2: Y is regressed on X, path c
fit_c <- lm(SSs ~ Gender, data=tqmp) 
summary(fit_c)
```

```
## 
## Call:
## lm(formula = SSs ~ Gender, data = tqmp)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -1.88379 -0.38379  0.01295  0.44955  1.34628 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  2.65372    0.04778  55.538  < 2e-16 ***
## Gender       0.23007    0.06769   3.399  0.00077 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.5813 on 293 degrees of freedom
## Multiple R-squared:  0.03793,	Adjusted R-squared:  0.03465 
## F-statistic: 11.55 on 1 and 293 DF,  p-value: 0.0007702
```

```r
# Model 3: Y is regressed on X and M (paths c' and b)
fit_cb <- lm(SSs ~ Gender+ ASI.TOT, data=tqmp) 
summary(fit_cb)
```

```
## 
## Call:
## lm(formula = SSs ~ Gender + ASI.TOT, data = tqmp)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -1.90432 -0.35627  0.01137  0.43682  1.36129 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  2.864067   0.074417  38.487  < 2e-16 ***
## Gender       0.201224   0.066792   3.013 0.002816 ** 
## ASI.TOT     -0.008049   0.002213  -3.636 0.000327 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.5695 on 292 degrees of freedom
## Multiple R-squared:  0.07961,	Adjusted R-squared:  0.07331 
## F-statistic: 12.63 on 2 and 292 DF,  p-value: 5.49e-06
```

```r
# Path b (not a required step to test for mediation but will be used in the write up)
fit_b <- lm(SSs ~ ASI.TOT, data=tqmp) 
summary(fit_b)
```

```
## 
## Call:
## lm(formula = SSs ~ ASI.TOT, data = tqmp)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -1.80681 -0.41348  0.00638  0.40406  1.32333 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  2.983622   0.063814  46.755  < 2e-16 ***
## ASI.TOT     -0.008841   0.002228  -3.968 9.11e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.5773 on 293 degrees of freedom
## Multiple R-squared:  0.05101,	Adjusted R-squared:  0.04777 
## F-statistic: 15.75 on 1 and 293 DF,  p-value: 9.108e-05
```

### Test for Indirect Effect

As mentioned in lecture (slide 17), there are multiple ways to estimate the indirect effect and its SE.
The one we will use is the bootstrapped confidence intervals (similar to the tutorial paper as noted on page 153).
In `mediation::mediate()`, we enter path a and combined path c and b to test the indirect effect using bootstrapping procedures.
In the tutorial paper, they used 10,000 bootstrap samples (pg 153).
This may take a minute to run.
You can cache the results by using `{r, cache = TRUE}` in the top part of the code chunk so it doesn't need to run every time you knit.
You may decrease the number to 1,000 if you find that's taking too long to run (the results may not be identical to the paper, however).

In the output, we look at the *ACME* (or Average Causal Mediation Effect) to evaluate the indirect effect of the mediator.
The *ADE* (or Average Direct Effects) provides the direct effect estimate, and the *Total Effect* provides the combined indirect and direct effects estimate.
*Prop. Mediated* is the ratio of these estimates (which we will not be using).


```r
fitMed <- mediation::mediate(fit_a, fit_cb, treat="Gender", mediator="ASI.TOT", boot= TRUE,  boot.ci.type = "perc", sims = 10000)
summary(fitMed)
```

```
## 
## Causal Mediation Analysis 
## 
## Nonparametric Bootstrap Confidence Intervals with the Percentile Method
## 
##                Estimate 95% CI Lower 95% CI Upper p-value    
## ACME           0.028847     0.000593         0.07  0.0456 *  
## ADE            0.201224     0.069159         0.33  0.0024 ** 
## Total Effect   0.230071     0.099398         0.36  0.0006 ***
## Prop. Mediated 0.125384     0.002647         0.37  0.0462 *  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Sample Size Used: 295 
## 
## 
## Simulations: 10000
```

![](simple_med_w_coef.JPG)

*Note*: This figure was taken from the paper.

### Full or Partial Mediation?

In this example, the direct effect is still significant when accounting the mediator.
We also see that the direct effect with the mediator (c' = 0.201) is less than the total effect (c = 0.230).
Thus, this is evidence of partial mediation (see Slide 10 in lecture).

### Sample Write-Up

*Note*: that the estimates from our estimation of the model here is slightly off from what is reported in the manuscript.
They used the `PROCESS`macro and we use the `mediate` package.

Results from a simple mediation analysis indicated that gender is indirectly related to sensation seeking through its relationship with anxiety sensitivity.
First, as can be seen in Figure 7, men reported less anxiety sensitivity than women (*a* = −3.584, *p* = .042), and lower reported anxiety sensitivity was subsequently related to more sensation seeking (*b* = −0.008, *p* =\< .001).
A 95% bias-corrected confidence interval based on 10,000 bootstrap samples indicated that the indirect effect (*ab* = 0.029) was entirely above zero (0.003 to 0.074).
Moreover, men reported greater sensation seeking even after taking into account gender's indirect effect through anxiety sensitivity (*c'* = 0.201, *p* = .003).

# Complex Mediation with Multiple Mediators

Here, we continue the tutorial paper's example and adding on multiple mediators (three) into the model: Physical, Cognitive, and Social Concerns.

-   Independent Variable (X): Gender.

-   Dependent Variable (Y): Sensation seeking (SSs)

-   Mediator 1 (M1): Physical Concerns (ASI.PHY)

-   Mediator 2 (M2): Social Concerns (ASI.SOC)

-   Mediator 3 (M3): Cognitive Concerns (ASI.COG)

![](multiple_med.JPG)

## Testing Statistical Assumptions

From the paper: "We verified the assumptions using the same methods we used for simple mediation, only this time we conducted seven additional regressions (i.e., X [gender] predicting each mediator [the three anxiety sensitivity dimensions]; each mediator predicting Y [sensation seeking]; X and all three mediators predicting Y ; note: we already had regressed Y on X for the simple mediation)." (pg 156).

We won't run all seven for this lab, but just note that this is what needs to be done with multiple mediators.

## Complex Mediation R Code

We are using the [`psych::mediate()`](http://personality-project.org/r/psych/HowTo/mediation.pdf) function to specify multiple mediators (you can do this in the other package).
This function will go through the steps of mediation as well as provide the tests of indirect effects for each mediator (three).
An important note (again), the results will only show up when you knit the markdown.
We spent *many, many* hours on this only to find that knitting reveals the specific indirect effect estimates and their significance tests.

### Test for Indirect Effect


```r
fitComplex <- psych::mediate(SSs ~ Gender + (ASI.PHY) + (ASI.SOC)+ (ASI.COG), n.iter = 10000, data = tqmp)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-7-1.png" width="672" />

```r
fitComplex
```

```
## 
## Mediation/Moderation Analysis 
## Call: psych::mediate(y = SSs ~ Gender + (ASI.PHY) + (ASI.SOC) + (ASI.COG), 
##     data = tqmp, n.iter = 10000)
## 
## The DV (Y) was  SSs . The IV (X) was  Gender . The mediating variable(s) =  ASI.PHY ASI.SOC ASI.COG .
## 
## Total effect(c) of  Gender  on  SSs  =  0.23   S.E. =  0.07  t  =  3.4  df=  293   with p =  0.00077
## Direct effect (c') of  Gender  on  SSs  removing  ASI.PHY ASI.SOC ASI.COG  =  0.17   S.E. =  0.07  t  =  2.57  df=  290   with p =  0.011
## Indirect effect (ab) of  Gender  on  SSs  through  ASI.PHY ASI.SOC ASI.COG   =  0.06 
## Mean bootstrapped indirect effect =  0.06  with standard error =  0.03  Lower CI =  0.01    Upper CI =  0.12
## R = 0.32 R2 = 0.1   F = 8.2 on 4 and 290 DF   p-value:  2.96e-07 
## 
##  To see the longer output, specify short = FALSE in the print statement or ask for the summary
```

### Sample Write-Up

Results from a parallel mediation analysis indicated that gender is indirectly related to sensation seeking through its relationship with the Physical Concerns subscale of anxiety sensitivity.
This dimension pertains to the fear of physiological sensations because of the belief that they may have negative consequences and are life-threatening.
First, as can be seen in Figure 9, men reported less fear of physiological sensations than women (*a1* = −2.021, *p* = .004), and lower reported fear of physiological sensations was subsequently related to more sensation seeking (*b1* = −0.029, *p* \< .001).
A 95% bias-corrected confidence interval based on 10,000 bootstrap samples indicated that the indirect effect through fear of physiological sensations (*a1\*b1* = 0.058), holding all other mediators constant, was entirely above zero (0.017 to 0.132).
In contrast, the indirect effects through both the Social and the Cognitive Concerns subscales of anxiety sensitivity were not different than zero (−0.004 to 0.038 and −0.047 to 0.005, respectively; see Figure 9 (in the paper) for the effects associated with these pathways).
Moreover, men reported greater sensation seeking even when taking into account gender's indirect effect through all three dimensions of anxiety sensitivity (*c'* = 0.172, *p* = .011).

## Next steps and resources

There are extensions of these models.
It is possible to add moderators to this.
This would allow for the mediatoinal relationships to vary by levels of the moderator.
There are resources out there to do this, which we won't cover here.
My hope is that you will have a foundation in moderation and mediation to extend your own knowledge.

[This](https://m-clark.github.io/posts/2019-03-12-mediation-models/) resources was a helpful way to learn about mediation and the different package in R.
