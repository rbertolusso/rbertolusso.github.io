---
layout: post
title: intubate <||> 1.4.0 (GPL >= 2)
---

The aim of `intubate` (logo `<||>`) is to offer a painless way to
add R functions that are non-pipe-aware
to data science pipelines implemented by `magrittr` with the
operator `%>%`, without having to rely on workarounds of
varying complexity. It also
implements three extensions called `intubOrders`, `intuEnv`, and `intuBags`.

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

## Introduction to pipelines (`%>%`)

### Strategies to perform data transformations in data science

* We will use the data `LifeCycleSavings` to illustrate examples:

```r
data(LifeCycleSavings)
head(LifeCycleSavings)
```

```
##              sr pop15 pop75     dpi ddpi
## Australia 11.43 29.35  2.87 2329.68 2.87
## Austria   12.07 23.32  4.41 1507.99 3.93
## Belgium   13.17 23.80  4.43 2108.47 3.82
## Bolivia    5.75 41.89  1.67  189.13 0.22
## Brazil    12.88 42.19  0.83  728.47 4.56
## Canada     8.79 31.72  2.85 2982.88 2.43
```

<br />

#### 1. Using direct subsetting:

```r
LifeCycleSavings[LifeCycleSavings$dpi >= 1000,
                 c("sr", "pop15", "pop75")]
```

```
##                   sr pop15 pop75
## Australia      11.43 29.35  2.87
## Austria        12.07 23.32  4.41
## Belgium        13.17 23.80  4.43
## Canada          8.79 31.72  2.85
## Denmark        16.85 24.42  3.93
## Finland        11.24 27.84  2.37
## France         12.64 25.06  4.70
## Germany        12.55 23.31  3.35
## Iceland         1.27 34.03  3.08
## Ireland        11.34 31.16  4.19
## Italy          14.28 24.52  3.48
## Japan          21.10 27.01  1.91
## Luxembourg     10.35 21.80  3.73
## Norway         10.25 25.95  3.67
## Netherlands    14.65 24.71  3.25
## New Zealand    10.67 32.61  3.17
## Sweden          6.86 21.44  4.54
## Switzerland    14.13 23.49  3.73
## United Kingdom  7.81 23.27  4.46
## United States   7.56 29.81  3.43
```

<br />

#### 2. Using `base` R `subset`:

```r
subset(LifeCycleSavings, dpi >= 1000,
       select=c(sr, pop15, pop75))
```

```
##                   sr pop15 pop75
## Australia      11.43 29.35  2.87
## Austria        12.07 23.32  4.41
## Belgium        13.17 23.80  4.43
## Canada          8.79 31.72  2.85
## Denmark        16.85 24.42  3.93
## Finland        11.24 27.84  2.37
## France         12.64 25.06  4.70
## Germany        12.55 23.31  3.35
## Iceland         1.27 34.03  3.08
## Ireland        11.34 31.16  4.19
## Italy          14.28 24.52  3.48
## Japan          21.10 27.01  1.91
## Luxembourg     10.35 21.80  3.73
## Norway         10.25 25.95  3.67
## Netherlands    14.65 24.71  3.25
## New Zealand    10.67 32.61  3.17
## Sweden          6.86 21.44  4.54
## Switzerland    14.13 23.49  3.73
## United Kingdom  7.81 23.27  4.46
## United States   7.56 29.81  3.43
```

<br />

#### 3. Using `dplyr`:

```r
## install.packages("dplyr")
library(dplyr)

A <- filter(LifeCycleSavings, dpi >= 1000)
B <- select(A, sr, pop15, pop75)
B
```

```
##       sr pop15 pop75
## 1  11.43 29.35  2.87
## 2  12.07 23.32  4.41
## 3  13.17 23.80  4.43
## 4   8.79 31.72  2.85
## 5  16.85 24.42  3.93
## 6  11.24 27.84  2.37
## 7  12.64 25.06  4.70
## 8  12.55 23.31  3.35
## 9   1.27 34.03  3.08
## 10 11.34 31.16  4.19
## 11 14.28 24.52  3.48
## 12 21.10 27.01  1.91
## 13 10.35 21.80  3.73
## 14 10.25 25.95  3.67
## 15 14.65 24.71  3.25
## 16 10.67 32.61  3.17
## 17  6.86 21.44  4.54
## 18 14.13 23.49  3.73
## 19  7.81 23.27  4.46
## 20  7.56 29.81  3.43
```

<br />

#### 4. Avoiding creation of temporary variables:

```r
select(filter(LifeCycleSavings, dpi >= 1000), sr, pop15, pop75)
```

```
##       sr pop15 pop75
## 1  11.43 29.35  2.87
## 2  12.07 23.32  4.41
## 3  13.17 23.80  4.43
## 4   8.79 31.72  2.85
## 5  16.85 24.42  3.93
## 6  11.24 27.84  2.37
## 7  12.64 25.06  4.70
## 8  12.55 23.31  3.35
## 9   1.27 34.03  3.08
## 10 11.34 31.16  4.19
## 11 14.28 24.52  3.48
## 12 21.10 27.01  1.91
## 13 10.35 21.80  3.73
## 14 10.25 25.95  3.67
## 15 14.65 24.71  3.25
## 16 10.67 32.61  3.17
## 17  6.86 21.44  4.54
## 18 14.13 23.49  3.73
## 19  7.81 23.27  4.46
## 20  7.56 29.81  3.43
```

<br />

#### 5. Using `magrittr` pipes:

Pipelines in R are made possible by the package `magrittr`,
by Stefan Milton Bache and Hadley Wickham.


```r
## install.packages("magrittr")
library(magrittr)

LifeCycleSavings %>% 
  filter(dpi >= 1000) %>% 
  select(sr, pop15, pop75)
```

```
##       sr pop15 pop75
## 1  11.43 29.35  2.87
## 2  12.07 23.32  4.41
## 3  13.17 23.80  4.43
## 4   8.79 31.72  2.85
## 5  16.85 24.42  3.93
## 6  11.24 27.84  2.37
## 7  12.64 25.06  4.70
## 8  12.55 23.31  3.35
## 9   1.27 34.03  3.08
## 10 11.34 31.16  4.19
## 11 14.28 24.52  3.48
## 12 21.10 27.01  1.91
## 13 10.35 21.80  3.73
## 14 10.25 25.95  3.67
## 15 14.65 24.71  3.25
## 16 10.67 32.61  3.17
## 17  6.86 21.44  4.54
## 18 14.13 23.49  3.73
## 19  7.81 23.27  4.46
## 20  7.56 29.81  3.43
```

<br />

### Non-pipe-aware functions and pipelines
* What if you want to include to the pipeline a statistical function,
such as a correlation test?


```r
LifeCycleSavings %>% 
      filter(dpi >= 1000) %>% 
      select(sr, pop15, pop75) %>%
      cor.test(pop15, pop75)
```

```
## Error in match.arg(alternative): object 'pop75' not found
```

or:

```r
LifeCycleSavings %>% 
      filter(dpi >= 1000) %>% 
      select(sr, pop15, pop75) %>%
      cor.test(~ pop15 + pop75)
```

```
## Error in cor.test.default(., ~pop15 + pop75): 'x' and 'y' must have the same length
```

To be able to add `cor.test` to the pipeline without error,
you need to employ a non-trivial workaround that adds complexity
and can be error prone, and difficult to interpret
(more on workarounds at the end).

### Solution proposed by `intubate`
* The original aim of `intubate` is to offer a *painless* way to add R functions that are *non-pipe-aware* to data science pipelines implemented by 'magrittr' with the operator %>%, without having to rely on *workarounds* of varying complexity.


```r
## install.packages("intubate")
library(intubate)
```

* To this end, `intubate` provides *interfaces*
(such as `ntbt_cor.test`) that let you do:


```r
LifeCycleSavings %>% 
  filter(dpi >= 1000) %>% 
  select(sr, pop15, pop75) %>%
  ntbt_cor.test(pop15, pop75)     ## 'x' 'y' variant
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

and:

```r
LifeCycleSavings %>% 
  filter(dpi >= 1000) %>% 
  select(sr, pop15, pop75) %>%
  ntbt_cor.test(~ pop15 + pop75)  ## Formula variant
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

without error.

<br />

* If, for example, you want to perform a regression analysis using `lm`,
you can use the provided interface `ntbt_lm`:

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

<img src="/images/2016-10-05-intubate-talk-ensor-group_files/figure-html/unnamed-chunk-15-1.png" style="display: block; margin: auto;" />

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

<img src="/images/2016-10-05-intubate-talk-ensor-group_files/figure-html/unnamed-chunk-17-1.png" style="display: block; margin: auto;" />
  
### Creating interfaces "on demand"

* If you like to use interfaces, and `intubate` does not provide one,
you can create your own "on demand". For example, to create an
interface to `lsfit`, all that is needed is the following line:


```r
ntbt_lsfit <- intubate
```

The *only* thing you need to remember is that the name of an interface
*must start* with `ntbt_` followed by the name of the *interfaced* function
(`cor.test` in this particular case), no matter which function you want to
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

<img src="/images/2016-10-05-intubate-talk-ensor-group_files/figure-html/unnamed-chunk-19-1.png" style="display: block; margin: auto;" />


## Extensions for pipelines provided by `intubate`

`intubate` implements three extensions:

* `intubOrders`,
* `intuEnv`, and
* `intuBags`.

## `intubOrders`
`intubOrders` allow, among other things, to:

* run, in place, functions on the input (`data`)
  to the interfaced function, such as `head`, `tail`, `dim`, `str`, `View`, ...

* run, in place, functions that use the result generated by the interfaced
  function, such as `print`, `summary`, `anova`, `plot`, ...
  
* forward the input to the interfaced function without using %T>%

* signal other modifications to the behavior of the interface

`intubOrders` are implemented by an `intuBorder`: `<||>` (from where the
logo of `intubate` originates).

The `intuBorder` contains 5 zones (`intuZones`?, maybe too much...):

`zone 1` **<** `zone 2` **|** `zone 3` **|** `zone 4` **>** `zone 5`

* `zone 1` and `zone 5` will be explained later;

* `zone 2` is used to indicate the functions that are to be applied
  to the **input** to the interfaced function;
  
* `zone 3` to modify the behavior of the interface;

* `zone 4` to indicate the functions that are to be applied to the
  **result** of the interfaced function.

<br />

For example, instead of running the following sequence
of function calls (results not shown):


