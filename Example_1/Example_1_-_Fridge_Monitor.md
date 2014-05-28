# Data Table Example 1 - Refrigerator monitor

### Gene Leynes
#### February 7, 2013


## Pictures of the device (for fun):

<img src="../resources/2013-01-28 02.46.11 (sm).jpg" align=middle height=150 width =235 />
<img src="../resources/2013-01-28 02.46.45 (sm).jpg" align=middle height=150 width =235 />
<img src="../resources/2013-01-28 02.46.54 (sm).jpg" align=middle height=150 width =235 />

## Introduction

After putting together an electronic device to monitor the temperature of my refrigerator, I discovered that it didn't work consistently.  It would run, crash, restart... and each time it would start a new file.  Still, I wanted to know if some basic things were working.  Specifically, did each file start at 1 second, and how long the files run before crashing.

Although this isn't the most exciting example in the world, it does show some nice features of the `data.table` package.  Especially, the `rbindlist` function.

#### PATTERN USED:

1. Initialize
2. List files
3. Read files into list structure
4. Combine using rbind list


<br>


### INITIALIZE


```r
opts_chunk$set(tidy = FALSE)
```



```r
rm(list=ls())
library(data.table)
```


### LIST AVAILABLE CSV FILES


```r
## The following line is needed to compile to HTML, please ignore
if (basename(getwd()) != "data.table_demo") {setwd("..")}

## LIST FILES
csvfiles = list.files(path = 'data/data logger results/', 
					  full.names=TRUE,
					  pattern = "[Cc][Ss][Vv]")
csvfiles
```

```
##  [1] "data/data logger results/LOGGER01.CSV"
##  [2] "data/data logger results/LOGGER02.CSV"
##  [3] "data/data logger results/LOGGER03.CSV"
##  [4] "data/data logger results/LOGGER04.CSV"
##  [5] "data/data logger results/LOGGER05.CSV"
##  [6] "data/data logger results/LOGGER06.CSV"
##  [7] "data/data logger results/LOGGER07.CSV"
##  [8] "data/data logger results/LOGGER08.CSV"
##  [9] "data/data logger results/LOGGER09.CSV"
## [10] "data/data logger results/LOGGER10.CSV"
## [11] "data/data logger results/LOGGER11.CSV"
## [12] "data/data logger results/LOGGER12.CSV"
## [13] "data/data logger results/LOGGER13.CSV"
## [14] "data/data logger results/LOGGER14.CSV"
```


### READ IN FILES USING LAPPLY

Notice that all the files have different sizes


```r
## The following line is needed to compile to HTML, please ignore
if (basename(getwd()) != "data.table_demo") {setwd("..")}

dat = lapply(csvfiles, read.csv)
str(dat, 1)
```

```
## List of 14
##  $ :'data.frame':	18 obs. of  6 variables:
##  $ :'data.frame':	5393 obs. of  6 variables:
##  $ :'data.frame':	7 obs. of  6 variables:
##  $ :'data.frame':	3 obs. of  6 variables:
##  $ :'data.frame':	17 obs. of  6 variables:
##  $ :'data.frame':	12 obs. of  6 variables:
##  $ :'data.frame':	201 obs. of  6 variables:
##  $ :'data.frame':	76 obs. of  6 variables:
##  $ :'data.frame':	36076 obs. of  6 variables:
##  $ :'data.frame':	175 obs. of  6 variables:
##  $ :'data.frame':	7119 obs. of  6 variables:
##  $ :'data.frame':	2633 obs. of  6 variables:
##  $ :'data.frame':	38196 obs. of  6 variables:
##  $ :'data.frame':	50 obs. of  6 variables:
```

```r
## ADD FILE NAME
for(i in 1:length(csvfiles)) {
	dat[[i]]$file = basename(csvfiles[i])
}
str(dat, 1)
```

```
## List of 14
##  $ :'data.frame':	18 obs. of  7 variables:
##  $ :'data.frame':	5393 obs. of  7 variables:
##  $ :'data.frame':	7 obs. of  7 variables:
##  $ :'data.frame':	3 obs. of  7 variables:
##  $ :'data.frame':	17 obs. of  7 variables:
##  $ :'data.frame':	12 obs. of  7 variables:
##  $ :'data.frame':	201 obs. of  7 variables:
##  $ :'data.frame':	76 obs. of  7 variables:
##  $ :'data.frame':	36076 obs. of  7 variables:
##  $ :'data.frame':	175 obs. of  7 variables:
##  $ :'data.frame':	7119 obs. of  7 variables:
##  $ :'data.frame':	2633 obs. of  7 variables:
##  $ :'data.frame':	38196 obs. of  7 variables:
##  $ :'data.frame':	50 obs. of  7 variables:
```


