# rb-k8s-pullthrough-cache

## Description

**Ref:** [How to deploy... (Perplexity.ai thread)](https://www.perplexity.ai/search/is-it-possible-to-deploy-a-con-KkymAxeOTKyOCrbxg11mBg)

Step-by-step guide to deploying a container image registry as a Kubernetes pull-through cache.

## Guide

### Prerequisites

- A Kubernetes cluster
- Helm
- Docker
- kubectl

### Step 1: Deploy the Registry

#### `registry.yaml`: Create the registry deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: docker-registry
spec:
  replicas: 1
  selector:
    matchLabels:
      app: docker-registry
  template:
    metadata:
      labels:
        app: docker-registry
    spec:
      containers:
      - name: registry
        image: registry:2
        ports:
        - containerPort: 5000
        volumeMounts:
        - name: registry-data
          mountPath: /var/lib/registry
      volumes:
      - name: registry-data
        emptyDir: {}
```

#### Deploy the registry

```bash
$ kubectl apply -f registry.yaml
```

### Step 2: Configure the registry as a pull-through cache

#### `config.yaml`: Configure the registry

```yaml
version: 0.1
storage:
  filesystem:
    rootdirectory: /var/lib/registry
http:
  addr: :5000
proxy:
  remoteurl: https://registry-1.docker.io  # Docker Hub

```

### Step 3: Use the cache in deployments

#### `deployment.yaml`: Use the cache in deployments (example)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: <your-registry-url>:5000/<your-username>/myapp:latest  # Replace with your registry URL, username, and app name
        ports:
        - containerPort: 80
```

#### Deploy the deployment

```bash
$ kubectl apply -f deployment.yaml
```









