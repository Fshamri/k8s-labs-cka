# Day 09 - *Basic Concept*: Service

## Today's Goal

- Purpose of Service
- Types of Services
  - ClusterIP
  - NodePort
  - LoadBalancer
  - ExternalName
  - Special Service: Multi-Port Service & Headless Service
- `kubectl port-forward`

---

## Why Do We Need Service?

In Day 03, we used Flannel or Calico as CNI to establish a virtual network for internal communication within the cluster.

When a Pod is created, it is assigned a virtual IP, but this virtual IP has two fatal flaws:

1. The life cycle of a Pod is short, and when a Pod is restarted, the IP will also change. It is obviously unrealistic to ask users to pay attention to IP changes at any time.
2. This virtual IP can only be used within the cluster and cannot be accessed from outside.

In order to solve these two problems, we need the help of **Service**.

---

## Service

The function of Service is to act as a proxy for Pod's external communication and serve as a unified interface for external access.

The Service will have its own IP and domain name. When a third party wants to access the Pod behind the Service, they only need to use the Service's IP or domain name, and `kube-proxy` will help forward the traffic to the Pod behind it. In this way, you can access the service through a stable Service IP without worrying about the Pod's IP changing due to restart.

In addition, it is not feasible to deploy a service with just one Pod, so we use a group of Pods (ReplicaSet, Deployment) to deploy services. A Service can not only proxy one Pod, but also a group of Pods.

![Service Example](https://ithelp.ithome.com.tw/upload/images/20240827/20168692OLUrXjxT7u.png)

---

## Supplement: Endpoint

Generally speaking, when K8s creates a Service, it also creates an Endpoint object to point to the Pod behind the Service:

```
User <--> Service <--> Endpoint <--> Pod
```

A Service identifies its Pods through Labels and Selectors.

---

## Service Domain Name

When you create a Service, K8s creates a DNS entry for internal access:

```
<service-name>.<namespace>.svc.cluster.local
```

Example:
```bash
kubectl exec -it <pod-name> -n <other-namespace> -- curl web-service.dev.svc.cluster.local
```

---

## Types of Services

According to different use cases, Services are divided into:

- ClusterIP
- NodePort
- LoadBalancer
- ExternalName

---

### ClusterIP

Default type. Only accessible within the cluster. Suitable for internal data communication.

---

### NodePort

Exposes a fixed port on each Node. External traffic accesses Pods through:

```
NodeIP:NodePort
```

![NodePort Example](https://ithelp.ithome.com.tw/upload/images/20240827/20168692llXDYys8zQ.png)

- **NodePort**: Port exposed on node (range: 30000–32767)
- **Port**: Service port (e.g., 80)
- **TargetPort**: Pod’s port

---

### ExternalName

Maps a Service to a DNS name.

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

### LoadBalancer

Used in cloud. External traffic is routed through a cloud LB like AWS ELB or GCP LB.

---

## Basic Implementation of Service

```bash
kubectl create deploy nginx-deploy --image nginx --port 80 --replicas 3
kubectl get pod --show-labels | grep nginx
```

**Label** is: `app=nginx-deploy`

Create YAML file:

```yaml
# nginx-deploy-svc.yaml
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

Apply:
```bash
kubectl apply -f nginx-deploy-svc.yaml
kubectl describe service nginx-deploy-svc
kubectl get endpoints nginx-deploy-svc
kubectl get pods -l app=nginx-deploy -o jsonpath='{.items[*].status.podIP}'
```

Test Service:
```bash
export SERVICE_IP=$(kubectl get svc nginx-deploy-svc -o jsonpath='{.spec.clusterIP}')
curl $SERVICE_IP:80 --max-time 2
```

From another namespace:
```bash
kubectl run test --image nginx -n kube-system -- curl nginx-deploy-svc.default.svc.cluster.local
kubectl logs -n kube-system test
```

---

## Expose to Outside with NodePort

Edit the YAML:
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

## Tips: Create Service via CLI

```bash
kubectl expose deploy nginx-deploy --type=NodePort --name=nginx-deploy-svc --port=80 --target-port=80
kubectl create service nodeport my-service --tcp=80:80 --node-port=30010
```

---

## Multi-Port Service

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
      protocol: TCP
      port: 80
      targetPort: 9376
    - name: https
      protocol: TCP
      port: 443
      targetPort: 9377
```

---

## Headless Service

```yaml
# headless-demo.yaml
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

## `kubectl port-forward`

```bash
kubectl run nginx --image=nginx --port=80
kubectl expose pod nginx --port=80 --target-port=80 --name=nginx-svc
kubectl port-forward pod/nginx 8080:80
# OR
kubectl port-forward svc/nginx-svc 8080:80
curl localhost:8080
```

---

## Today's Summary

We covered:
- Service concept and types
- ClusterIP, NodePort, LoadBalancer, ExternalName
- Multi-port and Headless Services
- DNS, Endpoints, and `kubectl port-forward`

---

## References

- [Service](https://kubernetes.io/docs/concepts/services-networking/service/)
- [[Day 9] Establish communication channels between external services and Pods - Services](https://ithelp.ithome.com.tw/articles/10194344)
- [Kubernetes Services - by Vladimir Romanov](https://www.kerno.io/learn/kubernetes-services#kubernetes-services-headless-services)
- [[Kubernetes Services: Definitions & Examples (2023)](https://kodekloud.com/blog/kubernetes-services/)
