

# Data table examples using public data

By Gene Leynes
Chicago Data Science

## Introduction

This code was used to demonstrate the features of the data.table package at the R User Group meetup held in Chicago on February 7th, 2013.

The data and project related to this exmaple comes from another meetup and code project.  I attended that meetup, and put together some simple visualizations for inspiration.  These visualizations were intended to be used as talking points for the project.  I repurposed this data to provide some `data.table` examples.

Meetup group: http://www.meetup.com/The-Chicago-Data-Visualization-Group/events/97690642/
Github project related to meetup: https://github.com/sc3/26thandcalifornia
My related project: https://github.com/geneorama/26_and_California

Exmples complied using `knitr`:
Original meetup example: http://chicagodatascience.com/public/26th_and_California_example_visualizations.html
This example: 


## INITIALIZE

```r
# source('00 Initialize.R')
rm(list = ls())
library(data.table)
library(ggplot2)
source("functions/ExtractIsoTime.R")
source("functions/wtf.R")
source("functions/NAsummary.R")
```


## READ CSV

This csv contains raw data collected from the Cook Count Sherrif's website.  It was downloaded using a web scraping utility maintained here: https://github.com/sc3/cookcountyjail



```r
rawdat = read.table(file = "Database 2013-01-21 (8zQ4cW7T).csv", sep = ",", 
    quote = "\"", flush = FALSE, header = TRUE, nrows = -1, fill = FALSE, stringsAsFactors = FALSE, 
    na.strings = c("None", ""))

str(rawdat)
```

```
## 'data.frame':	19207 obs. of  11 variables:
##  $ charges_citation       : chr  "720 ILCS 5 12-3.4(a)(2) [16145" "625 ILCS 5 6-101 [12935]" "720 ILCS 5 12-3(a)(1) [10529]" "720 ILCS 550 5(c) [5020200]" ...
##  $ race                   : chr  "WH" "LW" "BK" "BK" ...
##  $ age_at_booking         : int  26 37 18 32 49 26 41 56 40 20 ...
##  $ gender                 : chr  "M" "M" "M" "F" ...
##  $ booking_date           : chr  "2013-01-20T00:00:00" "2013-01-20T00:00:00" "2013-01-20T00:00:00" "2013-01-20T00:00:00" ...
##  $ jail_id                : chr  "2013-0120171" "2013-0120170" "2013-0120169" "2013-0120167" ...
##  $ bail_status            : chr  NA NA NA NA ...
##  $ housing_location       : chr  "05-" "05-" "05-L-2-2-1" "17-WR-N-A-2" ...
##  $ charges                : chr  NA NA NA NA ...
##  $ bail_amount            : int  5000 10000 5000 50000 5000 5000 25000 5000 25000 10000 ...
##  $ discharge_date_earliest: chr  NA NA NA NA ...
```

```r
dat = as.data.table(rawdat)
str(dat)
```

```
## Classes 'data.table' and 'data.frame':	19207 obs. of  11 variables:
##  $ charges_citation       : chr  "720 ILCS 5 12-3.4(a)(2) [16145" "625 ILCS 5 6-101 [12935]" "720 ILCS 5 12-3(a)(1) [10529]" "720 ILCS 550 5(c) [5020200]" ...
##  $ race                   : chr  "WH" "LW" "BK" "BK" ...
##  $ age_at_booking         : int  26 37 18 32 49 26 41 56 40 20 ...
##  $ gender                 : chr  "M" "M" "M" "F" ...
##  $ booking_date           : chr  "2013-01-20T00:00:00" "2013-01-20T00:00:00" "2013-01-20T00:00:00" "2013-01-20T00:00:00" ...
##  $ jail_id                : chr  "2013-0120171" "2013-0120170" "2013-0120169" "2013-0120167" ...
##  $ bail_status            : chr  NA NA NA NA ...
##  $ housing_location       : chr  "05-" "05-" "05-L-2-2-1" "17-WR-N-A-2" ...
##  $ charges                : chr  NA NA NA NA ...
##  $ bail_amount            : int  5000 10000 5000 50000 5000 5000 25000 5000 25000 10000 ...
##  $ discharge_date_earliest: chr  NA NA NA NA ...
##  - attr(*, ".internal.selfref")=<externalptr>
```


## Convert booking and discharge dates to date / time objects

This relies on a simple function I wrote called `ExtractIsoTime` (which is in the functions folder).   The thing to notice here is that when you create new columns using `data.table` you use `:=` to denote assignment.


```r
## EXAMPLE FORMAT: 2012-12-30T20:57:19.616186
dat[, `:=`(booking_date, ExtractIsoTime(dat$booking_date))]
```

```
##                      charges_citation race age_at_booking gender
##     1: 720 ILCS 5 12-3.4(a)(2) [16145   WH             26      M
##     2:       625 ILCS 5 6-101 [12935]   LW             37      M
##     3:  720 ILCS 5 12-3(a)(1) [10529]   BK             18      M
##     4:    720 ILCS 550 5(c) [5020200]   BK             32      F
##     5: 720 ILCS 5 12-3.2(a)(2) [10418   LW             49      M
##    ---                                                          
## 19203:                    95.5-11-501   WH             31      M
## 19204:                      56.5-1402   LW             28      M
## 19205:                      56.5-1402   BK             36      M
## 19206:                        38-10-5   LW             23      M
## 19207:                      56.5-1401   LW             27      M
##        booking_date      jail_id bail_status housing_location charges
##     1:   2013-01-20 2013-0120171          NA              05-      NA
##     2:   2013-01-20 2013-0120170          NA              05-      NA
##     3:   2013-01-20 2013-0120169          NA       05-L-2-2-1      NA
##     4:   2013-01-20 2013-0120167          NA      17-WR-N-A-2      NA
##     5:   2013-01-20 2013-0120165          NA              05-      NA
##    ---                                                               
## 19203:   1995-05-09 1995-9532061     NO BOND          15-DRAW      NA
## 19204:   1994-09-22 1994-9459745          NA          15-EMAW      NA
## 19205:   1993-09-24 1993-9357382          NA          15-DRAW      NA
## 19206:   1993-05-07 1993-9326844          NA          15-EMAW      NA
## 19207:   1993-01-16 1993-9303175          NA          15-EMAW      NA
##        bail_amount discharge_date_earliest
##     1:        5000                      NA
##     2:       10000                      NA
##     3:        5000                      NA
##     4:       50000                      NA
##     5:        5000                      NA
##    ---                                    
## 19203:          NA                      NA
## 19204:      250000                      NA
## 19205:       10000                      NA
## 19206:       60000                      NA
## 19207:      100000                      NA
```

```r
dat[, `:=`(discharge_date_earliest, ExtractIsoTime(dat$discharge_date_earliest))]
```

```
##                      charges_citation race age_at_booking gender
##     1: 720 ILCS 5 12-3.4(a)(2) [16145   WH             26      M
##     2:       625 ILCS 5 6-101 [12935]   LW             37      M
##     3:  720 ILCS 5 12-3(a)(1) [10529]   BK             18      M
##     4:    720 ILCS 550 5(c) [5020200]   BK             32      F
##     5: 720 ILCS 5 12-3.2(a)(2) [10418   LW             49      M
##    ---                                                          
## 19203:                    95.5-11-501   WH             31      M
## 19204:                      56.5-1402   LW             28      M
## 19205:                      56.5-1402   BK             36      M
## 19206:                        38-10-5   LW             23      M
## 19207:                      56.5-1401   LW             27      M
##        booking_date      jail_id bail_status housing_location charges
##     1:   2013-01-20 2013-0120171          NA              05-      NA
##     2:   2013-01-20 2013-0120170          NA              05-      NA
##     3:   2013-01-20 2013-0120169          NA       05-L-2-2-1      NA
##     4:   2013-01-20 2013-0120167          NA      17-WR-N-A-2      NA
##     5:   2013-01-20 2013-0120165          NA              05-      NA
##    ---                                                               
## 19203:   1995-05-09 1995-9532061     NO BOND          15-DRAW      NA
## 19204:   1994-09-22 1994-9459745          NA          15-EMAW      NA
## 19205:   1993-09-24 1993-9357382          NA          15-DRAW      NA
## 19206:   1993-05-07 1993-9326844          NA          15-EMAW      NA
## 19207:   1993-01-16 1993-9303175          NA          15-EMAW      NA
##        bail_amount discharge_date_earliest
##     1:        5000                    <NA>
##     2:       10000                    <NA>
##     3:        5000                    <NA>
##     4:       50000                    <NA>
##     5:        5000                    <NA>
##    ---                                    
## 19203:          NA                    <NA>
## 19204:      250000                    <NA>
## 19205:       10000                    <NA>
## 19206:       60000                    <NA>
## 19207:      100000                    <NA>
```



# Examples with data.table


```r

## Not directly related to data.table, but I like to see how many unique
## and how many missing values each column has.

NAsummary(dat)
```

```
##                         col Count   nNA    rNA nUnique rUnique
## charges_citation          1 19207   315 0.0164    1435  0.0747
## race                      2 19207     0 0.0000       9  0.0004
## age_at_booking            3 19207     0 0.0000      67  0.0034
## gender                    4 19207     0 0.0000       2  0.0001
## booking_date              5 19207   151 0.0078    4160  0.2165
## jail_id                   6 19207     0 0.0000   19207  1.0000
## bail_status               7 19207 10318 0.5371       5  0.0002
## housing_location          8 19207     5 0.0002    6115  0.3183
## charges                   9 19207  3052 0.1589    1075  0.0559
## bail_amount              10 19207  6415 0.3339     119  0.0061
## discharge_date_earliest  11 19207 10627 0.5532    8581  0.4467
```

```r


## Summary by bail amount the old way: (Don't forget the useNA argument!!)
table(dat$bail_amount)
```

