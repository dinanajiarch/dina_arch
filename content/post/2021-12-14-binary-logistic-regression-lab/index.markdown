---
title: Binary Logistic Regression Lab
author: 'Dina Arch'
date: '2021-05-30'
slug: binary-logistic-regression-lab
categories: Labs
tags: 
- r markdown
- binary regression
- labs
- odds ratios
- classification
subtitle: 'This lab that I created follows along the logistic regression lecture presented in a linear models class and provides two examples to demonstrate the models and their interpretation.'
summary: ''
authors: []
lastmod: '2021-12-14T19:39:18-08:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

``` r
library(pander) #pander()
library(psych) # describe()
library(gtsummary) #tbl_summary()
library(equatiomatic) # extract_eq()
library(sjPlot) # tab_xtab(), tab_model()
library(tidyverse)
```

This lab follows along the logistic regression lecture presented in class and provides two examples to demonstrate the models and their interpretation.

# What is Binary Logistic Regression?

-   It is a regression with an **outcome** **variable** (or **dependent variable**) that is dichotomous/binary (i.e., only two categories, such as Yes or No, 0 or 1, Disorder or No Disorder, Win or Lose).

    -   The **predictor variables** (or **independent variables** or **explanatory variables**) can be either continuous or categorical

-   In binary logistic regression, we are interested in predicting which of two possible events (e.g., win or lose) are going to happen given the predictor(s) variables

-   Assumes a binomial distribution or “a probability distribution that summarizes the likelihood that a value will take one of two independent values under a given set of parameters or assumptions.” ([Source](https://www.investopedia.com/terms/b/binomialdistribution.asp))

# Binary Logistic Regression in R

## Admissions Example

This example comes from the UCLA Statistical Consulting page found [here](https://stats.idre.ucla.edu/r/dae/logit-regression/).
This example assesses how GRE, GPA, and prestige of the undergraduate institutions effect graduate school admission.

| **Variable** | **Description**                                                                     |
|--------------|-------------------------------------------------------------------------------------|
| gre          | Graduate Record Exam (GRE) scores, *continuous*                                     |
| gpa          | Grade point average, *continuous*                                                   |
| rank         | Rank of undergraduate institutions, 1 = highest prestige, 4 = lowest, *categorical* |
| admit        | Admission decision, 0 = did not admit, 1 = admitted, *binary*                       |

**References**

Hosmer, D.
& Lemeshow, S.
(2000).
Applied Logistic Regression (Second Edition).
New York: John Wiley & Sons, Inc.

Long, J. Scott (1997).
Regression Models for Categorical and Limited Dependent Variables.
Thousand Oaks, CA: Sage Publications.

### Read in data

``` r
log <- read.csv("https://stats.idre.ucla.edu/stat/data/binary.csv") %>% 
  mutate(rank = factor(rank),  #we need to convert this variable to a factor
         admit = factor(admit, levels = c("0", "1"), labels = c("Not Admitted", "Admitted"))) # add labels to outcome
```

### Summary

``` r
# psych::describe()
round(describe(log),2) %>% 
  pander() 
```

|             | vars |  n  | mean  |  sd   | median | trimmed |  mad  | min  |
|:-----------:|:----:|:---:|:-----:|:-----:|:------:|:-------:|:-----:|:----:|
| **admit**\* |  1   | 400 | 1.32  | 0.47  |   1    |  1.27   |   0   |  1   |
|   **gre**   |  2   | 400 | 587.7 | 115.5 |  580   |  589.1  | 118.6 | 220  |
|   **gpa**   |  3   | 400 | 3.39  | 0.38  |  3.4   |   3.4   |  0.4  | 2.26 |
| **rank**\*  |  4   | 400 | 2.48  | 0.94  |   2    |  2.48   | 1.48  |  1   |

Table continues below

|             | max | range | skew  | kurtosis |  se  |
|:-----------:|:---:|:-----:|:-----:|:--------:|:----:|
| **admit**\* |  2  |   1   | 0.78  |  -1.39   | 0.02 |
|   **gre**   | 800 |  580  | -0.14 |  -0.36   | 5.78 |
|   **gpa**   |  4  | 1.74  | -0.21 |   -0.6   | 0.02 |
| **rank**\*  |  4  |   3   |  0.1  |  -0.91   | 0.05 |

``` r
# I like this table more, but you can do either! Especially if you're getting some errors with gtsummary(). 

# gtsummary::tbl_summary()
tbl_summary(log,
            statistic = list(all_continuous() ~ "{mean} ({sd})"),
                             missing = "no")
```

<div id="rnvuxdypxd" style="overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>html {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Helvetica Neue', 'Fira Sans', 'Droid Sans', Arial, sans-serif;
}

#rnvuxdypxd .gt_table {
  display: table;
  border-collapse: collapse;
  margin-left: auto;
  margin-right: auto;
  color: #333333;
  font-size: 16px;
  font-weight: normal;
  font-style: normal;
  background-color: #FFFFFF;
  width: auto;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #A8A8A8;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #A8A8A8;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
}

#rnvuxdypxd .gt_heading {
  background-color: #FFFFFF;
  text-align: center;
  border-bottom-color: #FFFFFF;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#rnvuxdypxd .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#rnvuxdypxd .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 0;
  padding-bottom: 6px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#rnvuxdypxd .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#rnvuxdypxd .gt_col_headings {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#rnvuxdypxd .gt_col_heading {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 6px;
  padding-left: 5px;
  padding-right: 5px;
  overflow-x: hidden;
}

