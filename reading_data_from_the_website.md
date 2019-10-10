reading\_data\_from\_the\_website
================
Guangling Xu
10/10/2019

``` r
library(tidyverse)
```

    ## -- Attaching packages ----------------------------------- tidyverse 1.2.1 --

    ## v ggplot2 3.2.1     v purrr   0.3.2
    ## v tibble  2.1.3     v dplyr   0.8.3
    ## v tidyr   1.0.0     v stringr 1.4.0
    ## v readr   1.3.1     v forcats 0.4.0

    ## -- Conflicts -------------------------------------- tidyverse_conflicts() --
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

``` r
library(rvest)
```

    ## Loading required package: xml2

    ## 
    ## Attaching package: 'rvest'

    ## The following object is masked from 'package:purrr':
    ## 
    ##     pluck

    ## The following object is masked from 'package:readr':
    ## 
    ##     guess_encoding

``` r
library(httr)
```

## reading data

``` r
url = "http://samhda.s3-us-gov-west-1.amazonaws.com/s3fs-public/field-uploads/2k15StateFiles/NSDUHsaeShortTermCHG2015.htm"
drug_use_xml = read_html(url)

drug_use_xml %>%
  html_nodes(css = "table")
```

    ## {xml_nodeset (15)}
    ##  [1] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ##  [2] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ##  [3] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ##  [4] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ##  [5] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ##  [6] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ##  [7] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ##  [8] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ##  [9] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ## [10] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ## [11] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ## [12] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ## [13] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ## [14] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ## [15] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...

``` r
##More easily extract pieces out of HTML documents using XPath and CSS selectors. 

table_marj = 
  (drug_use_xml %>% html_nodes(css = "table")) %>% 
  .[[1]] %>%
  html_table() %>%
  slice(-1) %>% 
  as_tibble()

table_marj
```

    ## # A tibble: 56 x 16
    ##    State `12+(2013-2014)` `12+(2014-2015)` `12+(P Value)` `12-17(2013-201~
    ##    <chr> <chr>            <chr>            <chr>          <chr>           
    ##  1 Tota~ 12.90a           13.36            0.002          13.28b          
    ##  2 Nort~ 13.88a           14.66            0.005          13.98           
    ##  3 Midw~ 12.40b           12.76            0.082          12.45           
    ##  4 South 11.24a           11.64            0.029          12.02           
    ##  5 West  15.27            15.62            0.262          15.53a          
    ##  6 Alab~ 9.98             9.60             0.426          9.90            
    ##  7 Alas~ 19.60a           21.92            0.010          17.30           
    ##  8 Ariz~ 13.69            13.12            0.364          15.12           
    ##  9 Arka~ 11.37            11.59            0.678          12.79           
    ## 10 Cali~ 14.49            15.25            0.103          15.03           
    ## # ... with 46 more rows, and 11 more variables: `12-17(2014-2015)` <chr>,
    ## #   `12-17(P Value)` <chr>, `18-25(2013-2014)` <chr>,
    ## #   `18-25(2014-2015)` <chr>, `18-25(P Value)` <chr>,
    ## #   `26+(2013-2014)` <chr>, `26+(2014-2015)` <chr>, `26+(P Value)` <chr>,
    ## #   `18+(2013-2014)` <chr>, `18+(2014-2015)` <chr>, `18+(P Value)` <chr>

## get harry poter data

``` r
hpsaga_html = 
  read_html("https://www.imdb.com/list/ls000630791/")
```

``` r
title_vec = 
  hpsaga_html %>%
  html_nodes(".lister-item-header a") %>%
  html_text()
## 单击选中想要的部分， 双击选中不想要的，拷贝链接到html_nodes()
gross_rev_vec = 
  hpsaga_html %>%
  html_nodes(".text-small:nth-child(7) span:nth-child(5)") %>%
  html_text()

runtime_vec = 
  hpsaga_html %>%
  html_nodes(".runtime") %>%
  html_text()

hpsaga_df = 
  tibble(
    title = title_vec,
    rev = gross_rev_vec,
    runtime = runtime_vec)
```

## Napoleon Dynamite

``` r
url = "https://www.amazon.com/product-reviews/B00005JNBQ/ref=cm_cr_arp_d_viewopt_rvwer?ie=UTF8&reviewerType=avp_only_reviews&sortBy=recent&pageNumber=1"

dynamite_html = read_html(url)

review_titles = 
  dynamite_html %>%
  html_nodes(".a-text-bold span") %>%
  html_text()

review_stars = 
  dynamite_html %>%
  html_nodes("#cm_cr-review_list .review-rating") %>%
  html_text()

review_text = 
  dynamite_html %>%
  html_nodes(".review-text-content span") %>%
  html_text()

reviews = tibble(
  title = review_titles,
  stars = review_stars,
  text = review_text
)
```

## USE API

``` r
nyc_water = 
  GET("https://data.cityofnewyork.us/resource/waf7-5gvc.json") %>% 
  content("text") %>%
  jsonlite::fromJSON() %>%
  as_tibble()
```

``` r
brfss_smart2010 = 
  GET("https://opendata.maryland.gov/api/views/3abd-txk7/rows.csv?accessType=DOWNLOAD") %>% 
  content("parsed")
```

    ## Parsed with column specification:
    ## cols(
    ##   Jurisdiction = col_character(),
    ##   Value = col_double(),
    ##   `Race/ ethnicity` = col_character(),
    ##   Year = col_double(),
    ##   Measure = col_character()
    ## )

``` r
poke = 
  GET("http://pokeapi.co/api/v2/pokemon/1") %>%
  content()

poke$name
```

    ## [1] "bulbasaur"
