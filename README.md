# Flights-Booking-Project-Demo

## Flight Booking Data Analytics Pipeline

(GCP | Airflow | Spark | BigQuery | CI/CD)

---

## 1. Project Overview

1. This project implements an end-to-end data engineering pipeline for processing and analyzing flight booking data using Google Cloud Platform (GCP) services.
2. The pipeline is fully automated, environment-aware (dev and prod), and CI/CD enabled using GitHub Actions with Workload Identity Federation (WIF), without using service account keys.
3. The system ingests raw CSV data from Google Cloud Storage, processes it using PySpark on Dataproc Serverless, and loads transformed and aggregated data into BigQuery.
4. Apache Airflow (Cloud Composer) orchestrates the complete workflow.

---

## 2. High-Level Architecture and Technologies Used

1. Google Cloud Storage (GCS) – Raw data and job artifact storage
2. Cloud Composer (Apache Airflow) – Workflow orchestration
3. Dataproc Serverless – Serverless Spark execution
4. PySpark – Data transformation and analytics
5. BigQuery – Analytical data warehouse
6. GitHub Actions – CI/CD automation
7. Workload Identity Federation (WIF) – Secure authentication
8. IAM – Fine-grained access control

---

## 3. Data Flow at a Glance

1. GitHub repository triggers GitHub Actions (CI/CD)
2. CI/CD uploads DAGs, Spark jobs, and variables to GCS
3. Airflow DAG is triggered
4. GCS sensor waits for source data
5. Dataproc Serverless executes PySpark job
6. Transformed data is written to BigQuery

---

## 5. Detailed Flow Explanation

---

### 5.1 Source Data (GCS)

1. Flight booking data is uploaded as a CSV file to:

   ```
   gs://sample_bucket-14/flight-booking-analysis-project/source-dev/
   ```

2. This bucket acts as the raw data landing zone for the pipeline.

---

### 5.2 CI/CD with GitHub Actions

#### 5.2.1 Authentication

1. GitHub Actions authenticates to GCP using Workload Identity Federation.
2. No service account keys are used.
3. The GitHub repository identity is trusted by GCP IAM.
4. The pipeline uses the following service account:

   ```
   github-wif@project-66b66f91-b0f9-40ed-9d0.iam.gserviceaccount.com
   ```

---

#### 5.2.2 CI/CD Responsibilities

1. CI/CD runs on every push based on the branch.

   | Branch | Environment |
   | ------ | ----------- |
   | dev    | DEV         |
   | main   | PROD        |

2. The pipeline performs the following steps:

   1. Upload Airflow variables

      * `variables.json` is uploaded to the Composer GCS bucket
      * Imported using `airflow variables import`
   2. Upload Spark job

      * `spark_transformation_job.py` is uploaded to GCS
   3. Upload Airflow DAG

      * DAG file is uploaded to the Composer DAGs folder

3. This ensures code, configuration, and orchestration remain in sync.

---

## 6. Airflow Orchestration (Cloud Composer)

1. The Airflow DAG used is:

   ```
   flight_booking_dataproc_bq_dag
   ```

2. The DAG is responsible for:

   1. Detecting source data
   2. Submitting Dataproc Serverless Spark jobs
   3. Managing environment-specific configuration

---

### 6.1 Environment Awareness

1. The DAG uses Airflow Variables for configuration:

   1. Environment (dev or prod)
   2. GCS bucket name
   3. BigQuery project
   4. BigQuery dataset
   5. Target table names

2. No values are hardcoded.

3. The same DAG runs in both environments using different variables.

---

## 7. Task 1: GCS File Sensor

1. Uses `GCSObjectExistenceSensor`.

2. Waits for the file:

   ```
   flight-booking-analysis-project/source-dev/flight_booking.csv
   ```

3. This ensures:

   1. Data-driven execution
   2. Spark job runs only after data arrives
   3. Empty or failed runs are prevented

---

## 8. Task 2: Dataproc Serverless Spark Job

1. Once the file is detected, Airflow submits a Dataproc Serverless batch job.
2. The job runs using PySpark.
3. Infrastructure is fully managed by GCP, with no cluster management required.
4. Spark job parameters passed include:

   1. Environment
   2. BigQuery project
   3. Dataset
   4. Table names

---

## 9. PySpark Processing Logic

### 9.1 Read Data from GCS

1. Data is read using:

   ```
   spark.read.csv(gs://..., header=True)
   ```

---

### 9.2 Transformations Performed

1. Derived columns:

   1. is_weekend
   2. lead_time_category
   3. booking_success_rate

2. Aggregations:

   1. Route-level analytics
   2. Booking origin insights

---

## 10. Writing Data to BigQuery

1. Spark writes data using the BigQuery Spark Connector.

2. Tables created:

   | Table Name                  | Purpose                   |
   | --------------------------- | ------------------------- |
   | transformed_flight_data_dev | Cleaned and enriched data |
   | route_insights_dev          | Route-level analytics     |
   | origin_insights_dev         | Booking-origin analytics  |

3. Write mode used is `overwrite`.

4. This enables idempotent re-runs.

---

## 11. IAM and Security Model

### 11.1 Dataproc Service Account Permissions

1. The Dataproc execution service account has:

   1. BigQuery Data Editor
   2. BigQuery Job User
   3. Storage Object Viewer

2. This ensures:

   1. Least privilege access
   2. Secure data access
   3. No manual credential handling

---

## 12. Environments Supported

| Feature              | DEV | PROD |
| -------------------- | --- | ---- |
| Same DAG             | Yes | Yes  |
| Same Spark job       | Yes | Yes  |
| Different variables  | Yes | Yes  |
| Different data paths | Yes | Yes  |

1. Environment behavior is driven entirely by `variables.json`.

---

## 13. Key Highlights

1. End-to-end automated data pipeline
2. Serverless Spark execution
3. Airflow-based orchestration
4. CI/CD using GitHub Actions
5. Secure, keyless authentication with WIF
6. Environment-aware design
7. Production-grade logging and error handling
8. Scalable and cost-efficient architecture

---

## 14. Final Summary

1. This project demonstrates how to build a modern, production-grade data engineering pipeline on GCP.
2. It follows industry best practices including:

   1. Declarative orchestration
   2. Secure authentication
   3. Serverless compute
   4. Clean separation of environments
   5. Automated CI/CD

---

If you want next, I can:

* Compress this into a **1-page recruiter README**
* Add a **system architecture diagram**
* Write **resume bullet points** from this
* Create a **LinkedIn project post**

Just say the word 👍