#rnvuxdypxd .gt_column_spanner_outer {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  padding-top: 0;
  padding-bottom: 0;
  padding-left: 4px;
  padding-right: 4px;
}

#rnvuxdypxd .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#rnvuxdypxd .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#rnvuxdypxd .gt_column_spanner {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 5px;
  overflow-x: hidden;
  display: inline-block;
  width: 100%;
}

#rnvuxdypxd .gt_group_heading {
  padding: 8px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
}

#rnvuxdypxd .gt_empty_group_heading {
  padding: 0.5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: middle;
}

#rnvuxdypxd .gt_from_md > :first-child {
  margin-top: 0;
}

#rnvuxdypxd .gt_from_md > :last-child {
  margin-bottom: 0;
}

#rnvuxdypxd .gt_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  margin: 10px;
  border-top-style: solid;
  border-top-width: 1px;
  border-top-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  overflow-x: hidden;
}

#rnvuxdypxd .gt_stub {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 12px;
}

#rnvuxdypxd .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#rnvuxdypxd .gt_first_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
}

#rnvuxdypxd .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#rnvuxdypxd .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#rnvuxdypxd .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#rnvuxdypxd .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#rnvuxdypxd .gt_footnotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#rnvuxdypxd .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding: 4px;
}

#rnvuxdypxd .gt_sourcenotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#rnvuxdypxd .gt_sourcenote {
  font-size: 90%;
  padding: 4px;
}

#rnvuxdypxd .gt_left {
  text-align: left;
}

#rnvuxdypxd .gt_center {
  text-align: center;
}

#rnvuxdypxd .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#rnvuxdypxd .gt_font_normal {
  font-weight: normal;
}

#rnvuxdypxd .gt_font_bold {
  font-weight: bold;
}

#rnvuxdypxd .gt_font_italic {
  font-style: italic;
}

#rnvuxdypxd .gt_super {
  font-size: 65%;
}

#rnvuxdypxd .gt_footnote_marks {
  font-style: italic;
  font-weight: normal;
  font-size: 65%;
}
</style>
<table class="gt_table">
  
  <thead class="gt_col_headings">
    <tr>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1"><strong>Characteristic</strong></th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1"><strong>N = 400</strong><sup class="gt_footnote_marks">1</sup></th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr><td class="gt_row gt_left">admit</td>
<td class="gt_row gt_center"></td></tr>
    <tr><td class="gt_row gt_left" style="text-align: left; text-indent: 10px;">Not Admitted</td>
<td class="gt_row gt_center">273 (68%)</td></tr>
    <tr><td class="gt_row gt_left" style="text-align: left; text-indent: 10px;">Admitted</td>
<td class="gt_row gt_center">127 (32%)</td></tr>
    <tr><td class="gt_row gt_left">gre</td>
<td class="gt_row gt_center">588 (116)</td></tr>
    <tr><td class="gt_row gt_left">gpa</td>
<td class="gt_row gt_center">3.39 (0.38)</td></tr>
    <tr><td class="gt_row gt_left">rank</td>
<td class="gt_row gt_center"></td></tr>
    <tr><td class="gt_row gt_left" style="text-align: left; text-indent: 10px;">1</td>
<td class="gt_row gt_center">61 (15%)</td></tr>
    <tr><td class="gt_row gt_left" style="text-align: left; text-indent: 10px;">2</td>
<td class="gt_row gt_center">151 (38%)</td></tr>
    <tr><td class="gt_row gt_left" style="text-align: left; text-indent: 10px;">3</td>
<td class="gt_row gt_center">121 (30%)</td></tr>
    <tr><td class="gt_row gt_left" style="text-align: left; text-indent: 10px;">4</td>
<td class="gt_row gt_center">67 (17%)</td></tr>
  </tbody>
  
  <tfoot>
    <tr class="gt_footnotes">
      <td colspan="2">
        <p class="gt_footnote">
          <sup class="gt_footnote_marks">
            <em>1</em>
          </sup>
           
          n (%); Mean (SD)
          <br />
        </p>
      </td>
    </tr>
  </tfoot>
</table>
</div>

### Contingency Table

Contingency tables, sometimes called cross tabulations or cross tabs, that displays the frequency distribution of your **categorical** variables.
In R, we use `sjPlot::tab_xtabs()` (Found by Karen!).
We want to check the contingency table to make sure there are values in each cell.
Since we only have one categorical variable, `rank`, we only need to check one contingency table.

``` r
sjPlot::tab_xtab(var.row = log$rank, var.col = log$admit, title = "Cross tab of Admit rate by Rank of Undergraduate Insitute", show.row.prc = TRUE, show.summary=F)
```

