# Mini-Project-Gitops-Application-Deployment

Module 2: Application Deployment and Management in ArgoCD
Introduction

In this module, we'll gain hands-on experience with ArgoCD, focusing on deploying and managing applications. We'll learn how to define applications in ArgoCD, manage their lifecycle, and structure Git repositories for optimal use with ArgoCD.

Lesson 2.1: Defining and Deploying Applications with ArgoCD
Objective: Learn how to define and deploy applications using ArgoCD in a Kubernetes environment.
Steps:

1. Define an Application in ArgoCD:

- Create a YAML file (' app-definition.yaml') defining your application.

- Code Snippet:

```

apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: sample-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://github.com/your-repo/sample-app.git'
    path: k8s
    targetRevision: HEAD
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: sample-namespace

```


- Explanation: This YAML file specifies the application's source repository, the path within the repo, and the target Kubernetes cluster and namespace.


2. Deploy the Application:


- Apply the YAML file using 'kubectl'.

- Command:

```

kubectl apply -f app-definition.yaml -n argocd


```

- Resources 


	- [ArgoCD Application Declarative Setup](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/)

Lesson 2.2: Managing Application Lifecycle: Syncing, Rollbacks, and Health Status
Objective: Understand how to manage the application lifecycle in ArgoCD.


Steps:

1. Syncing Application:


- Use ArgoCD CLI or Ul to synchronize the application.

- Command:

```

argocd app sync sample-app


```

- Explanation: This synchronizes the application's state with the desired state in the Git repository.



2. Deploy the Application:

- Apply the YAML file using 'kubectl'.

- Command:

```

argocd app get sample-app


```


- Explanation: This command provides information on the sync and health status of your application.


3. Performing Rollbacks:

- Rollback to a previous deployment state using ArgoCD Ul or CLI.

- Command:

```

argocd app rollback sample-app



```

* Explanation: This rolls back the application to its previous stable state.


