#  <img src="https://icon.icepanel.io/GCP/svg/Cloud-Composer.svg" style="width:64px"> Composer
## Links
* [Architecture](https://cloud.google.com/composer/docs/composer-versioning-overview)
* [Write Airflow DAGs](https://cloud.google.com/composer/docs/composer-3/write-dags)
* [Test DAGs](https://cloud.google.com/composer/docs/composer-3/test-dags)
* [Local Composer Environment](https://cloud.google.com/composer/docs/composer-2/run-local-airflow-environments)
* [Google providers](https://airflow.apache.org/docs/apache-airflow-providers-google/stable/index.html)

## General
1. Airflow only supports up to Python 3.11
2. Composer evironment setup time is like 22+ minutes
3. Version 2 builds a cluster for you; version 3 runs your stuff as pods on a google-managed cluster ([link](https://cloud.google.com/composer/docs/composer-versioning-overview))

## IAM Stuff
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