<table style="border-collapse:collapse; border:none;">
<caption style="font-weight: bold; text-align:left;">
Cross tab of Admit rate by Rank of Undergraduate Insitute
</caption>
<tr>
<th style="border-top:double; text-align:center; font-style:italic; font-weight:normal; border-bottom:1px solid;" rowspan="2">
rank
</th>
<th style="border-top:double; text-align:center; font-style:italic; font-weight:normal;" colspan="2">
admit
</th>
<th style="border-top:double; text-align:center; font-style:italic; font-weight:normal; font-weight:bolder; font-style:italic; border-bottom:1px solid; " rowspan="2">
Total
</th>
</tr>
<tr>
<td style="border-bottom:1px solid; text-align:center; padding:0.2cm;">
Not Admitted
</td>
<td style="border-bottom:1px solid; text-align:center; padding:0.2cm;">
Admitted
</td>
</tr>
<tr>
<td style="padding:0.2cm;  text-align:left; vertical-align:middle;">
1
</td>
<td style="padding:0.2cm; text-align:center; ">
<span style="color:black;">28</span><br><span style="color:#333399;">45.9 %</span>
</td>
<td style="padding:0.2cm; text-align:center; ">
<span style="color:black;">33</span><br><span style="color:#333399;">54.1 %</span>
</td>
<td style="padding:0.2cm; text-align:center;  ">
<span style="color:black;">61</span><br><span style="color:#333399;">100 %</span>
</td>
</tr>
<tr>
<td style="padding:0.2cm;  text-align:left; vertical-align:middle;">
2
</td>
<td style="padding:0.2cm; text-align:center; ">
<span style="color:black;">97</span><br><span style="color:#333399;">64.2 %</span>
</td>
<td style="padding:0.2cm; text-align:center; ">
<span style="color:black;">54</span><br><span style="color:#333399;">35.8 %</span>
</td>
<td style="padding:0.2cm; text-align:center;  ">
<span style="color:black;">151</span><br><span style="color:#333399;">100 %</span>
</td>
</tr>
<tr>
<td style="padding:0.2cm;  text-align:left; vertical-align:middle;">
3
</td>
<td style="padding:0.2cm; text-align:center; ">
<span style="color:black;">93</span><br><span style="color:#333399;">76.9 %</span>
</td>
<td style="padding:0.2cm; text-align:center; ">
<span style="color:black;">28</span><br><span style="color:#333399;">23.1 %</span>
</td>
<td style="padding:0.2cm; text-align:center;  ">
<span style="color:black;">121</span><br><span style="color:#333399;">100 %</span>
</td>
</tr>
<tr>
<td style="padding:0.2cm;  text-align:left; vertical-align:middle;">
4
</td>
<td style="padding:0.2cm; text-align:center; ">
<span style="color:black;">55</span><br><span style="color:#333399;">82.1 %</span>
</td>
<td style="padding:0.2cm; text-align:center; ">
<span style="color:black;">12</span><br><span style="color:#333399;">17.9 %</span>
</td>
<td style="padding:0.2cm; text-align:center;  ">
<span style="color:black;">67</span><br><span style="color:#333399;">100 %</span>
</td>
</tr>
<tr>
<td style="padding:0.2cm;  border-bottom:double; font-weight:bolder; font-style:italic; text-align:left; vertical-align:middle;">
Total
</td>
<td style="padding:0.2cm; text-align:center;   border-bottom:double;">
<span style="color:black;">273</span><br><span style="color:#333399;">68.2 %</span>
</td>
<td style="padding:0.2cm; text-align:center;   border-bottom:double;">
<span style="color:black;">127</span><br><span style="color:#333399;">31.8 %</span>
</td>
<td style="padding:0.2cm; text-align:center;   border-bottom:double;">
<span style="color:black;">400</span><br><span style="color:#333399;">100 %</span>
</td>
</tr>
</table>

This table shows the row and column totals, as well as row percentages.
You can also ask for column percentages, if you prefer.

### Binary Logistic Model

Recall in linear regression, we used the `lm()` function.
With logistic regression, we use `glm()`, or general linear model.
The set up is similar to linear regression, however.
One important note is to make sure we identify the type of “family function,” or the description of the error distribution used in the model.
For logistic regression, it’s `family = "binomial"`.

In our example, the predictors are GRE scores, GPA, and university rank related to graduate school admission status.

``` r
mylogit <- glm(admit ~ gre + gpa + rank, data = log, family = "binomial")
summary(mylogit)
```

    ## 
    ## Call:
    ## glm(formula = admit ~ gre + gpa + rank, family = "binomial", 
    ##     data = log)
    ## 
    ## Deviance Residuals: 
    ##     Min       1Q   Median       3Q      Max  
    ## -1.6268  -0.8662  -0.6388   1.1490   2.0790  
    ## 
    ## Coefficients:
    ##              Estimate Std. Error z value Pr(>|z|)    
    ## (Intercept) -3.989979   1.139951  -3.500 0.000465 ***
    ## gre          0.002264   0.001094   2.070 0.038465 *  
    ## gpa          0.804038   0.331819   2.423 0.015388 *  
    ## rank2       -0.675443   0.316490  -2.134 0.032829 *  
    ## rank3       -1.340204   0.345306  -3.881 0.000104 ***
    ## rank4       -1.551464   0.417832  -3.713 0.000205 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for binomial family taken to be 1)
    ## 
    ##     Null deviance: 499.98  on 399  degrees of freedom
    ## Residual deviance: 458.52  on 394  degrees of freedom
    ## AIC: 470.52
    ## 
    ## Number of Fisher Scoring iterations: 4

