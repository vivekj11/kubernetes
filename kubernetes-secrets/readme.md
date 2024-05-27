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
        

        kubectl apply -f secret.yml

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


        kubectl apply -f secretpod1.yml

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

        kubectl apply -f secretpod2.yml             

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


        kubectl apply -f secretpod3.yml        
 
    Now we can check if the secret file is created inside the pod using following command

        kubectl exec -it secret3 -- ls -l /private
        # this will show a file "secret.txt" available inside the pod.

    We can also read the content of the file.

        kubectl exec -it secret3 -- cat /private/secret.txt
        # this will show the content of  "secret.txt".



## Docker Pull Secrets

Another common scenario when working with Kubernetes secrets is authenticating with a private Docker registry. Often, we need to create pods using private images, and to pull these images, we must authenticate with the repository.


### Creating sample pod with private image

1. I have already pushed one nginx image to my private docker hub registry. Let's try to create a pod with this priavte docker image.

        vi privateimage-pod.yml

        apiVersion: v1
        kind: Pod
        metadata:
          name: priavteimage-pod
        spec:
          containers:
          - name: priavteimage-pod
            image: vivekjcloud/secret-test:nginx
            ports:
            - containerPort: 80


    Note the image here, its a private image available in my docker hub repository.


2. If you check the pod status, it must be showing `ErrImagePull` status.

3. If you describe the pod, there you can see events such as- 

    _Failed to pull image "vivekjcloud/secret-test:nginx": Error response from daemon: pull access denied for vivekjcloud/secret-test, repository does not exist or may require 'docker login': denied: requested access to the resource is denied_ 


### Creating docker registry secret

As we noticed above, the pod is in error state as it is not able to pull the docker image. Now its time to create our docker registry secret.

  We need following details to create a registry secret in kubernetes
  - `docker-server`: your registry URL such as https://hub.docker.io
  - `docker-username`: docker registry authentication user
  - `docker-password`: docker registry authentication password
  - `docker-email`: your email address


        kubectl create secret docker-registry my-docker-secret --docker-server=DOCKER_REGISTRY_SERVER --docker-username=DOCKER_USER --docker-password=DOCKER_PASSWORD --docker-email=DOCKER_EMAIL

        # For this setup, I have used this command

        kubectl create secret docker-registry my-docker-secret --docker-server=https://hub.docker.com --docker-username='vivekjcloud' --docker-password='mytopsecret' --docker-email=myemail@gmail.com

Once created, you can list the secrets and check the type of this secret. It should be kubernetes.io/dockerconfigjson.

### Define pull secret in pod YAML 

Now its time to specify the pull secret in the pod yaml file. This way, the kubelet would be able to authenticate to the registry and pull the image to create the required pod.


        vi privateimage-pod.yml

        apiVersion: v1
        kind: Pod
        metadata:
          name: priavteimage-pod
        spec:
          containers:
          - name: priavteimage-pod
            image: vivekjcloud/secret-test:nginx
            ports:
            - containerPort: 80



