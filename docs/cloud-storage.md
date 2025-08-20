# Cloud Storage

## Links
- [Hierarchical namespace](https://cloud.google.com/storage/docs/hns-overview)
- [Availability and replication](https://cloud.google.com/storage/docs/availability-durability)
- [Caching](https://cloud.google.com/storage/docs/caching)
- [Batching](https://cloud.google.com/storage/docs/batch)

## Hierarchical Namespaces

!!! note "Performance Boost"
    Big performance improvements for certain workloads; Hadoop and Spark especially

## Replication

1. **99.9% SLA** on write replication within an hour
2. **Turbo replication** is 100% in 15 minutes

## Python Client

1. **Mostly synchronous**
2. **Transfer manager is async**
   - Current and future method is using processes, so can do parallel writes but is copying files from disk, not in-memory