Resources: 

	- [ArgoCD Application Lifecycle Management](https://argo-cd.readthedocs.io/en/stable/user-guide/application_lifecycle/)


Lesson 2.3: Structuring Git Repositories for Optimal ArgoCD Use
Objective: Learn how to structure Git repositories effectively for use with ArgoCD


Steps:

1. Repository Structure:

- Organize your repository with a clear structure, separating manifests, configurations, and environments.

- Example Structure:


```

├── k8s/
│   ├── dev/
│   │   └── deployment.yaml
│   └── prod/
│       └── deployment.yaml
├── src/
└── README.md


```

2. Environment Separation:


- Use different directories or branches for different environments (e.g., dev, staging, prod).

- Explanation: This helps in managing different configurations and ensures clarity.

3. Version Control Best Practices:

- Commit changes with meaningful messages and use tags/releases for versioning.

Resources:

-  Best Practices for Structuring Repositories


-------------------------------------------------------------------------------------------------------------------------------

                                                PROJECT IMPLEMENTATION
-------------------------------------------------------------------------------------------------------------------------------

Lesson 2.1: Defining and Deploying Applications with ArgoCD
1. Create Application Definition
app-definition.yaml:

```

apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: sample-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://github.com/your-username/your-repo.git'
    path: k8s/dev
    targetRevision: HEAD
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: sample-namespace
  syncPolicy:
    automated:
      selfHeal: true
      prune: true

```

Explanation: This defines an ArgoCD Application that will:

Monitor the k8s/dev directory in your Git repository

Deploy resources to the sample-namespace in your current Kubernetes cluster

Automatically sync and self-heal


2. Deploy the Application


```

# First, ensure you can access ArgoCD (use port-forwarding if needed)
kubectl port-forward svc/argocd-server -n argocd 8090:443 

# Get the admin password
ARGOCD_PASSWORD=$(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d)

# Login to ArgoCD
argocd login localhost:8090 --insecure --username admin --password $ARGOCD_PASSWORD

# Create the target namespace
kubectl create namespace sample-namespace

# Deploy the ArgoCD Application
kubectl apply -f app-definition.yaml -n argocd

```

<img width="1648" height="56" alt="image" src="https://github.com/user-attachments/assets/f431ef22-46bb-4204-9777-c1b86580d721" />

Lesson 2.2: Managing Application Lifecycle
1. Syncing Application

```
# Manual sync
argocd app sync sample-app

# Check sync status
argocd app get sample-app

```

<img width="1812" height="1488" alt="image" src="https://github.com/user-attachments/assets/08e8e5a3-1965-4b1d-a9c8-e295723da781" />


2. Monitoring Application Health

```

# Get detailed application information
argocd app get sample-app

# Watch application status
argocd app wait sample-app

# Check application health
argocd app health sample-app

# List application resources
argocd app resources sample-app

```

<img width="1832" height="1042" alt="image" src="https://github.com/user-attachments/assets/d38ab851-70fe-4b71-8807-eb11c683bc58" />


3. Performing Rollbacks

```
 View application history
argocd app history sample-app

# Rollback to previous version
argocd app rollback sample-app

# Rollback to specific revision
argocd app rollback sample-app --id <history-id>

```

<img width="1836" height="340" alt="image" src="https://github.com/user-attachments/assets/b4f548a8-1bd7-4641-9eaf-02eb67cd55fe" />

Lesson 2.3: Repository Structure Implementation

1. Current Structure

2. Environment-Specific Manifests

k8s/dev/deployment.yaml:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-app
  namespace: sample-namespace
  labels:
    app: sample-app
    environment: dev
spec:
  replicas: 2
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
        environment: dev
    spec:
      containers:
      - name: sample-app
        image: nginx:1.25-alpine
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
---
apiVersion: v1
kind: Service
metadata:
  name: sample-app-service
  namespace: sample-namespace
  labels:
    app: sample-app
    environment: dev
spec:
  selector:
    app: sample-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP

```

k8s/prod/deployment.yaml (Production):

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-app
  namespace: sample-namespace
  labels:
    app: sample-app
    environment: prod
spec:
  replicas: 5
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
        environment: prod
    spec:
      containers:
      - name: sample-app
        image: nginx:1.25-alpine
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1"
---
apiVersion: v1
kind: Service
metadata:
  name: sample-app-service
  namespace: sample-namespace
  labels:
    app: sample-app
    environment: prod
spec:
  selector:
    app: sample-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer

```


```

2. Environment-Specific Manifests

k8s/dev/deployment.yaml (Development):


# Commit changes with meaningful messages
git add .
git commit -m "feat: add dev and prod deployment manifests with environment-specific configurations"

# Tag releases for production deployments
git tag -a v1.0.0 -m "Production release v1.0.0"
git push origin v1.0.0

```

<img width="1852" height="1508" alt="image" src="https://github.com/user-attachments/assets/4e01ff97-201e-4376-891d-4bd91ff5084b" />


Verification and Testing

1. Verify Application Deployment

```

# Check if application is deployed
kubectl get deployments -n sample-namespace

# Verify services
kubectl get services -n sample-namespace

# Check pods
kubectl get pods -n sample-namespace

```

<img width="1862" height="1488" alt="image" src="https://github.com/user-attachments/assets/c9e957a4-e9f6-4f9a-b70c-f3b6385b6721" />


<img width="1700" height="158" alt="image" src="https://github.com/user-attachments/assets/a7fa16eb-5c7f-4fe0-b5f1-47deb76d43ad" />

2. Test ArgoCD Sync

```
# Make a change to your deployment file
# Update the image version in k8s/dev/deployment.yaml

# Commit and push changes
git add k8s/dev/deployment.yaml
git commit -m "chore: update dev image to nginx:1.26-alpine"
git push origin main

# Watch ArgoCD auto-sync the changes
argocd app get sample-app

```

3. Test Rollback Functionality

```
# Introduce a breaking change
# Change to an invalid image in k8s/dev/deployment.yaml

# Commit and push
git add k8s/dev/deployment.yaml
git commit -m "chore: test with invalid image"
git push origin main

# Watch sync fail, then rollback
argocd app history sample-app
argocd app rollback sample-app

```


<img width="2386" height="1450" alt="image" src="https://github.com/user-attachments/assets/670ce7d5-f1dc-41cd-9661-0d4fdf3c8753" />


<img width="2396" height="1378" alt="image" src="https://github.com/user-attachments/assets/9053fa51-2963-4763-bf24-1cb3fee31d2b" />


<img width="2414" height="1426" alt="image" src="https://github.com/user-attachments/assets/b1bc066e-4ee8-44e8-96be-3228caa3a596" />


<img width="2276" height="1464" alt="image" src="https://github.com/user-attachments/assets/c04681c0-10cf-4183-8c54-f26ef92da2bf" />


<img width="2338" height="1352" alt="image" src="https://github.com/user-attachments/assets/a299262e-a93f-4954-8d8e-90ba8c467382" />


<img width="2210" height="1470" alt="image" src="https://github.com/user-attachments/assets/6cfb626a-053a-4137-ac17-6393e7e20cf7" />


**Key Takeaways**

ArgoCD requires at least two successful deployments in history to perform rollbacks 

The rollback feature depends on Git commit history 



**Best Practices**

Use auto-sync for development environments where continuous deployment is beneficial

Consider manual sync for production environments where controlled deployments are preferred

Implement a promotion process where changes move from auto-sync dev environments to manual-sync prod environments

Use Git revert instead of ArgoCD rollback when auto-sync is enabled