```
## 
##      10      80     100     146     200     250     300     303     350 
##       3       1       8       1       1       3       1       1       1 
##     500     559     600     663     750     751     905     940    1000 
##      21       1       1       1       1       1       1       1     139 
##    1350    1445    1500    1516    1580    1695    1936    2000    2500 
##       1       1      13       1       1       1       1      93      87 
##    2620    3000    3500    4000    4400    4500    5000    5500    6000 
##       1      81      46      11       1       1     991       1       5 
##    7000    7500    8000    8500    9000   10000   11000   12000   12500 
##       5     106       4       1       4    1311       2       8       1 
##   13000   13700   15000   17000   19000   19412   20000   24840   25000 
##       1       1     295       1       1       1     528       1    1194 
##   30000   35000   37500   40000   45000   48086   50000   55000   60000 
##     480      78       2     281       8       1    1912      83     104 
##   62500   65000   70000   75000   80000   85000   90000   95000  100000 
##       1       9      34     801     134       6     225      13     899 
##  110000  112000  115000  120000  125000  130000  135000  140000  150000 
##       1       1       1      10     143       2       1       1     516 
##  160000  175000  180000  200000  220000  225000  250000  275000  280000 
##       1      37       1     279       1       6     442      42       1 
##  300000  325000  350000  375000  400000  450000  475000  500000  510000 
##     233      12     101       1     113      31       1     267       1 
##  550000  600000  650000  700000  750000  755000  800000  850000  900000 
##       3      17       6       9      86       1      17       2      25 
##  950000 1000000 1200000 1250000 1500000 2000000 2500000 3000000 4000000 
##       5     176       1       1      26      68      10      14       1 
## 5000000 
##      18
```

```r
table(dat$bail_amount, useNA = "ifany")  # almost forgot
```

```
## 
##      10      80     100     146     200     250     300     303     350 
##       3       1       8       1       1       3       1       1       1 
##     500     559     600     663     750     751     905     940    1000 
##      21       1       1       1       1       1       1       1     139 
##    1350    1445    1500    1516    1580    1695    1936    2000    2500 
##       1       1      13       1       1       1       1      93      87 
##    2620    3000    3500    4000    4400    4500    5000    5500    6000 
##       1      81      46      11       1       1     991       1       5 
##    7000    7500    8000    8500    9000   10000   11000   12000   12500 
##       5     106       4       1       4    1311       2       8       1 
##   13000   13700   15000   17000   19000   19412   20000   24840   25000 
##       1       1     295       1       1       1     528       1    1194 
##   30000   35000   37500   40000   45000   48086   50000   55000   60000 
##     480      78       2     281       8       1    1912      83     104 
##   62500   65000   70000   75000   80000   85000   90000   95000  100000 
##       1       9      34     801     134       6     225      13     899 
##  110000  112000  115000  120000  125000  130000  135000  140000  150000 
##       1       1       1      10     143       2       1       1     516 
##  160000  175000  180000  200000  220000  225000  250000  275000  280000 
##       1      37       1     279       1       6     442      42       1 
##  300000  325000  350000  375000  400000  450000  475000  500000  510000 
##     233      12     101       1     113      31       1     267       1 
##  550000  600000  650000  700000  750000  755000  800000  850000  900000 
##       3      17       6       9      86       1      17       2      25 
##  950000 1000000 1200000 1250000 1500000 2000000 2500000 3000000 4000000 
##       5     176       1       1      26      68      10      14       1 
## 5000000    <NA> 
##      18    6415
```

```r

## Summary by bail amount Data Table
dat[, .N, by = bail_amount]
```

```
##      bail_amount    N
##   1:        5000  991
##   2:       10000 1311
##   3:       50000 1912
##   4:       25000 1194
##   5:      100000  899
##  ---                 
## 115:         146    1
## 116:      110000    1
## 117:      510000    1
## 118:        1350    1
## 119:       13000    1
```

```r
dat[, .N, keyby = bail_amount]
```

```
##      bail_amount    N
##   1:          NA 6415
##   2:          10    3
##   3:          80    1
##   4:         100    8
##   5:         146    1
##  ---                 
## 115:     2000000   68
## 116:     2500000   10
## 117:     3000000   14
## 118:     4000000    1
## 119:     5000000   18
```

```r

## Summary by race
dat[, .N, by = race]
```

```
##    race     N
## 1:   WH  2015
## 2:   LW  1239
## 3:   BK 13879
## 4:   LT  1803
## 5:   AS   117
## 6:    W    44
## 7:    B    29
## 8:   LB    68
## 9:   IN    13
```

```r

############################################# SUBSETTING:

## WRONG WAY:
dat[, 3]
```

```
## [1] 3
```

```r
dat[, "race"]
```

```
## [1] "race"
```

```r

## RIGHT WAY:
dat[1:10, race]
```

```
##  [1] "WH" "LW" "BK" "BK" "LW" "BK" "BK" "BK" "LW" "BK"
```

```r
dat[1:10, 3, with = F]
```

```
##     age_at_booking
##  1:             26
##  2:             37
##  3:             18
##  4:             32
##  5:             49
##  6:             26
##  7:             41
##  8:             56
##  9:             40
## 10:             20
```

```r


## Indexing works differently
dat[1]
```

```
##                  charges_citation race age_at_booking gender booking_date
## 1: 720 ILCS 5 12-3.4(a)(2) [16145   WH             26      M   2013-01-20
##         jail_id bail_status housing_location charges bail_amount
## 1: 2013-0120171          NA              05-      NA        5000
##    discharge_date_earliest
## 1:                    <NA>
```

```r

## DF:
df = as.data.frame(dat)
# df[1]
df[1, ]
```

```
##                 charges_citation race age_at_booking gender booking_date
## 1 720 ILCS 5 12-3.4(a)(2) [16145   WH             26      M   2013-01-20
##        jail_id bail_status housing_location charges bail_amount
## 1 2013-0120171        <NA>              05-    <NA>        5000
##   discharge_date_earliest
## 1                    <NA>
```

```r



############################################# AGGREGATING:

## Grouping is simple, but...
dat[, mean(age_at_booking), by = race]
```

```
##    race    V1
## 1:   WH 35.04
## 2:   LW 31.45
## 3:   BK 31.64
## 4:   LT 29.41
## 5:   AS 33.20
## 6:    W 29.89
## 7:    B 29.86
## 8:   LB 31.97
## 9:   IN 27.00
```

```r
dat[, age_at_booking, by = race]
```

```
##        race age_at_booking
##     1:   WH             26
##     2:   WH             52
##     3:   WH             58
##     4:   WH             39
##     5:   WH             26
##    ---                    
## 19203:   IN             37
## 19204:   IN             35
## 19205:   IN             19
## 19206:   IN             27
## 19207:   IN             18
```

```r
dat[i = TRUE, j = list(mean = mean(age_at_booking), sd = sd(age_at_booking)), 
    by = race]
```

```
##    race  mean    sd
## 1:   WH 35.04 11.90
## 2:   LW 31.45 10.05
## 3:   BK 31.64 12.02
## 4:   LT 29.41 10.35
## 5:   AS 33.20 12.65
## 6:    W 29.89 10.54
## 7:    B 29.86 10.63
## 8:   LB 31.97 12.08
## 9:   IN 27.00 10.25
```

```r

## Sometimes things happen that you may not expect It's good, but possibly
## a surprise
dat[i = TRUE, j = list(mean = mean(age_at_booking), sd = sd(age_at_booking), 
    age_at_booking), by = race]
```

```
##        race  mean    sd age_at_booking
##     1:   WH 35.04 11.90             26
##     2:   WH 35.04 11.90             52
##     3:   WH 35.04 11.90             58
##     4:   WH 35.04 11.90             39
##     5:   WH 35.04 11.90             26
##    ---                                
## 19203:   IN 27.00 10.25             37
## 19204:   IN 27.00 10.25             35
## 19205:   IN 27.00 10.25             19
## 19206:   IN 27.00 10.25             27
## 19207:   IN 27.00 10.25             18
```


# Complex query example


```r
NAsummary(dat)
```

```
##                         col Count   nNA    rNA nUnique rUnique
## charges_citation          1 19207   315 0.0164    1435  0.0747
## race                      2 19207     0 0.0000       9  0.0004
## age_at_booking            3 19207     0 0.0000      67  0.0034
## gender                    4 19207     0 0.0000       2  0.0001
## booking_date              5 19207   151 0.0078    4160  0.2165
## jail_id                   6 19207     0 0.0000   19207  1.0000
## bail_status               7 19207 10318 0.5371       5  0.0002
## housing_location          8 19207     5 0.0002    6115  0.3183
## charges                   9 19207  3052 0.1589    1075  0.0559
## bail_amount              10 19207  6415 0.3339     119  0.0061
## discharge_date_earliest  11 19207 10627 0.5532    8581  0.4467
```

```r

mysummary = dat[i = !is.na(charges) & !is.na(booking_date) & !is.na(bail_amount), 
    j = list(count = .N, coverage = diff(range(booking_date)), bailave = mean(bail_amount), 
        bailsd = sd(bail_amount), bailmin = min(bail_amount), bailmax = max(bail_amount)), 
    by = list(race, gender, age_at_booking)]
mysummary
```

```
##      race gender age_at_booking count coverage bailave bailsd bailmin
##   1:   WH      F             36     7 333 days   72286 103842    1000
##   2:   BK      F             19    24 630 days  170604 604854    2000
##   3:   WH      M             20    36 267 days  138250 295435    1000
##   4:   LT      M             27    27 829 days  154444 205671    5000
##   5:   LT      M             28    25 960 days  292100 607991    7500
##  ---                                                                 
## 467:    W      M             17     1   0 days  100000     NA  100000
## 468:    B      M             40     1   0 days 1000000     NA 1000000
## 469:   LB      M             23     1   0 days  250000     NA  250000
## 470:    W      M             51     1   0 days 5000000     NA 5000000
## 471:   AS      M             60     1   0 days 5000000     NA 5000000
##      bailmax
##   1:  300000
##   2: 3000000
##   3: 1500000
##   4:  900000
##   5: 3000000
##  ---        
## 467:  100000
## 468: 1000000
## 469:  250000
## 470: 5000000
## 471: 5000000
```

