dplyr: verbs for manipulating data-frames
========================================================
autosize: true
author: Tristan Mahr, @tjmahr
date: March 18, 2015
css: assets/custom.css

Madison R Users Group

Repository for this talk: https://github.com/tjmahr/MadR_Pipelines





Plan
========================================================

We [just covered pipelines][magrittr_talk]. Now we'll use pipelines to
work on tabular data.


```r
library("magrittr")
library("dplyr")
library("nycflights13")
```

## Examples

The examples for the main dplyr verbs were lifted from the
[introductory vignette][dplyr_intro].

[magrittr_talk]: http://rpubs.com/tjmahr/pipelines_2015
[dplyr_intro]: http://cran.rstudio.com/web/packages/dplyr/vignettes/introduction.html



There's a cheatsheet!
========================================================

![](assets/data-wrangling-cheatsheet.png)

[http://www.rstudio.com/resources/cheatsheets/][cheatsheets]

[cheatsheets]: http://www.rstudio.com/resources/cheatsheets/



nycflights13
========================================================

Convert data to a `tbl_df` so that it uses dplyr's nice `print` method.


```r
flights <- tbl_df(flights)
flights
```

```
Source: local data frame [336,776 x 16]

   year month day dep_time dep_delay
1  2013     1   1      517         2
2  2013     1   1      533         4
3  2013     1   1      542         2
4  2013     1   1      544        -1
..  ...   ... ...      ...       ...
Variables not shown: arr_time (int),
  arr_delay (dbl), carrier (chr), tailnum
  (chr), flight (int), origin (chr), dest
  (chr), air_time (dbl), distance (dbl),
  hour (dbl), minute (dbl)
```



glimpse
========================================================
incremental: true

Use `glimpse` to few some values in each column.


```r
glimpse(flights)
```

```
Observations: 336776
Variables:
$ year      (int) 2013, 2013, 2013, 2013...
$ month     (int) 1, 1, 1, 1, 1, 1, 1, 1...
$ day       (int) 1, 1, 1, 1, 1, 1, 1, 1...
$ dep_time  (int) 517, 533, 542, 544, 55...
$ dep_delay (dbl) 2, 4, 2, -1, -6, -4, -...
$ arr_time  (int) 830, 850, 923, 1004, 8...
$ arr_delay (dbl) 11, 20, 33, -18, -25, ...
$ carrier   (chr) "UA", "UA", "AA", "B6"...
$ tailnum   (chr) "N14228", "N24211", "N...
$ flight    (int) 1545, 1714, 1141, 725,...
$ origin    (chr) "EWR", "LGA", "JFK", "...
$ dest      (chr) "IAH", "IAH", "MIA", "...
$ air_time  (dbl) 227, 227, 160, 183, 11...
$ distance  (dbl) 1400, 1416, 1089, 1576...
$ hour      (dbl) 5, 5, 5, 5, 5, 5, 5, 5...
$ minute    (dbl) 17, 33, 42, 44, 54, 54...
```



Question
========================================================
type: prompt
incremental: true

> Is dpylr's print method really that useful since it's just
printing four rows of four columns on these slides?

It normally fills width of the console. I cheated to accommodate
the width of these slides, by setting some hidden flags:


```r
options(
  width = 45,
  dplyr.width = 44,
  dplyr.print_min = 4,
  dplyr.print_max = 4)
```



filter: inspect subsets of data
========================================================
incremental: true

How many flights flew to Madison in 2013?


```r
flights %>%
  filter(dest == "MSN")
```

```
Source: local data frame [572 x 16]

   year month day dep_time dep_delay
1  2013     1   1     1353        -4
2  2013     1   2     1422        25
3  2013     1   3     1415        24
4  2013     1   4     1345        -5
..  ...   ... ...      ...       ...
Variables not shown: arr_time (int),
  arr_delay (dbl), carrier (chr), tailnum
  (chr), flight (int), origin (chr), dest
  (chr), air_time (dbl), distance (dbl),
  hour (dbl), minute (dbl)
```



filter: multiple conditions
========================================================
incremental: true

How many flights flew to Madison in first week of January?


```r
# Comma separated conditions are combined with '&'
flights %>%
  filter(dest == "MSN", month == 1, day <= 7)
```

```
Source: local data frame [6 x 16]

   year month day dep_time dep_delay
1  2013     1   1     1353        -4
2  2013     1   2     1422        25
3  2013     1   3     1415        24
4  2013     1   4     1345        -5
..  ...   ... ...      ...       ...
Variables not shown: arr_time (int),
  arr_delay (dbl), carrier (chr), tailnum
  (chr), flight (int), origin (chr), dest
  (chr), air_time (dbl), distance (dbl),
  hour (dbl), minute (dbl)
```



Question
========================================================
type: prompt
incremental: true

> Commas in the filter statement are implicit `&` (and) operators.
Is there anything similar for `|` (or)?

Logical or statements are supported, but there's no shorthand.


```r
flights %>%
  filter(dest == "MSN" | dest == "ORD" | dest == "MDW")
```

For more complicated checks, I would try a set operation.


```r
flights %>%
  filter(is.element(dest, c("MSN", "ORD", "MDW")))
```



arrange: sort columns
========================================================
incremental: true

Sort by which airport they departed from in NYC, then year, month, day.


```r
flights %>%
  arrange(origin, year, month, day)
```

```
Source: local data frame [336,776 x 16]

   year month day dep_time dep_delay
1  2013     1   1      517         2
2  2013     1   1      554        -4
3  2013     1   1      555        -5
4  2013     1   1      558        -2
..  ...   ... ...      ...       ...
Variables not shown: arr_time (int),
  arr_delay (dbl), carrier (chr), tailnum
  (chr), flight (int), origin (chr), dest
  (chr), air_time (dbl), distance (dbl),
  hour (dbl), minute (dbl)
```



desc: reverses sorting of a column
========================================================
incremental: true

Find longest delayed flights to Madison.


```r
flights %>%
  filter(dest == "MSN") %>%
  arrange(desc(dep_delay))
```

```
Source: local data frame [572 x 16]

   year month day dep_time dep_delay
1  2013    12   5     2000       340
2  2013    11  17       45       310
3  2013     3   8     1907       302
4  2013     9  12     1841       291
..  ...   ... ...      ...       ...
Variables not shown: arr_time (int),
  arr_delay (dbl), carrier (chr), tailnum
  (chr), flight (int), origin (chr), dest
  (chr), air_time (dbl), distance (dbl),
  hour (dbl), minute (dbl)
```



select
========================================================
incremental: true

Select the columns you want.


```r
flights %>%
  select(origin, year, month, day)
```

```
Source: local data frame [336,776 x 4]

   origin year month day
1     EWR 2013     1   1
2     LGA 2013     1   1
3     JFK 2013     1   1
4     JFK 2013     1   1
..    ...  ...   ... ...
```



select's helpers
========================================================
incremental: true

`select` has many helper functions. See `?select`.


```r
flights %>%
  select(origin, year:day, starts_with("dep"))
```

```
Source: local data frame [336,776 x 6]

   origin year month day dep_time dep_delay
1     EWR 2013     1   1      517         2
2     LGA 2013     1   1      533         4
3     JFK 2013     1   1      542         2
4     JFK 2013     1   1      544        -1
..    ...  ...   ... ...      ...       ...
```



negative selecting
========================================================
incremental: true

We can drop columns by "negating" the name. Since helpers
give us column names, we can negate them too.


```r
flights %>%
  select(-dest, -starts_with("arr"),
         -ends_with("time"))
```

```
Source: local data frame [336,776 x 11]

   year month day dep_delay carrier tailnum
1  2013     1   1         2      UA  N14228
2  2013     1   1         4      UA  N24211
3  2013     1   1         2      AA  N619AA
4  2013     1   1        -1      B6  N804JB
..  ...   ... ...       ...     ...     ...
Variables not shown: flight (int), origin
  (chr), distance (dbl), hour (dbl), minute
  (dbl)
```



Recap: Verbs for inspecting data
=======================================================

* convert to a `tbl_df` - nice print method
* `glimpse` - some of each column
* `filter` - subsetting
* `arrange` - sorting (`desc` to reverse the sort)
* `select` - picking (and omiting) columns



rename
========================================================
incremental: true

Rename columns with `rename(NewName = OldName)`. To keep the order
correct, read/remember the renaming `=` as "was".


```r
flights %>%
  rename(y = year, m = month, d = day)
```

```
Source: local data frame [336,776 x 16]

      y m d dep_time dep_delay arr_time
1  2013 1 1      517         2      830
2  2013 1 1      533         4      850
3  2013 1 1      542         2      923
4  2013 1 1      544        -1     1004
..  ... . .      ...       ...      ...
Variables not shown: arr_delay (dbl),
  carrier (chr), tailnum (chr), flight
  (int), origin (chr), dest (chr), air_time
  (dbl), distance (dbl), hour (dbl), minute
  (dbl)
```



mutate
=========================================================
incremental: true

How much departure delay did the flight make up for in the air?


```r
flights %>%
  mutate(
    gain = arr_delay - dep_delay,
    speed = (distance / air_time) * 60,
    gain_per_hour = gain / (air_time / 60)) %>%
  select(gain:gain_per_hour)
```

```
Source: local data frame [336,776 x 3]

   gain    speed gain_per_hour
1     9 370.0441      2.378855
2    16 374.2731      4.229075
3    31 408.3750     11.625000
4   -17 516.7213     -5.573770
..  ...      ...           ...
```





group_by
=========================================================
incremental: true

Let's compute the average delay per month of flights to Madison.

Normally--in `aggregate`, `by` or plyr's `d*ply` functions--you
specify the grouping as an argument to the aggregation function.


```r
aggregate(dep_delay ~ month, flights, mean,
          subset = flights$dest == "MSN")
```

```
   month dep_delay
1      1  18.07692
2      2  20.11111
3      3  41.87097
4      4  29.40741
5      5  21.54839
6      6  29.83333
7      7  11.13333
8      8  19.06667
9      9  15.97183
10    10  19.26190
11    11  15.96250
12    12  42.68831
```



group_by
=========================================================
incremental: true

In dplyr, grouping is its own action. It is done as its own step in
the pipeline. Here, we `filter` to the flights to Madison and
`group_by` month.


```r
msn_by_month <- flights %>%
  filter(dest == "MSN") %>%
  group_by(month)
msn_by_month
```

```
Source: local data frame [572 x 16]
Groups: month

   year month day dep_time dep_delay
1  2013     1   1     1353        -4
2  2013     1   2     1422        25
3  2013     1   3     1415        24
4  2013     1   4     1345        -5
..  ...   ... ...      ...       ...
Variables not shown: arr_time (int),
  arr_delay (dbl), carrier (chr), tailnum
  (chr), flight (int), origin (chr), dest
  (chr), air_time (dbl), distance (dbl),
  hour (dbl), minute (dbl)
```



summarise
=========================================================
incremental: true

Now we use `summarise` to compute (several) aggregate values within
each group (month). `summarise` returns one row per group.


```r
msn_by_month %>%
  summarise(
    flights = n(),
    avg_delay = mean(dep_delay, na.rm = TRUE),
    n_planes = n_distinct(tailnum))
```

```
Source: local data frame [12 x 4]

   month flights avg_delay n_planes
1      1      27  18.07692       23
2      2      47  20.11111       36
3      3      31  41.87097       29
4      4      27  29.40741       25
..   ...     ...       ...      ...
```



tally
=========================================================
incremental: true

`tally` is a shortcut for counting number of items per group.

Number of flights from NYC by destination by month:


```r
flights %>%
  group_by(dest, month) %>%
  tally
```

```
Source: local data frame [1,113 x 3]
Groups: dest

   dest month  n
1   ABQ     4  9
2   ABQ     5 31
3   ABQ     6 30
4   ABQ     7 31
..  ...   ... ..
```



ungroup
=========================================================
incremental: true

Remove the grouping structure with `ungroup`.


```r
msn_by_month %>% ungroup
```

```
Source: local data frame [572 x 16]

   year month day dep_time dep_delay
1  2013     1   1     1353        -4
2  2013     1   2     1422        25
3  2013     1   3     1415        24
4  2013     1   4     1345        -5
..  ...   ... ...      ...       ...
Variables not shown: arr_time (int),
  arr_delay (dbl), carrier (chr), tailnum
  (chr), flight (int), origin (chr), dest
  (chr), air_time (dbl), distance (dbl),
  hour (dbl), minute (dbl)
```



Summarizing undoes grouping.
=========================================================
incremental: true

Each `summarise` statement peels off one layer of grouping (from the
right of the list of groups).


```r
# day gets peeled off
per_day <- flights %>%
  group_by(dest, year, month, day) %>%
  summarise(flights = n())
per_day
```

```
Source: local data frame [31,229 x 5]
Groups: dest, year, month

   dest year month day flights
1   ABQ 2013     4  22       1
2   ABQ 2013     4  23       1
3   ABQ 2013     4  24       1
4   ABQ 2013     4  25       1
..  ...  ...   ... ...     ...
```



=========================================================
title: false
incremental: true

Peel off `month` grouping.


```r
per_month <- per_day %>%
  summarise(flights = sum(flights))
per_month
```

```
Source: local data frame [1,113 x 4]
Groups: dest, year

   dest year month flights
1   ABQ 2013     4       9
2   ABQ 2013     5      31
3   ABQ 2013     6      30
4   ABQ 2013     7      31
..  ...  ...   ...     ...
```



=========================================================
title: false
incremental: true

Peel off `year` grouping.


```r
per_year <- per_month %>%
  summarise(flights = sum(flights))
per_year
```

```
Source: local data frame [105 x 3]
Groups: dest

   dest year flights
1   ABQ 2013     254
2   ACK 2013     265
3   ALB 2013     439
4   ANC 2013       8
..  ...  ...     ...
```



Question
========================================================
type: prompt

> Is there a way to keep a group after a `summarise` statement?

The grouping variable goes away because `summarise` returns one row
for each value of the grouping variable. If you `group_by` month and
`summarise`, you have a row for each month. You can add the month
grouping back to the data-frame, but this is almost always not
necessary because you have only row per group at that point.

Relatedly, you can group by row with `rowwise`.



mutate also respects grouping
=========================================================
incremental: true

Rank flight delays within each destination and month. (dplyr provides
a bunch of ranking functions. See the [cheatsheet][cheatsheets].)


```r
flights %>%
  group_by(dest, month) %>%
  mutate(timely = row_number(dep_delay),
         late = row_number(desc(dep_delay))) %>%
  select(dep_delay, timely:late)
```

```
Source: local data frame [336,776 x 5]
Groups: dest, month

   dest month dep_delay timely late
1   IAH     1         2    347  195
2   IAH     1         4    380  162
3   MIA     1         2    662  294
4   BQN     1        -1     46   38
..  ...   ...       ...    ...  ...
```

[cheatsheets]: http://www.rstudio.com/resources/cheatsheets/



mutate also respects grouping
=========================================================
incremental: true

This is also good for mean-centering variables within groups.


```r
mean_center <- function(xs) {
  xs - mean(xs, na.rm = TRUE)
}
flights %>%
  group_by(dest, month) %>%
  mutate(c_delay = mean_center(dep_delay)) %>%
  select(dep_delay, c_delay)
```

```
Source: local data frame [336,776 x 4]
Groups: dest, month

   dest month dep_delay   c_delay
1   IAH     1         2 -5.294643
2   IAH     1         4 -3.294643
3   MIA     1         2 -5.360656
4   BQN     1        -1 -6.870968
..  ...   ...       ...       ...
```



That covers 80% of dplyr
========================================================

- select
- filter
- arrange
- glimpse
- rename
- mutate
- group_by, ungroup
- summarise



Other 20%
========================================================

- assembly: bind_rows, bind_cols
- column-wise operations: mutate_each, summarise_each
- join tables together: left_join, right_join, inner_join, full_join
- filtering joins: semi_join, anti_join
- do: arbitrary code on each chunk
- different types of tabular data (databases, data.tables)



Questions?
========================================================
type: prompt



Last Section
========================================================

For fun, I demonstrated how I use [pipelines when making tables](http://rpubs.com/tjmahr/prettytables_2015).
