## <img src="https://icon.icepanel.io/GCP/svg/Identity-And-Access-Management.svg" style="width:64px"> Security stuff
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