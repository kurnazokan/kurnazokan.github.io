---
title: Building a Data Lakehouse Architecture at Scale
date: 2025-11-20 10:00:00 +0300
categories: [Data Engineering, Architecture]
tags: [lakehouse, apache-iceberg, starburst, s3, data-architecture]
---

## Introduction

During my time at ING Bank, I had the opportunity to work on a large-scale Data Lakehouse architecture. This post shares insights from that experience and the key technical decisions we made.

## What is a Data Lakehouse?

A Data Lakehouse combines the best features of:
- **Data Lakes**: Cost-effective storage, flexibility with various data formats
- **Data Warehouses**: ACID transactions, schema enforcement, query performance

## Our Tech Stack

We built our lakehouse on:

- **Storage Layer**: AWS S3 (object storage)
- **Table Format**: Apache Iceberg and Apache Hudi (for ACID transactions)
- **Query Engine**: Starburst (Trino-based distributed SQL engine)
- **File Formats**: ORC and Parquet (columnar storage)
- **Orchestration**: Apache Airflow
- **Processing**: Apache Spark (for ETL workloads)
- **Infrastructure**: OpenShift (Kubernetes platform)

## Why Apache Iceberg?

Apache Iceberg provides several advantages:

1. **ACID Transactions**: Ensures data consistency across concurrent reads/writes
2. **Time Travel**: Query historical data snapshots
3. **Schema Evolution**: Add, drop, or rename columns without rewriting data
4. **Hidden Partitioning**: Users don't need to know partitioning details
5. **Efficient Updates/Deletes**: Row-level updates without full table rewrites

## Architecture Overview

```
Data Sources → Ingestion Layer (Kafka/Airflow) → 
Bronze Layer (Raw Data in S3) → 
Silver Layer (Cleaned/Transformed in Iceberg) → 
Gold Layer (Business-Ready in Iceberg) → 
Query Layer (Starburst)
```

## Key Challenges

### 1. Performance Optimization
- Implemented proper partitioning strategies
- Used Z-ordering for better data locality
- Regular compaction of small files

### 2. Data Quality
- Integrated Apache Griffin for data quality checks
- Automated data validation in pipelines
- Monitoring and alerting for anomalies

### 3. Governance
- Metadata management and cataloging
- Data lineage tracking
- Access control and security policies

## Lessons Learned

1. **Start with a clear data model**: Define your bronze, silver, and gold layers upfront
2. **Invest in monitoring**: Observability is crucial at scale
3. **Automate everything**: From deployments to data quality checks
4. **Document thoroughly**: Architecture decisions and data contracts
5. **Think about costs**: Cloud storage and compute can get expensive

## Performance Results

Our lakehouse architecture handled:
- **100+ GB daily ingestion**
- **Sub-second query response times** for most analytical queries
- **Thousands of concurrent users**
- **GDPR compliance** with efficient data deletion

## Conclusion

Building a lakehouse at enterprise scale is challenging but rewarding. The combination of S3, Iceberg, and Starburst gave us the flexibility of a data lake with the performance and reliability of a warehouse.

If you're considering a lakehouse architecture, start small, validate your use cases, and scale gradually.

---

*Have questions about data lakehouse architecture? Feel free to reach out on [LinkedIn](https://www.linkedin.com/in/okankurnaz)!*

