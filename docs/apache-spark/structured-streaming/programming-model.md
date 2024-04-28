# Programming Model

The key idea in Structured Streaming is to treat a live data stream as a table that is being continuously appended. This leads to a new stream processing model that is very similar to a batch processing model. You will express your streaming computation as standard batch-like query as on a static table, and Spark runs it as an *incremental* query on the *unbounded* input table.

Considering the structure defined by Learning of Spark, the Structured Streaming Programming Model has five main steps:

- Define the source
- Transform the data
- Define output sink and output mode
- Set processing details
- Start the query

Here is a brief breakdown of each step:

## Define the source

Similar to batch data processing, the initial step in streaming is to establish a DataFrame from a data source. In contrast to batch processing, however, reading from a data stream source utilizes `spark.readStream`, which incorporates a `DataStreamReader`.

This reader functions similarly to the `DataFrameReader` used in `spark.read` with a few differences. The `format` method is used to specify the streaming source, and the `options` method is used to set source-specific options.

Once the source is defined, the `load` method is called to create a streaming DataFrame that represents the data stream.

```python
from pyspark.sql.types import StructType, StructField, StringType
from pyspark.sql import functions as F
from pyspark.sql import SparkSession

schema = StructType([
    StructField("lines", StringType(), True)
])

spark = SparkSession.builder.master("local[*]").appName("WordsCount").getOrCreate()

lines = (
    spark
    .readStream
    .format("csv")
    .schema(schema)
    .option("header", "false")
    .load("/structured-streaming-basics/data/readStream/")
)
```

## Transform the data

The `DataStreamReader` returns a `DataStream` data structure, which is essentially an unbounded table representation, sharing the same interface as DataFrames.

Furthermore, transformations in Structured Streaming are categorized into two types: stateless and stateful.

- **Stateless transformations:** Operations such as `select()`, `filter()`, and `map()` do not require any information from previous or subsequent rows to process the current row; each row can be processed independently. The absence of a state makes these operations stateless.

- **Stateful Transformations:** Operations that need to maintain state. Any operation involving grouping, aggregation, or joining falls under stateful transformations. In Structured Streaming, some combinations of stateful transformations are unsupported.

Overall, transforming data in streaming can be done similarly to batch processing, with some exceptions that are outlined in the [official documentation](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html#unsupported-operations).

```python
words = (
    lines
    .withColumn("words", F.explode(F.split(F.col("lines"), "\\s")))
    .select("words")
)

counts = words.groupBy("words").count()
```
## Define output sink and output mode

The third step is to define where and how the transformed data will be stored using the `DataStreamWriter` which is encapsulated by `DataFrame.writeStream`. Similar to `DataFrameWriter`, it can write in a variety of formats such as files and Apache Kafka. Additionally, you can write to arbitrary locations using the foreachBatch() and foreach() API methods.

In the vocabulary of Streaming Systems, this variety of formats is referred to as an “output sink.” Essentially, the sinks (or output sinks) are responsible for persisting the computed data consistently with the desired semantics, such as exactly-once or at-least-once guarantees.

Furthermore, the `outputMode` is also defined, dictating how data will be written to the sink. The `outputMode` can be one of the following values:

- **append:** Only new records are written to the sink.
- **complete:** The sink contains the complete output of the query for each trigger.
- **update:** Only the updated records are written to the sink.

```python
query = (
    counts
    .writeStream
    .format("console")
    .outputMode("complete")
)
```

## Set processing details

O passo final antes de iniciar a query é especificar os detalhes de triggering e checkponting.

Através da política de triggering é que definmos quando novos dados devem ser descobertos (coletados) e processados. Até a versão 3.5 do Spark, existem quatro opções de triggers.

- **default:** Streaming query executes micro-batches without explicit specification, triggering the next micro-batch when the previous one completes.

- **once:** Executes a single micro-batch and then stops.

- **available-now**. Similar to queries one-time micro-batch trigger. The difference is that it will process the data in (possibly) multiple micro-batches based on the source options.

- **fixed-time interval:** Executes micro-batches at a fixed time interval.

- **continuous:** An experimental mode (as of Spark 3.0) that processes data continuously instead of micro-batches, providing lower latency (only a subset of DataFrame operations support this mode).

And the checkpoint location is where the system stores the metadata of the query, such as the current state of the processing, the offsets, and the schema of the data. This information is crucial for fault tolerance and recovery.

```python
query = (
    counts
    .writeStream
    .format("console")
    .outputMode("complete")
    .option("checkpointLocation", "/structured-streaming-basics/data/checkpoints/")
    .trigger(processingTime="10 seconds")
)
```

## Start the query

Once everything has been specified, the final step is to start the query.

```python
query.start()
```

The returned object of type `StreamingQuery` represents an active query and can be used to manage the query, which we will cover later in this chapter.

Note that `start()` is a nonblocking method and returns once the query starts in the background. To block the main thread until query termination, use `StreamingQuery.awaitTermination()`. If the query fails, `awaitTermination()` also fails with the same exception.

You can also use `awaitTermination(timeoutMillis)` to wait for a specified duration and `stop()` to stop the query explicitly.

## Conclusion

To summarize, here is the complete code.

```python
from pyspark.sql.types import StructType, StructField, StringType
from pyspark.sql import functions as F
from pyspark.sql import SparkSession

schema = StructType([
    StructField("lines", StringType(), True)
])

spark = SparkSession.builder.master("local[*]").appName("WordsCount").getOrCreate()

lines = (
    spark
    .readStream
    .format("csv")
    .schema(schema)
    .option("header", "false")
    .load("/structured-streaming-basics/data/readStream/")
)

words = (
    lines
    .withColumn("words", F.explode(F.split(F.col("lines"), "\\s")))
    .select("words")
)

counts = words.groupBy("words").count()

query = (
    counts
    .writeStream
    .format("console")
    .option("checkpointLocation", "/structured-streaming-basics/data/checkpoints/")
    .outputMode("complete")
)
query.start()
query.awaitTermination()
```

## References

- [Learning Spark, 2nd Edition](https://pages.databricks.com/rs/094-YMS-629/images/LearningSpark2.0.pdf)
- [Structured Streaming Programming Guide](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html)