```r
head(LifeCycleSavings)
tail(LifeCycleSavings, n = 3)
dim(LifeCycleSavings)
str(LifeCycleSavings)
summary(LifeCycleSavings)
result <- lm(sr ~ pop15 + pop75 + dpi + ddpi, LifeCycleSavings)
print(result)
summary(result)
anova(result)
plot(result, which = 1)
```

you could have run, using an `intubOrder`:


```r
LifeCycleSavings %>%
  ntbt_lm(sr ~ pop15 + pop75 + dpi + ddpi,
          "< head; tail(#, n = 3); dim; str; summary
             |i|
             print; summary; anova; plot(#, which = 1) >")
```

```
## 
## ntbt_lm(data = ., sr ~ pop15 + pop75 + dpi + ddpi)
## 
## * head(#) <||> input *
##              sr pop15 pop75     dpi ddpi
## Australia 11.43 29.35  2.87 2329.68 2.87
## Austria   12.07 23.32  4.41 1507.99 3.93
## Belgium   13.17 23.80  4.43 2108.47 3.82
## Bolivia    5.75 41.89  1.67  189.13 0.22
## Brazil    12.88 42.19  0.83  728.47 4.56
## Canada     8.79 31.72  2.85 2982.88 2.43
## 
## * tail(#, n = 3) <||> input *
##            sr pop15 pop75    dpi  ddpi
## Uruguay  9.24 28.13  2.72 766.54  1.88
## Libya    8.89 43.69  2.07 123.58 16.71
## Malaysia 4.71 47.20  0.66 242.69  5.08
## 
## * dim(#) <||> input *
## [1] 50  5
## 
## * str(#) <||> input *
## 'data.frame':	50 obs. of  5 variables:
##  $ sr   : num  11.43 12.07 13.17 5.75 12.88 ...
##  $ pop15: num  29.4 23.3 23.8 41.9 42.2 ...
##  $ pop75: num  2.87 4.41 4.43 1.67 0.83 2.85 1.34 0.67 1.06 1.14 ...
##  $ dpi  : num  2330 1508 2108 189 728 ...
##  $ ddpi : num  2.87 3.93 3.82 0.22 4.56 2.43 2.67 6.51 3.08 2.8 ...
## 
## * summary(#) <||> input *
##        sr             pop15           pop75            dpi         
##  Min.   : 0.600   Min.   :21.44   Min.   :0.560   Min.   :  88.94  
##  1st Qu.: 6.970   1st Qu.:26.21   1st Qu.:1.125   1st Qu.: 288.21  
##  Median :10.510   Median :32.58   Median :2.175   Median : 695.66  
##  Mean   : 9.671   Mean   :35.09   Mean   :2.293   Mean   :1106.76  
##  3rd Qu.:12.617   3rd Qu.:44.06   3rd Qu.:3.325   3rd Qu.:1795.62  
##  Max.   :21.100   Max.   :47.64   Max.   :4.700   Max.   :4001.89  
##       ddpi       
##  Min.   : 0.220  
##  1st Qu.: 2.002  
##  Median : 3.000  
##  Mean   : 3.758  
##  3rd Qu.: 4.478  
##  Max.   :16.710  
## 
## * print(#) <||> result *
## 
## Call:
## lm(formula = sr ~ pop15 + pop75 + dpi + ddpi)
## 
## Coefficients:
## (Intercept)        pop15        pop75          dpi         ddpi  
##  28.5660865   -0.4611931   -1.6914977   -0.0003369    0.4096949  
## 
## 
## * summary(#) <||> result *
## 
## Call:
## lm(formula = sr ~ pop15 + pop75 + dpi + ddpi)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -8.2422 -2.6857 -0.2488  2.4280  9.7509 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)    
## (Intercept) 28.5660865  7.3545161   3.884 0.000334 ***
## pop15       -0.4611931  0.1446422  -3.189 0.002603 ** 
## pop75       -1.6914977  1.0835989  -1.561 0.125530    
## dpi         -0.0003369  0.0009311  -0.362 0.719173    
## ddpi         0.4096949  0.1961971   2.088 0.042471 *  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 3.803 on 45 degrees of freedom
## Multiple R-squared:  0.3385,	Adjusted R-squared:  0.2797 
## F-statistic: 5.756 on 4 and 45 DF,  p-value: 0.0007904
## 
## 
## * anova(#) <||> result *
## Analysis of Variance Table
## 
## Response: sr
##           Df Sum Sq Mean Sq F value    Pr(>F)    
## pop15      1 204.12 204.118 14.1157 0.0004922 ***
## pop75      1  53.34  53.343  3.6889 0.0611255 .  
## dpi        1  12.40  12.401  0.8576 0.3593551    
## ddpi       1  63.05  63.054  4.3605 0.0424711 *  
## Residuals 45 650.71  14.460                      
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

<img src="/images/2016-10-05-intubate-talk-ensor-group_files/figure-html/unnamed-chunk-21-1.png" style="display: block; margin: auto;" />

* Note:
    - `i` is used to force an invisible result
    - `#` is used as a placeholder either for the input or result in
      cases the input or result are not the first parameter, or the
      call requires extra parameters.

<hr />
* `intubOrders` may prove to be of interest to non-pipeline oriented people too
 (only plot shown):


```r
ntbt_lm(LifeCycleSavings, sr ~ pop15 + pop75 + dpi + ddpi,
        "< head; tail(#, n = 3); dim; str; summary
           |i|
           print; summary; anova; plot(#, which = 1) >")
```

<img src="/images/2016-10-05-intubate-talk-ensor-group_files/figure-html/unnamed-chunk-22-1.png" style="display: block; margin: auto;" />

<br />

### `intubOrders` with collections of inputs

When using pipelines, the receiving function has to deal with the *whole* object
that receives as its input. Then, it produces a result that, again, needs to be
consumed as a whole by the following function.

`intubOrders` allow you to work with a collection of objects of any kind in *one* pipeline, selecting at each step which input to use.

As an example, suppose you want to perform the following statistical procedures in
one pipeline (output not shown).


```r
CO2 %>%
  ntbt_lm(conc ~ uptake)

USJudgeRatings %>%
  ntbt_cor.test(CONT, INTG)

sleep %>%
  ntbt_t.test(extra ~ group)
```

We will first create a collection (a `list` in this case, but it could also be
`intuEnv` or an `intuBag`, explained later) containing the three dataframes:


```r
coll <- list(CO3 = CO2,
             USJudgeRatings1 = USJudgeRatings,
             sleep1 = sleep)
names(coll)
```

```
## [1] "CO3"             "USJudgeRatings1" "sleep1"
```

(We have changed the names to show we are not cheating...)

* Note: the objects of the collection *must* be named.

We will now use as source the *whole collection*.

The `intubOrder` will need the following info:

* `zone 1`, in each case, indicates which is the data.frame (or any other object)
  that we want to use as input in this particular function
* `zone 3` needs to include `f` to *forward* the input (if you want the next
  function to receive the whole collection, and not the result of this step)
* `zone 4` (optional) may contain a `print` (or `summary`) if you want
  something to be displayed


```r
coll %>%
  ntbt_lm(conc ~ uptake, "CO3 <|f| print >") %>%
  ntbt_cor.test(CONT, INTG, "USJudgeRatings1 <|f| print >") %>%
  ntbt_t.test(extra ~ group, "sleep1 <|f| print >") %>%
  names()
```

```
## 
## ntbt_lm(data = ., conc ~ uptake)
## 
## * print(#) <||> result *
## 
## Call:
## lm(formula = conc ~ uptake)
## 
## Coefficients:
## (Intercept)       uptake  
##       73.71        13.28  
## 
## 
## ntbt_cor.test(data = ., CONT, INTG)
## 
## * print(#) <||> result *
## 
## 	Pearson's product-moment correlation
## 
## data:  CONT and INTG
## t = -0.8605, df = 41, p-value = 0.3945
## alternative hypothesis: true correlation is not equal to 0
## 95 percent confidence interval:
##  -0.4168591  0.1741182
## sample estimates:
##        cor 
## -0.1331909 
## 
## 
## ntbt_t.test(data = ., extra ~ group)
## 
## * print(#) <||> result *
## 
## 	Welch Two Sample t-test
## 
## data:  extra by group
## t = -1.8608, df = 17.776, p-value = 0.07939
## alternative hypothesis: true difference in means is not equal to 0
## 95 percent confidence interval:
##  -3.3654832  0.2054832
## sample estimates:
## mean in group 1 mean in group 2 
##            0.75            2.33
```

```
## [1] "CO3"             "USJudgeRatings1" "sleep1"
```

* Note: `names()` was added at the end to show that we have forwarded
  the original collection to the end of the pipeline.

What happens if you would like to **save** the results of the function calls
(or intermediate results of data manipulations)?

<br />

## `intuEnv` and `intuBags`

`intuEnv` and `intuBags` allow to save intermediate results without
leaving the pipeline. They can also be used to contain the collections
of objects.

Let us first consider

## `intuEnv`

When `intubate` is loaded, it creates `intuEnv`, an empty environment that can
be populated with results that you want to use later.

You can access the `intuEnv` as follows:


```r
intuEnv()  ## intuEnv() returns invisibly, so nothing is output
```

You can verify that, initially, it is empty:


```r
ls(intuEnv())
```

```
## character(0)
```

How can `intuEnv` be used?

Suppose that we want, instead of, or in addition to, displaying the results of interfaced functions, save the objects returned by them.
One strategy is to save the results to `intuEnv` (the other is using `intuBags`).

#### How to save to `intuEnv`?

The `intubOrder` will need the following info:

* `zone 3` needs to include `f` to *forward* the input (if you want the next
  function to receive the whole collection, and not its result)
* `zone 5`, in each case, indicates the name that the result will have in the
  `intuEnv`


```r
coll %>%
  ntbt_lm(conc ~ uptake, "CO3 <|f|> lmfit") %>%
  ntbt_cor.test(CONT, INTG, "USJudgeRatings1 <|f|> ctres") %>%
  ntbt_t.test(extra ~ group, "sleep1 <|f|> ttres") %>%
  names()
```

```
## [1] "CO3"             "USJudgeRatings1" "sleep1"
```

As you can see, the collection stays unchanged, but look
inside `intuEnv`


```r
ls(intuEnv())
```

```
## [1] "ctres" "lmfit" "ttres"
```

`intuEnv` has collected the results, that are ready for use.

