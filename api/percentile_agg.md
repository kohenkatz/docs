# Percentile aggregate functions
The functions related to percentile aggregates.

## percentile_agg

```sql
percentile_agg(
    value DOUBLE PRECISION
) RETURNS UddSketch
```

This is the default percentile aggregation function. It uses the [UddSketch algorithm](https://github.com/timescale/timescale-analytics/blob/main/docs/uddsketch.md) with 200 buckets and an initial maximum error of 0.001. This is appropriate for most common use cases of percentile approximation. For more advanced use of percentile approximation algorithms, see [advanced usage](https://github.com/timescale/timescale-analytics/blob/main/docs/percentile_approximation.md#advanced-usage). This is the aggregation step of [two-step aggregates](https://github.com/timescale/timescale-analytics/blob/main/docs/two-step_aggregation.md), it is usually used with the [approx_percentile()](#approx_percentile) accessor function to extract an approximate percentile, however it is in a form that can be re-aggregated using the [summary form](#summary-form) of the function and any of the other [accessor functions](#accessor-functions).

### Required arguments

|Name|Type|Description|
|---|---|---|
|`value`|`DOUBLE PRECISION`|Column to aggregate|

### Returns

|Column|Type|Description|
|---|---|---|
|`percentile_agg`|`UddSketch`|A UddSketch object which may be passed to other percentile approximation APIs|

The `percentile_agg` function uses the UddSketch algorithm, so it returns the UddSketch data structure for use in further calls.

### Sample usage
Get the approximate first percentile using the `percentile_agg()` point form plus the [`approx_percentile`](#approx_percentile) accessor function.

```SQL
SELECT
    approx_percentile(0.01, percentile_agg(data))
FROM generate_series(0, 100) data;
```
```output
approx_percentile
-------------------
             0.999
```

They are often used to create continuous aggregates, after which you can use multiple accessors for [retrospective analysis](https://github.com/timescale/timescale-analytics/blob/main/docs/two-step_aggregation.md#retrospective-analysis-over-downsampled-data).

```SQL
CREATE MATERIALIZED VIEW foo_hourly
WITH (timescaledb.continuous)
AS SELECT
    time_bucket('1 h'::interval, ts) as bucket,
    percentile_agg(value) as pct_agg
FROM foo
GROUP BY 1;
```
---

## rollup (summary form)

```SQL
rollup(
    sketch uddsketch
) RETURNS UddSketch
```

This combines multiple outputs from the point form of the `percentile_agg()` function. This is especially useful for re-aggregation in a continuous aggregate. For example, bucketing by a larger `time_bucket`, or re-grouping on other dimensions included in an aggregation.

### Required arguments

|Name|Type|Description|
|---|---|---|
|`sketch`|`UddSketch`|The already constructed uddsketch from a previous `percentile_agg` call|

### Returns

|Column|Type|Description|
|---|---|---|
|`uddsketch`|`UddSketch`|A UddSketch object which may be passed to other UddSketch APIs|

Because the `percentile_agg` function uses the [UddSketch algorithm](/docs/uddsketch.md), `rollup` returns the UddSketch data structure for use in further calls.

### Sample usage
Using the continuous aggregate from the [point form example](#point-form-examples), use the `rollup` function to re-aggregate the results from the `foo_hourly` view and the `approx_percentile` accessor function to get the 95th and 99th percentiles over each day:

```SQL
SELECT
    time_bucket('1 day'::interval, bucket) as bucket,
    approx_percentile(0.95, rollup(pct_agg)) as p95,
    approx_percentile(0.99, rollup(pct_agg)) as p99
FROM foo_hourly
GROUP BY 1;
```

## error

```SQL
error(sketch UddSketch) RETURNS DOUBLE PRECISION
```

This returns the maximum relative error that a percentile estimate will have (relative to the correct value). This means the actual value will fall in the range defined by `approx_percentile(sketch) +/- approx_percentile(sketch)*error(sketch)`.

### Required arguments

|Name|Type|Description|
|---|---|---|
|`sketch`|`UddSketch`|The sketch to determine the error of, usually from a `percentile_agg` call|

### Returns

|Column|Type|Description|
|---|---|---|
|`error`|`DOUBLE PRECISION`|The maximum relative error of any percentile estimate|

### Sample usage

```SQL
SELECT error(percentile_agg(data))
FROM generate_series(0, 100) data;
```
```output
 error
-------
 0.001
```

## mean

```SQL
mean(sketch UddSketch) RETURNS DOUBLE PRECISION
```

Get the exact average of all the values in the percentile estimate. (Percentiles returned are estimates, the average is exact.

### Required arguments

|Name|Type|Description|
|---|---|---|
| `sketch` | `UddSketch` |  The sketch to extract the mean value from, usually from a [`percentile_agg()`](#aggregate-functions) call. |

### Returns

|Column|Type|Description|
|---|---|---|
| `mean` | `DOUBLE PRECISION` | The average of the values in the percentile estimate. |

### Sample usage

```SQL
SELECT mean(percentile_agg(data))
FROM generate_series(0, 100) data;
```
```output
 mean
------
 50
```

## num_vals

```SQL
num_vals(sketch UddSketch) RETURNS DOUBLE PRECISION
```

Get the number of values contained in a percentile estimate.

### Required arguments

|Name|Type|Description|
|---|---|---|
|`sketch`|`UddSketch`| The sketch to extract the number of values from, usually from a `percentile_agg` call|

### Returns

|Column|Type|Description|
|---|---|---|
|`uddsketch_count`|`DOUBLE PRECISION`|The number of values in the percentile estimate|

### Sample usage

```SQL
SELECT num_vals(percentile_agg(data))
FROM generate_series(0, 100) data;
```
```output
 num_vals
-----------
       101
```

## approx_percentile

```SQL
approx_percentile(
    percentile DOUBLE PRECISION,
    sketch  uddsketch
) RETURNS DOUBLE PRECISION
```

Get the approximate value at a percentile from a percentile estimate.

### Required arguments

|Name|Type|Description|
|---|---|---|
|`approx_percentile`|`DOUBLE PRECISION`|The desired percentile (0.0-1.0) to approximate|
|`sketch`|`UddSketch`|The sketch to compute the approx_percentile on, usually from a `percentile_agg`|

### Returns

|Column|Type|Description|
|---|---|---|
|`approx_percentile`|`DOUBLE PRECISION`|The estimated value at the requested percentile|

### Sample usage

```SQL
SELECT
    approx_percentile(0.01, percentile_agg(data))
FROM generate_series(0, 100) data;
```
```output
approx_percentile
-------------------
             0.999
```

## approx_percentile_rank

```SQL
approx_percentile_rank(
    value DOUBLE PRECISION,
    sketch UddSketch
) RETURNS UddSketch
```

Estimate what percentile a given value would be located at in a UddSketch.

### Required arguments

|Name|Type|Description|
|---|---|---|
|`value`|`DOUBLE PRECISION`|The value to estimate the percentile of|
|`sketch`|`UddSketch`|The sketch to compute the percentile on.

### Returns

|Column|Type|Description|
|---|---|---|
|`approx_percentile_rank`|`DOUBLE PRECISION`|The estimated percentile associated with the provided value|

### Sample usage

```SQL
SELECT
    approx_percentile_rank(99, percentile_agg(data))
FROM generate_series(0, 100) data;
```
```output
 approx_percentile_rank
----------------------------
         0.9851485148514851
```