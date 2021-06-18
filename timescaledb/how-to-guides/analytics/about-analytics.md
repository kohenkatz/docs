# Analytics
Timescale Analytics is a PostreSQL extension containing a specialized set of
functions that allow you to to analyze time-series data. You can use it to
analyze anything you have stored as time-series data, including IoT devices, IT
systems, marketing analytics, user behavior, financial metrics, and
cryptocurrency. Timescale Analytics uses [Promscale][doc-promscale], an
open-source analytics platform, for Prometheus monitoring data.

Timescale Analytics allows you to perform critical time-series queries quickly,
analyze time-series data, and extract meaningful information. It aims to
identify, build, and combine all of the functionality SQL needs to perform
time-series analysis into a single extension.

## Tools for graphing
Timescale Analytics brings graphing functions to the database. This allows you
to choose your graphing front-end based on how well it does graphing, not on how
well it does data analytics. It also allows you to run queries that stay
consistent across all front-end tools and consumers of your data. Additionally,
by doing all the graphing work in the database, you need to send a much smaller
number of data points over the network.

## Simplifying queries
SQL queries can get long, especially if you have multiple layers of aggregation
and function-calls. There are many scenarios where it's possible to write a
query in native SQL, but the resulting code is relatively complicated to write,
and to understand. Timescale Analytics can greatly simplify your queries by
using a two-step calling convention.

For example, a typical Timescale Analytics query to get the time-weighted
average of a set of values could look like this: ```sql SELECT
average(time_weight('LOCF', value)) as time_weighted_average FROM foo; ```

The first step in this query is to call the inner aggregate function, such as
`time_weighted_average`. The second step is to call the accessor function, such
as `average`.

This makes it easier to construct your queries, because it distinguishes the
parameters, and makes it clear which aggregates are being re-aggregated or
stacked. Additionally, because this query syntax is used in all Timescale
Analytics queries, when you are used to it, you can use it to construct more and
more complicated queries.

## Analytics features
Timescale Analytics features are developed in the open. As features are developed they are categorized as experimental, beta, stable, or deprecated. This table uses these icons:

* 🧪 Experimental: under active development, not supported. See the [Analytics repository][gh-docs] for more information and documentation.
* 🆕 Beta: In testing, and preparing to be released, not supported. See the [Analytics repository][gh-docs] for more information and documentation.
* ✅ Stable: Features that are fully ready for production, and supported. See the official documentation.
* ❌ Deprecated: Previously stable features that have been replaced by newer features, or that have deprecated dependencies, limited support. See the official documentation.

|Feature|Status|Notes|More information|
|-------|------|-----|----------------|
|ASAP smoothing|🧪|A data smoothing algorithm to generate human readable graphs|[https://github.com/timescale/timescale-analytics/blob/main/docs/asap.md][]|
|Hyperloglog|🧪|An approximate COUNT DISTINCT based on hashing|[https://github.com/timescale/timescale-analytics/blob/main/docs/hyperloglog.md][]|
|LTTB|🧪|A downsample method that preserves visual similarity.|[https://github.com/timescale/timescale-analytics/blob/main/docs/lttb.md][]|
|Percentile Approximation|✅|A simple percentile approximation interface|[/how-to-guides/analytics/approximate_percentile]|
|Time-weighted averages|✅|An aggregate that weights each value|[/how-to-guides/analytics/time-weighted-averages.md][]|

## Contribute to Timescale Analytics
We want and need your feedback! What are the frustrating parts of analyzing
time-series data? What takes far more code than you feel it should? What runs
slowly, or only runs quickly after many rewrites? We want to solve
community-wide problems and incorporate as much feedback as possible.

*   Join the [discussion][gh-discussions].
*   Check out the [proposed features][gh-proposed].
*   Explore the current [feature requests][gh-requests].
*   Add your own [feature request][gh-newissue].

https://github.com/timescale/timescale-analytics/tree/main/docs
[doc-promscale]: /tutorials/promscale
[gh-discussions]: https://github.com/timescale/timescale-analytics/discussions
[gh-proposed]: https://github.com/timescale/timescale-analytics/labels/proposed-feature
[gh-requests]: https://github.com/timescale/timescale-analytics/labels/feature-request
[gh-newissue]: https://github.com/timescale/timescale-analytics/issues/new?assignees=&labels=feature-request&template=feature-request.md&title=