Four strategies of using one of the collected results
are shown below (output not shown):

#### Strategy 1

```r
intuEnv()$lmfit %>%
  summary()
```

#### Strategy 2

```r
attach(intuEnv())
lmfit %>%
  summary()
detach()
```

#### Strategy 3

```r
intuEnv() %>%
  ntbt(summary, "lmfit <||>")
```

#### Strategy 4

```r
intuEnv() %>%
  ntbt(I, "lmfit <|i| summary >")
```

`clear_intuEnv` can be used to empty the contents of `intuEnv`.


```r
clear_intuEnv()

ls(intuEnv())
```

```
## character(0)
```

### Associating `intuEnv` with the Global Environment

If you want your results to be saved to the Global environment (it could be
*any* environment), you can associate `intuEnv` to it, so you can have your
results available as any other saved object.

First let's display the contents of the Global environment:


```r
ls()
```

```
## [1] "A"                "B"                "coll"            
## [4] "LifeCycleSavings" "ntbt_lsfit"
```

`set_intuEnv` let's you associate `intuEnv` to an environment. It takes
an environment as parameter, and returns the current `intuEnv`, in case you
want to save it to reinstate it later. If not, I think it will be just
garbage collected (I may be wrong).

Let's associate `intuEnv` to the global environment (saving the current
`intuEnv`):


```r
saved_intuEnv <- set_intuEnv(globalenv())
```

Now, we re-run the pipeline:


```r
coll %>%
  ntbt_lm(conc ~ uptake, "CO3 <|f|> lmfit") %>%
  ntbt_cor.test(CONT, INTG, "USJudgeRatings1 <|f|> ctres") %>%
  ntbt_t.test(extra ~ group, "sleep1 <|f|> ttres") %>%
  names()
```

```
## [1] "CO3"             "USJudgeRatings1" "sleep1"
```

Before forgetting, let's reinstate the original `intuEnv`:


```r
set_intuEnv(saved_intuEnv)    ## set_intuEnv() returns invisibly
```

And now, let's see if the results were saved to the global environment:


```r
ls()
```

```
## [1] "A"                "B"                "coll"            
## [4] "ctres"            "LifeCycleSavings" "lmfit"           
## [7] "ntbt_lsfit"       "saved_intuEnv"    "ttres"
```

They were.

Now the results are at your disposal to use as any other variable (result not
shown):


```r
lmfit %>%
  summary()
```

### Using `intuEnv` as source of the pipeline

You can use `intuEnv` (or any other environment) as the input
of your pipeline.

We already cleared the contents of `intuEnv`, but let's do it
again to get used to how to do it:


```r
clear_intuEnv()

ls(intuEnv())
```

```
## character(0)
```

Let's populate `intuEnv` with the same objects as before:


```r
intuEnv(CO3 = CO2,
        USJudgeRatings1 = USJudgeRatings,
        sleep1 = sleep)

ls(intuEnv())
```

```
## [1] "CO3"             "sleep1"          "USJudgeRatings1"
```

When using an environment, such as `intuEnv`, as the source of your pipeline,
there is no need to specify `f` in `zone 3`, as the environment is always forwarded
(the same happens when the source is an `intuBag`).

Keep in mind that, if you are saving results and your source is an environment
other than `intuEnv`, the results will be saved to `intuEnv`, and not to the source
enviromnent. If the source is an `intuBag`, the results will be saved to the
`intuBag`, and not to `intuEnv`.

We will run the same pipeline as before, but this time we will add `subset`
and `summary`(called directly with `ntbt`) to illustrate how we can use a previously
generated result (such as from data transformations) in the *same* pipeline in which
it was generated, when using `intuEnv` (or an `intuBag`) as the source of the pipeline.


```r
intuEnv() %>%
  ntbt(subset, Treatment == "nonchilled", "CO3 <||> CO3nc") %>%
  ntbt_lm(conc ~ uptake, "CO3nc <||> lmfit") %>%
  ntbt_cor.test(CONT, INTG, "USJudgeRatings1 <||> ctres") %>%
  ntbt_t.test(extra ~ group, "sleep1 <||> ttres") %>%
  ntbt(summary, "lmfit <||> lmsfit") %>%
  names()
```

```
## [1] "USJudgeRatings1" "ttres"           "CO3nc"           "ctres"          
## [5] "lmsfit"          "lmfit"           "sleep1"          "CO3"
```

* Note that, as `subset` is already pipe-aware (`data` is its first parameter),
  you have two ways of proceeding. One is the one illustrated
  above (same strategy used on non-pipe-aware functions). The other, that
  works *only* when using pipe-aware functions, is:


```r
intuEnv() %>%
  ntbt(subset, CO3, Treatment == "nonchilled", "<||> CO3nc")
```

<br />

## `intuBags`

`intuBags` differ from `intEnv` in that they are based on lists, instead than
on environments. Even if (with a little of care) you could keep track of several
`intuEnvs`, it seems natural (to me) to deal with only one, while several `intuBags`
(for example one for each database, or collection of objects) seem natural (to me).
`intuEnv` (being a function call) can be called directly from inside functions
(it always knows where the environment is), so you don't have to send it as an
argument, as in the case of an `intuBag`.

Other than that, using an `intuEnv` or an `intuBag` is a matter of 
personal taste.

What you can do with one you can do with the other.


```r
iBag <- intuBag(CO3 = CO2,
                USJudgeRatings1 = USJudgeRatings,
                sleep1 = sleep)
```



```r
iBag %>%
  ntbt(subset, Treatment == "nonchilled", "CO3 <||> CO3nc") %>%
  ntbt_lm(conc ~ uptake, "CO3nc <||> lmfit") %>%
  ntbt_cor.test(CONT, INTG, "USJudgeRatings1 <||> ctres") %>%
  ntbt_t.test(extra ~ group, "sleep1 <||> ttres") %>%
  ntbt(summary, "lmfit <||> lmsfit") %>%
  names()
```

```
## [1] "CO3"             "USJudgeRatings1" "sleep1"          "CO3nc"          
## [5] "lmfit"           "ctres"           "ttres"           "lmsfit"
```

When using `intuBags`, it is possible to
use `%<>%` if you want to save your results to the `intuBag`.
This way, instead of a long pipeline, you could run several
short ones.


```r
iBag <- intuBag(CO3 = CO2,
                USJudgeRatings1 = USJudgeRatings,
                sleep1 = sleep)

iBag %<>%
  ntbt(subset, CO3, Treatment == "nonchilled", "<||> CO3nc") %>%
  ntbt_lm(conc ~ uptake, "CO3nc <||> lmfit")

iBag %<>%
  ntbt_cor.test(CONT, INTG, "USJudgeRatings1 <||> ctres")

iBag %<>%
  ntbt_t.test(extra ~ group, "sleep1 <||> ttres") %>%
  ntbt(summary, "lmfit <||> lmsfit")

names(iBag)
```

```
## [1] "CO3"             "USJudgeRatings1" "sleep1"          "CO3nc"          
## [5] "lmfit"           "ctres"           "ttres"           "lmsfit"
```
 
The `intuBag` will collect all your results, in any way you prefer to use it.

The same happens with `intuEnv`. Just remember that `%<>%` should *not*
be used with `intuEnv` (you should always use `%>%`).

### Using more than one source

Suppose you have a "database" containing the following two "tables":


```r
iBag <- intuBag(members = data.frame(name=c("John", "Paul", "George",
                                            "Ringo", "Brian", NA),
                band=c("TRUE",  "TRUE", "TRUE", "TRUE", "FALSE", NA)),
           what_played = data.frame(name=c("John", "Paul", "Ringo",
                                           "George", "Stuart", "Pete"),
                instrument=c("guitar", "bass", "drums", "guitar", "bass", "drums")))
print(iBag)
```

```
## $members
##     name  band
## 1   John  TRUE
## 2   Paul  TRUE
## 3 George  TRUE
## 4  Ringo  TRUE
## 5  Brian FALSE
## 6   <NA>  <NA>
## 
## $what_played
##     name instrument
## 1   John     guitar
## 2   Paul       bass
## 3  Ringo      drums
## 4 George     guitar
## 5 Stuart       bass
## 6   Pete      drums
## 
## attr(,"intuBag")
## [1] TRUE
```

and you want to perform an inner join. In these cases, the functions
should receive the whole `intuBag` (or `intuEnv`, or collection), so
`zone 1` should be empty, and the names of the tables should be specified
directly, in the function call, in their corresponding order
(or by stating their parameter names).


```r
iBag %>%
  ntbt(merge, members, what_played, by = "name", "<|| print >")
```

```
## 
## ntbt(data = ., fti = merge, members, what_played, by = "name")
## 
## * print(#) <||> result *
##     name band instrument
## 1 George TRUE     guitar
## 2   John TRUE     guitar
## 3   Paul TRUE       bass
## 4  Ringo TRUE      drums
```

### Example of an `intuBag` acting as a database

