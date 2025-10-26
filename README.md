# AWS Serverless Data Lake Workshop

This workshop introduced AWS serverless data analytics services — **Kinesis Firehose**, **S3**, **Glue**, **Athena**, and **QuickSight** — to build an end-to-end, cloud-native data lake pipeline. Below are the screenshots and brief descriptions for each required task.

---

## Lab 1 — Data Ingestion & Storage (25 pts)

### 1) Batch Ingestion into S3

* Imported prepared data files into S3 using CLI.

### 2) Option 2: Real-Time Data Streaming into S3

Used **Amazon Kinesis Data Firehose** to stream data from **Kinesis Data Generator (KDG)** into S3.

**Configuration summary:**

* Delivery stream: `sdl-firehose-stream`
* Source: Direct PUT → Destination: Amazon S3
* Bucket: `sdl-immersion-day-<ACCOUNT_ID>`
* Prefix (Hive-style): `raw/year=!{timestamp:yyyy}/month=!{timestamp:MM}/day=!{timestamp:dd}/hour=!{timestamp:HH}/`
* Buffer interval: 60s; Compression: GZIP

**Evidence:**

* ✅ **Screenshot:** Kinesis Firehose Stream Metrics — `<img width="468" height="237" alt="image" src="https://github.com/user-attachments/assets/35a1bb5e-9f8a-4e01-b135-a8ef07e413ed" />
`
* ✅ **Screenshot:** S3 Raw Data Partitions after Streaming — `<img width="468" height="224" alt="image" src="https://github.com/user-attachments/assets/dc6732f8-7dec-4e30-9ff6-605c11378204" />
`
* ✅ **Screenshot:** S3 partitions under `/raw/year=…/month=…/day=…/hour=…/` — `screenshots/lab1-s3-raw-partitions.png`

---

## 🧭 Lab 2 — Data Cataloging & ETL

### 3) Create a Crawler to Auto-Discover Schema

Created **Glue Crawler** `sdl-demo-crawler` to scan `/raw/` and generate the `raw` table in database `sdl-demo-data`.

* ✅ **Screenshot:** Crawler setup — `screenshots/lab2-crawler-setup.png`
* ✅ **Screenshot:** Crawler run complete & tables discovered — `screenshots/lab2-crawler-results.png`

### 4) Create a Transformation Job in Glue Studio

Built visual ETL job **`transform-json-to-parquet`** to:

* Drop field: `color`
* Rename: `dateSoldSince → date_start`, `dateSoldUntil → date_until`
* Output: `s3://sdl-immersion-day-<ACCOUNT_ID>/compressed-parquet/` in **Parquet (Snappy)**

**Evidence:**

* ✅ **Screenshot:** Visual job graph — `screenshots/lab2-glue-job-visual.png`
* ✅ **Screenshot:** Run status = *Succeeded* — `screenshots/lab2-glue-job-run.png`
* ✅ **Screenshot:** Output Parquet files — `screenshots/lab2-s3-compressed-parquet.png`

### 5) Develop & Test ETL Scripts in Jupyter Notebook

Used **Glue Interactive Sessions (Jupyter)** to experiment with PySpark.

* Uploaded notebook: *Lab 2.3 - Advanced Data Preparation.ipynb*
* Executed transformations and wrote partitioned outputs.
* ✅ **Screenshots:** Notebook UI, executed cells, and S3 outputs — `screenshots/lab2-notebook-1.png`, `lab2-notebook-2.png`, `lab2-notebook-output.png`

### 6) Add a Trigger Function in AWS Glue

Created a **daily scheduled trigger** for `transform-json-to-parquet` job.

* ✅ **Screenshot:** Trigger configuration — `screenshots/lab2-trigger-config.png`
* ✅ **Screenshot:** Trigger active — `screenshots/lab2-trigger-active.png`

---

## 📊 Lab 3 — Data Analytics & Visualization

### 7) Create Glue Database using Athena

Created **`gdelt`** database and external tables for GDELT open dataset. Verified queries:

* Events per year
* Top 10 event categories (join with `eventcodes`)
* Obama/Merkel event counts

**Key statements (subset):**

