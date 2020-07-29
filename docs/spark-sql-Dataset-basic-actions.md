title: Basic Actions

# Dataset API -- Basic Actions

*Basic actions* are a group of operators (_methods_) of the <<spark-sql-dataset-operators.md#, Dataset API>> for transforming a `Dataset` into a session-scoped or global temporary view and _other basic actions_ (FIXME).

NOTE: Basic actions are the methods in the `Dataset` Scala class that are grouped in `basic` group name, i.e. `@group basic`.

[[methods]]
.Dataset API's Basic Actions
[cols="1,3",options="header",width="100%"]
|===
| Action
| Description

| `cache`
a| [[cache]]

[source, scala]
----
cache(): this.type
----

Marks the `Dataset` to be persisted (_cached_) and is actually a synonym of <<spark-sql-dataset-operators.md#persist, persist>> basic action

| <<checkpoint, checkpoint>>
a|

[source, scala]
----
checkpoint(): Dataset[T]
checkpoint(eager: Boolean): Dataset[T]
----

Checkpoints the `Dataset` in a reliable way (using a reliable HDFS-compliant file system, e.g. Hadoop HDFS or Amazon S3)

| <<columns, columns>>
a|

[source, scala]
----
columns: Array[String]
----

| <<createGlobalTempView, createGlobalTempView>>
a|

[source, scala]
----
createGlobalTempView(viewName: String): Unit
----

| <<createOrReplaceGlobalTempView, createOrReplaceGlobalTempView>>
a|

[source, scala]
----
createOrReplaceGlobalTempView(viewName: String): Unit
----

| <<createOrReplaceTempView, createOrReplaceTempView>>
a|

[source, scala]
----
createOrReplaceTempView(viewName: String): Unit
----

| <<createTempView, createTempView>>
a|

[source, scala]
----
createTempView(viewName: String): Unit
----

| <<dtypes, dtypes>>
a|

[source, scala]
----
dtypes: Array[(String, String)]
----

| <<explain, explain>>
a|

[source, scala]
----
explain(): Unit
explain(extended: Boolean): Unit
----

Displays the logical and physical plans of the `Dataset`, i.e. displays the logical and physical plans (with optional cost and codegen summaries) to the standard output

| <<hint, hint>>
a|

[source, scala]
----
hint(name: String, parameters: Any*): Dataset[T]
----

| <<inputFiles, inputFiles>>
a|

[source, scala]
----
inputFiles: Array[String]
----

| <<isEmpty, isEmpty>>
a|

[source, scala]
----
isEmpty: Boolean
----

(*New in 2.4.0*)

| <<isLocal, isLocal>>
a|

[source, scala]
----
isLocal: Boolean
----

| <<localCheckpoint, localCheckpoint>>
a|

[source, scala]
----
localCheckpoint(): Dataset[T]
localCheckpoint(eager: Boolean): Dataset[T]
----

Checkpoints the `Dataset` locally on executors (and therefore unreliably)

| `persist`
a| [[persist]]

[source, scala]
----
persist(): this.type // <1>
persist(newLevel: StorageLevel): this.type
----
<1> Assumes the default storage level `MEMORY_AND_DISK`

Marks the `Dataset` to be <<spark-sql-caching-and-persistence.md#, persisted>> the next time an action is executed

Internally, `persist` simply request the `CacheManager` to <<spark-sql-CacheManager.md#cacheQuery, cache the structured query>>.

NOTE: `persist` uses the <<SharedState.md#cacheManager, CacheManager>> from the <<SparkSession.md#sharedState, SharedState>> associated with the <<spark-sql-Dataset.md#sparkSession, SparkSession>> (of the Dataset).

| <<printSchema, printSchema>>
a|

[source, scala]
----
printSchema(): Unit
----

| <<rdd, rdd>>
a|

[source, scala]
----
rdd: RDD[T]
----

| <<schema, schema>>
a|

[source, scala]
----
schema: StructType
----

| <<storageLevel, storageLevel>>
a|

[source, scala]
----
storageLevel: StorageLevel
----

| <<toDF, toDF>>
a|

[source, scala]
----
toDF(): DataFrame
toDF(colNames: String*): DataFrame
----

