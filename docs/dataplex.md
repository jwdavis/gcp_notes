# Dataplex

## Links

### Ensemble models
- [Product page](https://cloud.google.com/dataplex?hl=en#intelligent-data-to-ai-governance)

### Client libraries
- [Dataplex client](https://cloud.google.com/python/docs/reference/dataplex/latest) (Python)
- [Data lineage client ](https://cloud.google.com/python/docs/reference/lineage/latest)(Python)
- [Data lineage overview](https://cloud.google.com/dataplex/docs/about-data-lineage)

## General
1. Now presented as Dataplex Universal Catalog
2. Main use cases include...
   1. Discovery (via catalog search)
   2. Data mesh
   3. Governance (profiling, quality, lineage, security)

## Data lineage
1. Tracks where data come from, where it goes, and transformation applied
2. Various data manipulation actions make calls to the lineage API
   1. Processes describe data transformations
      1. BQ Copy, load, query
      2. Data Fusion, Composer, Dataflow, Dataproc, Vertex AI
   2. Run describe executions of a process
   3. Events describe an operation where data flows from source to target
3. Graphs in the UI are a bit wonky
4. Significant delay (10+ minutes) in lineage data showing up
5. Cloud Composer uses the apache-airflow-providers-openlineage package to generate the lineage events that are sent to the Data Lineage API.
   1. Requires operator support (can be checked [here](https://airflow.apache.org/docs/apache-airflow-providers-openlineage/stable/supported_classes.html))