Let’s dissect this output.
It looks similar to regular simple linear regression but interpretation is different (as we saw in lecture).
When looking at the coefficients, we see `gre`, `gpa`, and the levels of `rank` are significant predictors of admission status.

With binary logistic regression, regression coefficient is the change in the log odds of the outcome for a one unit increase in the predictor variable, holding others constant.
The intercept is the log odds for students who has zero on all the predictors, a value not observed in this sample (e.g., no one has GPA or GRE score of zero).

So this is how we would interpret those same variables when running a logistic regression model (Note: it’s a z-statistic now since these are standardized coefficients):

-   GRE was significantly related to admission decision (*z* = 2.070, *p* = 0.038). Specifically, the results indicate that for every one-unit increase in GRE score, there is an increase of *B* = 0.002 in the **the log odds** of being admitted to graduate school, controlling for GPA and university rank.

Additionally, this is how we would interpret the categorical variable `rank`.
Note that because this is a categorical variable, we are comparing the categories to a comparison group (or a reference group).
In this example, the comparison group is **rank 1**:

-   Students who attended rank 2 universities are less likely to be admitted to graduate school, compared to rank 1 universities (*z* = -0.675, *p* = 0.033).

The log odds, or *logits*, are not always easily interpreted and can be converted to odds ratios for easier interpretation.

### Odds Ratios

The coefficients presented above are log odds.
To convert them to odd ratios, we take the exponent of the coefficients (Slide 21 in lecture):

$$
odds ratio = e^{\beta}
$$

``` r
#coverting the log odds (logits) to odds ratios
exp(coef(mylogit))
```

    ## (Intercept)         gre         gpa       rank2       rank3       rank4 
    ##   0.0185001   1.0022670   2.2345448   0.5089310   0.2617923   0.2119375

``` r
## odds ratios and 95% CI
exp(cbind(OR = coef(mylogit), confint(mylogit)))
```

    ##                    OR       2.5 %    97.5 %
    ## (Intercept) 0.0185001 0.001889165 0.1665354
    ## gre         1.0022670 1.000137602 1.0044457
    ## gpa         2.2345448 1.173858216 4.3238349
    ## rank2       0.5089310 0.272289674 0.9448343
    ## rank3       0.2617923 0.131641717 0.5115181
    ## rank4       0.2119375 0.090715546 0.4706961

These values likely make *a little more* sense.
Considering GRE in the example again:

-   GRE was significantly related to admission decision (**z** = 2.070, *p* = 0.038). Specifically, for every one-unit increase in GRE score, **the odds** **of being admitted to graduate school increase by a factor of 1.002**, controlling for GPA and university rank. (Note this is a very small amount because one point increase in GRE doesn’t represent a large difference).

Said differently,

-   GRE was significantly related to admission decision (**z** = 2.070, *p* = 0.038). Specifically, for every one-unit increase in GRE score, **we expect to see about a .002% increase in the odds of being admitted to graduate school**, controlling for GPA and university rank.

### Predicted Probability

To make the coefficients easier to interpret, we can take the odds ratio values and convert them to probabilities.
To do this, we take the odds ratio and divide it by the odds ratio plus 1 (See slide 25):

$$
probability = \frac{odds}{odds + 1}
$$

Before we dive right into conversions, we could also calculate the **predicted probabilities** of admission for specific values of the the predictors.
Recall in lecture (Slide 23) to find the log odds of an x-value, we want to plug it into our regression equation:

``` r
extract_eq(mylogit, use_coefs = TRUE)
```

$$
\log\left[ \frac { \widehat{P( \operatorname{admit} = \operatorname{Admitted} )} }{ 1 - \widehat{P( \operatorname{admit} = \operatorname{Admitted} )} } \right] = -3.99 + 0(\operatorname{gre}) + 0.8(\operatorname{gpa}) - 0.68(\operatorname{rank}_{\operatorname{2}}) - 1.34(\operatorname{rank}_{\operatorname{3}}) - 1.55(\operatorname{rank}_{\operatorname{4}})
$$

To create predicted probabilities, we need to create a new data frame with the x-value we want the independent variables to take on to create our predictions (In lecture, slide 23, the value was *SurvRate* = 40).
For this example, let’s use the mean.
And since we have a **categorical** predictor, `rank`, lets look at the mean of `gre` and `gpa` and each value of university `rank`.
We could, of course, do all the calculations by hand.
But R is easier :)

Let’s first create the data frame using `with`:

``` r
pred_prob <- with(log, data.frame(gre = mean(gre), gpa = mean(gpa), rank = factor(1:4)))
pred_prob
```

    ##     gre    gpa rank
    ## 1 587.7 3.3899    1
    ## 2 587.7 3.3899    2
    ## 3 587.7 3.3899    3
    ## 4 587.7 3.3899    4

So we have our mean of `gpa` and `gre` and each level of `rank`.
Which means we are going to get *four* predicted probabilities, at the mean of the other two variables.
If we did not have a categorical predictor, we would only have *one.*

