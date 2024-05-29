# Kubernetes ConfigMap and Usage

A ConfigMap is an API object used to store non-confidential data in key-value pairs. Pods can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a volume.

ConfigMap does not provide secrecy or encryption. If the data you want to store are confidential, use a Secret rather than a ConfigMap.


## Creating ConfigMaps

We can create configmaps directly using the kubectl command or using YAML files.

1. Create a configMap using literal values.
```
kubectl create configmap my-configmap1 --from-literal=key1=config1 --from-literal=key2=config2
```

Same ConfigMap creation using the yaml file-

```
vi configmap.yml
```
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-configmap1
data:
  key1: value1
  key2: value2
```
```
kubectl apply -f configmap.yml
```

2. Create a configmap using a file. 

```
# Create a file with some key value pairs in it

vi myconfig.txt

name=vivek
domain=cloud
year=2024
```
```
# Now create the configmap using this file.

kubectl create cm my-configmap2 --from-file=myconfig.txt  
```

## Mounting the above ConfigMaps in a Pod  


1. We are going to attach the configmap as an environemnt variable inside the pod.

```
vi cm-pod1.yml
```
```
apiVersion: v1
kind: Pod
metadata:
  name: cm-pod1
spec:
  containers:
  - name: cm-pod1
    image: nginx
    ports:
    - containerPort: 80
    envFrom:
    - configMapRef:
        name: my-configmap1
```
```
kubectl apply -f cm-pod1.yml
```

2. Mount the configmap as a file inside pod:-  So far, we have been adding the configmap as environment variables inside the pod. In this scenario, we will mount the configmap file inside the pod using volumes and volumeMounts.

```
vi cm-pod2.yml
```
``` 
apiVersion: v1
kind: Pod
metadata:
  name: cm-pod2
spec:
  containers:
  - name: cm-pod2
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: myconfig
      mountPath: /config
  volumes:
  - name: myconfig 
    configMap:
      name: my-configmap2
```
```
kubectl apply -f m-pod2.yml              
```


## Real Time Scenario-1

**Objective**: To create an Nginx pod accessible on port 8080 instead of the default port 80, provisioning should be done using a Deployment.

1. Create an nginx deployment in kubenetes cluster.

```
vi example.yml
```
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
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
        image: nginx:latest
        ports:
        - containerPort: 80
```

2. Once the pod is created, go inside the pod and check the nginx default configuration file. Notice the port number is set to 80.

```
kubectl exec -it <podname> -- bash

cd /etc/nginx/conf.d
cat default.conf

***you can test the connectivity inside the pod by installing the curl command***

apt-get update
apt-get install curl

curl localhost:80

```

3. You can also try port-forward the pod request so that you can access it on your local machine port for testing purpose. 

```
kubectl port-forward <podname> -- 9000:80
```

4. Head to your browser and access localhost:9000, you will see the nginx welcome page. 

5. Now, let's start the actual change by modifying our Nginx configuration to run on port 8080 instead of port 80. Go inside the pod again and copy the content of the /etc/nginx/conf.d/default.conf file. (You can also find this content directly on the internet.)

```
kubectl exec -it <podname> -- bash

cat /etc/nginx/conf.d/default.conf
```

6. Create a file on your local machine and paste the copied content inside it. Make sure to put the file extenstion as .conf. (for eg- mynginx.conf)

7. Change the listen port number from 80 to 8080 in this file and save it.

8. Create a configmap with this modified configuration file.

```
kubectl create cm nginx-configmap --from-file=mynginx.conf
```

9. Now modify the content of our deployment file.

```
vi example.yml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
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
        image: nginx:latest
        ports:
        - containerPort: 8080 
        volumeMounts:
        - name: nginxconfig
          mountPath: /etc/nginx/conf.d/
      volumes:
      - name: nginxconfig
        configMap:
          name: nginx-configmap        
```

10. Check the connectivity by going inside the pod again

```
kubectl exec -it <podname> -- bash

apt-get update
apt-get install curl

curl localhost:80 --> Will give error respose
curl localhost:8080 --> you will get the response here.
```

11. You can also try port forward the pod request so that you can access it on your local machine port for testing purpose. 

```
kubectl port-forward <podname> -- 9000:8080
```

12. Head to your browser and access localhost:9000, you will see the nginx welcome page. 

