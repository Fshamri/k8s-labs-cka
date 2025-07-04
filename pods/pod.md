# 🐳 Kubernetes Pods

Each entity created with Kubernetes — such as Pods, Services, Deployments, and ReplicationControllers — is treated as a **resource**.

Resources in Kubernetes can be defined using either **YAML** or **JSON** format.

---

## 📄 Resource Configuration Structure

The basic structure of a resource definition looks like this:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx
```

## 🔍 Useful Commands
✅ List all running Pods:
```bash 
kubectl get pods
```
✅ List all API  Resources:
```bash
kubectl api-resources
```


##  🚀 Kubernetes Command Cheat Sheet
A quick reference for commonly used `kubectl` commands. Ideal for CKA and day-to-day Kubernetes work.

---

## 📦 Pods

### ✅ List all Pods
```bash
kubectl get pods
```

🔍 Get detailed info about Pos
```bash 
kubectl describe pod <pod-name>
```

📄 Crear pod using a YAML file
```bash
kubectl apply -f pod.yaml
```


📚 YAML Template for a Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx
    ports:
    - containerPort: 80
```
apply using this command:
```bash
kubectl apply -f my-pod.yaml
```

## 🧪 Dry Run (Generate YAML Without Applying)
### 🚫 Don't create, just generate YAML 
```bash
kubectl run test-pod \
   --image=nginx \
   --dry-run=client \
   -o yaml > test-pod.yaml
```

## 📌 General Tips
Use ``kubectl explain pod`` to explore YAML specs

Use ``kubectl api-resources`` to list all Kubernetes resource types

Use ``kubectl get all`` to list all workloads in the current namespace