Now let’s create the predicted probabilities using the mean of `gre` and `gpa` and each level of `rank` using the `predict()` function.
What R is doing here is taking the logits from `mylogit` and converting them to predicted probabilities based on the values we provided in `pred_prob`.
We identify the object as `pred_prob$rankP` to create a new column of the within the `pred_prob` dataset called `rankP`.

``` r
pred_prob$rankP <- predict(mylogit, newdata = pred_prob, type = "response")
pred_prob
```

    ##     gre    gpa rank     rankP
    ## 1 587.7 3.3899    1 0.5166016
    ## 2 587.7 3.3899    2 0.3522846
    ## 3 587.7 3.3899    3 0.2186120
    ## 4 587.7 3.3899    4 0.1846684

Great!
Now we have predicted probabilities.
Here is our interpretation:

-   For those with an average GPA and GRE score (holding GPA and GRE constant), the predicted probabilities of being accepted into a graduate program is .516 for those in the highest ranked undergraduate institutions (rank = 1) and .184 for students from the lowest ranked undergraduate institutions (rank=4). This demonstrates the relation of rank of the university in the probability of being admitted to graduate school.

### Classification Table

A classification table is similar to a contingency table, but in this table we compare the actual values to the predicted values based on the binary logistic regression model.
This is a way to see how well the model did at predicting the outcome of being admitted to graduate school.

In this code, we take actual fitted values from the model (`mylogit$fitted.values`), and separate them by a cut off of *p* = .5.
Those who had a fitted value of above .50 were assigned “Admitted.” We are comparing the values given by the model (fitted values) and whether or not they actually got admitted.
The purpose of this is to see how effective our model is at predicting admission status

``` r
# Converting from probability to actual output
log$fitted_admit <- ifelse(mylogit$fitted.values >= 0.5, "1", "0") #1 = Admitted, 0 = Not Admitted

# Generating the classification table
ctab <- table(log$admit, log$fitted_admit)
ctab
```

    ##               
    ##                  0   1
    ##   Not Admitted 254  19
    ##   Admitted      97  30

``` r
#I (Karen) like this table better because it include row and column totals
sjPlot::tab_xtab(var.row = log$admit, var.col = log$fitted_admit, title = "Observed Admission versus Predited Admission counts", show.row.prc = TRUE, show.summary = F )
```

<table style="border-collapse:collapse; border:none;">
<caption style="font-weight: bold; text-align:left;">
Observed Admission versus Predited Admission counts
</caption>
<tr>
<th style="border-top:double; text-align:center; font-style:italic; font-weight:normal; border-bottom:1px solid;" rowspan="2">
admit
</th>
<th style="border-top:double; text-align:center; font-style:italic; font-weight:normal;" colspan="2">
fitted_admit
</th>
<th style="border-top:double; text-align:center; font-style:italic; font-weight:normal; font-weight:bolder; font-style:italic; border-bottom:1px solid; " rowspan="2">
Total
</th>
</tr>
<tr>
<td style="border-bottom:1px solid; text-align:center; padding:0.2cm;">
0
</td>
<td style="border-bottom:1px solid; text-align:center; padding:0.2cm;">
1
</td>
</tr>
<tr>
<td style="padding:0.2cm;  text-align:left; vertical-align:middle;">
Not Admitted
</td>
<td style="padding:0.2cm; text-align:center; ">
<span style="color:black;">254</span><br><span style="color:#333399;">93 %</span>
</td>
<td style="padding:0.2cm; text-align:center; ">
<span style="color:black;">19</span><br><span style="color:#333399;">7 %</span>
</td>
<td style="padding:0.2cm; text-align:center;  ">
<span style="color:black;">273</span><br><span style="color:#333399;">100 %</span>
</td>
</tr>
<tr>
<td style="padding:0.2cm;  text-align:left; vertical-align:middle;">
Admitted
</td>
<td style="padding:0.2cm; text-align:center; ">
<span style="color:black;">97</span><br><span style="color:#333399;">76.4 %</span>
</td>
<td style="padding:0.2cm; text-align:center; ">
<span style="color:black;">30</span><br><span style="color:#333399;">23.6 %</span>
</td>
<td style="padding:0.2cm; text-align:center;  ">
<span style="color:black;">127</span><br><span style="color:#333399;">100 %</span>
</td>
</tr>
<tr>
<td style="padding:0.2cm;  border-bottom:double; font-weight:bolder; font-style:italic; text-align:left; vertical-align:middle;">
Total
</td>
<td style="padding:0.2cm; text-align:center;   border-bottom:double;">
<span style="color:black;">351</span><br><span style="color:#333399;">87.8 %</span>
</td>
<td style="padding:0.2cm; text-align:center;   border-bottom:double;">
<span style="color:black;">49</span><br><span style="color:#333399;">12.2 %</span>
</td>
<td style="padding:0.2cm; text-align:center;   border-bottom:double;">
<span style="color:black;">400</span><br><span style="color:#333399;">100 %</span>
</td>
</tr>
</table>

