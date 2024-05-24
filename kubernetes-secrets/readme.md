# Kubernetes Secrets and Usage

A Secret is an object that contains a small amount of sensitive data such as a password, a token, or a key. 

Because Secrets can be created independently of the Pods that use them, there is less risk of the Secret (and its data) being exposed during the workflow of creating, viewing, and editing Pods

Secrets are similar to ConfigMaps but are specifically intended to hold confidential data.


### Uses for Secrets

1. Set environment variables for a container.
2. Provide credentials such as SSH keys or passwords to Pods.
3. Allow the kubelet to pull container images from private registries.


### Types of Secrets


| Build-in Type | Usage |
| --- | --- |
| Opaque        |   arbitrary user-defined data  |
| kubernetes.io/dockerconfigjson  |   serialized ~/.docker/config.json file   |
| kubernetes.io/tls    |  data for a TLS client or server  |

There are some other types of secrets available that can be used in different scenarios. 
Refer- https://kubernetes.io/docs/concepts/configuration/secret/#secret-types 


## Creating Secrets

We can create secret directly using the kubectl command or using YAML files.

1. Create a secret using literal values.

        kubectl create secret generic my-secret1 --from-literal=username=myuser --from-literal=password=topsecret


    Same secret creation using the yaml file-

        First you should encode the sensitive values. Run below commands in any bash terminal.

        echo "myuser" | base64 
        echo "topsecret" | base64 

        Now, create the file and put the encoded values 

        vi secret.yml

        apiVersion: v1
        kind: Secret
        type: Opaque
        metadata:
          name: my-secret1
        data:
          password: <encoded value that we get above>
          username: <encoded value that we get above>
        

2. Create a secret using a file. 

        echo "topsecret file that want to keep in pod" > secret.txt 

        kubectl create secret generic my-secret2 --from-file=secret.txt


        
## Mounting the above secrets in a Pod


1. Attaching the secret as an environment variable

    We are going to attach the secret as an environemnt variable inside the pod. 

        vi secretpod1.yml

        apiVersion: v1
        kind: Pod
        metadata:
          name: secret1
        spec:
          containers:
          - name: secret1
            image: nginx
            ports:
            - containerPort: 80
            envFrom:
            - secretRef:
                name: my-secret1


    This pod is referencing 'my-secret1' and adding those secrets as environment variables in the pod.

    We can check the environment variables inside the pod using the following command:

        kubectl exec -it secret1 -- env



2. Reference only specific secret

    Our 'my-secret1' contains two secret values, both of which were added as environment variables in our previous setup. Now, in this case, we are going to set only the password environment variable inside the pod.


        vi secretpod2.yml

        apiVersion: v1
        kind: Pod
        metadata:
          name: secret2
        spec:
          containers:
          - name: secret2
            image: nginx
            ports:
            - containerPort: 80
            env:
            - name: MY_PASSWORD
              valueFrom:
                secretKeyRef:
                    name: my-secret1
                    key: password    

    check the variable inside the pod using following command

        kubectl exec -it secret2 -- env

    you will be able to see only one environment variable i.e. the password variable.


3. Mount the secret as a file inside pod

    So far, we have been adding the secrets as environment variables inside the pod. In this scenario, we will mount the secret file inside the pod using volumes and volumeMounts.


        vi secretpod3.yml

        apiVersion: v1
        kind: Pod
        metadata:
          name: secret3
        spec:
          containers:
          - name: secret3
            image: nginx
            ports:
            - containerPort: 80
            volumeMounts:
            - name: mytestsecret
              mountPath: /private
          volumes:
          - name: mytestsecret 
            secret:
              secretName: my-secret2   
 
    Now we can check if the secret file is created inside the pod using following command

        kubectl exec -it secret3 -- ls -l /private
        # this will show a file "secret.txt" available inside the pod.

    We can also read the content of the file.

        kubectl exec -it secret3 -- cat /private/secret.txt
        # this will show the content of  "secret.txt".