```r
# wtf(mysummary)
```



# J and CJ


```r

dat[, .N, by = race]
```

```
##    race     N
## 1:   WH  2015
## 2:   LW  1239
## 3:   BK 13879
## 4:   LT  1803
## 5:   AS   117
## 6:    W    44
## 7:    B    29
## 8:   LB    68
## 9:   IN    13
```

```r

setkey(dat, "race")
dat["W"]
```

```
##     race               charges_citation age_at_booking gender
##  1:    W 720 ILCS 5 12-3.4(a)(1) [16128             34      M
##  2:    W    625 ILCS 5 6-303(a) [13526]             34      M
##  3:    W   625 ILCS 5 11-501(a) [14041]             34      M
##  4:    W    625 ILCS 5 6-303(a) [13526]             33      M
##  5:    W 720 ILCS 5 12-13(a)(3) [995700             39      M
##  6:    W                            000             26      M
##  7:    W     720 ILCS 5 12-3.2 [930200]             25      M
##  8:    W 720 ILCS 570 401(a)(2)(D) [509             35      M
##  9:    W 720 ILCS 5 12-3.05(d)(4) [1610             20      M
## 10:    W 720 ILCS 5 12-3.2(a)(1) [10416             43      M
## 11:    W 720 ILCS 570 401(c)(1) [13009]             21      M
## 12:    W    720 ILCS 5 16A-3(a) [15599]             23      M
## 13:    W  720 ILCS 570 402(c) [5101110]             42      M
## 14:    W                            000             19      M
## 15:    W    720 ILCS 5 12-1(a) [920000]             33      M
## 16:    W  720 ILCS 5 12-3(a)(2) [10530]             25      M
## 17:    W  625 ILCS 5 6-303(d) [5883000]             33      M
## 18:    W  720 ILCS 5 16A-3(a) [1060000]             20      M
## 19:    W       720 ILCS 5 12-3 [930000]             19      M
## 20:    W  720 ILCS 570 402(c) [5101110]             53      M
## 21:    W   720 ILCS 5 16-3(a) [1025000]             27      F
## 22:    W   720 ILCS 5 19-1(a) [1110000]             21      M
## 23:    W 720 ILCS 5 12-3.2(a)(1) [10416             36      M
## 24:    W 720 ILCS 5 12-3.2(a)(2) [10418             26      M
## 25:    W   625 ILCS 5 11-501(a) [14039]             24      F
## 26:    W  720 ILCS 570 402(c) [5101110]             43      M
## 27:    W   625 ILCS 5 11-501(a) [12809]             38      M
## 28:    W  720 ILCS 570 402(c) [5101110]             22      M
## 29:    W 625 ILCS 5 11-501(a)(2) [11309             23      M
## 30:    W             720 ILCS 5/19-1(a)             19      M
## 31:    W                        UNKNOWN             36      M
## 32:    W            625 ILCS 5/6-303(a)             46      M
## 33:    W           625 ILCS 5/11-501(a)             25      M
## 34:    W             720 ILCS 5/19-1(a)             19      M
## 35:    W           625 ILCS 5/11-501(a)             21      F
## 36:    W           720 ILCS 5/9-1(a)(1)             23      M
## 37:    W                38-24-3.1(a)(6)             20      M
## 38:    W           625 ILCS 5/11-501(a)             44      M
## 39:    W               720 ILCS 570/402             18      M
## 40:    W              720 ILCS 550/5(g)             57      M
## 41:    W       720 ILCS 5/12-14.1(a)(1)             29      M
## 42:    W           720 ILCS 5/12-4.3(a)             19      M
## 43:    W              720 ILCS 5/24-1.1             17      M
## 44:    W                  38-10-2(a)(3)             51      M
##     race               charges_citation age_at_booking gender
##            booking_date      jail_id     bail_status housing_location
##  1: 2013-01-19 00:00:00 2013-0119222              NA         08-2N-DR
##  2: 2013-01-18 00:00:00 2013-0118220              NA      11-AH-3-411
##  3: 2013-01-18 00:00:00 2013-0118212              NA              02-
##  4: 2013-01-18 00:00:00 2013-0118203              NA      02-D4-MU-1-
##  5: 2013-01-18 00:00:00 2013-0118056              NA       05-D-1-2-1
##  6: 2013-01-18 00:00:00 2013-0118051              NA      02-D1-H-3-H
##  7: 2013-01-13 00:00:00 2013-0113043         NO BOND       03-A-2-4-2
##  8: 2013-01-11 00:00:00 2013-0111097              NA      11-AF-3-311
##  9: 2013-01-04 00:00:00 2013-0104171         NO BOND       01-B-2-3-1
## 10: 2013-01-03 00:00:00 2013-0103001              NA      02-D3-HH-3-
## 11: 2012-12-28 00:00:00 2012-1228050         NO BOND      03-AX-D3-1-
## 12: 2012-12-27 00:00:00 2012-1227035              NA      02-D4-QU-1-
## 13: 2012-12-22 12:28:29 2012-1217166              NA      03-AX-B1-1-
## 14: 2012-12-07 00:00:00 2012-1207055         NO BOND       03-A-3-4-1
## 15: 2012-12-03 00:00:00 2012-1203014         NO BOND      02-D4-RL-1-
## 16: 2012-12-02 00:00:00 2012-1202178         NO BOND       10-A-3-8-2
## 17: 2012-12-22 11:37:58 2012-1121001              NA      02-D1-A-2-A
## 18: 2012-12-15 22:40:08 2012-1117082 Bond in Process      02-D2-W-2-W
## 19: 2012-11-16 00:00:00 2012-1116207              NA      02-D4-ML-1-
## 20: 2012-11-02 00:00:00 2012-1102164              NA      14-B4-4-42-
## 21: 2012-10-31 00:00:00 2012-1031214         NO BOND      04-J-1-11-1
## 22: 2012-10-30 00:00:00 2012-1030177         NO BOND      02-D1-A-2-A
## 23: 2012-10-29 00:00:00 2012-1029031         NO BOND      03-A-2-19-1
## 24: 2012-12-18 02:02:39 2012-1017203              NA      02-D2-U-3-U
## 25: 2012-12-15 03:10:58 2012-1011190              NA          17-SFFP
## 26: 2012-10-10 00:00:00 2012-1010016         NO BOND      03-A-1-26-1
## 27: 2012-09-20 00:00:00 2012-0920237         NO BOND      06-H-2-19-2
## 28: 2012-12-22 12:23:40 2012-0909136              NA      11-AC-1-208
## 29: 2012-09-04 00:00:00 2012-0904227 Bond in Process            15-EM
## 30: 2012-08-31 00:00:00 2012-0831240         NO BOND       01-E-1-1-1
## 31: 2012-08-29 00:00:00 2012-0829215              NA      01-G-3-16-1
## 32: 2012-08-08 00:00:00 2012-0808183         NO BOND            15-EM
## 33: 2012-07-10 00:00:00 2012-0710111              NA       06-D-1-3-2
## 34: 2012-07-08 00:00:00 2012-0708142         NO BOND            15-EM
## 35: 2012-07-02 00:00:00 2012-0702163              NA      17-WR-N-C-2
## 36: 2012-06-22 00:00:00 2012-0622226         NO BOND       01-A-3-3-2
## 37: 2012-06-16 00:00:00 2012-0616119              NA      06-H-1-12-2
## 38: 2012-05-09 00:00:00 2012-0509091              NA      02-D1-H-3-H
## 39: 2012-03-20 00:00:00 2012-0320179 Bond in Process            15-DR
## 40: 2011-12-14 00:00:00 2011-1214202              NA       10-C-1-7-1
## 41: 2011-11-16 00:00:00 2011-1116167         NO BOND          C DISCH
## 42: 2011-10-07 00:00:00 2011-1007202              NA      11-DH-3-411
## 43: 2010-09-09 00:00:00 2010-0909032              NA      06-C-1-13-1
## 44: 2010-04-19 00:00:00 2010-0419201              NA      01-H-1-13-2
##            booking_date      jail_id     bail_status housing_location
##                                              charges bail_amount
##  1:                                               NA       50000
##  2:                         DRVG ON SUSP LICENSE/FTA       10000
##  3:                                               NA       10000
##  4:                         DRVG ON SUSP LICENSE/FTA        3000
##  5:                        CRIM SEX ASSAULT/FAMILIES       50000
##  6:                           VIOLATION OF PROBATION      100000
##  7:                                 BATTERY/DOMESTIC        5000
##  8:                     MFG/DEL 900+ GR COCAINE/ANLG        5000
##  9:                        AGG BATTERY/PEACE OFFICER          NA
## 10:                     DOMESTIC BATTERY/BODILY HARM        5000
## 11:                                        MFG/DEL 1          NA
## 12:                            RET THEFT/DISP MERCH/        5000
## 13:                   POSS AMT CON SUB EXCEPT(A)/(D)        5000
## 14:                           VIOLATION OF PROBATION          NA
## 15:                       KNOW DAMAGE PROP >$300-10K          NA
## 16:                   BATTERY/MAKES PHYSICAL CONTACT       50000
## 17:                   DRIVING REVOKED/SUSPENDED 2ND+      500000
## 18:                       OBSTRUCTING IDENTIFICATION          NA
## 19:                                          BATTERY       25000
## 20:                   POSS AMT CON SUB EXCEPT(A)/(D)       75000
## 21:                    THEFT/LABOR/SERVICES/PROPERTY          NA
## 22:                                         BURGLARY          NA
## 23:                       OBSTRUCTING IDENTIFICATION          NA
## 24:                   DOMESTIC BTRY/PHYSICAL CONTACT       20000
## 25:                                           DUI/6+      200000
## 26:                   POSS AMT CON SUB EXCEPT(A)/(D)       50000
## 27:                    DUI/BAC .16 OR CHILD PASS/1ST          NA
## 28:                   POSS AMT CON SUB EXCEPT(A)/(D)       40000
## 29:                    DUI ALCOHOL W/CHILD PASSENGER          NA
## 30:              BURGLARY SCHOOL OR PLACE OF WORSHIP       10000
## 31:                         UNKNOWN OR NOT AVAILABLE      500000
## 32:                                               NA          NA
## 33:                        DUI/INTOXICATING COMPOUND     1000000
## 34:                                               NA          NA
## 35:                        DUI/INTOXICATING COMPOUND       95000
## 36:     FIRST DEGREE MURDER:INTENDS DEATH/GREAT HARM          NA
## 37: UNLAWFUL POSSESSION OF FIREARM:EXPLOSIVE BULLETS        5000
## 38:                        DUI/INTOXICATING COMPOUND       35000
## 39:      ILLEGAL POSSESSION OF CONTROLLED SUBSTANCES          NA
## 40:                    MAN/DEL CANNABIS/>5,000 GRAMS      500000
## 41:                   PREDATORY CRIMINAL SEXUAL ASLT      750000
## 42:  AGGRAVATED BATTERY OF A CHILD:FIREARM/DISC/HARM      200000
## 43: UNLAWFUL USE/POSSESSION OF WEAPON:FELON PRISONER      100000
## 44:                AGGRAVATED KIDNAPING:INFLICT HARM     5000000
##                                              charges bail_amount
##     discharge_date_earliest
##  1:                    <NA>
##  2:                    <NA>
##  3:     2013-01-19 06:32:06
##  4:                    <NA>
##  5:                    <NA>
##  6:                    <NA>
##  7:                    <NA>
##  8:                    <NA>
##  9:                    <NA>
## 10:     2013-01-14 06:27:53
## 11:     2012-12-30 18:25:17
## 12:                    <NA>
## 13:     2012-12-23 11:06:55
## 14:                    <NA>
## 15:                    <NA>
## 16:     2013-01-09 05:04:27
## 17:     2012-12-23 10:27:10
## 18:     2012-12-15 01:49:02
## 19:     2013-01-05 06:48:04
## 20:                    <NA>
## 21:     2012-12-19 23:27:34
## 22:                    <NA>
## 23:     2013-01-14 06:43:50
## 24:     2012-12-17 01:06:47
## 25:     2012-12-14 02:44:33
## 26:     2013-01-11 05:07:45
## 27:                    <NA>
## 28:     2012-12-23 11:01:37
## 29:     2012-12-19 23:27:30
## 30:                    <NA>
## 31:                    <NA>
## 32:     2013-01-06 06:41:44
## 33:                    <NA>
## 34:                    <NA>
## 35:                    <NA>
## 36:     2012-12-17 00:10:14
## 37:     2013-01-13 07:18:08
## 38:     2013-01-19 07:20:07
## 39:                    <NA>
## 40:                    <NA>
## 41:     2012-12-17 22:35:19
## 42:                    <NA>
## 43:                    <NA>
## 44:                    <NA>
##     discharge_date_earliest
```

