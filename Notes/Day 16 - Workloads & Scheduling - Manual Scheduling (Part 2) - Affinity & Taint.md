# Day 16 -[Workloads & Scheduling]: Manual Scheduling (Part 2): Affinity & Taint

## Today's goal

* Understand and use **Affinity** and **Anti-affinity** to schedule Pods

  * Node affinity
  *  Inter-pod **affinity** / **anti-affinity**
  
* Understanding and Using **Taint** & **Tolerations**

* Using **Affinity** with **Taint**

If you use labels to specify Pod destinations, the nodeSelector introduced in the previous chapter is the simplest method. However, if you want to add more candidates to the node filtering criteria or have more flexible filtering requirements, the nodeSelector cannot achieve this.

Today we will look at another **more flexible** approach, namely **Affinity/Anti-affinity**.

## Affinity/Anti-affinity
From the perspective of Pod Scheduling, nodes with higher affinity are more likely to be selected to execute a pod. So which nodes are considered to have affinity? You can filter by using the "Node or Pod Label":

* **Node affinity** : Filter out specific nodes through the "Node Label" and schedule Pods to qualified nodes.

* **Inter-pod affinity**: Filters specific Pods by "Pod Label" and then checks whether these specific Pods exist in the specified Node topology. If they do, the new Pod will be scheduled to run in that Node topology.

* **Inter-pod anti-affinity**: Filter out specific Pods through the "Pod Label", and then check whether these specific Pods exist in the specified Node topology. If they exist, the new Pods will be placed in other Node topologies for execution.
  
<br>

> What is Node topology? Please refer to the Appendix .
