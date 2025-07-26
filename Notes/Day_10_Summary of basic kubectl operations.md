# Day 10 - [Basic Concept]: Summary of basic kubectl operations

# Today's Goal  
**Common kubectl commands**

- get & describe  
- Pod  
- Deployment  
- Rollout history & Rollback  
- Service  
- Tips for using kubectl  

---

## Object Resource Abbreviation

| Object Type | Abbreviation |
| ----------- | ------------ |
| pod         | po           |
| deployment  | deploy       |
| service     | svc          |
| namespace   | ns           |

Use this command to see all:
```bash
kubectl api-resources
```

---

## Commonly Used Commands: `get` & `describe`

List everything:
```bash
kubectl get all
```

List specific type:
```bash
kubectl get <object-type>
```

List one object:
```bash
kubectl get <object-type> <object-name>
```

List in all namespaces:
```bash
kubectl get <object-type> -A
kubectl get all -A
```

Count:
```bash
kubectl get <object-type> | wc -l
kubectl get <object-type> --no-headers | wc -l
```

YAML output:
```bash
kubectl get <object-type> <object-name> -o yaml
```

Show labels:
```bash
kubectl get <object-type> --show-labels
```

Filter by label:
```bash
kubectl get <object-type> -l <key=value>
kubectl get <object-type> -l key1=value1,key2=value2
```

JsonPath:
```bash
kubectl get <object-type> -o jsonpath=<jsonpath-expression>
```

Describe:
```bash
kubectl describe <object-type> <object-name>
```

---

## Pod Commands

Create:
```bash
kubectl run <pod-name> --image <image-name>
kubectl run <pod-name> --image <image-name> --port <port-number>
kubectl run <pod-name> --image <image-name> --port <port> --expose
kubectl run <pod-name> --image <image-name> -l key=value
```

Update image:
```bash
kubectl set image pod <pod-name> <container-name>=<new-image-name>
```

Logs:
```bash
kubectl logs <pod-name>
kubectl logs -f <pod-name>
kubectl logs <pod-name> -c <container-name>
```

Pod IP:
```bash
kubectl get pod <pod-name> -o wide
kubectl get pod <pod-name> -o jsonpath='{.status.podIP}'
```

Copy files:
```bash
kubectl cp <local-path> <pod-name>:<pod-path>
kubectl cp <pod-name>:<pod-path> <local-path>
```

Access shell:
```bash
kubectl exec -it <pod-name> -- /bin/sh
kubectl exec <pod-name> -- <command>
```

Check node:
```bash
kubectl get pod <pod-name> -o wide
kubectl get pod <pod-name> -o jsonpath='{.spec.nodeName}'
```

---

## Deployment Commands

Create:
```bash
kubectl create deploy <name> --image <image> --replicas <num>
kubectl create deploy <name> --image <image> --port <port>
```

Scale:
```bash
kubectl scale deploy <name> --replicas <num>
```

Update image:
```bash
kubectl set image deploy <name> <container-name>=<image>
```

---

## Rollout History & Rollback

History:
```bash
kubectl rollout history deploy <name>
kubectl rollout history deploy <name> --revision=<num>
```

Undo:
```bash
kubectl rollout undo deploy <name>
kubectl rollout undo deploy <name> --to-revision=<num>
```

Annotate:
```bash
kubectl annotate deploy <name> kubernetes.io/change-cause="your reason"
```

Restart:
```bash
kubectl rollout restart deploy <name>
```

Status:
```bash
kubectl rollout status deploy <name>
```

---

## Service Commands

Expose:
```bash
kubectl expose <type> <name> --port <port> --target-port <target> --name <svc-name> --type <svc-type>
```

Create:
```bash
kubectl create service <type> <name> --tcp <port>:<target>
kubectl create service nodeport my-svc --tcp=80:80 --node-port=30010
```

Logs:
```bash
kubectl logs svc/<service-name>
```

---

## Tips for `kubectl`

### Generate YAML
```bash
kubectl create deploy nginx --image nginx --dry-run=client -o yaml
kubectl expose deploy nginx --port=80 --dry-run=client -o yaml
```

### Get Help
```bash
kubectl -h
kubectl run -h
kubectl create -h
```

### Force Delete
```bash
kubectl delete <type> <name> --force --grace-period=0
```

### Replace
```bash
kubectl replace -f new.yaml --force --grace-period=0
```

---

## Modify Existing Objects

### `kubectl edit`
```bash
kubectl edit deploy nginx
kubectl edit po nginx
```

Update image in YAML:
```yaml
- name: nginx
  image: nginx:1.19.1
```

Update CPU requests:
```yaml
resources:
  requests:
    cpu: "700m"
```

If it fails, reapply:
```bash
kubectl replace -f /tmp/kubectl-edit-xxxx.yaml --force --grace-period=0
```

---

## `kubectl patch`

Syntax:
```bash
kubectl patch <type> <name> -p <json-patch>
```

Example:
```bash
kubectl patch po nginx -p '{"spec":{"containers":[{"name":"nginx","image":"nginx:1.19.1"}]}}'
```

Dry run:
```bash
kubectl patch po nginx -p '{"spec":{"containers":[{"name":"nginx","image":"nginx:1.19.1"}]}}' --dry-run=client -o yaml
```

---

## Summary

Practice these commands to build muscle memory. Use `-h` and [k8s.io](https://k8s.io) for reference.

---

## References

- [Kubernetes kubectl cheatsheet](https://kubernetes.io/docs/reference/kubectl/quick-reference/)
- [k8s.io](https://kubernetes.io/docs/reference/kubectl/)