The following code has been extracted from chapter 13 of "R for data science",
by Garrett Grolemund and Hadley Wickham
(<http://r4ds.had.co.nz/relational-data.html>)

Original code (output not shown):


```r
library(dplyr)
library(nycflights13)
```


```r
flights2 <- flights %>% 
  select(year:day, hour, origin, dest, tailnum, carrier)
flights2

flights2 %>%
  select(-origin, -dest) %>% 
  left_join(airlines, by = "carrier")

## 13.4.5 Defining the key columns

flights2 %>%
  left_join(weather)

flights2 %>%
  left_join(planes, by = "tailnum")

flights2 %>%
  left_join(airports, c("dest" = "faa"))

flights2 %>%
  left_join(airports, c("origin" = "faa"))
```

`nycflights13` is a database. As such, we can deal with it using intuBags.
The following code illustrates how all the above can be performed using
an `intuBag` (or `intuEnv`) and *one* pipeline:


```r
iBag <- intuBag(flightsIB = flights,
                airlinesIB = airlines,
                weatherIB = weather,
                planesIB = planes,
                airportsIB = airports)
## Note we are changing the names, to make sure we are not cheating
## (by reading from globalenv()).

iBag %<>%
  ntbt(select, flightsIB, year:day, hour, origin, dest, tailnum, carrier, "<|| head > flights2") %>%
  ntbt(select, flights2, -origin, -dest, "<|| print > flights3") %>% 
  ntbt(left_join, flights3, airlinesIB, by = "carrier", "<|| print >") %>%
  ntbt(left_join, flights2, weatherIB, "<|| print >") %>%
  ntbt(left_join, flights2, planesIB, by = "tailnum", "<|| print >") %>%
  ntbt(left_join, flights2, airportsIB, c("dest" = "faa"), "<|| print >") %>%
  ntbt(left_join, flights2, airportsIB, c("origin" = "faa"), "<|| print >")
```

```
## 
## ntbt(data = ., fti = select, flightsIB, year:day, hour, origin, 
##     dest, tailnum, carrier)
## 
## * head(#) <||> flights2 *
## # A tibble: 6 × 8
##    year month   day  hour origin  dest tailnum carrier
##   <int> <int> <int> <dbl>  <chr> <chr>   <chr>   <chr>
## 1  2013     1     1     5    EWR   IAH  N14228      UA
## 2  2013     1     1     5    LGA   IAH  N24211      UA
## 3  2013     1     1     5    JFK   MIA  N619AA      AA
## 4  2013     1     1     5    JFK   BQN  N804JB      B6
## 5  2013     1     1     6    LGA   ATL  N668DN      DL
## 6  2013     1     1     5    EWR   ORD  N39463      UA
## 
## ntbt(data = ., fti = select, flights2, -origin, -dest)
## 
## * print(#) <||> flights3 *
## # A tibble: 336,776 × 6
##     year month   day  hour tailnum carrier
##    <int> <int> <int> <dbl>   <chr>   <chr>
## 1   2013     1     1     5  N14228      UA
## 2   2013     1     1     5  N24211      UA
## 3   2013     1     1     5  N619AA      AA
## 4   2013     1     1     5  N804JB      B6
## 5   2013     1     1     6  N668DN      DL
## 6   2013     1     1     5  N39463      UA
## 7   2013     1     1     6  N516JB      B6
## 8   2013     1     1     6  N829AS      EV
## 9   2013     1     1     6  N593JB      B6
## 10  2013     1     1     6  N3ALAA      AA
## # ... with 336,766 more rows
## 
## ntbt(data = ., fti = left_join, flights3, airlinesIB, by = "carrier")
## 
## * print(#) <||> result *
## # A tibble: 336,776 × 7
##     year month   day  hour tailnum carrier                     name
##    <int> <int> <int> <dbl>   <chr>   <chr>                    <chr>
## 1   2013     1     1     5  N14228      UA    United Air Lines Inc.
## 2   2013     1     1     5  N24211      UA    United Air Lines Inc.
## 3   2013     1     1     5  N619AA      AA   American Airlines Inc.
## 4   2013     1     1     5  N804JB      B6          JetBlue Airways
## 5   2013     1     1     6  N668DN      DL     Delta Air Lines Inc.
## 6   2013     1     1     5  N39463      UA    United Air Lines Inc.
## 7   2013     1     1     6  N516JB      B6          JetBlue Airways
## 8   2013     1     1     6  N829AS      EV ExpressJet Airlines Inc.
## 9   2013     1     1     6  N593JB      B6          JetBlue Airways
## 10  2013     1     1     6  N3ALAA      AA   American Airlines Inc.
## # ... with 336,766 more rows
## 
## ntbt(data = ., fti = left_join, flights2, weatherIB)
## 
## * print(#) <||> result *
## # A tibble: 336,776 × 18
##     year month   day  hour origin  dest tailnum carrier  temp  dewp humid
##    <dbl> <dbl> <int> <dbl>  <chr> <chr>   <chr>   <chr> <dbl> <dbl> <dbl>
## 1   2013     1     1     5    EWR   IAH  N14228      UA    NA    NA    NA
## 2   2013     1     1     5    LGA   IAH  N24211      UA    NA    NA    NA
## 3   2013     1     1     5    JFK   MIA  N619AA      AA    NA    NA    NA
## 4   2013     1     1     5    JFK   BQN  N804JB      B6    NA    NA    NA
## 5   2013     1     1     6    LGA   ATL  N668DN      DL 39.92 26.06 57.33
## 6   2013     1     1     5    EWR   ORD  N39463      UA    NA    NA    NA
## 7   2013     1     1     6    EWR   FLL  N516JB      B6 39.02 26.06 59.37
## 8   2013     1     1     6    LGA   IAD  N829AS      EV 39.92 26.06 57.33
## 9   2013     1     1     6    JFK   MCO  N593JB      B6 39.02 26.06 59.37
## 10  2013     1     1     6    LGA   ORD  N3ALAA      AA 39.92 26.06 57.33
## # ... with 336,766 more rows, and 7 more variables: wind_dir <dbl>,
## #   wind_speed <dbl>, wind_gust <dbl>, precip <dbl>, pressure <dbl>,
## #   visib <dbl>, time_hour <dttm>
## 
## ntbt(data = ., fti = left_join, flights2, planesIB, by = "tailnum")
## 
## * print(#) <||> result *
## # A tibble: 336,776 × 16
##    year.x month   day  hour origin  dest tailnum carrier year.y
##     <int> <int> <int> <dbl>  <chr> <chr>   <chr>   <chr>  <int>
## 1    2013     1     1     5    EWR   IAH  N14228      UA   1999
## 2    2013     1     1     5    LGA   IAH  N24211      UA   1998
## 3    2013     1     1     5    JFK   MIA  N619AA      AA   1990
## 4    2013     1     1     5    JFK   BQN  N804JB      B6   2012
## 5    2013     1     1     6    LGA   ATL  N668DN      DL   1991
## 6    2013     1     1     5    EWR   ORD  N39463      UA   2012
## 7    2013     1     1     6    EWR   FLL  N516JB      B6   2000
## 8    2013     1     1     6    LGA   IAD  N829AS      EV   1998
## 9    2013     1     1     6    JFK   MCO  N593JB      B6   2004
## 10   2013     1     1     6    LGA   ORD  N3ALAA      AA     NA
## # ... with 336,766 more rows, and 7 more variables: type <chr>,
## #   manufacturer <chr>, model <chr>, engines <int>, seats <int>,
## #   speed <int>, engine <chr>
## 
## ntbt(data = ., fti = left_join, flights2, airportsIB, c(dest = "faa"))
## 
## * print(#) <||> result *
## # A tibble: 336,776 × 14
##     year month   day  hour origin  dest tailnum carrier
##    <int> <int> <int> <dbl>  <chr> <chr>   <chr>   <chr>
## 1   2013     1     1     5    EWR   IAH  N14228      UA
## 2   2013     1     1     5    LGA   IAH  N24211      UA
## 3   2013     1     1     5    JFK   MIA  N619AA      AA
## 4   2013     1     1     5    JFK   BQN  N804JB      B6
## 5   2013     1     1     6    LGA   ATL  N668DN      DL
## 6   2013     1     1     5    EWR   ORD  N39463      UA
## 7   2013     1     1     6    EWR   FLL  N516JB      B6
## 8   2013     1     1     6    LGA   IAD  N829AS      EV
## 9   2013     1     1     6    JFK   MCO  N593JB      B6
## 10  2013     1     1     6    LGA   ORD  N3ALAA      AA
## # ... with 336,766 more rows, and 6 more variables: name <chr>, lat <dbl>,
## #   lon <dbl>, alt <int>, tz <dbl>, dst <chr>
## 
## ntbt(data = ., fti = left_join, flights2, airportsIB, c(origin = "faa"))
## 
## * print(#) <||> result *
## # A tibble: 336,776 × 14
##     year month   day  hour origin  dest tailnum carrier
##    <int> <int> <int> <dbl>  <chr> <chr>   <chr>   <chr>
## 1   2013     1     1     5    EWR   IAH  N14228      UA
## 2   2013     1     1     5    LGA   IAH  N24211      UA
## 3   2013     1     1     5    JFK   MIA  N619AA      AA
## 4   2013     1     1     5    JFK   BQN  N804JB      B6
## 5   2013     1     1     6    LGA   ATL  N668DN      DL
## 6   2013     1     1     5    EWR   ORD  N39463      UA
## 7   2013     1     1     6    EWR   FLL  N516JB      B6
## 8   2013     1     1     6    LGA   IAD  N829AS      EV
## 9   2013     1     1     6    JFK   MCO  N593JB      B6
## 10  2013     1     1     6    LGA   ORD  N3ALAA      AA
## # ... with 336,766 more rows, and 6 more variables: name <chr>, lat <dbl>,
## #   lon <dbl>, alt <int>, tz <dbl>, dst <chr>
```


```r
names(iBag)
```

```
## [1] "flightsIB"  "airlinesIB" "weatherIB"  "planesIB"   "airportsIB"
## [6] "flights2"   "flights3"
```

The same, using `intuEnv`, and avoiding creating flights3 (output not shown):


```r
clear_intuEnv()

intuEnv(flightsIB = flights,
        airlinesIB = airlines,
        weatherIB = weather,
        planesIB = planes,
        airportsIB = airports) %>%
  ntbt(select, flightsIB, year:day, hour, origin, dest, tailnum, carrier,
       "<|| head > flights2") %>%
  ntbt(left_join, select(flights2, -origin, -dest), airlinesIB, by = "carrier",
       "<|| print >") %>%
  ntbt(left_join, flights2, weatherIB, "<|| print >") %>%
  ntbt(left_join, flights2, planesIB, by = "tailnum", "<|| print >") %>%
  ntbt(left_join, flights2, airportsIB, c("dest" = "faa"), "<|| print >") %>%
  ntbt(left_join, flights2, airportsIB, c("origin" = "faa"), "<|| print >")
```


```r
ls(intuEnv())
```

```
## [1] "airlinesIB" "airportsIB" "flights2"   "flightsIB"  "planesIB"  
## [6] "weatherIB"
```

* Note: the book is still not published (as of 8/27/16), so the examples in the
        chapter may have changed by the time you are reading this.

### Saving results of function calls in `zone 2` and `zone 4` of the `intubOrder`:

Starting from version 1.2.0, you can save the results generated by functions
included in `zone 2` and `zone 4` of the `intubOrder`. The results will be
collected following the same rules as with a result specified in `zone 5`.

For example, the following case will be collected by `intubOrder`:


```r
clear_intuEnv()

LifeCycleSavings %>%
  ntbt_lm(sr ~ pop15 + pop75 + dpi + ddpi,
          "< head; LCSt <- tail(#, n = 3); dim; str; LCSs <- summary
             |i|
             print; sfit <- summary; afit <- anova > fit")
```

```
## 
## ntbt_lm(data = ., sr ~ pop15 + pop75 + dpi + ddpi)
## 
## * head(#) <||> input *
##              sr pop15 pop75     dpi ddpi
## Australia 11.43 29.35  2.87 2329.68 2.87
## Austria   12.07 23.32  4.41 1507.99 3.93
## Belgium   13.17 23.80  4.43 2108.47 3.82
## Bolivia    5.75 41.89  1.67  189.13 0.22
## Brazil    12.88 42.19  0.83  728.47 4.56
## Canada     8.79 31.72  2.85 2982.88 2.43
## 
## * LCSt <- tail(#, n = 3) <||> input *
##            sr pop15 pop75    dpi  ddpi
## Uruguay  9.24 28.13  2.72 766.54  1.88
## Libya    8.89 43.69  2.07 123.58 16.71
## Malaysia 4.71 47.20  0.66 242.69  5.08
## 
## * dim(#) <||> input *
## [1] 50  5
## 
## * str(#) <||> input *
## 'data.frame':	50 obs. of  5 variables:
##  $ sr   : num  11.43 12.07 13.17 5.75 12.88 ...
##  $ pop15: num  29.4 23.3 23.8 41.9 42.2 ...
##  $ pop75: num  2.87 4.41 4.43 1.67 0.83 2.85 1.34 0.67 1.06 1.14 ...
##  $ dpi  : num  2330 1508 2108 189 728 ...
##  $ ddpi : num  2.87 3.93 3.82 0.22 4.56 2.43 2.67 6.51 3.08 2.8 ...
## 
## * LCSs <- summary(#) <||> input *
##        sr             pop15           pop75            dpi         
##  Min.   : 0.600   Min.   :21.44   Min.   :0.560   Min.   :  88.94  
##  1st Qu.: 6.970   1st Qu.:26.21   1st Qu.:1.125   1st Qu.: 288.21  
##  Median :10.510   Median :32.58   Median :2.175   Median : 695.66  
##  Mean   : 9.671   Mean   :35.09   Mean   :2.293   Mean   :1106.76  
##  3rd Qu.:12.617   3rd Qu.:44.06   3rd Qu.:3.325   3rd Qu.:1795.62  
##  Max.   :21.100   Max.   :47.64   Max.   :4.700   Max.   :4001.89  
##       ddpi       
##  Min.   : 0.220  
##  1st Qu.: 2.002  
##  Median : 3.000  
##  Mean   : 3.758  
##  3rd Qu.: 4.478  
##  Max.   :16.710  
## 
## * print(#) <||> fit *
## 
## Call:
## lm(formula = sr ~ pop15 + pop75 + dpi + ddpi)
## 
## Coefficients:
## (Intercept)        pop15        pop75          dpi         ddpi  
##  28.5660865   -0.4611931   -1.6914977   -0.0003369    0.4096949  
## 
## 
## * sfit <- summary(#) <||> fit *
## 
## Call:
## lm(formula = sr ~ pop15 + pop75 + dpi + ddpi)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -8.2422 -2.6857 -0.2488  2.4280  9.7509 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)    
## (Intercept) 28.5660865  7.3545161   3.884 0.000334 ***
## pop15       -0.4611931  0.1446422  -3.189 0.002603 ** 
## pop75       -1.6914977  1.0835989  -1.561 0.125530    
## dpi         -0.0003369  0.0009311  -0.362 0.719173    
## ddpi         0.4096949  0.1961971   2.088 0.042471 *  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 3.803 on 45 degrees of freedom
## Multiple R-squared:  0.3385,	Adjusted R-squared:  0.2797 
## F-statistic: 5.756 on 4 and 45 DF,  p-value: 0.0007904
## 
## 
## * afit <- anova(#) <||> fit *
## Analysis of Variance Table
## 
## Response: sr
##           Df Sum Sq Mean Sq F value    Pr(>F)    
## pop15      1 204.12 204.118 14.1157 0.0004922 ***
## pop75      1  53.34  53.343  3.6889 0.0611255 .  
## dpi        1  12.40  12.401  0.8576 0.3593551    
## ddpi       1  63.05  63.054  4.3605 0.0424711 *  
## Residuals 45 650.71  14.460                      
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

```r
ls(intuEnv())
```

```
## [1] "afit" "fit"  "LCSs" "LCSt" "sfit"
```

By default, cases where you are saving results (on `zone 2` and `zone 4`) will be printed.

If you want to only assign, but not print, you need to specify `S` in `zone 3`:


```r
clear_intuEnv()

LifeCycleSavings %>%
  ntbt_lm(sr ~ pop15 + pop75 + dpi + ddpi,
          "< head; LCSt <- tail(#, n = 3); dim; str; LCSs <- summary
             |iS|
             print; sfit <- summary; afit <- anova > fit")
```

```
## 
## ntbt_lm(data = ., sr ~ pop15 + pop75 + dpi + ddpi)
## 
## * head(#) <||> input *
##              sr pop15 pop75     dpi ddpi
## Australia 11.43 29.35  2.87 2329.68 2.87
## Austria   12.07 23.32  4.41 1507.99 3.93
## Belgium   13.17 23.80  4.43 2108.47 3.82
## Bolivia    5.75 41.89  1.67  189.13 0.22
## Brazil    12.88 42.19  0.83  728.47 4.56
## Canada     8.79 31.72  2.85 2982.88 2.43
## 
## * dim(#) <||> input *
## [1] 50  5
## 
## * str(#) <||> input *
## 'data.frame':	50 obs. of  5 variables:
##  $ sr   : num  11.43 12.07 13.17 5.75 12.88 ...
##  $ pop15: num  29.4 23.3 23.8 41.9 42.2 ...
##  $ pop75: num  2.87 4.41 4.43 1.67 0.83 2.85 1.34 0.67 1.06 1.14 ...
##  $ dpi  : num  2330 1508 2108 189 728 ...
##  $ ddpi : num  2.87 3.93 3.82 0.22 4.56 2.43 2.67 6.51 3.08 2.8 ...
## 
## * print(#) <||> fit *
## 
## Call:
## lm(formula = sr ~ pop15 + pop75 + dpi + ddpi)
## 
## Coefficients:
## (Intercept)        pop15        pop75          dpi         ddpi  
##  28.5660865   -0.4611931   -1.6914977   -0.0003369    0.4096949
```

```r
ls(intuEnv())
```

```
## [1] "afit" "fit"  "LCSs" "LCSt" "sfit"
```

In the case the source is an `intuBag`, the result is collected, as expected, by the `intuBag`:


```r
iBag <- intuBag(CO3 = CO2,
                USJudgeRatings1 = USJudgeRatings,
                sleep1 = sleep)
iBag %<>%
  ntbt_lm(conc ~ uptake, "CO3 < CO3h <- head |S| sfit <- summary; afit <- anova > fit")
```

```
## 
## ntbt_lm(data = ., conc ~ uptake)
```

```r
names(iBag)
```

```
## [1] "CO3"             "USJudgeRatings1" "sleep1"          "CO3h"           
## [5] "sfit"            "afit"            "fit"
```

### Removing all interfaces defined in `package:intubate` environment
If you want to interface functions exclusively with `ntbt`, starting
from version 1.3.0 you can remove all the supplied interfaces (functions starting
with `ntbt_`) with the function `intubate_rm_all_interfaces`, that takes no
arguments. This will not remove the interfaces created "on demand" by the user in the
global environment (or any environment that is not the `package:intubate` environment).


```r
intubate_rm_all_interfaces()
```

This makes the footprint of `intubate` even smaller, if you are interested in
a minimalistic approach.

### Creating interfaces "on demand" in `package:intubate` environment
If you want to create your own interfaces "on demand", but you do not want to
pollute the global environment with the names, starting from version 1.4.0 there
is a new approach.

Instead of using:

```r
ntbt_fntointerface1 <- ntbt_fntointerface2 <- intubate
```

that creates the interfaces `ntbt_fntointerface1` and
`ntbt_fntointerface2` in the global environment (or any other 
environment where you are creating it), you can do:


```r
intubate(fntointerface1, "fntointerface2")
```

that creates the interfaces `ntbt_fntointerface1` and
`ntbt_fntointerface2` in the `package:intubate` environment (where
the interfaces provided by `intubate` are defined). This way your
global environment will only contain variables related to your work,
and not interfaces. As demonstrated above, you can supply either the names
directly without quotations, or the strings containing the names.

Interfaces created this way are removed if you call `intubate_rm_all_interfaces`.

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

### Logo of `intubate`
The logo of `intubate` is: **`<||>`**. It corresponds to an **intuBorder**. I have not
found it in a Google search as of 2016/08/08. I intend to use it as a visual
identification of `intubate`. If you know of it having being in use before this date
in any software related project, please let me know, and I will change it.

### Names used
*intuBorder(s)* and *intubOrder(s)*, as of 2016/08/08, only has been found, on Google,
in a snippet of code for the name of a variable (`intUBorder`) (http://www.office-loesung.de/ftopic246897_0_0_asc.php) that would mean something
like an "integer upper border". There is also an `intLBorder` for the lower border.

*intuBag(s)*, as of 2016/08/08, seems to be used for a small bag for bikes (InTuBag,
meaning Inner Tub Bag)
(https://felvarrom.com/products/intubag-bike-tube-bag-medium-blue-inside?variant=18439367751),
but not for anything software related. If `intubate` succeeds, they may end selling
more InTuBags!

*intubate*, as of 2016/08/08, seems to be used related to the medical procedure, perhaps
also by the oil pipeline industry (at least "entubar" in Spanish is more general than the
medical procedure), but not for software related projects.

*intuEnv*, as of 2016/08/18, was found only in some Latin text.

I intend to use "intubate", "<||>", "intuBorder", "intubOrder(s)", "intuBag(s)",
"intuEnv(s)"and other derivations starting with "intu", in relation to the use
and promotion of "intubate" for software related activities.

At some point I intend to register the names and logo as trademarks.

### What can you do if you do not want to use `intubate` and you still want to use non-pipe-aware functions in pipelines?

#### Example 1:
`lm` can still be added directly to the pipeline,
without error, by specifying the name of the parameter
associated with the model (`formula` in this case).

```r
tmp %>%
  lm(formula = stweight ~ stheight)
```

```
## Error in eval(expr, envir, enclos): object 'tmp' not found
```

(So what is the big fuss about `intubate`?)

The drawback of this approach is that not all functions
use `formula` to specify the model.

So far I have encountered 5 variants:

* `formula`
* `x`
* `object`
* `model`, and
* `fixed`

The following are examples of functions using the other variants.

#### Example 2:
Using `xyplot` directly in a data pipeline will raise an error

```r
library(lattice)
iris %>%
  xyplot(Sepal.Length + Sepal.Width ~ Petal.Length + Petal.Width | Species,
         scales = "free", layout = c(2, 2),
         auto.key = list(x = .6, y = .7, corner = c(0, 0)))
```

```
## Error in UseMethod("xyplot"): no applicable method for 'xyplot' applied to an object of class "data.frame"
```

unless `x` is specified.

```r
iris %>%
  xyplot(x = Sepal.Length + Sepal.Width ~ Petal.Length + Petal.Width | Species,
         scales = "free", layout = c(2, 2),
         auto.key = list(x = .6, y = .7, corner = c(0, 0)))
```

<img src="/images/2016-10-05-intubate-talk-ensor-group_files/figure-html/unnamed-chunk-64-1.png" style="display: block; margin: auto;" />

#### Example 3: 
Using `tmd` (a *different* function in the *same* package)
directly in a data pipeline will raise an error

```r
library(lattice)

iris %>%
  tmd(Sepal.Length + Sepal.Width ~ Petal.Length + Petal.Width | Species,
      scales = "free", layout = c(2, 2),
      auto.key = list(x = .6, y = .7, corner = c(0, 0)))
```

```
## Error in UseMethod("tmd"): no applicable method for 'tmd' applied to an object of class "data.frame"
```

unless `object` is specified.

```r
iris %>%
  tmd(object = Sepal.Length + Sepal.Width ~ Petal.Length + Petal.Width | Species,
      scales = "free", layout = c(2, 2),
      auto.key = list(x = .6, y = .7, corner = c(0, 0)))
```

<img src="/images/2016-10-05-intubate-talk-ensor-group_files/figure-html/unnamed-chunk-66-1.png" style="display: block; margin: auto;" />


#### Example 4:
Using `gls` directly in a data pipeline
            will raise an error

```r
library(nlme)
```

```
## 
## Attaching package: 'nlme'
```

```
## The following object is masked from 'package:dplyr':
## 
##     collapse
```

```r
Ovary %>%
  gls(follicles ~ sin(2*pi*Time) + cos(2*pi*Time),
      correlation = corAR1(form = ~ 1 | Mare))
```

```
## Error in gls(., follicles ~ sin(2 * pi * Time) + cos(2 * pi * Time), correlation = corAR1(form = ~1 | : 
## model must be a formula of the form "resp ~ pred"
```

unless `model` is specified.

```r
Ovary %>%
  gls(model = follicles ~ sin(2*pi*Time) + cos(2*pi*Time),
      correlation = corAR1(form = ~ 1 | Mare))
```

```
## Generalized least squares fit by REML
##   Model: follicles ~ sin(2 * pi * Time) + cos(2 * pi * Time) 
##   Data: . 
##   Log-restricted-likelihood: -780.7273
## 
## Coefficients:
##        (Intercept) sin(2 * pi * Time) cos(2 * pi * Time) 
##         12.2163982         -2.7747122         -0.8996047 
## 
## Correlation Structure: AR(1)
##  Formula: ~1 | Mare 
##  Parameter estimate(s):
##       Phi 
## 0.7532079 
## Degrees of freedom: 308 total; 305 residual
## Residual standard error: 4.616172
```


#### Example 5:
Using `lme` directly in a data pipeline
            will raise an error

```r
library(nlme)

Orthodont %>%
      lme(distance ~ age)
```

unless `fixed`(!) is specified.

```r
Orthodont %>%
  lme(fixed = distance ~ age)
```

```
## Linear mixed-effects model fit by REML
##   Data: . 
##   Log-restricted-likelihood: -221.3183
##   Fixed: distance ~ age 
## (Intercept)         age 
##  16.7611111   0.6601852 
## 
## Random effects:
##  Formula: ~age | Subject
##  Structure: General positive-definite
##             StdDev    Corr  
## (Intercept) 2.3270339 (Intr)
## age         0.2264276 -0.609
## Residual    1.3100399       
## 
## Number of Observations: 108
## Number of Groups: 27
```


 Having to remember the name of the
 parameter associated to the model in each case
 may be error prone, and gives an
 inconsistent look and feel to an otherwise elegant
 interface.
 
 Moreover, it is consider good practice 
 in R to not specify the name of the first two parameters, and
 name the remaining.
 
 Not having to specify the name of the
 model argument completely hides the heterogeneity of names
 that can be associated with it. You only write the model
 and completely forget which name has been assigned to it.

### More complicated workarounds
 There are functions that rely on the order of the parameters
 (such as `aggregate`, `cor.test` and other 28 I found so far) that will still
 raise an error *even if you name the model*.
 
 There are cases where it is *not
 true* that if in a function call you name the parameters
 you can write them in any order you want?
 
 One example is `cor.test`?

#### 1) Unnamed parameters in the natural order. Works

```r
cor.test(~ CONT + INTG, USJudgeRatings)
```

```
## 
## 	Pearson's product-moment correlation
## 
## data:  CONT and INTG
## t = -0.8605, df = 41, p-value = 0.3945
## alternative hypothesis: true correlation is not equal to 0
## 95 percent confidence interval:
##  -0.4168591  0.1741182
## sample estimates:
##        cor 
## -0.1331909
```
#### 2) Named parameters in the natural order. Works

```r
cor.test(formula = ~ CONT + INTG, data = USJudgeRatings)
```

```
## 
## 	Pearson's product-moment correlation
## 
## data:  CONT and INTG
## t = -0.8605, df = 41, p-value = 0.3945
## alternative hypothesis: true correlation is not equal to 0
## 95 percent confidence interval:
##  -0.4168591  0.1741182
## sample estimates:
##        cor 
## -0.1331909
```

#### 3) Named parameters with the order changed. Doesn't work!

```r
cor.test(data = USJudgeRatings, formula = ~ CONT + INTG)
```

```
## Error in cor.test.default(data = USJudgeRatings, formula = ~CONT + INTG): argument "x" is missing, with no default
```

Let's see what happens if we want to add these cases to the `%>%` pipeline.

#### Example of error 1: `cor.test`
Using cor.test directly in a data pipeline
       will raise an error

```r
USJudgeRatings %>%
  cor.test(~ CONT + INTG)
```

```
## Error in cor.test.default(., ~CONT + INTG): 'x' and 'y' must have the same length
```

*even* when specifying `formula` (as it should be according to
the documentation).

```r
USJudgeRatings %>%
  cor.test(formula = ~ CONT + INTG)
```

```
## Error in cor.test.default(., formula = ~CONT + INTG): argument "y" is missing, with no default
```

Was it `y` then?

```r
USJudgeRatings %>%
  cor.test(y = ~ CONT + INTG)
```

```
## Error in cor.test.default(., y = ~CONT + INTG): 'x' and 'y' must have the same length
```

No...

Was it `x` then?

```r
USJudgeRatings %>%
  cor.test(x = ~ CONT + INTG)
```

```
## Error in cor.test.formula(., x = ~CONT + INTG): 'formula' missing or invalid
```

No

#### Example of error 2: `aggregate`
Using `aggregate` directly in a data pipeline
         will raise an error

```r
ToothGrowth %>%
  aggregate(len ~ ., mean)
```

```
## Error in aggregate.data.frame(., len ~ ., mean): 'by' must be a list
```

even when specifying `formula`

```r
ToothGrowth %>%
  aggregate(formula=len ~ ., mean)
```

```
## Error in match.fun(FUN): argument "FUN" is missing, with no default
```

or other variants.

#### Example of error 3: `lda`
Using `lda` directly in a data pipeline
         will raise an error

```r
library(MASS)

Iris <- data.frame(rbind(iris3[,,1], iris3[,,2], iris3[,,3]),
                   Sp = rep(c("s","c","v"), rep(50,3)))
Iris %>%
  lda(Sp ~ .)
```

```
## Error in lda.default(x, grouping, ...): nrow(x) and length(grouping) are different
```

even when specifying `formula`.

```r
Iris %>%
  lda(formula = Sp ~ .)
```

```
## Error in lda.default(x, grouping, ...): argument "grouping" is missing, with no default
```

or other variants.

Let's try another strategy. Let's see
if the %$% operator, that
expands the names of the variables inside
the data structure, can be of help.

```r
Iris %$%
  lda(Sp ~ .)
```

```
## Error in terms.formula(formula, data = data): '.' in formula and no 'data' argument
```

Still no...

One last try...

```r
Iris %$%
  lda(Sp ~ Sepal.L. + Sepal.W. + Petal.L. + Petal.W.)
```

```
## Call:
## lda(Sp ~ Sepal.L. + Sepal.W. + Petal.L. + Petal.W.)
## 
## Prior probabilities of groups:
##         c         s         v 
## 0.3333333 0.3333333 0.3333333 
## 
## Group means:
##   Sepal.L. Sepal.W. Petal.L. Petal.W.
## c    5.936    2.770    4.260    1.326
## s    5.006    3.428    1.462    0.246
## v    6.588    2.974    5.552    2.026
## 
## Coefficients of linear discriminants:
##                 LD1         LD2
## Sepal.L. -0.8293776  0.02410215
## Sepal.W. -1.5344731  2.16452123
## Petal.L.  2.2012117 -0.93192121
## Petal.W.  2.8104603  2.83918785
## 
## Proportion of trace:
##    LD1    LD2 
## 0.9912 0.0088
```

**Finally!** But... we had to specify all the variables 
(and they may be a lot), and use `%$%` instead of `%>%`.

There is still another workaround that allows
these functions to be used directly in a pipeline.
It requires the use of another function (`with`)
encapsulating the offending function. Here it goes:


```r
Iris %>%
  with(lda(Sp ~ ., .))
```

```
## Call:
## lda(Sp ~ ., data = .)
## 
## Prior probabilities of groups:
##         c         s         v 
## 0.3333333 0.3333333 0.3333333 
## 
## Group means:
##   Sepal.L. Sepal.W. Petal.L. Petal.W.
## c    5.936    2.770    4.260    1.326
## s    5.006    3.428    1.462    0.246
## v    6.588    2.974    5.552    2.026
## 
## Coefficients of linear discriminants:
##                 LD1         LD2
## Sepal.L. -0.8293776  0.02410215
## Sepal.W. -1.5344731  2.16452123
## Petal.L.  2.2012117 -0.93192121
## Petal.W.  2.8104603  2.83918785
## 
## Proportion of trace:
##    LD1    LD2 
## 0.9912 0.0088
```

In the case of `aggregate` it goes like

```r
ToothGrowth %>%
  with(aggregate(len ~ ., ., mean))
```

```
##   supp dose   len
## 1   OJ  0.5 13.23
## 2   VC  0.5  7.98
## 3   OJ  1.0 22.70
## 4   VC  1.0 16.77
## 5   OJ  2.0 26.06
## 6   VC  2.0 26.14
```

In addition, there is the added complexity of
interpreting the meaning of each of those `.`
(unfortunately they do not mean the same)
which may cause confusion, particularly at a future
time when you may have to remember why you had to
do *this* to *yourself*.

 (the first is specifying to include in the
  rhs of the model all the variables in the data but `len`,
  the second is the name of the data
  structure passed by the pipe. Yes, it is called `.`!)

 Undoubtedly, there may be more elegant workarounds that
 I am unaware of. But the point is that, no matter how elegant,
 they will be, well,
 *still* workarounds. You want to *force* unbehaving functions
 into something that is unnatural to them:
 
* In one case you had to name the parameters,
* in the other you had to use `%$%` instead of `%>%` and where not allowed
 to use `.` in your model definition,
* if you wanted to use `%>%` you had to use
 also `which` and include `.` as the second parameter.
 
The idea of avoiding such "hacks"
 motivated me to write `intubate`.

### Which was, again, the `intubate` alternative?
(Well... if you insist...)

#### For Example 1:
No need to specify `formula`.

```r
tmp %>%
  ntbt_lm(stweight ~ stheight)
```

```
## Error in eval(expr, envir, enclos): object 'tmp' not found
```

#### For Example 2:
No need to specify `x`.

```r
iris %>%
  ntbt_xyplot(Sepal.Length + Sepal.Width ~ Petal.Length + Petal.Width | Species,
              scales = "free", layout = c(2, 2),
              auto.key = list(x = .6, y = .7, corner = c(0, 0)))
```

<img src="/images/2016-10-05-intubate-talk-ensor-group_files/figure-html/unnamed-chunk-87-1.png" style="display: block; margin: auto;" />

#### For Example 3:
No need to specify `object`.

```r
iris %>%
  ntbt_tmd(Sepal.Length + Sepal.Width ~ Petal.Length + Petal.Width | Species,
           scales = "free", layout = c(2, 2),
           auto.key = list(x = .6, y = .7, corner = c(0, 0)))
```

<img src="/images/2016-10-05-intubate-talk-ensor-group_files/figure-html/unnamed-chunk-88-1.png" style="display: block; margin: auto;" />

#### For Example 4:
No need to specify `model`.

```r
Ovary %>%
  ntbt_gls(follicles ~ sin(2*pi*Time) + cos(2*pi*Time),
           correlation = corAR1(form = ~ 1 | Mare))
```

```
## Generalized least squares fit by REML
##   Model: follicles ~ sin(2 * pi * Time) + cos(2 * pi * Time) 
##   Data: NULL 
##   Log-restricted-likelihood: -780.7273
## 
## Coefficients:
##        (Intercept) sin(2 * pi * Time) cos(2 * pi * Time) 
##         12.2163982         -2.7747122         -0.8996047 
## 
## Correlation Structure: AR(1)
##  Formula: ~1 | Mare 
##  Parameter estimate(s):
##       Phi 
## 0.7532079 
## Degrees of freedom: 308 total; 305 residual
## Residual standard error: 4.616172
```

#### For Example 5:
No need to specify `fixed`.

```r
Orthodont %>%
  ntbt_lme(distance ~ age)
```

```
## Linear mixed-effects model fit by REML
##   Data: . 
##   Log-restricted-likelihood: -221.3183
##   Fixed: distance ~ age 
## (Intercept)         age 
##  16.7611111   0.6601852 
## 
## Random effects:
##  Formula: ~age | Subject
##  Structure: General positive-definite
##             StdDev    Corr  
## (Intercept) 2.3270339 (Intr)
## age         0.2264276 -0.609
## Residual    1.3100399       
## 
## Number of Observations: 108
## Number of Groups: 27
```

#### For Example of error 1:
It simply works.

```r
USJudgeRatings %>%
  ntbt_cor.test(~ CONT + INTG)
```

```
## 
## 	Pearson's product-moment correlation
## 
## data:  CONT and INTG
## t = -0.8605, df = 41, p-value = 0.3945
## alternative hypothesis: true correlation is not equal to 0
## 95 percent confidence interval:
##  -0.4168591  0.1741182
## sample estimates:
##        cor 
## -0.1331909
```

#### For Example of error 2:
It simply works.

```r
ToothGrowth %>%
  ntbt_aggregate(len ~ ., mean)
```

```
##   supp dose   len
## 1   OJ  0.5 13.23
## 2   VC  0.5  7.98
## 3   OJ  1.0 22.70
## 4   VC  1.0 16.77
## 5   OJ  2.0 26.06
## 6   VC  2.0 26.14
```

#### For Example of error 3:
It simply works.

```r
Iris %>%
  ntbt_lda(Sp ~ .)
```

```
## Call:
## lda(Sp ~ ., data = .)
## 
## Prior probabilities of groups:
##         c         s         v 
## 0.3333333 0.3333333 0.3333333 
## 
## Group means:
##   Sepal.L. Sepal.W. Petal.L. Petal.W.
## c    5.936    2.770    4.260    1.326
## s    5.006    3.428    1.462    0.246
## v    6.588    2.974    5.552    2.026
## 
## Coefficients of linear discriminants:
##                 LD1         LD2
## Sepal.L. -0.8293776  0.02410215
## Sepal.W. -1.5344731  2.16452123
## Petal.L.  2.2012117 -0.93192121
## Petal.W.  2.8104603  2.83918785
## 
## Proportion of trace:
##    LD1    LD2 
## 0.9912 0.0088
```

 I think the approach `intubate` proposes
 looks consistent, elegant, simple and clean,
 less error prone, and easy to follow
 
 (But, keep in mind that I have a vested interest).

 After all, the complication should be in
 the analysis you are performing,
 and not in how you are performing it.


### Advanced example demonstrating two strategies of using `intubOrders`

This example uses the code provided in the vignette of
package `survey`, that can be found in:

<https://cran.r-project.org/web/packages/survey/vignettes/survey.pdf>

The original code on the vignette (output not shown) is:


```r
library(survey)

data(api)

vars<-names(apiclus1)[c(12:13,16:23,27:37)] 

dclus1 <- svydesign(id = ~dnum, weights = ~pw, data = apiclus1, fpc = ~fpc)
summary(dclus1)
svymean(~api00, dclus1)
svyquantile(~api00, dclus1, quantile=c(0.25,0.5,0.75), ci=TRUE)
svytotal(~stype, dclus1)
svytotal(~enroll, dclus1)
svyratio(~api.stu,~enroll, dclus1)
svyratio(~api.stu, ~enroll, design=subset(dclus1, stype=="H"))
svymean(make.formula(vars),dclus1,na.rm=TRUE)
svyby(~ell+meals, ~stype, design=dclus1, svymean)
regmodel <- svyglm(api00~ell+meals,design=dclus1)
logitmodel <- svyglm(I(sch.wide=="Yes")~ell+meals, design=dclus1, family=quasibinomial()) 
summary(regmodel)
summary(logitmodel)
```

<br />

Two strategies of using intubOrders are illustrated.

##### **Strategy 1**: long pipeline, light use of intubOrders:


```r
apiclus1 %>%
  ntbt(svydesign, id = ~dnum, weights = ~ pw, fpc = ~ fpc, "<|| summary >") %>%
  ntbt(svymean, ~ api00, "<|f| print >") %>%
  ntbt(svyquantile, ~ api00, quantile = c(0.25,0.5,0.75), ci = TRUE, "<|f| print >") %>%
  ntbt(svytotal, ~ stype, "<|f| print >") %>%
  ntbt(svytotal, ~ enroll, "<|f| print >") %>%
  ntbt(svyratio, ~ api.stu, ~ enroll, "<|f| print >") %>%
  ntbt(svyratio, ~ api.stu, ~ enroll, design = subset("#", stype == "H"), "<|f| print >") %>%
  ntbt(svymean, make.formula(vars), na.rm = TRUE, "<|f| print >") %>%
  ntbt(svyby, ~ ell + meals, ~ stype, svymean, "<|f| print >") %>%
  ntbt(svyglm, api00 ~ ell + meals, "<|f| summary >") %>%
  ntbt(svyglm, I(sch.wide == "Yes") ~ ell + meals, family = quasibinomial(), "<|f| summary >")
```

```
## 
## ntbt(data = ., fti = svydesign, id = ~dnum, weights = ~pw, fpc = ~fpc)
## 
## * summary(#) <||> result *
## 1 - level Cluster Sampling design
## With (15) clusters.
## svydesign(id = ~dnum, weights = ~pw, fpc = ~fpc, data = .)
## Probabilities:
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
## 0.02954 0.02954 0.02954 0.02954 0.02954 0.02954 
## Population size (PSUs): 757 
## Data variables:
##  [1] "cds"      "stype"    "name"     "sname"    "snum"     "dname"   
##  [7] "dnum"     "cname"    "cnum"     "flag"     "pcttest"  "api00"   
## [13] "api99"    "target"   "growth"   "sch.wide" "comp.imp" "both"    
## [19] "awards"   "meals"    "ell"      "yr.rnd"   "mobility" "acs.k3"  
## [25] "acs.46"   "acs.core" "pct.resp" "not.hsg"  "hsg"      "some.col"
## [31] "col.grad" "grad.sch" "avg.ed"   "full"     "emer"     "enroll"  
## [37] "api.stu"  "fpc"      "pw"      
## 
## ntbt(data = ., fti = svymean, ~api00)
## 
## * print(#) <||> result *
##         mean     SE
## api00 644.17 23.542
## 
## ntbt(data = ., fti = svyquantile, ~api00, quantile = c(0.25, 
##     0.5, 0.75), ci = TRUE)
## 
## * print(#) <||> result *
## $quantiles
##         0.25 0.5  0.75
## api00 551.75 652 717.5
## 
## $CIs
## , , api00
## 
##            0.25      0.5     0.75
## (lower 493.2835 564.3250 696.0000
## upper) 622.6495 710.8375 761.1355
## 
## 
## 
## ntbt(data = ., fti = svytotal, ~stype)
## 
## * print(#) <||> result *
##          total      SE
## stypeE 4873.97 1333.32
## stypeH  473.86  158.70
## stypeM  846.17  167.55
## 
## ntbt(data = ., fti = svytotal, ~enroll)
## 
## * print(#) <||> result *
##          total     SE
## enroll 3404940 932235
## 
## ntbt(data = ., fti = svyratio, ~api.stu, ~enroll)
## 
## * print(#) <||> result *
## Ratio estimator: svyratio.survey.design2(~api.stu, ~enroll, .)
## Ratios=
##            enroll
## api.stu 0.8497087
## SEs=
##              enroll
## api.stu 0.008386297
## 
## ntbt(data = ., fti = svyratio, ~api.stu, ~enroll, design = subset("#", 
##     stype == "H"))
## 
## * print(#) <||> result *
## Ratio estimator: svyratio.survey.design2(~api.stu, ~enroll, design = .res_expr.)
## Ratios=
##            enroll
## api.stu 0.8300683
## SEs=
##             enroll
## api.stu 0.01472607
## 
## ntbt(data = ., fti = svymean, make.formula(vars), na.rm = TRUE)
## 
## * print(#) <||> result *
##                   mean      SE
## api00       643.203822 25.4936
## api99       605.490446 25.4987
## sch.wideNo    0.127389  0.0247
## sch.wideYes   0.872611  0.0247
## comp.impNo    0.273885  0.0365
## comp.impYes   0.726115  0.0365
## bothNo        0.273885  0.0365
## bothYes       0.726115  0.0365
## awardsNo      0.292994  0.0397
## awardsYes     0.707006  0.0397
## meals        50.636943  6.6588
## ell          26.891720  2.1567
## yr.rndNo      0.942675  0.0358
## yr.rndYes     0.057325  0.0358
## mobility     17.719745  1.4555
## pct.resp     67.171975  9.6553
## not.hsg      23.082803  3.1976
## hsg          24.847134  1.1167
## some.col     25.210191  1.4709
## col.grad     20.611465  1.7305
## grad.sch      6.229299  1.5361
## avg.ed        2.621529  0.1054
## full         87.127389  2.1624
## emer         10.968153  1.7612
## enroll      573.713376 46.5959
## api.stu     487.318471 41.4182
## 
## ntbt(data = ., fti = svyby, ~ell + meals, ~stype, svymean)
## 
## * print(#) <||> result *
##   stype      ell    meals   se.ell se.meals
## E     E 29.69444 53.09028 1.411617 7.070399
## H     H 15.00000 37.57143 5.347065 5.912262
## M     M 22.68000 43.08000 2.952862 6.017110
## 
## ntbt(data = ., fti = svyglm, api00 ~ ell + meals)
## 
## * summary(#) <||> result *
## 
## Call:
## svyglm(formula = api00 ~ ell + meals, .)
## 
## Survey design:
## svydesign(id = ~dnum, weights = ~pw, fpc = ~fpc, data = .)
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept) 817.1823    18.6709  43.768 1.32e-14 ***
## ell          -0.5088     0.3259  -1.561    0.144    
## meals        -3.1456     0.3018 -10.423 2.29e-07 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for gaussian family taken to be 3161.207)
## 
## Number of Fisher Scoring iterations: 2
## 
## 
## ntbt(data = ., fti = svyglm, I(sch.wide == "Yes") ~ ell + meals, 
##     family = quasibinomial())
## 
## * summary(#) <||> result *
## 
## Call:
## svyglm(formula = I(sch.wide == "Yes") ~ ell + meals, ., family = quasibinomial())
## 
## Survey design:
## svydesign(id = ~dnum, weights = ~pw, fpc = ~fpc, data = .)
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)   
## (Intercept)  1.899557   0.509915   3.725  0.00290 **
## ell          0.039925   0.012443   3.209  0.00751 **
## meals       -0.019115   0.008825  -2.166  0.05117 . 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for quasibinomial family taken to be 0.9627734)
## 
## Number of Fisher Scoring iterations: 5
```

<br />

##### **Strategy 2**: short pipeline, heavy use of *one* intubOrder:


```r
apiclus1 %>%
  ntbt(svydesign, id = ~dnum, weights = ~pw, fpc = ~fpc,
       "<|f|
         summary;
         svymean(~api00, #);
         svyquantile(~api00, #, quantile = c(0.25, 0.5, 0.75), ci = TRUE);
         svytotal(~stype, #);
         svytotal(~enroll, #);
         svyratio(~api.stu,~enroll, #);
         svyratio(~api.stu, ~enroll, design = subset(#, stype == 'H'));
         svymean(make.formula(vars), #, na.rm = TRUE);
         svyby(~ell+meals, ~stype, #, svymean);
         summary(svyglm(api00~ell+meals, #));
         summary(svyglm(I(sch.wide == 'Yes')~ell+meals, #, family = quasibinomial())) >")
```

```
## 
## ntbt(data = ., fti = svydesign, id = ~dnum, weights = ~pw, fpc = ~fpc)
## 
## * summary(#) <||> result *
## 1 - level Cluster Sampling design
## With (15) clusters.
## svydesign(id = ~dnum, weights = ~pw, fpc = ~fpc, data = .)
## Probabilities:
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
## 0.02954 0.02954 0.02954 0.02954 0.02954 0.02954 
## Population size (PSUs): 757 
## Data variables:
##  [1] "cds"      "stype"    "name"     "sname"    "snum"     "dname"   
##  [7] "dnum"     "cname"    "cnum"     "flag"     "pcttest"  "api00"   
## [13] "api99"    "target"   "growth"   "sch.wide" "comp.imp" "both"    
## [19] "awards"   "meals"    "ell"      "yr.rnd"   "mobility" "acs.k3"  
## [25] "acs.46"   "acs.core" "pct.resp" "not.hsg"  "hsg"      "some.col"
## [31] "col.grad" "grad.sch" "avg.ed"   "full"     "emer"     "enroll"  
## [37] "api.stu"  "fpc"      "pw"      
## 
## * svymean(~api00, #) <||> result *
##         mean     SE
## api00 644.17 23.542
## 
## * svyquantile(~api00, #, quantile = c(0.25, 0.5, 0.75), ci = TRUE) <||> result *
## $quantiles
##         0.25 0.5  0.75
## api00 551.75 652 717.5
## 
## $CIs
## , , api00
## 
##            0.25      0.5     0.75
## (lower 493.2835 564.3250 696.0000
## upper) 622.6495 710.8375 761.1355
## 
## 
## 
## * svytotal(~stype, #) <||> result *
##          total      SE
## stypeE 4873.97 1333.32
## stypeH  473.86  158.70
## stypeM  846.17  167.55
## 
## * svytotal(~enroll, #) <||> result *
##          total     SE
## enroll 3404940 932235
## 
## * svyratio(~api.stu,~enroll, #) <||> result *
## Ratio estimator: svyratio.survey.design2(~api.stu, ~enroll, object_value)
## Ratios=
##            enroll
## api.stu 0.8497087
## SEs=
##              enroll
## api.stu 0.008386297
## 
## * svyratio(~api.stu, ~enroll, design = subset(#, stype == 'H')) <||> result *
## Ratio estimator: svyratio.survey.design2(~api.stu, ~enroll, design = subset(object_value, 
##     stype == "H"))
## Ratios=
##            enroll
## api.stu 0.8300683
## SEs=
##             enroll
## api.stu 0.01472607
## 
## * svymean(make.formula(vars), #, na.rm = TRUE) <||> result *
##                   mean      SE
## api00       643.203822 25.4936
## api99       605.490446 25.4987
## sch.wideNo    0.127389  0.0247
## sch.wideYes   0.872611  0.0247
## comp.impNo    0.273885  0.0365
## comp.impYes   0.726115  0.0365
## bothNo        0.273885  0.0365
## bothYes       0.726115  0.0365
## awardsNo      0.292994  0.0397
## awardsYes     0.707006  0.0397
## meals        50.636943  6.6588
## ell          26.891720  2.1567
## yr.rndNo      0.942675  0.0358
## yr.rndYes     0.057325  0.0358
## mobility     17.719745  1.4555
## pct.resp     67.171975  9.6553
## not.hsg      23.082803  3.1976
## hsg          24.847134  1.1167
## some.col     25.210191  1.4709
## col.grad     20.611465  1.7305
## grad.sch      6.229299  1.5361
## avg.ed        2.621529  0.1054
## full         87.127389  2.1624
## emer         10.968153  1.7612
## enroll      573.713376 46.5959
## api.stu     487.318471 41.4182
## 
## * svyby(~ell+meals, ~stype, #, svymean) <||> result *
##   stype      ell    meals   se.ell se.meals
## E     E 29.69444 53.09028 1.411617 7.070399
## H     H 15.00000 37.57143 5.347065 5.912262
## M     M 22.68000 43.08000 2.952862 6.017110
## 
## * summary(svyglm(api00~ell+meals, #)) <||> result *
## 
## Call:
## svyglm(formula = api00 ~ ell + meals, object_value)
## 
## Survey design:
## svydesign(id = ~dnum, weights = ~pw, fpc = ~fpc, data = .)
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept) 817.1823    18.6709  43.768 1.32e-14 ***
## ell          -0.5088     0.3259  -1.561    0.144    
## meals        -3.1456     0.3018 -10.423 2.29e-07 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for gaussian family taken to be 3161.207)
## 
## Number of Fisher Scoring iterations: 2
## 
## 
## * summary(svyglm(I(sch.wide == 'Yes')~ell+meals, #, family = quasibinomial())) <||> result *
## 
## Call:
## svyglm(formula = I(sch.wide == "Yes") ~ ell + meals, object_value, 
##     family = quasibinomial())
## 
## Survey design:
## svydesign(id = ~dnum, weights = ~pw, fpc = ~fpc, data = .)
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)   
## (Intercept)  1.899557   0.509915   3.725  0.00290 **
## ell          0.039925   0.012443   3.209  0.00751 **
## meals       -0.019115   0.008825  -2.166  0.05117 . 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for quasibinomial family taken to be 0.9627734)
## 
## Number of Fisher Scoring iterations: 5
```

<br />
