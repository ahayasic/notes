# Sources

## Introduction

A source is the abstraction that enables the consumption of data from a streaming data producer. It acts as a data provider by presenting the data to the stream processing engine
as an unbounded table. More than just reading data, the source implementation also includes the logic to manage offsets, which are used to track the position in the data stream and enable fault-tolerance and exactly-once semantics.

Sources are defined declaratively using the `SparkSession.readStream()` method, which exposes a `DataStreamReader`. This interface allows users to specify a streaming source through the `format()` method and set source-specific options using the `options()` method. Once the source is defined, the `load()` method is called to create a streaming DataFrame that represents the data stream.

!!! info "Multiple sources"
    A streaming query can define multiple input sources (streaming and batch), which can be combined using `DataFrame` operations like unions and joins.

!!! info "Lazy loading"
    The loading of streaming sources is a lazy operation. Initially, what is obtained is a representation of the stream, encapsulated within a streaming DataFrame instance from which transformations can be defined to implement specific business logic.

    Data consumption and processing only begin when the stream is materialized through the initiation of a query.

Spark Structured Streaming natively supports reading data streams from various sources. These include TCP sockets, Apache Kafka, and numerous file-based formats that the DataFrameReader also supports, such as Parquet, ORC, JSON, CSV and text files.

Until Spark 3.5, the native sources implemented are as follows:

- **File-based formats (Parquet, ORC, JSON, CSV, text files):** These sources continuously monitor a specified directory, treating it as a data stream where new files are regularly added. Once detected, these files are read and processed based on their modification times. Files must be placed completely and independently in the directory (atomically).
- **Kafka**. Connects to Apache Kafka as a consumer to retrieve data from Kafka topics.
- **Socket:** Establishes a connection to a TCP server and reads UTF8 text data from it. The listening server socket is at the driver. Used only for testing since doesn't not provide end-to-end fault-tolerance guarantees.
- **Rate:** Generates a stream of rows at a specified rate. Each rows contains a `timestamp` and `value` column. This source is intended for testing and benchmarking.
- **Rate Per Micro-Batch:** This source generates data at a specified number of rows per micro-batch, with each row containing a `timestamp` and a `value`. Unlike rate data source, this data source provides a consistent set of input rows per micro-batch regardless of query execution (configuration of trigger, query being lagging, etc.). This source is intended for testing and benchmarking.

!!! tip "Practical examples"
    In the repository, there examples of how each source can be used within the different options available.

## Replayability

To guarantee fault tolerance and exactly-once semantics, one of the requirements is that the source must be replayable. This means that the source must be able to retrieve a part of the stream that had already been requested but not yet committed. In Structured Streaming, this is done by managing [offsets](#offsets).

To a source to be replayable, it must be reliable. A source is considered reliable when it can produce an uncommitted offset range even after a total failure of the Structured Streaming process. Then, in this failure recovery process, offsets are restored from their last known checkpoint and requested again from the source.

!!! info "Replayability and idempotence"
    The second requirement for exactly-once semantic is that the sink must be idempotent. More details about it in the Sinks section.

!!! note "Non-replayable sources"
    Not all native sources are replayable. For example, the socket source is not replayable because it doesn't have a way to retrieve data that was already sent. In this case, the source is not able to provide the data that was lost in the failure recovery process.

#### Offsets

The concept behind the source interface is that streaming data is a continuous flow of events over time that can be seen as a sequence, indexed with a monotonously incrementing counter (i.e. offsets).

Offsets are used to request data from the external source and to indicate what data has already been consumed. Every streaming source is assumed to have offsets (similar to Kafka offsets, or Kinesis sequence numbers) to track the read position in the stream.

Structured Streaming knows when there is data to process by asking the current offset from the external system and comparing it to the last processed offset. The data to be processed is requested by getting a *batch* between two offsets start and end (in the practice, it's done calling `getBatch` operation with the offset range that we want to receive)

The source is informed that the data has been processed when Spark commit a given offset. The engine uses checkpointing and write-ahead logs to record the offset range of the data being processed in each trigger (to recover from eventual failure, those offsets are often *checkpointed* to external storage).

The source contract guarantees that all data with an offset less than or equal to the committed offset has been processed and that subsequent requests will stipulate only offsets greater than that committed offset.

!!! note "It's just an abstraction"
    The current explanation might give the impression that external source systems must understand or implement the concept of *offsets*, but this is not the case. It is a combination of the external source system's capabilities and the source implementation.

    As mentioned previously, Apache Spark natively supports various data streams. This support stems from the source abstraction, where each source is implemented in a particular way. Offsets are one of the concepts abstracted in the source abstraction.

    An example is file-based sources, where Spark treats the presence of new files as an increment in offset, even though the underlying files do not possess an inherent concept of offsets, unlike Kafka.

## Schemas

In Apache Spark's structured APIs, schemas are essential as they outline the data's structure, enabling optimizations from query planning to storage. Data sources like JSON or CSV files require predefined schemas for accurate parsing, while other sources might have fixed schemas focusing on metadata, leaving data interpretation to the application.

Schema-driven design in streaming applications enhances the understanding and organization of data flows across multiple stages. Schemas in Structured Streaming can be defined programmatically using `StructType` and `StructField` to build complex structures, or they can be inferred from Scala's case classes, which simplifies the process and maintains type safety. Alternatively, schemas can be extracted from existing datasets like Parquet files, useful during prototyping but requiring maintenance to avoid complexity.

## Custom Sources

Structured Streaming enables the use of custom sources by either extending the Source interface or incorporating sources provided by third parties through custom JARs and libraries.

An interesting additional source that Apache Spark supports (natively) is [Delta Tables](https://docs.delta.io/latest/delta-streaming.html), a robust storage file format that enables ACID transactions for Spark and big data workloads. Delta Tables are designed to allow reliable and concurrent reads and writes on datasets, which is ideal for complex data pipelines and streaming scenarios.

## References

- [Stream Processing with Apache Spark](https://a.co/d/95AeqKu)
- [Learning Spark, 2nd Edition](https://pages.databricks.com/rs/094-YMS-629/images/LearningSpark2.0.pdf)
- [Structured Streaming Programming Guide](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html)
