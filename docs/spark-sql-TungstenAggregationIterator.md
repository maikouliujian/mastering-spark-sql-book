# TungstenAggregationIterator

`TungstenAggregationIterator` is a [AggregationIterator](spark-sql-AggregationIterator.md) that is used when the [HashAggregateExec](physical-operators/HashAggregateExec.md) aggregate physical operator is executed (to process [UnsafeRow](UnsafeRow.md)s per partition and calculate aggregations).

`TungstenAggregationIterator` prefers hash-based aggregation (before <<switchToSortBasedAggregation, switching to sort-based aggregation>>).

```text
val q = spark.range(10).
  groupBy('id % 2 as "group").
  agg(sum("id") as "sum")
val execPlan = q.queryExecution.sparkPlan
scala> println(execPlan.numberedTreeString)
00 HashAggregate(keys=[(id#0L % 2)#11L], functions=[sum(id#0L)], output=[group#3L, sum#7L])
01 +- HashAggregate(keys=[(id#0L % 2) AS (id#0L % 2)#11L], functions=[partial_sum(id#0L)], output=[(id#0L % 2)#11L, sum#13L])
02    +- Range (0, 10, step=1, splits=8)

import org.apache.spark.sql.execution.aggregate.HashAggregateExec
val hashAggExec = execPlan.asInstanceOf[HashAggregateExec]
val hashAggExecRDD = hashAggExec.execute

// MapPartitionsRDD is in private[spark] scope
// Use :paste -raw for the following helper object
package org.apache.spark
object AccessPrivateSpark {
  import org.apache.spark.rdd.RDD
  def mapPartitionsRDD[T](hashAggExecRDD: RDD[T]) = {
    import org.apache.spark.rdd.MapPartitionsRDD
    hashAggExecRDD.asInstanceOf[MapPartitionsRDD[_, _]]
  }
}
// END :paste -raw

import org.apache.spark.AccessPrivateSpark
val mpRDD = AccessPrivateSpark.mapPartitionsRDD(hashAggExecRDD)
val f = mpRDD.iterator(_, _)

import org.apache.spark.sql.execution.aggregate.TungstenAggregationIterator
// FIXME How to show that TungstenAggregationIterator is used?
```

