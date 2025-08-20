# GCP Links and Notes
> [!NOTE]
> Brought to you by Jeff Davis and ROI Training


## Demos, code, activities

* [gcp-demos](https://github.com/roitraining/gcp-demos) Github repository
* [Do-it-now activities](https://roitraining.github.io/gcp-demos/)
* [Cloud Skills Boost](https://www.cloudskillsboost.google/)


---

## <img src="https://icon.icepanel.io/GCP/svg/Cloud-Storage.svg" style="width:24px"> Cloud Storage

### Links
- [Hierarchical namespace](https://cloud.google.com/storage/docs/hns-overview)
- [Availability and replication](https://cloud.google.com/storage/docs/availability-durability)
- [Caching](https://cloud.google.com/storage/docs/caching)
- [Batching](https://cloud.google.com/storage/docs/batch)
- 
  
### Hierarchical namespaces
1. Big performance improvements for certain workloads; Hadoop and Spark expecially

### Replication
1. 99.9% SLA on write replication within an hour
2. Turbo replication is 100% in 15 minutes

### Python Client
1. Mostly synchronous
2. Transfer manager is async
   1. Current and future method is using processes, so can do parallel writes but is copying files from disk, not in-memory

---

## <img src="https://icon.icepanel.io/GCP/svg/BigQuery.svg" style="width:24px"> BigQuery
### Links

* [Dataform](https://cloud.google.com/dataform/docs/overview)
* [Bigquery pricing](https://cloud.google.com/bigquery/pricing?hl=en#storage)

### Cost management

1. Queries with **high compute/read ratios are cheaper with on-demand pricing**, though they will likely be slower than if you through a lot of slots at them
   1. Example
      1. Creating a denormalized table from large normalized tables
      2. Joins are computationally expensive
      3. Even large normalized tables are relatively small
      4. roi-bq-demos.bq_demo tables are 2.5TB in size, but generate a 16TB denormalized table
      5. 10x more expensive with compute capacity vs. on-demand
      6. However, much slower with on-demand vs. using 10K slots due to waiting
2. Queries with **high read/compute ratios are cheaper with compute capacity** pricing
3. Tables with very **high compression ratios** are likely cheaper with physical storage pricing
   1. It has to be better than the 1.74x compression, because with physical pricing you're also paying for time travel and fail-safe storage with physical pricing.
   2. Better than 3-4x compression is typically a good indicator that physical pricing is a win
4. Caching with partitions - query results are read from cache if the partitions read haven't changed, even if other partitions have changed.


### Dataform

1. Tooling to manage transformations using Gitops (ELT)
2. Create a Dataform repo to connect to a Git repo
3. Git repo contains project details
   1. dataform.json -> project config
   2. package.json -> dependencies
   3. definitions (SQLX)
      1. table declarations
      2. transformation logic
      3. javascript commands to affect things like IAM
   4. includes
      1. javascript code to include
      2. can be referenced by javascript in sqlx
4. Uses dataform core as a meta-language
   1. Gets compiled to sql

### Data canvas

1. GUI that overlays regular table ui, regular query editor UI, Notebooks UI
2. Integrates GenAI for writing queries and notebook cells
3. Intended to provide visual thinkes a canvas they can use to explore data
4. Feels very POC
5. Hard to view even on ultrawide

### Biglake tables for Apache Iceberg
1. Basically, swapping storage engine from native to Iceberg in GCS
2. Retains much of the native storage functionality

---

## <img src="https://icon.icepanel.io/GCP/svg/Dataproc.svg" style="width:24px"> Dataproc
### Links
- [Dataproc templates](https://github.com/GoogleCloudPlatform/dataproc-templates/tree/main/java/src/main/java/com/google/cloud/dataproc/templates)   
- [Google Serverless for Apache Spark pricing](https://cloud.google.com/dataproc-serverless/pricing?hl=en)
  
### Local SSDs
1. When provisioned, these are automically used for HDFS and scratch data like shuffle outputs are stored here
2. With no ssds, shuffle data stored on **boot disks**
   
### Auto-created buckets
1. **Staging bucket**: Used to stage cluster job dependencies, job driver output, and cluster config files. Also receives output from the gcloud CLI gcloud dataproc clusters diagnose command.
2. **Temp bucket**: Used to store ephemeral cluster and jobs data, such as Spark and MapReduce history files.

### HDFS and GCS
1. Can eliminate use of disks for HDFS, and set **defaultFS** as a bucket
2. Caching is available; **only for Spark**
3. Block size selection
   1. Task Overhead vs. Parallelism Trade-off
      1. Too small (< 64MB): High task startup overhead, scheduler bottlenecks
      2. Too large (> 1GB): Underutilized cores, memory pressure, poor load balancing
      3. Sweet spot: Usually 128MB-512MB per partition
3. DistCP
   1. There are two primary models for using distcp for this migration
      1. Push Model (simplest): Run distcp from your on-premises Hadoop cluster.
      2. Pull Model (recommended for large scale): Run distcp from a temporary Dataproc cluster in Google Cloud, pulling data from on-prem.
   2. **Incremental Copies**: For ongoing data synchronization, distcp can be run with the -update option to copy only new or modified files. This is useful for maintaining a hybrid environment during a phased migration.
 
### Autoscaling

1. **Scale-up factor**
   1. Higher scale-up factor (e.g., 0.8-1.0): Ideal for latency-sensitive workloads or when you expect sudden, large spikes in demand (like your overnight batch jobs). It prioritizes performance and ensures jobs get resources quickly. The risk is potentially over-provisioning slightly if the spike is short-lived.
   2. Lower scale-up factor (e.g., 0.2-0.5): Suitable for workloads where a slight delay in scaling up is acceptable, or if you want to be more conservative with costs during scaling. This can prevent rapid, potentially unnecessary additions of nodes.
2. **Scale-down factor**
   1. Higher scale-down factor (e.g., 0.8-1.0): Best for cost optimization, as it quickly reclaims idle resources. However, it carries a higher risk of removing workers that might still be needed for upcoming small tasks or if the workload is bursty, potentially leading to immediate scale-up again. **Example**: At 9 AM, multiple large jobs are submitted. The YARN scheduler can't allocate containers for them, so pending_memory skyrockets. The autoscaling policy sees this and triggers a scale-up operation. It calculates the number of new nodes to add as ceil(0.8 * current_worker_count). If your cluster has 10 workers, it will attempt to add ceil(0.8 * 10) = 8 new workers in a single operation.
   2. Lower scale-down factor (e.g., 0.2-0.5): Prioritizes stability and avoids "thrashing" (rapid scale-up and scale-down). It's more forgiving for fluctuating workloads but might keep idle resources for longer, incurring higher costs.
3. **Structured streaming**: While Dataproc autoscaling primarily relies on YARN metrics (which are well-suited for batch), applying it to Spark Structured Streaming requires careful consideration and potentially different tuning strategies. Structured Streaming jobs are long-running and continuously consuming data, making the concept of "pending memory" less straightforward.
   1. Dedicated Clusters: Run streaming jobs on dedicated Dataproc clusters that are sized appropriately for their expected baseline load.
4. By default, YARN uses memory metrics for resource allocation. For CPU-intensive applications, a best practice is to configure YARN to use the Dominant Resource Calculator. To do this, set the following property when you create a cluster:
   1. `capacity-scheduler:yarn.scheduler.capacity.resource-calculator=org.apache.hadoop.yarn.util.resource.DominantResourceCalculator`

### Dataproc serverless

1. Renamed to **Google Cloud Serverless for Spark**
2. Basically just google-managed ephemeral clusters
   1. No big benefit in speed (still like 1.5-2 minutes to create cluster)
   2. Not really a great benefit in cost
3. Use cases
   1. High variance in workloads; avoid clusters costs when not running jobs
   2. Can invoke pyspark interactive jobs through BigQuery console
      1. Create notebook with spark template
      2. Runtime for notebook
      3. Dataproc serverless for running spark code
      4. Big startup times; not good for quick interactive stuff
4. Exposed through **dataproc > serverelss > batches**
5. Run from GUI, CLI, API
   1. **basic CLI**
      ```bash
      gcloud dataproc batches submit pyspark \
      gs://your-code-bucket/process_sales.py \
      --region=us-central1 \
      --jars=gs://spark-lib/bigquery/spark-bigquery-with-dependencies_2.12-0.26.0.jar \
      -- --input-data-path=gs://your-sales-data/raw/ \
      --output-results-path=gs://your-sales-data/processed/
      ```
   2. **from template**
      ```bash
      gcloud dataproc batches submit --project jwd-gcp-demos --region us-central1 spark --batch batch-50ee --class com.google.cloud.dataproc.templates.main.DataProcTemplate --version 1.2 --jars file:///usr/lib/spark/connector/spark-avro.jar,gs://dataproc-templates-binaries/latest/java/dataproc-templates.jar --subnet default -- --template GCSTOBIGQUERY --templateProperty project.id=jwd-gcp-demos --templateProperty gcs.bigquery.input.location=jwd-gcp-demos/orders_partitioned/order_date=2018-01-01/000767_0 --templateProperty gcs.bigquery.input.format=parquet --templateProperty gcs.bigquery.output.dataset=demos --templateProperty gcs.bigquery.output.table=dataproc_output
      ```
      1. Note that Serverless for Apache Spark templates are not the same as Dataproc Workflow templates.

6. Pricing
   1. Based on DCUs, accelerators, shuffle
   2. Billed per second (5 minute min for accelerators)
   3. vCPU is 0.6DCU
   4. Memory is .1 or .2 DCU
   5. By default, min of 12DCU for workload
      1. driver = 4/16
      2. 2 executors 4/16 each
      3. you can customize with spark properties
   6. Shuffle and accelerator pricing are in docs

### IAM
1. Should use a user-generated service account ([link](https://cloud.google.com/dataproc/docs/concepts/configuring-clusters/service-accounts))
2. Needs roles
   1. dataproc.worker
   2. any of the roles required to interact with other google cloud services

### Maintenance
1. gcloud dataproc clusters update is used for num-workers or labels

### Right-sizing machines
1. Can enable custom metrics when creating clusters
2. Yarn pending metrics; YARN interface
3. Spark interface

---

## <img src="https://icon.icepanel.io/GCP/svg/Dataflow.svg" style="width:24px"> Dataflow

* placeholder
---

## <img src="https://icon.icepanel.io/GCP/svg/Cloud-Composer.svg" style="width:24px"> Composer
### Links
* [Architecture](https://cloud.google.com/composer/docs/composer-versioning-overview)
* [Write Airflow DAGs](https://cloud.google.com/composer/docs/composer-3/write-dags)
* [Test DAGs](https://cloud.google.com/composer/docs/composer-3/test-dags)
* [Local Composer Environment](https://cloud.google.com/composer/docs/composer-2/run-local-airflow-environments)
* [Google providers](https://airflow.apache.org/docs/apache-airflow-providers-google/stable/index.html)

### General
1. Airflow only supports up to Python 3.11
2. Composer evironment setup time is like 22+ minutes
3. Version 2 builds a cluster for you; version 3 runs your stuff as pods on a google-managed cluster ([link](https://cloud.google.com/composer/docs/composer-versioning-overview))

### IAM Stuff
1. Your composer environment service account needs permissions...
   1. composer.worker
   2. for the dataproc example it needs
      1. dataproc.editor
      2. bigquery.jobuser
      3. iam.serviceaccountuser
      4. storage.objectviewer
      5. bigquery.dataowner on the dataset

### Pricing
1. See [docs](https://cloud.google.com/composer/pricing#cloud-composer-pricing)
2. Costs are non-trivial
3. Very different models for 2 and 3
   
### Airflow DAG development
1. DAG development with Cloud Composer can be frustrating because of all the lags
   1. You have to copy DAG to GCS
   2. You have to wait for Composer to see and process the DAG - by default, this entails long delays
3. You can run a composer-like environment locally for dev/testing
   1. See [link](https://cloud.google.com/composer/docs/composer-2/run-local-airflow-environments)
   2. Here's the step by step
      1. clone the repo
         ```bash
         git clone https://github.com/GoogleCloudPlatform/composer-local-dev.git
         cd composer-local-dev
         ```
      2. Make sure you're using a supported version of Python
         ```bash
         pyenv local 3.11.13
         python -m venv .venv
         source .venv/bin/activate
         pip install .
         ```
      3. Create the local environment using a named Composer image
         ```bash
         composer-dev create \
         --from-image-version composer-3-airflow-2.10.5-build.10 \
         composer-local
         ```
      4. Start the environment
         ```bash
         composer-dev start 
         ```

   3. To emulate an existing composer environment:
      ```bash
      composer-dev create LOCAL_ENVIRONMENT_NAME \
      --from-source-environment ENVIRONMENT_NAME \
      --location LOCATION \
      --project PROJECT_ID \
      --port WEB_SERVER_PORT \
      --dags-path LOCAL_DAGS_PATH
      ```
---

## <img src="https://icon.icepanel.io/GCP/svg/PubSub.svg" style="width:24px"> Pub/Sub
### Bigquery subscriptions
1. Primarily intended for raw ingestion
2. Can do transformation using SMT UDFs
3. For complex transformation, exactly once procecssing, aggregation - Dataflow
   1. Could write dupes to BQ and handle in the warehouse
   2. If doing transformation, should test performance
   3. Without transformation, expect lower latency with sub; but test
4. Pricing
   1. $50/TB (first 10TB free)
   2. Cost/benefit vs. Dataflow; test
      1. 1TB/day through pub/sub to BQ (about $2700)
      2. 1TB/day through pub/sub to dataflow to BQ with minimal cluster (about $2700)
---

## <img src="https://icon.icepanel.io/GCP/svg/Identity-And-Access-Management.svg" style="width:24px"> Security stuff
### Links
* [Auth decision tree](https://cloud.google.com/docs/authentication#auth-decision-tree)
  
### Impersonating service accounts
1. Alternative to running with key file or with personal account
   1. Appropriate for testing things that will run with service account
   2. Better than creating and distributing keys
   3. Requires user to have permissions to impersonate
      1. Service Account OpenID Connect Identity Token Creator role
      2. Service Account Token Creator role
   4. https://cloud.google.com/docs/authentication#auth-decision-tree
   
2. Example for single call
   ```bash
   gcloud storage buckets list --impersonate-service-account=SERVICE_ACCT_EMAIL
   ```
3. Example - default for all calls
   ```bash
   gcloud config set auth/impersonate_service_account SERVICE_ACCT_EMAIL
   ```

4. Example for ADC
   ```bash
   gcloud auth application-default login --impersonate-service-account SERVICE_ACCT_EMAIL
   ```

5. Example for REST call
   ```bash
   curl -X GET \
    -H "Authorization: Bearer $(gcloud auth print-access-token --impersonate-service-account=PRIV_SA)" \
    "https://cloudresourcemanager.googleapis.com/v3/projects/PROJECT_ID"
    ```

6. Example granting permissions
   ```bash
   gcloud iam service-accounts add-iam-policy-binding PRIV_SA \
    --member=user:CALLER_ACCOUNT --role=roles/iam.serviceAccountTokenCreator --format=json
   ```