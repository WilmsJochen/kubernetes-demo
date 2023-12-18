# Exercise: Deploy your own docker container in the kubernetes cluster.
In this exercise, the goal is to deploy your previously created docker image in a kubernetes cluster.


## Environment
You can execute this exercise on all machines with a connection to a k8's cluster. In case of issues, you could use the playgrounds of killercoda.
https://killercoda.com/playgrounds/scenario/kubernetes

## Instructions

### check out all the yaml files in the manifest folder. These define some basic kubernetes resources.
Have a quick look on how all these manifest define kubernetes resources.

### Checkout git repo.

1. Clone this exercise repo.
    ```
    git clone https://github.com/WilmsJochen/kubernetes-demo.git
    ```
3. Change directory to exercise 2.
    ```
    cd kubernetes-demo
    ```

### get to know some kubectl commands

Execute following commands and investigate their output. Fill in the Names with a name you found in the get command.

```
** kubectl get RESOURCE **
kubectl get pod
kubectl get deployment

** kubectl describe RESOURCE RESOURCE_NAME **
kubectl describe pod POD_NAME
kubectl describe deployment DEPLOYMENT_NAME

** kubectl edit RESOURCE RESOURCE_NAME **
KUBE_EDITOR="nano" kubectl edit pod POD_NAME

** kubectl apply -f YAML_FILE **
kubectl apply -f ./manifest/pod-example.yaml

** kubectl logs POD_NAME **
kubectl logs example-pod       

** kubectl edit RESOURCE RESOURCE_NAME **
kubectl delete example-pod
    
```

### deploy a pod with your docker image of choice.

You can deploy pods by applying the manifest pod.yaml. Please adapt the yaml according to your preferences and your image.
Editing the manifest can be done with nano from inside the terminal or any other editing tool. 

```
nano manifests/pod.yaml
```
**_TIP:_**  your image name looks like: `REPO_NAME/YOUR_NAME:VERSION `

Apply the pod manifest to your cluster.
```
kubectl apply -f manifests/pod.yaml
```

Congratulations! Your docker image of choice is running in a kubernetes cluster.
You can verify that the pod is running with the command`kubctl get pod`

## Make a deployment from your pod.

First we need to cleanup our cluster to make some space.
```
kubectl delete pod POD_NAME
```

Now we need to modify the deployment.yaml file with your own variables.
```
nano manifests/deployment.yaml
```

Apply your deployment to the cluster.
```
kubectl apply -f manifests/deployment.yaml
```
Play with the number of replicas.

```
kubectl scale deployment DEPLOYMENT_NAME --replicas 2
```

And scale back to 1 pod after you have verified the scaling with `kubectl get pods`.
```
kubectl scale deployment DEPLOYMENT_NAME --replicas 1
```
Delete the pod created by the deployment and look what happens. 
```
kubectl delete pod POD_NAME
```

## Create a service
To communicate with a pod from a specific deployment, we need to add a service to these pods. We don't care which pod, as long as we have one available.
```
nano manifests/service.yaml
```
Make sure to name the service selector exactly the same as your deployment matchlabel value in the config.
If you followed the guidelines, this will be your name.

```
kubectl apply -f manifests/service.yaml
```

## Expose your service to the world.
From this point we can talk to a service instead of a pod. But to expose this service to the outside world, we need to have a public ip address and some routing. 
Luckily for us most of the cloud providers support ingress and will automatically assign a public ip.
We only need to create an ingress rule to route traffic to your services.

As you can see there is already an ingress defined in our cluster. From this command you will see the ip where it is running on.
```
kubectl get ingress
```
To create add your own ingress rule, the ingress.yaml file should be configured
```
nano manifests/ingress.yaml
```
and applied:
```
kubectl apply -f manifests/ingress.yaml
```

Now you could browse to the ingress.

## Time to communicate with each other inside our cluster. 

This is the moment you can create a docker image from the Jobs folder in Ex1. In the job folder you will find some code that will execute a http request and stop running.
As you can see this use case won't fit in a regular deployment. This is why kubernetes added some jobs and cronjob resources.

NOTE: To save time you can also use the image `desyco/docker-demo-job`

You can again apply the manifest after configuring you own variables.

As you can see the Job failed because the url wasn't configured. The code will read the url from an environment variable named URL.
There are 2 ways to add an environment variable: 

1. add the environment directly in the container config of the deployment.
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
2. Be cool, group env variables in a configmap, add a reference to that configmap in the containers config of your resource.
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

 Don't forget to apply the configmap `kubectl apply -f manifests/configmap.yaml` and the Job again after changes.


## Tips:
- If something is wrong in the pod, you can see logs of your application with `kubectl logs POD_NAME`

- If something is wrong in your manifest, you better describe the resource `kubectl describe KIND NAME`

- Use tab key for auto completion of commands.

- This link can be useful to create a cron expression for a cronjob.
https://crontab.guru
