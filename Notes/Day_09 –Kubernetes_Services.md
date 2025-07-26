
# Day 09 – **Basic Concept: Service**

## 📌 Today's Goal

- Understand the **purpose of Service**
- Learn **Types of Services**:
  - ClusterIP
  - NodePort
  - LoadBalancer
  - ExternalName
  - Multi-Port & Headless Services
- Practice with `kubectl port-forward`

---

## 🤔 Why Do We Need Service?

On **Day 03**, we used **Flannel** or **Calico** as CNI to establish internal cluster networking.

When a Pod is created, it gets a virtual IP. However, this IP has two key issues:

1. **Short lifecycle** – Pod restarts lead to IP changes.
2. **Only accessible inside the cluster**.

➡️ To solve this, Kubernetes provides **Service**.

---

## 🚪 What Is a Service?

A **Service** provides a stable **IP** and **DNS name** to access a group of Pods:

```
Client <--> Service IP/DNS <--> kube-proxy <--> Pod(s)
```

- Service hides Pod IP changes.
- Supports multiple Pods behind one access point.
- Performs load balancing.

---

## 🔗 Endpoint

When a Service is created, Kubernetes also creates an **Endpoint**:

```
User <--> Service <--> Endpoint <--> Pod
```

Service uses **Label Selectors** to target the Pods.

---

## 🧭 Service DNS

Service DNS format in a cluster:

```
<service-name>.<namespace>.svc.cluster.local
```

Example:
```bash
kubectl exec -it <pod-name> -n <other-namespace> -- curl web-service.dev.svc.cluster.local
```

---

## ⚙️ Types of Services

1. ClusterIP
2. NodePort
3. LoadBalancer
4. ExternalName

---

### 🔹 ClusterIP

- Default type
- Internal-only access
- Common for internal microservice communication

---

### 🔹 NodePort

- Exposes a port on each node (30000–32767)
- Access with `NodeIP:NodePort`

**Terminology:**

- `NodePort`: Port exposed on the node (e.g., 30010)
- `Port`: Port the service listens on (e.g., 80)
- `TargetPort`: Port on the Pod (e.g., 80)

---

### 🔹 ExternalName

- Points to an **external DNS name** instead of Pods
- Useful for accessing external services like databases

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
```

---

### 🔹 LoadBalancer

- Used in cloud environments (e.g., AWS, GCP)
- Automatically provisions an external load balancer

---

## 🔧 Create a Service for a Deployment

```bash
kubectl create deploy nginx-deploy --image=nginx --port=80 --replicas=3
kubectl get pod --show-labels | grep nginx
```

Label will be: `app=nginx-deploy`

### nginx-deploy-svc.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-deploy-svc
spec:
  selector:
    app: nginx-deploy
  ports:
  - port: 80
    targetPort: 80
```

```bash
kubectl apply -f nginx-deploy-svc.yaml
kubectl describe service nginx-deploy-svc
kubectl get endpoints nginx-deploy-svc
```

Verify Pod IPs:
```bash
kubectl get pods -l app=nginx-deploy -o jsonpath='{.items[*].status.podIP}'
```

---

## ✅ Test the Service

```bash
export SERVICE_IP=$(kubectl get svc nginx-deploy-svc -o jsonpath='{.spec.clusterIP}')
curl $SERVICE_IP:80 --max-time 2
```

Access from another namespace:
```bash
kubectl run test --image nginx -n kube-system -- curl nginx-deploy-svc.default.svc.cluster.local
kubectl logs -n kube-system test
```

---

## 🌍 NodePort for External Access

Edit `nginx-deploy-svc.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-deploy-svc
spec:
  type: NodePort
  selector:
    app: nginx-deploy
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30010
```

```bash
kubectl apply -f nginx-deploy-svc.yaml
curl localhost:30010 --max-time 2
```

---

## ⚙️ Create Services via CLI

```bash
kubectl expose deploy nginx-deploy --type=NodePort --name=nginx-deploy-svc --port=80 --target-port=80
kubectl create service nodeport my-service --tcp=80:80 --node-port=30010
```

---

## 🔁 Multi-Port Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
  - name: http
    port: 80
    targetPort: 9376
  - name: https
    port: 443
    targetPort: 9377
```

---

## 🧠 Headless Service

- No ClusterIP
- Returns Pod IPs directly (used with StatefulSet)

### headless-demo.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  clusterIP: None
  selector:
    app: nginx
  ports:
  - port: 80
    name: web
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx
spec:
  serviceName: nginx
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          name: web
```

```bash
kubectl apply -f headless-demo.yaml
kubectl get ep nginx
kubectl run test2 --image busybox:1.28 --restart=Never -- nslookup nginx.default.svc.cluster.local
kubectl logs test2
```

---

## 🔄 `kubectl port-forward`

Allows local access to Pods or Services without exposing NodePorts.

```bash
kubectl port-forward pod/nginx 8080:80
kubectl port-forward svc/nginx-svc 8080:80
curl localhost:8080
```

Use `Ctrl + C` to stop.

---

## 📚 Summary

- Services provide stable network identities for Pods
- Types: ClusterIP, NodePort, LoadBalancer, ExternalName
- Special types: Multi-port and Headless Services
- Covered DNS, Endpoints, and `kubectl port-forward`

---

## 🔗 References

- [Kubernetes Services – Official Docs](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Day 9 – Services on ITHelp](https://ithelp.ithome.com.tw/)
- Kubernetes Services by Vladimir Romanov
- Kubernetes Services: Definitions & Examples (2023)
