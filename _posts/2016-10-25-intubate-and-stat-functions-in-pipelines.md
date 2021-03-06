---
layout: post
title: "intubate <||> R stat functions in data science pipelines"
author: "Roberto Bertolusso"
categories: [intubate]
tags: [intubate, magrittr, data science, R, r-project, statistics]
date: "2016-10-25"
---



The aim of `intubate` (logo `<||>`) is to offer a painless way to
add R functions that are non-pipe-aware
to data science pipelines implemented by `magrittr` with the
operator `%>%`, without having to rely on workarounds of
varying complexity.

### Installation

* the latest released version from CRAN (1.0.0) with


```r
install.packages("intubate")
```

* the latest development version from github (1.4.0) with


```r
# install.packages("devtools")
devtools::install_github("rbertolusso/intubate")
```


### Pipelines
`dplyr`, by Hadley Wickham, Romain Francois, and RStudio,
is used here to illustrate data transformations.

Suppose you have the following code:


```r
library(dplyr)

tmp <- filter(LifeCycleSavings, dpi >= 1000)
to_fit <- select(tmp, sr, pop15, pop75)
```

and you would like to avoid creating the temporary object `tmp`.
One approach could be the following:


```r
to_fit <- select(filter(LifeCycleSavings, dpi >= 1000), sr, pop15, pop75)
```

The problem with this approach is that, as the number of intermediate
steps increase, it is error prone and becomes more complicated to understand.

Pipes in R are made possible by the package `magrittr`,
by Stefan Milton Bache and Hadley Wickham. They provide
an elegant alternative:


```r
library(magrittr)

LifeCycleSavings %>% 
  filter(dpi >= 1000) %>% 
  select(sr, pop15, pop75) ->
to_fit
```