| <<unpersist, unpersist>>
a|

[source, scala]
----
unpersist(): this.type
unpersist(blocking: Boolean): this.type
----

Unpersists the `Dataset`

| <<write, write>>
a|

[source, scala]
----
write: DataFrameWriter[T]
----

Returns a <<spark-sql-DataFrameWriter.md#, DataFrameWriter>> for saving the content of the (non-streaming) `Dataset` out to an external storage
|===

=== [[checkpoint]] Reliably Checkpointing Dataset -- `checkpoint` Basic Action

[source, scala]
----
checkpoint(): Dataset[T]  // <1>
checkpoint(eager: Boolean): Dataset[T]  // <2>
----
<1> `eager` and `reliableCheckpoint` flags enabled
<2> `reliableCheckpoint` flag enabled

NOTE: `checkpoint` is an experimental operator and the API is evolving towards becoming stable.

`checkpoint` simply requests the `Dataset` to <<checkpoint-internal, checkpoint>> with the given `eager` flag and the `reliableCheckpoint` flag enabled.

=== [[createTempView]] `createTempView` Basic Action

[source, scala]
----
createTempView(viewName: String): Unit
----

`createTempView`...FIXME

NOTE: `createTempView` is used when...FIXME

=== [[createOrReplaceTempView]] `createOrReplaceTempView` Basic Action

[source, scala]
----
createOrReplaceTempView(viewName: String): Unit
----

`createOrReplaceTempView`...FIXME

NOTE: `createOrReplaceTempView` is used when...FIXME

=== [[createGlobalTempView]] `createGlobalTempView` Basic Action

[source, scala]
----
createGlobalTempView(viewName: String): Unit
----

`createGlobalTempView`...FIXME

NOTE: `createGlobalTempView` is used when...FIXME

=== [[createOrReplaceGlobalTempView]] `createOrReplaceGlobalTempView` Basic Action

[source, scala]
----
createOrReplaceGlobalTempView(viewName: String): Unit
----

`createOrReplaceGlobalTempView`...FIXME

NOTE: `createOrReplaceGlobalTempView` is used when...FIXME

=== [[createTempViewCommand]] `createTempViewCommand` Internal Method

[source, scala]
----
createTempViewCommand(
  viewName: String,
  replace: Boolean,
  global: Boolean): CreateViewCommand
----

`createTempViewCommand`...FIXME

NOTE: `createTempViewCommand` is used when the following `Dataset` operators are used: <<spark-sql-dataset-operators.md#createTempView, Dataset.createTempView>>, <<spark-sql-dataset-operators.md#createOrReplaceTempView, Dataset.createOrReplaceTempView>>, <<spark-sql-dataset-operators.md#createGlobalTempView, Dataset.createGlobalTempView>> and <<spark-sql-dataset-operators.md#createOrReplaceGlobalTempView, Dataset.createOrReplaceGlobalTempView>>.

=== [[explain]] Displaying Logical and Physical Plans, Their Cost and Codegen -- `explain` Basic Action

[source, scala]
----
explain(): Unit // <1>
explain(extended: Boolean): Unit
----
<1> Turns the `extended` flag on

`explain` prints the spark-sql-LogicalPlan.md[logical] and (with `extended` flag enabled) SparkPlan.md[physical] plans, their cost and codegen to the console.

TIP: Use `explain` to review the structured queries and optimizations applied.

Internally, `explain` creates a spark-sql-LogicalPlan-ExplainCommand.md[ExplainCommand] logical command and requests `SessionState` to SessionState.md#executePlan[execute it] (to get a spark-sql-QueryExecution.md[QueryExecution] back).

NOTE: `explain` uses spark-sql-LogicalPlan-ExplainCommand.md[ExplainCommand] logical command that, when spark-sql-LogicalPlan-ExplainCommand.md#run[executed], gives different text representations of spark-sql-QueryExecution.md[QueryExecution] (for the Dataset's spark-sql-LogicalPlan.md[LogicalPlan]) depending on the flags (e.g. extended, codegen, and cost which are disabled by default).