Here, we can see that the model correctly predicted 30 of 127, or 23.6%, of the students that actually got admitted.
Thus, it misclassified 76.4% (97) of the admitted students were predicted to be “not admitted.” Additionally, we see that it correctly classified 254 of 273 (93%) of the not admitted students.
Thus this model did a better job at predicting who was not admitted than those who were admitted.

### Accuracy

We can check the accuracy percentage:

``` r
accuracy <- sum(diag(ctab))/sum(ctab)*100
accuracy
```

    ## [1] 71

Overall, the accuracy of our model is 71%, or our logistic model is able to classify 71% of the observations correctly.

## Hierarchical model building with logistic regression

Just as we did in linear regression, we can proceed through a model building process with logistic regression.
In this context, we are interested in seeing how we can improve the prediction of the outcome in a hierarchical fashion.

Let’s say we fit the following models:

1.  Predicting admission from just GPA (log1)

2.  Predicting admission from GPA and GRE (log2)

3.  Predicting admission using GPA, GRE, and rank (log3)

We can create a table of all three models using `tab_model` of the `sjPlot` package.
We can compare the R2 estimates across the models.
Using the `anova`function, we can also test if there is a statistical improvement in fit by adding the predictor(s).
In logistic regression models, we test if the residual deviance significant decreases.
We are using the ANOVA function, but it’s not an ANOVA test like you may have done before.
In this case, we are using ANOVA to compare the deviance for each model to the subsequent model.
If the chi-square value is significant we can interpret that the addition of the extra variables (going from log1 to log 2, for example) significantly improves the model prediction (e.g., lowers the residual variance).

``` r
log1 <- glm(admit ~ gpa , data = log, family = "binomial")
log2 <- glm(admit ~ gpa + gre , data = log, family = "binomial")
log3 <- glm(admit ~ gpa + gre + rank, data = log, family = "binomial")

tab_model(log1, log2, log3)
```

<table style="border-collapse:collapse; border:none;">
<tr>
<th style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm;  text-align:left; ">
 
</th>
<th colspan="3" style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm; ">
admit
</th>
<th colspan="3" style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm; ">
admit
</th>
<th colspan="3" style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm; ">
admit
</th>
</tr>
<tr>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  text-align:left; ">
Predictors
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">
Odds Ratios
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">
CI
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">
p
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">
Odds Ratios
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">
CI
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  col7">
p
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  col8">
Odds Ratios
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  col9">
CI
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  0">
p
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
(Intercept)
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.01
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.00 – 0.09
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
<strong>\<0.001</strong>
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.01
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.00 – 0.06
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col7">
<strong>\<0.001</strong>
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col8">
0.02
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col9">
0.00 – 0.17
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  0">
<strong>\<0.001</strong>
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
gpa
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
2.86
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
1.61 – 5.20
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
<strong>\<0.001</strong>
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
2.13
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
1.14 – 4.02
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col7">
<strong>0.018</strong>
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col8">
2.23
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col9">
1.17 – 4.32
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  0">
<strong>0.015</strong>
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
gre
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
1.00
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
1.00 – 1.00
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col7">
<strong>0.011</strong>
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col8">
1.00
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col9">
1.00 – 1.00
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  0">
<strong>0.038</strong>
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
rank \[2\]
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col7">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col8">
0.51
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col9">
0.27 – 0.94
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  0">
<strong>0.033</strong>
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
rank \[3\]
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col7">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col8">
0.26
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col9">
0.13 – 0.51
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  0">
<strong>\<0.001</strong>
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
rank \[4\]
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col7">
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col8">
0.21
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  col9">
0.09 – 0.47
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  0">
<strong>\<0.001</strong>
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm; border-top:1px solid;">
Observations
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left; border-top:1px solid;" colspan="3">
400
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left; border-top:1px solid;" colspan="3">
400
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left; border-top:1px solid;" colspan="3">
400
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">
R<sup>2</sup> Tjur
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">
0.032
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">
0.047
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">
0.102
</td>
</tr>
</table>

``` r
anova(log1, log2, log3, test="Chisq")
```

    ## Analysis of Deviance Table
    ## 
    ## Model 1: admit ~ gpa
    ## Model 2: admit ~ gpa + gre
    ## Model 3: admit ~ gpa + gre + rank
    ##   Resid. Df Resid. Dev Df Deviance  Pr(>Chi)    
    ## 1       398     486.97                          
    ## 2       397     480.34  1   6.6236   0.01006 *  
    ## 3       394     458.52  3  21.8265 7.088e-05 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

Based on the ANOVA results, we see that the addition of each additional predictor signficnalty improves the prediction of graduate school admission.
Note, you don’t have to do this, but if you have research questions about the specific predictive power, controloing for the others, you can use this apporach.

## The API Example

