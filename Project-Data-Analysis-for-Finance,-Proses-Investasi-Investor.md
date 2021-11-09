Project Data Analysis for Finance, Proses Investasi Investor
================

## Data Used

``` r
df_event <- read.csv('https://storage.googleapis.com/dqlab-dataset/event.csv', stringsAsFactors = F)
dplyr::glimpse(df_event)
```

    ## Rows: 33,571
    ## Columns: 4
    ## $ loan_id     <int> 2, 2, 2, 133, 133, 133, 2693, 2693, 2693, 6, 6, 6, 2769, 2~
    ## $ investor_id <int> 114, 114, 114, 114, 114, 114, 8159, 8159, 8159, 163, 163, ~
    ## $ nama_event  <chr> "investor_view_loan", "investor_order_loan", "investor_pay~
    ## $ created_at  <chr> "2019-07-07 11:47:58", "2019-07-07 11:48:16", "2019-07-07 ~

It is seen that there are 33,571 rows of data (Observations) and there
are 4 columns (Variables), namely:

1.  ‘loan_id’ : unique ID of loan uploaded to marketplace
2.  ‘investor_id’ : unique ID of a registered investor
3.  ‘nama_event’ : activities carried out by investors and changes in
    loan status
4.  ‘created_at’ : time (until seconds) the event occurs

## Change created_at column to a Timestamp type

By using the ymd_hms function from lubridate package converts characters
in the format year-month-date hour-minute-second to a timestamp type.

``` r
library(lubridate)
```

    ## 
    ## Attaching package: 'lubridate'

    ## The following objects are masked from 'package:base':
    ## 
    ##     date, intersect, setdiff, union

``` r
df_event$created_at <- ymd_hms(df_event$created_at)
dplyr::glimpse(df_event)
```

    ## Rows: 33,571
    ## Columns: 4
    ## $ loan_id     <int> 2, 2, 2, 133, 133, 133, 2693, 2693, 2693, 6, 6, 6, 2769, 2~
    ## $ investor_id <int> 114, 114, 114, 114, 114, 114, 8159, 8159, 8159, 163, 163, ~
    ## $ nama_event  <chr> "investor_view_loan", "investor_order_loan", "investor_pay~
    ## $ created_at  <dttm> 2019-07-07 11:47:58, 2019-07-07 11:48:16, 2019-07-07 13:1~

## Summary Event

Existing data, provided in the form of log per event, then we need to
see what is the content of this event, and how it flows. From the
data.frame, df_event that have been created in the previous section,
grouped with group by nama_event and then calculated with summarise, -
jumlah_event : to find out the number of events, or how many lines

-   loan : To find out the amount of unique loan_id

-   investor : To find out the amount of unique investor_id

``` r
library(dplyr)
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
df_event %>%
  group_by(nama_event) %>%
  summarise(jumlah_event=n(),
            loan=n_distinct(loan_id),
            investor=n_distinct(investor_id))
```

    ## # A tibble: 5 x 4
    ##   nama_event          jumlah_event  loan investor
    ##   <chr>                      <int> <int>    <int>
    ## 1 investor_order_loan         3714  3641      804
    ## 2 investor_pay_loan           3632  3632      771
    ## 3 investor_register          17931     1    17931
    ## 4 investor_view_loan          4616  3678     1095
    ## 5 loan_to_marketplace         3678  3678        1

Based on these results, there are 5 events. With the following
explanation:

``` r
summary(cars)
```

    ##      speed           dist       
    ##  Min.   : 4.0   Min.   :  2.00  
    ##  1st Qu.:12.0   1st Qu.: 26.00  
    ##  Median :15.0   Median : 36.00  
    ##  Mean   :15.4   Mean   : 42.98  
    ##  3rd Qu.:19.0   3rd Qu.: 56.00  
    ##  Max.   :25.0   Max.   :120.00

## Including Plots

You can also embed plots, for example:

![](Project-Data-Analysis-for-Finance,-Proses-Investasi-Investor_files/figure-gfm/pressure-1.png)<!-- -->

Note that the `echo = FALSE` parameter was added to the code chunk to
prevent printing of the R code that generated the plot.
