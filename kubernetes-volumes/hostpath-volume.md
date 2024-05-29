
# HostPath Volume

A hostPath volume mounts a directory from the host node’s filesystem into our Pod. It means, we can use any folder in the node file system and mount it to our pod. 

Basically our pod is using the Node's storage.

1. Create a folder on your node and add a file in this folder.

```
mkdir pv-test
cd pv-test
echo "This is a testfile for HostPath volume" > testfile.txt
```

2. Create a persistent volume with hostPath type

```
vi hostpath-pv.yml
```
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: hostpath-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/home/vivekj/kubernetes/pv-test"
```
```
kubectl apply -f hostpath-pv.yml

kubectl get pv
```

3. Create a persistent volume claim to use this pv

```
vi hostpath-pvc.yml
```
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: hostpath-volume-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```
```
kubectl apply -f hostpath-pvc.yml

kubectl get pvc
```

4. Now its time to create a pod that would use this pvc

```
vi hostpath-pod.yml
```
```
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-pod
spec:
  containers:
    - name: hostpath-pod-container
      image: nginx
      ports:
      - containerPort: 80
      volumeMounts:
      - name: hostpath-volume
        mountPath: "/external-volume"          
  volumes:
  - name: hostpath-volume
    persistentVolumeClaim:
      claimName: hostpath-volume-claim
```
```
kubectl apply -f hostpath-pod.yml

kubectl get pods
```

5. Check the volume status inside the pod. You should be able to see the file that was craeted in host machine.

```
kubectl exec -it <podname> -- bash

cd /external-volume

ls -l

```