# BigQuery

## Links

* [Dataform](https://cloud.google.com/dataform/docs/overview)
* [Bigquery pricing](https://cloud.google.com/bigquery/pricing?hl=en#storage)
* [Iceberg](https://cloud.google.com/bigquery/docs/iceberg-tables#read-iceberg-tables-from-spark)

## Performance

1. Partitioning a 7.5B row table has minimal impact when joining to a 75B unpartitioned table
   1. Running the schema-demo normalized query against a partitioned order table saves 15-20% time

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

1. Tooling to manage transformations using GitOps (ELT)
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
4. Uses Dataform core as a meta-language
   1. Gets compiled to sql

## Data canvas

1. GUI that overlays regular table ui, regular query editor UI, Notebooks UI
2. Integrates GenAI for writing queries and notebook cells
3. Intended to provide visual thinks a canvas they can use to explore data
4. Feels very POC
5. Hard to view even on ultrawide

## BigLake
1. User -> Dremel -> Connection/SA -> GCS
2. Makes BigLake the security layer (users granted access there not on GCS)
3. GCS, S3, ABS as backing stores
4. Can support cross-cloud joins
5. Makes BigQuery the standard access mechanism whether data is in native storage or object storage
6. Supports the BigQuery Storage Read/Write APIs
7. Allows BigQuery ML, SDP, and other stuff to run through BQ on data in GCS

## BigLake tables for Apache Iceberg
[image-ref]: https://cloud.google.com/static/bigquery/images/biglake-iceberg-table-arch.png "Architecture"

1. Basically, swapping storage engine from native to Iceberg in GCS
2. Retains much of the native storage functionality
3. Uses BigLake metastore as source of truth for metadata
4. Write requests and background optimizations create new files
   1. DML statements
   2. Streaming inserts

## AEAD

1. Alternative to col-level permissions with masked reader
2. More importantly, allows crypto-shredding
   1. For each user/customer, create and store a keyset row
   2. When ingesting data for that entity, encrypt using the keyset
   3. Delete the keyset row and that data becomes unreadable
3. Supports AAD, so key, data, and supplementary piece of data
   1. To decrypt, must provide key but also AAD

## Analytics Hub
1. Now known as BigQuery sharing (fka Analytics Hub)
2. Effectively, a data asset marketplace interface
3. Can monetize shared data through marketplace
4. Creates a linked resource in your project that references the original published resource
5. Not recommended to publish inside VPCSC
6. Permissions can be granted at exchange and listing level
7. Exchanges can be public or private
8. Listing contains metadata that describes the shared assets and how to consume
9. Can configure egress options to limit export of data by subscribers
10. Some limits
   1.  1000 linked datasets
   2.  10K linked topics
   3.  No table-specific IAM
   4.  And [others](https://cloud.google.com/bigquery/docs/analytics-hub-introduction#limitations)
11. Supports analysis rules
    1.  aggregation threshold (n distinct entities must be present)
    2.  differential privacy
    3.  overlap analysis rule
    4.  uses privacy_policy option in view definition

## Materialized views
1. Can included cache-enabled BigLake tables
2. Limitations
   1. Same org as base tables
   2. Can't be nested
   3. For join views, incremental updates works if only left able has appended data
      1. Put the most frequently changing table first in join operations
   4. Incremental updates aren't used updates or deletions have happened in source
   5. No compute, filtering, or joining based on aggregations
   6. Only specific aggregations supported
3. Max staleness
   1. If last refresh occurred within interval, returns from view (doesn't check source tables)
   2. If outside, tries to combine view with changes and return result
   3. If max is 4 and refresh occurred 7, you'll get data that is fresh to 4 hours ago
   4. configure as option when creating a view
4. Non-incremental MVs
   1. Wider support (having, unions, outer joins, analytic functions)
   2. created with `allow_non_incremental_definition` option
   3. Must have `max_staleness`
   4. Should have a refresh policy