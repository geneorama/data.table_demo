

# Data table examples using public data

By Gene Leynes
Chicago Data Science

## Introduction

This code was used to demonstrate the features of the data.table package at the R User Group meetup held in Chicago on February 7th, 2013.

The data and project related to this exmaple comes from another meetup and code project.  I attended that meetup, and put together some simple visualizations for inspiration.  These visualizations were intended to be used as talking points for the project.  I repurposed this data to provide some `data.table` examples.

Meetup group: http://www.meetup.com/The-Chicago-Data-Visualization-Group/events/97690642/ <br>
Github project related to meetup: https://github.com/sc3/26thandcalifornia <br>
My related project: https://github.com/geneorama/26_and_California <br>

Exmples complied using `knitr`: <br>
Original meetup example: http://chicagodatascience.com/public/26th_and_California_example_visualizations.html <br>


## INITIALIZE

```r
## The following line is needed to compile to HTML, please ignore
if (basename(getwd()) != "data.table_demo") {
    setwd("..")
}

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
## The following line is needed to compile to HTML, please ignore
if (basename(getwd()) != "data.table_demo") {
    setwd("..")
}

rawdat = read.table(file = "data/Database 2013-01-21 (8zQ4cW7T).csv", sep = ",", 
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



## Examples of grouping with data.table


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



## Examples of subsetting with data.table

This is probably the most confusing thing at first


```r
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

```


## Examples of grouping and aggregating with data.table

This is very fast, and very useful for generating any kind of summary statistics.


```r

## Grouping is simple
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


## Examples of grouping and aggregating with data.table

But look at what happens here!

Sometimes things happen that you may not expect. It's good, but possibly a surprise. By including `age_at_booking` without any aggregating function, it automatically expands the data.table result


```r
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


### Complex query example


```r
mysummary = dat[
	i = !is.na(charges) &
		!is.na(booking_date) &
		!is.na(bail_amount),
	j = list(
		count = .N,
		coverage = diff(range(booking_date)),
		bailave = mean(bail_amount),
		bailsd = sd(bail_amount),
		bailmin = min(bail_amount),
		bailmax = max(bail_amount)
		),
	by = list(race,gender,age_at_booking)
	]
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

## Uncomment to open summary as csv file, in Excel probably
# wtf(mysummary)
```


***
<br>
<br>
## NOTE: FOR THE REST OF THE EXAMPLES I'M GOING TO USE FEWER COLUMNS, JUST BECAUSE IT GETS VERY VERBOSE TO PRINT EVERYTHING
<br>
<br>

```r
datSmall = dat[, list(race, gender, charges_citation, housing_location)]
```



# J and CJ

Here are some examples of how to use (and not use) the Join (J) and Cross Join (CJ)

Personally, I rarely used these functions at first.  I found it easier to give up some performance for simplicity. 



```r

## Count of observations by race
datSmall[, .N, by = race]
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

## Set the key to be 'race'', and do some joins
setkey(datSmall, "race")

