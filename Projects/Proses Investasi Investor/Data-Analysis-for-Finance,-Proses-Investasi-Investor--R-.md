Proses Investasi Investor
================

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

1.  loan_id : unique ID of loan uploaded to marketplace
2.  investor_id : unique ID of a registered investor
3.  nama_event : activities carried out by investors and changes in loan
    status
4.  created_at : time (until seconds) the event occurs

## Change created_at column to a Timestamp type

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
grouped with group by nama_event and then calculated with summarise,

-   jumlah_event : to find out the number of events, or how many lines

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

-   investor_register : Event when Investor registers. The number of
    events is the same as the unique investor, means that each investor
    does this event only 1 time. The loan amount is only 1, this is NA,
    because this register does not require a loan.

-   loan_to_marketplace : Event when loan is uploaded to marketplace,
    The number of events is equal to the number of loans, means that
    each loan is uploaded only 1 time. The number of investors is only
    1, this is the content of NA, because when uploaded to the
    marketplace is not related to investors.

-   investor_view_loan : Event when investors viewing loan details on
    the marketplace. The number of events is not the same as unique
    loans or unique investors, means that 1 investor can see the same
    loan several times, and 1 loan can be seen by several different
    investors.

-   investor_order_loan : Events when investors book loans, waiting for
    payment. The number of events is not the same as unique loans or
    unique investors, means that 1 loan can be ordered by several
    different investors (if previous bookings are not paid)

-   investor_pay_loan : Event when the investor pays a loan from the
    previous order. The number of events is the same as a unique loan,
    means that 1 loan can only be paid by 1 investor. The number of
    investors is less than the loan amount means that 1 investor can buy
    many loans.

## Event loan uploaded to marketplace

``` r
df_marketplace<-df_event %>% 
  filter(nama_event=='loan_to_marketplace') %>% 
  select(loan_id,marketplace=created_at)

head(df_marketplace)
```

    ##   loan_id         marketplace
    ## 1       1 2019-07-06 09:03:04
    ## 2       2 2019-07-06 09:00:00
    ## 3       3 2019-07-06 09:03:04
    ## 4       4 2019-07-06 09:03:04
    ## 5       5 2019-07-05 11:45:07
    ## 6       6 2019-07-08 16:35:28

## Event investors view loan details

``` r
df_view_loan<-df_event %>% 
  filter(nama_event=='investor_view_loan') %>% 
  group_by(loan_id,investor_id) %>% 
  summarise(view_count=n(),
           first_view=min(created_at),
           last_view=max(created_at)
           )
```

    ## `summarise()` has grouped output by 'loan_id'. You can override using the `.groups` argument.

``` r
head(df_view_loan)
```

    ## # A tibble: 6 x 5
    ## # Groups:   loan_id [6]
    ##   loan_id investor_id view_count first_view          last_view          
    ##     <int>       <int>      <int> <dttm>              <dttm>             
    ## 1       1         107          1 2019-07-07 11:48:11 2019-07-07 11:48:11
    ## 2       2         114          1 2019-07-07 11:47:58 2019-07-07 11:47:58
    ## 3       3          97          1 2019-07-06 09:50:00 2019-07-06 09:50:00
    ## 4       4          97          1 2019-07-06 09:49:20 2019-07-06 09:49:20
    ## 5       5         107          1 2019-07-05 12:54:25 2019-07-05 12:54:25
    ## 6       6         163          1 2019-07-08 16:40:31 2019-07-08 16:40:31

-   view_count : To know how many times one investor views the loan,
-   first_view : To know when investors first view the details of the
    loan,
-   last_view : To know when investors last view the details of the
    loan.

## Event investors book and pay loans

