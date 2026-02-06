# Graded Assignment on Container Orchestration

The goal of this assignment is to showcase: <br>

CI/CD pipeline automation using Jenkins tool <br>
Kubernetes deployment with MERN-based application <br>
Helm charts and templates <br>

# 🛒 ShopNow E-Commerce - Kubernetes Learning Project

ShopNow is a **Kubernetes learning project** built around a full-stack MERN e-commerce application:
- **Customer App** (React frontend)  
- **Admin Dashboard** (React admin panel)  
- **Backend API** (Express + MongoDB)  

#  ShopNow application having the following components:

Admin UI – React-based admin dashboard <br>
Backend (BE) – Node.js + Express REST API <br>
Frontend (FE) – React application served via Nginx <br>
Database – MongoDB (external MongoDB Atlas) <br>

## 📁 Project Structure

```
shopNow/
├── backend/               # Node.js API server
├── frontend/              # React customer app
├── admin/                 # React admin dashboard
├── kubernetes
│   ├── k8s-manifests/     # Raw Kubernetes YAML files
│   ├── helm/              # Helm charts for package management
│   │   └── charts/        # Individual charts
│   ├── argocd/            # GitOps deployment configs
│   └── pre-req/           # Cluster prerequisites
├── jenkins/               # Pipeline definitions (CI & CD)       
├── docs/                  # learning resources and guides
└── scripts/               # Automation and utility scripts
```

## 📋 Assignment: Kubernetes, Helm, and Jenkins Automation

This section documents the assignment implementation for Kubernetes deployment, Helm chart creation, and Jenkins CI/CD automation for the ShopNow MERN application.

### Assignment Overview

The assignment required the following deliverables:

1. **Kubernetes Deployment Files** - Deploy both frontend and backend components with seamless scalability
2. **HELM Charts** - Streamline deployment and configuration management
3. **Jenkins CI/CD Pipelines** - Automate build and deployment processes

### What Was Implemented

#### 1. Kubernetes Deployment Files

**Location**: `kubernetes/k8s-manifests/`

The deployment files include:

- **Backend Deployment** (`backend/deployment.yaml`)
  - Containerized Express.js API server
  - Health checks (liveness & readiness probes)
  - Resource requests and limits (100m CPU / 200Mi memory)
  - Secrets-based environment configuration for MongoDB URI

- **Frontend Deployment** (`frontend/deployment.yaml`)
  - React customer app served via Nginx
  - Proxy configuration to backend API
  - SPA routing support

- **Admin Deployment** (`admin/deployment.yaml`)
  - React admin dashboard with Nginx

- **Database Deployment** (`database/mongo-statefulset.yaml`)
  - MongoDB StatefulSet for persistent data
  - Headless service for internal cluster communication
  - Persistent volume claims for data durability

- **Services** (`**/service-*.yaml`)
  - ClusterIP for internal service-to-service communication
  - Ingress for external HTTP/HTTPS routing

- **Ingress** (`ingress/ingress-shopnow.yaml`)
  - Multi-path routing for customer app, admin, and backend API
  - nginx-ingress controller integration

- **Horizontal Pod Autoscalers (HPA)** (`**/hpa.yaml`)
  - Auto-scaling based on CPU utilization (80% threshold)
  - Min replicas: 1, Max replicas: 3

- **ConfigMaps & Secrets** 
  - `cm-*.yaml` - Nginx and application configurations
  - `secrets-*.yaml` - Database credentials (MongoDB URI, authentication)

#### 2. HELM Charts

**Location**: `kubernetes/helm/charts/.`

Helm charts created for each component:

- **Backend Helm Chart** (`backend/`)
  - `Chart.yaml` - Chart metadata
  - `values.yaml` - Default configuration values
  - `templates/` - Kubernetes resource templates
    - `deployment.yaml` - Backend deployment template
    - `service.yaml` - Backend service template
    - `configmap.yaml` - Configuration management
    - `secret.yaml` - Secret management for MongoDB URI
    - `hpa.yaml` - Auto-scaling rules

- **Frontend Helm Chart** (`frontend/`)
  - Similar structure to backend
  - Nginx configuration templating
  - Ingress routing configuration

- **Admin Helm Chart** (`admin/`)
  - Admin dashboard deployment with Helm

- **MongoDB Helm Chart** (`mongo/`)
  - Database StatefulSet deployment
  - Persistent storage configuration
  - Service definitions

**Key Features**:
- Configurable image repositories, tags, and pull policies
- Customizable resource limits and requests
- HPA settings for auto-scaling
- Environment variable templates
- Secret injection via Kubernetes Secrets

**Example Deployment**:
```bash
helm upgrade --install shopnow-backend kubernetes/helm/charts/backend \
  --namespace shopnow --create-namespace \
  --set image.repository=<registry>/shopnow-backend \
  --set image.tag=latest \
  --set secret.MONGODB_URI="<mongodb-connection-string>" \
  --wait --timeout 5m
```

#### 3. Jenkins CI/CD Pipelines

**Location**: `jenkins/`

Jenkins automation includes:

**CI Pipelines** (Build & Push Images):

