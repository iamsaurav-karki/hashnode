---
title: "from-devops-to-mlops-building-production-ready-ml-pipelines"
seoTitle: "from-devops-to-mlops-building-production-ready-ml-pipelines"
datePublished: Thu Feb 12 2026 16:59:40 GMT+0000 (Coordinated Universal Time)
cuid: cmljpev7p000w02jxa38f6ilg
slug: from-devops-to-mlops-building-production-ready-ml-pipelines
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1770915301198/e7e3f042-c093-4808-84cc-4bc771ac1e86.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1770915549651/971201de-bb5a-45f4-b319-fb1dbacb944d.png
tags: docker, kubernetes, devops, prometheus, grafana, fastapi, mlops, keda, mlflow

---

## **Introduction**

As a DevOps engineer, I’ve practiced developing CI/CD pipelines, managing Kubernetes clusters, and ensuring reliable deployments. But when machine learning entered the picture, I realized something: the traditional DevOps playbook wasn't enough. ML models aren't just code—they're experiments, data, and continuously evolving artifacts that need a whole new level of operational maturity.

This led me to explore **MLOps**—the intersection of Machine Learning, DevOps, and Data Engineering. I recently completed the "**Ultimate DevOps to MLOps Bootcamp**" course, where I built an end-to-end house price prediction system using industry-standard tools. This article shows my journey and serves as a comprehensive guide for fellow DevOps engineers looking to make the same transition.

## **Why MLOps Matters for DevOps Engineers**

Before diving into the technical details, let me address the elephant in the room: why should DevOps engineers care about MLOps?

**The answer is simple**: Machine learning is rapidly becoming a core component of modern applications. Companies aren't just shipping code anymore—they're shipping intelligent systems that learn and adapt. As DevOps engineers, we're uniquely positioned to bridge the gap between data science teams and production environments.

Traditional software development follows a relatively linear path: code → build → test → deploy. ML systems, however, introduce new complexities:

* **Data dependencies**: Models are only as good as their training data
    
* **Experiment tracking**: Multiple model versions need systematic management
    
* **Model drift**: Performance degrades over time as real-world data changes
    
* **Resource-intensive training**: GPU requirements and distributed computing challenges
    
* **A/B testing**: Comparing model performance in production
    

Without proper MLOps practices, ML projects often fail to make it to production or become maintenance nightmares when they do.

## **Project Overview: House Price Predictor**

For this learning journey, I built a complete house price prediction system—not just a model, but a production-ready pipeline with proper versioning, monitoring, and deployment automation.

```plaintext
**house-price-predictor-ml/**

├── .github/workflows
├── configs/
├── data/
├── deployment/
│   ├── kubernetes/
│   ├── mlflow/
│   ├── monitoring/
├── models/trained
├── notebooks/
├── src/
├── streamlit_app/
├── .dockerignore
├── .gitignore
├── Dockerfile
├── LICENSE
├── README.md
├── docker-compose.yml
── requirements.txt
```

