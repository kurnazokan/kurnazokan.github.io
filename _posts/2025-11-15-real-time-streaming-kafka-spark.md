---
title: Real-Time Data Streaming with Kafka and Spark
date: 2025-11-15 14:00:00 +0300
categories: [Data Engineering, Streaming]
tags: [kafka, spark, streaming, real-time, debezium, cdc]
---

## Introduction

Real-time data processing has become essential for modern businesses. During my time at Monotech Gaming, I built a streaming pipeline that processed millions of events daily. Here's what I learned.

## The Use Case

We needed to:
- Capture database changes in real-time (CDC)
- Process and enrich streaming data
- Make data available for analytics within seconds
- Handle high throughput (10K+ events/second)

## Architecture

```
PostgreSQL → Debezium → Kafka → Spark Structured Streaming → 
ClickHouse/BigQuery → Superset
```

## Technology Stack

- **CDC**: Debezium (for Change Data Capture from PostgreSQL)
- **Message Broker**: Apache Kafka
- **Stream Processing**: Spark Structured Streaming
- **Storage**: ClickHouse (OLAP) and GCP BigQuery
- **Ingestion API**: FastAPI (for low-latency event ingestion)
- **Visualization**: Apache Superset

## Setting Up Change Data Capture

### Why Debezium?

Debezium captures row-level changes from your database transaction log without impacting database performance.

**Key Benefits:**
- Low latency (near real-time)
- No changes to application code
- Captures all operations (INSERT, UPDATE, DELETE)
- Exactly-once semantics

### Basic Debezium Configuration

```json
{
  "name": "postgres-connector",
  "config": {
    "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
    "database.hostname": "postgres-host",
    "database.port": "5432",
    "database.user": "user",
    "database.password": "password",
    "database.dbname": "mydb",
    "database.server.name": "dbserver",
    "table.include.list": "public.users,public.transactions"
  }
}
```

## Spark Structured Streaming

Spark Structured Streaming provides a high-level API for stream processing with the same semantics as batch processing.

### Sample Code

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import *

spark = SparkSession.builder \
    .appName("KafkaStreaming") \
    .getOrCreate()

# Read from Kafka
df = spark \
    .readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "user-events") \
    .load()

# Parse JSON and transform
parsed_df = df.select(
    from_json(col("value").cast("string"), schema).alias("data")
).select("data.*")

# Aggregate and write
query = parsed_df \
    .groupBy(window("timestamp", "1 minute"), "user_id") \
    .count() \
    .writeStream \
    .format("console") \
    .outputMode("complete") \
    .start()

query.awaitTermination()
```

## Performance Optimization

### 1. Kafka Tuning

- **Partitioning**: Used consistent hashing for even distribution
- **Batch Size**: Increased batch size to 100K for better throughput
- **Compression**: Enabled LZ4 compression (3x reduction)

### 2. Spark Optimization

- **Checkpointing**: Enabled checkpointing for fault tolerance
- **Trigger Interval**: Set to 30 seconds (balance between latency and throughput)
- **Parallelism**: Matched Kafka partitions to Spark partitions

### 3. Sink Optimization

- **Batch Writes**: Buffered writes to ClickHouse
- **Partitioning**: Time-based partitioning for better query performance

## Monitoring and Observability

Essential metrics to track:

- **Lag**: Kafka consumer lag (should be < 1000 messages)
- **Throughput**: Events processed per second
- **Latency**: End-to-end processing time (target: < 5 seconds)
- **Error Rate**: Failed messages rate

## Real-World Results

Our streaming pipeline achieved:
- **< 3 seconds** end-to-end latency
- **15K events/second** throughput
- **99.9% uptime**
- Enabled real-time dashboards for business teams

## Common Pitfalls

1. **Not handling late data**: Always define watermarks
2. **Memory issues**: Monitor memory usage and tune accordingly
3. **Schema evolution**: Plan for schema changes from day one
4. **Exactly-once semantics**: Understand when you need it vs. at-least-once

## Conclusion

Building real-time streaming pipelines requires careful planning and the right tool choices. The combination of Kafka, Debezium, and Spark Structured Streaming provides a robust foundation for most use cases.

Start with a clear understanding of your latency requirements and scale gradually.

---

*Want to discuss real-time architectures? Connect with me on [LinkedIn](https://www.linkedin.com/in/okankurnaz)!*