```r
dat[J("W")]
```

```
##     race               charges_citation age_at_booking gender
##  1:    W 720 ILCS 5 12-3.4(a)(1) [16128             34      M
##  2:    W    625 ILCS 5 6-303(a) [13526]             34      M
##  3:    W   625 ILCS 5 11-501(a) [14041]             34      M
##  4:    W    625 ILCS 5 6-303(a) [13526]             33      M
##  5:    W 720 ILCS 5 12-13(a)(3) [995700             39      M
##  6:    W                            000             26      M
##  7:    W     720 ILCS 5 12-3.2 [930200]             25      M
##  8:    W 720 ILCS 570 401(a)(2)(D) [509             35      M
##  9:    W 720 ILCS 5 12-3.05(d)(4) [1610             20      M
## 10:    W 720 ILCS 5 12-3.2(a)(1) [10416             43      M
## 11:    W 720 ILCS 570 401(c)(1) [13009]             21      M
## 12:    W    720 ILCS 5 16A-3(a) [15599]             23      M
## 13:    W  720 ILCS 570 402(c) [5101110]             42      M
## 14:    W                            000             19      M
## 15:    W    720 ILCS 5 12-1(a) [920000]             33      M
## 16:    W  720 ILCS 5 12-3(a)(2) [10530]             25      M
## 17:    W  625 ILCS 5 6-303(d) [5883000]             33      M
## 18:    W  720 ILCS 5 16A-3(a) [1060000]             20      M
## 19:    W       720 ILCS 5 12-3 [930000]             19      M
## 20:    W  720 ILCS 570 402(c) [5101110]             53      M
## 21:    W   720 ILCS 5 16-3(a) [1025000]             27      F
## 22:    W   720 ILCS 5 19-1(a) [1110000]             21      M
## 23:    W 720 ILCS 5 12-3.2(a)(1) [10416             36      M
## 24:    W 720 ILCS 5 12-3.2(a)(2) [10418             26      M
## 25:    W   625 ILCS 5 11-501(a) [14039]             24      F
## 26:    W  720 ILCS 570 402(c) [5101110]             43      M
## 27:    W   625 ILCS 5 11-501(a) [12809]             38      M
## 28:    W  720 ILCS 570 402(c) [5101110]             22      M
## 29:    W 625 ILCS 5 11-501(a)(2) [11309             23      M
## 30:    W             720 ILCS 5/19-1(a)             19      M
## 31:    W                        UNKNOWN             36      M
## 32:    W            625 ILCS 5/6-303(a)             46      M
## 33:    W           625 ILCS 5/11-501(a)             25      M
## 34:    W             720 ILCS 5/19-1(a)             19      M
## 35:    W           625 ILCS 5/11-501(a)             21      F
## 36:    W           720 ILCS 5/9-1(a)(1)             23      M
## 37:    W                38-24-3.1(a)(6)             20      M
## 38:    W           625 ILCS 5/11-501(a)             44      M
## 39:    W               720 ILCS 570/402             18      M
## 40:    W              720 ILCS 550/5(g)             57      M
## 41:    W       720 ILCS 5/12-14.1(a)(1)             29      M
## 42:    W           720 ILCS 5/12-4.3(a)             19      M
## 43:    W              720 ILCS 5/24-1.1             17      M
## 44:    W                  38-10-2(a)(3)             51      M
##     race               charges_citation age_at_booking gender
##            booking_date      jail_id     bail_status housing_location
##  1: 2013-01-19 00:00:00 2013-0119222              NA         08-2N-DR
##  2: 2013-01-18 00:00:00 2013-0118220              NA      11-AH-3-411
##  3: 2013-01-18 00:00:00 2013-0118212              NA              02-
##  4: 2013-01-18 00:00:00 2013-0118203              NA      02-D4-MU-1-
##  5: 2013-01-18 00:00:00 2013-0118056              NA       05-D-1-2-1
##  6: 2013-01-18 00:00:00 2013-0118051              NA      02-D1-H-3-H
##  7: 2013-01-13 00:00:00 2013-0113043         NO BOND       03-A-2-4-2
##  8: 2013-01-11 00:00:00 2013-0111097              NA      11-AF-3-311
##  9: 2013-01-04 00:00:00 2013-0104171         NO BOND       01-B-2-3-1
## 10: 2013-01-03 00:00:00 2013-0103001              NA      02-D3-HH-3-
## 11: 2012-12-28 00:00:00 2012-1228050         NO BOND      03-AX-D3-1-
## 12: 2012-12-27 00:00:00 2012-1227035              NA      02-D4-QU-1-
## 13: 2012-12-22 12:28:29 2012-1217166              NA      03-AX-B1-1-
## 14: 2012-12-07 00:00:00 2012-1207055         NO BOND       03-A-3-4-1
## 15: 2012-12-03 00:00:00 2012-1203014         NO BOND      02-D4-RL-1-
## 16: 2012-12-02 00:00:00 2012-1202178         NO BOND       10-A-3-8-2
## 17: 2012-12-22 11:37:58 2012-1121001              NA      02-D1-A-2-A
## 18: 2012-12-15 22:40:08 2012-1117082 Bond in Process      02-D2-W-2-W
## 19: 2012-11-16 00:00:00 2012-1116207              NA      02-D4-ML-1-
## 20: 2012-11-02 00:00:00 2012-1102164              NA      14-B4-4-42-
## 21: 2012-10-31 00:00:00 2012-1031214         NO BOND      04-J-1-11-1
## 22: 2012-10-30 00:00:00 2012-1030177         NO BOND      02-D1-A-2-A
## 23: 2012-10-29 00:00:00 2012-1029031         NO BOND      03-A-2-19-1
## 24: 2012-12-18 02:02:39 2012-1017203              NA      02-D2-U-3-U
## 25: 2012-12-15 03:10:58 2012-1011190              NA          17-SFFP
## 26: 2012-10-10 00:00:00 2012-1010016         NO BOND      03-A-1-26-1
## 27: 2012-09-20 00:00:00 2012-0920237         NO BOND      06-H-2-19-2
## 28: 2012-12-22 12:23:40 2012-0909136              NA      11-AC-1-208
## 29: 2012-09-04 00:00:00 2012-0904227 Bond in Process            15-EM
## 30: 2012-08-31 00:00:00 2012-0831240         NO BOND       01-E-1-1-1
## 31: 2012-08-29 00:00:00 2012-0829215              NA      01-G-3-16-1
## 32: 2012-08-08 00:00:00 2012-0808183         NO BOND            15-EM
## 33: 2012-07-10 00:00:00 2012-0710111              NA       06-D-1-3-2
## 34: 2012-07-08 00:00:00 2012-0708142         NO BOND            15-EM
## 35: 2012-07-02 00:00:00 2012-0702163              NA      17-WR-N-C-2
## 36: 2012-06-22 00:00:00 2012-0622226         NO BOND       01-A-3-3-2
## 37: 2012-06-16 00:00:00 2012-0616119              NA      06-H-1-12-2
## 38: 2012-05-09 00:00:00 2012-0509091              NA      02-D1-H-3-H
## 39: 2012-03-20 00:00:00 2012-0320179 Bond in Process            15-DR
## 40: 2011-12-14 00:00:00 2011-1214202              NA       10-C-1-7-1
## 41: 2011-11-16 00:00:00 2011-1116167         NO BOND          C DISCH
## 42: 2011-10-07 00:00:00 2011-1007202              NA      11-DH-3-411
## 43: 2010-09-09 00:00:00 2010-0909032              NA      06-C-1-13-1
## 44: 2010-04-19 00:00:00 2010-0419201              NA      01-H-1-13-2
##            booking_date      jail_id     bail_status housing_location
##                                              charges bail_amount
##  1:                                               NA       50000
##  2:                         DRVG ON SUSP LICENSE/FTA       10000
##  3:                                               NA       10000
##  4:                         DRVG ON SUSP LICENSE/FTA        3000
##  5:                        CRIM SEX ASSAULT/FAMILIES       50000
##  6:                           VIOLATION OF PROBATION      100000
##  7:                                 BATTERY/DOMESTIC        5000
##  8:                     MFG/DEL 900+ GR COCAINE/ANLG        5000
##  9:                        AGG BATTERY/PEACE OFFICER          NA
## 10:                     DOMESTIC BATTERY/BODILY HARM        5000
## 11:                                        MFG/DEL 1          NA
## 12:                            RET THEFT/DISP MERCH/        5000
## 13:                   POSS AMT CON SUB EXCEPT(A)/(D)        5000
## 14:                           VIOLATION OF PROBATION          NA
## 15:                       KNOW DAMAGE PROP >$300-10K          NA
## 16:                   BATTERY/MAKES PHYSICAL CONTACT       50000
## 17:                   DRIVING REVOKED/SUSPENDED 2ND+      500000
## 18:                       OBSTRUCTING IDENTIFICATION          NA
## 19:                                          BATTERY       25000
## 20:                   POSS AMT CON SUB EXCEPT(A)/(D)       75000
## 21:                    THEFT/LABOR/SERVICES/PROPERTY          NA
## 22:                                         BURGLARY          NA
## 23:                       OBSTRUCTING IDENTIFICATION          NA
## 24:                   DOMESTIC BTRY/PHYSICAL CONTACT       20000
## 25:                                           DUI/6+      200000
## 26:                   POSS AMT CON SUB EXCEPT(A)/(D)       50000
## 27:                    DUI/BAC .16 OR CHILD PASS/1ST          NA
## 28:                   POSS AMT CON SUB EXCEPT(A)/(D)       40000
## 29:                    DUI ALCOHOL W/CHILD PASSENGER          NA
## 30:              BURGLARY SCHOOL OR PLACE OF WORSHIP       10000
## 31:                         UNKNOWN OR NOT AVAILABLE      500000
## 32:                                               NA          NA
## 33:                        DUI/INTOXICATING COMPOUND     1000000
## 34:                                               NA          NA
## 35:                        DUI/INTOXICATING COMPOUND       95000
## 36:     FIRST DEGREE MURDER:INTENDS DEATH/GREAT HARM          NA
## 37: UNLAWFUL POSSESSION OF FIREARM:EXPLOSIVE BULLETS        5000
## 38:                        DUI/INTOXICATING COMPOUND       35000
## 39:      ILLEGAL POSSESSION OF CONTROLLED SUBSTANCES          NA
## 40:                    MAN/DEL CANNABIS/>5,000 GRAMS      500000
## 41:                   PREDATORY CRIMINAL SEXUAL ASLT      750000
## 42:  AGGRAVATED BATTERY OF A CHILD:FIREARM/DISC/HARM      200000
## 43: UNLAWFUL USE/POSSESSION OF WEAPON:FELON PRISONER      100000
## 44:                AGGRAVATED KIDNAPING:INFLICT HARM     5000000
##                                              charges bail_amount
##     discharge_date_earliest
##  1:                    <NA>
##  2:                    <NA>
##  3:     2013-01-19 06:32:06
##  4:                    <NA>
##  5:                    <NA>
##  6:                    <NA>
##  7:                    <NA>
##  8:                    <NA>
##  9:                    <NA>
## 10:     2013-01-14 06:27:53
## 11:     2012-12-30 18:25:17
## 12:                    <NA>
## 13:     2012-12-23 11:06:55
## 14:                    <NA>
## 15:                    <NA>
## 16:     2013-01-09 05:04:27
## 17:     2012-12-23 10:27:10
## 18:     2012-12-15 01:49:02
## 19:     2013-01-05 06:48:04
## 20:                    <NA>
## 21:     2012-12-19 23:27:34
## 22:                    <NA>
## 23:     2013-01-14 06:43:50
## 24:     2012-12-17 01:06:47
## 25:     2012-12-14 02:44:33
## 26:     2013-01-11 05:07:45
## 27:                    <NA>
## 28:     2012-12-23 11:01:37
## 29:     2012-12-19 23:27:30
## 30:                    <NA>
## 31:                    <NA>
## 32:     2013-01-06 06:41:44
## 33:                    <NA>
## 34:                    <NA>
## 35:                    <NA>
## 36:     2012-12-17 00:10:14
## 37:     2013-01-13 07:18:08
## 38:     2013-01-19 07:20:07
## 39:                    <NA>
## 40:                    <NA>
## 41:     2012-12-17 22:35:19
## 42:                    <NA>
## 43:                    <NA>
## 44:                    <NA>
##     discharge_date_earliest
```