### Combine files using Data Table's `rbindlist`


```r
datAll = rbindlist(dat)
```


Now just print the data object itself to see the head and tail:


```r
datAll
```

```
##        millis.1000 sens0 sens1 sens2 sens3 sens4         file
##     1:           1     0     0   153   116    75 LOGGER01.CSV
##     2:           2     0     0   151   118    71 LOGGER01.CSV
##     3:           3     0     0   153   119    75 LOGGER01.CSV
##     4:           4     0     0   153   117    72 LOGGER01.CSV
##     5:           5     0     0   152   117    72 LOGGER01.CSV
##    ---                                                       
## 89972:          46     0     0   138   110    74 LOGGER14.CSV
## 89973:          47     0     0   138   110    74 LOGGER14.CSV
## 89974:          48     0     0   138   110    75 LOGGER14.CSV
## 89975:          49     0     0   138   110    75 LOGGER14.CSV
## 89976:          50     0     0   137   110    75 LOGGER14.CSV
```


# Other simple `data.table` examples

### Change a column name: "`millis.1000`" should be "`seconds`" <br> Data Table's `setnames` will fix that


```r
## Change to "seconds"
setnames(datAll, 'millis.1000', 'seconds')
colnames(datAll)
```

```
## [1] "seconds" "sens0"   "sens1"   "sens2"   "sens3"   "sens4"   "file"
```

```r

## NOTE:
## The alternative expression still works (with a warning), if you prefer.
## But who wants to read that??
# colnames(datAll)[which(colnames(datAll)=='millis.1000')] = 'seconds'
```



### Simple question: Did all the files start at 1 second?

First, let's look at some simple plots and tables to make sure that we know what we've got.


```r
## Plot to see what the index values look like
## Seems like each file starts at one and marches 
## through time linearly... which is what it should do.
indx = 1:nrow(datAll)
plot(seconds ~ indx, 
	 data = datAll, 
	 col = factor(file),
	 main = 'Time index by file')
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8.png) 

```r

## Seeing how many starts we had, when seconds==1
datAll[seconds==1]
```

```
##     seconds sens0 sens1 sens2 sens3 sens4         file
##  1:       1     0     0   153   116    75 LOGGER01.CSV
##  2:       1     0     0   154   114    72 LOGGER02.CSV
##  3:       1     0     0   149   111    73 LOGGER03.CSV
##  4:       1     0     0   149   111    73 LOGGER04.CSV
##  5:       1     0     0   149   111    73 LOGGER05.CSV
##  6:       1     0     0   146   119    73 LOGGER06.CSV
##  7:       1     0     0   146   118    74 LOGGER07.CSV
##  8:       1     0     0   149   115    71 LOGGER08.CSV
##  9:       1     0     0   141   109    74 LOGGER09.CSV
## 10:       1     0     0   142   109    73 LOGGER10.CSV
## 11:       1     0     4   141   116    74 LOGGER11.CSV
## 12:       1     0     0   149   112    73 LOGGER12.CSV
## 13:       1     0     0   153   119    68 LOGGER13.CSV
## 14:       1     0     0   138   109    74 LOGGER14.CSV
```

```r

## This shows us that it was the first element
## that had "seconds" == 1 when we group
## the data by "file"
datAll[i = TRUE,
	   j = which(seconds==1), 
	   by = file]
```

```
##             file V1
##  1: LOGGER01.CSV  1
##  2: LOGGER02.CSV  1
##  3: LOGGER03.CSV  1
##  4: LOGGER04.CSV  1
##  5: LOGGER05.CSV  1
##  6: LOGGER06.CSV  1
##  7: LOGGER07.CSV  1
##  8: LOGGER08.CSV  1
##  9: LOGGER09.CSV  1
## 10: LOGGER10.CSV  1
## 11: LOGGER11.CSV  1
## 12: LOGGER12.CSV  1
## 13: LOGGER13.CSV  1
## 14: LOGGER14.CSV  1
```

```r

