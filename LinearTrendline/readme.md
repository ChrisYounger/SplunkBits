# LinearTrendline

This folder contains resources to support a presentation I delivered to the Splunk UserGroup meetup in Brisbane on Feb 9th 2021.  For a good real-world example of the calculation, see `../DiskCapacityForecasting`

`ChrisYounger_SplunkUG_LinearTrendline_20210209.pptx` - The powerpoint presentation
`macros.conf` - A macro definition file for the calculation

Linear trendlines are particularly useful for understanding trends in very sparse datasets, or for running simple forecasts (e.g. disk capacity). Typically used with timeseries data, but can be used with any two dataseries that are numerical. Like most problems in Splunk, calculating a linear trendline can be done in numerous ways. The below method is what I have found to be the easist and compulationally cheapest. It can be used to plot a trendline on a graph and optionally extend it as far into the future as you like, just by changing the time picker. Additionally this algorithm also outputs the per-second change amount (`slope`). This value can be used to determine how many seconds/days/etc until the metric will reach a particular value (such as a threshold or limit). 

The reusable calculation takes the output of timechart (with fields "_time" and "value") and returns a new field called "trendline" which can be charted.

```
| eval x = if(isnull(value), null(), _time)
| eventstats count(x) as numevents sum(x) as sumX sum(value) as sumY sum(eval(x * value)) as sumXY sum(eval(x * x)) as sumX2 sum(eval(value * value)) as sumY2
| eval slope = ((numevents * sumXY) - (sumX * sumY)) / ((numevents * sumX2) - (sumX * sumX)) 
| eval yintercept = (sumY - (slope * sumX)) / numevents 
| eval trendline = (yintercept + (slope * _time)) 
```

Example use:

```
index=os host="server1" mount="C:"
| timechart avg(value) as value 

| eval x = if(isnull(value), null(), _time)
| eventstats count(x) as numevents sum(x) as sumX sum(value) as sumY sum(eval(x * value)) as sumXY sum(eval(x * x)) as sumX2 sum(eval(value * value)) as sumY2
| eval slope = ((numevents * sumXY) - (sumX * sumY)) / ((numevents * sumX2) - (sumX * sumX)) 
| eval yintercept = (sumY - (slope * sumX)) / numevents 
| eval trendline = (yintercept + (slope * _time)) 

| fields + _time value trendline
| rename value as "Usage %" trendline as "Trend"
```

Example using the macro:

```
index=_internal 
| timechart span=1m count as value 
| `lineartrend(_time,value)` 
| table _time value trendline
```


There is plenty more that can be extended from this starting point. 

Using a "by" argument on the `eventstats`, the trendline can be calculated on multiple metrics at once. See `../DiskCapacityForecasting` for an example of this.

Note that `eventstats` can run out of memory with massive datasets. Increase memory limits or refactor the calculations into two phases(searches), and replace eventstats with stats.

Forecast into the future using the time picker or time modifiers (e.g. +3mon)

Set the trendline to a dotted line using custom XML:
```
<option name="charting.fieldColors">{"Trend":"#dddddd"}</option>
<option name="charting.fieldDashStyles">{"Trend":"dash"}</option>
```

The R2 (a measure of the fit accuracy) of the trendline can be calculated like so:
```
| eval R = ((numevents * sumXY) - (sumX * sumY)) / sqrt (((numevents * sumX2) - (sumX * sumX)) * ((numevents * sumY2) - (sumY * sumY))) 
| eval R2 = R * R
```


Credit to https://wiki.splunk.com/Community:Plotting_a_linear_trendline for initial work on this. 