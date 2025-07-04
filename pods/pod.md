# Ì†ΩÌ∞≥ Kubernetes Pods

Each entity created with Kubernetes ‚Äî such as Pods, Services, Deployments, and ReplicationControllers ‚Äî is treated as a **resource**.

Resources in Kubernetes can be defined using either **YAML** or **JSON** format.

---

## Ì†ΩÌ≥Ñ Resource Configuration Structure

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

## Ì†ΩÌ¥ç Useful Commands
‚úÖ List all running Pods:
```bash 
kubectl get pods
```
‚úÖ List all API  Resources:
```bash
kubectl api-resources
```


##  Ì†ΩÌ∫Ä Kubernetes Command Cheat Sheet
A quick reference for commonly used `kubectl` commands. Ideal for CKA and day-to-day Kubernetes work.

---

## Ì†ΩÌ≥¶ Pods

### ‚úÖ List all Pods
```bash
kubectl get pods
```

Ì†ΩÌ¥ç Get detailed info about Pos
```bash 
kubectl describe pod <pod-name>
```

Ì†ΩÌ≥Ñ Crear pod using a YAML file
```bash
kubectl apply -f pod.yaml
```


Ì†ΩÌ≥ö YAML Template for a Pod
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

## Ì†æÌ∑™ Dry Run (Generate YAML Without Applying)
### Ì†ΩÌ∫´ Don't create, just generate YAML 
```bash
kubectl run test-pod \
   --image=nginx \
   --dry-run=client \
   -o yaml > test-pod.yaml
```

## Ì†ΩÌ≥å General Tips
Use ``kubectl explain pod`` to explore YAML specs

Use ``kubectl api-resources`` to list all Kubernetes resource types

Use ``kubectl get all`` to list all workloads in the current namespace


