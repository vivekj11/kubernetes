# Namespaces

Namespaces provide a mechanism for isolating groups of resources within a single cluster.
It is a very good way to seggregate the work/resources based on different environments or users.  

Consider our house as a Kubernetes cluster and the rooms within it as namespaces. Just as rooms in a house are isolated and serve different purposes, such as a bedroom, kitchen, or living room, namespaces in Kubernetes are used to isolate and organize resources for different purposes.

<img src="images/namespace.png" alt="Before image">

### Not all objects are in a namespace 

Some common namespaced resources in kubernetes- pod, deployment, services etc.

```
# To see the resources that are in namespace
kubectl api-resources --namespaced=true

# To see the resources that are not in namespace
kubectl api-resources --namespaced=false

```


Kubernetes cluster starts with four initial namespaces:

- `default`: This is where all our resources would be created if we do not specifically define one.
- `kube-node-lease`: This namespace holds Lease objects associated with each node. (Lease provide a mechanism to lock shared resources and coordinate activity between members of a set.)
- `kube-public`: This namespace is readable by all clients (including those not authenticated). This namespace is mostly reserved for cluster usage, in case that some resources should be visible and readable publicly throughout the whole cluster.
- `kube-system`: The namespace for objects created by the Kubernetes system.


## Working with namespaces

While working with namespaces, we can use the word namespace or ns in short. 

1. Listing all the available namespaces

```
kubectl get namespaces 
```

2. Work on resources in a particular namespace. 

```
# To list the deployment in a namespace.  
kubectl get deployment -n <namespace name>

# To create a pod in a namespace
kubectl run nginx --image=nginx -n <namespace name>
```

3. Creating a namespace 

```
kubectl create namespace <namespace name>
```

4. Deleting a namespace

```
kubectl delete namespace <namespace name>

Note- When you delete a namespace, all the resources inside it would also be deleted.
```