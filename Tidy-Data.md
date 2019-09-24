Tidy Data
================
Dionna Attinson
9/24/2019

## Wide to Long

``` r
pulse_data = 
  haven::read_sas("./data/public_pulse_data.sas7bdat") %>% 
  janitor::clean_names()
  pivot_longer(
    pulse_data,
    bdi_score_bl:bdi_score_12m,
    names_to = "visit",
    names_prefix = "bdi_score_",
    values_to = "bdi"
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
    ## # … with 4,338 more rows

## Seperate in Litters

``` r
litters_data = 
    read_csv("./data/FAS_litters.csv") %>% 
    janitor::clean_names() %>% 
    separate(col = group, into = c("dose", "day_of_tx"),3)
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

## Go Untidy

``` r
analysis_result = tibble(
  group = c("treatment", "treatment", "placebo", "placebo"),
  time = c("pre","post","pre","post"),
  mean = c(4,8,3.5,4)
)

pivot_wider(
  analysis_result,
  names_from = time,
  values_from = mean)
```

    ## # A tibble: 2 x 3
    ##   group       pre  post
    ##   <chr>     <dbl> <dbl>
    ## 1 treatment   4       8
    ## 2 placebo     3.5     4

## Bind Rows

``` r
fellowship_data = 
  readxl::read_excel("./data/LotR_Words.xlsx", range = "B3:D6") %>% 
  mutate(movie = "fellowship")

two_towers = 
  readxl::read_excel("./data/LotR_Words.xlsx", range = "F3:H6") %>%
  mutate(movie = "two_towers")

return_king = 
  readxl::read_excel("./data/LotR_Words.xlsx", range = "J3:L6") %>%
  mutate(movie = "return_king")

bind_rows(fellowship_data, two_towers, return_king)
```

    ## # A tibble: 9 x 4
    ##   Race   Female  Male movie      
    ##   <chr>   <dbl> <dbl> <chr>      
    ## 1 Elf      1229   971 fellowship 
    ## 2 Hobbit     14  3644 fellowship 
    ## 3 Man         0  1995 fellowship 
    ## 4 Elf       331   513 two_towers 
    ## 5 Hobbit      0  2463 two_towers 
    ## 6 Man       401  3589 two_towers 
    ## 7 Elf       183   510 return_king
    ## 8 Hobbit      2  2673 return_king
    ## 9 Man       268  2459 return_king

``` r
lotr_data = 
  bind_rows(fellowship_data, two_towers, return_king) %>% 
  janitor::clean_names() %>% 
  pivot_longer(
    female:male,
    names_to = "sex",
    values_to = "words"
  ) %>% 
  mutate(race = str_to_lower(race)) %>% 
  select(movie, everything()) 
```

## Something else

``` r
pups_data = 
  read_csv("./data/FAS_pups.csv", col_types = "ciiiii") %>%
  janitor::clean_names() %>%
  mutate(sex = recode(sex, `1` = "male", `2` = "female")) 

litters_data = 
  read_csv("./data/FAS_litters.csv", col_types = "ccddiiii") %>%
  janitor::clean_names() %>%
  select(-pups_survive) %>%
  mutate(
    wt_gain = gd18_weight - gd0_weight,
    group = str_to_lower(group))
```

## Gonna join these datasets

``` r
fas_data = 
  left_join(pups_data, litters_data, by = "litter_number")
```
