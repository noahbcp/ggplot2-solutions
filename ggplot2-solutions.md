# ggplot2-solutions

These are my solutions (and thought processes behind) to the exercises provided in Hadley Wickham's [fantastic crash](https://ggplot2-book.org/) course on ggplot2.


### 2.2 - Fuel economy data

First, we load the [ggplot2](https://ggplot2.tidyverse.org/) & [dplyr](https://dplyr.tidyverse.org/) packages and fetch the mpg [tibble](https://tibble.tidyverse.org/).
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
