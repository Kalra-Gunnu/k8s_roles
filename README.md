# 🐳 FinX Flask App on Kubernetes with RBAC

This project demonstrates how to deploy a Flask-based microservice in a Kubernetes cluster using **Role-Based Access Control (RBAC)**. It validates restricted access for different user roles (`dev`, `qa`, and `prod`) using Kubernetes `ServiceAccounts`.

---

## 📁 Project Structure

.
├── app.py # Simple Flask app
├── Dockerfile # Image build file
├── finx-flask.yaml # Deployment and Service manifest
├── roles/ # Namespace, Role, RoleBinding, and ServiceAccount YAMLs
│ ├── dev-role/
│ ├── qa-clusterroles/
│ └── prod-role/
└── README.md # You're here!

---

## 🚀 Flask App Overview

A basic Flask app that returns a greeting from FinX Cloud Solutions:

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello from FinX Cloud Solutions!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)

```

🛠 Prerequisites

- Kubernetes cluster (e.g., Minikube, kind, EKS)
- kubectl configured
- Docker (for building the image)

🧱 Building and Pushing Docker Image

```bash
docker build -t {dockerhub-username}/finx-flask:latest .
docker push {dockerhub-username}/finx-flask:latest
```
Update finx-flask.yaml to use the above image.

🧩 Kubernetes Manifests

finx-flask.yaml includes:
- Deployment: Flask app running on port 5000
- Service: Exposes port 3001 mapped to container port 5000

🔐 RBAC Roles & Access Control

Namespaces:
- dev
- qa
- prod

Each namespace has:
- A ServiceAccount
- A Role
- A RoleBinding

✅ Validation Scenarios

🧪 a. Deploy as Dev, QA, and Prod
🟢 Dev: Can deploy only in dev
```bash
kubectl apply -f finx-flask.yaml -n dev --as=system:serviceaccount:dev:dev-team
```
✅ Should succeed in dev.

🔵 QA: Can list pods in all namespaces
```bash
kubectl get pods -n dev --as=system:serviceaccount:qa:qa-team
kubectl get pods -n qa --as=system:serviceaccount:qa:qa-team
kubectl get pods -n prod --as=system:serviceaccount:qa:qa-team
```
✅ Can view resources

```bash
kubectl apply -f finx-flask.yaml -n dev --as=system:serviceaccount:qa:qa-team
```
❌ Should fail — not authorized to create

🔴 Prod: Can deploy only in prod
```bash
kubectl apply -f finx-flask.yaml -n prod --as=system:serviceaccount:prod:prod-team
```
✅ Should succeed in prod.

🚫 b. Attempt Unauthorized Actions
✅ Dev trying to deploy in prod:

```bash
kubectl apply -f finx-flask.yaml -n prod --as=system:serviceaccount:dev:dev-team
```
❌ Forbidden

✅ QA trying to modify/delete:

```bash
kubectl delete pod <pod-name> -n qa --as=system:serviceaccount:qa:qa-team
```
❌ Forbidden

---

📜 Conclusion
This setup demonstrates environment-specific access control using RBAC in Kubernetes. Each team can only perform operations in their respective scope, enforcing least-privilege access.

---