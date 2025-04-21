<h1 align="center" style="margin-top: 0px;">relevance-classifier</h1>

## Description
This microservice is part of the [Feed.UVL Project](https://github.com/feeduvl) and is responsible for the **automatic classification of online feedback sentences** based on their relevance for requirements engineering.

Each sentence is classified as either **"Informative"** or **"Non-Informative"**, using a fine-tuned BERT model. The model builds on the pre-trained [BERT-base-uncased](https://huggingface.co/google-bert/bert-base-uncased) and has been fine-tuned specifically for this task using **Hugging Face Transformers**.

Five different runs are executed, where models are fine-tuned on different datasets and dataset combinations. The models and their evaluation results are stored in **MLflow**.

### Datasets
The following datasets are used for fine-tuning and evaluation:
- **P2-Golden dataset**: Contains 1,242 app review sentences from [^fn1].
- **Manual labeled Komoot dataset**: Consists of 2,199 app review sentences for the app "Komoot".

### Training Methods
Five different training methods are used:
1. **Training and testing** using only the P2-Golden dataset.
2. **Training and testing** using only the Komoot dataset.
3. **Training and testing** by combining both the P2-Golden and Komoot datasets.
4. **Training** with the P2-Golden dataset and **testing** with the Komoot dataset.
5. **Training** with the Komoot dataset and **testing** with the P2-Golden dataset.

### Evaluation
The trained models were evaluated using **5-fold cross-validation**. Metrics such as **precision**, **recall**, and **F1-score** were calculated for each dataset combination. Additionally, a **confusion matrix** for each model was calculated.
The evaluation results can be found in the `evaluation` package.

### API Methods
The microservice provides the following functionalities:
- **Service status**: Get the current status of the microservice.
- **Annotation creation**: Automatically generate annotations and create a new dataset containing only informative app review sentences.

### Technologies Used
- **Hugging Face Transformers**
- **Docker**
- **Flask**
- **MLflow**
- **Python 3.11+**

### Reproducibility Notice

This microservice is part of a larger, interconnected system and was developed for a real-world application in a multi-service architecture.

Due to data protection and infrastructure constraints:
- Datasets are not included
- Trained models in MLflow are not publicly accessible
- Other required microservices – such as services for data crawling, annotation initialization, and dataset persistence - are not included in this repository

You can still review:
- The architecture and design of this microservice
- The training pipeline and annotation/dataset creation pipeline
- The evaluation results
- The Docker setup and local configuration logic

## Requirements

### Runtime Dependencies
- Running **MLflow server**
- Existing model in MLflow
- Adjusted experiment parameters depending on your model choice in the notebook "ComponentRelevanceClassifierServiceSetup"

### Software
- >=Python 3.11

## Getting Started – as a Containerized Microservice

```sh
docker build -t <CONTAINER_NAME> -f "./Dockerfile" \
  --build-arg mlflow_tracking_username=XXXXXX \
  --build-arg mlflow_tracking_password=XXXXXX \
  --build-arg mlflow_tracking_uri=XXXXXX .

docker run -p 9698:9698 --name <CONTAINER_NAME>
```
→ Replace `XXXXXX` with your MLflow credentials

## Getting Started for Local Testing

### 1. Clone the repository

```sh
git clone https://github.com/dkeyGit/relevance-classifier.git
cd relevance-classifier
```

### 2. (Optional) Create a Virtual Environment

#### Option A: Using `venv` <br>
...

#### Option B: Using `conda`

**1. Install Miniconda (if not already installed):** <br>
[Miniconda Installation Guide](https://docs.anaconda.com/free/miniconda/miniconda-install)

**2. Create and activate the environment:**
```sh
conda create -n rc python=3.11
conda activate rc
```

**3. Configure local MLflow environment variables:**

```sh
cd $CONDA_PREFIX
mkdir -p ./etc/conda/activate.d
mkdir -p ./etc/conda/deactivate.d
echo \#\!/bin/bash >> ./etc/conda/activate.d/env_vars.sh
echo "export MLFLOW_TRACKING_USERNAME=" >> ./etc/conda/activate.d/env_vars.sh
echo "export MLFLOW_TRACKING_PASSWORD=" >> ./etc/conda/activate.d/env_vars.sh
echo "export MLFLOW_TRACKING_URI='http://127.0.0.1:5000'" >> ./etc/conda/activate.d/env_vars.sh

echo \#\!/bin/bash >> ./etc/conda/deactivate.d/env_vars.sh
echo "unset MLFLOW_TRACKING_USERNAME" >> ./etc/conda/deactivate.d/env_vars.sh
echo "unset MLFLOW_TRACKING_PASSWORD" >> ./etc/conda/deactivate.d/env_vars.sh
echo "unset MLFLOW_TRACKING_URI" >> ./etc/conda/deactivate.d/env_vars.sh

conda deactivate
conda activate rc
```

### 3. Start the MLflow Server (locally)
```sh
cd <THE_BASE_DIRECTORY_FOR_MLFLOW>
mlflow server
```

### 4. Set Up the Development Environment

```sh
cd <THE_BASE_DIRECTORY_FOR_THE_relevance-classifier_SERVICE>
pip install -e .
```

### 5. Train or Load the relevance-classifier Model

**If there is no existing relevance-classifier model on MLflow, train the models:** <br>
Start the notebook "ComponentRelevanceClassifierServiceTraining"

**Load the existing model from MLflow, you want to use for the relevance classification:** <br>
Start the notebook "ComponentRelevanceClassifierServiceSetup"
→ in this notebook, you have to adjust the experiment parameters depending on your model choice

### 6. Start the relevance-classifier service
```sh
./start.sh
```

## References
[^fn1]:van Vliet, M., Groen, E., Dalpiaz, F., Brinkkemper, S.: Crowd-annotation results: Identifying and classifying user requirements in online feedback (2020), https://doi.org/10.5281/zenodo.3754721, Zenodo