``` r
library(tidyr)
df_order_pay<-df_event %>% 
  filter(nama_event %in% c('investor_order_loan','investor_pay_loan')) %>%
  spread(nama_event, created_at) %>%
  select(loan_id,investor_id,order=investor_order_loan,pay=investor_pay_loan)
head (df_order_pay)
```

    ##   loan_id investor_id               order                 pay
    ## 1       1         107 2019-07-07 11:48:57 2019-07-07 12:02:18
    ## 2       2         114 2019-07-07 11:48:16 2019-07-07 13:14:39
    ## 3       3          97 2019-07-06 09:50:02 2019-07-06 10:14:44
    ## 4       4          97 2019-07-06 09:49:23 2019-07-06 09:59:51
    ## 5       5         107 2019-07-05 12:55:15 2019-07-05 13:55:54
    ## 6       6         163 2019-07-08 16:42:03 2019-07-08 16:45:56

## Investment Loan Data Combined

``` r
df_loan_invest<- df_marketplace %>% 
  left_join(df_view_loan,by='loan_id') %>% 
  left_join(df_order_pay,by=c('loan_id','investor_id'))
head (df_loan_invest)
```

    ##   loan_id         marketplace investor_id view_count          first_view
    ## 1       1 2019-07-06 09:03:04         107          1 2019-07-07 11:48:11
    ## 2       2 2019-07-06 09:00:00         114          1 2019-07-07 11:47:58
    ## 3       3 2019-07-06 09:03:04          97          1 2019-07-06 09:50:00
    ## 4       4 2019-07-06 09:03:04          97          1 2019-07-06 09:49:20
    ## 5       5 2019-07-05 11:45:07         107          1 2019-07-05 12:54:25
    ## 6       6 2019-07-08 16:35:28         163          1 2019-07-08 16:40:31
    ##             last_view               order                 pay
    ## 1 2019-07-07 11:48:11 2019-07-07 11:48:57 2019-07-07 12:02:18
    ## 2 2019-07-07 11:47:58 2019-07-07 11:48:16 2019-07-07 13:14:39
    ## 3 2019-07-06 09:50:00 2019-07-06 09:50:02 2019-07-06 10:14:44
    ## 4 2019-07-06 09:49:20 2019-07-06 09:49:23 2019-07-06 09:59:51
    ## 5 2019-07-05 12:54:25 2019-07-05 12:55:15 2019-07-05 13:55:54
    ## 6 2019-07-08 16:40:31 2019-07-08 16:42:03 2019-07-08 16:45:56

## Observing the relationship of the number of views with order

``` r
df_loan_invest %>%
  mutate(order_status= ifelse(is.na(order), 'not_order','order')) %>% 
  count(view_count,order_status) %>% 
  spread(order_status,n, fill = 0) %>% 
  mutate(percent_order = scales::percent(order/(order + not_order)))
```

    ##   view_count not_order order percent_order
    ## 1          1       570  3513         86.0%
    ## 2          2        20   173         89.6%
    ## 3          3         3    23         88.5%
    ## 4          4         0     3        100.0%
    ## 5          5         1     1         50.0%
    ## 6          7         0     1        100.0%
    ## 7         40         1     0          0.0%

And it turns out that there is no specific pattern that states the
relationship of many views with the decision of investors to order the
loan. It’s almost evenly distributed that more than 85% of investors who
already see a loan will order it.

For The Number of Views 4 or more, because there are very few events it
can be ignored.

## How long does it take an investor to order since the investor first views the details of the loan?

``` r
df_loan_invest %>%
  filter(!is.na(order)) %>% 
  mutate(length_order_view = as.numeric(difftime(order, first_view, units = "mins"))) %>% 
  group_by(view_count) %>% 
  summarise_at(vars(length_order_view), funs(total = n(), min, median, mean, max)) %>% 
  mutate_if(is.numeric, funs(round(.,2)))
```

    ## Warning: `funs()` was deprecated in dplyr 0.8.0.
    ## Please use a list of either functions or lambdas: 
    ## 
    ##   # Simple named list: 
    ##   list(mean = mean, median = median)
    ## 
    ##   # Auto named with `tibble::lst()`: 
    ##   tibble::lst(mean, median)
    ## 
    ##   # Using lambdas
    ##   list(~ mean(., trim = .2), ~ median(., na.rm = TRUE))
    ## This warning is displayed once every 8 hours.
    ## Call `lifecycle::last_lifecycle_warnings()` to see where this warning was generated.

    ## # A tibble: 6 x 6
    ##   view_count total     min  median    mean    max
    ##        <dbl> <dbl>   <dbl>   <dbl>   <dbl>  <dbl>
    ## 1          1  3513    0.03    1.35    2.97   79.6
    ## 2          2   173    0.43   22.1    61.1  2446. 
    ## 3          3    23    7.25   32.0    66.4   495. 
    ## 4          4     3   17.1    33.9    34.1    51.2
    ## 5          5     1 1113.   1113.   1113.   1113. 
    ## 6          7     1  549.    549.    549.    549.

