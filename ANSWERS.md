# FarAlpha DevOps Internship — Assignment Answers

## 1. Benefits of using a virtual environment (Cookie Point)

- Virtual environment isolates project dependencies.
- Allows different projects to use different package versions safely.
- Prevents conflicts with system-wide Python packages.
- Keeps the project reproducible and portable.
- Protects global Python installation from accidental damage.

---

## 2. How authentication was enabled in MongoDB

- A Kubernetes Secret was created to store `MONGO_INITDB_ROOT_USERNAME` and `MONGO_INITDB_ROOT_PASSWORD`.
- These values were injected into the MongoDB StatefulSet using environment variables.
- MongoDB automatically enables authentication when credentials are provided.
- Flask app was updated to connect using: mongodb://<username>:<password>@mongodb-service:27017/
- This ensures secure access and prevents unauthenticated connections.

---

## 3. Why MongoDB uses StatefulSet instead of Deployment

- MongoDB requires stable network identity (like `mongodb-0`, `mongodb-1`).
- Needs persistent storage that stays attached across pod restarts.
- Requires ordered, predictable startup and shutdown.
- StatefulSet maintains pod identity + volume consistency.
- Deployment cannot guarantee stable pod names or persistent volume mapping.

---

## 4. Kubernetes Service Types Used & Why

### MongoDB → ClusterIP  
- Should not be exposed externally.  
- Only Flask application must access it internally.  
- Secure internal-only communication.  

### Flask App → NodePort  
- Needed external access from host browser.
- Simple method for Minikube / local Kubernetes.
- Makes the Flask API reachable via `localhost:<nodePort>`.

---

## 5. PV & PVC Purpose + Why Used

- MongoDB stores data at `/data/db` and must persist it.
- Pod restarts, rescheduling, or failures should NOT delete data.
- **Persistent Volume (PV)** allocates physical storage.
- **Persistent Volume Claim (PVC)** requests storage dynamically.
- Ensures MongoDB always keeps its data even if pod is recreated.

---

## 6. Horizontal Pod Autoscaler (HPA) Explanation

- HPA monitors CPU usage of Flask pods.
- Scales replicas automatically based on load.
- Minimum replicas: **2**  
- Maximum replicas: **5**  
- Trigger threshold: **70% CPU**  
- When load increases → adds more pods.
- When load decreases → scales back to 2 pods.
- Ensures application reliability under peak load.

---

## 7. DNS Resolution in Kubernetes (Very Important)

Inside Kubernetes, every Service gets its own DNS name:
service-name.namespace.svc.cluster.local

For MongoDB:
mongodb-service.default.svc.cluster.local

How it works:

1. Flask application receives MongoDB service name in connection string.
2. kube-dns (or CoreDNS) resolves the name into the ClusterIP.
3. ClusterIP forwards requests to MongoDB pod(s).
4. No IP hardcoding needed → everything stays dynamic.

This ensures:

- Automatic service discovery  
- Smooth communication between pods  
- No manual IP management  
- Fully internal traffic routing  

---

## 8. Resource Requests & Limits Explanation

### Requests
- Minimum guaranteed CPU & memory for a pod.
- Kubernetes scheduler uses them to decide placement.
- Ensures application always gets required baseline resources.

### Limits
- Maximum CPU/memory a pod is allowed to consume.
- Prevents runaway processes.
- Protects the cluster from single-pod overconsumption.

### Required values used:
- **Requests:** 0.2 CPU, 250Mi  
- **Limits:** 0.5 CPU, 500Mi  

These values ensure stability and efficient resource allocation.

---

## 9. Design Choices

- **MongoDB StatefulSet:** Required for stable identity + persistence.
- **Flask Deployment:** Best for stateless scalable applications.
- **NodePort for Flask:** Needed for external access on local clusters.
- **ClusterIP for MongoDB:** Internal-only access for security.
- **Secrets for credentials:** Prevents passwords from being stored in plain YAML.
- **PV + PVC:** Ensures database persistence.
- **HPA:** Enables auto scaling based on CPU load.
- **YAML separation:** Makes configuration modular and easy to manage.

Alternatives considered:
- Using Deployment for MongoDB → rejected due to lack of stability.
- Using LoadBalancer → unnecessary for local development.
- Using ConfigMaps instead of secrets → not secure for passwords.

---

## 10. Testing Scenarios (Cookie Point)

### Autoscaling Test
Command used:
kubectl run load --image=busybox --restart=Never -it -- sh -c "while true; do wget -qO- http://flask-service:5000 > /dev/null; done"

Observations:
- CPU usage increased above 70%.
- HPA automatically increased replicas from 2 → 3 → 4.
- After load stopped, replicas scaled back down to 2.
- Autoscaling behaved correctly.

### Database Test
Command:
curl -X POST -H "Content-Type: application/json" -d '{"sampleKey":"sampleValue"}' http://localhost:30080/data

Retrieve:
curl http://localhost:30080/data

Results:
- Data inserted successfully.
- Data persisted even after restarting MongoDB pod.
- Verified that PV/PVC were working correctly.
