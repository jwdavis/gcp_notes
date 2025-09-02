# AI/ML

## Links

### Ensemble models
- [Seeing contributing models](https://cloud.google.com/vertex-ai/docs/tabular-data/classification-regression/logging#final-payload)
- [Use in tabular automal](https://cloud.google.com/vertex-ai/docs/tabular-data/tabular-workflows/forecasting)
- [Use in forecasting](https://cloud.google.com/vertex-ai/docs/tabular-data/tabular-workflows/forecasting)

### Feature store
- [Notebooks for using Feature Store](https://cloud.google.com/vertex-ai/docs/featurestore/latest/notebooks)

### Prediction
- [Inference containers](https://cloud.google.com/vertex-ai/docs/predictions/pre-built-containers#optimized-tensorflow-runtime)
- [Custom containers](https://cloud.google.com/vertex-ai/docs/predictions/custom-container-requirements)
- [Custom inference routines](https://cloud.google.com/vertex-ai/docs/predictions/custom-prediction-routines)

### Pipelines
- [Samples](https://github.com/GoogleCloudPlatform/vertex-pipelines-end-to-end-samples)

### Distilling
- [Step by step paper](https://research.google/blog/distilling-step-by-step-outperforming-larger-language-models-with-less-training-data-and-smaller-model-sizes/)
- [Example in notebook](https://colab.research.google.com/github/GoogleCloudPlatform/vertex-ai-samples/blob/main/notebooks/official/generative_ai/distillation.ipynb)
- 

## Transfer learning

1. A model is trained on a broad, rich dataset and learns general representations
2. The learned weights are taken as the starting point for a new model training
3. Then a fine-tuning job is done with a specialized dataset
   1. Some or most layers may be frozen
   2. The model may be retrained entirely, but with previously learned weights to start

## AutoML Ensembles

1. For tabular, trains boosted trees, linear models, neural networks
2. Builds an ensemble of top candidates
3. Predictions from each top model are then fed to a meta-model, which knows how to combine to generate prediction (or a weighted average as a simple version of this)
4. Top models are those that score high, but also maximize complementarity
5. See links for details on ensembles

## Custom training ([slides 48-51](https://drive.google.com/file/d/1jud0JDlPONm7RU3l0k9Y0-Th65KrY1Mi/view))

1. Nice overview at [link](https://cloud.google.com/vertex-ai/docs/training/overview)
2. Code structure
   1. Script with prebuilt container
   2. Training app with prebuilt container (python source distro)
   3. Custom container
3. Prebuilt containers with single Python script
   1. Create a TrainingPipeline
   2. It automatically creates a source distribution
   3. Then the process follows the source distribution mechanism below
4. Prebuilt containers and source distributions
   1. Takes source distro, compresses it, uploads to GCS
   2. When container is launched, services does a pip install of your distro
   3. Executes the entry point `python -m trainer.task`
   4. Creates a CustomJob
5. Prebuilt containers with autopackaging
   1. Builds a custom container based on a prebuilt
   2. Takes your training application, builds a new Docker image, uploads to Artifact Registry
   3. Creates a CustomJob
6. Custom container
   1. Create your app with framework of choice
   2. Create container image with all required dependencies
   3. Load container image into Artifact registry
   4. Create a CustomJob
7. CustomJobs output a trained model, which is then upload to registry, etc.

### Vertex AI Feature Store
1. Exposes features for model training (current and historical) and for low-latency inference
2. Data is stored in BigQuery; Feature Store offers metadata layer
3. Online store is for prediction; can use BigTable or optimized

### Prediction
1. Predefined or custom serving containers running on Vertex AI Endpoints or Batch
   1. Can also do inference elsewhere, like Run or GKE
2. Predefined containers 
   1. Have an http server built in to handle requests
   2. Load model from GCS
   3. Request payload must follow specific guidelines
   4. Request response follows specific guidelines
3. You can use custom prediction routines to modify inference logic
   1. Useful for sending raw inputs and having inference server preprocess data

### Monitoring
1. Identify training-serving skew (features you see in production for inference are distributed differently than in the training dataset)
2. Inference drift (feature distribution changes over time, but you're not comparing to original training data)
3. V1 supports detection for categorical and numerical features
4. In GUI, configured when creating endpoint and deploying model
   1. In API, configured as a ModelDeploymentMonitoringJob

### Distillation
1. You have a big model that works, but may be slow and expensive (and big)
2. You want a smaller, faster, cheaper, less demanding model with most of the ability
3. Your big model becomes the teacher; you take a small off-the-shelf model as as the student
4. You train the small model on a new dataset so that it learns to make the same predictions and the create the same probability distributions (soft targets) as the big model
5. If you don't have the new dataset, the teacher model can create it
6. You use loss functions to measure how good the newly trained small model is at predicting the prediction made by the teacher model (uses ground truth loss and distillation loss with weights)
7. Example scenario: categorizing medical patient records
   1. Start with Med-PaLM as teacher
   2. Start with much simpler classification model using BERT arch as student
8. Example scenario: small fast model for describing accessability features in images
   1. Start with Gemini as teacher model
   2. Start with vision-only model like MobileNet as student
