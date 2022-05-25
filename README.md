# Kubernetes-Resource-Requests-limits

Requests and limits are the mechanisms Kubernetes uses to control resources such as CPU and memory. Requests are what the container is guaranteed to get. If a container requests a resource, Kubernetes will only schedule it on a node that can give it that resource. Limits, on the other hand, make sure a container never goes above a certain value. The container is only allowed to go up to the limit, and then it is restricted.
It is important to remember that the limit can never be lower than the request. If you try this, Kubernetes will throw an error and won’t let you run the container.

Requests and limits are on a per-container basis. While Pods usually contain a single container, it’s common to see Pods with multiple containers as well. Each container in the Pod gets its own individual limit and request, but because Pods are always scheduled as a group, you need to add the limits and requests for each container together to get an aggregate value for the Pod.

## Container settings
There are two types of resources: CPU and Memory. The Kubernetes scheduler uses these to figure out where to run your pods.
A typical Pod spec for resources might look something like this. This pod has two containers:
```
containers:
  - name: container1
    image: busybox
    resources:
      requests:
        memory: "32Mi"
        cpu: "200m"
      limits:
        memory: "64Mi"
        cpu: "250m"
  - name: container2
    image: busybox
    resources:
      requests:
        memory: "96Mi"
        cpu: "300m"
      limits:
        memory: "192Mi"
        cpu: "750m"
```

Each container in the Pod can set its own requests and limits, and these are all additive. So in the above example, the Pod has a total request of 500 mCPU and 128 MiB of memory, and a total limit of 1 CPU and 256MiB of memory.

## CPU
CPU resources are defined in millicores. If your container needs two full cores to run, you would put the value “2000m”. If your container only needs ¼ of a core, you would put a value of “250m”.

One thing to keep in mind about CPU requests is that if you put in a value larger than the core count of your biggest node, your pod will never be scheduled. Let’s say you have a pod that needs four cores, but your Kubernetes cluster is comprised of dual core VMs—your pod will never be scheduled!

Unless your app is specifically designed to take advantage of multiple cores (scientific computing and some databases come to mind), it is usually a best practice to keep the CPU request at ‘1’ or below, and run more replicas to scale it out. This gives the system more flexibility and reliability.

## Memory
Memory resources are defined in bytes. Normally, you give a mebibyte value for memory (this is basically the same thing as a megabyte), but you can give anything from bytes to petabytes.

Just like CPU, if you put in a memory request that is larger than the amount of memory on your nodes, the pod will never be scheduled.

## Nodes
It is important to remember that you cannot set requests that are larger than resources provided by your nodes. For example, if you have a cluster of dual-core machines, a Pod with a request of 2.5 cores will never be scheduled! You can find the total resources for Kubernetes Engine VMs here:
```
kubectl describe nodes server-1 | grep -i "allocatable"
```

# Namespace Settings
In an ideal world, Kubernetes’ Container settings would be good enough to take care of everything, but the world is a dark and terrible place. People can easily forget to set the resources, or a rogue team can set the requests and limits very high and take up more than their fair share of the cluster.
To prevent these scenarios, you can set up ResourceQuotas and LimitRanges at the Namespace level.

## ResourceQuotas
After creating Namespaces, you can lock them down using ResourceQuotas. ResourceQuotas are very powerful, but let’s just look at how you can use them to restrict CPU and Memory resource usage.

A Quota for resources might look something like this:
```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: demo
spec:
  hard:
    requests.cpu: 500m
    requests.memory: 100Mib
    limits.cpu: 700m
    limits.memory: 500Mib
```
Looking at this example, you can see there are four sections. Configuring each of these sections is optional.

**requests.cpu** is the maximum combined CPU requests in millicores for all the containers in the Namespace. In the above example, you can have 50 containers with 10m requests, five containers with 100m requests, or even one container with a 500m request. As long as the total requested CPU in the Namespace is less than 500m!

**requests.memory** is the maximum combined Memory requests for all the containers in the Namespace. In the above example, you can have 50 containers with 2MiB requests, five containers with 20MiB CPU requests, or even a single container with a 100MiB request. As long as the total requested Memory in the Namespace is less than 100MiB!

**limits.cpu** is the maximum combined CPU limits for all the containers in the Namespace. It’s just like requests.cpu but for the limit.

**limits.memory** is the maximum combined Memory limits for all containers in the Namespace. It’s just like requests.memory but for the limit.