## Simple inner Join, new syntax (and what I normally use)
datSmall["W"]
```

```
##     race gender               charges_citation housing_location
##  1:    W      M 720 ILCS 5 12-3.4(a)(1) [16128         08-2N-DR
##  2:    W      M    625 ILCS 5 6-303(a) [13526]      11-AH-3-411
##  3:    W      M   625 ILCS 5 11-501(a) [14041]              02-
##  4:    W      M    625 ILCS 5 6-303(a) [13526]      02-D4-MU-1-
##  5:    W      M 720 ILCS 5 12-13(a)(3) [995700       05-D-1-2-1
##  6:    W      M                            000      02-D1-H-3-H
##  7:    W      M     720 ILCS 5 12-3.2 [930200]       03-A-2-4-2
##  8:    W      M 720 ILCS 570 401(a)(2)(D) [509      11-AF-3-311
##  9:    W      M 720 ILCS 5 12-3.05(d)(4) [1610       01-B-2-3-1
## 10:    W      M 720 ILCS 5 12-3.2(a)(1) [10416      02-D3-HH-3-
## 11:    W      M 720 ILCS 570 401(c)(1) [13009]      03-AX-D3-1-
## 12:    W      M    720 ILCS 5 16A-3(a) [15599]      02-D4-QU-1-
## 13:    W      M  720 ILCS 570 402(c) [5101110]      03-AX-B1-1-
## 14:    W      M                            000       03-A-3-4-1
## 15:    W      M    720 ILCS 5 12-1(a) [920000]      02-D4-RL-1-
## 16:    W      M  720 ILCS 5 12-3(a)(2) [10530]       10-A-3-8-2
## 17:    W      M  625 ILCS 5 6-303(d) [5883000]      02-D1-A-2-A
## 18:    W      M  720 ILCS 5 16A-3(a) [1060000]      02-D2-W-2-W
## 19:    W      M       720 ILCS 5 12-3 [930000]      02-D4-ML-1-
## 20:    W      M  720 ILCS 570 402(c) [5101110]      14-B4-4-42-
## 21:    W      F   720 ILCS 5 16-3(a) [1025000]      04-J-1-11-1
## 22:    W      M   720 ILCS 5 19-1(a) [1110000]      02-D1-A-2-A
## 23:    W      M 720 ILCS 5 12-3.2(a)(1) [10416      03-A-2-19-1
## 24:    W      M 720 ILCS 5 12-3.2(a)(2) [10418      02-D2-U-3-U
## 25:    W      F   625 ILCS 5 11-501(a) [14039]          17-SFFP
## 26:    W      M  720 ILCS 570 402(c) [5101110]      03-A-1-26-1
## 27:    W      M   625 ILCS 5 11-501(a) [12809]      06-H-2-19-2
## 28:    W      M  720 ILCS 570 402(c) [5101110]      11-AC-1-208
## 29:    W      M 625 ILCS 5 11-501(a)(2) [11309            15-EM
## 30:    W      M             720 ILCS 5/19-1(a)       01-E-1-1-1
## 31:    W      M                        UNKNOWN      01-G-3-16-1
## 32:    W      M            625 ILCS 5/6-303(a)            15-EM
## 33:    W      M           625 ILCS 5/11-501(a)       06-D-1-3-2
## 34:    W      M             720 ILCS 5/19-1(a)            15-EM
## 35:    W      F           625 ILCS 5/11-501(a)      17-WR-N-C-2
## 36:    W      M           720 ILCS 5/9-1(a)(1)       01-A-3-3-2
## 37:    W      M                38-24-3.1(a)(6)      06-H-1-12-2
## 38:    W      M           625 ILCS 5/11-501(a)      02-D1-H-3-H
## 39:    W      M               720 ILCS 570/402            15-DR
## 40:    W      M              720 ILCS 550/5(g)       10-C-1-7-1
## 41:    W      M       720 ILCS 5/12-14.1(a)(1)          C DISCH
## 42:    W      M           720 ILCS 5/12-4.3(a)      11-DH-3-411
## 43:    W      M              720 ILCS 5/24-1.1      06-C-1-13-1
## 44:    W      M                  38-10-2(a)(3)      01-H-1-13-2
##     race gender               charges_citation housing_location
```

```r
## Simple inner Join, old syntax
datSmall[J("W")]
```

```
##     race gender               charges_citation housing_location
##  1:    W      M 720 ILCS 5 12-3.4(a)(1) [16128         08-2N-DR
##  2:    W      M    625 ILCS 5 6-303(a) [13526]      11-AH-3-411
##  3:    W      M   625 ILCS 5 11-501(a) [14041]              02-
##  4:    W      M    625 ILCS 5 6-303(a) [13526]      02-D4-MU-1-
##  5:    W      M 720 ILCS 5 12-13(a)(3) [995700       05-D-1-2-1
##  6:    W      M                            000      02-D1-H-3-H
##  7:    W      M     720 ILCS 5 12-3.2 [930200]       03-A-2-4-2
##  8:    W      M 720 ILCS 570 401(a)(2)(D) [509      11-AF-3-311
##  9:    W      M 720 ILCS 5 12-3.05(d)(4) [1610       01-B-2-3-1
## 10:    W      M 720 ILCS 5 12-3.2(a)(1) [10416      02-D3-HH-3-
## 11:    W      M 720 ILCS 570 401(c)(1) [13009]      03-AX-D3-1-
## 12:    W      M    720 ILCS 5 16A-3(a) [15599]      02-D4-QU-1-
## 13:    W      M  720 ILCS 570 402(c) [5101110]      03-AX-B1-1-
## 14:    W      M                            000       03-A-3-4-1
## 15:    W      M    720 ILCS 5 12-1(a) [920000]      02-D4-RL-1-
## 16:    W      M  720 ILCS 5 12-3(a)(2) [10530]       10-A-3-8-2
## 17:    W      M  625 ILCS 5 6-303(d) [5883000]      02-D1-A-2-A
## 18:    W      M  720 ILCS 5 16A-3(a) [1060000]      02-D2-W-2-W
## 19:    W      M       720 ILCS 5 12-3 [930000]      02-D4-ML-1-
## 20:    W      M  720 ILCS 570 402(c) [5101110]      14-B4-4-42-
## 21:    W      F   720 ILCS 5 16-3(a) [1025000]      04-J-1-11-1
## 22:    W      M   720 ILCS 5 19-1(a) [1110000]      02-D1-A-2-A
## 23:    W      M 720 ILCS 5 12-3.2(a)(1) [10416      03-A-2-19-1
## 24:    W      M 720 ILCS 5 12-3.2(a)(2) [10418      02-D2-U-3-U
## 25:    W      F   625 ILCS 5 11-501(a) [14039]          17-SFFP
## 26:    W      M  720 ILCS 570 402(c) [5101110]      03-A-1-26-1
## 27:    W      M   625 ILCS 5 11-501(a) [12809]      06-H-2-19-2
## 28:    W      M  720 ILCS 570 402(c) [5101110]      11-AC-1-208
## 29:    W      M 625 ILCS 5 11-501(a)(2) [11309            15-EM
## 30:    W      M             720 ILCS 5/19-1(a)       01-E-1-1-1
## 31:    W      M                        UNKNOWN      01-G-3-16-1
## 32:    W      M            625 ILCS 5/6-303(a)            15-EM
## 33:    W      M           625 ILCS 5/11-501(a)       06-D-1-3-2
## 34:    W      M             720 ILCS 5/19-1(a)            15-EM
## 35:    W      F           625 ILCS 5/11-501(a)      17-WR-N-C-2
## 36:    W      M           720 ILCS 5/9-1(a)(1)       01-A-3-3-2
## 37:    W      M                38-24-3.1(a)(6)      06-H-1-12-2
## 38:    W      M           625 ILCS 5/11-501(a)      02-D1-H-3-H
## 39:    W      M               720 ILCS 570/402            15-DR
## 40:    W      M              720 ILCS 550/5(g)       10-C-1-7-1
## 41:    W      M       720 ILCS 5/12-14.1(a)(1)          C DISCH
## 42:    W      M           720 ILCS 5/12-4.3(a)      11-DH-3-411
## 43:    W      M              720 ILCS 5/24-1.1      06-C-1-13-1
## 44:    W      M                  38-10-2(a)(3)      01-H-1-13-2
##     race gender               charges_citation housing_location
```

```r
## You can select more than one key
datSmall[J(c("W", "WH"))]
```

```
##       race gender               charges_citation housing_location
##    1:    W      M 720 ILCS 5 12-3.4(a)(1) [16128         08-2N-DR
##    2:    W      M    625 ILCS 5 6-303(a) [13526]      11-AH-3-411
##    3:    W      M   625 ILCS 5 11-501(a) [14041]              02-
##    4:    W      M    625 ILCS 5 6-303(a) [13526]      02-D4-MU-1-
##    5:    W      M 720 ILCS 5 12-13(a)(3) [995700       05-D-1-2-1
##   ---                                                            
## 2055:   WH      M            720 ILCS 5/32-10(a)       01-H-1-6-1
## 2056:   WH      M                         38-9-1          15-EMAW
## 2057:   WH      M                        38-19-3          15-EMAW
## 2058:   WH      M                       56.5-704          15-EMAW
## 2059:   WH      M                    95.5-11-501          15-DRAW
```

```r
## This still works
datSmall[c("W", "WH")]
```

```
##       race gender               charges_citation housing_location
##    1:    W      M 720 ILCS 5 12-3.4(a)(1) [16128         08-2N-DR
##    2:    W      M    625 ILCS 5 6-303(a) [13526]      11-AH-3-411
##    3:    W      M   625 ILCS 5 11-501(a) [14041]              02-
##    4:    W      M    625 ILCS 5 6-303(a) [13526]      02-D4-MU-1-
##    5:    W      M 720 ILCS 5 12-13(a)(3) [995700       05-D-1-2-1
##   ---                                                            
## 2055:   WH      M            720 ILCS 5/32-10(a)       01-H-1-6-1
## 2056:   WH      M                         38-9-1          15-EMAW
## 2057:   WH      M                        38-19-3          15-EMAW
## 2058:   WH      M                       56.5-704          15-EMAW
## 2059:   WH      M                    95.5-11-501          15-DRAW
```


## Examples with two keys: `race` and `gender`

```r
## Set the key
setkeyv(datSmall, c("race", "gender"))
```


Let's get records where race == "WH" and gender == "M"
There should be 1682 records of that sort based on this table:

```r
datSmall[, .N, keyby = list(race, gender)]
```

```
##     race gender     N
##  1:   AS      F     6
##  2:   AS      M   111
##  3:    B      F     3
##  4:    B      M    26
##  5:   BK      F  1209
##  6:   BK      M 12670
##  7:   IN      F     6
##  8:   IN      M     7
##  9:   LB      F     9
## 10:   LB      M    59
## 11:   LT      F    73
## 12:   LT      M  1730
## 13:   LW      F   100
## 14:   LW      M  1139
## 15:    W      F     3
## 16:    W      M    41
## 17:   WH      F   333
## 18:   WH      M  1682
```


## Try using J and CJ on two keys


```r
## This is the first way that I would have guessed to get results for
## 'white' and 'male', but it doesn't work:
datSmall[c("WH", "M")]
```

```
##       race gender               charges_citation housing_location
##    1:   WH      F 720 ILCS 5 12-3.2(a)(2) [10418      04-Q-1-11-1
##    2:   WH      F    720 ILCS 5 16A-3(a) [15601]      17-WR-N-A-2
##    3:   WH      F  720 ILCS 5 16A-3(a) [1060000]      04-Q-1-17-1
##    4:   WH      F   720 ILCS 5 11-14(a) [855900]      04-Q-1-19-2
##    5:   WH      F 625 ILCS 5 11-501(a)(1) [14721      04-Q-1-16-1
##   ---                                                            
## 2012:   WH      M                         38-9-1          15-EMAW
## 2013:   WH      M                        38-19-3          15-EMAW
## 2014:   WH      M                       56.5-704          15-EMAW
## 2015:   WH      M                    95.5-11-501          15-DRAW
## 2016:    M     NA                             NA               NA
```

```r
## If I add the summary using .N, you can see that it also selects Females
## (and returns 1 NA):
datSmall[c("WH", "M")][, .N, list(race, gender)]
```

```
##    race gender    N
## 1:   WH      F  333
## 2:   WH      M 1682
## 3:    M     NA    1
```

```r