When <<creating-instance, created>>, `TungstenAggregationIterator` gets SQL metrics from the [HashAggregateExec](physical-operators/HashAggregateExec.md#metrics) aggregate physical operator being executed, i.e. <<numOutputRows, numOutputRows>>, <<peakMemory, peakMemory>>, <<spillSize, spillSize>> and <<avgHashProbe, avgHashProbe>> metrics.

* <<numOutputRows, numOutputRows>> is used when `TungstenAggregationIterator` is requested for the <<next, next UnsafeRow>> (and it <<hasNext, has one>>)

* <<peakMemory, peakMemory>>, <<spillSize, spillSize>> and <<avgHashProbe, avgHashProbe>> are used at the <<TaskCompletionListener, end of every task>> (one per partition)

The metrics are then displayed as part of [HashAggregateExec](physical-operators/HashAggregateExec.md) aggregate physical operator (e.g. in web UI in [Details for Query](SQLTab.md#ExecutionPage)).

![HashAggregateExec in web UI (Details for Query)](images/spark-sql-HashAggregateExec-webui-details-for-query.png)

[[internal-registries]]
.TungstenAggregationIterator's Internal Properties (e.g. Registries, Counters and Flags)
[cols="1m,2",options="header",width="100%"]
|===
| Name
| Description

| aggregationBufferMapIterator
| [[aggregationBufferMapIterator]] `KVIterator[UnsafeRow, UnsafeRow]`

Used when...FIXME

| hashMap
a| [[hashMap]] [UnsafeFixedWidthAggregationMap](UnsafeFixedWidthAggregationMap.md) with the following:

* <<initialAggregationBuffer, initialAggregationBuffer>>

* <<StructType.md#fromAttributes, StructType>> built from (the <<spark-sql-Expression-AggregateFunction.md#aggBufferAttributes, aggBufferAttributes>> of) the <<spark-sql-AggregationIterator.md#aggregateFunctions, aggregate function expressions>>

* <<StructType.md#fromAttributes, StructType>> built from (the <<spark-sql-Expression-NamedExpression.md#toAttribute, attributes>> of) the <<groupingExpressions, groupingExpressions>>

* `1024 * 16` initial capacity

* The page size of the `TaskMemoryManager` (defaults to `spark.buffer.pageSize` configuration)

Used when `TungstenAggregationIterator` is requested for the <<next, next UnsafeRow>>, to <<outputForEmptyGroupingKeyWithoutInput, outputForEmptyGroupingKeyWithoutInput>>, <<processInputs, processInputs>>, to initialize the <<aggregationBufferMapIterator, aggregationBufferMapIterator>> and <<TaskCompletionListener, every time a partition has been processed>>.

| initialAggregationBuffer
| [[initialAggregationBuffer]] <<UnsafeRow.md#, UnsafeRow>> that is the aggregation buffer containing initial buffer values.

Used when...FIXME

| externalSorter
| [[externalSorter]] `UnsafeKVExternalSorter` used for sort-based aggregation

| sortBased
| [[sortBased]] Flag to indicate whether `TungstenAggregationIterator` uses sort-based aggregation (not hash-based aggregation).

`sortBased` flag is disabled (`false`) by default.

Enabled (`true`) when `TungstenAggregationIterator` is requested to <<switchToSortBasedAggregation, switch to sort-based aggregation>>.

Used when...FIXME
|===

=== [[processInputs]] `processInputs` Internal Method

[source, scala]
----
processInputs(fallbackStartsAt: (Int, Int)): Unit
----

`processInputs`...FIXME

NOTE: `processInputs` is used exclusively when `TungstenAggregationIterator` is <<creating-instance, created>> (and sets the internal flags to indicate whether to use a hash-based aggregation or, in the worst case, a sort-based aggregation when there is not enough memory for groups and their buffers).

=== [[switchToSortBasedAggregation]] Switching to Sort-Based Aggregation (From Preferred Hash-Based Aggregation) -- `switchToSortBasedAggregation` Internal Method

[source, scala]
----
switchToSortBasedAggregation(): Unit
----

`switchToSortBasedAggregation`...FIXME

NOTE: `switchToSortBasedAggregation` is used exclusively when `TungstenAggregationIterator` is requested to <<processInputs, processInputs>> (and the <<externalSorter, externalSorter>> is used).

==== [[next]] Getting Next UnsafeRow -- `next` Method

[source, scala]
----
next(): UnsafeRow
----

NOTE: `next` is part of Scala's http://www.scala-lang.org/api/2.11.11/#scala.collection.Iterator[scala.collection.Iterator] interface that returns the next element and discards it from the iterator.

`next`...FIXME

=== [[hasNext]] `hasNext` Method

[source, scala]
----
hasNext: Boolean
----

NOTE: `hasNext` is part of Scala's http://www.scala-lang.org/api/2.11.11/#scala.collection.Iterator[scala.collection.Iterator] interface that tests whether this iterator can provide another element.

`hasNext`...FIXME

=== [[creating-instance]] Creating TungstenAggregationIterator Instance

`TungstenAggregationIterator` takes the following when created:

* [[partIndex]] Partition index
* [[groupingExpressions]] Grouping <<spark-sql-Expression-NamedExpression.md#, named expressions>>
* [[aggregateExpressions]] [Aggregate expressions](expressions/AggregateExpression.md)
* [[aggregateAttributes]] Aggregate <<spark-sql-Expression-Attribute.md#, attributes>>
* [[initialInputBufferOffset]] Initial input buffer offset
* [[resultExpressions]] Output <<spark-sql-Expression-NamedExpression.md#, named expressions>>
* [[newMutableProjection]] Function to create a new `MutableProjection` given Catalyst expressions and attributes (i.e. `(Seq[Expression], Seq[Attribute]) => MutableProjection`)
* [[originalInputAttributes]] Output attributes (of the [child](physical-operators/HashAggregateExec.md#child) of the [HashAggregateExec](physical-operators/HashAggregateExec.md) physical operator)
* [[inputIter]] Iterator of [InternalRow](InternalRow.md)s (from a single partition of the [child](physical-operators/HashAggregateExec.md#child) of the [HashAggregateExec](physical-operators/HashAggregateExec.md) physical operator)
* [[testFallbackStartsAt]] (used for testing) Optional ``HashAggregateExec``'s [testFallbackStartsAt](physical-operators/HashAggregateExec.md#testFallbackStartsAt)
* [[numOutputRows]] `numOutputRows` <<spark-sql-SQLMetric.md#, SQLMetric>>
* [[peakMemory]] `peakMemory` <<spark-sql-SQLMetric.md#, SQLMetric>>
* [[spillSize]] `spillSize` <<spark-sql-SQLMetric.md#, SQLMetric>>
* [[avgHashProbe]] `avgHashProbe` <<spark-sql-SQLMetric.md#, SQLMetric>>

!!! note
    The SQL metrics (<<numOutputRows, numOutputRows>>, <<peakMemory, peakMemory>>, <<spillSize, spillSize>> and <<avgHashProbe, avgHashProbe>>) belong to the [HashAggregateExec](physical-operators/HashAggregateExec.md#metrics) physical operator that created the `TungstenAggregationIterator`.

`TungstenAggregationIterator` initializes the <<internal-registries, internal registries and counters>>.

`TungstenAggregationIterator` starts <<processInputs, processing input rows>> and pre-loads the first key-value pair from the <<hashMap, UnsafeFixedWidthAggregationMap>> if did not <<sortBased, switch to sort-based aggregation>>.

=== [[generateResultProjection]] `generateResultProjection` Method

[source, scala]
----
generateResultProjection(): (UnsafeRow, InternalRow) => UnsafeRow
----

NOTE: `generateResultProjection` is part of the <<spark-sql-AggregationIterator.md#generateResultProjection, AggregationIterator Contract>> to...FIXME.

`generateResultProjection`...FIXME

=== [[outputForEmptyGroupingKeyWithoutInput]] Creating UnsafeRow -- `outputForEmptyGroupingKeyWithoutInput` Method

[source, scala]
----
outputForEmptyGroupingKeyWithoutInput(): UnsafeRow
----

`outputForEmptyGroupingKeyWithoutInput`...FIXME

NOTE: `outputForEmptyGroupingKeyWithoutInput` is used when...FIXME

=== [[TaskCompletionListener]] TaskCompletionListener

`TungstenAggregationIterator` registers a `TaskCompletionListener` that is executed on task completion (for every task that processes a partition).

When executed (once per partition), the `TaskCompletionListener` updates the following metrics:

* <<peakMemory, peakMemory>>

* <<spillSize, spillSize>>

* <<avgHashProbe, avgHashProbe>>
