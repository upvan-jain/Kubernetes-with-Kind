# Kubernetes with Kind 

This guide explains how to set up and use Kubernetes locally with **Kind** on Windows. 
It includes deployment, services, Kubernetes Dashboard, and Metrics Server setup.

---
## Prerequisites
Install Docker Desktop (if not installed). https://www.docker.com/products/docker-desktop/
Enable WSL2 backend during installation.
Give Docker at least 2 CPUs and 2GB RAM in settings.

Install kubectl (Kubernetes CLI): 
```bash
choco install kubernetes-cli
```
(Or download from official Kubernetes site)

Install kind:
```bash
choco install kind
```


## 1. Create a Kind Cluster
```bash
kind create cluster --name demo-cluster
kubectl get nodes
```

---

## 2. Deploy an Application
Example: Nginx Deployment
```bash
kubectl create deployment nginx --image=nginx --replicas=2
kubectl get deployments
kubectl get pods -o wide
```

---

## 3. Expose the Deployment as a Service
```bash
kubectl expose deployment nginx --type=NodePort --name=nginx-service --port=80
kubectl get svc nginx-service
```

Access it locally:
```bash
kubectl port-forward service/nginx-service 8080:80
```
Open browser: http://localhost:8080

---

## 4. Deploy Kubernetes Dashboard
Deploy:
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

Create Admin User (`dashboard-admin.yaml`):
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```

Apply and get token:
```bash
kubectl apply -f dashboard-admin.yaml
kubectl -n kubernetes-dashboard create token admin-user
```

Access:
```bash
kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard 8443:443
```
Browser: https://localhost:8443

---

## 5. Install Metrics Server
Deploy:
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Patch for Kind:
```bash
kubectl -n kube-system patch deployment metrics-server   --type='json'   -p='[{"op":"add","path":"/spec/template/spec/containers/0/args","value":["--kubelet-insecure-tls","--kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname"]}]'
```

Verify:
```bash
kubectl top nodes
kubectl top pods -A
```

---

## 6. Why These Flags are Required?
- **--kubelet-insecure-tls** → Bypass cert check (needed for local/self-signed clusters).  
- **--kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname** → Ensures Metrics Server can reach kubelet inside Docker.

⚠️ In production, proper TLS certs and DNS should be used instead.

---

## 7. Cleanup
```bash
kubectl delete svc nginx-service
kubectl delete deployment nginx
kind delete cluster --name demo-cluster
```

---

✅ With these steps, you can run Kubernetes locally using Kind, deploy apps, use the Dashboard, and monitor metrics!