## This one works.
datSmall[data.table("WH", "M")]
```

```
##       race gender               charges_citation housing_location
##    1:   WH      M 720 ILCS 5 12-3.4(a)(2) [16145              05-
##    2:   WH      M     720 ILCS 5 12-3.2 [930200]       05-L-2-1-2
##    3:   WH      M 720 ILCS 5 12-3.2(a)(1) [10416         08-2N-DR
##    4:   WH      M 720 ILCS 5 12-3.2(a)(1) [10416       05-E-2-3-2
##    5:   WH      M        720 ILCS 5 17-3 [11968]      11-BB-1-210
##   ---                                                            
## 1678:   WH      M            720 ILCS 5/32-10(a)       01-H-1-6-1
## 1679:   WH      M                         38-9-1          15-EMAW
## 1680:   WH      M                        38-19-3          15-EMAW
## 1681:   WH      M                       56.5-704          15-EMAW
## 1682:   WH      M                    95.5-11-501          15-DRAW
```

```r
datSmall[data.table("WH", "M")][, .N, list(race, gender)]
```

```
##    race gender    N
## 1:   WH      M 1682
```

```r

## But it's a little sloppy because you're actually making this data
## table, and then joining it with the master data:
data.table("WH", "M")
```

```
##    WH M
## 1: WH M
```

```r

