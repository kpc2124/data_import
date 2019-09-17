Data Import
================
Kristi Chau
9/17/2019

## Load in Dataset

``` r
## reads in a dataset (absolute (NO) vs relative (YES) paths)

litters_data = read_csv(file = "./data/FAS_litters.csv")
```

    ## Parsed with column specification:
    ## cols(
    ##   Group = col_character(),
    ##   `Litter Number` = col_character(),
    ##   `GD0 weight` = col_double(),
    ##   `GD18 weight` = col_double(),
    ##   `GD of Birth` = col_double(),
    ##   `Pups born alive` = col_double(),
    ##   `Pups dead @ birth` = col_double(),
    ##   `Pups survive` = col_double()
    ## )

``` r
litters_data = janitor::clean_names(litters_data)

## rewriting dataset
```
