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
---
