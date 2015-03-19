Bonus Slides: Making pretty regression tables with pipes
========================================================
autosize: true
author: Tristan Mahr, @tjmahr
date: March 18, 2015
css: assets/custom.css

Madison R Users Group

Repository for this talk: https://github.com/tjmahr/MadR_Pipelines





Plan
========================================================

Earlier, I [covered pipelines][magrittr_talk].

Then, I [used pipelines with dplyr][dplyr_talk].

Now, I will demonstrate off how I use pipelines to make
pretty-printed regression tables.


```r
library("magrittr")
library("dplyr")
library("broom")
library("stringr")
library("knitr")
```


[magrittr_talk]: http://rpubs.com/tjmahr/pipelines_2015
[dplyr_talk]: http://rpubs.com/tjmahr/dplyr_2015


linear regression
========================================================
title: false

I want to print a regression summary table.


```r
model <- lm(Sepal.Length ~ Sepal.Width, iris)
summary(model)
```

```

Call:
lm(formula = Sepal.Length ~ Sepal.Width, data = iris)

Residuals:
    Min      1Q  Median      3Q     Max 
-1.5561 -0.6333 -0.1120  0.5579  2.2226 

Coefficients:
            Estimate Std. Error t value
(Intercept)   6.5262     0.4789   13.63
Sepal.Width  -0.2234     0.1551   -1.44
            Pr(>|t|)    
(Intercept)   <2e-16 ***
Sepal.Width    0.152    
---
Signif. codes:  
  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1  ' ' 1

Residual standard error: 0.8251 on 148 degrees of freedom
Multiple R-squared:  0.01382,	Adjusted R-squared:  0.007159 
F-statistic: 2.074 on 1 and 148 DF,  p-value: 0.1519
```



broom::tidy
========================================================

`broom`'s `tidy` function extracts summary values into a data-frame.


```r
m_table <- tidy(model)
m_table %>% print(digits = 3)
```

```
         term estimate std.error statistic
1 (Intercept)    6.526     0.479     13.63
2 Sepal.Width   -0.223     0.155     -1.44
   p.value
1 6.47e-28
2 1.52e-01
```



knitr::kable
========================================================

`knitr`'s `kable` function prints a data-frame as a Markdown table,
which rmarkdown/pandoc can convert to HTML or Word.


```r
kable(m_table, digits = 3,
      col.names = c("Param", "B", "SE", "t", "p"))
```

Raw Markdown output

```


|Param       |      B|    SE|      t|     p|
|:-----------|------:|-----:|------:|-----:|
|(Intercept) |  6.526| 0.479| 13.628| 0.000|
|Sepal.Width | -0.223| 0.155| -1.440| 0.152|


```



========================================================
title: false

Which renders as:


```r
kable(m_table, digits = 3,
      col.names = c("Param", "B", "SE", "t", "p"))
```



|Param       |      B|    SE|      t|     p|
|:-----------|------:|-----:|------:|-----:|
|(Intercept) |  6.526| 0.479| 13.628| 0.000|
|Sepal.Width | -0.223| 0.155| -1.440| 0.152|



Almost there
========================================================
incremental: true

Two more things that I want.

1. APA style for printing numbers
    * Two digits of precision for estimates, errors and statistics.
    * Three digits for p-values.
    * Bounded values (correlations, p-values) don't need leading zero.
    * Tiny p-values should not be 0.000. Use `< .001` instead.
2. Flexibility for many related models. (I don't know the
   final in-text model beforehand.)



Approach
========================================================
incremental: true

- Build a set of generic, reusable functions for formatting numbers.
- Make a custom pipeline to format the `term` column.
- Assemble a pipeline to convert a model into a pretty table.



Digit Formatters
========================================================

The `formatC` function is big and confusing, but after I
figured out something that works well enough, I extracted a
function. Now I don't have to worry about solving that
problem again.


```r
# Print with n digits of precision
fixed_digits <- function(xs, n = 2) {
  formatC(xs, digits = n, format = "f")
}

# Want .100 to print with two digits
c(1000.012, 0.0001, 0.100) %>% fixed_digits(2)
```

```
[1] "1000.01" "0.00"    "0.10"   
```



Remove leading zeros
========================================================

Here I do a sanity check to see if this operation is necessary.


```r
# Don't print leading zero on bounded numbers.
remove_leading_zero <- function(xs) {
  # Problem if any value is greater than 1.0
  digit_matters <- xs %>% as.numeric %>%
    abs %>% is_greater_than(1)
  if (any(digit_matters)) {
    warning("Non-zero leading digit")
  }
  str_replace(xs, "^(-?)0", "\\1")
}
c(1.00, -0.12,  0.87, 0.82) %>% remove_leading_zero
```

```
[1] "1"    "-.12" ".87"  ".82" 
```



p-value formatter
========================================================

I format p-values with the previous two functions and overwrite the
really small ones with the `< .001` template.


```r
# Print three digits of a p-value, but use
# the "< .001" notation on tiny values.
format_pval <- function(ps, html = FALSE) {
  tiny <- ifelse(html, "&lt;&nbsp;.001", "< .001")
  ps_chr <- ps %>% fixed_digits(3) %>%
    remove_leading_zero
  ps_chr[ps < 0.001] <- tiny
  ps_chr
}
m_table$p.value %>% format_pval
```

```
[1] "< .001" ".152"  
```



Renaming variables
=======================================================

A few packages already solve the pretty-table problem--stargazer,
texreg, xtable--and they do amazing things. But their functions
usually  rely on an argument where you can specify custom labels
for the regression predictors. This is fine for one-off tables.

