# ggplot2-solutions

These are my solutions (and thought processes behind) to the exercises provided in Hadley Wickham's [fantastic crash](https://ggplot2-book.org/) course on ggplot2.

---

## 2.2 - Fuel economy data
### Exercises 2.2.1

First, we want to load the [ggplot2](https://ggplot2.tidyverse.org/) & [dplyr](https://dplyr.tidyverse.org/) packages and fetch the mpg [tibble](https://tibble.tidyverse.org/).
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

* `class` - allows us to check the class of the `mpg` object
* `str` - returns the structure of the `mpg` object
* `summary` - gives us a summary of the `mpg` object including some basic statistics (mean, median, mode, quartiles, etc.)
* `View` - opens the entire `mpg` object allowing it to be viewed as a spreadsheet
* `dim` - returns the dimensions (nrow, ncol) of the `mpg` object. `str` and `summary` will also return this information.



>How can you find out what other datasets are included with ggplot2?

We can check out all the included datasets in the `ggplot2` package by heading over to the data folder within ggplot2's [github](https://github.com/tidyverse/ggplot2/tree/main/data)



>Apart from the US, most countries use fuel consumption (fuel consumed over fixed distance) rather than fuel economy (distance travelled with fixed amount of fuel). How could you convert cty and hwy into the European standard of l/100km?

We want to convert the `$cty` and `$hwy` columns from the imperial 'miles per gallon' to the metric 'liters per 100 kilometers'.
There's a number of ways one could do this:

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
