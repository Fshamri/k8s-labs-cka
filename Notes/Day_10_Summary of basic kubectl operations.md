# Day 10 - [Basic Concept]: Summary of basic kubectl operations

# Today's Goal

## Common kubectl Commands

### get & describe
- Pod  
- Deployment  
- Rollout history & Rollback  
- Service  

### Tips for using kubectl
- Object Resource Abbreviation  
- Quickly generate YAML samples  
- Forgot command syntax? Use `-h`  
- Force deletion and re-establishment (`replace`)  
- Modify existing objects (Part 1): `kubectl edit`  
- Modify existing objects (Part 2): `kubectl patch`  

---

## Intro

Today is the **penultimate article** of the "Basic Concept" chapter.

In the previous few days, we introduced the basic objects in Kubernetes. A large number of operations are completed through the `kubectl` command, so it is best for administrators to use it quickly and proficiently.

Today, we'll go over the commonly used instructions and techniques related to:
- Pod
- Deployment
- Service
- Namespace

---

## Reminder: Namespace

To specify the namespace in kubectl, use the `-n` flag:

```bash
kubectl -n kube-system get po