When I have a bunch of related models, I prefer to use a
string-manipulation pipeline to convert R names into long-hand names.



Renaming variables
=======================================================


```r
fix_names <- . %>%
  str_replace(".Intercept.", "Intercept") %>%
  str_replace("Species", "") %>%
  # Capitalize species names
  str_replace("setosa", "Setosa") %>%
  str_replace("versicolor", "Versicolor") %>%
  str_replace("virginica", "Virginica") %>%
  # Clean up special characters
  str_replace_all(".Width", " Width") %>%
  str_replace_all(".Length", " Length") %>%
  str_replace_all(":", " x ")
```



Formatting pipeline
=======================================================

1. Format the numbers
2. Rename the terms
3. Rename the column headings


```r
two_digits <- . %>% fixed_digits(2)
table_names <- c("Parameter", "Estimate", "SE",
                 "_t_", "_p_")

format_model_table <- . %>%
  mutate_each(funs(two_digits),
              -term, -p.value) %>%
  mutate(term = fix_names(term),
         p.value = format_pval(p.value)) %>%
  set_colnames(table_names)
```



Overall pipeline
=======================================================

1. Fit a model
2. Extract values for summary table
3. Format table
4. Print as markdown


```r
lm(..., iris) %>%
  tidy %>%
  format_model_table %>%
  kable
```


========================================================
title: false


```r
lm(Sepal.Length ~ Sepal.Width, iris) %>%
  tidy %>%
  format_model_table %>%
  kable(align = "r")
```



|   Parameter| Estimate|   SE|   _t_|    _p_|
|-----------:|--------:|----:|-----:|------:|
|   Intercept|     6.53| 0.48| 13.63| < .001|
| Sepal Width|    -0.22| 0.16| -1.44|   .152|



========================================================
title: false


```r
lm(Sepal.Length ~ Species * Sepal.Width, iris) %>%
  tidy %>%
  format_model_table %>%
  kable(align = "r")
```



|                Parameter| Estimate|   SE|  _t_|    _p_|
|------------------------:|--------:|----:|----:|------:|
|                Intercept|     2.64| 0.57| 4.62| < .001|
|               Versicolor|     0.90| 0.80| 1.13|   .261|
|                Virginica|     1.27| 0.82| 1.55|   .123|
|              Sepal Width|     0.69| 0.17| 4.17| < .001|
| Versicolor x Sepal Width|     0.17| 0.26| 0.67|   .503|
|  Virginica x Sepal Width|     0.21| 0.26| 0.83|   .411|

========================================================
title: false



```r
lm(Sepal.Length ~ Petal.Length * Species, iris) %>%
  tidy %>%
  format_model_table %>%
  kable(align = "r")
```



|                 Parameter| Estimate|   SE|   _t_|    _p_|
|-------------------------:|--------:|----:|-----:|------:|
|                 Intercept|     4.21| 0.41| 10.34| < .001|
|              Petal Length|     0.54| 0.28|  1.96|   .052|
|                Versicolor|    -1.81| 0.60| -3.02|   .003|
|                 Virginica|    -3.15| 0.63| -4.97| < .001|
| Petal Length x Versicolor|     0.29| 0.30|  0.97|   .334|
|  Petal Length x Virginica|     0.45| 0.29|  1.56|   .120|



========================================================
title: false


```r
lm(Sepal.Length ~ Species * Sepal.Width + Petal.Length, iris) %>%
  tidy %>%
  format_model_table %>%
  kable(align = "r")
```



|                Parameter| Estimate|   SE|   _t_|    _p_|
|------------------------:|--------:|----:|-----:|------:|
|                Intercept|     1.67| 0.41|  4.11| < .001|
|               Versicolor|     0.28| 0.56|  0.51|   .614|
|                Virginica|    -0.65| 0.59| -1.10|   .274|
|              Sepal Width|     0.62| 0.12|  5.40| < .001|
|             Petal Length|     0.82| 0.07| 12.42| < .001|
| Versicolor x Sepal Width|    -0.45| 0.19| -2.39|   .018|
|  Virginica x Sepal Width|    -0.29| 0.18| -1.57|   .119|


========================================================
title: false


```r
lm(Sepal.Length ~ Species * Sepal.Width * Petal.Length, iris) %>%
  tidy %>%
  format_model_table %>%
  kable(align = "r")
```



|                               Parameter| Estimate|   SE|   _t_|  _p_|
|---------------------------------------:|--------:|----:|-----:|----:|
|                               Intercept|    -1.37| 3.91| -0.35| .727|
|                              Versicolor|     3.17| 5.50|  0.58| .565|
|                               Virginica|    -0.99| 5.27| -0.19| .851|
|                             Sepal Width|     1.72| 1.12|  1.54| .126|
|                            Petal Length|     2.88| 2.75|  1.05| .297|
|                Versicolor x Sepal Width|    -1.36| 1.86| -0.73| .466|
|                 Virginica x Sepal Width|    -0.43| 1.66| -0.26| .797|
|               Versicolor x Petal Length|    -2.07| 2.90| -0.71| .476|
|                Virginica x Petal Length|    -1.42| 2.82| -0.51| .614|
|              Sepal Width x Petal Length|    -0.74| 0.79| -0.95| .345|
| Versicolor x Sepal Width x Petal Length|     0.72| 0.86|  0.84| .405|
|  Virginica x Sepal Width x Petal Length|     0.56| 0.81|  0.69| .488|




