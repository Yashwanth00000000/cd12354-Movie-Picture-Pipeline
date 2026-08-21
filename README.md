# Movie Picture Pipeline

A full-stack web application serving as a catalog for Movie Pictures, automated with a complete CI/CD pipeline using GitHub Actions and deployed to AWS EKS (Elastic Kubernetes Service).

The project consists of two applications:

1. **Frontend** – A React (TypeScript-compatible) UI for browsing and viewing movie details.
2. **Backend** – A Python Flask REST API serving movie data.

## Project Structure

```
.
+-- .github/
¦   +-- workflows/
¦       +-- frontend-ci.yaml      # Frontend Continuous Integration
¦       +-- frontend-cd.yaml      # Frontend Continuous Deployment
¦       +-- backend-ci.yaml       # Backend Continuous Integration
¦       +-- backend-cd.yaml       # Backend Continuous Deployment
+-- setup/
¦   +-- terraform/                # AWS infrastructure as code
¦   +-- init.sh                   # Kubernetes IAM setup helper
¦   +-- workspace-setup.sh        # Environment setup script
+-- starter/
    +-- frontend/                 # React frontend application
    +-- backend/                  # Flask backend API
```

## CI/CD Workflows

This repository contains 4 GitHub Actions workflows:

| Workflow File | Name | Trigger | Jobs |
|---|---|---|---|
| `frontend-ci.yaml` | Frontend Continuous Integration | PR ? `main` (frontend changes) | lint ? test ? build |
| `backend-ci.yaml` | Backend Continuous Integration | PR ? `main` (backend changes) | lint ? test ? build |
| `frontend-cd.yaml` | Frontend Continuous Deployment | Push ? `main` (frontend changes) | lint ? test ? build+ECR ? deploy EKS |
| `backend-cd.yaml` | Backend Continuous Deployment | Push ? `main` (backend changes) | lint ? test ? build+ECR ? deploy EKS |

All workflows also support **manual triggering** via `workflow_dispatch`.

### Required GitHub Secrets

| Secret | Description |
|---|---|
| `AWS_ACCESS_KEY_ID` | IAM access key for `github-action-user` |
| `AWS_SECRET_ACCESS_KEY` | IAM secret access key for `github-action-user` |
| `AWS_REGION` | AWS region (e.g. `us-east-1`) |
| `AWS_ACCOUNT_ID` | Numeric AWS account ID |
| `ECR_REPO_FRONTEND` | ECR repository name for frontend (e.g. `frontend`) |
| `ECR_REPO_BACKEND` | ECR repository name for backend (e.g. `backend`) |
| `EKS_CLUSTER_NAME` | EKS cluster name (Terraform default: `cluster`) |
| `REACT_APP_MOVIE_API_URL` | External URL of backend LoadBalancer |

## Infrastructure Setup

### Prerequisites

- AWS account with appropriate IAM permissions
- Terraform installed
- kubectl installed
- AWS CLI configured

### Create AWS Infrastructure

```bash
cd setup/terraform
terraform apply
```

Take note of the Terraform outputs — you'll need them for configuring GitHub Secrets.

```bash
cd setup/terraform
terraform output
```

### Add GitHub Actions User to Kubernetes

```bash
cd setup
./init.sh
```

### Generate AWS Access Keys

1. Go to IAM ? Users ? `github-action-user`
2. Under Security Credentials ? Create access key
3. Select "Application running outside AWS"
4. Copy the keys and add them as GitHub Secrets

## Frontend Development

### Running Tests

```bash
cd starter/frontend

# Use correct NodeJS version
nvm use

# Install dependencies
npm ci

# Run tests
CI=true npm test

# Expected output
# PASS src/components/__tests__/MovieList.test.js
# PASS src/components/__tests__/App.test.js
# Tests: 3 passed, 3 total
```

### Running Linter

```bash
npm run lint
```

### Build & Run Locally

```bash
cd starter/frontend
npm ci
REACT_APP_MOVIE_API_URL=http://localhost:5000 npm start
```

### Docker Build

```bash
docker build --build-arg=REACT_APP_MOVIE_API_URL=http://localhost:5000 --tag=mp-frontend:latest .
docker run --name mp-frontend -p 3000:3000 -d mp-frontend
```

### Deploy Kubernetes Manifests

```bash
cd starter/frontend/k8s
kustomize edit set image frontend=<ECR_REPO_URL>:<NEW_TAG_HERE>
kustomize build | kubectl apply -f -
```

## Backend Development

### Running Tests

```bash
cd starter/backend

# Install dependencies
pipenv install

# Run tests
pipenv run test

# Expected output
# test_app.py::test_movies_endpoint_returns_200 PASSED
# test_app.py::test_movies_endpoint_returns_json PASSED
# test_app.py::test_movies_endpoint_returns_valid_data PASSED
```

### Running Linter

```bash
pipenv run lint
```

### Build & Run Locally

```bash
cd starter/backend
pipenv install
pipenv run serve
```

### Docker Build

```bash
cd starter/backend
docker build --tag mp-backend:latest .
docker run -p 5000:5000 --name mp-backend -d mp-backend

# Test the API
curl http://localhost:5000/movies
# {"movies":[{"id":"123","title":"Top Gun: Maverick"},{"id":"456","title":"Sonic the Hedgehog"},{"id":"789","title":"A Quiet Place"}]}
```

### Deploy Kubernetes Manifests

```bash
cd starter/backend/k8s
kustomize edit set image backend=<ECR_REPO_URL>:<NEW_TAG_HERE>
kustomize build | kubectl apply -f -
```

## Dependencies

| Tool | Purpose |
|---|---|
| [docker](https://docs.docker.com/desktop/install/debian/) | Build frontend and backend images |
| [kubectl](https://kubernetes.io/docs/tasks/tools/) | Apply Kubernetes manifests |
| [pipenv](https://pipenv.pypa.io/en/latest/install/) | Python version and dependency management |
| [nvm](https://github.com/nvm-sh/nvm) | Node.js version management |
| [terraform](https://developer.hashicorp.com/terraform) | AWS infrastructure provisioning |
| [kustomize](https://kubectl.docs.kubernetes.io/installation/kustomize/) | Kubernetes manifest management |
| [jq](https://stedolan.github.io/jq/download/) | JSON parsing on command line |

## License

[MIT License](LICENSE.md) © 2024 Yashwanth Kattamuri