```r
dat[J(c("W", "WH"))]
```

```
##       race               charges_citation age_at_booking gender
##    1:    W 720 ILCS 5 12-3.4(a)(1) [16128             34      M
##    2:    W    625 ILCS 5 6-303(a) [13526]             34      M
##    3:    W   625 ILCS 5 11-501(a) [14041]             34      M
##    4:    W    625 ILCS 5 6-303(a) [13526]             33      M
##    5:    W 720 ILCS 5 12-13(a)(3) [995700             39      M
##   ---                                                          
## 2055:   WH            720 ILCS 5/32-10(a)             50      M
## 2056:   WH                         38-9-1             18      M
## 2057:   WH                        38-19-3             32      M
## 2058:   WH                       56.5-704             26      M
## 2059:   WH                    95.5-11-501             31      M
##       booking_date      jail_id bail_status housing_location
##    1:   2013-01-19 2013-0119222          NA         08-2N-DR
##    2:   2013-01-18 2013-0118220          NA      11-AH-3-411
##    3:   2013-01-18 2013-0118212          NA              02-
##    4:   2013-01-18 2013-0118203          NA      02-D4-MU-1-
##    5:   2013-01-18 2013-0118056          NA       05-D-1-2-1
##   ---                                                       
## 2055:   1996-12-18 1996-9683431     NO BOND       01-H-1-6-1
## 2056:   1996-10-03 1996-9664677          NA          15-EMAW
## 2057:   1996-07-12 1996-9644229          NA          15-EMAW
## 2058:   1995-07-27 1995-9551250          NA          15-EMAW
## 2059:   1995-05-09 1995-9532061     NO BOND          15-DRAW
##                            charges bail_amount discharge_date_earliest
##    1:                           NA       50000                    <NA>
##    2:     DRVG ON SUSP LICENSE/FTA       10000                    <NA>
##    3:                           NA       10000     2013-01-19 06:32:06
##    4:     DRVG ON SUSP LICENSE/FTA        3000                    <NA>
##    5:    CRIM SEX ASSAULT/FAMILIES       50000                    <NA>
##   ---                                                                 
## 2055: VIO BAIL BOND/CLASS M CONVIC          NA                    <NA>
## 2056:                           NA      100000                    <NA>
## 2057:                           NA       15000                    <NA>
## 2058:                           NA       85000                    <NA>
## 2059:                           NA          NA                    <NA>
```

```r

setkeyv(dat, c("race", "gender"))
dat[c("W", "WH")]
```

```
##       race               charges_citation age_at_booking gender
##    1:    W   720 ILCS 5 16-3(a) [1025000]             27      F
##    2:    W   625 ILCS 5 11-501(a) [14039]             24      F
##    3:    W           625 ILCS 5/11-501(a)             21      F
##    4:    W 720 ILCS 5 12-3.4(a)(1) [16128             34      M
##    5:    W    625 ILCS 5 6-303(a) [13526]             34      M
##   ---                                                          
## 2055:   WH            720 ILCS 5/32-10(a)             50      M
## 2056:   WH                         38-9-1             18      M
## 2057:   WH                        38-19-3             32      M
## 2058:   WH                       56.5-704             26      M
## 2059:   WH                    95.5-11-501             31      M
##              booking_date      jail_id bail_status housing_location
##    1: 2012-10-31 00:00:00 2012-1031214     NO BOND      04-J-1-11-1
##    2: 2012-12-15 03:10:58 2012-1011190          NA          17-SFFP
##    3: 2012-07-02 00:00:00 2012-0702163          NA      17-WR-N-C-2
##    4: 2013-01-19 00:00:00 2013-0119222          NA         08-2N-DR
##    5: 2013-01-18 00:00:00 2013-0118220          NA      11-AH-3-411
##   ---                                                              
## 2055: 1996-12-18 00:00:00 1996-9683431     NO BOND       01-H-1-6-1
## 2056: 1996-10-03 00:00:00 1996-9664677          NA          15-EMAW
## 2057: 1996-07-12 00:00:00 1996-9644229          NA          15-EMAW
## 2058: 1995-07-27 00:00:00 1995-9551250          NA          15-EMAW
## 2059: 1995-05-09 00:00:00 1995-9532061     NO BOND          15-DRAW
##                             charges bail_amount discharge_date_earliest
##    1: THEFT/LABOR/SERVICES/PROPERTY          NA     2012-12-19 23:27:34
##    2:                        DUI/6+      200000     2012-12-14 02:44:33
##    3:     DUI/INTOXICATING COMPOUND       95000                    <NA>
##    4:                            NA       50000                    <NA>
##    5:      DRVG ON SUSP LICENSE/FTA       10000                    <NA>
##   ---                                                                  
## 2055:  VIO BAIL BOND/CLASS M CONVIC          NA                    <NA>
## 2056:                            NA      100000                    <NA>
## 2057:                            NA       15000                    <NA>
## 2058:                            NA       85000                    <NA>
## 2059:                            NA          NA                    <NA>
```