## This is the right way to do it.
datSmall[J("WH", "M")]
```

```
##       race gender               charges_citation housing_location
##    1:   WH      M 720 ILCS 5 12-3.4(a)(2) [16145              05-
##    2:   WH      M     720 ILCS 5 12-3.2 [930200]       05-L-2-1-2
##    3:   WH      M 720 ILCS 5 12-3.2(a)(1) [10416         08-2N-DR
##    4:   WH      M 720 ILCS 5 12-3.2(a)(1) [10416       05-E-2-3-2
##    5:   WH      M        720 ILCS 5 17-3 [11968]      11-BB-1-210
##   ---                                                            
## 1678:   WH      M            720 ILCS 5/32-10(a)       01-H-1-6-1
## 1679:   WH      M                         38-9-1          15-EMAW
## 1680:   WH      M                        38-19-3          15-EMAW
## 1681:   WH      M                       56.5-704          15-EMAW
## 1682:   WH      M                    95.5-11-501          15-DRAW
```

```r
datSmall[CJ("WH", "M")]
```

```
##       race gender               charges_citation housing_location
##    1:   WH      M 720 ILCS 5 12-3.4(a)(2) [16145              05-
##    2:   WH      M     720 ILCS 5 12-3.2 [930200]       05-L-2-1-2
##    3:   WH      M 720 ILCS 5 12-3.2(a)(1) [10416         08-2N-DR
##    4:   WH      M 720 ILCS 5 12-3.2(a)(1) [10416       05-E-2-3-2
##    5:   WH      M        720 ILCS 5 17-3 [11968]      11-BB-1-210
##   ---                                                            
## 1678:   WH      M            720 ILCS 5/32-10(a)       01-H-1-6-1
## 1679:   WH      M                         38-9-1          15-EMAW
## 1680:   WH      M                        38-19-3          15-EMAW
## 1681:   WH      M                       56.5-704          15-EMAW
## 1682:   WH      M                    95.5-11-501          15-DRAW
```

```r

