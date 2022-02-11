# ggplot2-solutions

These are my solutions, and the thought processes behind them, to the exercises provided 
in Hadley Wickham's [fantastic crash](https://ggplot2-book.org/) course on ggplot2.

---

## 2.2 - Fuel economy data
### Exercises 2.2.1

First, we want to load the [ggplot2](https://ggplot2.tidyverse.org/) & 
[dplyr](https://dplyr.tidyverse.org/) packages and fetch the mpg [tibble](https://tibble.tidyverse.org/).
I've opted to save this as the variable `mpg` so I can keep track of it in RStudio's environment panel.

```R
ggplot2_req <- require(ggplot2)
dplyr_req <- require(dplyr)
if (ggplot2_req == FALSE) {install.packages('ggplot2')}
if (dplyr_req == FALSE) {install.packages('dplyr')}
library(ggplot2)
library(dplyr)
mpg <- mpg
print(mpg)
```
```R
# A tibble: 234 × 11
   manufacturer model      displ  year   cyl trans      drv     cty   hwy fl    class  
   <chr>        <chr>      <dbl> <int> <int> <chr>      <chr> <int> <int> <chr> <chr>  
 1 audi         a4           1.8  1999     4 auto(l5)   f        18    29 p     compact
 2 audi         a4           1.8  1999     4 manual(m5) f        21    29 p     compact
 3 audi         a4           2    2008     4 manual(m6) f        20    31 p     compact
 4 audi         a4           2    2008     4 auto(av)   f        21    30 p     compact
 5 audi         a4           2.8  1999     6 auto(l5)   f        16    26 p     compact
 6 audi         a4           2.8  1999     6 manual(m5) f        18    26 p     compact
 7 audi         a4           3.1  2008     6 auto(av)   f        18    27 p     compact
 8 audi         a4 quattro   1.8  1999     4 manual(m5) 4        18    26 p     compact
 9 audi         a4 quattro   1.8  1999     4 auto(l5)   4        16    25 p     compact
10 audi         a4 quattro   2    2008     4 manual(m6) 4        20    28 p     compact
```



>List five functions that you could use to get more information about the mpg dataset.

* `class()` - allows us to check the class of the `mpg` object
* `str()` - returns the structure of the `mpg` object
* `summary()` - gives us a summary of the `mpg` object including some basic statistics (mean, median, mode, quartiles, etc.)
* `View()` - opens the entire `mpg` object allowing it to be viewed as a spreadsheet
* `dim()` - returns the dimensions (nrow, ncol) of the `mpg` object. `str` and `summary` will also return this information.



>How can you find out what other datasets are included with ggplot2?

We can check out all the included datasets in the `ggplot2` package by 
heading over to the data folder within 
ggplot2's [github](https://github.com/tidyverse/ggplot2/tree/main/data)



>Apart from the US, most countries use fuel consumption (fuel consumed over fixed distance) 
rather than fuel economy (distance travelled with fixed amount of fuel). 
How could you convert cty and hwy into the European standard of l/100km?

We want to convert the `$cty` and `$hwy` columns from the imperial 
'miles per gallon' to the metric 'liters per 100 kilometers'.
There's a few ways one could do this:

**Via column maths**
```R
MilesToKM <- 1.60934
GalToLitre <- 3.78541
## Create a new object named mpg_metric
mpg_metric <- mpg
## Convert
mpg_metric$cty <- ((100 / mpg$cty) / MilesToKM) * GalToLitre
mpg_metric$hwy <- ((100 / mpg$hwy) / MilesToKM) * GalToLitre
```

We can also define a function, `mpg.to.kml()` to do these calculations 
(and make future calculations a little bit easier!):
```R
mpg.to.kml <- function(mpg, n.kilometers = 100) {
    MilesToKM <- 1.60934
    GalToLitre <- 3.78541
    ((100 / mpg) / MilesToKM) * GalToLitre
}
## Convert
mpg_metric$cty <- mpg.to.kml(mpg$cty)
mpg_metric$hwy <- mpg.to.kml(mpg$hwy)
```



**Via dplyr's `mutate` function**
```R
mpg_metric <- mpg %>%
    mutate(cty = mpg.to.kml(cty),
           hwy = mpg.to.kml(hwy),
           .keep = c('unused'))
```

The combined fuel economy is also pretty important to know. So, lets use `mutate()` and 
`select()` to add a new column, `combined`, to our dataset that is the mean of the 
respective rows in `hwy` and `cty`. 

```R
mpg_metric <- mpg %>%
    mutate(cty = mpg.to.kml(cty),
           hwy = mpg.to.kml(hwy),
           .keep = c('unused')) %>%
    mutate(combined = rowMeans(select(mpg_metric, hwy, cty)))
```



>Which manufacturer has the most models in this dataset? Which model has the most variations? 
Does your answer change if you remove the redundant specification of drive train (e.g. “pathfinder 4wd”, “a4 quattro”) from the model name?

An easy way to generate a count summary of a column is the `table()` function. 
However, this won't give us sorted data (i.e. descending/ascending order) which could be troublesome if you have many variables.
```R
table(mpg$manufacturer)

#      audi  chevrolet      dodge       ford      honda    hyundai       jeep land rover    lincoln    mercury     nissan    pontiac 
#        18         19         37         25          9         14          8          4          3          4         13          5 
#    subaru     toyota volkswagen 
#        14         34         27 
```

Instead, the `group_by()` and `tally()` functions could be used:
```R
car_manufacturers <- group_by(mpg, manufacturer) %>% tally(sort = TRUE)

#   manufacturer     n
#   <chr>        <int>
#  1 dodge           37
#  2 toyota          34
#  3 volkswagen      27
#  4 ford            25
#  5 chevrolet       19
#  6 audi            18
#  7 hyundai         14
#  8 subaru          14
#  9 nissan          13
# 10 honda            9
# 11 jeep             8
# 12 pontiac          5
# 13 land rover       4
# 14 mercury          4
# 15 lincoln          3
```

We can also find the manufacturer with the most *unique* models:
```R
mpg_models <- mpg %>%
    ## group mpg by manufacturer; this doesn't change how the data looks but how 
    ## it acts when other dplyr verbs are applied to it
    group_by(manufacturer) %>%
    ## generates a new column, 'unique', which includes the respective manufacturer's number of unique models
    transmute('unique' = length(unique(model))) %>%
    ## restricts data to unique values only (i.e. leaves only 1 of each manufacturer)
    unique() %>%
    ## grouping by manufacturer is now superfluous as only one entry of each manufacturer is in the data frame
    ungroup() %>%
    ## arranges the data by the unique column in descending order
    arrange(desc(unique))

#    manufacturer unique
#    <chr>         <int>
#  1 toyota            6
#  2 chevrolet         4
#  3 dodge             4
#  4 ford              4
#  5 volkswagen        4
#  6 audi              3
#  7 nissan            3
#  8 hyundai           2
#  9 subaru            2
# 10 honda             1
# 11 jeep              1
# 12 land rover        1
# 13 lincoln           1
# 14 mercury           1
# 15 pontiac           1
```


To remove the drive train references in the model name we can employ the `stringr` package. 
First, we want to detect all references to drive train in the model name and remove them. 
To do this we'll invoke the `str_remove_all()` function and we'll then remove any leftover
white spaces using `str_squish()` and convert it to a table before finally returning a tibble
that tells us how many releases of each car model released, including those with different drive trains.

```R
stringr_req <- require(stringr)
if (stringr_req == FALSE) {install.packages('stringr')}
## 'quattro', '4wd', '2wd' and 'awd' all reference drivetrains and are thus redundant
remove <- c('quattro' = '', '4wd' = '', '2wd' = '', 'awd' = '')
remove_redundant <- mpg$model %>%
    ## str_remove_all must be used if we are to remove ALL occurrences
    str_remove_all(remove) %>%
    str_squish() %>%
    table() %>%
    ## Note that as.data.frame() will also work here if you wish to avoid tibbles
    as_tibble() %>%
    ## Renames column 1 and column 2; rename_at doesn't support concatenated inputs
    rename_at(1, ~'Model') %>% rename_at(2, ~'n') %>%
    ## Arranges the tibble by descending order of column 'n'
    arrange(desc(n)) %>%
    print()
    
#   A tibble: 37 x 2
#   Model                n
#   <chr>            <int>
#  1 a4                 15
#  2 caravan            11
#  3 ram 1500 pickup    10
#  4 civic               9
#  5 dakota pickup       9
#  6 jetta               9
#  7 mustang             9
#  8 grand cherokee      8
#  9 impreza             8
# 10 camry               7
#  ... with 27 more rows
```

If we instead want to see which *manufacturer* released the most models we just
need to change the `mpg$model` at the beginning of the pipe function to `mpg$manufacturer`.
```R
stringr_req <- require(stringr)
if (stringr_req == FALSE) {install.packages('stringr')}
## 'quattro', '4wd', '2wd' and 'awd' all reference drivetrains and are thus redundant
remove <- c('quattro' = '', '4wd' = '', '2wd' = '', 'awd' = '')
remove_redundant <- mpg$manufacturer %>%
    ## str_remove_all must be used if we are to remove ALL occurrences
    str_remove_all(remove) %>%
    str_squish() %>%
    table() %>%
    ## Note that as.data.frame() will also work here if you wish to avoid tibbles
    as_tibble() %>%
    ## Renames column 1 and column 2; rename_at doesn't support concatenated inputs
    rename_at(1, ~'Model') %>% rename_at(2, ~'n') %>%
    ## Arranges the tibble by descending order of column 'n'
    arrange(desc(n)) %>%
    print()

#  A tibble: 15 x 2
#   Model          n
#   <chr>      <int>
#  1 dodge         37
#  2 toyota        34
#  3 volkswagen    27
#  4 ford          25
#  5 chevrolet     19
#  6 audi          18
#  7 hyundai       14
#  8 subaru        14
#  9 nissan        13
# 10 honda          9
# 11 jeep           8
# 12 pontiac        5
# 13 land rover     4
# 14 mercury        4
# 15 lincoln        3
```