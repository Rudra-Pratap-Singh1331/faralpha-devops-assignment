# FarAlpha DevOps Internship Assignment — Flask + MongoDB on Kubernetes

This repository contains my complete submission for the FarAlpha Technologies DevOps Internship Assignment.  
The project implements a Flask application connected to a MongoDB database, containerized using Docker, and deployed on a Kubernetes cluster with StatefulSet, Secrets, Volumes, Services, Autoscaling, and DNS-based pod communication.

All assignment requirements have been fully completed.

---

# 📁 Project Structure

```faralpha-devops-assignment/
├── README.md
├── ANSWERS.md
└── flask-mongodb-app/
├── app.py
├── Dockerfile
├── requirements.txt
├── mongo-secret.yaml
├── mongo-pv.yaml
├── mongo-pvc.yaml
├── mongo-statefulset.yaml
├── mongo-service.yaml
├── flask-deployment.yaml
├── flask-service.yaml
├── flask-hpa.yaml
└── .gitignore
 ```

# 🚀 1. Flask Application Overview

The Flask app includes two endpoints:

### `/`

Returns: Welcome to the Flask app! The current time is: <timestamp>

### `/data`

- **POST** → Inserts JSON into MongoDB
- **GET** → Returns all stored documents

Flask runs on port **5000** inside the container.

---

# 🐳 2. Dockerfile (Flask Application)

FROM python:3.10-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY app.py .
EXPOSE 5000
CMD ["python", "app.py"]

---

# 🏗 3. Build & Push Docker Image

### Build:

docker build -t <dockerhub-username>/flask-mongo-app:latest .

### Login:

docker login

### Push:

docker push <dockerhub-username>/flask-mongo-app:latest

---

# ☸️ 4. Kubernetes Deployment Steps (Minikube / Docker Desktop)

## Step 1 — Create Secret (DB Username & Password)

kubectl apply -f mongo-secret.yaml

## Step 2 — Create Persistent Volume & PVC

kubectl apply -f mongo-pv.yaml
kubectl apply -f mongo-pvc.yaml

## Step 3 — Deploy MongoDB StatefulSet (With Authentication)

kubectl apply -f mongo-statefulset.yaml

## Step 4 — Expose MongoDB (ClusterIP)

kubectl apply -f mongo-service.yaml

## Step 5 — Deploy Flask Application (2 Replicas)

kubectl apply -f flask-deployment.yaml

## Step 6 — Expose Flask Using NodePort

kubectl apply -f flask-service.yaml

## Step 7 — Apply Horizontal Pod Autoscaler

kubectl apply -f flask-hpa.yaml

---

# 🔍 5. Verify All Deployments

kubectl get pods
kubectl get svc
kubectl get pvc
kubectl get pv
kubectl get hpa

---

# 🌐 6. Access Application

Flask NodePort service exposes the app at: http://localhost:30080

---

# 🔐 7. MongoDB Authentication

MongoDB credentials stored in Secret:

- `MONGO_INITDB_ROOT_USERNAME`
- `MONGO_INITDB_ROOT_PASSWORD`

Flask connects using secure auth URI: mongodb://mongo-user:mongo-pass@mongodb-service:27017/

MongoDB is **not exposed externally** — only accessible inside cluster.

---

# 📦 8. Why MongoDB Uses StatefulSet

- Needs **stable hostname** (`mongodb-0`)
- Needs **persistent volume** per pod
- Requires **ordered startup/shutdown**
- StatefulSet ensures pod identity + consistent storage

---

# 🔁 9. DNS Resolution inside Kubernetes

Every service gets DNS like: service-name.namespace.svc.cluster.local

Flask connects to MongoDB using: mongodb-service.default.svc.cluster.local

KubeDNS automatically resolves → ClusterIP → MongoDB Pod.

No IP hardcoding required.

---

# ⚙️ 10. Resource Requests & Limits

Configured values (as required):

| Resource | Requests | Limits  |
| -------- | -------- | ------- |
| CPU      | 0.2 CPU  | 0.5 CPU |
| Memory   | 250Mi    | 500Mi   |

**Requests** = minimum guaranteed  
**Limits** = maximum allowed

Ensures cluster stability and prevents resource overuse.

---

# 📈 11. Horizontal Pod Autoscaler (HPA)

- Minimum replicas: **2**
- Maximum replicas: **5**
- CPU threshold: **70%**

Pods scale **up** when CPU > 70%  
Scale **down** automatically when load reduces.

---

# 🧪 12. Testing Scenarios (Cookie Points)

## Autoscaling Load Test

kubectl run load --image=busybox --restart=Never -it -- sh -c "while true; do wget -qO- http://flask-service:5000

> /dev/null; done"

Observed:

- CPU usage exceeded 70%
- HPA increased replicas dynamically
- Scaled back when load stopped

## Database Test

Insert: curl -X POST -H "Content-Type: application/json" -d '{"sampleKey":"sampleValue"}' http://localhost:30080/data

Retrieve: curl http://localhost:30080/data

MongoDB persisted data even after Pod restart (verified PV/PVC).

---

# 🧠 13. Design Choices Summary

- **MongoDB StatefulSet** for persistence + stable identity
- **Flask Deployment** for scalability
- **ClusterIP for MongoDB** to keep DB internal
- **NodePort for Flask** to allow browser access
- **Secrets** for secure credentials
- **PV/PVC** for persistent MongoDB storage
- **HPA** for automatic load management
- **YAML separation** for clean modular configuration