`explain` then requests `QueryExecution` for the spark-sql-QueryExecution.md#executedPlan[optimized physical query plan] and SparkPlan.md#executeCollect[collects the records] (as spark-sql-InternalRow.md[InternalRow] objects).

[NOTE]
====
`explain` uses Dataset's spark-sql-Dataset.md#sparkSession[SparkSession] to SparkSession.md#sessionState[access the current `SessionState`].
====

In the end, `explain` goes over the `InternalRow` records and converts them to lines to display to console.

NOTE: `explain` "converts" an `InternalRow` record to a line using spark-sql-InternalRow.md#getString[getString] at position `0`.

TIP: If you are serious about query debugging you could also use the spark-sql-debugging-query-execution.md[Debugging Query Execution facility].

[source, scala]
----
scala> spark.range(10).explain(extended = true)
== Parsed Logical Plan ==
Range (0, 10, step=1, splits=Some(8))

== Analyzed Logical Plan ==
id: bigint
Range (0, 10, step=1, splits=Some(8))

== Optimized Logical Plan ==
Range (0, 10, step=1, splits=Some(8))

== Physical Plan ==
*Range (0, 10, step=1, splits=Some(8))
----

=== [[hint]] Specifying Hint -- `hint` Basic Action

[source, scala]
----
hint(name: String, parameters: Any*): Dataset[T]
----

`hint` operator is part of spark-sql-hint-framework.md[Hint Framework] to specify a *hint* (by `name` and `parameters`) for a `Dataset`.

