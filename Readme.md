# Ex2: Deploy your own docker container in the kubernetes cluster.

in this exercise, the goal is to deploy your previously created docker image in the kubernetes cluster. And discover each others secret messages.

## Checkout git repo.
1. Change your directory back to YOUR_NAME. Use `cd` to go back to your home folder.
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
   
## Sorts of kubernetes concept:
- Pod
- deployment
- service
- configmap
- job
- cronjob
- ingress

These are all the kubernetes concepts we will certainly touch today. You can find an example of how the resources are defined in the manifest folder.
I invite you to first take a look on all these definitions. There are more

## get to know some kubectl commands

Execute following commands and investigate their output. Fill in the Names with a name you found in the get command.

```
kubectl get pod
kubectl get OTHER_RESOURCE 
kubectl describe pod POD_NAME
kubectl describe OTHER_RESOURCE RESOURCE_NAME
KUBE_EDITOR="nano" kubectl edit pod POD_NAME
KUBE_EDITOR="nano" kubectl edit OTHER_RESOURCE RESOURCE_NAME
kubectl logs POD_NAME       
```

## deploy your first pod with your docker image.

You can simply deploy a pod with the definition you found in the manifest. You need to adapt that definition with your own variables.
Editing the variables can be done with nano from the terminal. 

```
nano manifests/pod.yaml
```

**_TIP:_**  when you are ready editing, you exit nano with ctrl+x.

Now you can apply the pod manifest to your cluster.
```
kubectl apply -f manifests/pod.yaml
```

Congratulations! Your own docker image is running in a kubernetes cluster.
You can verify that the pod is running with `kubctl get pod`

## Make a deployment from you pod.

First we need to delete your pod from the cluster to make some space.
```
kubectl delete pod POD_NAME
```

Now we need to modify the deployment.yaml file with our own variables.
```
nano manifests/deployment.yaml
```

Apply your deployment to the cluster.
```
kubectl apply -f manifests/deployment.yaml
```
Play with the number of replicas you want to have of your deployment.
**_NOTE:_**  Please scale only to 2 pods so we don't overload our small cluster with a lot of pods.
```
kubectl scale deployment YOUR_NAME --replicas 2
```
And scale back to 1 pod after you have verified the scaling with `kubectl get pods`.
```
kubectl scale deployment YOUR_NAME --replicas 1
```

## Expose your service to the world.
To communicate with a pod from a specific deployment, we need to add a service to these pods. We don't care which pod, as long as we have one available.
```
kubectl apply -f manifests/service.yaml
```

From this point we can talk to a service instead of a pod. But to expose this service to the outside world, we need to have a public ip address. Luckily for us most of the cloud providers support ingress and will automatically assign an ip for us.
Form this ingress we can route a path to our service.

As you can see there is already an ingress controller defined in our cluster.
```
kubectl get ingress
```

Because we only want 1 ingress controller and different path's for all our services, we are going to edit this ingress instead of applying/overwriting the ingress.
I think we can have some merge conflicts as people are adding their path's on the same time. Ask permission to edit this ingress before editing it.
```
KUBE_EDITOR="nano" kubectl edit ingress
```

Add the following statement in the correct place.
```
  - path: /YOUR_NAME
    backend:
      serviceName: YOUR_NAME-service
      servicePort: 80
```

Now you can go to the ip of the ingress on your path in your browser. It will look like http://xx.xx.xx.xx/YOUR_NAME

## Time to communicate to each other with a job. 

This is the moment you can create a docker image from the Jobs folder in Ex1. In the job folder you will find some code that will execute a http request and stop running.
As you can see this use case won't fit in a regular deployment. This why kubernetes added some jobs and cronjobs. Cronjobs are fancy jobs that will execute at a given time.

You can again apply the manifest after configuring you own variables.

As you can see the Job failed because the url wasn't set correctly. The code is configured to read this url from an environment variable named URL.
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
2. Or by grouping them in a configmap, and add a reference to that configmap in the manifest.
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
- If something is wrong in the pod, you can see log of your application with `kubectl logs POD_NAME`

- If something is wrong in your manifest, you can see a description with `kubectl describe KIND NAME`

- Use tab key for auto completion of commands.

- This link can be useful to create a cron expression for a cronjob.
https://crontab.guru
