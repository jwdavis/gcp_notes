# BigQuery

## Links

* [Dataform](https://cloud.google.com/dataform/docs/overview)
* [Bigquery pricing](https://cloud.google.com/bigquery/pricing?hl=en#storage)

## Cost management

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


## Dataform

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

## Data canvas

1. GUI that overlays regular table ui, regular query editor UI, Notebooks UI
2. Integrates GenAI for writing queries and notebook cells
3. Intended to provide visual thinkes a canvas they can use to explore data
4. Feels very POC
5. Hard to view even on ultrawide

## Biglake tables for Apache Iceberg
1. Basically, swapping storage engine from native to Iceberg in GCS
2. Retains much of the native storage functionality