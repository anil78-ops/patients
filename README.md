# 🏥 Patient Service

A lightweight **Node.js microservice** for managing patient information.

This service provides REST APIs to create and retrieve patient records and is designed to run inside **Docker containers** and deploy automatically to **AWS EKS using GitHub Actions CI/CD**.

---

# 🚀 Tech Stack

| Technology        | Purpose                                      |
| ----------------- | -------------------------------------------- |
| 🟢 Node.js        | Backend runtime                              |
| ⚡ Express.js      | REST API framework                           |
| 🐳 Docker         | Containerization                             |
| ☁️ Amazon ECR     | Container registry                           |
| ☸️ Amazon EKS     | Kubernetes cluster                           |
| 🔁 GitHub Actions | CI/CD automation                             |
| 🔐 AWS OIDC       | Secure authentication between GitHub and AWS |

---

# 📂 Project Structure

```
patients/
│
├── .github/
│   └── workflows/
│       └── patients-ci-cd.yaml        # CI/CD pipeline
│
├── manifests/
│   └── patients-dev.yaml              # Kubernetes deployment manifest
│
├── src/
│   └── index.js                       # Express application
│
├── Dockerfile                         # Docker build instructions
├── package.json
├── package-lock.json
├── .gitignore
└── README.md
```

---

# 🌐 API Endpoints

## 🩺 Health Check

```
GET /health
```

Response

```json
{
  "status": "OK",
  "service": "Patient Service"
}
```

---

# 📋 Get All Patients

```
GET /patients
```

Response

```json
{
  "message": "Patients retrieved successfully",
  "count": 2,
  "patients": []
}
```

---

# 🔍 Get Patient by ID

```
GET /patients/:id
```

Example

```
GET /patients/1
```

Response

```json
{
  "message": "Patient found",
  "patient": {
    "id": "1",
    "name": "John Doe",
    "age": 30,
    "condition": "Healthy"
  }
}
```

---

# ➕ Create Patient

```
POST /patients
```

Request Body

```json
{
  "name": "John Doe",
  "age": 30,
  "condition": "Healthy"
}
```

Response

```json
{
  "message": "Patient added successfully",
  "patient": {}
}
```

---

# 🖥️ Running Locally

### Install dependencies

```bash
npm install
```

### Start the application

```bash
node src/index.js
```

Service runs at

```
http://localhost:3000
```

---

# 🐳 Docker

### Build Docker Image

```bash
docker build -t patients-service .
```

### Run Container

```bash
docker run -p 3000:3000 patients-service
```

---

# ☸️ Kubernetes Deployment

The Kubernetes manifest is located at:

```
manifests/patients-dev.yaml
```

Deploy manually using:

```bash
kubectl apply -f manifests/patients-dev.yaml
```

Verify deployment:

```bash
kubectl get pods
kubectl get services
```

---

# 🔁 CI/CD Pipeline (GitHub Actions)

The project uses **GitHub Actions** to automatically build, push, and deploy the service.

Workflow file:

```
.github/workflows/patients-ci-cd.yaml
```

The pipeline triggers when:

* Code is pushed to the **main branch**
* The workflow is triggered manually using **workflow_dispatch**

---

# ⚙️ CI/CD Pipeline Steps Explained

The CI/CD workflow defined in:

```
.github/workflows/patients-ci-cd.yaml
```

performs the following steps automatically.

---

## 1️⃣ Checkout Repository

```yaml
- name: Checkout
  uses: actions/checkout@v4
```

This step downloads the repository code into the GitHub Actions runner so the pipeline can access the project files.

---

# 2️⃣ Configure AWS Credentials via OIDC

```yaml
- name: Configure AWS credentials via OIDC
  uses: aws-actions/configure-aws-credentials@v4
```

This step authenticates GitHub Actions with AWS using **OIDC (OpenID Connect)**.

Instead of storing AWS keys, GitHub assumes an IAM role defined in:

```
AWS_dev_OIDC_ARN
```

This provides **temporary secure credentials**.

---

# 3️⃣ Login to Amazon ECR

```yaml
- name: Login to Amazon ECR
  uses: aws-actions/amazon-ecr-login@v2
```

This step logs into **Amazon Elastic Container Registry (ECR)** so Docker images can be pushed.

---

# 4️⃣ Generate Docker Image URI

The pipeline dynamically generates the Docker image path using:

* AWS Account ID
* ECR repository
* Git commit SHA

Example

```
123456789012.dkr.ecr.us-east-1.amazonaws.com/patients:commitSHA
```

---

# 5️⃣ Build Docker Image

```bash
docker build -t $IMAGE_URI .
```

Builds the Docker container image using the **Dockerfile** in the repository.

---

# 6️⃣ Push Image to ECR

```bash
docker push $IMAGE_URI
```

Uploads the Docker image to **Amazon ECR** so Kubernetes can pull it.

---

# 7️⃣ Update kubeconfig for EKS

```bash
aws eks update-kubeconfig
```

This command connects the GitHub Actions runner to the **Amazon EKS cluster** so it can execute `kubectl` commands.

---

# 8️⃣ Update Image in Kubernetes Manifest

```bash
sed -i "s|image:.*|image: $IMAGE_URI|g" manifests/patients-dev.yaml
```

This replaces the container image in the Kubernetes manifest with the **newly built Docker image**.

Example

Before

```
image: old-image
```

After

```
image: 123456789012.dkr.ecr.us-east-1.amazonaws.com/patients:commitSHA
```

---

# 9️⃣ Install kubectl

```yaml
uses: azure/setup-kubectl@v4
```

This installs **kubectl** in the GitHub runner so it can communicate with the Kubernetes cluster.

---

# 🔟 Deploy to EKS

```bash
kubectl apply -f manifests/patients-dev.yaml
```

This deploys the updated application to the **Amazon EKS cluster**.

Kubernetes will:

* Pull the Docker image from **Amazon ECR**
* Create or update **pods**
* Run the latest version of the patient service

---

# 🔐 Required GitHub Secrets

Configure the following secrets in **GitHub → Repository Settings → Secrets**.

| Secret           | Description                                  |
| ---------------- | -------------------------------------------- |
| AWS_dev_OIDC_ARN | IAM Role used for GitHub OIDC authentication |
| AWS_dev_REGION   | AWS region                                   |
| AWS_ACCOUNT_ID   | AWS account ID                               |
| EKS_CLUSTER_NAME | Name of the EKS cluster                      |

---

# 🔄 CI/CD Flow Overview

```
Developer Push Code
        │
        ▼
GitHub Actions Trigger
        │
        ▼
Build Docker Image
        │
        ▼
Push Image → Amazon ECR
        │
        ▼
Update Kubernetes Manifest
        │
        ▼
Deploy to AWS EKS
        │
        ▼
Patient Service Running in Kubernetes
```
