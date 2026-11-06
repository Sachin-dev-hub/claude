# Data Platform — Interview Q&A (JD: data lakes, ingestion, processing)

**The JD is for an AWS data platform — know these at a high level. Simple language.**

---

## Q1. What is a data lake vs a data warehouse?

- **Data warehouse:** structured, cleaned, ready for BI reports (tables). E.g., Redshift, Snowflake.
- **Data lake:** raw data of ANY format (CSV, JSON, logs, images) stored cheaply (S3). Process later. Foundation of big data.
- **Lakehouse** = data lake + warehouse features (Iceberg/Hudi — JD mentions them).
- **On AWS:** S3 (lake storage) + Glue (catalog/ETL) + Athena (query S3 with SQL, no servers).

---

## Q2. ETL vs ELT?

- **ETL:** Extract → Transform → Load. Transform BEFORE loading. (Traditional, Glue Jobs.)
- **ELT:** Extract → Load → Transform. Load raw first, transform when needed. (Modern data lakes — store raw, transform on demand with Athena/Spark.)

---

## Q3. How would you deploy a data ingestion/processing workload on EKS?

- Containerize the ingestion job (e.g., Spark or custom Python) with Docker.
- **Streaming (real-time):** Kafka → consumer pods (StatefulSet) → process → write to S3/data store. SQS/SNS/EventBridge for event triggers.
- **Batch (scheduled):** CronJob or Argo Workflows for scheduled processing.
- Scale with HPA/Cluster Autoscaler based on queue length (custom metrics from SQS).
- Monitor with Prometheus + CloudWatch, alert on lag/failures.
- Keep raw data in S3 with proper IAM/bucket policies (data platform security).

---

## Q4. AWS data services — one line each (JD mentions Glue, Athena, Lake Formation)

| Service | One-liner |
|---|---|
| **S3** | The storage — where raw/processed data lives (data lake) |
| **Glue** | Serverless ETL + data catalog (finds/describes your data) |
| **Athena** | Run SQL queries directly on S3 — no servers, pay per query |
| **Lake Formation** | Security & permissions for the data lake |
| **Redshift** | Data warehouse — SQL analytics on structured data |
| **Kafka (MSK)** | Streaming — real-time data pipeline |
| **Kinesis** | AWS streaming for real-time ingestion |

---

## Q5. What is the data engineer's workflow on a platform?

1. **Ingest** raw data → S3 (from Kafka, SQS, batch uploads, APIs).
2. **Catalog** it → Glue Data Catalog (knows what tables exist).
3. **Process/transform** → Glue Jobs / Spark on EKS (ETL).
4. **Query/analyze** → Athena (SQL on S3) or Redshift for BI.
5. **Serve** → applications/dashboards read the results.

Your role as DevOps/Platform: make steps 1–4 scalable, automated, secure, monitored, and cost-efficient — EKS for Spark workloads, IAM for security, CI/CD for the jobs, CloudWatch/Prometheus for monitoring.

---

## Quick terms to remember

| Term | One-liner |
|---|---|
| Data lake | Raw data in S3, any format |
| Data warehouse | Cleaned, structured, for BI |
| Lakehouse | Lake + warehouse features |
| ETL | Transform before load |
| ELT | Load raw first, transform later |
| Iceberg / Hudi | Table formats that make S3 data updatable/queryable (like warehouse tables) |
| Glue | Serverless ETL + catalog |
| Athena | SQL directly on S3 |