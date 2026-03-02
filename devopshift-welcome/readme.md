# Crypto Price Tracker Application

A Python-based application with a frontend and backend service that communicates via API and stores cryptocurrency price data in a MySQL database.

## Repository Structure

```
devopshift-welcome/
├── exam-code/
│   └── docker/
│       ├── fe/                     # Frontend service
│       │   ├── Dockerfile
│       │   ├── requirements.txt
│       │   └── src/
│       │       └── main.py
│       ├── be/                     # Backend service
│       │   ├── Dockerfile
│       │   ├── requirements.txt
│       │   └── src/
│       │       └── main.py
│       ├── docker-compose.yaml     # Docker Compose orchestration
│       ├── k8s/                    # Kubernetes manifests
│       │   ├── frontend-deployment.yaml
│       │   ├── frontend-service.yaml
│       │   ├── frontend-loadbalancer.yaml
│       │   ├── backend-deployment.yaml
│       │   ├── backend-service.yaml
│       │   ├── mysql-deployment.yaml
│       │   └── mysql-service.yaml
│       └── helm/                   # Helm charts
│           ├── frontend/
│           │   ├── Chart.yaml
│           │   ├── values.yaml
│           │   └── templates/
│           ├── backend/
│           │   ├── Chart.yaml
│           │   ├── values.yaml
│           │   └── templates/
│           └── mysql/
│               ├── Chart.yaml
│               ├── values.yaml
│               └── templates/
```

## Services Overview

| Service | Port | Description |
|---------|------|-------------|
| Frontend | 5002 | Python Flask web UI |
| Backend | 5001 | Python Flask API service |
| MySQL | 3306 | Database for crypto price data |

## Deployment Options

### Option 1: Docker Compose (Local Development)

```bash
cd exam-code/docker

# Build and run
docker-compose up --build

# Access the app
# Frontend: http://localhost:5002
# Backend: http://localhost:5001
```

### Option 2: Kubernetes (Raw Manifests)

```bash
cd exam-code/docker

# Build and push images (replace with your Docker Hub username)
docker build -t yourusername/frontend:latest ./fe
docker build -t yourusername/backend:latest ./be
docker push yourusername/frontend:latest
docker push yourusername/backend:latest

# Deploy to Kubernetes
kubectl apply -f k8s/

# Check status
kubectl get pods
kubectl get svc

# Access the app (Kind cluster)
kubectl port-forward svc/frontend-lb 8080:80
# Open http://localhost:8080
```

### Option 3: Helm Charts (Recommended for Production)

#### Prerequisites
- Kubernetes cluster running (Docker Desktop, Kind, Minikube, or cloud)
- Helm 3.x installed
- kubectl configured

#### Deployment Steps

```bash
cd exam-code/docker

# 1. Clean any existing deployments (optional)
helm uninstall frontend backend mysql 2>/dev/null
kubectl delete -f k8s/ 2>/dev/null

# 2. Deploy with Helm (order matters)
helm install mysql ./helm/mysql
helm install backend ./helm/backend
helm install frontend ./helm/frontend

# 3. Verify deployment
helm list
kubectl get pods
kubectl get svc

# 4. Wait for all pods to be Running
kubectl get pods -w

# 5. Access the application
kubectl port-forward svc/frontend-frontend-lb 8080:80
# Open http://localhost:8080 in browser
```

#### Helm Commands Reference

```bash
# List installed releases
helm list

# Check release status
helm status frontend

# Upgrade a release after changes
helm upgrade frontend ./helm/frontend

# Uninstall all releases
helm uninstall frontend backend mysql

# Validate chart syntax
helm lint ./helm/frontend
helm lint ./helm/backend
helm lint ./helm/mysql

# Preview what will be deployed (dry-run)
helm install frontend ./helm/frontend --dry-run=client
```

#### Customizing Values

Each Helm chart has a `values.yaml` file for customization:

**Frontend (helm/frontend/values.yaml)**
```yaml
replicaCount: 2
image:
  repository: danielshmatnik/frontend
  tag: latest
```

**Backend (helm/backend/values.yaml)**
```yaml
replicaCount: 2
mysql:
  host: mysqldb
  password: "123456"
```

**MySQL (helm/mysql/values.yaml)**
```yaml
mysql:
  rootPassword: "123456"
  database: crypto_db
```

Override values during install:
```bash
helm install frontend ./helm/frontend --set replicaCount=3
```

## Troubleshooting

### Check pod logs
```bash
kubectl logs -l app=frontend
kubectl logs -l app=backend
kubectl logs -l app=mysqldb
```

### Pod not starting (ImagePullBackOff)
- Ensure images are pushed to Docker Hub
- Check image name matches in deployment

### Backend can't connect to MySQL
- Ensure MySQL pod is running first
- Check service name is `mysqldb`
- Verify MYSQL_HOST environment variable

### Frontend can't connect to Backend
- Ensure backend service name is `backend-service`
- Check backend pod is running

## Architecture

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Frontend  │────>│   Backend   │────>│    MySQL    │
│   (Flask)   │     │   (Flask)   │     │   (5.7)     │
│   :5002     │     │   :5001     │     │   :3306     │
└─────────────┘     └─────────────┘     └─────────────┘
       │
       │ LoadBalancer :80
       ▼
   External Access
```