## The .N is very useful!!
## Totals by file
datAll[ , .N, by=file]
```

```
##             file     N
##  1: LOGGER01.CSV    18
##  2: LOGGER02.CSV  5393
##  3: LOGGER03.CSV     7
##  4: LOGGER04.CSV     3
##  5: LOGGER05.CSV    17
##  6: LOGGER06.CSV    12
##  7: LOGGER07.CSV   201
##  8: LOGGER08.CSV    76
##  9: LOGGER09.CSV 36076
## 10: LOGGER10.CSV   175
## 11: LOGGER11.CSV  7119
## 12: LOGGER12.CSV  2633
## 13: LOGGER13.CSV 38196
## 14: LOGGER14.CSV    50
```



### Another way to answer the same question:

The above code tells us how many items we had in each group of files, and that each file started with "1" in the seconds field.  We could also ask what's the max and min of "seconds" in each file group to get the same answer, and confirm that the timer (probably) didn't skip any seconds... if the count by file == the max seconds by file I think it's safe to assume that the timer is at least working.

I'm going to do the query the old way, using data frames, then using data tables.


```r
## Data frame approach 1
with(datAll, tapply(seconds, file, range))
```

```
## $LOGGER01.CSV
## [1]  1 18
## 
## $LOGGER02.CSV
## [1]    1 5393
## 
## $LOGGER03.CSV
## [1] 1 7
## 
## $LOGGER04.CSV
## [1] 1 3
## 
## $LOGGER05.CSV
## [1]  1 17
## 
## $LOGGER06.CSV
## [1]  1 12
## 
## $LOGGER07.CSV
## [1]   1 201
## 
## $LOGGER08.CSV
## [1]  1 76
## 
## $LOGGER09.CSV
## [1]     1 36076
## 
## $LOGGER10.CSV
## [1]   1 175
## 
## $LOGGER11.CSV
## [1]    1 7119
## 
## $LOGGER12.CSV
## [1]    1 2633
## 
## $LOGGER13.CSV
## [1]     1 38196
## 
## $LOGGER14.CSV
## [1]  1 50
```

```r

## Data frame approach 2  (cleaned up)
do.call(rbind, 
		with(datAll, tapply(seconds, file, range)))
```

```
##              [,1]  [,2]
## LOGGER01.CSV    1    18
## LOGGER02.CSV    1  5393
## LOGGER03.CSV    1     7
## LOGGER04.CSV    1     3
## LOGGER05.CSV    1    17
## LOGGER06.CSV    1    12
## LOGGER07.CSV    1   201
## LOGGER08.CSV    1    76
## LOGGER09.CSV    1 36076
## LOGGER10.CSV    1   175
## LOGGER11.CSV    1  7119
## LOGGER12.CSV    1  2633
## LOGGER13.CSV    1 38196
## LOGGER14.CSV    1    50
```

```r

## Data table approach 1 (very simple)
datAll[ , range(seconds), by=file]
```

```
##             file    V1
##  1: LOGGER01.CSV     1
##  2: LOGGER01.CSV    18
##  3: LOGGER02.CSV     1
##  4: LOGGER02.CSV  5393
##  5: LOGGER03.CSV     1
##  6: LOGGER03.CSV     7
##  7: LOGGER04.CSV     1
##  8: LOGGER04.CSV     3
##  9: LOGGER05.CSV     1
## 10: LOGGER05.CSV    17
## 11: LOGGER06.CSV     1
## 12: LOGGER06.CSV    12
## 13: LOGGER07.CSV     1
## 14: LOGGER07.CSV   201
## 15: LOGGER08.CSV     1
## 16: LOGGER08.CSV    76
## 17: LOGGER09.CSV     1
## 18: LOGGER09.CSV 36076
## 19: LOGGER10.CSV     1
## 20: LOGGER10.CSV   175
## 21: LOGGER11.CSV     1
## 22: LOGGER11.CSV  7119
## 23: LOGGER12.CSV     1
## 24: LOGGER12.CSV  2633
## 25: LOGGER13.CSV     1
## 26: LOGGER13.CSV 38196
## 27: LOGGER14.CSV     1
## 28: LOGGER14.CSV    50
##             file    V1
```

```r

