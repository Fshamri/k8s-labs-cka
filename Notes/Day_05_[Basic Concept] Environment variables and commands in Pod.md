# Day 05 - [Basic Concept]: Environment variables and commands in Pod

## Today's goal
* Setting Pod environment variables

* Set the Pod's command and arguments

## Environment variables in Pods
in many application scenarios, "environment variables" are a very common setting. Environment variables usually consist of a key-value pair, such as USER=root.

Then in the Pod yaml, you can define environment variables under "spec.containers[].env[]":

<pre># env-demo.yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-demo
spec:
  containers:
  - command: ["sleep", "300"]
    env:   # environment variables
    - name: USER     # key
      value: michael # value
    image: busybox
    name: busybox
</pre>

<pre>kubectl apply -f env-demo.yaml</pre>
After creating the Pod, test to see if the environment variables have been set successfully:

<pre>kubectl exec -it env-demo -- /bin/sh
</pre>