```sql
CREATE DATABASE gdelt;

CREATE EXTERNAL TABLE IF NOT EXISTS gdelt.events (
  globaleventid INT,
  day INT,
  monthyear INT,
  year INT,
  fractiondate FLOAT,
  actor1code string,
  actor1name string,
  -- … (columns omitted for brevity)
  sourceurl string
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
WITH SERDEPROPERTIES ('serialization.format'='\t','field.delim'='\t')
LOCATION 's3://gdelt-open-data/events/';
```

**Evidence:**

* ✅ **Screenshot:** DB/table creation — `screenshots/lab3-athena-ddl.png`
* ✅ **Screenshot:** Query: events per year — `screenshots/lab3-athena-events-per-year.png`
* ✅ **Screenshot:** Query: top 10 event categories join — `screenshots/lab3-athena-top10-join.png`
* ✅ **Screenshot:** Query: Obama/Merkel — `screenshots/lab3-athena-obama-merkel.png`

### 8) Visualize Data in QuickSight — *15 pts*

Built two analyses in **Amazon QuickSight**.

**a) Stream Data Visualization** (from `sdl-demo-data.raw` via Athena)

* Visual: **Horizontal bar**

  * Y: `department`
  * Value: `price (Sum)`
  * Color: `product`
* ✅ **Screenshots:** Dataset preview, chart, drill-down — `screenshots/lab3-qS-raw-bar.png`, `lab3-qS-raw-focus-toys.png`

**b) GDELT Visualization** (from `gdelt.qs_events` + `eventcodes`)

* Created `gdelt.qs_events` view for Obama/Merkel date range
* Joined with `eventcodes` and/or used Custom SQL for counts by category
* Visual: **Vertical bar** (X: `description`, Value: `nb_events (Sum)`)
* ✅ **Screenshots:** Join editor, Custom SQL, chart — `screenshots/lab3-qS-gdelt-join.png`, `lab3-qS-gdelt-sql.png`, `lab3-qS-gdelt-bar.png`

---

## 🧾 Summary of Accomplishments

| Task                | AWS Service           | Outcome                                           |
| ------------------- | --------------------- | ------------------------------------------------- |
| Batch Ingestion     | S3                    | Data files imported                               |
| Real-Time Streaming | Kinesis Firehose → S3 | Live data written to Hive-style partitions        |
| Crawler             | Glue                  | `raw` table discovered in `sdl-demo-data`         |
| Visual ETL          | Glue Studio           | JSON → Parquet (Snappy) in `/compressed-parquet/` |
| Interactive ETL     | Glue Interactive      | PySpark notebook transformations                  |
| Trigger             | Glue                  | Daily job schedule configured                     |
| Athena              | Athena + Glue Catalog | GDELT tables created; joins & aggregations run    |
| Visualization       | QuickSight            | Dashboards for stream + GDELT datasets            |

---

## 💡 Reflection

This hands-on built a **serverless data pipeline**: ingestion (**Firehose**), storage (**S3**), catalog/ETL (**Glue**), SQL analytics (**Athena**), and dashboards (**QuickSight**). The stack eliminated server management while handling streaming and large-scale open data.

---

## 🧹 Clean-Up Checklist (post-lab)

* Stop/delete Firehose delivery stream
* Drop Glue tables/databases if not needed
* Delete QuickSight datasets/analyses (optional)
* Empty S3 prefixes (`/raw/`, `/compressed-parquet/`, `/athena-results/`) if desired
* Remove CloudFormation stacks used for workshop

✅ **Screenshot (optional):** Stack deletion confirmations — `screenshots/cleanup-stacks.png`

---

## 📎 Appendix — Common Gotchas & Fixes

* **Athena LOCATION typo**: use `s3://bucket/path/` (not `s3://s3:/...`). If wrong, `DROP TABLE` and recreate.
* **QuickSight no tables**: verify region matches (Athena + QuickSight), grant S3 + Athena access in **Manage QuickSight → Security & permissions**, and add the data lake bucket + Athena results bucket with write permissions.
* **Preview failed** in QuickSight: refresh dataset (SPICE), ensure `price` is numeric, clear filters, and reselect the correct dataset.