It turns out that the majority of investors immediately order a loan
when opening the details, which is under 5 minutes for investors who see
the details of the loan 1 time only and then order. For those who open
2-4 times the time is about 30 minutes. At view_count 2 and 3, as there
is an outlier of old messages far from the median, this makes the
average value a high of 1 hour.

## Average booking time since loan is uploaded every week

``` r
library(ggplot2)
df_length_order_per_week <- df_loan_invest %>% 
  filter(!is.na(order)) %>%
  mutate(date = floor_date(marketplace, 'week'),length_order = as.numeric(difftime(order, marketplace, units = "hour"))) %>% 
  group_by(date) %>%
  summarise(length_order = median(length_order))
ggplot(df_length_order_per_week) + 
       geom_line(aes(x = date, y = length_order)) +
       theme_bw() +
       labs(title = "Average order length in 2020 longer than 2019",
            x = "Date",
            y = "time in the marketplace until order (hours)")
```

![](Data-Analysis-for-Finance,-Proses-Investasi-Investor--R-_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

## Does the investor pay for the order they made?

``` r
df_pay_per_week <- df_loan_invest %>% 
  filter(!is.na(order)) %>%
  mutate(date = floor_date(marketplace, 'week')) %>% 
  group_by(date) %>%
  summarise(percent_paid = mean(!is.na(pay)))

ggplot(df_pay_per_week) +
  geom_line(aes(x = date, y = percent_paid)) +
  scale_y_continuous(labels = scales::percent) +
  theme_bw() + 
  labs(title = "About 95% of orders are paid. At the end of May there is an outlier because of eid", 
       x = "Date",
       y = "Paid Orders")
```

![](Data-Analysis-for-Finance,-Proses-Investasi-Investor--R-_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->

## The time it takes for investors to pay for an order

``` r
df_length_pay_per_week <- df_loan_invest %>% 
  filter(!is.na(pay)) %>%
  mutate(date = floor_date(order, 'week'),
         length_pay = as.numeric(difftime(pay, order, units = "hour"))) %>% 
  group_by(date) %>%
  summarise(length_pay = median(length_pay)) 
ggplot(df_length_pay_per_week) +
  geom_line(aes(x = date, y = length_pay)) +
  theme_bw() + 
  labs(title= "The trend's payment time is likely to worsen, 2 times longer than before", 
       x = "Date", 
       y = "paid order time (hours)")
```

![](Data-Analysis-for-Finance,-Proses-Investasi-Investor--R-_files/figure-gfm/unnamed-chunk-12-1.png)<!-- -->

## Trend Investor Register

``` r
df_investor_register <- df_event %>% 
  filter(nama_event=='investor_register') %>%
  mutate(date = floor_date(created_at, 'week')) %>% 
  group_by(date) %>%
  summarise(investor = n_distinct(investor_id)) 
ggplot(df_investor_register) +
    geom_line(aes(x = date, y = investor)) +
    theme_bw() + 
    labs(title="Investor registers briefly rose in early 2020 but have fallen again",
    x="Date", 
    y="Investor Register")
```

![](Data-Analysis-for-Finance,-Proses-Investasi-Investor--R-_files/figure-gfm/unnamed-chunk-13-1.png)<!-- -->
## Trend of the First Investment of the Investor

``` r
df_investor_first_invest <- df_event %>% 
  filter(nama_event=='investor_pay_loan') %>%
  group_by(investor_id) %>% 
  summarise(first_invest = min(created_at)) %>% 
  mutate(date = floor_date(first_invest, 'week')) %>% 
  group_by(date) %>% 
  summarise(investor = n_distinct(investor_id)) 
ggplot(df_investor_first_invest) +
  geom_line(aes(x = date, y = investor)) +
  theme_bw() + 
  labs(title = "There is a trend of increasing the number of investors invested, but it drops dramatically from March 2020.", 
       x = "Date", 
       y = "First Invest")
```

![](Data-Analysis-for-Finance,-Proses-Investasi-Investor--R-_files/figure-gfm/unnamed-chunk-14-1.png)<!-- -->
## Cohort First Invest by Month Register

``` r
df_register_per_investor <- df_event %>%
  filter(nama_event == 'investor_register') %>% 
  rename(date_register = created_at) %>%  
  mutate(month_register = floor_date(date_register, 'month'))  %>%  
  select(investor_id, date_register, month_register) 

df_first_invest_per_investor <- df_event %>%
  filter(nama_event == 'investor_pay_loan') %>% 
  group_by(investor_id) %>% 
  summarise(first_invest = min(created_at))
```

``` r
df_register_per_investor %>% 
  left_join(df_first_invest_per_investor, by = 'investor_id') %>% 
  mutate(length_invest = as.numeric(difftime(first_invest, date_register, units = "day")) %/% 30) %>%  
  group_by(month_register, length_invest) %>% 
  summarise(investor_per_month = n_distinct(investor_id)) %>% 
  group_by(month_register) %>% 
  mutate(register = sum(investor_per_month)) %>% 
  filter(!is.na(length_invest)) %>% 
  mutate(invest = sum(investor_per_month)) %>% 
  mutate(persen_invest = scales::percent(invest/register)) %>% 
  mutate(breakdown_persen_invest = scales::percent(investor_per_month)) %>%  
  select(-investor_per_month) %>%  
  spread(length_invest, breakdown_persen_invest) 
```

    ## `summarise()` has grouped output by 'month_register'. You can override using the `.groups` argument.

    ## # A tibble: 11 x 14
    ## # Groups:   month_register [11]
    ##    month_register      register invest persen_invest `0`    `1`    `2`    `3`  
    ##    <dttm>                 <int>  <int> <chr>         <chr>  <chr>  <chr>  <chr>
    ##  1 2019-07-01 00:00:00     2142     73 3%            4 500% 600%   500%   400% 
    ##  2 2019-08-01 00:00:00     1458     74 5%            4 100% 600%   1 100% 800% 
    ##  3 2019-09-01 00:00:00     1763     94 5%            6 300% 2 000% 400%   200% 
    ##  4 2019-10-01 00:00:00     1437     83 6%            6 400% 700%   400%   600% 
    ##  5 2019-11-01 00:00:00     1607     87 5%            6 600% 1 000% 800%   100% 
    ##  6 2019-12-01 00:00:00     1085     55 5%            3 800% 900%   400%   300% 
    ##  7 2020-01-01 00:00:00     1138     78 7%            6 100% 1 200% 300%   200% 
    ##  8 2020-02-01 00:00:00     1520    115 8%            9 900% 800%   700%   100% 
    ##  9 2020-03-01 00:00:00     2776     53 2%            5 000% 300%   <NA>   <NA> 
    ## 10 2020-04-01 00:00:00     2034     51 3%            4 400% 700%   <NA>   <NA> 
    ## 11 2020-05-01 00:00:00      971      8 1%            800%   <NA>   <NA>   <NA> 
    ## # ... with 6 more variables: 4 <chr>, 5 <chr>, 6 <chr>, 7 <chr>, 8 <chr>,
    ## #   9 <chr>

It is noticed that for the most total registers is in March 2020, as in
the previous chart, there has not been 2% who have invested, very far
compared to the previous month, which could reach 7% more. Which is the
highest conversion rate.

In general, only 5% of investors from all investors who sign up will
convert. And the majority of them do so in the first month (less than 30
days) from registration.

## Cohort Retention Invest

``` r
df_investment_per_investor <- df_event %>%
  filter(nama_event == 'investor_pay_loan') %>%
  rename(date_invest = created_at) %>% 
  select(investor_id, date_invest)
```

``` r
df_first_invest_per_investor %>% 
  mutate(first_month_invest = floor_date(first_invest, 'month'))  %>% 
  inner_join(df_investment_per_investor, by = 'investor_id') %>%
  mutate(range_invest = as.numeric(difftime(date_invest, first_invest, units = "day")) %/% 30) %>% 
  group_by(first_month_invest, range_invest) %>%
  summarise(investor_per_month = n_distinct(investor_id)) %>%
  group_by(first_month_invest) %>%
  mutate(investor = max(investor_per_month)) %>%
  mutate(breakdown_percent_invest = scales::percent(investor_per_month/investor)) %>%
  select(-investor_per_month) %>%
  spread(range_invest, breakdown_percent_invest) %>% 
  select(-`0`)
```

    ## `summarise()` has grouped output by 'first_month_invest'. You can override using the `.groups` argument.

    ## # A tibble: 11 x 11
    ## # Groups:   first_month_invest [11]
    ##    first_month_invest  investor `1`   `2`   `3`   `4`   `5`   `6`   `7`   `8`  
    ##    <dttm>                 <int> <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr>
    ##  1 2019-07-01 00:00:00       31 25.8% 25.8% 19.4% 6.5%  16.1% 16.1% 19.4% 12.9%
    ##  2 2019-08-01 00:00:00       51 35.3% 19.6% 25.5% 19.6% 19.6% 21.6% 9.8%  2.0% 
    ##  3 2019-09-01 00:00:00       70 25.7% 18.6% 18.6% 15.7% 18.6% 10.0% 1.4%  <NA> 
    ##  4 2019-10-01 00:00:00       80 32.5% 28.8% 17.5% 23.7% 8.7%  6.2%  <NA>  <NA> 
    ##  5 2019-11-01 00:00:00       99 30.3% 24.2% 24.2% 8.1%  7.1%  1.0%  <NA>  <NA> 
    ##  6 2019-12-01 00:00:00       63 38.1% 30.2% 3.2%  4.8%  1.6%  <NA>  <NA>  <NA> 
    ##  7 2020-01-01 00:00:00       71 32.4% 12.7% 4.2%  1.4%  <NA>  <NA>  <NA>  <NA> 
    ##  8 2020-02-01 00:00:00      115 16.5% 3.5%  0.9%  <NA>  <NA>  <NA>  <NA>  <NA> 
    ##  9 2020-03-01 00:00:00      102 10.8% 1.0%  <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
    ## 10 2020-04-01 00:00:00       58 5%    <NA>  <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
    ## 11 2020-05-01 00:00:00       31 <NA>  <NA>  <NA>  <NA>  <NA>  <NA>  <NA>  <NA> 
    ## # ... with 1 more variable: 10 <chr>

It is noticed that in February there are investors who make the most
first investments compared to other months. But the retention is worse
than others. In the month after the first investment, only 16% of
investors invest again. This is only half the trend in the previous
month, where about 30% of investors will invest again 1 month after the
first investment.

The most stable cohort was in August 2019. Around the figure of 20%
every month, although in the seventh month the percentage should also
fall as well.

## Conclusion

Based on all the analysis that has been done, it can be concluded that:

1.  In general, the finance company is actually in positive growth,
    fluctuating up and down occurs due to differences in behavior on
    certain dates, which are influenced by other things, such as payday.
2.  In March, April to May there was a lot of decline in the metrics
    analyzed, this may be due to the Covid19 pandemic, it needs to be
    further analyzed whether it is because of it.
3.  In general, 5% of the total investors who register each month, will
    make an investment, and the majority is made in the first 30 days
    after the register, and a small percentage in the second month. In
    the next month the chances are very small to be able to convert. So
    it needs to be ascertained how the investor’s journey is smooth in
    the first month, so that investors want to convert investment in the
    finance company.
4.  Furthermore, it needs to be seen also after that first invest,
    whether investors invest again in the next month. In general, 30% of
    investors will invest again in the following month.
5.  In February, the conversion rate was good, the highest was 7.57%, in
    numbers also the most, but when looking at retention, only 16%
    invested in the following month, only half of the category of other
    months.
