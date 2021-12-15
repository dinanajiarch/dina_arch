---
title: Nonlinear Least Squares Example
author: ''
date: '2021-03-15'
slug: nonlinear-least-squares-example
categories: R Project
tags: 
- r markdown
- data analysis
- visualization
subtitle: ''
summary: ''
authors: []
lastmod: '2021-12-14T17:56:44-08:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

## Parameter Estimation - Wild Fish Catch

**Source**: Source: Global wild fish catch and aquaculture production, compiled by Earth Policy Institute within 1950-2010 from U.N. Food and Agriculture Organization (FAO), Global Capture Production and Global Aquaculture Production, electronic databases, at www.fao.org/fishery/topic/16140/en.

“For Task 2, you will find an equation with parameters estimated by nonlinear least squares for the increase in global wild fish catch from 1950 – 2012.”

### Data Wrangling

Read in data and recode year for analysis set up:

``` r
fish_df <- read_csv("fish_catch.csv") %>% 
  mutate(year_coded = 0:62) #recoded year for analyses
```

### A & B. Graph of wild catch over time

Create an exploratory graph over wild catch over time:

``` r
fish_df %>% 
  ggplot(aes(x = year, y = wild_catch_mil_tons)) +
  geom_point() +
  theme_minimal() +
  labs(x = "Years", y = "Wild Catch (Million Tons)", title = "Wild Catch Over Time (Years)")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-2-1.png" width="672" />

``` r
#Log transformed
fish_df %>% 
  ggplot(aes(x = year, y = log(wild_catch_mil_tons))) +
  geom_point() +
  theme_minimal() +
  labs(x = "Years", y = "ln(Wild Catch)", title = "Wild Catch Over Time (Years)")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-2-2.png" width="672" />

In this plot of wild catch over time, a possible logistic growth relatiosnhip describes the trend:

`$$P(t) = \frac{K} {1 + Ae^{-kt}}$$`
where

`$$A = \frac{K - P_0} {P_0}$$`

My initial estimates for the parameters in the model are: `\(K\)` = 90, `\(P_0\)` = 20, `\(k\)` = .05, `\(A\)` = 3.5.

### C. Nonlinear least squares

Using nonlinear least squares (NLS) to find parameters for the wild catch model. First, we estimate `\(k\)`:

``` r
df_exp <- fish_df %>% 
  filter(year < 1990) %>% 
  mutate(ln_fish = log(wild_catch_mil_tons))
  
lm_k <- lm(ln_fish ~ year, data = df_exp)
lm_k
```

    ## 
    ## Call:
    ## lm(formula = ln_fish ~ year, data = df_exp)
    ## 
    ## Coefficients:
    ## (Intercept)         year  
    ##   -66.38553      0.03562

``` r
# Coefficient (k) ~ 0.04
```

The initial coefficient `\(k\)` = 0.4.

Then, we use `stats::nls()` with the estimated starting parameter values:

``` r
df_nls <- nls(wild_catch_mil_tons ~ K/(1 + A*exp(-r*year_coded)),
              data = fish_df,
              start = list(K = 90, A = 3.5, r = 0.4),
              trace = FALSE
              )

# See the model summary
#summary(df_nls)

# Use broom:: functions to get model outputs in tidier format: 
model_out <- broom::tidy(df_nls) 

model_out %>% 
  gt()
```

<div id="nbvbqfdjmn" style="overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>html {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Helvetica Neue', 'Fira Sans', 'Droid Sans', Arial, sans-serif;
}

#nbvbqfdjmn .gt_table {
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

#nbvbqfdjmn .gt_heading {
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

#nbvbqfdjmn .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#nbvbqfdjmn .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 0;
  padding-bottom: 6px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#nbvbqfdjmn .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#nbvbqfdjmn .gt_col_headings {
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

#nbvbqfdjmn .gt_col_heading {
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

#nbvbqfdjmn .gt_column_spanner_outer {
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

#nbvbqfdjmn .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#nbvbqfdjmn .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#nbvbqfdjmn .gt_column_spanner {
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

#nbvbqfdjmn .gt_group_heading {
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

#nbvbqfdjmn .gt_empty_group_heading {
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

#nbvbqfdjmn .gt_from_md > :first-child {
  margin-top: 0;
}

#nbvbqfdjmn .gt_from_md > :last-child {
  margin-bottom: 0;
}

#nbvbqfdjmn .gt_row {
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

#nbvbqfdjmn .gt_stub {
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

#nbvbqfdjmn .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#nbvbqfdjmn .gt_first_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
}

#nbvbqfdjmn .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#nbvbqfdjmn .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#nbvbqfdjmn .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#nbvbqfdjmn .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#nbvbqfdjmn .gt_footnotes {
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

#nbvbqfdjmn .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding: 4px;
}

#nbvbqfdjmn .gt_sourcenotes {
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

#nbvbqfdjmn .gt_sourcenote {
  font-size: 90%;
  padding: 4px;
}

#nbvbqfdjmn .gt_left {
  text-align: left;
}

#nbvbqfdjmn .gt_center {
  text-align: center;
}

#nbvbqfdjmn .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#nbvbqfdjmn .gt_font_normal {
  font-weight: normal;
}

#nbvbqfdjmn .gt_font_bold {
  font-weight: bold;
}

#nbvbqfdjmn .gt_font_italic {
  font-style: italic;
}

#nbvbqfdjmn .gt_super {
  font-size: 65%;
}

#nbvbqfdjmn .gt_footnote_marks {
  font-style: italic;
  font-weight: normal;
  font-size: 65%;
}
</style>
<table class="gt_table">
  
  <thead class="gt_col_headings">
    <tr>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1">term</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1">estimate</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1">std.error</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1">statistic</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_right" rowspan="1" colspan="1">p.value</th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr><td class="gt_row gt_left">K</td>
<td class="gt_row gt_right">100.27835748</td>
<td class="gt_row gt_right">2.734132433</td>
<td class="gt_row gt_right">36.67648</td>
<td class="gt_row gt_right">8.569260e-43</td></tr>
    <tr><td class="gt_row gt_left">A</td>
<td class="gt_row gt_right">4.31633154</td>
<td class="gt_row gt_right">0.292941193</td>
<td class="gt_row gt_right">14.73446</td>
<td class="gt_row gt_right">1.340788e-21</td></tr>
    <tr><td class="gt_row gt_left">r</td>
<td class="gt_row gt_right">0.06988688</td>
<td class="gt_row gt_right">0.004648183</td>
<td class="gt_row gt_right">15.03531</td>
<td class="gt_row gt_right">5.141952e-22</td></tr>
  </tbody>
  
  
</table>
</div>

The model with estimated parameters for wild catch over time is:
`$$P(t) = \frac{100.3}{1+4.32e^{-0.07t}}$$`

### D) Visualize model showing both original data and model output

``` r
# Make predictions for the wild catch at years: 
p_predict <- predict(df_nls)

# Bind predictions to original data frame:
df_complete <- data.frame(fish_df, p_predict)

# Plot them all together:
ggplot(data = df_complete, aes(x = year, y = wild_catch_mil_tons)) +
  geom_point() +
  geom_line(aes(x = year, y = p_predict)) +
  theme_calc() +
  scale_x_continuous(breaks = seq(1950, 2012, 10)) +
  labs(title = "Wild Catch (Million Tons) Over Time", x = "Year", y = "Wild Catch (Million Tons)")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-5-1.png" width="672" />