- **Jenkinsfile.ci.backend**
  - Checks out code from GitHub
  - Generates image tag from git commit hash
  - Builds Docker image for backend
  - Pushes to ECR registry
  - Archives image tag artifact
  - **Automatically triggers CD pipeline** with produced IMAGE_TAG

- **Jenkinsfile.ci.frontend**
  - Similar workflow for frontend React app
  - Builds and pushes frontend image
  - Automatically triggers frontend CD job

- **Jenkinsfile.ci.admin**
  - CI pipeline for admin dashboard component

**CD Pipelines** (Deploy via Helm):

- **Jenkinsfile.cd.backend**
  - Receives IMAGE_TAG parameter from CI job
  - Checks out code from GitHub
  - Deploys backend using Helm with the specified image tag
  - Kubernetes manifest path: `kubernetes/helm/charts/backend`
  - Monitors rollout status (3-minute timeout)
  - Uses kubeconfig credential from Jenkins secrets

- **Jenkinsfile.cd.frontend**
  - Deploys frontend using Helm
  - Kubernetes manifest path: `kubernetes/helm/charts/frontend`
  - Similar rollout monitoring

**CI/CD Flow**:
```
GitHub Code Push 
  → CI Job (build & push image) 
    → Artifact: image-tag.txt 
      → Auto-trigger CD Job (deploy with image tag)
        → Helm upgrade on Kubernetes cluster
          → Rollout verification
```

**Jenkins Configuration Required**:

Create the following Jenkins credentials:
- **docker-reg-cred** (Username/Password)
  - Username: AWS
  - Password: ECR access token
  - Used for: Docker registry authentication

- **kubeconfig-credential** (File)
  - Content: Kubernetes cluster kubeconfig
  - Used for: kubectl/helm cluster access

**Example CI Job Parameters**:
- Git repository: https://github.com/durganaresh83/CapStone-B13-shopNow.git
- Branch: feature/assignment (or main)

**Example CD Job Parameters**:
- IMAGE_TAG: Passed from CI job automatically
- NAMESPACE: shopnow
- CHART_PATH: kubernetes/helm/charts/backend

### MongoDB Atlas Integration

The backend is configured to connect to MongoDB Atlas cloud database:

**Connection String**:
```
mongodb+srv://durganaresh:xxxxxxxxx@durga-cluster.htudsah.mongodb.net/?retryWrites=true&w=majority&appName=durga-cluster
```

**Configuration Locations**:
- **K8s Manifest**: `kubernetes/k8s-manifests/backend/secrets-db.yaml`
- **Helm Values**: `kubernetes/helm/charts/backend/values.yaml` (secret.MONGODB_URI)
- **Environment Variable**: `MONGODB_URI` injected into backend container

The MongoDB URI is stored as a Kubernetes Secret and mounted as an environment variable in the backend deployment.

### Deployment Methods

#### Method 1: Raw Kubernetes Manifests
```bash
kubectl apply -f kubernetes/k8s-manifests/namespace/
kubectl apply -f kubernetes/k8s-manifests/backend/secrets-db.yaml
kubectl apply -f kubernetes/k8s-manifests/backend/
kubectl apply -f kubernetes/k8s-manifests/frontend/
```

#### Method 2: Helm Charts (Recommended)
```bash
helm upgrade --install shopnow-backend kubernetes/helm/charts/backend \
  --namespace shopnow --create-namespace

helm upgrade --install shopnow-frontend kubernetes/helm/charts/frontend \
  --namespace shopnow
```

#### Method 3: Jenkins CI/CD Pipeline
1. Configure Jenkins with the required credentials
2. Create CI job (Jenkinsfile.ci.backend) → builds image
3. Create CD job (Jenkinsfile.cd.backend) → deploys via Helm
4. CI job automatically triggers CD job with image tag
5. Application deployed to a Kubernetes cluster


### Testing & Verification

To verify successful deployment:

# Check pod status
kubectl get pods -n shopnow

<img width="896" height="236" alt="image" src="https://github.com/user-attachments/assets/07879f54-1cd0-4017-ac4a-de6b774faeac" />


# Verify secrets are mounted
kubectl exec <backend-pod> -n shopnow -- env | grep MONGODB_URI

# Test application endpoints
curl http://<ingress-ip>/api/health


### Repository

**GitHub Repository**: https://github.com/durganaresh83/CapStone-B13-shopNow
**Branch**: feature/assignment

### Conclusion

<img width="1147" height="583" alt="image" src="https://github.com/user-attachments/assets/1e3da1a5-abba-49b3-8c69-78412d11f65a" />


This assignment demonstrates a comprehensive understanding of:
- **Kubernetes**: Deployment, StatefulSets, Services, Ingress, HPA, ConfigMaps, Secrets
- **Helm**: Chart templating, values customization, release management
- **Jenkins**: CI/CD pipeline design, artifact management, parameterized job triggering
- **DevOps**: Automation, scalability, security best practices in containerized environments

The implementation provides a production-ready deployment framework for MERN applications on Kubernetes with automated CI/CD pipelines.

---



