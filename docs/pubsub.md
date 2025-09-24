# <img src="https://icon.icepanel.io/GCP/svg/PubSub.svg" style="width:64px"> pubsub
## Bigquery subscriptions
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

## Message storage policy
1. Controls where messages are stored and processed
2. Only addresses message content (not service metadata)
3. Can also control publish and subscribe operations in enfintrans is set
4. Topic stores in three zones; sync to two, best effort to 3rd

## Misc.
1. Message is comprised of
   1. data
   2. ordering key
   3. attributes
   4. id
   5. timestamp
6. Import topics include
   1. Kinesis
   2. Event hub
   3. Cloud Storage
   4. Amazon managed streaming for Kafka
   5. Confluent Cloud
7. Compression
   1. Messages can be Gzip compressed before publishing
8. Schemas
   1. Defined as Avro or Protobuf
   2. Can have a range of versions supported
   3. Can configure dead letter topics to receive rejected messages
9.  SMT
   1.  Example: Single message goes to topic, but some systems get redacted content
   2.  Javascript UDF that does the transformation in line

## Subscription options
1.  flow control: smooth out delivery flow to subs (in sub client)
2.  lease management
3.  ordering
    1.  messages must be published into a single region
    2.  subscribers can be in any region
    3.  ordering key must be set
    4.  message ordering is a subscription setting
    5.  streamingpull clicents us affinity (messages for a key go to one instance)
    6.  don't use ordering with Dataflow
4.  Exactly once delivery
    1.  redeliveries still happen; duplicates don't
    2.  requires subscribers to connect to service in same region
    3.  Significantly higher latencies
    4.  

## Retention/snapshot/seek
1.  Retention can be set at topic or subscription
2.  Snapshot is point-in-time view of message ack state in a sub, then accumulates messages published after creation
3. Seeking to a timestamp sets every message prior as acked and every after as unacked
   1. Can seek to future to purge all messages
   2. Can cause some issues because receipt time is not the same as event time, and there might be clock skew
4. Snapshots generally created when an important event or threshold occurs
   
##