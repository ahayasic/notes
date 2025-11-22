# Event Time-Based Processing

When analyzing patterns in event data over time, like aggregating data into specific time windows, it's essential to process the events as if we were observing them at the moment they were generated. Event-time processing refers to looking at the stream of events from the timeline at which they were produced and applying the processing logic from that perspective.

For instance, consider a temperature sensor that sends data every minute. To detect if any sensor is reporting a number higher than a threshold, we can check the max temperature received from each sensor in five-minute intervals.

To do this, the device or system that generates the event needs to "stamp" the events with the time of creation. Structured Streaming uses the timestamp field in events to maintain a monotonically increasing upper bound, forming a nonlinear event timeline that guides the event-time processing. This timeline is used to handle with time-based features in Structured Streaming, such as windowed aggregations.

The ability of Structured Streaming to understand the time flow of the event source decouples event generation from processing time, allowing for the replay of a sequence of past events and have Structured Streaming produce the correct results for all event-time aggregations (backfilling).

!!! note "Nonlinear event timeline"
    The timeline in Spark is effectively formed by the event-time timestamps. The timeline is not a simple sequence of events but a representation of the event-time of the events. Spark maintains a monotonically increasing upper bound on this timeline, ensuring that out-of-order events that fall within the watermark delay are still considered for processing. Due to the possibility of out-of-order events, the timeline is nonlinear. Events do not strictly follow real-time but are aligned according to their event-time timestamps. This nonlinear timeline allows Spark to process events based on when they occurred, rather than when they are processed.

## Window Operations on Event Time

Aggregations over a sliding event-time window are straightforward with Structured Streaming and are very similar to grouped aggregations. In a grouped aggregation, aggregate values (e.g. counts) are maintained for each unique value in the user-specified grouping column. In case of window-based aggregations, aggregate values are maintained for each window the event-time of a row falls into.

## Watermarks

External factors can affect the deliv‐ ery of event messages and, hence, when using event time for processing, we didn’t have a guarantee of order or delivery. Events might be late or never arrive at all. So, how late is too late? For how long do we hold partial aggregations before considering them complete?

A watermark is a time threshold that dictates how long we wait for events before declaring that they are too late. Events that are considered late beyond the watermark are discarded. Watermarks are computed as a threshold based on the "internal time representation". The watermark line is a shifted line from the event-time timeline inferred from the event’s time information. The watermark is, at any given moment, the oldest timestamp that we will accept on the data stream. Any events that are older than this expectation are not taken into the results of the stream processing. The streaming engine can choose to process them in an alternative way, like report them in a late arrivals channel, for example.

---

To account for possibly delayed events, we use the concept called watermark. A watermark is usually much larger than the average delay we expect in the delivery of the events. Note also that this watermark is a fluid value that monotonically increases over time,2 sliding a win‐ dow of delay-tolerance as the time observed in the data-stream progresses.

When we apply this concept of watermark to our event-time diagram, as illustrated in
Figure 2-7, we can appreciate that the watermark closes the open boundary left by the definition of event-time window, providing criteria to decide what events belong to the window, and what events are too late to be considered for processing.

## Questions

- How the output mode influences the event-time processing?
- How backfilling is done in Structured Streaming?
