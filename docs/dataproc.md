# Dataproc

## Links
- [Dataproc templates](https://github.com/GoogleCloudPlatform/dataproc-templates/tree/main/java/src/main/java/com/google/cloud/dataproc/templates)   
- [Google Serverless for Apache Spark pricing](https://cloud.google.com/dataproc-serverless/pricing?hl=en)
  
## Local SSDs
1. When provisioned, these are automically used for HDFS and scratch data like shuffle outputs are stored here
2. With no ssds, shuffle data stored on **boot disks**
   
## Auto-created buckets
1. **Staging bucket**: Used to stage cluster job dependencies, job driver output, and cluster config files. Also receives output from the gcloud CLI gcloud dataproc clusters diagnose command.
2. **Temp bucket**: Used to store ephemeral cluster and jobs data, such as Spark and MapReduce history files.

## HDFS and GCS
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
 
## Autoscaling

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

## Dataproc serverless

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

## IAM
1. Should use a user-generated service account ([link](https://cloud.google.com/dataproc/docs/concepts/configuring-clusters/service-accounts))
2. Needs roles
   1. dataproc.worker
   2. any of the roles required to interact with other google cloud services

## Maintenance
1. gcloud dataproc clusters update is used for num-workers or labels

## Right-sizing machines
1. Can enable custom metrics when creating clusters
2. Yarn pending metrics; YARN interface
3. Spark interface
