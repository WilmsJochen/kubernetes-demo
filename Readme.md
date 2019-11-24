# Ex2: Deploy your own docker container in the kubernetes cluster.

in this exercise, the goal is to deploy your previously created docker image in the kubernetes cluster and let them talk to each other over http requests.

## Checkout the git folder on your working directory.
1. Change your directory back to YOUR_NAME. 
    ```
    * YOUR_NAME
      * ex1
        * Gp
        * nodeJs
        * python
        * Readme
    ```
2. In this directory you can clone the second exercise repo.
    ```
    git clone https://github.com/WilmsJochen/ex2.git
    ```
3. Change directory to exercise 2.
    ```
    cd ex2/
    ```
   
## Sorts of kubernetes resources:
- Pod
- deployment
- service
- configmap
- persistence volume
- persistence volume claim
- statefullset
- job
- cronjob

We will touch all of these kubernetes resources in this exercise. You can find an example of how the resource is defined in the manifest folder. I invite you to first take a look on all these definitions.

## get to know some kubectl commands

Execute following commands and investigate their output. Fill in the Names with a name you found in the get command.

```
   kubectl get pod
   kubectl get OTHER_RESOURCE 
   kubectl describe pod POD_NAME
   kubectl describe OTHER_RESOURCE RESOURCE_NAME
   kubectl edit pod POD_NAME
   kubectl edit OTHER_RESOURCE RESOURCE_NAME
   kubectl logs POD_NAME       
```
## deploy your first pod with your docker image.

you can simply deploy a pod with the definition you found in the manifest. offcourse you need to adapt that definition with your own variables.
Editing the variables can be done with nano from the terminal. 

```
nano manifests/pod.yaml
```

**_TIP:_**  when you are ready editing, you exit nano with ctrl+x.

Now you can apply the pod to your cluster.
```
kubectl apply -f manifests/pod.yaml
```

Congratulations you have your own docker image running in a kubernetes cluster.
You can verify that the pod is running with `kubctl get pod`

## Make a deployment from you pod.

First we need to delete your pod from the cluster.
```
kubectl delete pod POD_NAME
```

Now we need to modify our deployment.yaml with our own variables.
```
nano manifests/deployment.yaml
```

Apply your deployment to the cluster.
```
kubectl apply -f manifests/deployment.yaml
```
Now you can play with the number of replicas you want to have of your deployment.
**_NOTE:_**  please scale only to 2 pods so we don't overload our cluster with a lot of pods.
```
kubectl scale deployment YOUR_NAME --replicas 2
```
And scale back to 1 pod after you have verified the scaling with `kubectl get pods`.
```
kubectl scale deployment YOUR_NAME --replicas 1
```

## Time to communicate with each other. 
To communicate with other pods we need to create a service for our deployment.
First edit the service file and apply the yaml to the cluster.

If you created a docker container from the nodeJs or Go folder, you set up a server and send a request to environment variable URL.
To use this functionality we should add environment variables to our deployment. We can do this by adapting the container section in the deployment config:
```
 containers:     
    - name: YOUR_NAME
      image: YOUR_IMAGE
      ports:
        - containerPort: 8080
      env:
       - name: URL
         value: "localhost:8080"
```
Or by adding a reference to a configmap where all environment variables are stored in one file:
```
 containers:     
    - name: YOUR_NAME
      image: YOUR_IMAGE
      ports:
        - containerPort: 8080
      envFrom:
          - configMapRef:
              name: YOUR_NAME
```

Now we need to apply the configmap `kubectl apply -f manifests/configmap.yaml` and the deployment again.

Second question is which url we should configure. Therefore we need to look up the service name `kubectl get service`. And we can simply fill in the service name as a url.

