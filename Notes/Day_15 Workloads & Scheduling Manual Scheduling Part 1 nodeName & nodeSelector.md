# Day 15 -[Workloads & Scheduling]: Manual Scheduling (Part 1): nodeName & nodeSelector

## Today's goal
Use `nodeName` and `nodeSelector` to schedule Pod to a specific Node

We have talked about the Master Node (control plane) before `kube-scheduler`. Its job is to allocate resources in the cluster, for example, it will automatically arrange which Node to place the Pod on.

In the "Storage" section, we used `spec.nodeName` fields in the YAML file to specify the node to schedule the Pod to a node with a hostPath. This manual configuration, without kube-scheduler scheduling, is called "manual scheduling."

Generally speaking, there are four main methods of Manula Scheduling:

- **nodeName**
- **nodeSelector**
- **Affinity**
- **Taint and Toleration**
  
Today, let's take a look at the first two methods: nodeName and nodeSelector.

> If you have a single-node cluster, the operation will also succeed, but the scheduling effect will be less noticeable. You can follow along by setting up an environment with at least two nodes on killercoda or Play with Kubernetes . This will give you a more noticeable effect.

  **nodeName**

Let's pause kube-scheduler and deploy a Pod to see what happens:

- Pause kube-scheduler:
  
 <pre>
  mv /etc/kubernetes/manifests/kube-scheduler.yaml ~/  </pre>

> This will pause the kube-scheduler. The principle will be introduced after discussing the concept of "Static Pod" in this chapter.

* Deploy a Pod:
<pre>kubectl run test --image nginx
</pre>

- You can see that the Pod is in the "Pending" state because it lacks kube-scheduler scheduling and we have not manually specified the Pod's destination:
  
<pre>kubectl get po test </pre>


<pre>NAME   READY   STATUS    RESTARTS   AGE
test   0/1     Pending   0          16s </pre>

At this point, we can use nodeName to manually schedule. As the name suggests, nodeName directly specifies the name of the Node and lets the Pod execute on that Node. It is quite simple and crude:

<pre>
# test-nodename.yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-nodename
spec:
  containers:
  - name: nginx
    image: nginx
  nodeName: node01 
</pre>

<pre>
kubectl apply -f test-nodename.yaml
</pre>

After creating a Pod, you can find that even without a scheduler, we can specify the destination of the Node through manual scheduling:

<pre>
kubectl get po test test-nodename -o wide
</pre>

<pre>
NAME            READY   STATUS    RESTARTS   AGE     IP            NODE     NOMINATED NODE   READINESS GATES
test            0/1     Pending   0          5m49s   <none>        <none>   <none>           <none>
test-nodename   1/1     Running   0          91s     192.168.1.4   node01   <none>           <none>
</pre>

> You can also use jsonpath to view the Node where the Pod is located:

<pre>
kubectl get po test-nodename -o jsonpath="{.spec.nodeName}{'\n'}"
</pre>

When using nodeName, please note the following:

- If the specified Node does not exist, the Pod cannot execute normally and may be automatically deleted.
  
- If the specified Node does not have enough resources to accommodate the Pod, the Pod will fail to execute.

<br>
  
  So when using nodeName, pay special attention to the status of the Node, such as whether the resources on the Node are sufficient, and avoid typos.

  - Now restart kube-scheduler:

<pre>
 mv ~/kube-scheduler.yaml /etc/kubernetes/manifests/
</pre>


- After a while, you can find that the previously pending test Pod has been successfully executed and automatically scheduled to node01:

<br>
<pre>
kubectl get po test -o wide
</pre>

<pre>
NAME   READY   STATUS    RESTARTS   AGE   IP            NODE     NOMINATED NODE   READINESS GATES
test   1/1     Running   0          11m   192.168.1.5   node01   <none>           <none>
</pre>

## nodeSelector
When you see Selector, do you think of something `Label`?

We can mark the Node with a special Label, and then specify the NodeSelector in the YAML to specify the destination of the Pod.

Compared to nodeName, nodeSelector has more flexibility, for example:

> Today, I just want to put the Pod on the Node with the Label "has=gpu". There may be several Nodes with this Label. If I use NodeSelector, I don't need to know the names of these Nodes, I just need to know that they have this Label.

But how do we add a label to a Node? To do this, we need to use the `kubectl label` command:

But how do we add a label to a Node? To do this, we need to use the “kubectl label” command:

The kubectl label instruction format is:

```bash
 kubectl label <object-type> <object-name> <label-key>=<label-value> 
 ```

- The command to remove a Label is:
```
kubectl label <object-type> <object-name> <label-key>-
```
For example, if I want to add a label to node01 `has=gpu`, I can issue the following command:

```
kubectl label node node01 has=gpu
```
After adding a Label to the Node, you can use the nodeSelector in the Pod yaml:

```
# need-gpu.yaml
apiVersion: v1
kind: Pod
metadata:
  name: need-gpu
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    has: gpu
```
After creating the Pod, check whether it meets the requirements:

```
kubectl apply -f need-gpu.yaml
```
```
kubectl get po need-gpu -o wide
NAME       READY   STATUS    RESTARTS   AGE   IP            NODE     NOMINATED NODE   READINESS GATES
need-gpu   1/1     Running   0          15s   192.168.1.4   node01   <none>           <none>
```
## Today's Summary
Today we introduced two manual scheduling methods: nodeNameand nodeSelector. The former directly specifies the node name, while the latter requires a combination of a label and a nodeSelector. Tomorrow we'll look at the last two methods: Affinity and Taint.

## References

[Assign Pods to Nodes](https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes/)