## Data table approach 2
datAll[ , list(min(seconds), max(seconds)), by=file]
```

```
##             file V1    V2
##  1: LOGGER01.CSV  1    18
##  2: LOGGER02.CSV  1  5393
##  3: LOGGER03.CSV  1     7
##  4: LOGGER04.CSV  1     3
##  5: LOGGER05.CSV  1    17
##  6: LOGGER06.CSV  1    12
##  7: LOGGER07.CSV  1   201
##  8: LOGGER08.CSV  1    76
##  9: LOGGER09.CSV  1 36076
## 10: LOGGER10.CSV  1   175
## 11: LOGGER11.CSV  1  7119
## 12: LOGGER12.CSV  1  2633
## 13: LOGGER13.CSV  1 38196
## 14: LOGGER14.CSV  1    50
```

```r

## Data table approach 3
datAll[i = TRUE, 
	   j = list(min = min(seconds), 
	   		    max = max(seconds)), 
	   by = file]
```

```
##             file min   max
##  1: LOGGER01.CSV   1    18
##  2: LOGGER02.CSV   1  5393
##  3: LOGGER03.CSV   1     7
##  4: LOGGER04.CSV   1     3
##  5: LOGGER05.CSV   1    17
##  6: LOGGER06.CSV   1    12
##  7: LOGGER07.CSV   1   201
##  8: LOGGER08.CSV   1    76
##  9: LOGGER09.CSV   1 36076
## 10: LOGGER10.CSV   1   175
## 11: LOGGER11.CSV   1  7119
## 12: LOGGER12.CSV   1  2633
## 13: LOGGER13.CSV   1 38196
## 14: LOGGER14.CSV   1    50
```


Notice that after cleaning up the code I ended up with something that seems a little verbose:

```r
datAll[i = TRUE, 
	   j = list(min = min(seconds), 
	   		    max = max(seconds)), 
	   by = file]
```


To make a nice table the old way it takes several steps.  I would argue that the data.table solution is much more readable than either of these (and it's definitely much faster):

```r
## NOT EVALUATED
## (also has no column names)
do.call(rbind, 
		with(datAll, tapply(seconds,file,range)))
```


```r
## NOT EVALUATED
## (also has no column names)
aggregate(datAll$seconds, by=list(file=datAll$file), function(x)c(min(x),max(x)))
```


```r
## NOT EVALUATED
data.frame(
	file =aggregate(datAll$file, by=list(file=datAll$file), '[', 1)$x,
	min = aggregate(datAll$seconds, by=list(file=datAll$file), min)$x,
	max = aggregate(datAll$seconds, by=list(file=datAll$file), max)$x,
	stringsAsFactors=FALSE)
```




<br><br><br><br>

## Some plots of the Sensors (has nothing to do with data.table)

Temperatures are converted to Fahrenheit, but the temperatures don't make much sense.  The values for Sens3 are especially suspect since it's not a calibrated instrument, but the other two are.  The freezer was in fact freezing, so the high temperatures don't make sense.

Also, I know that the pressure sensor was pressed more than once!!


```r
plot(datAll$sens0, main='pressure sensor tab (pressed randomly)')
```

![plot of chunk unnamed-chunk-14](figure/unnamed-chunk-141.png) 

```r
plot(datAll$sens1, main='light sensor (means fridge is open)')
```

![plot of chunk unnamed-chunk-14](figure/unnamed-chunk-142.png) 

```r
plot(datAll$sens2/10*9/5+32, main='fridge sensor')
```

![plot of chunk unnamed-chunk-14](figure/unnamed-chunk-143.png) 

```r
plot(datAll$sens3/10*9/5+32, main='external sensor (not a calibrated sensor)')
```

![plot of chunk unnamed-chunk-14](figure/unnamed-chunk-144.png) 

```r
plot(datAll$sens4/10*9/5+32, main='freezer sensor')
```

![plot of chunk unnamed-chunk-14](figure/unnamed-chunk-145.png) 

```r

boxplot(sens2~file, datAll)
```

![plot of chunk unnamed-chunk-14](figure/unnamed-chunk-146.png) 

```r
boxplot(sens3~file, datAll)
```

![plot of chunk unnamed-chunk-14](figure/unnamed-chunk-147.png) 

```r
boxplot(sens4~file, datAll)
```

![plot of chunk unnamed-chunk-14](figure/unnamed-chunk-148.png) 








