# Chicken-Disease-Classification--Project


## Workflows

1. Update config.yaml
2. Update secrets.yaml [Optional]
3. Update params.yaml
4. Update the entity (entity- return type of a function)
5. Update the configuration manager in src config
6. Update the components
7. Update the pipeline 
8. Update the main.py
9. Update the dvc.yaml

Here’s a detailed explanation of each component and process involved in your project, breaking down the key stages and how everything connects:

### 1. **Objective of the Project**
The goal of this project is to build a **CNN-based image classifier** that identifies whether a chicken is healthy or has Coccidiosis, a parasitic disease. The project pipeline is designed for **automated data ingestion, model training, evaluation**, and finally **continuous integration, delivery, and deployment** (CI/CD) using **GitHub Actions** with deployment to AWS EC2 and ECR.

---

### 2. **Project Structure**
The project has several key stages, from the initial data ingestion to final deployment. Here's an overview of each part:

---

### **a. CNN Classifier Code**

#### **`predict.py`** 
This Python script is responsible for making predictions based on a saved model:
- **Loading the Model**: The model saved in the `artifacts/training/model.keras` is loaded using Keras.
- **Image Preprocessing**: The input image is preprocessed by resizing it to (224, 224), converting it into an array, and then expanding its dimensions to match the model's input format.
- **Prediction**: The model predicts the class of the image, returning a label based on the model’s output.
- **Result Mapping**: The predicted output (whether the image is of a healthy chicken or has Coccidiosis) is mapped to its corresponding label.

---

#### **`stage_01_data_ingestion.py`**
- **Goal**: This stage downloads and extracts the dataset.
- **Steps**: 
  - Configuration is loaded to obtain the file URL for the dataset.
  - The dataset is downloaded and extracted to `artifacts/data_ingestion/Chicken-fecal-images`.

---

#### **`stage_02_prepare_base_model.py`**
- **Goal**: Prepare the base CNN model (e.g., VGG16 or another pre-trained model).
- **Steps**:
  - **Get Base Model**: Loads a pre-trained model (e.g., VGG16) with or without the top layers (depending on configuration).
  - **Update Base Model**: The top layers are modified to suit the number of classes (in this case, binary classification), and it adjusts the learning rate and other hyperparameters.

---

#### **`stage_03_training.py`**
- **Goal**: Train the model with the prepared dataset.
- **Steps**:
  - **Callbacks**: Sets up callbacks for logging (TensorBoard) and model checkpointing.
  - **Model Training**: Uses data augmentation, defines the train/validation generators, and trains the model using the loaded dataset. The model is then saved as `model.keras`.

---

#### **`stage_04_evaluation.py`**
- **Goal**: Evaluate the trained model.
- **Steps**:
  - **Evaluation**: Loads the validation set and evaluates the model’s performance on this dataset.
  - **Save Score**: Saves the evaluation metrics to a JSON file (`scores.json`), storing important metrics like accuracy or F1 score.

---

### 3. **DVC (Data Version Control) Integration**
DVC is used to manage the stages and dependencies of the machine learning workflow. It tracks data, model files, and pipelines, ensuring reproducibility.

#### **`dvc.yaml`**
Defines the stages of the ML pipeline:
- **`data_ingestion`**: Downloads and extracts the dataset.
- **`prepare_base_model`**: Prepares the CNN model.
- **`training`**: Trains the model and saves it.
- **`evaluation`**: Evaluates the model and stores the results in a JSON file.
Each stage is connected by their dependencies (e.g., the model in the `training` stage depends on the data from `data_ingestion` and the base model from `prepare_base_model`).

---

### 4. **CI/CD with GitHub Actions**

#### **`.github/workflows/main.yaml`**
This file defines the CI/CD pipeline, automating the integration, delivery, and deployment process using GitHub Actions.

##### **Job 1: Continuous Integration (CI)**
- **Linting and Unit Testing**:
  - First, it checks out the code and performs **linting** to ensure code quality.
  - It also runs **unit tests** (in this case, placeholder commands) to verify code correctness.

##### **Job 2: Build and Push Docker Image to ECR**
- **Install Utilities**: Installs necessary utilities such as `jq` (for JSON parsing) and `unzip`.
- **AWS Authentication**: Uses secrets for AWS credentials (stored securely in GitHub Secrets) to authenticate with AWS.
- **Login to ECR**: Logs into **Amazon Elastic Container Registry (ECR)**.
- **Build and Push Docker Image**: 
  - Builds a Docker image of the application using the Dockerfile.
  - Tags and pushes the image to ECR for deployment.

##### **Job 3: Continuous Deployment (CD)**
- **Pull Latest Image**: Pulls the latest image from ECR.
- **Stop Existing Container**: Stops and removes any running container (if applicable).
- **Run New Docker Container**: Deploys the latest image on an **EC2 instance**, running it as a Docker container on port 8080, serving users.

---

### 5. **AWS Services**

#### **Amazon Elastic Container Registry (ECR)**:
- ECR stores Docker images, providing a place to push and pull images used in deployments.

#### **Amazon EC2**:
- EC2 instances are used to deploy the application. The Docker container runs on an EC2 instance, serving the prediction model API.

---

### 6. **GitHub Secrets**
GitHub Secrets are used to store sensitive information securely. In your workflow, the following secrets are used:
- **AWS_ACCESS_KEY_ID**: AWS Access Key for authenticating with AWS services.
- **AWS_SECRET_ACCESS_KEY**: AWS Secret Key for authentication.
- **ECR_REPOSITORY_NAME**: Name of the ECR repository to which Docker images are pushed.
- **AWS_REGION**: The AWS region where the EC2 instance and ECR are located.

---

### 7. **Summary**

- **CNN Classifier Pipeline**: The ML model pipeline is built using four stages: data ingestion, model preparation, training, and evaluation. These stages are managed using DVC to track data and models efficiently.
- **CI/CD Pipeline**: The CI/CD workflow automates the process of integrating, delivering, and deploying the application using **GitHub Actions**. Docker images are built and stored in ECR, while deployment is handled using EC2.
- **Automation**: With this setup, every code change automatically triggers tests, builds the Docker image, and deploys the latest version without manual intervention.

This project shows a complete integration of **machine learning model development** with modern **DevOps practices** for deployment on the cloud, ensuring efficiency and scalability.