![Project Structure](https://i.ibb.co/fzbnLTnt/project-structrure.png align="left")

### **Tech Stack**

Here's what I used to build this end-to-end system:

* **Experiment Tracking**: MLflow
    
* **Data Processing**: Pandas, Scikit-learn
    
* **Model Serving**: FastAPI
    
* **Web Interface**: Streamlit
    
* **Containerization**: Docker
    
* **Orchestration**: Kubernetes (KIND for local development)
    
* **CI/CD**: GitHub Actions
    
* **GitOps**: ArgoCD
    
* **Monitoring**: Prometheus & Grafana
    

## **Phase 1: Setting Up the MLOps Foundation**

### **Environment Setup**

Coming from a DevOps background, I'm particular about reproducible environments. I used **UV**, a modern Python package manager, to ensure consistent dependency management:

\# Create isolated virtual environment

```plaintext
uv venv --python python3.11
source .venv/bin/activate
```

\# Install dependencies

`uv pip install -r requirements.txt`

**Pro tip**: Using UV over pip gave me significantly faster dependency resolution and better lock file management—crucial when dealing with the complex dependency trees of ML libraries.

### **MLflow: The Experiment Tracking Backbone**

One of the biggest challenges in ML is keeping track of experiments. Data scientists might train dozens of models with different hyperparameters. Without proper tracking, it becomes impossible to reproduce results or understand what worked.

I deployed MLflow using Docker Compose:

\# deployment/mlflow/mlflow-docker-compose.yml

```plaintext
version: '3'
services:
  mlflow:
    image: ghcr.io/mlflow/mlflow:latest
    ports:
      - "5555:5000"
    command: mlflow server --host 0.0.0.0 --backend-store-uri sqlite:///mlflow.db --default-artifact-root /tmp/mlruns
    container_name: mlflow-tracking-server
```

Starting the service:

```plaintext
cd deployment/mlflow
docker compose -f docker-compose.yml up -d
```

The MLflow UI at [**http://localhost:5555**](http://localhost:5555/) became my command center for comparing model performance, tracking metrics, and managing model versions.

**MlFlow Dashboard showing Exterimental Trackings**

![MlFlow Experiment Tracking](https://i.ibb.co/jkkhqg3n/mlflow-experiment-tracking.png align="left")

## **Phase 2: Building the ML Pipeline**

### **Data Processing & Feature Engineering**

The foundation of any ML system is clean, well-prepared data. I created modular scripts that separate concerns:

\# Step 1: Clean and process raw data

```plaintext
python src/data/run_processing.py   --input data/raw/house_data.csv   --output data/processed/cleaned_house_data.csv
```

\# Step 2: Feature engineering

```plaintext
python src/features/engineer.py   --input data/processed/cleaned_house_data.csv   --output data/processed/featured_house_data.csv   --preprocessor models/trained/preprocessor.pkl
```

**Key insight**: I saved the preprocessor as a pickle file so that the same data transformations used during training are also used during prediction. This helps avoid differences between training and production.

### **Model Training with Experiment Tracking**

This is where MLOps truly shines. Instead of training models in isolation, everything gets logged to MLflow:

```plaintext

python src/models/train_model.py   --config configs/model_config.yaml   --data data/processed/featured_house_data.csv   --models-dir models   --mlflow-tracking-uri http://localhost:5555
```

The model\_config.yaml allowed me to version control my model configurations:

Model:

```plaintext
 best\_model: GradientBoosting
  feature\_sets:
    rfe:
    - '0'
    - '1'
    - '2'
    - '3'
    - '4'
    - '5'
    - '9'
    - '10'
    - '13'
    - '14'
  mae: 10357.238519517934
  name: house_price_model
  parameters:
    alpha: 0.9
    ccp_alpha: 0.0
    criterion: friedman_mse
    init: null
    learning_rate: 0.1
    loss: squared_error
    max_depth: 3
    max_features: null
    max_leaf_nodes: null
    min_impurity_decrease: 0.0
    min_samples_leaf: 1
    min_samples_split: 2
    min_weight_fraction_leaf: 0.0
    n_estimators: 100
    n_iter_no_change: null
    random_state: null
    subsample: 1.0
    tol: 0.0001
    validation_fraction: 0.1
    verbose: 0
    warm_start: false
  r2_score: 0.9935436022386502
  target_variable: price
```

Every training run automatically logged:

* Model parameters
    
* Performance metrics (RMSE, MAE, R²)
    
* Training artifacts (model file, preprocessor)
    
* Code version (Git commit hash)
    

## **Phase 3: Model Serving with FastAPI**

A model that can't be queried is useless. I built a REST API using FastAPI to serve predictions:

\# src/api/main.py (simplified example)

```plaintext

# Health check endpoint 
@app.get("/health", response_model=dict)
async def health_check():
    return {"status": "healthy", "model_loaded": True}

# Prediction endpoint
@app.post("/predict", response_model=PredictionResponse)
async def predict(request: HousePredictionRequest):
    return predict_price(request)

# Batch prediction endpoint
@app.post("/batch-predict", response_model=list)
async def batch_predict_endpoint(requests: list[HousePredictionRequest]):
    return batch_predict(requests)
```

### **Containerizing the Application**

I created a  Dockerfile for optimal image size:

```plaintext
FROM python:3.11-slim
WORKDIR /app
# Install dependencies
COPY src/api .
RUN pip install --no-cache-dir -r requirements.txt

# Copy the trained model files into the container
COPY models/trained/*.pkl /app/models/trained/

# Expose the port for the API and Prometheus metrics
EXPOSE 8000 8001

# Command to run the API, using uvicorn as the ASGI server
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]

```

Testing the API:

```plaintext
curl -X POST "http://localhost:8000/predict" \

-H "Content-Type: application/json" \

-d '{

  "sqft": 1500,

  "bedrooms": 3,

  "bathrooms": 2,

  "location": "suburban",

  "year\_built": 2000,

  "condition": "fair"

}'
```

\# Response:

\# {"predicted\_price": 285000.50}

## **Phase 4: User Interface with Streamlit**

While APIs are great for programmatic access, stakeholders often want a visual interface. Streamlit made this trivial:

So , I used streamlit to create ui for the house price predictor 

**Containerized the Streamlit UI application:**

```plaintext

FROM python:3.9-slim

WORKDIR /app

COPY app.py requirements.txt ./

RUN pip install --no-cache-dir -r requirements.txt

EXPOSE 8501

CMD \["streamlit", "run", "app.py", "--server.port=8501", "--server.address=0.0.0.0"\]
```

### **Docker Compose for Local Development**

I orchestrated both services using Docker Compose:

\# docker-compose.yaml

```plaintext
version: "3.8"

networks:
  app_net:
    driver: bridge

services:
  flask_api:
    image: sauravkarki/housepricepredictor:rest-v1
    container_name: flask_api
    expose:
      - "8000"          # only visible inside Docker network
    networks:
      - app_net
    restart: unless-stopped


  streamlit:
    image: sauravkarki/housepricepredictor:streamlit-v4
    container_name: streamlit_app
    ports:
      - "8501:8501"     # public entry point
    environment:
      API_URL: http://flask_api:8000
    depends_on:
      - flask_api
    networks:
      - app_net
    restart: unless-stopped
```

Starting everything:

`docker compose up -d`

## **Phase 5: CI/CD with GitHub Actions**

Manual deployments are error-prone and don't scale. I automated the entire pipeline using GitHub Actions:

\# .github/workflows/streamlit-ci.yml

```plaintext
name: Streamlit CI

on:
  push:
    paths:
      - 'streamlit_app/**'
  workflow_dispatch:

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to DockerHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: docker.io
          username: ${{ vars.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: ./streamlit_app
          push: true
          tags: docker.io/${{ vars.DOCKERHUB_USERNAME }}/streamlit:latest
          platforms: linux/amd64,linux/arm64
```

\# .github/workflows/model-ci.yml

```plaintext
name: MLOps Pipeline

on:
  # push:
  #   branches: [ main ]
  #   tags: [ 'v*.*.*' ]
  # pull_request:
  #   branches: [ main ]
  workflow_dispatch:
    inputs:
      run_all:
        description: 'Run all jobs of the pipeline'
        required: false
        default: 'true'
      run_data_processing:
        description: 'Run data processing job'
        required: false
        default: 'true'
      run_feature_engineering:
        description: 'Run feature engineering job'
        required: false
        default: 'true'
      run_model_training:
        description: 'Run model training job'
        required: false
        default: 'true'
      run_build_and_publish:
        description: 'Run build and publish job'
        required: false
        default: 'false'
  release:
    types: [ created ]
    branches: [ main ]
    tags: [ 'v*.*.*' ]
    
jobs:
  data-processing:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v2
      
    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: '3.11.14'
        
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
        
    - name: Process data
      run: |
        python src/data/run_processing.py --input data/raw/house_data.csv --output data/processed/cleaned_house_data.csv 
        
    - name: Engineer features
      run: |
        python src/features/engineer.py --input data/processed/cleaned_house_data.csv --output data/processed/featured_house_data.csv --preprocessor models/trained/preprocessor.pkl
        
    - name: Upload processed data
      uses: actions/upload-artifact@v4
      with:
        name: processed-data
        path: data/processed/featured_house_data.csv
        
    - name: Upload preprocessor
      uses: actions/upload-artifact@v4
      with:
        name: preprocessor
        path: models/trained/preprocessor.pkl
        
  model-training:
    needs: data-processing
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v2
      
    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: '3.11.9'
        
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
        
    - name: Download processed data
      uses: actions/download-artifact@v4
      with:
        name: processed-data
        path: data/processed/
        
    - name: Set up MLflow
      run: |
        docker pull ghcr.io/mlflow/mlflow:latest
        docker run -d -p 5000:5000 --name mlflow-server ghcr.io/mlflow/mlflow:latest mlflow server --host 0.0.0.0 --backend-store-uri sqlite:///mlflow.db
        
    - name: Wait for MLflow to start
      run: |
        for i in {1..10}; do
          curl -f http://localhost:5000/health || sleep 5;
        done
        
    - name: Train model
      run: |
        mkdir -p models
        python src/models/train_model.py --config configs/model_config.yaml --data data/processed/featured_house_data.csv --models-dir models --mlflow-tracking-uri http://localhost:5000
        
    - name: Upload trained model
      uses: actions/upload-artifact@v4
      with:
        name: trained-model
        path: models/
        
    - name: Clean up MLflow
      run: |
        docker stop mlflow-server || true
        docker rm mlflow-server || true
        
  build-and-publish:
    needs: model-training
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v2
      
    - name: Download trained model
      uses: actions/download-artifact@v4
      with:
        name: trained-model
        path: models/
        
    - name: Download preprocessor
      uses: actions/download-artifact@v4
      with:
        name: preprocessor
        path: models/trained/
        
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3
      

    - name: Log in to DockerHub Container Registry
      uses: docker/login-action@v3
      with:
        registry: docker.io
        username: ${{ vars.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}


    - name: Build and push Docker image
      uses: docker/build-push-action@v5
      with:
          context: .
          file: ./Dockerfile
          push: true
          tags: docker.io/${{ vars.DOCKERHUB_USERNAME }}/house-price-model:latest
          platforms: linux/amd64,linux/arm64
```

![mlops pipeline](https://i.ibb.co/Ng158PZ5/fast-api-pipeline.png align="left")

## **Phase 6: Kubernetes Deployment with KIND**

For local Kubernetes testing, I used KIND (Kubernetes IN Docker):

\# Create cluster

```plaintext
kind create cluster --name mlops-cluster
```

\# Verify cluster

```plaintext
kubectl cluster-info --context kind-mlops-cluster
```

### **Kubernetes Manifests**

I created deployment manifests for both services: Also Setup KEDA as well as HPA, VPA for proper scaling of models.

Deploying to Kubernetes:

```plaintext
Kubectl apply -f <manifests>
```

Also Setup Event Driven Autoscaling with KEDA. Here is the yaml config for scaled objects.

```plaintext
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: model-scaler
  namespace: house-price-predictor
spec:
  scaleTargetRef:
    name: house-price-model
  minReplicaCount: 1
  maxReplicaCount: 5
  pollingInterval: 30
  cooldownPeriod: 300
  triggers:
    # 1️⃣ Latency-based scaling (P95)
    - type: prometheus
      metadata:
        serverAddress: http://prom-kube-prometheus-stack-prometheus.monitoring.svc:9090
        metricName: fastapi_latency_p95
        query: |
          histogram_quantile(
            0.95,
            sum(
              rate(http_request_duration_seconds_bucket{app="model-svc"}[1m])
            ) by (le)
          )
        threshold: "0.5"

    # 2️⃣ RPS-based scaling
    - type: prometheus
      metadata:
        serverAddress: http://prom-kube-prometheus-stack-prometheus.monitoring.svc:9090
        metricName: fastapi_rps
        query: |
          sum(rate(http_requests_total[1m]))
        threshold: "1000"

    # 3️⃣ Custom metric-based scaling (e.g., CPU usage)
    - type: cpu
      metricType: Utilization
      metadata:
        value: "80"
```

![kubernetes resources](https://i.ibb.co/0y9PtrFL/k8s-resources.png align="left")

## **Phase 7: GitOps with ArgoCD**

ArgoCD brought GitOps principles to my ML deployments. I installed ArgoCD in the cluster:

Install ArgoCD

```plaintext
kubectl create namespace argocd

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Reset admin password to password

```plaintext
# bcrypt(password)=$2a$10$rRyBsGSHK6.uc8fntPwVIuLVHgsAhAX7TcdrqW/RADU0uh7CaChLa
kubectl -n argocd patch secret argocd-secret \
  -p '{"stringData": {
    "admin.password": "$2a$10$rRyBsGSHK6.uc8fntPwVIuLVHgsAhAX7TcdrqW/RADU0uh7CaChLa",
    "admin.passwordMtime": "'$(date +%FT%T%Z)'"
  }}'
```

```plaintext
kubectl get all -n argocd
kubectl patch svc argocd-server -n argocd --patch \
  '{"spec": { "type": "NodePort", "ports": [ { "nodePort": 32100, "port": 443, "protocol": "TCP", "targetPort": 8080 } ] } }'
```

Created an ArgoCD Application:

\# argocd/application.yaml

```plaintext

apiVersion: argoproj.io/v1alpha1

kind: Application

metadata:

  name: house-price-predictor

  namespace: argocd

spec:

  project: default

  source:

    repoURL: https://github.com/iamsaurav-karki/house-price-predictor-ml

    targetRevision: main

    path: deployment/kubernetes

  destination:

    server: https://kubernetes.default.svc

    namespace: default

  syncPolicy:

    automated:

      prune: true

      selfHeal: true

    syncOptions:

    - CreateNamespace=true
```

![argocd](https://i.ibb.co/zhdnBCbv/argocd.png align="left")

Now, any changes in Git automatically trigger a deployment. This is the essence of GitOps—Git as the single source of truth.

![model predictions](https://i.ibb.co/Y4Dk41LZ/prediction-ui.png align="left")

![model predictions](https://i.ibb.co/d0tmBWnX/model-serving.png align="left")

## **Phase 8: Monitoring with Prometheus & Grafana**

A production ML system needs observability. I deployed Prometheus and Grafana to monitor:

* **API metrics**: Request rate, latency, error rate
    
* **Model metrics**: Prediction distribution, inference time
    
* **Resource metrics**: CPU, memory, pod health
    

### **Instrumenting FastAPI**

I added Prometheus metrics to the API:

```plaintext

from prometheus_fastapi_instrumentator import Instrumentator # For monitoring and metrics collection
from prometheus_client import start_http_server
import threading

# Initialize nad instrument prometheus metrics
Instrumentator().instrument(app).expose(app)  # Set up Prometheus metrics collection , expose metrics at /metrics endpoint

# Start Prometheus metrics server in a separate thread
def start_metrics_server():
    start_http_server(8001)  # Prometheus metrics will be available at http://localhost:8001/metrics

threading.Thread(target=start_metrics_server, daemon=True).start()

```

This will expose the model metrics in /metrics endpoints.

For Prometheus to scrap this metrics i have created a servicemonitor as:

```plaintext
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: house-price-fast-api-scrap
  namespace: house-price-predictor
  labels:
    release: prom
spec:
  selector:
    matchLabels:
      app: model-svc
  namespaceSelector:
    matchNames:
      - house-price-predictor
  endpoints:
    - port: metrics        # ✅ Use the port name from Service
      path: /
      interval: 30s
      scrapeTimeout: 10s
```

Dashboard:

![mlmodel monitoring dashboard](https://i.ibb.co/cKC5D7YS/grafana2.png align="left")

![mlmodel monitoring dashboard](https://i.ibb.co/vxDNq9r1/grafana3.png align="left")

## **Key Learnings & Best Practices**

After completing this project, here are the critical lessons I learned:

### **1\. Treat ML Models as First-Class Artifacts**

Just like you version code, version your:

\- Training data snapshots

\- Model binaries

\- Preprocessing pipelines

\- Configuration files

MLflow made this seamless with its model registry.

### **2\. Separation of Concerns Matters**

Keep these concerns separate:

\- **Data Engineering**: Data collection, cleaning, validation

\- **ML Engineering**: Feature engineering, model training, evaluation

\- **DevOps/MLOps**: Deployment, monitoring, infrastructure

Each discipline has its own tools and best practices.

### **3\. Automate Everything**

From data validation to model deployment, automation is non-negotiable. Manual steps lead to errors and don't scale.

### **4\. Monitor Beyond Infrastructure**

Traditional DevOps monitors CPU, memory, and request rates. MLOps adds:

\- Model performance metrics (accuracy, precision, recall)

\- Data drift (input distribution changes)

\- Prediction drift (output distribution changes)

\- Feature importance shifts

## **The DevOps-to-MLOps Transition**

As a DevOps engineer, I found the transition to MLOps natural yet challenging. Here's how my existing skills translated:

```plaintext
| DevOps Skill | MLOps Application |

|--------------|-------------------|

| CI/CD Pipelines | ML training pipelines, automated retraining |

| Containerization | Model packaging, reproducible environments |

| Kubernetes | Scalable inference, distributed training |

| Monitoring | Model performance tracking, drift detection |

| IaC (Infrastructure as Code) | DaC (Data as Code), model versioning |

| Git workflows | Experiment tracking, model governance |
```

The main paradigm shift: **ML systems are non-deterministic**. Unlike traditional software where the same input always produces the same output, ML models evolve with data and can degrade over time.

## **Conclusion**

Transitioning from DevOps to MLOps isn't about learning entirely new skills—it's about applying DevOps principles to the unique challenges of machine learning systems. The tools might be different (MLflow instead of Jenkins, model registries instead of artifact repositories), but the core philosophy remains: **automate, monitor, and iterate**.

The future of software is intelligent systems, and MLOps engineers who can bridge data science and production are in high demand. This project taught me entire ML systems lifecycle as well as the role of data engineer, data scientists, ml engineer and mlops engineer in ML systems and following the right tools and practice makes productionizing ML models easy.