# AWS Serverless Data Lake Workshop

This workshop introduced AWS serverless data analytics services **(i.e. **Kinesis Firehose**, **S3**, **Glue**, **Athena**, and **QuickSight**)** to build an end-to-end, cloud-native data lake pipeline. Below are the screenshots and brief descriptions for each required task.

---

## Lab 1: Data Ingestion & Storage

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

* **Screenshot:** Kinesis Firehose Stream Metrics ![image](https://hackmd.io/_uploads/HJfv8M30xl.png)
* **Screenshot:** S3 Raw Data Partitions after Streaming ![image](https://hackmd.io/_uploads/BJVt8G20eg.png)

---

## Lab 2: Data Cataloging & ETL

### **2.1: Create a Crawler and Define Table Schema**

We created a **Glue Crawler** to automatically scan the `/raw/` folder in the S3 data lake and infer schema metadata. The crawler stored results in the **`sdl-demo-data`** database, creating the table **`sdl_immersion_day_310649874981`**.

After running the crawler, the table was visible in the Glue Data Catalog with JSON classification. We then edited the schema manually to verify column names, data types, and partition keys (`year`, `month`, `day`, `hour`).

**Evidence:**

* **Screenshot:** Glue table created after crawler run ![image](https://hackmd.io/_uploads/S1-nwz2Rgx.png)
* **Screenshot:** Schema metadata editing ![image](https://hackmd.io/_uploads/S162Dz30xe.png)

---

### **2.2: Create and Run Transformation Job in Glue Studio**

We used **AWS Glue Studio** to visually design an ETL job named **`transform-json-to-parquet`**. The job reads JSON data from S3, applies schema transformations, and writes the cleaned output in **Parquet (Snappy)** format to the destination path `/compressed-parquet/`.

**Transformations applied:**

* Dropped the redundant `color` field.
* Renamed `dateSoldSince` → `date_start`, and `dateSoldUntil` → `date_until`.
* Converted to optimized Parquet output for efficient querying in Athena.

**Evidence:**

* **Screenshot:** Visual ETL job pipeline ![image](https://hackmd.io/_uploads/S1CTwf3Cgl.png)
* **Screenshot:** Auto-generated PySpark script ![image](https://hackmd.io/_uploads/Hy5RDMnRll.png)
* **Screenshot:** Output Parquet files in S3 ![image](https://hackmd.io/_uploads/Sk9l_fnRxl.png)
* **Screenshot:** Job run metrics confirming success ![image](https://hackmd.io/_uploads/SyWWOM20xe.png)

---

### **2.3: Interactive ETL Code Development (Jupyter Session)**

We then used **AWS Glue Interactive Sessions** (PySpark in Jupyter) to test advanced ETL logic interactively. This allowed us to clean and transform data, then write outputs partitioned by `department` into `/output-etl-nb-jobs/byDepartment/`.

**Evidence:**

* **Screenshot:** Interactive session output files ![image](https://hackmd.io/_uploads/rJbMuzh0ll.png)
* **Screenshot:** Partitioned department folders ![image](https://hackmd.io/_uploads/ryAf_f2Rxe.png)

---

### **2.4: Add a Trigger Function in AWS Glue**

To automate daily ETL runs, we created a **Glue Trigger** that launches the `transform-json-to-parquet` job at a scheduled time each evening. This enables continuous updates as new streaming data lands in S3.

**Evidence:**

* **Screenshot:** Trigger configuration and review ![image](https://hackmd.io/_uploads/HJsQ_G3Age.png)

---

### **Summary of Lab 2 Results**

* **Crawler**: Successfully created and generated schema for JSON data.
* **Glue Studio Job**: Converted data to Parquet format using a drag‑and‑drop ETL workflow.
* **Interactive Session**: Tested custom PySpark transformations interactively.
* **Trigger**: Automated daily job scheduling for continuous updates.

Overall, this lab demonstrated a full end-to-end **data cataloging and ETL workflow** using AWS Glue from schema discovery and transformation to automation and partitioned storage.

## Lab 3: Data Analytics & Visualization

### **3.1: Create Database and Query with Athena**

We used **Amazon Athena** to query the GDELT open dataset stored in S3. A new database named **`gdelt`** was created to store tables for analytical querying.

**Steps and Queries Executed:**

1. **Database Creation**: initialized the workspace for GDELT data analysis.

   * **Screenshot:** Database creation query in Athena ![image](https://hackmd.io/_uploads/HyrGKMnRll.png)

2. **Query: Count Events per Year**: summarized total global events by year.

   * **Screenshot:** Query results showing yearly event counts ![image](https://hackmd.io/_uploads/SySXFzhRxg.png)

3. **Query: Top 10 Event Categories by Count**: grouped by event code and joined with event descriptions.

   * **Screenshot:** Top 10 event categories and counts ![image](https://hackmd.io/_uploads/BJGNYf2Axl.png)

4. **Query: Obama Events per Year**: filtered the dataset for `actor1Name = 'BARACK OBAMA'` and aggregated event counts by year.

   * **Screenshot:** Obama yearly event activity ![image](https://hackmd.io/_uploads/rJPHFG2Ceg.png)

5. **Query: Obama vs. Merkel Interaction Events**: compared event codes between `actor1Name = 'BARACK OBAMA'` and `actor2Name = 'ANGELA MERKEL'`.

   * **Screenshot:** Obama–Merkel event comparison results ![image](https://hackmd.io/_uploads/rJnIFf2Reg.png)

---

### **3.2: Data Visualization with Amazon QuickSight**

We used **Amazon QuickSight** to create visual dashboards for both stream data and GDELT analytics.

#### **a) Stream Data Visualization (S3 → Athena → QuickSight)**

Using the `sdl-demo-data.raw` dataset, we visualized sales metrics by department and product.

* Chart Type: **Horizontal Bar Chart**
* Y-Axis: `department`
* Value: `Sum(price)`
* Group/Color: `product`
* **Screenshot:** Overall department-product price visualization ![image](https://hackmd.io/_uploads/Sy4uFG3Axg.png)

**Focus View:** Applied filter for `department = 'Toys'` to isolate sales within the Toys category.

* **Screenshot:** Filtered visualization (Toys only) ![image](https://hackmd.io/_uploads/B1XKYfh0ll.png)

#### **b) GDELT Event Visualization**

Visualized event counts grouped by `description` and `eventcode`, highlighting the interaction between Obama and Merkel.

* Chart Type: **Vertical Bar Chart**
* X-Axis: `description`
* Value: `Sum(nb_events)`
* Group/Color: `eventcode`
* **Screenshot:** GDELT vertical bar visualization ![image](https://hackmd.io/_uploads/r1xcFzh0gg.png)

---

### Summary of Accomplishments

| Task                    | AWS Service                   | Outcome                                                                                                       |
| ----------------------- | ----------------------------- | ------------------------------------------------------------------------------------------------------------- |
| **Batch Ingestion**     | Amazon S3                     | Uploaded prepared datasets for structured storage in a data lake.                                             |
| **Real-Time Streaming** | Kinesis Data Firehose → S3    | Delivered live event data into partitioned folders for continuous ingestion.                                  |
| **Schema Discovery**    | AWS Glue Crawler              | Automatically scanned `/raw/` data and created the `sdl_immersion_day` table in the `sdl-demo-data` database. |
| **ETL Design**          | AWS Glue Studio               | Built a drag-and-drop ETL pipeline converting JSON to optimized Parquet (Snappy).                             |
| **Interactive ETL**     | AWS Glue Interactive Sessions | Used Jupyter notebooks for ad-hoc PySpark transformations and partitioned writes.                             |
| **Automation**          | AWS Glue Trigger              | Scheduled daily job executions for automated data refresh.                                                    |
| **SQL Analytics**       | Amazon Athena                 | Queried large-scale datasets (GDELT) and performed joins, filters, and aggregations.                          |
| **Visualization**       | Amazon QuickSight             | Created visual dashboards for both sales stream and GDELT event analytics.                                    |

---

### Reflection

This hands-on workshop demonstrated how AWS serverless services integrate seamlessly to form a **scalable, low-maintenance data lake architecture**.

* **Scalable Ingestion**: Kinesis Firehose and S3 supported both batch and streaming data with no manual server management.
* **Automated Processing**: Glue handled schema detection, transformation, and scheduling of recurring ETL jobs.
* **On-Demand Analytics**: Athena provided instant SQL access to S3 data without provisioning compute resources.
* **Visual Insights**: QuickSight turned the processed datasets into rich, shareable dashboards for business intelligence.

Together, these tools enabled a full end-to-end pipeline, from raw data ingestion to visualization, showing us the **efficiency, flexibility, and cost-effectiveness** of a serverless analytics workflow.

---

### Appendix: Common Gotchas & Fixes

| Issue                           | Likely Cause                                   | Recommended Fix                                                                                                                                                |
| ------------------------------- | ---------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Athena LOCATION Error**       | Incorrect S3 URI format (e.g., `s3://s3:/...`) | Use correct format `s3://bucket/path/`. If corrupted, run `DROP TABLE` and recreate.                                                                           |
| **QuickSight Dataset Missing**  | Region mismatch or missing IAM permissions     | Ensure Athena and QuickSight use the same region; grant access under **QuickSight → Security & Permissions** for both the data lake and Athena results bucket. |
| **Preview Fails in QuickSight** | SPICE cache issues or incorrect data types     | Refresh SPICE dataset, verify numeric columns (e.g., `price`), clear filters, and reload.                                                                      |
| **Glue Job Timeout**            | Insufficient DPUs for data size                | Increase DPU count in job settings or optimize with partitioned inputs.                                                                                        |
| **Trigger Not Running**         | Misaligned time zone or dependency mismatch    | Confirm UTC schedule matches local expectation and verify job link is correct.                                                                                 |