```r
dat[c("W", "M")]
```

```
##     race               charges_citation age_at_booking gender
##  1:    W   720 ILCS 5 16-3(a) [1025000]             27      F
##  2:    W   625 ILCS 5 11-501(a) [14039]             24      F
##  3:    W           625 ILCS 5/11-501(a)             21      F
##  4:    W 720 ILCS 5 12-3.4(a)(1) [16128             34      M
##  5:    W    625 ILCS 5 6-303(a) [13526]             34      M
##  6:    W   625 ILCS 5 11-501(a) [14041]             34      M
##  7:    W    625 ILCS 5 6-303(a) [13526]             33      M
##  8:    W 720 ILCS 5 12-13(a)(3) [995700             39      M
##  9:    W                            000             26      M
## 10:    W     720 ILCS 5 12-3.2 [930200]             25      M
## 11:    W 720 ILCS 570 401(a)(2)(D) [509             35      M
## 12:    W 720 ILCS 5 12-3.05(d)(4) [1610             20      M
## 13:    W 720 ILCS 5 12-3.2(a)(1) [10416             43      M
## 14:    W 720 ILCS 570 401(c)(1) [13009]             21      M
## 15:    W    720 ILCS 5 16A-3(a) [15599]             23      M
## 16:    W  720 ILCS 570 402(c) [5101110]             42      M
## 17:    W                            000             19      M
## 18:    W    720 ILCS 5 12-1(a) [920000]             33      M
## 19:    W  720 ILCS 5 12-3(a)(2) [10530]             25      M
## 20:    W  625 ILCS 5 6-303(d) [5883000]             33      M
## 21:    W  720 ILCS 5 16A-3(a) [1060000]             20      M
## 22:    W       720 ILCS 5 12-3 [930000]             19      M
## 23:    W  720 ILCS 570 402(c) [5101110]             53      M
## 24:    W   720 ILCS 5 19-1(a) [1110000]             21      M
## 25:    W 720 ILCS 5 12-3.2(a)(1) [10416             36      M
## 26:    W 720 ILCS 5 12-3.2(a)(2) [10418             26      M
## 27:    W  720 ILCS 570 402(c) [5101110]             43      M
## 28:    W   625 ILCS 5 11-501(a) [12809]             38      M
## 29:    W  720 ILCS 570 402(c) [5101110]             22      M
## 30:    W 625 ILCS 5 11-501(a)(2) [11309             23      M
## 31:    W             720 ILCS 5/19-1(a)             19      M
## 32:    W                        UNKNOWN             36      M
## 33:    W            625 ILCS 5/6-303(a)             46      M
## 34:    W           625 ILCS 5/11-501(a)             25      M
## 35:    W             720 ILCS 5/19-1(a)             19      M
## 36:    W           720 ILCS 5/9-1(a)(1)             23      M
## 37:    W                38-24-3.1(a)(6)             20      M
## 38:    W           625 ILCS 5/11-501(a)             44      M
## 39:    W               720 ILCS 570/402             18      M
## 40:    W              720 ILCS 550/5(g)             57      M
## 41:    W       720 ILCS 5/12-14.1(a)(1)             29      M
## 42:    W           720 ILCS 5/12-4.3(a)             19      M
## 43:    W              720 ILCS 5/24-1.1             17      M
## 44:    W                  38-10-2(a)(3)             51      M
## 45:    M                             NA             NA     NA
##     race               charges_citation age_at_booking gender
##            booking_date      jail_id     bail_status housing_location
##  1: 2012-10-31 00:00:00 2012-1031214         NO BOND      04-J-1-11-1
##  2: 2012-12-15 03:10:58 2012-1011190              NA          17-SFFP
##  3: 2012-07-02 00:00:00 2012-0702163              NA      17-WR-N-C-2
##  4: 2013-01-19 00:00:00 2013-0119222              NA         08-2N-DR
##  5: 2013-01-18 00:00:00 2013-0118220              NA      11-AH-3-411
##  6: 2013-01-18 00:00:00 2013-0118212              NA              02-
##  7: 2013-01-18 00:00:00 2013-0118203              NA      02-D4-MU-1-
##  8: 2013-01-18 00:00:00 2013-0118056              NA       05-D-1-2-1
##  9: 2013-01-18 00:00:00 2013-0118051              NA      02-D1-H-3-H
## 10: 2013-01-13 00:00:00 2013-0113043         NO BOND       03-A-2-4-2
## 11: 2013-01-11 00:00:00 2013-0111097              NA      11-AF-3-311
## 12: 2013-01-04 00:00:00 2013-0104171         NO BOND       01-B-2-3-1
## 13: 2013-01-03 00:00:00 2013-0103001              NA      02-D3-HH-3-
## 14: 2012-12-28 00:00:00 2012-1228050         NO BOND      03-AX-D3-1-
## 15: 2012-12-27 00:00:00 2012-1227035              NA      02-D4-QU-1-
## 16: 2012-12-22 12:28:29 2012-1217166              NA      03-AX-B1-1-
## 17: 2012-12-07 00:00:00 2012-1207055         NO BOND       03-A-3-4-1
## 18: 2012-12-03 00:00:00 2012-1203014         NO BOND      02-D4-RL-1-
## 19: 2012-12-02 00:00:00 2012-1202178         NO BOND       10-A-3-8-2
## 20: 2012-12-22 11:37:58 2012-1121001              NA      02-D1-A-2-A
## 21: 2012-12-15 22:40:08 2012-1117082 Bond in Process      02-D2-W-2-W
## 22: 2012-11-16 00:00:00 2012-1116207              NA      02-D4-ML-1-
## 23: 2012-11-02 00:00:00 2012-1102164              NA      14-B4-4-42-
## 24: 2012-10-30 00:00:00 2012-1030177         NO BOND      02-D1-A-2-A
## 25: 2012-10-29 00:00:00 2012-1029031         NO BOND      03-A-2-19-1
## 26: 2012-12-18 02:02:39 2012-1017203              NA      02-D2-U-3-U
## 27: 2012-10-10 00:00:00 2012-1010016         NO BOND      03-A-1-26-1
## 28: 2012-09-20 00:00:00 2012-0920237         NO BOND      06-H-2-19-2
## 29: 2012-12-22 12:23:40 2012-0909136              NA      11-AC-1-208
## 30: 2012-09-04 00:00:00 2012-0904227 Bond in Process            15-EM
## 31: 2012-08-31 00:00:00 2012-0831240         NO BOND       01-E-1-1-1
## 32: 2012-08-29 00:00:00 2012-0829215              NA      01-G-3-16-1
## 33: 2012-08-08 00:00:00 2012-0808183         NO BOND            15-EM
## 34: 2012-07-10 00:00:00 2012-0710111              NA       06-D-1-3-2
## 35: 2012-07-08 00:00:00 2012-0708142         NO BOND            15-EM
## 36: 2012-06-22 00:00:00 2012-0622226         NO BOND       01-A-3-3-2
## 37: 2012-06-16 00:00:00 2012-0616119              NA      06-H-1-12-2
## 38: 2012-05-09 00:00:00 2012-0509091              NA      02-D1-H-3-H
## 39: 2012-03-20 00:00:00 2012-0320179 Bond in Process            15-DR
## 40: 2011-12-14 00:00:00 2011-1214202              NA       10-C-1-7-1
## 41: 2011-11-16 00:00:00 2011-1116167         NO BOND          C DISCH
## 42: 2011-10-07 00:00:00 2011-1007202              NA      11-DH-3-411
## 43: 2010-09-09 00:00:00 2010-0909032              NA      06-C-1-13-1
## 44: 2010-04-19 00:00:00 2010-0419201              NA      01-H-1-13-2
## 45:                <NA>           NA              NA               NA
##            booking_date      jail_id     bail_status housing_location
##                                              charges bail_amount
##  1:                    THEFT/LABOR/SERVICES/PROPERTY          NA
##  2:                                           DUI/6+      200000
##  3:                        DUI/INTOXICATING COMPOUND       95000
##  4:                                               NA       50000
##  5:                         DRVG ON SUSP LICENSE/FTA       10000
##  6:                                               NA       10000
##  7:                         DRVG ON SUSP LICENSE/FTA        3000
##  8:                        CRIM SEX ASSAULT/FAMILIES       50000
##  9:                           VIOLATION OF PROBATION      100000
## 10:                                 BATTERY/DOMESTIC        5000
## 11:                     MFG/DEL 900+ GR COCAINE/ANLG        5000
## 12:                        AGG BATTERY/PEACE OFFICER          NA
## 13:                     DOMESTIC BATTERY/BODILY HARM        5000
## 14:                                        MFG/DEL 1          NA
## 15:                            RET THEFT/DISP MERCH/        5000
## 16:                   POSS AMT CON SUB EXCEPT(A)/(D)        5000
## 17:                           VIOLATION OF PROBATION          NA
## 18:                       KNOW DAMAGE PROP >$300-10K          NA
## 19:                   BATTERY/MAKES PHYSICAL CONTACT       50000
## 20:                   DRIVING REVOKED/SUSPENDED 2ND+      500000
## 21:                       OBSTRUCTING IDENTIFICATION          NA
## 22:                                          BATTERY       25000
## 23:                   POSS AMT CON SUB EXCEPT(A)/(D)       75000
## 24:                                         BURGLARY          NA
## 25:                       OBSTRUCTING IDENTIFICATION          NA
## 26:                   DOMESTIC BTRY/PHYSICAL CONTACT       20000
## 27:                   POSS AMT CON SUB EXCEPT(A)/(D)       50000
## 28:                    DUI/BAC .16 OR CHILD PASS/1ST          NA
## 29:                   POSS AMT CON SUB EXCEPT(A)/(D)       40000
## 30:                    DUI ALCOHOL W/CHILD PASSENGER          NA
## 31:              BURGLARY SCHOOL OR PLACE OF WORSHIP       10000
## 32:                         UNKNOWN OR NOT AVAILABLE      500000
## 33:                                               NA          NA
## 34:                        DUI/INTOXICATING COMPOUND     1000000
## 35:                                               NA          NA
## 36:     FIRST DEGREE MURDER:INTENDS DEATH/GREAT HARM          NA
## 37: UNLAWFUL POSSESSION OF FIREARM:EXPLOSIVE BULLETS        5000
## 38:                        DUI/INTOXICATING COMPOUND       35000
## 39:      ILLEGAL POSSESSION OF CONTROLLED SUBSTANCES          NA
## 40:                    MAN/DEL CANNABIS/>5,000 GRAMS      500000
## 41:                   PREDATORY CRIMINAL SEXUAL ASLT      750000
## 42:  AGGRAVATED BATTERY OF A CHILD:FIREARM/DISC/HARM      200000
## 43: UNLAWFUL USE/POSSESSION OF WEAPON:FELON PRISONER      100000
## 44:                AGGRAVATED KIDNAPING:INFLICT HARM     5000000
## 45:                                               NA          NA
##                                              charges bail_amount
##     discharge_date_earliest
##  1:     2012-12-19 23:27:34
##  2:     2012-12-14 02:44:33
##  3:                    <NA>
##  4:                    <NA>
##  5:                    <NA>
##  6:     2013-01-19 06:32:06
##  7:                    <NA>
##  8:                    <NA>
##  9:                    <NA>
## 10:                    <NA>
## 11:                    <NA>
## 12:                    <NA>
## 13:     2013-01-14 06:27:53
## 14:     2012-12-30 18:25:17
## 15:                    <NA>
## 16:     2012-12-23 11:06:55
## 17:                    <NA>
## 18:                    <NA>
## 19:     2013-01-09 05:04:27
## 20:     2012-12-23 10:27:10
## 21:     2012-12-15 01:49:02
## 22:     2013-01-05 06:48:04
## 23:                    <NA>
## 24:                    <NA>
## 25:     2013-01-14 06:43:50
## 26:     2012-12-17 01:06:47
## 27:     2013-01-11 05:07:45
## 28:                    <NA>
## 29:     2012-12-23 11:01:37
## 30:     2012-12-19 23:27:30
## 31:                    <NA>
## 32:                    <NA>
## 33:     2013-01-06 06:41:44
## 34:                    <NA>
## 35:                    <NA>
## 36:     2012-12-17 00:10:14
## 37:     2013-01-13 07:18:08
## 38:     2013-01-19 07:20:07
## 39:                    <NA>
## 40:                    <NA>
## 41:     2012-12-17 22:35:19
## 42:                    <NA>
## 43:                    <NA>
## 44:                    <NA>
## 45:                    <NA>
##     discharge_date_earliest
```