Data from [UCLA - Academic Performance Index](https://stats.idre.ucla.edu/r/seminars/introduction-to-regression-in-r/).
Note: This is the same example as the simple linear regression lab (Lab 5).
However, we are changing the model to have a binary outcome for logistic regression.
Below is the list of variables we used in this analysis for lab, noting that there are more variables in the API dataset.

|              |                                                                                        |
|--------------|----------------------------------------------------------------------------------------|
| **Variable** | **Description**                                                                        |
| api00        | Academic Performance Index, 0 = under statewide target of 800, 1 = above 800, *binary* |
| enroll       | School enrollment, *continuous*                                                        |
| Meals        | Number of free and reduced lunch, *continuous*                                         |
| full         | Percentage of teachers will full credentials, *continuous*                             |

### Read in data

Here, we are reading in the data directly from the UCLA website.
Additionally, we are dichotomizing the API (`api00`) variable to separate schools above statewide target of 800 and below.
This cut-off may be outdated and/or controversial, however, we are going to use this for the purpose of providing an example of how to do logistic regression in R.

``` r
api <- read.csv(
"https://stats.idre.ucla.edu/wp-content/uploads/2019/02/elemapi2v2.csv") %>% 
  select(api00, full, enroll, meals) %>%
  mutate(api_factor = cut(api00,
                     levels = c("0", "1"),
                     breaks = c(0, 800, 940),
                     labels = c("Below Target", "At or Above Target"))) # here we are dichotomizing API, see lab 1 for a refresher on what this all does.
```

### Summary

For a short descriptor of each variable, scroll down [here](https://stats.idre.ucla.edu/spss/webbooks/reg/chapter1/regressionwith-spsschapter-1-simple-and-multiple-regression/).

``` r
# psych::describe()
round(describe(api),2) %>% 
  pander() 
```

|                  | vars |  n  | mean  |  sd   | median | trimmed |  mad  | min |
|:----------------:|:----:|:---:|:-----:|:-----:|:------:|:-------:|:-----:|:---:|
|    **api00**     |  1   | 400 | 647.6 | 142.2 |  643   |  645.8  | 177.2 | 369 |
|     **full**     |  2   | 400 | 84.55 | 14.95 |   88   |  86.6   | 14.83 | 37  |
|    **enroll**    |  3   | 400 | 483.5 | 226.4 |  435   |  459.4  | 202.4 | 130 |
|    **meals**     |  4   | 400 | 60.31 | 31.91 |  67.5  |  62.18  | 37.81 |  0  |
| **api_factor**\* |  5   | 400 | 1.19  | 0.39  |   1    |  1.12   |   0   |  1  |

Table continues below

|                  | max  | range | skew  | kurtosis |  se   |
|:----------------:|:----:|:-----:|:-----:|:--------:|:-----:|
|    **api00**     | 940  |  571  |  0.1  |  -1.13   | 7.11  |
|     **full**     | 100  |  63   | -0.97 |   0.17   | 0.75  |
|    **enroll**    | 1570 | 1440  | 1.34  |   3.02   | 11.32 |
|    **meals**     | 100  |  100  | -0.41 |   -1.2   |  1.6  |
| **api_factor**\* |  2   |   1   | 1.55  |   0.42   | 0.02  |

``` r
# I like this table more, but you can do either! Especially if you're getting some errors with gtsummary(). Useful when you have a subset, but here we left all variables in the sample so it's a long table. 

# gtsummary::gtsummary()
#tbl_summary(api,
#            statistic = list(all_continuous() ~ "{mean} ({sd})"),
#                             missing = "no")
```

There is no contingency table in this example because all of our predictors are continuous.

### Binary Logistic Model

We are using a binary logistic regression model to see how school characteristics relate to a school making adequate progress (api=1) or not (api=0).

Binary outcome: API adequate progress =1, not adequate progress=0 Predictors: % fully credential teachers, student enrollment, and % on free and reduced lunch

``` r
mylogit <- glm(api_factor ~ full + enroll + meals, data = api, family = "binomial")
summary(mylogit)
```

    ## 
    ## Call:
    ## glm(formula = api_factor ~ full + enroll + meals, family = "binomial", 
    ##     data = api)
    ## 
    ## Deviance Residuals: 
    ##      Min        1Q    Median        3Q       Max  
    ## -1.86557  -0.16221  -0.03269  -0.01028   3.12996  
    ## 
    ## Coefficients:
    ##               Estimate Std. Error z value Pr(>|z|)    
    ## (Intercept) -3.0746093  2.7320279  -1.125   0.2604    
    ## full         0.0582641  0.0282187   2.065   0.0389 *  
    ## enroll       0.0004592  0.0013843   0.332   0.7401    
    ## meals       -0.1096184  0.0144185  -7.603  2.9e-14 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for binomial family taken to be 1)
    ## 
    ##     Null deviance: 391.86  on 399  degrees of freedom
    ## Residual deviance: 147.45  on 396  degrees of freedom
    ## AIC: 155.45
    ## 
    ## Number of Fisher Scoring iterations: 8

### Odds Ratios

``` r
exp(coef(mylogit))
```

    ## (Intercept)        full      enroll       meals 
    ##  0.04620768  1.05999495  1.00045932  0.89617604

``` r
## odds ratios and 95% CI
exp(cbind(OR = coef(mylogit), confint(mylogit)))
```

    ##                     OR        2.5 %    97.5 %
    ## (Intercept) 0.04620768 0.0001656294 7.8435280
    ## full        1.05999495 1.0055761469 1.1237801
    ## enroll      1.00045932 0.9978836793 1.0032716
    ## meals       0.89617604 0.8684201733 0.9192615

### Predicted Probability

``` r
#creating the means of each variable
pred_prob_api<- with(api, data.frame(full = mean(full), enroll = mean(enroll), meals = mean(meals)))
pred_prob_api
```

    ##    full  enroll  meals
    ## 1 84.55 483.465 60.315

``` r
pred_prob_api$apiP<- predict(mylogit, newdata = pred_prob_api, type = "response")
pred_prob_api
```

    ##    full  enroll  meals       apiP
    ## 1 84.55 483.465 60.315 0.01058167

This result says that for schools that have average on all the predictors, the probablity of being at target is .01.
*yikes!*

### Classification Table

``` r
# Converting from probability to actual output
api$fitted_api <- ifelse(mylogit$fitted.values >= 0.5, "1", "0") #1 = Above Average, 0 = Below

# Generating the classification table
ctab <- table(api$api_factor, api$fitted_api)
ctab
```

    ##                     
    ##                        0   1
    ##   Below Target       304  19
    ##   At or Above Target  15  62

``` r
#I (Karen) like this table better because it include row and column totals
sjPlot::tab_xtab(var.row = api$api_factor, var.col = api$fitted_api, title = "Observed Progress versus Predited progress counts", show.row.prc = TRUE, show.summary = F )
```

<table style="border-collapse:collapse; border:none;">
<caption style="font-weight: bold; text-align:left;">
Observed Progress versus Predited progress counts
</caption>
<tr>
<th style="border-top:double; text-align:center; font-style:italic; font-weight:normal; border-bottom:1px solid;" rowspan="2">
api_factor
</th>
<th style="border-top:double; text-align:center; font-style:italic; font-weight:normal;" colspan="2">
fitted_api
</th>
<th style="border-top:double; text-align:center; font-style:italic; font-weight:normal; font-weight:bolder; font-style:italic; border-bottom:1px solid; " rowspan="2">
Total
</th>
</tr>
<tr>
<td style="border-bottom:1px solid; text-align:center; padding:0.2cm;">
0
</td>
<td style="border-bottom:1px solid; text-align:center; padding:0.2cm;">
1
</td>
</tr>
<tr>
<td style="padding:0.2cm;  text-align:left; vertical-align:middle;">
Below Target
</td>
<td style="padding:0.2cm; text-align:center; ">
<span style="color:black;">304</span><br><span style="color:#333399;">94.1 %</span>
</td>
<td style="padding:0.2cm; text-align:center; ">
<span style="color:black;">19</span><br><span style="color:#333399;">5.9 %</span>
</td>
<td style="padding:0.2cm; text-align:center;  ">
<span style="color:black;">323</span><br><span style="color:#333399;">100 %</span>
</td>
</tr>
<tr>
<td style="padding:0.2cm;  text-align:left; vertical-align:middle;">
At or Above Target
</td>
<td style="padding:0.2cm; text-align:center; ">
<span style="color:black;">15</span><br><span style="color:#333399;">19.5 %</span>
</td>
<td style="padding:0.2cm; text-align:center; ">
<span style="color:black;">62</span><br><span style="color:#333399;">80.5 %</span>
</td>
<td style="padding:0.2cm; text-align:center;  ">
<span style="color:black;">77</span><br><span style="color:#333399;">100 %</span>
</td>
</tr>
<tr>
<td style="padding:0.2cm;  border-bottom:double; font-weight:bolder; font-style:italic; text-align:left; vertical-align:middle;">
Total
</td>
<td style="padding:0.2cm; text-align:center;   border-bottom:double;">
<span style="color:black;">319</span><br><span style="color:#333399;">79.8 %</span>
</td>
<td style="padding:0.2cm; text-align:center;   border-bottom:double;">
<span style="color:black;">81</span><br><span style="color:#333399;">20.2 %</span>
</td>
<td style="padding:0.2cm; text-align:center;   border-bottom:double;">
<span style="color:black;">400</span><br><span style="color:#333399;">100 %</span>
</td>
</tr>
</table>

### Accuracy

``` r
accuracy <- sum(diag(ctab))/sum(ctab)*100
accuracy
```

    ## [1] 91.5

## Sample write up

Using a binary logistic regression, we explored how school’s characteristics (% fully credential teachers, studnet enrollment, and % on free and reduced lunch) relate to the adequate progress of a school.
Both the percent of credentialed teachers (*z* = 2.06, *p* = .04) and the % free and reduced lunch (*z* = -.11, *p* \< .01) were significant predictors.
Specifically the percent of fully credentialed teachers has an *OR* = 1.05, 95%CI\[1.01, 1.12\].
This implies that for a one percent increase in the fully credentialed teachers, schools have a 5% higher chance of making target.
Free and reduced lunch had estimated *OR* =.90, 95% CI \[.86, .92\], which indicated that for each additional percent of students on free and reduced lunch, there is a significant decrease in the log odds of being at target (*z* = -7.60, *p* \< .01).
That is, for each additional percent increase of free or reduced lunch, that school has a 11% decrease in the odds of making it to target growth, controlling for the other predictors.
There was no effect of the enrollment of the school, when controlling for the other variables.
Overall, this model had high prediction accuracy, predicting 91% schools accurately.

Hope this is useful….
