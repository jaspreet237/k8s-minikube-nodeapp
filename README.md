Node.js App on Minikube — Deployment & Monitoring
Overview
This project deploys a tiny Node.js HTTP server to a local Kubernetes cluster (Minikube).
Goal: practice Kubernetes basics — containerize the app, deploy to Minikube, expose and test it locally, scale pods, and monitor using the Kubernetes Dashboard and kubectl.

Important: This project uses local images (no DockerHub). We built/loaded the image into Minikube so Kubernetes can pull it locally.

File / Folder Structure

k8s-minikube-nodeapp/

├── app.js                  # Simple Node.js HTTP server

├── package.json

├── Dockerfile

└── k8s/
    ├── deployment.yaml     # Kubernetes Deployment manifest
        └── service.yaml        # Kubernetes Service manifest
    
Why Node.js?
Lightweight and quick to run / test.

The Node.js HTTP server (app.js) gives a real app to deploy (not just a static server).

Good for demonstrating containerization, scaling, and logging in Kubernetes.


How the app works
app.js listens on port 3000 and returns a simple text response.

We build a Docker image nodeapp:latest locally and ensure it is available to the Minikube VM.

Kubernetes Deployment manages the Pods (replicas of the container).

A Service exposes the Deployment internally; we used kubectl port-forward to access it on localhost (instead of NodePort).

Manifests — short explanation

deployment.yaml (key points)

replicas: number of pod copies to run.

image: nodeapp:latest — local image name.

imagePullPolicy: Never — instructs K8s not to pull from remote registry (used when image is built/loaded locally).

container ports.containerPort: 3000.

service.yaml

Service selects pods by label app: nodeapp.

We keep the service as ClusterIP (default). For local access we used kubectl port-forward to map service → localhost.

Windows-friendly steps (what we did)

Pre-reqs: minikube, kubectl, docker (Docker Desktop), PowerShell.

**1. Start Minikube**
   
minikube start

**2a. Option A — Build image inside Minikube Docker (recommended on Windows PowerShell)**

This makes Kubernetes able to use the image directly:

# Point PowerShell's Docker client at Minikube's Docker daemon

& minikube -p minikube docker-env --shell powershell | Invoke-Expression

# Build the image into Minikube's Docker

docker build -t nodeapp:latest .

After these commands, nodeapp:latest lives inside Minikube's Docker and can be used by pods.

**2b. Option B — (Alternative) Build with Docker Desktop then load into Minikube**

If you already built locally with Docker Desktop (e.g. docker build -t nodeapp:latest .), load it into Minikube:

# Build locally (if not already)

docker build -t nodeapp:latest .

# Transfer image into Minikube

minikube image load nodeapp:latest

**3. Deploy to Kubernetes**

kubectl apply -f k8s/deployment.yaml

kubectl apply -f k8s/service.yaml

**4. Verify pods & service**

kubectl get pods

kubectl get svc

You will see pod names like nodeapp-deployment-xxxxx and SERVICE nodeapp-service.

How we exposed the app (port-forward)

We used kubectl port-forward (easy & ephemeral — avoids NodePort):

# Forward service port 3000 to localhost:3000

kubectl port-forward svc/nodeapp-service 3000:3000

Now open in browser:

http://localhost:3000

You should see:

Hello from Node.js app running in Kubernetes!

Note: You can also port-forward a specific pod:

kubectl port-forward pod/<pod-name> 3000:3000

Scaling the deployment

Scale from 2 → 4 pods

Scale from 2 → 4 pods

kubectl scale deployment nodeapp-deployment --replicas=4

kubectl get pods

(You can confirm replicas with kubectl get deployment nodeapp-deployment)

Logs and troubleshooting

View logs (for a pod)

kubectl logs <pod-name>

# Follow logs
kubectl logs -f <pod-name>

Describe resource (see events, reason for failures)

kubectl describe pod <pod-name>

kubectl describe deployment nodeapp-deployment

Monitoring

Kubernetes Dashboard (visual monitoring)

Start dashboard (opens a browser):

minikube dashboard

Dashboard shows:

Workloads (Deployments, ReplicaSets, Pods)

Services

Pod logs (select pod → Logs)

Events & resource status

CLI monitoring
kubectl get pods --watch — watch pods appear/terminate.

kubectl get events — watch cluster events.

Quick command summary (copy/paste)
powershell
Copy
Edit
minikube start

# Option A - PowerShell minikube docker env
& minikube -p minikube docker-env --shell powershell | Invoke-Expression
docker build -t nodeapp:latest .

# Option B - build via Docker Desktop then load
# docker build -t nodeapp:latest .
# minikube image load nodeapp:latest

kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml

kubectl get pods
kubectl get svc

# Expose locally
kubectl port-forward svc/nodeapp-service 3000:3000

# Scale
kubectl scale deployment nodeapp-deployment --replicas=0
kubectl scale deployment nodeapp-deployment --replicas=2
kubectl scale deployment nodeapp-deployment --replicas=4

# Logs & describe
kubectl logs <pod-name>
kubectl describe pod <pod-name>

# Dashboard
minikube dashboard

# Screenshots
<img width="517" height="138" alt="image" src="https://github.com/user-attachments/assets/063acc3f-4644-4dd2-aba7-92bb474e3b5b" />

<img width="940" height="574" alt="image" src="https://github.com/user-attachments/assets/330d7cdb-b82c-447f-9f72-e51235149766" />

<img width="940" height="570" alt="image" src="https://github.com/user-attachments/assets/f759d2ff-d981-4f15-a608-0af4029d5827" />

# Commands and Output for replicas and logs

PS C:\Devops\Project\k8s-minikube-nodeapp> kubectl scale deployment nodeapp-deployment --replicas=4

deployment.apps/nodeapp-deployment scaled

PS C:\Devops\Project\k8s-minikube-nodeapp> kubectl get pods

NAME                                  READY   STATUS    RESTARTS   AGE
nodeapp-deployment-749759b7c6-dfmv5   1/1     Running   0          10m
nodeapp-deployment-749759b7c6-flbbc   1/1     Running   0          18s
nodeapp-deployment-749759b7c6-rj2k4   1/1     Running   0          10m
nodeapp-deployment-749759b7c6-wwqbj   1/1     Running   0          18s

PS C:\Devops\Project\k8s-minikube-nodeapp> kubectl logs nodeapp-deployment-749759b7c6-dfmv5

> nodejs-k8s-demo@1.0.0 start
> node app.js

Server running at port 3000  

PS C:\Devops\Project\k8s-minikube-nodeapp> kubectl describe deployment nodeapp-deployment  

Name:                   nodeapp-deployment
Namespace:              default
CreationTimestamp:      Mon, 11 Aug 2025 10:06:15 +0530
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=nodeapp
Replicas:               4 desired | 4 updated | 4 total | 4 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nodeapp
  Containers:
   nodeapp:
    Image:         nodeapp:latest
    Port:          3000/TCP
    Host Port:     0/TCP
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Progressing    True    NewReplicaSetAvailable
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nodeapp-deployment-749759b7c6 (4/4 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  14m    deployment-controller  Scaled up replica set nodeapp-deployment-749759b7c6 from 0 to 2
  Normal  ScalingReplicaSet  4m34s  deployment-controller  Scaled up replica set nodeapp-deployment-749759b7c6 from 2 to 4