```r
dat[data.table("WH", "M")]
```

```
##       race gender               charges_citation age_at_booking
##    1:   WH      M 720 ILCS 5 12-3.4(a)(2) [16145             26
##    2:   WH      M     720 ILCS 5 12-3.2 [930200]             52
##    3:   WH      M 720 ILCS 5 12-3.2(a)(1) [10416             39
##    4:   WH      M 720 ILCS 5 12-3.2(a)(1) [10416             41
##    5:   WH      M        720 ILCS 5 17-3 [11968]             25
##   ---                                                          
## 1678:   WH      M            720 ILCS 5/32-10(a)             50
## 1679:   WH      M                         38-9-1             18
## 1680:   WH      M                        38-19-3             32
## 1681:   WH      M                       56.5-704             26
## 1682:   WH      M                    95.5-11-501             31
##       booking_date      jail_id bail_status housing_location
##    1:   2013-01-20 2013-0120171          NA              05-
##    2:   2013-01-20 2013-0120151          NA       05-L-2-1-2
##    3:   2013-01-20 2013-0120145          NA         08-2N-DR
##    4:   2013-01-20 2013-0120123          NA       05-E-2-3-2
##    5:   2013-01-20 2013-0120094          NA      11-BB-1-210
##   ---                                                       
## 1678:   1996-12-18 1996-9683431     NO BOND       01-H-1-6-1
## 1679:   1996-10-03 1996-9664677          NA          15-EMAW
## 1680:   1996-07-12 1996-9644229          NA          15-EMAW
## 1681:   1995-07-27 1995-9551250          NA          15-EMAW
## 1682:   1995-05-09 1995-9532061     NO BOND          15-DRAW
##                            charges bail_amount discharge_date_earliest
##    1:                           NA        5000                    <NA>
##    2:                           NA        5000                    <NA>
##    3:                           NA       25000                    <NA>
##    4:                           NA       10000                    <NA>
##    5:                           NA       10000                    <NA>
##   ---                                                                 
## 1678: VIO BAIL BOND/CLASS M CONVIC          NA                    <NA>
## 1679:                           NA      100000                    <NA>
## 1680:                           NA       15000                    <NA>
## 1681:                           NA       85000                    <NA>
## 1682:                           NA          NA                    <NA>
```

```r

dat[J("WH", "M")]
```

```
##       race gender               charges_citation age_at_booking
##    1:   WH      M 720 ILCS 5 12-3.4(a)(2) [16145             26
##    2:   WH      M     720 ILCS 5 12-3.2 [930200]             52
##    3:   WH      M 720 ILCS 5 12-3.2(a)(1) [10416             39
##    4:   WH      M 720 ILCS 5 12-3.2(a)(1) [10416             41
##    5:   WH      M        720 ILCS 5 17-3 [11968]             25
##   ---                                                          
## 1678:   WH      M            720 ILCS 5/32-10(a)             50
## 1679:   WH      M                         38-9-1             18
## 1680:   WH      M                        38-19-3             32
## 1681:   WH      M                       56.5-704             26
## 1682:   WH      M                    95.5-11-501             31
##       booking_date      jail_id bail_status housing_location
##    1:   2013-01-20 2013-0120171          NA              05-
##    2:   2013-01-20 2013-0120151          NA       05-L-2-1-2
##    3:   2013-01-20 2013-0120145          NA         08-2N-DR
##    4:   2013-01-20 2013-0120123          NA       05-E-2-3-2
##    5:   2013-01-20 2013-0120094          NA      11-BB-1-210
##   ---                                                       
## 1678:   1996-12-18 1996-9683431     NO BOND       01-H-1-6-1
## 1679:   1996-10-03 1996-9664677          NA          15-EMAW
## 1680:   1996-07-12 1996-9644229          NA          15-EMAW
## 1681:   1995-07-27 1995-9551250          NA          15-EMAW
## 1682:   1995-05-09 1995-9532061     NO BOND          15-DRAW
##                            charges bail_amount discharge_date_earliest
##    1:                           NA        5000                    <NA>
##    2:                           NA        5000                    <NA>
##    3:                           NA       25000                    <NA>
##    4:                           NA       10000                    <NA>
##    5:                           NA       10000                    <NA>
##   ---                                                                 
## 1678: VIO BAIL BOND/CLASS M CONVIC          NA                    <NA>
## 1679:                           NA      100000                    <NA>
## 1680:                           NA       15000                    <NA>
## 1681:                           NA       85000                    <NA>
## 1682:                           NA          NA                    <NA>
```

```r
dat[CJ("WH", "M")]
```

```
##       race gender               charges_citation age_at_booking
##    1:   WH      M 720 ILCS 5 12-3.4(a)(2) [16145             26
##    2:   WH      M     720 ILCS 5 12-3.2 [930200]             52
##    3:   WH      M 720 ILCS 5 12-3.2(a)(1) [10416             39
##    4:   WH      M 720 ILCS 5 12-3.2(a)(1) [10416             41
##    5:   WH      M        720 ILCS 5 17-3 [11968]             25
##   ---                                                          
## 1678:   WH      M            720 ILCS 5/32-10(a)             50
## 1679:   WH      M                         38-9-1             18
## 1680:   WH      M                        38-19-3             32
## 1681:   WH      M                       56.5-704             26
## 1682:   WH      M                    95.5-11-501             31
##       booking_date      jail_id bail_status housing_location
##    1:   2013-01-20 2013-0120171          NA              05-
##    2:   2013-01-20 2013-0120151          NA       05-L-2-1-2
##    3:   2013-01-20 2013-0120145          NA         08-2N-DR
##    4:   2013-01-20 2013-0120123          NA       05-E-2-3-2
##    5:   2013-01-20 2013-0120094          NA      11-BB-1-210
##   ---                                                       
## 1678:   1996-12-18 1996-9683431     NO BOND       01-H-1-6-1
## 1679:   1996-10-03 1996-9664677          NA          15-EMAW
## 1680:   1996-07-12 1996-9644229          NA          15-EMAW
## 1681:   1995-07-27 1995-9551250          NA          15-EMAW
## 1682:   1995-05-09 1995-9532061     NO BOND          15-DRAW
##                            charges bail_amount discharge_date_earliest
##    1:                           NA        5000                    <NA>
##    2:                           NA        5000                    <NA>
##    3:                           NA       25000                    <NA>
##    4:                           NA       10000                    <NA>
##    5:                           NA       10000                    <NA>
##   ---                                                                 
## 1678: VIO BAIL BOND/CLASS M CONVIC          NA                    <NA>
## 1679:                           NA      100000                    <NA>
## 1680:                           NA       15000                    <NA>
## 1681:                           NA       85000                    <NA>
## 1682:                           NA          NA                    <NA>
```

