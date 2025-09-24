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

### Lock and retentions
1. Bucket retention policies
   1. Retention policy says objects can only be deleted on their age is greater than retention period
   2. Retroactively applies to existing objects and gets added to new ones
2. Bucket retention lock
   1. Once you lock, you cannot remove or reduce the period
   2. You cannot delete bucket until every object has met retention period
3. Object retention
   1. Applies to individual objects
   2. Retain until time specifies a date prior to which, the object can't be deleted
   3. Retention mode controls what changes you can make to the retention configurations; unloded of locked