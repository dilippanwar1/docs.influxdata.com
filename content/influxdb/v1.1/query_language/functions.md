---
title: Functions

menu:
  influxdb_1_1:
    weight: 60
    parent: query_language
---

Aggregate, select, transform, and predict data with InfluxQL functions.

<table style="width:100%">
  <tr>
    <td><b>Aggregations:</b></td>
    <td><b>Selectors:</b></td>
    <td><b>Transformations:</b></td>
    <td><b>Predictors:</b></td>
    <td><b>General Functions Syntax:</b></td>
  </tr>
  <tr>
    <td><a href="#count">COUNT()</a></td>
    <td><a href="#bottom">BOTTOM()</a></td>
    <td><a href="#ceiling">CEILING()</a></td>
    <td><a href="#holt-winters">HOLT_WINTERS()</a></td>
    <td><a href="#multiple-functions-in-a-query">Multiple Functions in a Query</a></td>
  </tr>
  <tr>
    <td><a href="#distinct">DISTINCT()</a></td>
    <td><a href="#first">FIRST()</a></td>
    <td><a href="#cumulative-sum">CUMULATIVE_SUM()</a></td>
    <td><a href="#"></a></td>
    <td><a href="#rename-the-output-column-s-header">Rename the Output Column's Header</a></td>
  </tr>
  <tr>
    <td><a href="#integral">INTEGRAL()</a></td>
    <td><a href="#last">LAST()</a></td>
    <td><a href="#derivative">DERIVATIVE()</a></td>
    <td><a href="#"></a></td>
    <td><a href="#"common-issues-with-functions>Common Issues with Functions</a></td>
  </tr>
  <tr>
    <td><a href="#mean">MEAN()</a></td>
    <td><a href="#max">MAX()</a></td>
    <td><a href="#difference">DIFFERENCE()</a></td>
    <td><a href="#"></a></td>
    <td><a href="#"></a></td>
  </tr>
  <tr>
    <td><a href="#median">MEDIAN()</a></td>
    <td><a href="#min">MIN()</a></td>
    <td><a href="#elapsed">ELAPSED()</a></td>
    <td><a href="#"></a></td>
    <td><a href="#"></a></td>
  </tr>
  <tr>
    <td><a href="#mode">MODE()</a></td>
    <td><a href="#percentile">PERCENTILE()</a></td>
    <td><a href="#floor">FLOOR()</a></td>
    <td><a href="#"></a></td>
    <td><a href="#"></a></td>
  </tr>
  <tr>
    <td><a href="#spread">SPREAD()</a></td>
    <td><a href="#sample">SAMPLE()</a></td>
    <td><a href="#histogram">HISTOGRAM()</a></td>
    <td><a href="#"></a></td>
    <td><a href="#"></a></td>
  </tr>
  <tr>
    <td><a href="#stddev">STDDEV()</a></td>
    <td><a href="#top">TOP()</a></td>
    <td><a href="#moving-average">MOVING_AVERAGE()</a></td>
    <td><a href="#"></a></td>
    <td><a href="#"></a></td>
  </tr>
  <tr>
    <td><a href="#sum">SUM()</a></td>
    <td><a href="#""></a></td>
    <td><a href="#non-negative-derivative">NON_NEGATIVE_DERIVATIVE()</a></td>
    <td><a href="#"></a></td>
    <td><a href="#"></a></td>
  </tr>
</table>

### Sample Data
The data used in this document are available for download on the [Sample Data](/influxdb/v1.1/query_language/data_download/) page.

<br>
# Aggregation Functions

