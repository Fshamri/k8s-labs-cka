# 🐳 Kubernetes ReplicaSet & ReplicationController

## 📛 Table of Contents

- [📘 What Are Controllers?](#-what-are-controllers)
- [📆 ReplicationController (RC)](#-replicationcontroller-rc)
  - [✅ Key Features](#️-key-features)
  - [📄 YAML: ReplicationController Example](#-yaml-replicationcontroller-example)
  - [🛠 RC Commands](#-rc-commands)
- [⚙️ ReplicaSet (RS)](#️-replicaset-rs)
  - [✅ Key Features](#️-key-features-1)
  - [📄 YAML: ReplicaSet Example](#-yaml-replicaset-example)
  - [🛠 RS Commands](#-rs-commands)
- [🏷️ Labels and Selectors](#️-labels-and-selectors)
- [🔁 Updating Replica Count](#-updating-replica-count)
  - [📄 Option 1: Update YAML](#-option-1-update-yaml)
  - [🧪 Option 2: Use kubectl scale](#-option-2-use-kubectl-scale)
- [📌 Summary Commands](#-summary-commands)

---

## 📘 What Are Controllers?

Controllers are the **brains** behind Kubernetes. They continuously monitor Kubernetes resources and ensure that the current state matches the desired state defined by the user.

---

## 📆 ReplicationController (RC)

A **ReplicationController** ensures that a specified number of identical Pods are always running in the cluster.

### ✅ Key Features

- Maintains a stable set of replica Pods
- Ensures **high availability**
- Replaces failed Pods automatically
- Spreads load across worker nodes
- ⚠️ Deprecated in favor of **ReplicaSet**

---

### 📄 YAML: ReplicationController Example

File: `rc-definition.yaml`

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    app: myApp
    type: front-end
spec:
  replicas: 3
  selector:
    app: myApp
  template:
    metadata:
      labels:
        app: myApp
        type: front-end
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
```

📝 Notes:

- `replicas: 3` tells Kubernetes to run 3 identical Pods.
- `selector` must match the labels in the Pod template.

---

### 🛠 RC Commands

```bash
# Create the ReplicationController
kubectl create -f rc-definition.yaml

# List RCs
kubectl get rc

# Describe RC
kubectl describe rc myapp-rc
```

---

## ⚙️ ReplicaSet (RS)

A **ReplicaSet** is the next-generation controller that ensures a specified number of Pods are always running. It's used by Deployments and supports more flexible label matching.

### ✅ Key Features

- Ensures a set number of Pods are running at any time
- Supports advanced **label selectors**
- Can **adopt existing Pods** based on labels
- Belongs to the `apps/v1` API group

---

### 📄 YAML: ReplicaSet Example

File: `rs-definition.yaml`

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-rs
  labels:
    app: myApp
    type: front-end
spec:
  replicas: 3
  selector:
    matchLabels:
      type: front-end
  template:
    metadata:
      labels:
        app: myApp
        type: front-end
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
```

📝 Notes:

- `matchLabels` must match the Pod's metadata labels.
- The selector helps the ReplicaSet identify and manage existing or new Pods.

---

### 🛠 RS Commands

```bash
# Create ReplicaSet
kubectl apply -f rs-definition.yaml

# List ReplicaSets
kubectl get rs

# Describe ReplicaSet
kubectl describe rs myapp-rs
```

---

## 🏷️ Labels and Selectors

- Labels are key-value pairs assigned to Pods.
- ReplicaSets use **selectors** to find which Pods they should manage.
- Example:

```yaml
selector:
  matchLabels:
    type: front-end
```

- If matching Pods exist, the ReplicaSet adopts them.
- If not, it creates new Pods using the template.

---

## 🔁 Updating Replica Count

You can scale the number of Pods in two main ways:

---

### 📄 Option 1: Update YAML

Edit `replicas` value in the YAML:

```yaml
replicas: 6
```

Then reapply:

```bash
kubectl replace -f rs-definition.yaml
```

---

### 🧪 Option 2: Use `kubectl scale`

```bash
# Scale using the YAML file
kubectl scale --replicas=6 -f rs-definition.yaml

# Or scale by name
kubectl scale --replicas=6 replicaset myapp-rs
```

---

## 📌 Summary Commands

```bash
# Apply a resource from YAML
kubectl apply -f <file>.yaml

# Create an RC (deprecated)
kubectl create -f rc-definition.yaml

# Create an RS
kubectl apply -f rs-definition.yaml

# Get all ReplicaSets
kubectl get rs

# Get all ReplicationControllers
kubectl get rc

# Describe RS or RC
kubectl describe rs <rs-name>
kubectl describe rc <rc-name>

# Scale RS
kubectl scale --replicas=N replicaset <rs-name>
```

