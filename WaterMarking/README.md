# Feature Deep Dive: Watermarking in Apache Spark Structured Streaming

## Key Takeaways
- **Watermarks** help Spark understand the processing progress based on event time, determine when to produce windowed aggregates, and when to trim the aggregation state.
- When **joining streams of data**, Spark, by default, uses a single, global watermark that evicts state based on the minimum event time seen across the input streams.
- **RocksDB** can be leveraged to reduce pressure on cluster memory and garbage collection (GC) pauses.
- **StreamingQueryProgress** and **StateOperatorProgress** objects contain key information about how watermarks affect your stream.

## Introduction
When building real-time pipelines, one of the realities that teams have to work with is that distributed data ingestion is inherently unordered. Additionally, in the context of stateful streaming operations, teams need to be able to properly track event time progress in the stream of data they are ingesting for the proper calculation of time-window aggregations and other stateful operations. We can solve this using **Structured Streaming**.

For example, let’s say we are a team working on building a pipeline to help our company do proactive maintenance on our mining machines that we lease to our customers. These machines always need to be running in top condition, so we monitor them in real-time. We need to perform stateful aggregations on the streaming data to understand and identify problems in the machines.

This is where we need to leverage **Structured Streaming** and **Watermarking** to produce the necessary stateful aggregations that will help inform decisions around predictive maintenance and more for these machines.

## What Is Watermarking?
Generally speaking, when working with real-time streaming data, there will be delays between event time and processing time due to how data is ingested and whether the overall application experiences issues like downtime. Due to these potential variable delays, the engine that you use to process this data needs to have some mechanism to decide when to close the aggregate windows and produce the aggregate result.

While the natural inclination to remedy these issues might be to use a fixed delay based on the wall clock time, we will show in this upcoming example why this is not the best solution.

![Solution Architecture](WaterMarking/Assests/watermarking.png)

### Example: Watermarking in Action
Let’s take a scenario where we are receiving data at various times from around 10:50 AM to 11:20 AM. We are creating 10-minute tumbling windows that calculate the average of the temperature and pressure readings that came in during the windowed period.

In this first scenario, the tumbling windows trigger at 11:00 AM, 11:10 AM, and 11:20 AM, leading to the result tables shown at the respective times. When the second batch of data comes around 11:10 AM with data that has an event time of 10:53 AM, this gets incorporated into the temperature and pressure averages calculated for the 11:00 AM to 11:10 AM window that closes at 11:10 AM, which does not give the correct result.

To ensure we get the correct results for the aggregates we want to produce, we need to define a watermark that will allow Spark to understand when to close the aggregate window and produce the correct aggregate result.

![Solution Architecture](WaterMarking/Assests/water marking 2.png)

### How Watermarking Works
In **Structured Streaming** applications, we can ensure that all relevant data for the aggregations we want to calculate is collected by using a feature called watermarking. By defining a watermark, Spark Structured Streaming then knows when it has ingested all data up to some time, T, based on a set lateness expectation, so that it can close and produce windowed aggregates up to timestamp T.

In contrast to the first scenario, with watermarking, Spark waits to close and output the windowed aggregation once the max event time seen minus the specified watermark is greater than the upper bound of the window.

This produces the correct result by properly incorporating the data based on the expected lateness defined by the watermark. Once the results are emitted, the corresponding state is removed from the state store.

## Incorporating Watermarking into Your Pipelines
To understand how to incorporate these watermarks into our Structured Streaming pipelines, let’s explore a scenario where we are ingesting all our sensor data from a Kafka cluster in the cloud and want to calculate temperature and pressure averages every ten minutes with an expected time skew of ten minutes.

Here’s how the Structured Streaming pipeline with watermarking would look:

```python
sensorStreamDF = spark \
  .readStream \
  .format("kafka") \
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2") \
  .option("subscribe", "tempAndPressureReadings") \
  .load()

sensorStreamDF = sensorStreamDF \
.withWatermark("eventTimestamp", "10 minutes") \
.groupBy(window(sensorStreamDF.eventTimestamp, "10 minutes")) \
.avg(sensorStreamDF.temperature,
     sensorStreamDF.pressure)

sensorStreamDF.writeStream
  .format("delta")
  .outputMode("append")
  .option("checkpointLocation", "/delta/events/_checkpoints/temp_pressure_job/")
  .start("/delta/temperatureAndPressureAverages")

