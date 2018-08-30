sentinel
================

S3 class that allows different flavors of missing in numeric vectors.

One can divide measures into two groups: qualitative and quantitative. However, record formats often mix the two. Some of the values are simply interpreted as is: a `2` is a 2. Some of the values are codes which represent qualities instead of numbers: an `8` means the measure's not applicable. These are sometimes called "sentinel values." And, of course, some values are just plain missing.

When handling these data in R, a common idiom is to split the column in twain: a numeric vector for the quantitative and a factor for the qualitative. This is the simplest solution and will often work fine. But it does something risky: it separates linked data. The user must remember to keep them together, and usually does this with clever variable or column names.

Clever is bad. Code with `my_data[, paste0(vars, c("_num", "_flag"))]` is hard to read. Code with `get` is hard to follow.

The `sentinel` package offers the `sentineled` class to bundle numeric and categorical missing values into a single object.

``` r
library(sentineled)

x <- sentineled(
  c(10, 20, 98, 99, NA),
  sentinels = c(98, 99),
  labels    = c("refused", "not recorded")
)
x
```

    ## [1] 10             20             <refused>      <not recorded>
    ## [5] NA            
    ## sentinel values: "" "refused" "not recorded"

The numbers are numbers, the categories are categorical, and the unknowns are just unknown.

Still a vector
--------------

A `sentineled` object is a vector. When subsetting, a it will remain a `sentineled` object with the same possible sentinel values.

``` r
x[1]
```

    ## [1] 10

``` r
x[1:2]
```

    ## [1] 10 20

``` r
x[[3]]
```

    ## [1] NA

``` r
x[x < 15]
```

    ## [1] 10 NA NA NA

A `sentineled` vector can be used in arithmetic, with all non-missing values acting like normal numeric values. If possible, a `sentineled` object with the appropriate sentinel values will be the result.

``` r
mean(x, na.rm = TRUE)
```

    ## [1] 15

``` r
x / 100
```

    ## [1] 0.1            0.2            <refused>      <not recorded>
    ## [5] NA            
    ## sentinel values: "" "refused" "not recorded"

Using the missing values
------------------------

The sentinel codes are treated as missing, but the different categories of missing are stored as a factor vector in the `"sentinels"` attribute of the object. Use the `sentinels` function to access them.

``` r
sentinels(x)
```

    ## [1]                           refused      not recorded <NA>        
    ## Levels:  refused not recorded

``` r
x[sentinels(x) != "refused"]
```

    ## [1] 10 20 NA NA

Notice that, for the non-missing values in `x`, their respective sentinel codes are blanks (`""`).

``` r
as.character(sentinels(x))
```

    ## [1] ""             ""             "refused"      "not recorded"
    ## [5] NA

It's recommended to use explanatory sentinel levels for all expected types of missing. That way, if a value is shown as just plain `NA`, it's a sign something went wrong in the analysis.