```r

dat[J(c("W", "WH"))]
```

```
##       race               charges_citation age_at_booking gender
##    1:    W   720 ILCS 5 16-3(a) [1025000]             27      F
##    2:    W   625 ILCS 5 11-501(a) [14039]             24      F
##    3:    W           625 ILCS 5/11-501(a)             21      F
##    4:    W 720 ILCS 5 12-3.4(a)(1) [16128             34      M
##    5:    W    625 ILCS 5 6-303(a) [13526]             34      M
##   ---                                                          
## 2055:   WH            720 ILCS 5/32-10(a)             50      M
## 2056:   WH                         38-9-1             18      M
## 2057:   WH                        38-19-3             32      M
## 2058:   WH                       56.5-704             26      M
## 2059:   WH                    95.5-11-501             31      M
##              booking_date      jail_id bail_status housing_location
##    1: 2012-10-31 00:00:00 2012-1031214     NO BOND      04-J-1-11-1
##    2: 2012-12-15 03:10:58 2012-1011190          NA          17-SFFP
##    3: 2012-07-02 00:00:00 2012-0702163          NA      17-WR-N-C-2
##    4: 2013-01-19 00:00:00 2013-0119222          NA         08-2N-DR
##    5: 2013-01-18 00:00:00 2013-0118220          NA      11-AH-3-411
##   ---                                                              
## 2055: 1996-12-18 00:00:00 1996-9683431     NO BOND       01-H-1-6-1
## 2056: 1996-10-03 00:00:00 1996-9664677          NA          15-EMAW
## 2057: 1996-07-12 00:00:00 1996-9644229          NA          15-EMAW
## 2058: 1995-07-27 00:00:00 1995-9551250          NA          15-EMAW
## 2059: 1995-05-09 00:00:00 1995-9532061     NO BOND          15-DRAW
##                             charges bail_amount discharge_date_earliest
##    1: THEFT/LABOR/SERVICES/PROPERTY          NA     2012-12-19 23:27:34
##    2:                        DUI/6+      200000     2012-12-14 02:44:33
##    3:     DUI/INTOXICATING COMPOUND       95000                    <NA>
##    4:                            NA       50000                    <NA>
##    5:      DRVG ON SUSP LICENSE/FTA       10000                    <NA>
##   ---                                                                  
## 2055:  VIO BAIL BOND/CLASS M CONVIC          NA                    <NA>
## 2056:                            NA      100000                    <NA>
## 2057:                            NA       15000                    <NA>
## 2058:                            NA       85000                    <NA>
## 2059:                            NA          NA                    <NA>
```

```r
dat[CJ("WH", "M")]
```

```
##       race gender               charges_citation age_at_booking
##    1:   WH      M 720 ILCS 5 12-3.4(a)(2) [16145             26
##    2:   WH      M     720 ILCS 5 12-3.2 [930200]             52
##    3:   WH      M 720 ILCS 5 12-3.2(a)(1) [10416             39
##    4:   WH      M 720 ILCS 5 12-3.2(a)(1) [10416             41
##    5:   WH      M        720 ILCS 5 17-3 [11968]             25
##   ---                                                          
## 1678:   WH      M            720 ILCS 5/32-10(a)             50
## 1679:   WH      M                         38-9-1             18
## 1680:   WH      M                        38-19-3             32
## 1681:   WH      M                       56.5-704             26
## 1682:   WH      M                    95.5-11-501             31
##       booking_date      jail_id bail_status housing_location
##    1:   2013-01-20 2013-0120171          NA              05-
##    2:   2013-01-20 2013-0120151          NA       05-L-2-1-2
##    3:   2013-01-20 2013-0120145          NA         08-2N-DR
##    4:   2013-01-20 2013-0120123          NA       05-E-2-3-2
##    5:   2013-01-20 2013-0120094          NA      11-BB-1-210
##   ---                                                       
## 1678:   1996-12-18 1996-9683431     NO BOND       01-H-1-6-1
## 1679:   1996-10-03 1996-9664677          NA          15-EMAW
## 1680:   1996-07-12 1996-9644229          NA          15-EMAW
## 1681:   1995-07-27 1995-9551250          NA          15-EMAW
## 1682:   1995-05-09 1995-9532061     NO BOND          15-DRAW
##                            charges bail_amount discharge_date_earliest
##    1:                           NA        5000                    <NA>
##    2:                           NA        5000                    <NA>
##    3:                           NA       25000                    <NA>
##    4:                           NA       10000                    <NA>
##    5:                           NA       10000                    <NA>
##   ---                                                                 
## 1678: VIO BAIL BOND/CLASS M CONVIC          NA                    <NA>
## 1679:                           NA      100000                    <NA>
## 1680:                           NA       15000                    <NA>
## 1681:                           NA       85000                    <NA>
## 1682:                           NA          NA                    <NA>
```

```r

dat[CJ(c("WH", "W"))]
```

```
##       race               charges_citation age_at_booking gender
##    1:    W   720 ILCS 5 16-3(a) [1025000]             27      F
##    2:    W   625 ILCS 5 11-501(a) [14039]             24      F
##    3:    W           625 ILCS 5/11-501(a)             21      F
##    4:    W 720 ILCS 5 12-3.4(a)(1) [16128             34      M
##    5:    W    625 ILCS 5 6-303(a) [13526]             34      M
##   ---                                                          
## 2055:   WH            720 ILCS 5/32-10(a)             50      M
## 2056:   WH                         38-9-1             18      M
## 2057:   WH                        38-19-3             32      M
## 2058:   WH                       56.5-704             26      M
## 2059:   WH                    95.5-11-501             31      M
##              booking_date      jail_id bail_status housing_location
##    1: 2012-10-31 00:00:00 2012-1031214     NO BOND      04-J-1-11-1
##    2: 2012-12-15 03:10:58 2012-1011190          NA          17-SFFP
##    3: 2012-07-02 00:00:00 2012-0702163          NA      17-WR-N-C-2
##    4: 2013-01-19 00:00:00 2013-0119222          NA         08-2N-DR
##    5: 2013-01-18 00:00:00 2013-0118220          NA      11-AH-3-411
##   ---                                                              
## 2055: 1996-12-18 00:00:00 1996-9683431     NO BOND       01-H-1-6-1
## 2056: 1996-10-03 00:00:00 1996-9664677          NA          15-EMAW
## 2057: 1996-07-12 00:00:00 1996-9644229          NA          15-EMAW
## 2058: 1995-07-27 00:00:00 1995-9551250          NA          15-EMAW
## 2059: 1995-05-09 00:00:00 1995-9532061     NO BOND          15-DRAW
##                             charges bail_amount discharge_date_earliest
##    1: THEFT/LABOR/SERVICES/PROPERTY          NA     2012-12-19 23:27:34
##    2:                        DUI/6+      200000     2012-12-14 02:44:33
##    3:     DUI/INTOXICATING COMPOUND       95000                    <NA>
##    4:                            NA       50000                    <NA>
##    5:      DRVG ON SUSP LICENSE/FTA       10000                    <NA>
##   ---                                                                  
## 2055:  VIO BAIL BOND/CLASS M CONVIC          NA                    <NA>
## 2056:                            NA      100000                    <NA>
## 2057:                            NA       15000                    <NA>
## 2058:                            NA       85000                    <NA>
## 2059:                            NA          NA                    <NA>
```

```r
dat[CJ(c("WH", "W"))]
```

```
##       race               charges_citation age_at_booking gender
##    1:    W   720 ILCS 5 16-3(a) [1025000]             27      F
##    2:    W   625 ILCS 5 11-501(a) [14039]             24      F
##    3:    W           625 ILCS 5/11-501(a)             21      F
##    4:    W 720 ILCS 5 12-3.4(a)(1) [16128             34      M
##    5:    W    625 ILCS 5 6-303(a) [13526]             34      M
##   ---                                                          
## 2055:   WH            720 ILCS 5/32-10(a)             50      M
## 2056:   WH                         38-9-1             18      M
## 2057:   WH                        38-19-3             32      M
## 2058:   WH                       56.5-704             26      M
## 2059:   WH                    95.5-11-501             31      M
##              booking_date      jail_id bail_status housing_location
##    1: 2012-10-31 00:00:00 2012-1031214     NO BOND      04-J-1-11-1
##    2: 2012-12-15 03:10:58 2012-1011190          NA          17-SFFP
##    3: 2012-07-02 00:00:00 2012-0702163          NA      17-WR-N-C-2
##    4: 2013-01-19 00:00:00 2013-0119222          NA         08-2N-DR
##    5: 2013-01-18 00:00:00 2013-0118220          NA      11-AH-3-411
##   ---                                                              
## 2055: 1996-12-18 00:00:00 1996-9683431     NO BOND       01-H-1-6-1
## 2056: 1996-10-03 00:00:00 1996-9664677          NA          15-EMAW
## 2057: 1996-07-12 00:00:00 1996-9644229          NA          15-EMAW
## 2058: 1995-07-27 00:00:00 1995-9551250          NA          15-EMAW
## 2059: 1995-05-09 00:00:00 1995-9532061     NO BOND          15-DRAW
##                             charges bail_amount discharge_date_earliest
##    1: THEFT/LABOR/SERVICES/PROPERTY          NA     2012-12-19 23:27:34
##    2:                        DUI/6+      200000     2012-12-14 02:44:33
##    3:     DUI/INTOXICATING COMPOUND       95000                    <NA>
##    4:                            NA       50000                    <NA>
##    5:      DRVG ON SUSP LICENSE/FTA       10000                    <NA>
##   ---                                                                  
## 2055:  VIO BAIL BOND/CLASS M CONVIC          NA                    <NA>
## 2056:                            NA      100000                    <NA>
## 2057:                            NA       15000                    <NA>
## 2058:                            NA       85000                    <NA>
## 2059:                            NA          NA                    <NA>
```