Pipelines seem to be a popular way, these days, of doing data
science in R. If you need an introduction about pipelines, please follow this link (<http://r4ds.had.co.nz/transform.html>) to the chapter on data transformation of the
forthcoming book "R for Data Science" by Garrett Grolemund and Hadley Wickham.

### R statistical functions and pipelines
Suppose you want to perform a regression analysis
of `sr` on `pop15` and `pop75` (assuming
for the sake of argument that it is a valid analysis
to perform).

As most R functions are not pipeline-aware, you
should do something like the following:


```r
fitted <- lm(sr ~ ., to_fit)
summary(fitted)
```

```
## 
## Call:
## lm(formula = sr ~ ., data = to_fit)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -6.5438 -2.1996  0.4071  2.2060  5.4754 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  38.5981     9.6146   4.015 0.000898 ***
## pop15        -0.6574     0.2481  -2.650 0.016843 *  
## pop75        -2.7315     1.2458  -2.193 0.042536 *  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 3.558 on 17 degrees of freedom
## Multiple R-squared:  0.3213,	Adjusted R-squared:  0.2415 
## F-statistic: 4.024 on 2 and 17 DF,  p-value: 0.03709
```

This is an absolutely correct approach.

But what if, in addition to the data transformation, you
would also like to perform your data modeling/analysis under the
same pipeline paradigm (by adding lm to it),
which would impart notation consistency and
would avoid the need of creating the temporary object `to_fit`?


```r
LifeCycleSavings %>% 
  filter(dpi >= 1000) %>% 
  select(sr, pop15, pop75) %>% 
  lm(sr ~ .) %>%               ## Adding lm to the pipeline
  summary()
```

```
## Error in as.data.frame.default(data): cannot coerce class ""formula"" to a data.frame
```

You get an **error**.

The reason of this failure is that pipeline-aware functions (such as the ones
in `dplyr` that were specifically designed to work in pipelines) receive the data as
the **first** parameter, and most
statistical procedures (or graphical functions such as the ones in package `lattice`) that work with **formulas** to specify the **model**,
such as `lm` and lots of other rock solid reliable functions that implement
well established  statistical procedures, receive the data as the **second**
parameter.

There are alternatives that allow
to include `lm` (and others) in the pipeline without errors and without `intubate`.
They require workarounds
of varying levels of complexity. Some of the possible approaches are illustrated in the post <a href="https://rbertolusso.github.io/posts/stat-functions-in-pipelines-without-intubate">Workarounds to include R stat functions in data science pipelines</a>.

If you choose `intubate` is because you do not want to bother about workarounds when working with pipelines that include statistical procedures, or other non-pipe-aware functions.

By the way, `intubate` also implements three extensions for pipelines called `intubOrders`, `intuEnv`, and `intuBags`. These extensions will be treated in
following posts.

### intubate

* The original aim of `intubate` is to offer a *painless* way to add R functions that are *non-pipe-aware* to data science pipelines implemented by 'magrittr' with the operator %>%, without having to rely on *workarounds* of varying complexity.


```r
## install.packages("intubate")
library(intubate)
```

* To this end, `intubate` provides *interfaces*
(such as `ntbt_lm`) that let you do:


```r
LifeCycleSavings %>% 
  filter(dpi >= 1000) %>% 
  select(sr, pop15, pop75) %>%
  ntbt_lm(sr ~ pop15 + pop75) %>% 
  summary()
```

```
## 
## Call:
## lm(formula = sr ~ pop15 + pop75)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -6.5438 -2.1996  0.4071  2.2060  5.4754 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  38.5981     9.6146   4.015 0.000898 ***
## pop15        -0.6574     0.2481  -2.650 0.016843 *  
## pop75        -2.7315     1.2458  -2.193 0.042536 *  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 3.558 on 17 degrees of freedom
## Multiple R-squared:  0.3213,	Adjusted R-squared:  0.2415 
## F-statistic: 4.024 on 2 and 17 DF,  p-value: 0.03709
```

without error.

<br />

* With `intubate`, you can push boundaries and do things like:

```r
LifeCycleSavings %>% 
  filter(dpi >= 1000) %>% 
  select(sr, pop15, pop75) %>%
  ntbt_lm(sr ~ pop15 + pop75) %>% 
  ntbt_plot(which = 1) %>%        ## Adding a residual plot
  summary()
```

<img src="/figure/source/2016-10-25-intubate-and-stat-functions-in-pipelines/unnamed-chunk-10-1.png" title="plot of chunk unnamed-chunk-10" alt="plot of chunk unnamed-chunk-10" style="display: block; margin: auto;" />

```
## 
## Call:
## lm(formula = sr ~ pop15 + pop75)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -6.5438 -2.1996  0.4071  2.2060  5.4754 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  38.5981     9.6146   4.015 0.000898 ***
## pop15        -0.6574     0.2481  -2.650 0.016843 *  
## pop75        -2.7315     1.2458  -2.193 0.042536 *  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 3.558 on 17 degrees of freedom
## Multiple R-squared:  0.3213,	Adjusted R-squared:  0.2415 
## F-statistic: 4.024 on 2 and 17 DF,  p-value: 0.03709
```
(as `plot` returns `NULL`, `intubate` automatically forwards
its input so `summary` receives the result of `lm`).

<br />

* Currently, intubate provides more than 450 interfaces to 88 data science related packages in CRAN or in R installation (list of packages in Appendix).

<hr />

* Moreover, the user can:
    - call non-pipe-aware functions "on the fly" without
    needing to create an interface;
    - create interfaces "on demand".

### Calling non-pipe-aware functions "on the fly"

* If you do not want to use interfaces, you can use the function
`ntbt` to call the non-pipe-aware functions directly "on the fly":

```r
LifeCycleSavings %>% 
  ntbt(lm, sr ~ pop15 + pop75)
```

```
## 
## Call:
## lm(formula = sr ~ pop15 + pop75)
## 
## Coefficients:
## (Intercept)        pop15        pop75  
##     30.6277      -0.4708      -1.9341
```
Note: this approach works with any function, including the ones lacking
interfaces.

<br />

* For example, `lsfit` does not currently have an interface provided
  by `intubate`, but you still can call it "on the fly" with `ntbt`:
  

```r
LifeCycleSavings %>%
  ntbt_plot(pop75, sr) %>%
  ntbt(lsfit, pop75, sr) %>%    # Calling lsfit "on the fly" with ntbt
  abline()
```

<img src="/figure/source/2016-10-25-intubate-and-stat-functions-in-pipelines/unnamed-chunk-12-1.png" title="plot of chunk unnamed-chunk-12" alt="plot of chunk unnamed-chunk-12" style="display: block; margin: auto;" />
  
### Creating interfaces "on demand"

* If you like to use interfaces, and `intubate` does not provide one,
you can create your own "on demand". For example, to create an
interface to `lsfit`, all that is needed is the following line:


```r
ntbt_lsfit <- intubate
```

The *only* thing you need to remember is that the name of an interface
*must start* with `ntbt_` followed by the name of the *interfaced* function
(`lsfit` in this particular case), no matter which function you want to
interface.

<br />

You can now use the newly created interface as any other provided
by `intubate`:


```r
LifeCycleSavings %>%
  ntbt_plot(pop75, sr) %>%
  ntbt_lsfit(pop75, sr) %>%    # Using just created "on demand" interface
  abline()
```

<img src="/figure/source/2016-10-25-intubate-and-stat-functions-in-pipelines/unnamed-chunk-14-1.png" title="plot of chunk unnamed-chunk-14" alt="plot of chunk unnamed-chunk-14" style="display: block; margin: auto;" />


Just in case, let's clarify that the `intubate` machinery does not perform any
statistical computation. The *interfaced* functions
(those that are already well tested) are the ones performing the computations.

### Non-formula variants:
Some functions offer non-formula variants (or both variants). For example,
including `cor.test` in a pipeline in any of its variants produces an error:


```r
LifeCycleSavings %>% 
  filter(dpi >= 1000) %>% 
  select(sr, pop15, pop75) %>%
  cor.test(pop15, pop75)   ## Non-formula variant
```

```
## Error in match.arg(alternative): object 'pop75' not found
```

or:

```r
LifeCycleSavings %>% 
  filter(dpi >= 1000) %>% 
  select(sr, pop15, pop75) %>%
  cor.test(~ pop15 + pop75)   ## Formula variant
```

```
## Error in cor.test.default(., ~pop15 + pop75): 'x' and 'y' must have the same length
```

Both variants work when using any of the approaches provided by `intubate`:

```r
LifeCycleSavings %>% 
  filter(dpi >= 1000) %>% 
  select(sr, pop15, pop75) %>%
  ntbt_cor.test(pop15, pop75)   ## Non-formula variant
```

```
## 
## 	Pearson's product-moment correlation
## 
## data:  pop15 and pop75
## t = -2.4193, df = 18, p-value = 0.02636
## alternative hypothesis: true correlation is not equal to 0
## 95 percent confidence interval:
##  -0.76924958 -0.06766132
## sample estimates:
##        cor 
## -0.4953505
```

or:

```r
LifeCycleSavings %>% 
  filter(dpi >= 1000) %>% 
  select(sr, pop15, pop75) %>%
  ntbt(cor.test, ~ pop15 + pop75)   ## Formula variant
```

```
## 
## 	Pearson's product-moment correlation
## 
## data:  pop15 and pop75
## t = -2.4193, df = 18, p-value = 0.02636
## alternative hypothesis: true correlation is not equal to 0
## 95 percent confidence interval:
##  -0.76924958 -0.06766132
## sample estimates:
##        cor 
## -0.4953505
```

## Appendix

### Packages containing interfaces

The 88 R packages that have interfaces implemented so far are:

* `adabag`: Multiclass AdaBoost.M1, SAMME and Bagging
* `AER`: Applied Econometrics with R
* `aod`: Analysis of Overdispersed Data
* `ape`: Analyses of Phylogenetics and Evolution
* `arm`: Data Analysis Using Regression and Multilevel/Hierarchical Models
* `betareg`: Beta Regression
* `brglm`: Bias reduction in binomial-response generalized linear models
* `caper`: Comparative Analyses of Phylogenetics and Evolution in R
* `car`: Companion to Applied Regression
* `caret`: Classification and Regression Training
* `coin`: Conditional Inference Procedures in a Permutation Test Framework
* `CORElearn`: Classification, Regression and Feature Evaluation
* `drc`: Analysis of Dose-Response Curves
* `e1071`: Support Vector Machines
* `earth`: Multivariate Adaptive Regression Splines
* `EnvStats`: Environmental Statistics, Including US EPA Guidance
* `fGarch`: Rmetrics - Autoregressive Conditional Heteroskedastic Modelling
* `flexmix`: Flexible Mixture Modeling
* `forecast`: Forecasting Functions for Time Series and Linear Models
* `frontier`: Stochastic Frontier Analysis
* `gam`: Generalized Additive Models
* `gbm`: Generalized Boosted Regression Models
* `gee`: Generalized Estimation Equation Solver
* `glmnet`: Lasso and Elastic-Net Regularized Generalized Linear Models
* `glmx`: Generalized Linear Models Extended
* `gmnl`: Multinomial Logit Models with Random Parameters
* `gplots`: Various R Programming Tools for Plotting Data
* `gss`: General Smoothing Splines
* `graphics`: The R Graphics Package
* `hdm`: High-Dimensional Metrics
* `Hmisc`: Harrell Miscellaneous
* `ipred`: Improved Predictors
* `iRegression`: Regression Methods for Interval-Valued Variables
* `ivfixed`: Instrumental fixed effect panel data model
* `kernlab`: Kernel-Based Machine Learning Lab
* `kknn`: Weighted k-Nearest Neighbors
* `klaR`: Classification and Visualization
* `lars`: Least Angle Regression, Lasso and Forward Stagewise
* `lattice`: Trellis Graphics for R
* `latticeExtra`: Extra Graphical Utilities Based on Lattice
* `leaps`: Regression Subset Selection
* `lfe`: Linear Group Fixed Effects
* `lme4`: Linear Mixed-Effects Models using 'Eigen' and S4
* `lmtest`: Testing Linear Regression Models
* `MASS`: Robust Regression, Linear Discriminant Analysis, Ridge Regression,
          Probit Regression, ...
* `MCMCglmm`: MCMC Generalised Linear Mixed Models
* `mda`: Mixture and Flexible Discriminant Analysis
* `metafor`: Meta-Analysis Package for R
* `mgcv`: Mixed GAM Computation Vehicle with GCV/AIC/REML Smoothness Estimation
* `minpack.lm`: R Interface to the Levenberg-Marquardt Nonlinear Least-Squares
                Algorithm Found in MINPACK, Plus Support for Bounds
* `mhurdle`: Multiple Hurdle Tobit Models
* `mlogit`: Multinomial logit model
* `mnlogit`: Multinomial Logit Model
* `modeltools`: Tools and Classes for Statistical Models
* `nlme`: Linear and Nonlinear Mixed Effects Models
* `nlreg`: Higher Order Inference for Nonlinear Heteroscedastic Models
* `nnet`: Feed-Forward Neural Networks and Multinomial Log-Linear Models
* `ordinal`: Regression Models for Ordinal Data
* `party`: A Laboratory for Recursive Partytioning
* `partykit`: A Toolkit for Recursive Partytioning
* `plotrix`: Various Plotting Functions
* `pls`: Partial Least Squares and Principal Component Regression
* `pROC`: Display and Analyze ROC Curves
* `pscl`: Political Science Computational Laboratory, Stanford University
* `psychomix`: Psychometric Mixture Models
* `psychotools`: Infrastructure for Psychometric Modeling
* `psychotree`: Recursive Partitioning Based on Psychometric Models
* `quantreg`: Quantile Regression
* `randomForest`: Random Forests for Classification and Regression
* `Rchoice`: Discrete Choice (Binary, Poisson and Ordered) Models with Random Parameters
* `rminer`: Data Mining Classification and Regression Methods 
* `rms`: Regression Modeling Strategies
* `robustbase`: Basic Robust Statistics
* `rpart`: Recursive Partitioning and Regression Trees
* `RRF`: Regularized Random Forest
* `RWeka`: R/Weka Interface
* `sampleSelection`: Sample Selection Models
* `sem`: Structural Equation Models
* `spBayes`: Univariate and Multivariate Spatial-temporal Modeling
* `stats`: The R Stats Package (glm, lm, loess, lqs, nls, ...)
* `strucchange`: Testing, Monitoring, and Dating Structural Changes
* `survey`: Analysis of Complex Survey Samples
* `survival`: Survival Analysis
* `SwarmSVM`: Ensemble Learning Algorithms Based on Support Vector Machines
* `systemfit`: Estimating Systems of Simultaneous Equations
* `tree`: Classification and Regression Trees
* `vcd`: Visualizing Categorical Data
* `vegan`: Community Ecology Package

### Bugs and Feature requests
The robustness and generality of the interfacing machinery still needs to be
further *verified* (and very likely improved),
as there are thousands of potential functions to interface and
certainly some are bound to fail when interfaced. Some have already been addressed
when implementing provided interfaces (as their examples failed).

The goal is to make `intubate` each time more robust by
addressing the peculiarities of newly discovered failing functions.

For the time being, only cases where the
*interfaces provided with* `intubate` *fail* will be considered as *bugs*.

Cases of failing *user defined interfaces* or when using `ntbt` to call functions
directly that do not have interfaces provided with released versions of `intubate`,
will be considered *feature requests*.

Of course, it will be greatly appreciated,
if you have some coding skills and can follow the code of the interface,
if you could provide the proposed *solution*, that *shouldn't break anything else*,
together with the feature request.

### Next

<a href="https://rbertolusso.github.io/posts/stat-functions-in-pipelines-without-intubate">Workarounds to include R stat functions in data science pipelines</a>
