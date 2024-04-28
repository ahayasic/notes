# Structured Streaming

The Apache Spark Structured Streaming is a scalable and fault-tolerant stream processing engine built on the Spark SQL engine [1]. By extending the Spark SQL APIs to support streaming workloads, developers can express the streaming computation the same way they would express a batch computation on static data in all the supported language bindings for Spark SQL and including streaming aggregations, event-time windows, stream-to-batch joins, while inherits the underlying optimizations of Spark SQL, in special the use of the Catalyst query optimizer and the low overhead memory management and code generation delivered by Project Tungsten.

Not only The Spark SQL engine will take care of running it incrementally and continuously and updating the final result as streaming data continues to arrive following the micro-batch processing model as well system ensures end-to-end exactly-once fault-tolerance guarantees through checkpointing and Write-Ahead Logs. In short, *Structured Streaming provides fast, scalable, fault-tolerant, end-to-end exactly-once stream processing without the user having to reason about streaming.*

## Topics to write about

- [ ]  Incremental execution. How it works?
- [ ]  It’s possible to change the streaming query once it’s started?
- [ ]  How delivery semantics works? What’s the relationship with source, sink and checkpoints?
- [ ]

## References

[1] Apache Spark Structured Streaming Official Documentation.

[2]