## Notice that with two keys this no longer works:
datSmall[J("WH", "W")]
```

```
##    race gender charges_citation housing_location
## 1:   WH      W               NA               NA
```

```r
## Notice that with two keys, but this does:
datSmall[J(c("WH", "W"))]
```

```
##       race gender               charges_citation housing_location
##    1:   WH      F 720 ILCS 5 12-3.2(a)(2) [10418      04-Q-1-11-1
##    2:   WH      F    720 ILCS 5 16A-3(a) [15601]      17-WR-N-A-2
##    3:   WH      F  720 ILCS 5 16A-3(a) [1060000]      04-Q-1-17-1
##    4:   WH      F   720 ILCS 5 11-14(a) [855900]      04-Q-1-19-2
##    5:   WH      F 625 ILCS 5 11-501(a)(1) [14721      04-Q-1-16-1
##   ---                                                            
## 2055:    W      M              720 ILCS 550/5(g)       10-C-1-7-1
## 2056:    W      M       720 ILCS 5/12-14.1(a)(1)          C DISCH
## 2057:    W      M           720 ILCS 5/12-4.3(a)      11-DH-3-411
## 2058:    W      M              720 ILCS 5/24-1.1      06-C-1-13-1
## 2059:    W      M                  38-10-2(a)(3)      01-H-1-13-2
```





