data.table_demo
===============

## Introduction

This code was used to demonstrate the features of the data.table package at the R User Group meetup held in Chicago on February 7th, 2013.

I used two sets of data:
1. Some data that I generated
2. Data from a data visualization project

<br><br>

### Copies of this project's exmples can be found here (precomplied to HTML using `knitr`):

Example 1: http://chicagodatascience.com/public/Example_1_-_Fridge_Monitor.html

Example 2: http://chicagodatascience.com/public/Example_2_-_26th_and_Calif.html

<br><br><br>

## Background for first example:

After putting together an electronic device to monitor the temperature of my refrigerator, I discovered that it didn't work consistently.  It would run, crash, restart... and each time it would start a new file.  Still, I wanted to know if some basic things were working.  Specifically, did each file start at 1 second, and how long the files run before crashing.

Although this isn't the most exciting example in the world, it does show some nice features of the `data.table` package.  Especially, the `rbindlist` function.

#### Picture of the device:

<img src="resources/2013-01-28 02.46.45 (sm).jpg" align=middle height=150 width =235 />

<br><br><br>

## Background for second example:
The data and project related to this exmaple comes from another meetup and code project.  I attended that meetup, and put together some simple visualizations for inspiration.  These visualizations were intended to be used as talking points for the project.  I repurposed this data to provide some `data.table` examples.

Meetup group: http://www.meetup.com/The-Chicago-Data-Visualization-Group/events/97690642/ <br>
Github project related to meetup: https://github.com/sc3/26thandcalifornia <br>
My project related to that meetup: https://github.com/geneorama/26_and_California <br>
Original meetup example: http://chicagodatascience.com/public/26th_and_California_example_visualizations.html <br>