Internally, `hint` simply attaches spark-sql-LogicalPlan-UnresolvedHint.md[UnresolvedHint] unary logical operator to an "analyzed" `Dataset` (i.e. the spark-sql-Dataset.md#logicalPlan[analyzed logical plan] of a `Dataset`).

[source, scala]
----
val ds = spark.range(3)
val plan = ds.queryExecution.logical
scala> println(plan.numberedTreeString)
00 Range (0, 3, step=1, splits=Some(8))

// Attach a hint
val dsHinted = ds.hint("myHint", 100, true)
val plan = dsHinted.queryExecution.logical
scala> println(plan.numberedTreeString)
00 'UnresolvedHint myHint, [100, true]
01 +- Range (0, 3, step=1, splits=Some(8))
----

NOTE: `hint` adds an <<spark-sql-LogicalPlan-UnresolvedHint.md#, UnresolvedHint>> unary logical operator to an analyzed logical plan that indirectly triggers spark-sql-QueryExecution.md#analyzed[analysis phase] that executes spark-sql-LogicalPlan-Command.md[logical commands] and their unions as well as resolves all hints that have already been added to a logical plan.

[source, scala]
----
// FIXME Demo with UnresolvedHint
----

=== [[localCheckpoint]] Locally Checkpointing Dataset -- `localCheckpoint` Basic Action

[source, scala]
----
localCheckpoint(): Dataset[T] // <1>
localCheckpoint(eager: Boolean): Dataset[T]
----
<1> `eager` flag enabled

`localCheckpoint` simply uses <<checkpoint, Dataset.checkpoint>> operator with the input `eager` flag and `reliableCheckpoint` flag disabled (`false`).

=== [[checkpoint-internal]] `checkpoint` Internal Method

[source, scala]
----
checkpoint(eager: Boolean, reliableCheckpoint: Boolean): Dataset[T]
----

`checkpoint` requests spark-sql-Dataset.md#queryExecution[QueryExecution] (of the `Dataset`) to spark-sql-QueryExecution.md#toRdd[generate an RDD of internal binary rows] (aka `internalRdd`) and then requests the RDD to make a copy of all the rows (by adding a `MapPartitionsRDD`).

Depending on `reliableCheckpoint` flag, `checkpoint` marks the RDD for (reliable) checkpointing (`true`) or local checkpointing (`false`).

With `eager` flag on, `checkpoint` counts the number of records in the RDD (by executing `RDD.count`) that gives the effect of immediate eager checkpointing.

`checkpoint` requests spark-sql-Dataset.md#queryExecution[QueryExecution] (of the `Dataset`) for spark-sql-QueryExecution.md#executedPlan[optimized physical query plan] (the plan is used to get the SparkPlan.md#outputPartitioning[outputPartitioning] and SparkPlan.md#outputOrdering[outputOrdering] for the result `Dataset`).

In the end, `checkpoint` spark-sql-Dataset.md#ofRows[creates a DataFrame] with a new spark-sql-LogicalPlan-LogicalRDD.md#creating-instance[logical plan node for scanning data from an RDD of InternalRows] (`LogicalRDD`).

NOTE: `checkpoint` is used in the `Dataset` <<spark-sql-Dataset-untyped-transformations.md#, untyped transformations>>, i.e. <<spark-sql-Dataset-untyped-transformations.md#checkpoint, checkpoint>> and <<spark-sql-Dataset-untyped-transformations.md#localCheckpoint, localCheckpoint>>.

=== [[rdd]] Generating RDD of Internal Binary Rows -- `rdd` Basic Action

[source, scala]
----
rdd: RDD[T]
----

Whenever you are in need to convert a `Dataset` into a `RDD`, executing `rdd` method gives you the RDD of the proper input object type (not spark-sql-DataFrame.md#features[Row as in DataFrames]) that sits behind the `Dataset`.

[source, scala]
----
scala> val rdd = tokens.rdd
rdd: org.apache.spark.rdd.RDD[Token] = MapPartitionsRDD[11] at rdd at <console>:30
----

Internally, it looks spark-sql-ExpressionEncoder.md[ExpressionEncoder] (for the `Dataset`) up and accesses the `deserializer` expression. That gives the spark-sql-DataType.md[DataType] of the result of evaluating the expression.

NOTE: A deserializer expression is used to decode an spark-sql-InternalRow.md[InternalRow] to an object of type `T`. See spark-sql-ExpressionEncoder.md[ExpressionEncoder].

It then executes a spark-sql-LogicalPlan-DeserializeToObject.md[`DeserializeToObject` logical operator] that will produce a `RDD[InternalRow]` that is converted into the proper `RDD[T]` using the `DataType` and `T`.

NOTE: It is a lazy operation that "produces" a `RDD[T]`.

=== [[schema]] Accessing Schema -- `schema` Basic Action

A `Dataset` has a *schema*.

[source, scala]
----
schema: StructType
----

[TIP]
====
You may also use the following methods to learn about the schema:

* `printSchema(): Unit`
* <<spark-sql-Dataset-basic-actions.md#explain, explain>>
====

=== [[toDF]] Converting Typed Dataset to Untyped DataFrame -- `toDF` Basic Action

[source, scala]
----
toDF(): DataFrame
toDF(colNames: String*): DataFrame
----

`toDF` converts a spark-sql-Dataset.md[Dataset] into a spark-sql-DataFrame.md[DataFrame].

Internally, the empty-argument `toDF` creates a `Dataset[Row]` using the ``Dataset``'s SparkSession.md[SparkSession] and spark-sql-QueryExecution.md[QueryExecution] with the encoder being spark-sql-RowEncoder.md[RowEncoder].

CAUTION: FIXME Describe `toDF(colNames: String*)`

=== [[unpersist]] Unpersisting Cached Dataset -- `unpersist` Basic Action

[source, scala]
----
unpersist(): this.type
unpersist(blocking: Boolean): this.type
----

`unpersist` uncache the `Dataset` possibly by `blocking` the call.

Internally, `unpersist` requests `CacheManager` spark-cachemanager.md#uncacheQuery[to uncache the query].

CAUTION: FIXME

=== [[write]] Accessing DataFrameWriter (to Describe Writing Dataset) -- `write` Basic Action

[source, scala]
----
write: DataFrameWriter[T]
----

`write` gives spark-sql-DataFrameWriter.md[DataFrameWriter] for records of type `T`.

[source, scala]
----
import org.apache.spark.sql.{DataFrameWriter, Dataset}
val ints: Dataset[Int] = (0 to 5).toDS
val writer: DataFrameWriter[Int] = ints.write
----

=== [[isEmpty]] `isEmpty` Typed Transformation

[source, scala]
----
isEmpty: Boolean
----

`isEmpty`...FIXME

=== [[isLocal]] `isLocal` Typed Transformation

[source, scala]
----
isLocal: Boolean
----

`isLocal`...FIXME