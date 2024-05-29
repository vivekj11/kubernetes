# EmptyDir Volume

EmptyDir volume is created when the Pod is assigned to a node. When a Pod is removed from a node for any reason, the data in the emptyDir is deleted permanently.

Since emptyDir volume is ephemaral in nature, we do not need to create any persistent volume for it. 


1. Create a pod that uses the emptyDir volume

```
vi emptydir-pod.yml
```
```
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-pod
spec:
  containers:
  - image: nginx
    name: emptydir-pod
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
```

2. Go inside the pod and check the cache folder.

```
kubectl exec -it <podname> -- bash 

ls -l /cache
```

