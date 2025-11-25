---
title: Building a Data Quality Framework That Actually Works
date: 2025-11-10 09:00:00 +0300
categories: [Data Engineering, Data Quality]
tags: [data-quality, data-governance, testing, apache-griffin, dbt]
---

## The Problem with Data Quality

Bad data costs organizations millions of dollars annually. During my career, I've seen data quality issues cause:

- Incorrect business decisions
- Failed ML models
- Lost customer trust
- Wasted engineering time

Here's how I approach building data quality frameworks that actually catch issues before they impact the business.

## Data Quality Dimensions

A comprehensive data quality framework should cover:

### 1. **Completeness**
- Are all required fields present?
- Is data missing from expected sources?

### 2. **Accuracy**
- Does the data reflect reality?
- Are calculations correct?

### 3. **Consistency**
- Is data consistent across systems?
- Do related fields align?

### 4. **Timeliness**
- Is data arriving on schedule?
- Is it fresh enough for its use case?

### 5. **Validity**
- Does data conform to expected formats?
- Are values within acceptable ranges?

### 6. **Uniqueness**
- Are there unexpected duplicates?
- Is the primary key truly unique?

## Our Tech Stack

At ING Bank, we used:
- **Apache Griffin**: Open-source data quality platform
- **dbt tests**: For SQL-based testing
- **Great Expectations**: For Python-based validation
- **Airflow**: For orchestration and alerting

## Implementation Approach

### Layer 1: Schema Validation

First line of defense—validate schema on ingestion:

```python
from pyspark.sql.types import *

expected_schema = StructType([
    StructField("user_id", StringType(), False),
    StructField("event_time", TimestampType(), False),
    StructField("amount", DecimalType(10,2), True)
])

# Validate incoming data
df = spark.read.schema(expected_schema).json(input_path)
```

### Layer 2: dbt Tests

Built-in and custom tests in dbt:

```sql
-- models/users.sql
{{ config(
    materialized='table'
) }}

SELECT 
    user_id,
    email,
    created_at,
    updated_at
FROM {{ source('raw', 'users') }}

-- tests/users_email_format.sql
SELECT email
FROM {{ ref('users') }}
WHERE email NOT LIKE '%@%.%'
```

### Layer 3: Apache Griffin

For cross-system accuracy checks:

```json
{
  "name": "order_amount_reconciliation",
  "measure.type": "accuracy",
  "data.sources": [
    {
      "name": "source",
      "connectors": [{"type": "postgres", "config": {...}}]
    },
    {
      "name": "target", 
      "connectors": [{"type": "bigquery", "config": {...}}]
    }
  ],
  "evaluate.rule": {
    "rules": [
      {
        "dsl.type": "griffin-dsl",
        "rule": "source.amount = target.amount"
      }
    ]
  }
}
```

### Layer 4: Great Expectations

For complex validations:

```python
import great_expectations as ge

df = ge.read_csv('transactions.csv')

# Basic expectations
df.expect_column_values_to_not_be_null('transaction_id')
df.expect_column_values_to_be_unique('transaction_id')

# Custom business rules
df.expect_column_values_to_be_between(
    'amount', 
    min_value=0, 
    max_value=1000000
)

df.expect_column_pair_values_A_to_be_greater_than_B(
    'end_date', 
    'start_date'
)
```

## Monitoring Strategy

### Real-Time Alerts

Set up alerts for critical issues:

```python
# Airflow DAG
from airflow.operators.python import PythonOperator

def check_data_quality():
    # Run quality checks
    results = run_quality_checks()
    
    # Alert on failures
    if results['critical_failures'] > 0:
        send_slack_alert(results)
        raise AirflowException("Critical data quality issues")

quality_check = PythonOperator(
    task_id='data_quality_check',
    python_callable=check_data_quality
)
```

### Dashboard for Visibility

Created a data quality dashboard showing:
- Quality score trends over time
- Failed checks by severity
- Data freshness metrics
- Schema drift detection

## Quality Metrics

Track these KPIs:

1. **Data Quality Score**: % of passing checks (target: > 99%)
2. **Mean Time to Detection (MTTD)**: How fast we detect issues
3. **Mean Time to Resolution (MTTR)**: How fast we fix issues
4. **Coverage**: % of tables with quality checks

## Organizational Practices

### 1. Data Contracts

Define explicit contracts between producers and consumers:

```yaml
# data_contract.yml
table: user_events
owner: analytics-team
description: User behavior events
schema:
  - name: user_id
    type: string
    required: true
    pii: true
  - name: event_type
    type: string
    required: true
    enum: [click, view, purchase]
  - name: timestamp
    type: timestamp
    required: true
freshness: 
  expected: 15 minutes
  warn_after: 30 minutes
  error_after: 60 minutes
```

### 2. Data Quality SLAs

Set clear expectations:
- **Tier 1 (Critical)**: 99.9% quality, < 5 min detection
- **Tier 2 (Important)**: 99.5% quality, < 30 min detection
- **Tier 3 (Nice-to-have)**: 95% quality, daily checks

### 3. Ownership Model

Every dataset needs:
- **Data Owner**: Business stakeholder
- **Data Steward**: Technical owner
- **Quality Champion**: Monitors and improves quality

## Lessons Learned

1. **Start simple**: Don't try to check everything at once
2. **Focus on impact**: Prioritize checks that catch real business issues
3. **Make it easy**: If tests are hard to write, they won't get written
4. **Fix the root cause**: Don't just alert, fix the underlying issue
5. **Measure and improve**: Track metrics and iterate

## Real Results

After implementing this framework:
- Reduced data incidents by **70%**
- Detected issues **5x faster**
- Increased trust in data across the organization
- Saved countless hours of debugging

## Conclusion

Good data quality doesn't happen by accident. It requires:
- The right tools and processes
- Clear ownership and accountability
- Continuous monitoring and improvement

Start with your most critical data and expand gradually. The investment pays off quickly.

---

*What's your approach to data quality? Let's discuss on [LinkedIn](https://www.linkedin.com/in/okankurnaz)!*

