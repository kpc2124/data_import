Tidy Data
================
Kristi Chau
9/24/2019

## Wide to Long

``` r
pulse_data = 
  haven::read_sas("./data/public_pulse_data.sas7bdat") %>%
  janitor::clean_names()
  pivot_longer(
    pulse_data,
    bdi_score_bl:bdi_score_12m,
    names_to = "visit", ## corresponds to column names
    names_prefix = "bdi_score_",
    values_to = "bdi" ## corresponds to values in the corresponding columns
  ) %>%
  mutate(
    visit = recode(visit, "bl" = "00m")
  )
```

    ## # A tibble: 4,348 x 5
    ##       id   age sex   visit   bdi
    ##    <dbl> <dbl> <chr> <chr> <dbl>
    ##  1 10003  48.0 male  00m       7
    ##  2 10003  48.0 male  01m       1
    ##  3 10003  48.0 male  06m       2
    ##  4 10003  48.0 male  12m       0
    ##  5 10015  72.5 male  00m       6
    ##  6 10015  72.5 male  01m      NA
    ##  7 10015  72.5 male  06m      NA
    ##  8 10015  72.5 male  12m      NA
    ##  9 10022  58.5 male  00m      14
    ## 10 10022  58.5 male  01m       3
    ## # ... with 4,338 more rows

## separate in litters

``` r
litters_data = 
    read_csv("./data/FAS_litters.csv") %>% 
    janitor::clean_names() %>% 
    separate(col = group, into = c("dose","day_of_tx"), 3)
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

## go untidy…

``` r
analysis_result = tibble(
  group = c("treatment", "treatment", "placebo", "placebo"),
  time = c("pre", "post", "pre", "post"),
  mean = c(4, 8, 3.5, 4)
)

pivot_wider(
  analysis_result,
  names_from = time,
  values_from = mean
) %>%  view
```

## bind rows

``` r
fellowship_ring = 
    readxl::read_excel("./data/LotR_Words.xlsx", range = "B3:D6") %>% 
    mutate(movie = "fellowship") ## add in movie "indicator"

two_towers = 
    readxl::read_excel("./data/LotR_Words.xlsx", range = "F3:H6") %>% 
    mutate(movie = "two_towers")

return_king = 
    readxl::read_excel("./data/LotR_Words.xlsx", range = "J3:L6") %>% 
    mutate(movie = "return_king")

bind_rows(fellowship_ring, two_towers, return_king) %>% 
  janitor::clean_names() %>% 
  pivot_longer(
      female:male,
      names_to = "sex",
      values_to = "words"
  ) %>% 
  select(movie, race, sex, words)
```

    ## # A tibble: 18 x 4
    ##    movie       race   sex    words
    ##    <chr>       <chr>  <chr>  <dbl>
    ##  1 fellowship  Elf    female  1229
    ##  2 fellowship  Elf    male     971
    ##  3 fellowship  Hobbit female    14
    ##  4 fellowship  Hobbit male    3644
    ##  5 fellowship  Man    female     0
    ##  6 fellowship  Man    male    1995
    ##  7 two_towers  Elf    female   331
    ##  8 two_towers  Elf    male     513
    ##  9 two_towers  Hobbit female     0
    ## 10 two_towers  Hobbit male    2463
    ## 11 two_towers  Man    female   401
    ## 12 two_towers  Man    male    3589
    ## 13 return_king Elf    female   183
    ## 14 return_king Elf    male     510
    ## 15 return_king Hobbit female     2
    ## 16 return_king Hobbit male    2673
    ## 17 return_king Man    female   268
    ## 18 return_king Man    male    2459

## JOINS\!

``` r
pup_data = 
  read_csv("./data/FAS_pups.csv", col_types = "ciiiii") %>%
  janitor::clean_names() %>%
  mutate(sex = recode(sex, `1` = "male", `2` = "female")) 

litter_data = 
  read_csv("./data/FAS_litters.csv", col_types = "ccddiiii") %>%
  janitor::clean_names() %>%
  select(-pups_survive) %>%
  mutate(
    wt_gain = gd18_weight - gd0_weight,
    group = str_to_lower(group))
```

try to join these datasets

``` r
## joins are difficult to reconcile with piping, just keep it separate

fas_data = 
    left_join(pup_data, litter_data, by = "litter_number") ## using left_join: what I care about is pup specific outcomes, so the pup dataset has all the variables I need. Use by to specify the unique "key" you are joining by.

full_join(pup_data, litter_data, by = "litter_number") %>% 
  filter(is.na(sex))
```

    ## # A tibble: 2 x 13
    ##   litter_number sex   pd_ears pd_eyes pd_pivot pd_walk group gd0_weight
    ##   <chr>         <chr>   <int>   <int>    <int>   <int> <chr>      <dbl>
    ## 1 #112          <NA>       NA      NA       NA      NA low7        23.9
    ## 2 #7/82-3-2     <NA>       NA      NA       NA      NA mod8        26.9
    ## # ... with 5 more variables: gd18_weight <dbl>, gd_of_birth <int>,
    ## #   pups_born_alive <int>, pups_dead_birth <int>, wt_gain <dbl>