## COUNT()
Returns the number of non-null [field values](/influxdb/v1.1/concepts/glossary/#field-value).

### Syntax

```
SELECT COUNT( [ * | <field_key> | /<regular_expression>/ ] ) [INTO_clause] FROM_clause [WHERE_clause] [GROUP_BY_clause] [ORDER_BY_clause] [LIMIT_clause] [OFFSET_clause] [SLIMIT_clause] [SOFFSET_clause]
```

#### Nested Syntax
```
SELECT COUNT(DISTINCT( [ * | <field_key> | /<regular_expression>/ ] )) [...]
```

### Description of Syntax

`COUNT(field_key)`  
&emsp;&emsp;&emsp;
Returns the number of values in a single field.

`COUNT(/regular_expression/)`  
&emsp;&emsp;&emsp;
Returns the number of values for each field with a key that matches the [regular expression](/influxdb/v1.1/query_language/data_exploration/#regular-expressions-in-queries).

`COUNT(*)`  
&emsp;&emsp;&emsp;
Returns the number of values for every field.

`COUNT()` supports all field value [data types](/influxdb/v1.1/write_protocols/line_protocol_reference/#data-types).
InfluxQL supports nesting [`DISTINCT()`](#distinct) with `COUNT()`.

### Examples

#### Example 1: Count the number of non-null field values of a single field key
```
> SELECT COUNT("water_level") FROM "h2o_feet"

name: h2o_feet
time                   count
----                   -----
1970-01-01T00:00:00Z   15258
```
The query returns the number of non-null values in the `water_level`
[field key](/influxdb/v1.1/concepts/glossary/#field-key) in the `h2o_feet`
[measurement](/influxdb/v1.1/concepts/glossary/#measurement).

#### Example 2: Count the number of non-null field values for all field keys in a measurement
```
> SELECT COUNT(*) FROM "h2o_feet"

name: h2o_feet
time                   count_level description   count_water_level
----                   -----------------------   -----------------
1970-01-01T00:00:00Z   15258                     15258
```
The query returns the number of non-null values for every field key associated
with the `h2o_feet` measurement.

#### Example 3: Count the number of non-null field values for field keys that match a regular expression
```
> SELECT COUNT(/water/) FROM "h2o_feet"

name: h2o_feet
time                   count_water_level
----                   -----------------
1970-01-01T00:00:00Z   15258
```
The query returns the number of non-null values for every field key that
contains the word `water` in the `h2o_feet` measurement .

#### Example 4: Count the number of non-null field values for a single field key and include several clauses
```
> SELECT COUNT("water_level") FROM "h2o_feet" WHERE time >= '2015-08-17T23:48:00Z' AND time <= '2015-08-18T00:54:00Z' GROUP BY time(12m),* fill(200) LIMIT 7 SLIMIT 1

name: h2o_feet
tags: location=coyote_creek
time                   count
----                   -----
2015-08-17T23:48:00Z   200
2015-08-18T00:00:00Z   2
2015-08-18T00:12:00Z   2
2015-08-18T00:24:00Z   2
2015-08-18T00:36:00Z   2
2015-08-18T00:48:00Z   2
```
The query returns the number of non-null values in the `water_level` field key.
It covers the time range between `2015-08-17T23:48:00Z` and `2015-08-18T00:54:00Z`
and groups results into 12-minute time intervals and per tag.
The query fills empty time intervals with `200` and limits the number of points
and series returned to seven and one.

#### Example 5: Nest DISTINCT() in COUNT() to count the number of unique values for a single field key
```
> SELECT COUNT(DISTINCT("level description")) FROM "h2o_feet"

name: h2o_feet
time                   count
----                   -----
1970-01-01T00:00:00Z   4
```

The query returns the number of unique values in the `level description`
field key and the `h2o_feet` measurement.

### Common Issues with COUNT()

#### Issue 1: COUNT() and fill()
Most InfluxQL functions report `null` values for time interval with no data, and
[`fill(<fill_option>)`](/influxdb/v1.1/query_language/data_exploration/#group-by-time-intervals-and-fill)
changes the value reported for time intervals that have no data.
`COUNT()` reports `0` for time intervals with no data.
Using `fill(<fill_option>)` with `COUNT()` replaces any `0` values with the given
`fill_option`.

##### Example
<br>
The first query in the codeblock below does not include `fill()`.
The last time interval has no data so the `count` value for that time interval is zero.
The second query includes `fill(800000)`;
it replaces the zero in the last interval with `800000`.

```
> SELECT COUNT("water_level") FROM "h2o_feet" WHERE time >= '2015-09-18T21:24:00Z' AND time <= '2015-09-18T21:54:00Z' GROUP BY time(12m)
name: h2o_feet
time                   count
----                   -----
2015-09-18T21:24:00Z   2
2015-09-18T21:36:00Z   2
2015-09-18T21:48:00Z   0

> SELECT COUNT("water_level") FROM "h2o_feet" WHERE time >= '2015-09-18T21:24:00Z' AND time <= '2015-09-18T21:54:00Z' GROUP BY time(12m) fill(800000)
name: h2o_feet
time                   count
----                   -----
2015-09-18T21:24:00Z   2
2015-09-18T21:36:00Z   2
2015-09-18T21:48:00Z   800000
```

## DISTINCT()
Returns a list of unique [field values](/influxdb/v1.1/concepts/glossary/#field-value).

### Syntax
```
SELECT DISTINCT( [ * | <field_key> | /<regular_expression>/ ] ) FROM_clause [WHERE_clause] [GROUP_BY_clause] [ORDER_BY_clause] [LIMIT_clause] [OFFSET_clause] [SLIMIT_clause] [SOFFSET_clause]
```

#### Nested Syntax
```
SELECT COUNT(DISTINCT( [ * | <field_key> | /<regular_expression>/ ] )) [...]
```

### Description of Syntax

`DISTINCT(field_key)`  
&emsp;&emsp;&emsp;
Returns the unique values in a single field.

`DISTINCT(/regular_expression/)`  
&emsp;&emsp;&emsp;
Returns the unique values for each field with a key that matches the [regular expression](/influxdb/v1.1/query_language/data_exploration/#regular-expressions-in-queries).

`DISTINCT(*)`  
&emsp;&emsp;&emsp;
Returns the unique values for every field.

`DISTINCT()` supports all field value [data types](/influxdb/v1.1/write_protocols/line_protocol_reference/#data-types).
InfluxQL supports nesting `DISTINCT()` with [`COUNT()`](#count).

### Examples

#### Example 1: List the unique field values for a single field key
```
> SELECT DISTINCT("level description") FROM "h2o_feet"

name: h2o_feet
time                   distinct
----                   --------
1970-01-01T00:00:00Z   between 6 and 9 feet
1970-01-01T00:00:00Z   below 3 feet
1970-01-01T00:00:00Z   between 3 and 6 feet
1970-01-01T00:00:00Z   at or greater than 9 feet
```
The query returns a list in tabular format of each unique value of the `level description` [field](/influxdb/v1.1/concepts/glossary/#field) in the `h2o_feet` [measurement](/influxdb/v1.1/concepts/glossary/#measurement).

#### Example 2: List the unique field values for all field keys in a measurement
```
> SELECT DISTINCT(*) FROM "h2o_feet" LIMIT 5

name: h2o_feet
time                   distinct_level description   distinct_water_level
----                   --------------------------   --------------------
1970-01-01T00:00:00Z   below 3 feet                 2.064
1970-01-01T00:00:00Z   between 6 and 9 feet         8.12
1970-01-01T00:00:00Z   between 3 and 6 feet         2.116
1970-01-01T00:00:00Z                                8.005
1970-01-01T00:00:00Z                                2.028
```
The query returns a list of unique values for every field key associated with
the `h2o_feet` measurement.
TODO: unique combinations of field values?

#### Example 3: List the unique field values for field keys that match a regular expression
```
> SELECT DISTINCT(/description/) FROM "h2o_feet"

name: h2o_feet
time                   distinct_level description
----                   --------------------------
1970-01-01T00:00:00Z   below 3 feet
1970-01-01T00:00:00Z   between 6 and 9 feet
1970-01-01T00:00:00Z   between 3 and 6 feet
1970-01-01T00:00:00Z   at or greater than 9 feet
```
The query returns a list of unique values for every field key that contains the
word `description` in the `h2o_feet` measurement.

#### Example 4: List the unique field values for a single field key and include several clauses
```
>  SELECT DISTINCT("level description") FROM "h2o_feet" WHERE time >= '2015-08-17T23:48:00Z' AND time <= '2015-08-18T00:54:00Z' GROUP BY time(12m),* SLIMIT 1

name: h2o_feet
tags: location=coyote_creek
time                   distinct
----                   --------
2015-08-18T00:00:00Z   between 6 and 9 feet
2015-08-18T00:12:00Z   between 6 and 9 feet
2015-08-18T00:24:00Z   between 6 and 9 feet
2015-08-18T00:36:00Z   between 6 and 9 feet
2015-08-18T00:48:00Z   between 6 and 9 feet
```
The query returns the unique values in the `level description` field key.
It covers the time range between `2015-08-17T23:48:00Z` and
`2015-08-18T00:54:00Z` and groups results into 12-minute time intervals and
per tag.
The query also limits the number of series returned to one.

#### Example 5: Nest DISTINCT() in COUNT() to count the number of unique values for a single field key
```
> SELECT COUNT(DISTINCT("level description")) FROM "h2o_feet"

name: h2o_feet
time                   count
----                   -----
1970-01-01T00:00:00Z   4
```

The query returns the number of unique values in the `level description`
field key and the `h2o_feet` measurement.

### Common Issues with DISTINCT()

#### Issue 1: DISTINCT() and the INTO clause

Using the `DISTINCT()` function with the
[`INTO` clause](/influxdb/v1.1/query_language/data_exploration/#the-into-clause)
can cause InfluxDB to overwrite points in the destination measurement.
`DISTINCT()` often returns several results with the same timestamp; InfluxDB
assumes [points](/influxdb/v1.1/concepts/glossary/#point) with the same series
and timestamp are duplicate points so it simply overwrites points in the destination
measurement.

##### Example
<br>
Run a `DISTINCT()` query that returns several points with the same timestamp:
```
>  SELECT DISTINCT("level description") FROM "h2o_feet"

name: h2o_feet
time                   distinct
----                   --------
1970-01-01T00:00:00Z   below 3 feet
1970-01-01T00:00:00Z   between 6 and 9 feet
1970-01-01T00:00:00Z   between 3 and 6 feet
1970-01-01T00:00:00Z   at or greater than 9 feet
```
Run the same query with an `INTO` clause:
```
>  SELECT DISTINCT("level description") INTO "distincts" FROM "h2o_feet"

name: result
time                   written
----                   -------
1970-01-01T00:00:00Z   4
```
Query the data in the destination measurement:
```
> SELECT * FROM "distincts"

name: distincts
time                   distinct
----                   --------
1970-01-01T00:00:00Z   at or greater than 9 feet
```
Every time InfluxDB writes a point to `distincts` it overwrites the previous point
because each point has the same series and timestamp.
Instead of having four points in `distincts`, we end up with just one point.

## INTEGRAL()
`INTEGRAL()` is not yet functional.

<dt> See GitHub Issue [#5930](https://github.com/influxdata/influxdb/issues/5930) for more information.
</dt>

## MEAN()
Returns the arithmetic mean (average) of [field values](/influxdb/v1.1/concepts/glossary/#field-value).

### Syntax
```
SELECT MEAN( [ * | <field_key> | /<regular_expression>/ ] ) [INTO_clause] FROM_clause [WHERE_clause] [GROUP_BY_clause] [ORDER_BY_clause] [LIMIT_clause] [OFFSET_clause] [SLIMIT_clause] [SOFFSET_clause]
```

### Description of Syntax

`MEAN(field_key)`  
&emsp;&emsp;&emsp;
Returns the average value for a single field.

`MEAN(/regular_expression/)`  
&emsp;&emsp;&emsp;
Returns the average values for each field with a key that matches the [regular expression](/influxdb/v1.1/query_language/data_exploration/#regular-expressions-in-queries).

`MEAN(*)`  
&emsp;&emsp;&emsp;
Returns the average values for every field.

`MEAN()` supports int64 and float64 field value [data types](/influxdb/v1.1/write_protocols/line_protocol_reference/#data-types).

### Examples

#### Example 1: Calculate the average field values in a single field key
```
> SELECT MEAN("water_level") FROM "h2o_feet"

name: h2o_feet
time                   mean
----                   ----
1970-01-01T00:00:00Z   4.442107025822522
```
The query returns the average value in the `water_level`
[field key](/influxdb/v1.1/concepts/glossary/#field-key) in the `h2o_feet`
[measurement](/influxdb/v1.1/concepts/glossary/#measurement).

#### Example 2: Calculate the average field values for all field keys in a measurement
```
> SELECT MEAN(*) FROM "h2o_feet"

name: h2o_feet
time                   mean_water_level
----                   ----------------
1970-01-01T00:00:00Z   4.442107025822522
```
The query returns the average value for every field key that stores numerical
values in the `h2o_feet` measurement.

#### Example 3: Calculate the average field values for field keys that match a regular expression
```
> SELECT MEAN(/water/) FROM "h2o_feet"

name: h2o_feet
time                   mean_water_level
----                   ----------------
1970-01-01T00:00:00Z   4.442107025822523
```
The query returns the average value for every field key that stores numerical
values and includes the word `water` in the `h2o_feet` measurement.

#### Example 4: Calculate the average field values for a single field key and include several clauses
```
> SELECT MEAN("water_level") FROM "h2o_feet" WHERE time >= '2015-08-17T23:48:00Z' AND time <= '2015-08-18T00:54:00Z' GROUP BY time(12m),* fill(9.01) LIMIT 7 SLIMIT 1

name: h2o_feet
tags: location=coyote_creek
time                   mean
----                   ----
2015-08-17T23:48:00Z   9.01
2015-08-18T00:00:00Z   8.0625
2015-08-18T00:12:00Z   7.8245
2015-08-18T00:24:00Z   7.5675
2015-08-18T00:36:00Z   7.303
2015-08-18T00:48:00Z   7.11
```
The query returns the average of the values in the `water_level` field key.
It covers the time range between `2015-08-17T23:48:00Z` and `2015-08-18T00:54:00Z`
and groups results into 12-minute time intervals and per tag.
The query fills empty time intervals with `9.01` and limits the number of points
and series returned to seven and one.

## MEDIAN()
Returns the middle value from a sorted list of [field values](/influxdb/v1.1/concepts/glossary/#field-value).


### Syntax
```
SELECT MEDIAN( [ * | <field_key> | /<regular_expression>/ ] ) [INTO_clause] FROM_clause [WHERE_clause] [GROUP_BY_clause] [ORDER_BY_clause] [LIMIT_clause] [OFFSET_clause] [SLIMIT_clause] [SOFFSET_clause]
```

### Description of Syntax

`MEDIAN(field_key)`  
&emsp;&emsp;&emsp;
Returns the middle value for a single field.

`MEDIAN(/regular_expression/)`  
&emsp;&emsp;&emsp;
Returns the middle values for each field with a key that matches the [regular expression](/influxdb/v1.1/query_language/data_exploration/#regular-expressions-in-queries).

`MEDIAN(*)`  
&emsp;&emsp;&emsp;
Returns the middle values for every field.

`MEDIAN()` supports int64 and float64 field value [data types](/influxdb/v1.1/write_protocols/line_protocol_reference/#data-types).

> **Note:** `MEDIAN()` is nearly equivalent to [`PERCENTILE(field_key, 50)`](/influxdb/v1.1/query_language/functions/#percentile), except `MEDIAN()` returns the average of the two middle values if the field contains an even number of points.

### Examples

#### Example 1: Calculate the median field values in a single field key
```
> SELECT MEDIAN("water_level") FROM "h2o_feet"

name: h2o_feet
time                   median
----                   ------
1970-01-01T00:00:00Z   4.124
```
The query returns the median value in the `water_level`
[field key](/influxdb/v1.1/concepts/glossary/#field-key) in the `h2o_feet`
[measurement](/influxdb/v1.1/concepts/glossary/#measurement).

#### Example 2: Calculates the median field values for all field keys in a measurement
```
> SELECT MEDIAN(*) FROM "h2o_feet"

name: h2o_feet
time                   median_water_level
----                   ------------------
1970-01-01T00:00:00Z   4.124
```
The query returns the median value for every field key that stores numerical values
in the `h2o_feet` measurement.

#### Example 3: Calculate the median field values for field keys that match a regular expression
```
> SELECT MEDIAN(/water/) FROM "h2o_feet"

name: h2o_feet
time                   median_water_level
----                   ------------------
1970-01-01T00:00:00Z   4.124
```
The query returns the median value for every field key that stores numerical values
and includes the word `water` in the `h2o_feet` measurement.

#### Example 4: Calculate the median field values for a single field key and include several clauses
```
> SELECT MEDIAN("water_level") FROM "h2o_feet" WHERE time >= '2015-08-17T23:48:00Z' AND time <= '2015-08-18T00:54:00Z' GROUP BY time(12m),* fill(700) LIMIT 7 SLIMIT 1 SOFFSET 1

name: h2o_feet
tags: location=santa_monica
time                   median
----                   ------
2015-08-17T23:48:00Z   700
2015-08-18T00:00:00Z   2.09
2015-08-18T00:12:00Z   2.077
2015-08-18T00:24:00Z   2.0460000000000003
2015-08-18T00:36:00Z   2.0620000000000003
2015-08-18T00:48:00Z   700
```
The query returns the median of the values in the `water_level` field key.
It covers the time range between `2015-08-17T23:48:00Z` and `2015-08-18T00:54:00Z` and groups results into 12-minute time intervals and per tag.
The query fills empty time intervals with `700 `, limits the number of points and series returned to seven and one, and offsets the series returned by one.

## MODE()
Returns the most frequent value in a list of [field values](/influxdb/v1.1/concepts/glossary/#field-value).


### Syntax
```
SELECT MODE( [ * | <field_key> | /<regular_expression>/ ] ) [INTO_clause] FROM_clause [WHERE_clause] [GROUP_BY_clause] [ORDER_BY_clause] [LIMIT_clause] [OFFSET_clause] [SLIMIT_clause] [SOFFSET_clause]
```

### Description of Syntax

`MODE(field_key)`  
&emsp;&emsp;&emsp;
Returns the most frequent value in a single field.

`MODE(/regular_expression/)`  
&emsp;&emsp;&emsp;
Returns the most frequent values for each field with a key that matches the [regular expression](/influxdb/v1.1/query_language/data_exploration/#regular-expressions-in-queries).

`MODE(*)`  
&emsp;&emsp;&emsp;
Returns the most frequent values for every field.

`MODE()` supports all field value [data types](/influxdb/v1.1/write_protocols/line_protocol_reference/#data-types).

> **Note:** `MODE()` returns value with the earliest [timestamp](/influxdb/v1.1/concepts/glossary/#timestamps) if there's a tie between two or more values for maximum occurrences.

### Examples

#### Example 1: Calculate the mode for a single field
```
> SELECT MODE("level description") FROM "h2o_feet"

name: h2o_feet
time                   mode
----                   ----
1970-01-01T00:00:00Z   between 3 and 6 feet
```
The query returns the mode for all values associated with the `level description` [field key](/influxdb/v1.1/concepts/glossary/#field-key) in the `h2o_feet` [measurement](/influxdb/v1.1/concepts/glossary/#measurement).

#### Example 2: Calculate the mode for every field in a measurement
```
> SELECT MODE(*) FROM "h2o_feet"

name: h2o_feet
time                   mode_level description   mode_water_level
----                   ----------------------   ----------------
1970-01-01T00:00:00Z   between 3 and 6 feet     2.69
```
The query returns the modes for every field in the `h2o_feet` measurement.

#### Example 3: Calculate the mode for fields with keys that match a regular expression
```
> SELECT MODE(/water/) FROM "h2o_feet"

name: h2o_feet
time                   mode_water_level
----                   ----------------
1970-01-01T00:00:00Z   2.69
```
The query returns the mode for every field in the `h2o_feet` measurement that
includes the word `/water/` in the field key.

#### Example 4: Calculate the mode for a single field and include several clauses
```
> SELECT MODE("level description") FROM "h2o_feet" WHERE time >= '2015-08-17T23:48:00Z' AND time <= '2015-08-18T00:54:00Z' GROUP BY time(12m),* LIMIT 3 SLIMIT 1 SOFFSET 1

name: h2o_feet
tags: location=santa_monica
time                   mode
----                   ----
2015-08-17T23:48:00Z
2015-08-18T00:00:00Z   below 3 feet
2015-08-18T00:12:00Z   below 3 feet
```
The query returns the mode of the values associated with the `water_level` field key.
It covers the time range between `2015-08-17T23:48:00Z` and `2015-08-18T00:54:00Z` and groups results into 12-minute time intervals and per tag.
The query limits the number of points and series returned to three and one, and it offsets the series returned by one.

## SPREAD()
Returns the difference between a [field's](/influxdb/v1.1/concepts/glossary/#field) minimum and maximum values.

### Syntax
```
SELECT SPREAD( [ * | <field_key> | /<regular_expression>/ ] ) [INTO_clause] FROM_clause [WHERE_clause] [GROUP_BY_clause] [ORDER_BY_clause] [LIMIT_clause] [OFFSET_clause] [SLIMIT_clause] [SOFFSET_clause]
```

### Description of Syntax

`SPREAD(field_key)`  
&emsp;&emsp;&emsp;
Returns the difference between the minimum and maximum values in a single field.

`SPREAD(/regular_expression/)`  
&emsp;&emsp;&emsp;
Returns the difference between the minimum and maximum values for each field with a key that matches the [regular expression](/influxdb/v1.1/query_language/data_exploration/#regular-expressions-in-queries).

`SPREAD(*)`  
&emsp;&emsp;&emsp;
Returns the difference between the minimum and maximum values for every field.

`SPREAD()` supports int64 and float64 field value [data types](/influxdb/v1.1/write_protocols/line_protocol_reference/#data-types).

### Examples

#### Example 1: Calculate the spread for a single field
```
> SELECT SPREAD("water_level") FROM "h2o_feet"

name: h2o_feet
time                   spread
----                   ------
1970-01-01T00:00:00Z   10.574
```

The query returns the difference between the minimum and maximum values associated with the `water_level` field key in the `h2o_feet` measurement.

#### Example 2: Calculate the spread for every field in a measurement
```
> SELECT SPREAD(*) FROM "h2o_feet"

name: h2o_feet
time                   spread_water_level
----                   ------------------
1970-01-01T00:00:00Z   10.574
```

The query returns the difference between the minimum and maximum values associated with each numerical field in the `h2o_feet` measurement.
There's only one numerical field in the `h2o_feet` measurement so the query returns one result.

#### Example 3: Calculate the spread for fields with keys that match a regular expression
```
> SELECT SPREAD(/water/) FROM "h2o_feet"

name: h2o_feet
time                   spread_water_level
----                   ------------------
1970-01-01T00:00:00Z   10.574
```

The query returns the difference between the minimum and maximum values for every field in the `h2o_feet` measurement that includes the word `water` in its field key.
There's only one numerical field that includes the word `water` in its field key so the query returns one result.

#### Example 4: Calculate the spread for a single field and include several clauses
```
> SELECT SPREAD("water_level") FROM "h2o_feet" WHERE time >= '2015-08-17T23:48:00Z' AND time <= '2015-08-18T00:54:00Z' GROUP BY time(12m),* fill(18) LIMIT 3 SLIMIT 1 SOFFSET 1

name: h2o_feet
tags: location=santa_monica
time                   spread
----                   ------
2015-08-17T23:48:00Z   18
2015-08-18T00:00:00Z   0.052000000000000046
2015-08-18T00:12:00Z   0.09799999999999986
```

The query returns the difference between the minimum and maximum values for the `water_level` field.
It covers the time range between `2015-08-17T23:48:00Z` and `2015-08-18T00:54:00Z `and groups results into 12-minute time intervals and per tag.
The query fills empty time intervals with `18`, limits the number of points and series returned to three and one, and offsets the series returned by one.

## STDDEV()
Returns the standard deviation of a [field's](/influxdb/v1.1/concepts/glossary/#field) values.

### Syntax
```
SELECT STDDEV( [ * | <field_key> | /<regular_expression>/ ] ) [INTO_clause] FROM_clause [WHERE_clause] [GROUP_BY_clause] [ORDER_BY_clause] [LIMIT_clause] [OFFSET_clause] [SLIMIT_clause] [SOFFSET_clause]
```

### Description of Syntax

`STDDEV(field_key)`  
&emsp;&emsp;&emsp;
Returns the standard deviation for a single field.

`STDDEV(/regular_expression/)`  
&emsp;&emsp;&emsp;
Returns the standard deviations for each field with a key that matches the [regular expression](/influxdb/v1.1/query_language/data_exploration/#regular-expressions-in-queries).

`STDDEV(*)`  
&emsp;&emsp;&emsp;
Returns the standard deviations for every field.

`STDDEV()` supports int64 and float64 field value [data types](/influxdb/v1.1/write_protocols/line_protocol_reference/#data-types).

### Examples

#### Example 1: Calculate the standard deviation for a single field
```
> SELECT STDDEV("water_level") FROM "h2o_feet"

name: h2o_feet
time                   stddev
----                   ------
1970-01-01T00:00:00Z   2.279144584196145
```

The query returns standard deviation of all values associated with the `water_level` field key in the `h2o_feet` measurement.

#### Example 2: Calculate the standard deviation for every field in a measurement
```
> SELECT STDDEV(*) FROM "h2o_feet"

name: h2o_feet
time                   stddev_water_level
----                   ------------------
1970-01-01T00:00:00Z   2.279144584196145
```

The query returns the standard deviation for each numerical field in the `h2o_feet` measurement.
There's only one numerical field in the `h2o_feet` measurement so the query returns one result.

#### Example 3: Calculate the standard deviation for fields with keys that match a regular expression
```
> SELECT STDDEV(/water/) FROM "h2o_feet"

name: h2o_feet
time                   stddev_water_level
----                   ------------------
1970-01-01T00:00:00Z   2.279144584196145
```

The query returns the standard deviation for each numerical field in the `h2o_feet` measurement that includes the word `water` in its field key.
There's only one numerical field that includes the word `water` in its field key so the query returns one result.

#### Example 4: Calculate the standard deviation for a single field and include several clauses
```
> SELECT STDDEV("water_level") FROM "h2o_feet" WHERE time >= '2015-08-17T23:48:00Z' AND time <= '2015-08-18T00:54:00Z' GROUP BY time(12m),* fill(18000) LIMIT 2 SLIMIT 1 SOFFSET 1

name: h2o_feet
tags: location=santa_monica
time			stddev
----			------
2015-08-17T23:48:00Z	18000
2015-08-18T00:00:00Z	0.03676955262170051
```

The query returns the standard deviation of all values associated with the `water_level` field.
It covers the time range between `2015-08-17T23:48:00Z` and `2015-08-18T00:54:00Z` and groups results into 12-minute time intervals and per tag.
The query fills empty time intervals with `18000`, limits the number of points and series returned to two and one, and offsets the series returned by one.

## SUM()
Returns the sum of a [field's](/influxdb/v1.1/concepts/glossary/#field) values.

### Syntax
```
SELECT SUM( [ * | <field_key> | /<regular_expression>/ ] ) [INTO_clause] FROM_clause [WHERE_clause] [GROUP_BY_clause] [ORDER_BY_clause] [LIMIT_clause] [OFFSET_clause] [SLIMIT_clause] [SOFFSET_clause]
```

### Description of Syntax

`SUM(field_key)`  
&emsp;&emsp;&emsp;
Returns the sum of values in a single field.

`SUM(/regular_expression/)`  
&emsp;&emsp;&emsp;
Returns the sums of values for each field with a key that matches the [regular expression](/influxdb/v1.1/query_language/data_exploration/#regular-expressions-in-queries).

`SUM(*)`  
&emsp;&emsp;&emsp;
Returns the sums of values for every field.

`SUM()` supports int64 and float64 field value [data types](/influxdb/v1.1/write_protocols/line_protocol_reference/#data-types).

### Examples:

#### Example 1: Calculate the sum for a single field
```
> SELECT SUM("water_level") FROM "h2o_feet"

name: h2o_feet
time                   sum
----                   ---
1970-01-01T00:00:00Z   67777.66900000002
```

The query returns the summed total of all values associated with the `water_level` field key in the `h2o_feet` measurement.

#### Example 2: Calculate the sum for every field in a measurement
```
> SELECT SUM(*) FROM "h2o_feet"

name: h2o_feet
time                   sum_water_level
----                   ---------------
1970-01-01T00:00:00Z   67777.66900000004
```

The query returns the summed total of all values associated with each numerical field in the `h2o_feet` measurement.
There's only one numerical field in the `h2o_feet` measurement so the query returns one result.

#### Example 3: Calculate the sum for fields with keys that match a regular expression
```
> SELECT SUM(/water/) FROM "h2o_feet"

name: h2o_feet
time                   sum_water_level
----                   ---------------
1970-01-01T00:00:00Z   67777.66900000004
```

The query returns the summed total of all values associated with each numerical field in the `h2o_feet` measurement that includes the word `water` in its field key.
There's only one numerical field that includes the word `water` in its field key so the query returns one result.

#### Example 4: Calculate the sum for a single field and include several clauses
```
> SELECT SUM("water_level") FROM "h2o_feet" WHERE time >= '2015-08-17T23:48:00Z' AND time <= '2015-08-18T00:54:00Z' GROUP BY time(12m),* fill(18000) LIMIT 4 SLIMIT 1

name: h2o_feet
tags: location=coyote_creek
time                   sum
----                   ---
2015-08-17T23:48:00Z   18000
2015-08-18T00:00:00Z   16.125
2015-08-18T00:12:00Z   15.649
2015-08-18T00:24:00Z   15.135
```

The query returns the difference summed total of all values associated with the `water_level` field.
It covers the time range between `2015-08-17T23:48:00Z` and `2015-08-18T00:54:00Z` and groups results into 12-minute time intervals and per tag. The query fills empty time intervals with 18000, and it limits the number of points and series returned to four and one.


# Selectors

## BOTTOM()
Returns the smallest `N` values in a [field](/influxdb/v1.1/concepts/glossary/#field).

### Syntax
```
SELECT BOTTOM( <field_key>[,<tag_key(s)>],<N> )[,tag_key(s)] [INTO_clause] FROM_clause [WHERE_clause] [GROUP_BY_clause] [ORDER_BY_clause] [LIMIT_clause] [OFFSET_clause] [SLIMIT_clause] [SOFFSET_clause]
```

### Description of Syntax

`BOTTOM(field_key,N)`  
&emsp;&emsp;&emsp;
Returns the field's smallest N values.

`BOTTOM(field_key,tag_key(s),N)`  
&emsp;&emsp;&emsp;
Returns the smallest field value for N tag values of the tag key.

`BOTTOM(field_key,N),tag_key(s)`  
&emsp;&emsp;&emsp;
Returns the field's smallest N values and the relevant tag key-value.

The field key must store [int64 or float64](/influxdb/v1.1/write_protocols/line_protocol_reference/#data-types) values.

### Examples

#### Example 1: Select the smallest three values for a field
```
> SELECT BOTTOM("water_level",3) FROM "h2o_feet"

name: h2o_feet
time                   bottom
----                   ------
2015-08-29T14:30:00Z   -0.61
2015-08-29T14:36:00Z   -0.591
2015-08-30T15:18:00Z   -0.594
```
The query returns the lowest three values associated with the `water_level` field key in the `h2o_feet` measurement.

#### Example 2: Select the smallest value for a field and two tag key-value pairs
```
> SELECT BOTTOM("water_level","location",2) FROM "h2o_feet"

name: h2o_feet
time                   bottom   location
----                   ------   --------
2015-08-29T10:36:00Z   -0.243   santa_monica
2015-08-29T14:30:00Z   -0.61    coyote_creek
```
The query returns the lowest values in the `water_level` field key for the two tag key-value pairs associated with the `location` tag key.

#### Example 3: Select the smallest four values for a field and the relevant tag key-values
```
> SELECT BOTTOM("water_level",4),location FROM "h2o_feet"

name: h2o_feet
time                   bottom   location
----                   ------   --------
2015-08-29T14:24:00Z   -0.587   coyote_creek
2015-08-29T14:30:00Z   -0.61    coyote_creek
2015-08-29T14:36:00Z   -0.591   coyote_creek
2015-08-30T15:18:00Z   -0.594   coyote_creek
```
The query returns the lowest four values in the `water_level` field key and the relevant values of the `location` tag.

#### Example 4: Select the smallest three values for a field and include several clauses
```
> SELECT BOTTOM("water_level",3),"location" FROM "h2o_feet" WHERE time >= '2015-08-18T00:00:00Z' AND time <= '2015-08-18T00:54:00Z' GROUP BY time(24m) ORDER BY time DESC

name: h2o_feet
time                   bottom   location
----                   ------   --------
2015-08-18T00:48:00Z   1.991    santa_monica
2015-08-18T00:48:00Z   2.054    santa_monica
2015-08-18T00:48:00Z   6.982    coyote_creek
2015-08-18T00:24:00Z   2.041    santa_monica
2015-08-18T00:24:00Z   2.051    santa_monica
2015-08-18T00:24:00Z   2.057    santa_monica
2015-08-18T00:00:00Z   2.028    santa_monica
2015-08-18T00:00:00Z   2.064    santa_monica
2015-08-18T00:00:00Z   2.116    santa_monica
```
The query returns the smallest three values in the `water_level` field key.
It covers the time range between `2015-08-18T00:00:00Z` and `2015-08-18T00:54:00Z`, groups results into 24-minute time intervals and returns results in descending timestamp order.

Notice that the `GROUP BY time()` clause overrides the points' original timestamps.
The timestamps in the results indicate the the start of each 24-minute time interval;
the last three points in the results are for the time interval between `2015-08-18T00:00:00Z` and just before `2015-08-18T00:24:00Z`.

### Common Issues with `BOTTOM()`

#### Issue 1: BOTTOM(), the INTO clause, and the GROUP BY time() clause

Using the `BOTTOM()` function with the
[`INTO` clause](/influxdb/v1.1/query_language/data_exploration/#the-into-clause)
and the [`GROUP BY time()` clause](/influxdb/v1.1/query_language/data_exploration/#the-group-by-time-clause)
can cause InfluxDB to overwrite points in the destination measurement.
Using `BOTTOM()` with the `GROUP BY time()` clause often returns several results with the same timestamp; InfluxDB
assumes [points](/influxdb/v1.1/concepts/glossary/#point) with the same series
and timestamp are duplicate points so it simply overwrites points in the destination
measurement.

##### Example
<br>
Run a `BOTTOM()` query that returns several points with the same timestamp:
```
> SELECT BOTTOM("water_level",2),"location" FROM "h2o_feet" WHERE time >= '2015-08-18T00:00:00Z' AND time <= '2015-08-18T00:24:00Z' GROUP BY time(24m)

name: h2o_feet
time                   bottom   location
----                   ------   --------
2015-08-18T00:00:00Z   2.028    santa_monica
2015-08-18T00:00:00Z   2.064    santa_monica
2015-08-18T00:24:00Z   2.041    santa_monica
2015-08-18T00:24:00Z   7.635    coyote_creek
```
Run the same query with an `INTO` clause:
```
> SELECT BOTTOM("water_level",2),"location" INTO "bottom_dweller" FROM "h2o_feet" WHERE time >= '2015-08-18T00:00:00Z' AND time <= '2015-08-18T00:24:00Z' GROUP BY time(24m)

name: result
time                   written
----                   -------
1970-01-01T00:00:00Z   4
```
Query the data in the destination measurement:
```
> SELECT * FROM "bottom_dweller"

name: bottom_dweller
time                   bottom   location
----                   ------   --------
2015-08-18T00:00:00Z   2.064    santa_monica
2015-08-18T00:24:00Z   7.635    coyote_creek
```
When InfluxDB writes the results to `bottom_dweller` it overwrites any points with the same series and timestamp.
Instead of having four points in `bottom_dweller`, we end up with just two points.

#### Issue 2: BOTTOM() and a tag key with limited tag values

Queries with the syntax `SELECT BOTTOM(<field_key>,<tag_key>,<N>)` can return fewer points than expected.
If the tag key has `X` tag values, the query specifies `N` values, and `X` is smaller than `N`, the query returns `X` points.

##### Example
<br>
```
> SELECT BOTTOM("water_level","location",3) FROM "h2o_feet"

name: h2o_feet
time                   bottom   location
----                   ------   --------
2015-08-29T10:36:00Z   -0.243   santa_monica
2015-08-29T14:30:00Z   -0.61    coyote_creek
```

The query specifies the smallest field values of `water_level` for three tag values of the `location` tag key.
Because the `location` tag key has two tag values (`santa_monica` and `coyote_creek`), the query returns two points instead of three.

#### Issue 3: BOTTOM() and tied smallest values

InfluxDB returns the field value with the earliest timestamp if more than one point has the smallest field value.

##### Example
<br>
```
> SELECT BOTTOM("water_level",2) FROM "h2o_feet" WHERE time >= '2015-08-18T04:00:00Z' AND time < '2015-08-18T04:24:00Z' AND "location" = 'santa_monica'

name: h2o_feet
time                   bottom
----                   ------
2015-08-18T04:00:00Z   3.911
2015-08-18T04:06:00Z   4.055
```

In the raw data, `water_level` equals `4.055` at `2015-08-18T04:06:00Z` and at `2015-08-18T04:12:00Z`.
In the case of a tie, InfluxDB returns the point with the earlier timestamp.

## FIRST()
Returns the oldest value (determined by the timestamp) of a single [field](/influxdb/v1.1/concepts/glossary/#field).

### Syntax
```
SELECT FIRST(<field_key>)[,tag_key(s)] [INTO_clause] FROM_clause [WHERE_clause] [GROUP_BY_clause] [ORDER_BY_clause] [LIMIT_clause] [OFFSET_clause] [SLIMIT_clause] [SOFFSET_clause]
```

### Description of Syntax

`FIRST(field_key)`  
&emsp;&emsp;&emsp;
Returns the field's oldest value.

`FIRST(field_key),tag_key(s)`  
&emsp;&emsp;&emsp;
Returns the field's oldest value and the relevant tag key-value.

`FIRST()` supports all field value [data types](/influxdb/v1.1/write_protocols/line_protocol_reference/#data-types).

### Examples

#### Example 1: Select the oldest value in a field
```
> SELECT FIRST("level description") FROM "h2o_feet"

name: h2o_feet
time                   first
----                   -----
2015-08-18T00:00:00Z   between 6 and 9 feet
```
The query returns the oldest value associated with the `level description` field key in the `h2o_feet` measurement.

#### Example 2: Select the oldest value in a field and the relevant tag key-value
```
> SELECT FIRST("level description"),"location" FROM "h2o_feet"

name: h2o_feet
time                   first                  location
----                   -----                  --------
2015-08-18T00:00:00Z   between 6 and 9 feet   coyote_creek
```
The query returns the oldest values associated with the `level description` field key in and the relevant value of the `location` tag.

#### Example 3: Select the oldest value in a field and include several clauses
```
> SELECT FIRST("water_level") FROM "h2o_feet" WHERE time >= '2015-08-17T23:48:00Z' AND time <= '2015-08-18T00:54:00Z' GROUP BY time(12m),* fill(9.01) LIMIT 4 SLIMIT 1

name: h2o_feet
tags: location=coyote_creek
time                   first
----                   -----
2015-08-17T23:48:00Z   9.01
2015-08-18T00:00:00Z   8.12
2015-08-18T00:12:00Z   7.887
2015-08-18T00:24:00Z   7.635
```
The query returns the oldest value in the `water_level` field key.
It covers the time range between `2015-08-17T23:48:00Z` and `2015-08-18T00:54:00Z` and groups results in to 12-minute time intervals and per tag.
The query fills empty time intervals with `9.01`, and it limits the number of points and series returned to four and one.

## LAST()
Returns the newest value (determined by the timestamp) of a single [field](/influxdb/v1.1/concepts/glossary/#field).

### Syntax
```
SELECT LAST(<field_key>)[,tag_key(s)] [INTO_clause] FROM_clause [WHERE_clause] [GROUP_BY_clause] [ORDER_BY_clause] [LIMIT_clause] [OFFSET_clause] [SLIMIT_clause] [SOFFSET_clause]
```

### Description of Syntax

`LAST(field_key)`  
&emsp;&emsp;&emsp;
Returns the field's newest value.

`LAST(field_key),tag_key(s)`  
&emsp;&emsp;&emsp;
Returns the field's newest value and the relevant tag key-value.

`LAST()` supports all field value [data types](/influxdb/v1.1/write_protocols/line_protocol_reference/#data-types).

### Examples

#### Example 1: Select the newest value in a field
```
> SELECT LAST("level description") FROM "h2o_feet"

name: h2o_feet
time                   last
----                   ----
2015-09-18T21:42:00Z   between 3 and 6 feet
```
The query returns the newest value associated with the `level description` field key in the `h2o_feet` measurement.

#### Example 2: Select the newest value in a field and the relevant tag key-value
```
> SELECT LAST("level description"),"location" FROM "h2o_feet"

name: h2o_feet
time                   last                   location
----                   ----                   --------
2015-09-18T21:42:00Z   between 3 and 6 feet   santa_monica
```
The query returns the newest values associated with the `level description` field key in and the relevant value of the `location` tag.

#### Example 3: Select the oldest value in a field and include several clauses
```
> SELECT LAST("water_level") FROM "h2o_feet" WHERE time >= '2015-08-17T23:48:00Z' AND time <= '2015-08-18T00:54:00Z' GROUP BY time(12m),* fill(9.01) LIMIT 4 SLIMIT 1

name: h2o_feet
tags: location=coyote_creek
time                   last
----                   ----
2015-08-17T23:48:00Z   9.01
2015-08-18T00:00:00Z   8.005
2015-08-18T00:12:00Z   7.762
2015-08-18T00:24:00Z   7.5
```
The query returns the newest value in the `water_level` field key.
It covers the time range between `2015-08-17T23:48:00Z` and `2015-08-18T00:54:00Z` and groups results in to 12-minute time intervals and per tag.
The query fills empty time intervals with `9.01`, and it limits the number of points and series returned to four and one.

## MAX()
Returns the greatest value of a single [field](/influxdb/v1.1/concepts/glossary/#field).

### Syntax
```
SELECT MAX(<field_key>)[,tag_key(s)] [INTO_clause] FROM_clause [WHERE_clause] [GROUP_BY_clause] [ORDER_BY_clause] [LIMIT_clause] [OFFSET_clause] [SLIMIT_clause] [SOFFSET_clause]
```

### Description of Syntax

`MAX(field_key)`  
&emsp;&emsp;&emsp;
Returns the field's greatest value.

`MAX(field_key),tag_key(s)`  
&emsp;&emsp;&emsp;
Returns the field's greatest value and the relevant tag key-value.

`MAX()` supports field keys with int64 and float64 [values](/influxdb/v1.1/write_protocols/line_protocol_reference/#data-types).

### Examples

#### Example 1: Select the maximum value in a field
```
> SELECT MAX("water_level") FROM "h2o_feet"

name: h2o_feet
time                   max
----                   ---
2015-08-29T07:24:00Z   9.964
```
The query returns the greatest value associated with the `water_level` field key in the `h2o_feet` measurement.

#### Example 2: Select the maximum value in a field and the relevant tag key-value
```
> SELECT MAX("water_level"),"location" FROM "h2o_feet"

name: h2o_feet
time                   max     location
----                   ---     --------
2015-08-29T07:24:00Z   9.964   coyote_creek
```
The query returns the greatest values associated with the `water_level` field key in and the relevant value of the `location` tag.

#### Example 3: Select the maximum value in a field and include several clauses
```
> SELECT MAX("water_level") FROM "h2o_feet" WHERE time >= '2015-08-17T23:48:00Z' AND time <= '2015-08-18T00:54:00Z' GROUP BY time(12m),* fill(9.01) LIMIT 4 SLIMIT 1

name: h2o_feet
tags: location=coyote_creek
time                   max
----                   ---
2015-08-17T23:48:00Z   9.01
2015-08-18T00:00:00Z   8.12
2015-08-18T00:12:00Z   7.887
2015-08-18T00:24:00Z   7.635
```
The query returns the greatest value in the `water_level` field key.
It covers the time range between `2015-08-17T23:48:00Z` and `2015-08-18T00:54:00Z` and groups results in to 12-minute time intervals and per tag.
The query fills empty time intervals with `9.01`, and it limits the number of points and series returned to four and one.

## MIN()
Returns the lowest value of a single [field](/influxdb/v1.1/concepts/glossary/#field).

### Syntax
```
SELECT MIN(<field_key>)[,tag_key(s)] [INTO_clause] FROM_clause [WHERE_clause] [GROUP_BY_clause] [ORDER_BY_clause] [LIMIT_clause] [OFFSET_clause] [SLIMIT_clause] [SOFFSET_clause]
```

### Description of Syntax

`MIN(field_key)`  
&emsp;&emsp;&emsp;
Returns the field's lowest value.

`MIN(field_key),tag_key(s)`  
&emsp;&emsp;&emsp;
Returns the field's lowest value and the relevant tag key-value.

`MIN()` supports field keys with int64 and float64 [values](/influxdb/v1.1/write_protocols/line_protocol_reference/#data-types).

### Examples

#### Example 1: Select the minimum value in a field
```
> SELECT MIN("water_level") FROM "h2o_feet"

name: h2o_feet
time                   min
----                   ---
2015-08-29T14:30:00Z   -0.61
```
The query returns the lowest value associated with the `water_level` field key in the `h2o_feet` measurement.

#### Example 2: Select the minimum value in a field and the relevant tag key-value
```
> SELECT MIN("water_level"),"location" FROM "h2o_feet"

name: h2o_feet
time                   min     location
----                   ---     --------
2015-08-29T14:30:00Z   -0.61   coyote_creek
```
The query returns the lowest values associated with the `water_level` field key in and the relevant value of the `location` tag.

#### Example 3: Select the minimum value in a field and include several clauses
```
> SELECT MIN("water_level") FROM "h2o_feet" WHERE time >= '2015-08-17T23:48:00Z' AND time <= '2015-08-18T00:54:00Z' GROUP BY time(12m),* fill(9.01) LIMIT 4 SLIMIT 1

name: h2o_feet
tags: location=coyote_creek
time                   min
----                   ---
2015-08-17T23:48:00Z   9.01
2015-08-18T00:00:00Z   8.005
2015-08-18T00:12:00Z   7.762
2015-08-18T00:24:00Z   7.5
```
The query returns the lowest value in the `water_level` field key.
It covers the time range between `2015-08-17T23:48:00Z` and `2015-08-18T00:54:00Z` and groups results in to 12-minute time intervals and per tag.
The query fills empty time intervals with `9.01`, and it limits the number of points and series returned to four and one.

## PERCENTILE()
Returns the `N`th percentile value for the sorted values of a single [field](/influxdb/v1.1/concepts/glossary/#field).

### Syntax
```
SELECT PERCENTILE(<field_key>, <N>)[,tag_key(s)] [INTO_clause] FROM_clause [WHERE_clause] [GROUP_BY_clause] [ORDER_BY_clause] [LIMIT_clause] [OFFSET_clause] [SLIMIT_clause] [SOFFSET_clause]
```

### Description of Syntax
The field must be of type int64 or float64.
The percentile `N` must be an integer or floating point number between 0 and 100, inclusive.

### Examples

#### Example 1: Calculate the fifth percentile in a field
```
> SELECT PERCENTILE("water_level",5) FROM "h2o_feet"

name: h2o_feet
time                   percentile
----                   ----------
2015-08-31T03:42:00Z   1.122
```
The query returns the field value that is larger than 5% of the values in the `water_level` field.

#### Example 2: Calculate the fifth percentile in a field and return the relevant tag key-value
```
> SELECT PERCENTILE("water_level",5),"location" FROM "h2o_feet"

name: h2o_feet
time                   percentile   location
----                   ----------   --------
2015-08-31T03:42:00Z   1.122        coyote_creek
```
The query returns the field value that is larger than 5% of the values in the `water_level` field and the relevant value of the `location` tag.

#### Example 3: Calculate the twentieth percentile in a field and include several clauses
```
> SELECT PERCENTILE("water_level",20) FROM "h2o_feet" WHERE time >= '2015-08-17T23:48:00Z' AND time <= '2015-08-18T00:54:00Z' GROUP BY time(24m) fill(15) LIMIT 2

name: h2o_feet
time                   percentile
----                   ----------
2015-08-17T23:36:00Z   15
2015-08-18T00:00:00Z   2.064
```
The query returns the field value that is larger than 20% of the values in the `water_level` field.
It covers the time range between `2015-08-17T23:48:00Z` and `2015-08-18T00:54:00Z` and groups results into 24-minute intervals.
It fills empty time intervals with `15` and it limits the number of points returned to two.

### Common Issues with PERCENTILE()

#### Issue 1: PERCENTILE() vs. other InfluxQL functions

* `PERCENTILE(<field_key>,100)` is equivalent to [`MAX(<field_key>)`](#max).
* `PERCENTILE(<field_key>, 50)` is nearly equivalent to [`MEDIAN(<field_key>)`](#median), except the MEDIAN() function returns the average of the two middle values if the field contains an even number of points.
* `PERCENTILE(<field_key>,0)` is not equivalent to [`MIN(<field_key>)`](#min). This is a known [issue](https://github.com/influxdata/influxdb/issues/4418).

## SAMPLE()
Returns a random sample of `N` values for the specified [field](/influxdb/v1.1/concepts/glossary/#field).
InfluxDB uses [reservoir sampling](https://en.wikipedia.org/wiki/Reservoir_sampling) to generate the random points.

### Syntax
```
SELECT SAMPLE(<field_key>, <N>)[,tag_key(s)] [INTO_clause] FROM_clause [WHERE_clause] [GROUP_BY_clause] [ORDER_BY_clause] [LIMIT_clause] [OFFSET_clause] [SLIMIT_clause] [SOFFSET_clause]
```

### Description of Syntax

`SAMPLE(field_key)`  
&emsp;&emsp;&emsp;
Returns the field's oldest value.

`SAMPLE(field_key),tag_key(s)`  
&emsp;&emsp;&emsp;
Returns the field's oldest value and the relevant tag key-value.

`SAMPLE()` supports all field value [data types](/influxdb/v1.1/write_protocols/line_protocol_reference/#data-types).

### Examples

#### Example 1: Select a random sample from a field
```
> SELECT SAMPLE("water_level",2) FROM "h2o_feet"

name: h2o_feet
time                   sample
----                   ------
2015-09-09T21:48:00Z   5.659
2015-09-18T10:00:00Z   6.939
```
The query returns two randomly selected points from the `water_level` field
in the `h2o_feet` measurement.

#### Example 2: Select a random sample from a field and the relevant tag key-values
```
> SELECT SAMPLE("water_level",2),"location" FROM "h2o_feet"

name: h2o_feet
time                   sample   location
----                   ------   --------
2015-08-28T13:06:00Z   2.533    santa_monica
2015-09-07T06:12:00Z   6.145    coyote_creek
```
The query returns two randomly selected points from the `water_level` field and the relevant values of the `location` tag.

#### Example 3: Select a random sample of one point per `GROUP BY time()` interval
```
> SELECT SAMPLE("water_level",1) FROM "h2o_feet" WHERE time >= '2015-08-18T00:00:00Z' AND time <= '2015-08-18T00:30:00Z' AND "location" = 'santa_monica' GROUP BY time(18m)

name: h2o_feet
time                   sample
----                   ------
2015-08-18T00:12:00Z   2.028
2015-08-18T00:30:00Z   2.051
```

The query returns one randomly selected point per 18-minute `GROUP BY time()`
interval.
Note that the timestamps returned are the points' original timestamps.

### Common Issues with `SAMPLE()`

#### Issue 1: `SAMPLE()` with a `GROUP BY time()` clause
Queries with `SAMPLE()` and a `GROUP BY time()` clause return the specified
number of points (`N`) per `GROUP BY time()` interval.
For
[most `GROUP BY time()` queries](/influxdb/v1.1/query_language/data_exploration/#group-by-time-intervals),
the returned timestamps mark the start of the `GROUP BY time()` interval.
`GROUP BY time()` queries with the `SAMPLE()` function behave differently;
they maintain the timestamp of the original data point.

##### Example
<br>
The query below returns two randomly selected points per 18-minute
`GROUP BY time()` interval.
Notice that the returned timestamps are the original timestamps; they
are not forced to match the start of the `GROUP BY time()` intervals.

```
> SELECT SAMPLE("water_level",2) FROM "h2o_feet" WHERE time >= '2015-08-18T00:00:00Z' AND time <= '2015-08-18T00:30:00Z' AND "location" = 'santa_monica' GROUP BY time(18m)

name: h2o_feet
time                   sample
----                   ------
                           __
2015-08-18T00:06:00Z   2.116 |
2015-08-18T00:12:00Z   2.028 | <------- Randomly-selected points for the first time interval
                           --
                           __
2015-08-18T00:18:00Z   2.126 |
2015-08-18T00:30:00Z   2.051 | <------- Randomly-selected points for the second time interval
                           --
```

#### Issue 2: `SAMPLE()` with `*`

Currently, `SAMPLE(*,<N>)` ignores fields with string values.
See GitHub Issue [#7621](https://github.com/influxdata/influxdb/issues/7621)
for more information.

## TOP()
Returns the largest `N` values in a single [field](/influxdb/v1.1/concepts/glossary/#field).
The field type must be int64 or float64.
```
SELECT TOP(<field_key>[,<tag_keys>],<N>)[,<tag_keys>] FROM <measurement_name> [WHERE <stuff>] [GROUP BY <stuff>]
```

Examples:

* Select the largest three values of `water_level`:

```
> SELECT TOP("water_level",3) FROM "h2o_feet"
name: h2o_feet
--------------
time			               top
2015-08-29T07:18:00Z	 9.957
2015-08-29T07:24:00Z	 9.964
2015-08-29T07:30:00Z	 9.954
```

* Select the largest three values of `water_level` and include the relevant `location` tag in the output:

```
> SELECT TOP("water_level",3),"location" FROM "h2o_feet"
name: h2o_feet
--------------
time			               top	   location
2015-08-29T07:18:00Z	 9.957	 coyote_creek
2015-08-29T07:24:00Z	 9.964	 coyote_creek
2015-08-29T07:30:00Z	 9.954	 coyote_creek
```

* Select the largest value of `water_level` within each tag value of `location`:

```
> SELECT TOP("water_level","location",2) FROM "h2o_feet"
name: h2o_feet
--------------
time			               top	   location
2015-08-29T03:54:00Z	 7.205	 santa_monica
2015-08-29T07:24:00Z	 9.964	 coyote_creek
```

The output shows the top values of `water_level` for each tag value of `location` (`santa_monica` and `coyote_creek`).

> **Note:** Queries with the syntax `SELECT TOP(<field_key>,<tag_key>,<N>)`, where the tag has `X` distinct values, return `N` or `X` field values, whichever is smaller, and each returned point has a unique tag value.
To demonstrate this behavior, see the results of the above example query where `N` equals `3` and `N` equals `1`.

> * `N` = `3`

>
```
> SELECT TOP("water_level","location",3) FROM "h2o_feet"
name: h2o_feet
--------------
time			               top	   location
2015-08-29T03:54:00Z	 7.205	 santa_monica
2015-08-29T07:24:00Z	 9.964	 coyote_creek
```

> InfluxDB returns two values instead of three because the `location` tag has only two values (`santa_monica` and `coyote_creek`).

> * `N` = `1`

>
```
> SELECT TOP("water_level","location",1) FROM "h2o_feet"
name: h2o_feet
--------------
time			               top	   location
2015-08-29T07:24:00Z	 9.964	 coyote_creek
```

> InfluxDB compares the top values of `water_level` within each tag value of `location` and returns the larger value of `water_level`.

* Select the largest two values of `water_level` between August 18, 2015 at 4:00:00 and August 18, 2015 at 4:18:00 for every tag value of `location`:

```
> SELECT TOP("water_level",2) FROM "h2o_feet" WHERE time >= '2015-08-18T04:00:00Z' AND time < '2015-08-18T04:24:00Z' GROUP BY "location"
name: h2o_feet
tags: location=coyote_creek
time			               top
----			               ---
2015-08-18T04:00:00Z	 2.943
2015-08-18T04:06:00Z	 2.831


name: h2o_feet
tags: location=santa_monica
time			               top
----			               ---
2015-08-18T04:06:00Z	 4.055
2015-08-18T04:18:00Z	 4.124
```

* Select the largest two values of `water_level` between August 18, 2015 at 4:00:00 and August 18, 2015 at 4:18:00 in `santa_monica`:

```
> SELECT TOP("water_level",2) FROM "h2o_feet" WHERE time >= '2015-08-18T04:00:00Z' AND time < '2015-08-18T04:24:00Z' AND "location" = 'santa_monica'
name: h2o_feet
--------------
time			               top
2015-08-18T04:06:00Z	 4.055
2015-08-18T04:18:00Z	 4.124
```

Note that in the raw data, `water_level` equals `4.055` at `2015-08-18T04:06:00Z` and at `2015-08-18T04:12:00Z`.
In the case of a tie, InfluxDB returns the value with the earlier timestamp.

# Transformations

## CEILING()
`CEILING()` is not yet functional.

<dt> See GitHub Issue [#5930](https://github.com/influxdata/influxdb/issues/5930) for more information.
</dt>

## CUMULATIVE_SUM()
Returns the cumulative sum of consecutive field values for a single
[field](/influxdb/v1.1/concepts/glossary/#field).
The field type must be int64 or float64.

### Basic CUMULATIVE_SUM() Syntax
```
SELECT CUMULATIVE_SUM(<field_key>) FROM_clause WHERE_clause
```

### Advanced CUMULATIVE_SUM() Syntax
```
SELECT CUMULATIVE_SUM(<function>(<field_key>)) FROM_clause WHERE_clause GROUP BY time(<interval>)[,<tag_key>]
```

Supported functions:
[`COUNT()`](#count),
[`MEAN()`](#mean),
[`MEDIAN()`](#median),
[`MODE()`](#mode),
[`SUM()`](#sum),
[`FIRST()`](#first),
[`LAST()`](#last),
[`MIN()`](#min),
[`MAX()`](#max), and
[`PERCENTILE()`](#percentile).

### Examples

The examples below work with the following subsample of the `NOAA_water_database`
data:
```
> SELECT "water_level" FROM "h2o_feet" WHERE time >= '2015-08-18T00:00:00Z' AND time <= '2015-08-18T00:30:00Z' AND "location" = 'santa_monica'

name: h2o_feet
time                   water_level
----                   -----------
2015-08-18T00:00:00Z   2.064
2015-08-18T00:06:00Z   2.116
2015-08-18T00:12:00Z   2.028
2015-08-18T00:18:00Z   2.126
2015-08-18T00:24:00Z   2.041
2015-08-18T00:30:00Z   2.051
```

#### Example 1: Use cumulative sum on a single time range
```
> SELECT CUMULATIVE_SUM("water_level") FROM "h2o_feet" WHERE time >= '2015-08-18T00:00:00Z' AND time <= '2015-08-18T00:30:00Z' AND "location" = 'santa_monica'

name: h2o_feet
time                   cumulative_sum
----                   --------------
2015-08-18T00:00:00Z   2.064
2015-08-18T00:06:00Z   4.18
2015-08-18T00:12:00Z   6.208
2015-08-18T00:18:00Z   8.334
2015-08-18T00:24:00Z   10.375
2015-08-18T00:30:00Z   12.426
```

The query returns the cumulative sum of `water_level`'s field values.
The second point in the results is the sum of `2.064` and `2.116`, the third point is the sum of `2.064`, `2.116`, and `2.028`, and so on.

#### Example 2: Use cumulative sum with a `GROUP BY time()` clause
```
> SELECT CUMULATIVE_SUM(MEAN("water_level")) FROM "h2o_feet" WHERE time >= '2015-08-18T00:00:00Z' AND time <= '2015-08-18T00:30:00Z' AND "location" = 'santa_monica' GROUP BY time(12m)

name: h2o_feet
time                   cumulative_sum
----                   --------------
2015-08-18T00:00:00Z   2.09
2015-08-18T00:12:00Z   4.167
2015-08-18T00:24:00Z   6.213
```

The query returns the cumulative sum of average `water_level`s that are calculated at 12-minute intervals between `2015-08-18T00:00:00Z` and `2015-08-18T00:30:00Z`.

To get those results, InfluxDB first calculates average `water_level`s at 12-minute intervals.
This step is the same as using the raw `MEAN()` function:
```
> SELECT MEAN("water_level") FROM "h2o_feet" WHERE time >= '2015-08-18T00:00:00Z' AND time <= '2015-08-18T00:30:00Z' AND "location" = 'santa_monica' GROUP BY time(12m)

name: h2o_feet
time                   mean
----                   ----
2015-08-18T00:00:00Z   2.09
2015-08-18T00:12:00Z   2.077
2015-08-18T00:24:00Z   2.0460000000000003
```

Next, InfluxDB calculates the cumulative sum of those averages.
The second point in the final results is the sum of `2.09` and `2.077`
and the third point is the sum of `2.09`, `2.077`, and `2.0460000000000003`.

## DERIVATIVE()
Returns the rate of change for the values in a single [field](/influxdb/v1.1/concepts/glossary/#field) in a [series](/influxdb/v1.1/concepts/glossary/#series).
InfluxDB calculates the difference between chronological field values and converts those results into the rate of change per `unit`.
The `unit` argument is optional and, if not specified, defaults to one second (`1s`).

The basic `DERIVATIVE()` query:
```
SELECT DERIVATIVE(<field_key>, [<unit>]) FROM <measurement_name> [WHERE <stuff>]
```

Valid time specifications for `unit` are:  
`u` microseconds  
`s` seconds  
`m` minutes  
`h` hours  
`d` days  
`w` weeks  

`DERIVATIVE()` also works with a nested function coupled with a `GROUP BY time()` clause.
For queries that include those options, InfluxDB first performs the aggregation, selection, or transformation across the time interval specified in the `GROUP BY time()` clause.
It then calculates the difference between chronological field values and
converts those results into the rate of change per `unit`.
The `unit` argument is optional and, if not specified, defaults to the same
interval as the `GROUP BY time()` interval.


The `DERIVATIVE()` query with an aggregation function and `GROUP BY time()` clause:
```
SELECT DERIVATIVE(AGGREGATION_FUNCTION(<field_key>),[<unit>]) FROM <measurement_name> WHERE <stuff> GROUP BY time(<aggregation_interval>)
```

Examples:

The following examples work with the first six observations of the `water_level` field in the measurement `h2o_feet` with the tag set `location = santa_monica`:
```bash
name: h2o_feet
--------------
time			               water_level
2015-08-18T00:00:00Z	 2.064
2015-08-18T00:06:00Z	 2.116
2015-08-18T00:12:00Z	 2.028
2015-08-18T00:18:00Z	 2.126
2015-08-18T00:24:00Z	 2.041
2015-08-18T00:30:00Z	 2.051
```

* `DERIVATIVE()` with a single argument:  
Calculate the rate of change per one second

```
> SELECT DERIVATIVE("water_level") FROM "h2o_feet" WHERE "location" = 'santa_monica' LIMIT 5
name: h2o_feet
--------------
time			               derivative
2015-08-18T00:06:00Z	 0.00014444444444444457
2015-08-18T00:12:00Z	 -0.00024444444444444465
2015-08-18T00:18:00Z	 0.0002722222222222218
2015-08-18T00:24:00Z	 -0.000236111111111111
2015-08-18T00:30:00Z	 2.777777777777842e-05
```

Notice that the first field value (`0.00014`) in the `derivative` column is **not** `0.052` (the difference between the first two field values in the raw data: `2.116` - `2.604` = `0.052`).
Because the query does not specify the `unit` option, InfluxDB automatically calculates the rate of change per one second, not the rate of change per six minutes.
The calculation of the first value in the `derivative` column looks like this:
<br>
<br>
```
(2.116 - 2.064) / (360s / 1s)
```

The numerator is the difference between chronological field values.
The denominator is the difference between the relevant timestamps in seconds (`2015-08-18T00:06:00Z` - `2015-08-18T00:00:00Z` = `360s`) divided by `unit` (`1s`).
This returns the rate of change per second from `2015-08-18T00:00:00Z` to `2015-08-18T00:06:00Z`.

* `DERIVATIVE()` with two arguments:  
Calculate the rate of change per six minutes

```
> SELECT DERIVATIVE("water_level",6m) FROM "h2o_feet" WHERE "location" = 'santa_monica' LIMIT 5
name: h2o_feet
--------------
time			               derivative
2015-08-18T00:06:00Z	 0.052000000000000046
2015-08-18T00:12:00Z	 -0.08800000000000008
2015-08-18T00:18:00Z	 0.09799999999999986
2015-08-18T00:24:00Z	 -0.08499999999999996
2015-08-18T00:30:00Z	 0.010000000000000231
```

The calculation of the first value in the `derivative` column looks like this:
<br>
<br>
```
(2.116 - 2.064) / (6m / 6m)
```

The numerator is the difference between chronological field values.
The denominator is the difference between the relevant timestamps in minutes (`2015-08-18T00:06:00Z` - `2015-08-18T00:00:00Z` = `6m`) divided by `unit` (`6m`).
This returns the rate of change per six minutes from `2015-08-18T00:00:00Z` to `2015-08-18T00:06:00Z`.

* `DERIVATIVE()` with two arguments:  
Calculate the rate of change per 12 minutes

```
> SELECT DERIVATIVE("water_level",12m) FROM "h2o_feet" WHERE "location" = 'santa_monica' LIMIT 5
name: h2o_feet
--------------
time			               derivative
2015-08-18T00:06:00Z	 0.10400000000000009
2015-08-18T00:12:00Z	 -0.17600000000000016
2015-08-18T00:18:00Z	 0.19599999999999973
2015-08-18T00:24:00Z	 -0.16999999999999993
2015-08-18T00:30:00Z	 0.020000000000000462
```

The calculation of the first value in the `derivative` column looks like this:
<br>
<br>
```
(2.116 - 2.064 / (6m / 12m)
```

The numerator is the difference between chronological field values.
The denominator is the difference between the relevant timestamps in minutes (`2015-08-18T00:06:00Z` - `2015-08-18T00:00:00Z` = `6m`) divided by `unit` (`12m`).
This returns the rate of change per 12 minutes from `2015-08-18T00:00:00Z` to `2015-08-18T00:06:00Z`.

> **Note:** Specifying `12m` as the `unit` **does not** mean that InfluxDB calculates the rate of change for every 12 minute interval of data.
Instead, InfluxDB calculates the rate of change per 12 minutes for each interval of valid data.

* `DERIVATIVE()` with one argument, a function, and a `GROUP BY time()` clause:  
Select the `MAX()` value at 12 minute intervals and calculate the rate of change per 12 minutes

```
> SELECT DERIVATIVE(MAX("water_level")) FROM "h2o_feet" WHERE "location" = 'santa_monica' AND time >= '2015-08-18T00:00:00Z' AND time < '2015-08-18T00:36:00Z' GROUP BY time(12m)
name: h2o_feet
--------------
time			               derivative
2015-08-18T00:12:00Z	 0.009999999999999787
2015-08-18T00:24:00Z	 -0.07499999999999973
```

To get those results, InfluxDB first aggregates the data by calculating the `MAX()` `water_level` at the time interval specified in the `GROUP BY time()` clause (`12m`).
Those results look like this:
```bash
name: h2o_feet
--------------
time			               max
2015-08-18T00:00:00Z	 2.116
2015-08-18T00:12:00Z	 2.126
2015-08-18T00:24:00Z	 2.051
```

Second, InfluxDB calculates the rate of change per `12m` (the same interval as the `GROUP BY time()` interval) to get the results in the `derivative` column above.
The calculation of the first value in the `derivative` column looks like this:
<br>
<br>
```
(2.126 - 2.116) / (12m / 12m)
```

The numerator is the difference between chronological field values.
The denominator is the difference between the relevant timestamps in minutes (`2015-08-18T00:12:00Z` - `2015-08-18T00:00:00Z` = `12m`) divided by `unit` (`12m`).
This returns rate of change per 12 minutes for the aggregated data from `2015-08-18T00:00:00Z` to `2015-08-18T00:12:00Z`.

* `DERIVATIVE()` with two arguments, a function, and a `GROUP BY time()` clause:  
Aggregate the data to 18 minute intervals and calculate the rate of change per six minutes

```
> SELECT DERIVATIVE(SUM("water_level"),6m) FROM "h2o_feet" WHERE "location" = 'santa_monica' AND time >= '2015-08-18T00:00:00Z' AND time < '2015-08-18T00:36:00Z' GROUP BY time(18m)
name: h2o_feet
--------------
time			               derivative
2015-08-18T00:18:00Z	 0.0033333333333332624
```

To get those results, InfluxDB first aggregates the data by calculating the `SUM()` of `water_level` at the time interval specified in the `GROUP BY time()` clause (`18m`).
The aggregated results look like this:
```bash
name: h2o_feet
--------------
time			               sum
2015-08-18T00:00:00Z	 6.208
2015-08-18T00:18:00Z	 6.218
```

Second, InfluxDB calculates the rate of change per `unit` (`6m`) to get the results in the `derivative` column above.
The calculation of the first value in the `derivative` column looks like this:
<br>
<br>
```
(6.218 - 6.208) / (18m / 6m)
```

The numerator is the difference between chronological field values.
The denominator is the difference between the relevant timestamps in minutes (`2015-08-18T00:18:00Z` - `2015-08-18T00:00:00Z` = `18m`) divided by `unit` (`6m`).
This returns the rate of change per six minutes for the aggregated data from `2015-08-18T00:00:00Z` to `2015-08-18T00:18:00Z`.

## DIFFERENCE()
Returns the difference between consecutive chronological values in a single [field](/influxdb/v1.1/concepts/glossary/#field).
The field type must be int64 or float64.

The basic `DIFFERENCE()` query:
```
SELECT DIFFERENCE(<field_key>) FROM <measurement_name> [WHERE <stuff>]
```

The `DIFFERENCE()` query with a nested function and a `GROUP BY time()` clause:
```
SELECT DIFFERENCE(<function>(<field_key>)) FROM <measurement_name> WHERE <stuff> GROUP BY time(<time_interval>)
```

Functions that work with `DIFFERENCE()` include
[`COUNT()`](/influxdb/v1.1/query_language/functions/#count),
[`MEAN()`](/influxdb/v1.1/query_language/functions/#mean),
[`MEDIAN()`](/influxdb/v1.1/query_language/functions/#median),
[`SUM()`](/influxdb/v1.1/query_language/functions/#sum),
[`FIRST()`](/influxdb/v1.1/query_language/functions/#first),
[`LAST()`](/influxdb/v1.1/query_language/functions/#last),
[`MIN()`](/influxdb/v1.1/query_language/functions/#min),
[`MAX()`](/influxdb/v1.1/query_language/functions/#max), and
[`PERCENTILE()`](/influxdb/v1.1/query_language/functions/#percentile).

Examples:

The following examples focus on the field `water_level` in `santa_monica`
between `2015-08-18T00:00:00Z` and `2015-08-18T00:36:00Z`:
```
> SELECT "water_level" FROM "h2o_feet" WHERE "location"='santa_monica' AND time >= '2015-08-18T00:00:00Z' AND time <= '2015-08-18T00:36:00Z'
name: h2o_feet
--------------
time			                water_level
2015-08-18T00:00:00Z	  2.064
2015-08-18T00:06:00Z	  2.116
2015-08-18T00:12:00Z	  2.028
2015-08-18T00:18:00Z	  2.126
2015-08-18T00:24:00Z	  2.041
2015-08-18T00:30:00Z	  2.051
2015-08-18T00:36:00Z	  2.067
```

* Calculate the difference between `water_level` values:

```
> SELECT DIFFERENCE("water_level") FROM "h2o_feet" WHERE "location"='santa_monica' AND time >= '2015-08-18T00:00:00Z' AND time <= '2015-08-18T00:36:00Z'
name: h2o_feet
--------------
time			                difference
2015-08-18T00:06:00Z	  0.052000000000000046
2015-08-18T00:12:00Z	  -0.08800000000000008
2015-08-18T00:18:00Z	  0.09799999999999986
2015-08-18T00:24:00Z	  -0.08499999999999996
2015-08-18T00:30:00Z	  0.010000000000000231
2015-08-18T00:36:00Z	  0.016000000000000014
```

The first value in the `difference` column is `2.116 - 2.064`, and the second
value in the `difference` column is `2.028 - 2.116`.
Please note that the extra decimal places are the result of floating point
inaccuracies.

* Select the minimum `water_level` values at 12 minute intervals and calculate
the difference between those values:

```
> SELECT DIFFERENCE(MIN("water_level")) FROM "h2o_feet" WHERE "location"='santa_monica' AND time >= '2015-08-18T00:00:00Z' AND time <= '2015-08-18T00:36:00Z' GROUP BY time(12m)
name: h2o_feet
--------------
time			                difference
2015-08-18T00:12:00Z	  -0.03600000000000003
2015-08-18T00:24:00Z	  0.0129999999999999
2015-08-18T00:36:00Z	  0.026000000000000245
```

To get the values in the `difference` column, InfluxDB first selects the `MIN()`
values at 12 minute intervals:
```
> SELECT MIN("water_level") FROM "h2o_feet" WHERE "location"='santa_monica' AND time >= '2015-08-18T00:00:00Z' AND time <= '2015-08-18T00:36:00Z' GROUP BY time(12m)
name: h2o_feet
--------------
time			                min
2015-08-18T00:00:00Z  	2.064
2015-08-18T00:12:00Z  	2.028
2015-08-18T00:24:00Z  	2.041
2015-08-18T00:36:00Z  	2.067
```

It then uses those values to calculate the difference between chronological
values; the first value in the `difference` column is `2.028 - 2.064`.

## ELAPSED()
Returns the difference between subsequent timestamps in a single
[field](/influxdb/v1.1/concepts/glossary/#field).
The `unit` argument is an optional
[duration literal](/influxdb/v1.1/query_language/spec/#durations)
and, if not specified, defaults to one nanosecond.

```
SELECT ELAPSED(<field_key>, <unit>) FROM <measurement_name> [WHERE <stuff>]
```

Examples:

* Calculate the difference (in nanoseconds) between the timestamps in the field
`h2o_feet`:

```
> SELECT ELAPSED("water_level") FROM "h2o_feet" WHERE "location" = 'santa_monica' AND time >= '2015-08-18T00:00:00Z' AND time <= '2015-08-18T00:24:00Z'
name: h2o_feet
--------------
time			                elapsed
2015-08-18T00:06:00Z	  360000000000
2015-08-18T00:12:00Z	  360000000000
2015-08-18T00:18:00Z	  360000000000
2015-08-18T00:24:00Z	  360000000000
```

* Calculate the number of one minute intervals between the timestamps in the
field `h2o_feet`:

```
> SELECT ELAPSED("water_level",1m) FROM "h2o_feet" WHERE "location" = 'santa_monica' AND time >= '2015-08-18T00:00:00Z' AND time <= '2015-08-18T00:24:00Z'
name: h2o_feet
--------------
time			                elapsed
2015-08-18T00:06:00Z	  6
2015-08-18T00:12:00Z	  6
2015-08-18T00:18:00Z	  6
2015-08-18T00:24:00Z	  6
```

> **Note:** InfluxDB returns `0` if `unit` is greater than the difference
between the timestamps.
For example, the timestamps in `h2o_feet` occur at six minute intervals.
If the query asks for the number of one hour intervals between the
timestamps, InfluxDB returns `0`:
>
```
> SELECT ELAPSED("water_level",1h) FROM "h2o_feet" WHERE "location" = 'santa_monica' AND time >= '2015-08-18T00:00:00Z' AND time <= '2015-08-18T00:24:00Z'
name: h2o_feet
--------------
time			                elapsed
2015-08-18T00:06:00Z	  0
2015-08-18T00:12:00Z	  0
2015-08-18T00:18:00Z	  0
2015-08-18T00:24:00Z	  0
```

## FLOOR()
`FLOOR()` is not yet functional.

<dt> See GitHub Issue [#5930](https://github.com/influxdb/influxdb/issues/5930) for more information.
</dt>

## HISTOGRAM()
`HISTOGRAM()` is not yet functional.

<dt> See GitHub Issue [#5930](https://github.com/influxdb/influxdb/issues/5930) for more information.
</dt>

## MOVING_AVERAGE()
Returns the moving average across a `window` of consecutive chronological field values for a single [field](/influxdb/v1.1/concepts/glossary/#field).
The field type must be int64 or float64.

The basic `MOVING_AVERAGE()` query:
```
SELECT MOVING_AVERAGE(<field_key>,<window>) FROM <measurement_name> [WHERE <stuff>]
```

The `MOVING_AVERAGE()` query with a nested function and a `GROUP BY time()` clause:
```
SELECT MOVING_AVERAGE(<function>(<field_key>),<window>) FROM <measurement_name> WHERE <stuff> GROUP BY time(<time_interval>)
```

Functions that work with `MOVING_AVERAGE()` include
[`COUNT()`](/influxdb/v1.1/query_language/functions/#count),
[`MEAN()`](/influxdb/v1.1/query_language/functions/#mean),
[`MEDIAN()`](/influxdb/v1.1/query_language/functions/#median),
[`SUM()`](/influxdb/v1.1/query_language/functions/#sum),
[`FIRST()`](/influxdb/v1.1/query_language/functions/#first),
[`LAST()`](/influxdb/v1.1/query_language/functions/#last),
[`MIN()`](/influxdb/v1.1/query_language/functions/#min),
[`MAX()`](/influxdb/v1.1/query_language/functions/#max), and
[`PERCENTILE()`](/influxdb/v1.1/query_language/functions/#percentile).

Examples:

The following examples focus on the field `water_level` in `santa_monica`
between `2015-08-18T00:00:00Z` and `2015-08-18T00:36:00Z`:
```
> SELECT "water_level" FROM "h2o_feet" WHERE "location" = 'santa_monica' AND time >= '2015-08-18T00:00:00Z' AND time <= '2015-08-18T00:36:00Z'
name: h2o_feet
--------------
time			                water_level
2015-08-18T00:00:00Z	  2.064
2015-08-18T00:06:00Z	  2.116
2015-08-18T00:12:00Z	  2.028
2015-08-18T00:18:00Z	  2.126
2015-08-18T00:24:00Z	  2.041
2015-08-18T00:30:00Z	  2.051
2015-08-18T00:36:00Z	  2.067
```

* Calculate the moving average across every 2 field values:

```
> SELECT MOVING_AVERAGE("water_level",2) FROM "h2o_feet" WHERE "location" = 'santa_monica' AND time >= '2015-08-18T00:00:00Z' AND time <= '2015-08-18T00:36:00Z'
name: h2o_feet
--------------
time			                moving_average
2015-08-18T00:06:00Z	  2.09
2015-08-18T00:12:00Z	  2.072
2015-08-18T00:18:00Z	  2.077
2015-08-18T00:24:00Z	  2.0835
2015-08-18T00:30:00Z	  2.0460000000000003
2015-08-18T00:36:00Z	  2.059
```

The first value in the `moving_average` column is the average of `2.064` and
`2.116`, the second value in the `moving_average` column is the average of
`2.116` and `2.028`.

* Select the minimum `water_level` at 12 minute intervals and calculate the
moving average across every 2 field values:

```
> SELECT MOVING_AVERAGE(MIN("water_level"),2) FROM "h2o_feet" WHERE "location" = 'santa_monica' AND time >= '2015-08-18T00:00:00Z' AND time <= '2015-08-18T00:36:00Z' GROUP BY time(12m)
name: h2o_feet
--------------
time			                moving_average
2015-08-18T00:12:00Z	  2.0460000000000003
2015-08-18T00:24:00Z	  2.0345000000000004
2015-08-18T00:36:00Z	  2.0540000000000003
```

To get those results, InfluxDB first selects the `MIN()` `water_level` for every
12 minute interval:
```
name: h2o_feet
--------------
time			                min
2015-08-18T00:00:00Z	  2.064
2015-08-18T00:12:00Z	  2.028
2015-08-18T00:24:00Z	  2.041
2015-08-18T00:36:00Z	  2.067
```

It then uses those values to calculate the moving average across every 2 field
values; the first result in the `moving_average` column the average of `2.064`
and `2.028`, and the second result is the average of `2.028` and `2.041`.

## NON_NEGATIVE_DERIVATIVE()
Returns the non-negative rate of change for the values in a single [field](/influxdb/v1.1/concepts/glossary/#field) in a [series](/influxdb/v1.1/concepts/glossary/#series).
InfluxDB calculates the difference between chronological field values and converts those results into the rate of change per `unit`.
The `unit` argument is optional and, if not specified, defaults to one second (`1s`).

The basic `NON_NEGATIVE_DERIVATIVE()` query:
```
SELECT NON_NEGATIVE_DERIVATIVE(<field_key>, [<unit>]) FROM <measurement_name> [WHERE <stuff>]
```

Valid time specifications for `unit` are:  
`u` microseconds  
`s` seconds  
`m` minutes  
`h` hours  
`d` days  
`w` weeks  

`NON_NEGATIVE_DERIVATIVE()` also works with a nested function coupled with a `GROUP BY time()` clause.
For queries that include those options, InfluxDB first performs the aggregation, selection, or transformation across the time interval specified in the `GROUP BY time()` clause.
It then calculates the difference between chronological field values and
converts those results into the rate of change per `unit`.
The `unit` argument is optional and, if not specified, defaults to the same
interval as the `GROUP BY time()` interval.

The `NON_NEGATIVE_DERIVATIVE()` query with an aggregation function and `GROUP BY time()` clause:
```
SELECT NON_NEGATIVE_DERIVATIVE(AGGREGATION_FUNCTION(<field_key>),[<unit>]) FROM <measurement_name> WHERE <stuff> GROUP BY time(<aggregation_interval>)
```

See [`DERIVATIVE()`](/influxdb/v1.1/query_language/functions/#derivative) for example queries.
All query results are the same for `DERIVATIVE()` and `NON_NEGATIVE_DERIVATIVE` except that `NON_NEGATIVE_DERIVATIVE()` returns only the positive values.

## Include multiple functions in a single query
Separate multiple functions in one query with a `,`.

Calculate the [minimum](/influxdb/v1.1/query_language/functions/#min) `water_level` and the [maximum](/influxdb/v1.1/query_language/functions/#max) `water_level` with a single query:
```
> SELECT MIN("water_level"), MAX("water_level") FROM "h2o_feet"
name: h2o_feet
--------------
time			               min	   max
1970-01-01T00:00:00Z	 -0.61	 9.964
```

## Change the value reported for intervals with no data with `fill()`
By default, queries with an InfluxQL function report `null` values for intervals with no data.
Append `fill()` to the end of your query to alter that value.
For a complete discussion of `fill()`, see [Data Exploration](/influxdb/v1.1/query_language/data_exploration/#group-by-time-intervals-and-fill).

> **Note:** `fill()` works differently with `COUNT()`.
See [the documentation on `COUNT()`](/influxdb/v1.1/query_language/functions/#count-and-controlling-the-values-reported-for-intervals-with-no-data) for a function-specific use of `fill()`.

## Rename the output column's title with `AS`

By default, queries that include a function output a column that has the same name as that function.
If you'd like a different column name change it with an `AS` clause.

Before:
```
> SELECT MEAN("water_level") FROM "h2o_feet"
name: h2o_feet
--------------
time			               mean
1970-01-01T00:00:00Z	 4.442107025822521
```

After:
```
> SELECT MEAN("water_level") AS "dream_name" FROM "h2o_feet"
name: h2o_feet
--------------
time			               dream_name
1970-01-01T00:00:00Z	 4.442107025822521
```

# Predictors

## HOLT_WINTERS()
Returns N number of predicted values for a single
[field](/influxdb/v1.1/concepts/glossary/#field) using the
[Holt-Winters](https://www.otexts.org/fpp/7/5) seasonal method.
The seasonal adjustment of the predicted values is optional.
The field must be of type int64 or float64.

Use `HOLT_WINTERS()` to:

* Predict when data values will cross a given threshold.
* Compare predicted values with actual values to detect anomalies in your data.

Returns only the predicted values:
```
SELECT HOLT_WINTERS(FUNCTION(<field_key>),<N>,<S>) FROM <measurement_name> WHERE <stuff> GROUP BY time(<interval>)[,<stuff>]
```

Returns the fitted values and the predicted values:
```
SELECT HOLT_WINTERS_WITH_FIT(FUNCTION(<field_key>),<N>,<S>) FROM <measurement_name> WHERE <stuff> GROUP BY time(<interval>)[,<stuff>]
```

Syntax explanation:

`N` is the number of predicted values.
Those values occur at the same interval as the `GROUP BY time()` interval.
If your `GROUP BY time()` interval is `6m` and `N` is `3` you'll
receive three predicted values that are each six minutes apart.

`S` is the seasonal pattern parameter.
The parameter delimits the length of a seasonal pattern according to the
`GROUP BY time()` interval.
If your `GROUP BY time()` interval is `2m` and `S` is `3`, then the
seasonal pattern occurs every six minutes, that is, every three data points.
If you do not want to seasonally adjust your predicted values, set `S` to `0`
or `1.`

Notice that `HOLT_WINTERS()` requires the use of another InfluxQL function and a
`GROUP BY time()` clause.
That ensures that the function receives data that occur at consistent time
intervals.

> **Note:** In some cases, users may receive fewer predicted points than
requested by the `N` parameter.
That behavior occurs when the math becomes unstable and cannot forecast more
points.
It implies that either `HOLT_WINTERS()` is not suited for the dataset or that
the seasonal adjustment parameter is invalid and is confusing the algorithm.

**Example:**

In the following example we'll apply `HOLT_WINTERS()` to the data in the
`NOAA_water_database`, and we'll show the results using
[Chronograf](https://docs.influxdata.com/chronograf/v1.1/) visualizations.

**Raw Data**

Our query focuses on the raw `water_level` data in `santa_monica` between August
22, 2015 at 22:12 and August 28, 2015 at 03:00:
```
SELECT "water_level" FROM "h2o_feet" WHERE "location"='santa_monica' AND time >= '2015-08-22 22:12:00' AND time <= '2015-08-28 03:00:00'
```

![Raw Data](/img/influxdb/hw_raw_data.png)

**Step 1: Create a Query to Match the Trends of the Raw Data**

We start by writing a `GROUP BY time()` query to match the general trends of the
raw `water_level` data.
We could do this with almost any InfluxQL function, but here we use
[`FIRST()`](#first).

Focusing on the `GROUP BY time()` arguments in the query below, the first
argument (`379m`) matches the length of time that occurs between each peak and
trough of the `water_level` data.
The second argument (`348m`) is the
[offset interval](/influxdb/v1.1/query_language/data_exploration/#advanced-group-by-time-syntax).
The offset interval alters InfluxDB's default `GROUP BY time()` boundaries to
match the time range of the raw data.

The green line shows the results of the query:
```
SELECT first("water_level") FROM "h2o_feet" WHERE "location"='santa_monica' and time >= '2015-08-22 22:12:00' and time <= '2015-08-28 03:00:00' GROUP BY time(379m,348m)
```

![First step](/img/influxdb/hw_first_step.png)

**Step 2: Determine the Seasonal Pattern**

Now that we have a query that matches the trends of the raw `water_level` data
it's time to determine the seasonal pattern in the data.
We'll use this information when we implement the `HOLT_WINTERS()` function in
the next step.

Focusing on the green line in the chart below, notice the pattern that repeats
about every 25 hours and 15 minutes.
That's four data points per season, so `4` is our seasonal pattern argument.

![Second step](/img/influxdb/hw_second_step.png)

**Step 3: Include the HOLT_WINTERS() function**

Now we add the `HOLT_WINTERS()` function to our query.
Here, we'll use `HOLT_WINTERS_WITH_FIT()` so that the query results show both
the fitted values and the predicted values.

Focusing on the `HOLT_WINTERS_WITH_FIT()` arguments in the query below, the
first argument (`10`) tells the function to predict 10 data points.
Each point will be `379m` apart, the same interval as the first argument in the
`GROUP BY time()` clause.

The second argument (`4`) is the seasonal pattern that we determined in the
previous step.

```
SELECT holt_winters_with_fit(first(water_level),10,4) FROM h2o_feet where location='santa_monica' and time >= '2015-08-22 22:12:00' and time <= '2015-08-28 03:00:00' group by time(379m,348m)
```

![Third step](/img/influxdb/hw_third_step.png)

And that's it!
We've successfully predicted water levels in Santa Monica between August 28,
2015 at 04:32 and August 28, 2015 at 13:23.

## Common Issues with Functions

### Time ranges after now
TODO

### Aggregation Functions

#### Issue 1: Understanding the returned timestamp
TODO
Aggregation functions return epoch 0 (`1970-01-01T00:00:00Z`) as the timestamp unless you specify a lower bound on the time range.
Then they return the lower bound as the timestamp.

#### Issue 2: Mixing aggregation functions with non-aggregates
TODO

#### Issue 3 (maybe): Getting slightly different results
TODO
Executing `mean()` on the same set of float64 points may yield slightly
different results.
InfluxDB does not sort points before it applies the function which results in
those small discrepancies.

### Selector Functions
TODO
sorts series lexicographically.
