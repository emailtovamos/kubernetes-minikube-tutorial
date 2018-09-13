# kubernetes-minikube-tutorial
Basic tutorial for minikube of Kubernetes (**Only for MacOS**)

Minikube is a tool to run Kubernetes locally on your machine. It can run inside a virtual machine situated on your laptop to try out Kubernetes.

This tutorial shows how to have a simple Node.js app run on Kubernetes cluster on your laptop. The steps after the setting up part are: 

    Create Minikube cluster
    
    Create Node.js application file

    Create Dockerfile & Turn the Node.js program into a Docker container image & build

    Run the image on Minikube by creating a Deployment
    
    Create a Service
    
After this we will see how to: 

    View application logs
    
    Update the application image
    
Eventually we will do the cleanup the resources we created in the cluster. 

## Initial setting up

Install minikube: 

    brew cask install minikube
  
Install [NodeJS](https://nodejs.org/en/)

Install [Docker](https://docs.docker.com/docker-for-mac/)

Install [VirtualBox](https://www.virtualbox.org/wiki/Downloads). This will be the detault Virtual Machine driver. 

## Create Minikube cluster

Install latest release of Minikube:

    brew cask install minikube
    
Install the command-line tool called **kubectl** which can be used to interact with Kubernetes clusters: 

    brew install kubernetes-cli
    
Make sure you have started the Docker daemon. Check it using command: 

    docker images
    
Assuming [no proxy is required](https://kubernetes.io/docs/tutorials/hello-minikube/#create-your-node-js-application), start the Minikube cluster: 

    minikube start
    
**kubectl** needs to know which cluster it is interacting with because there can be multiple cluster options. All the available contexts in **~/.kube/config** file. So ```open ~/.kube/config``` to see. 

Since we want to interact with the Minikube cluster, we have to set the context: 

    kubectl config use-context minikube
        
Check the configuration: 

    kubectl cluster-info 
    
& check if you can open Kubernetes dashboard in browser: 

    minikube dashboard
    
## Create Node.js application file

Save the following code in a folder named ```minikube``` in a file named ```server.js```

    var http = require('http');

    var handleRequest = function(request, response) {
    console.log('Received request for URL: ' + request.url);
    response.writeHead(200);
    response.end('Namaste World!');
    };
    var www = http.createServer(handleRequest);
    www.listen(8080);
    
Running this application: 

    node server.js
    
will show "Namaste World!" at http://localhost:8080

Next is to package the application in a Docker container so that it can be deployed.

## Create Dockerfile & Turn the Node.js program into a Docker container image & build

Create a file named ```Dockerfile``` inside the same folder where ```server.js``` is. A Dockerfile describes the **image** we want to build. An image of an application basically has all the instructions for a complete and executable version of the application. An instance of an image is called a **container**. One can have many running containers of the same image. One can build a Docker container image by extending an existing image. Here we extend an exsting Node.js image in the Dockerfile: 

    FROM node:6.9.2
    EXPOSE 8080
    COPY server.js .
    CMD node server.js
    
Then we can `build` the image from the Dockerfile. Before that we need to configure the local environment to re-use the Docker daemon inside the Minikube instance. We do this by: 

    eval $(minikube docker-env)
    
By doing this we make life easier for us by building and running Docker images inside the Minikube environment. Otherwise we would have to build the docker image on the host machine, push the image to minikube and THEN set up deployment in Minikube to use the image. But after doing the above configuration we simply have to build the docker image using Minikube's docker instance which pushes the image to Minikube's Docker registry after which we can set up the deployment in Minikube to use the image. 

So now we build the Docker image, using the Minikube Docker daemon: 

    docker build -t hello-node:v1 .
    
## Run the image on Minikube by creating a Deployment

A Kubernetes cluster has a `Master` & `Nodes`. The applications run on Nodes. Nodes have the containerized applications which run on them. A `Deployment` is responsible for creating and updating the instances of our applications. Deployment instructs Kubernetes how to create and update instances of our applications. Once application instances are created, Kubernetes Deployment controller continuously monitors those instances. For example, if a Node goes down then the Deployment controller replaces it. 

Each Node has at least one Pod which is a group of one or more containers tied together. These containers can communicate with each other. In our current case, the pod only has one container. The Kubernetes Deployment will check the health of our Pod and restarts the Pod's container if it terminates. 

The Deployment command in our case: 

    kubectl run hello-node --image=hello-node:v1 --port=8080 --image-pull-policy=Never

Here `hello-node` is name of the Deployment, `hello-node:v1` is the image. `imge-pull-policy` is set to `Never` since we are not going to pull it from Docker registry but rather from the Minikube Docker registry itself. 

We can view the Deployment: 

    kubectl get deployments
    
Output: 

    NAME         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    hello-node   1         1         1            1           2h
    
Also we can view the Pod: 

    kubectl get pods

Output: 

    NAME                          READY     STATUS    RESTARTS   AGE
    hello-node-57c6b66f9c-6z64t   1/1       Running   0          2h
    
Also we can see cluster events: 

    kubectl get events
    
Output: 

    LAST SEEN   FIRST SEEN   COUNT     NAME                                           KIND         SUBOBJECT                     TYPE      REASON                  SOURCE                  MESSAGE
    1m          1m           1         hello-node-57c6b66f9c-6z64t.1553f1c2cf5a17e2   Pod                                        Normal    Scheduled               default-scheduler       Successfully assigned hello-node-57c6b66f9c-6z64t to minikube
    1m          1m           1         hello-node-57c6b66f9c-6z64t.1553f1c2de141016   Pod                                        Normal    SuccessfulMountVolume   kubelet, minikube       MountVolume.SetUp succeeded for volume "default-token-vkjn6"
    1m          1m           1         hello-node-57c6b66f9c-6z64t.1553f1c2f8a4226a   Pod          spec.containers{hello-node}   Normal    Pulled                  kubelet, minikube       Container image "hello-node:v1" already present on machine
    1m          1m           1         hello-node-57c6b66f9c-6z64t.1553f1c2fbd41249   Pod          spec.containers{hello-node}   Normal    Created                 kubelet, minikube       Created container
    1m          1m           1         hello-node-57c6b66f9c-6z64t.1553f1c3044b36d5   Pod          spec.containers{hello-node}   Normal    Started                 kubelet, minikube       Started container
    1m          1m           1         hello-node-57c6b66f9c.1553f1c2cb4478d3         ReplicaSet                                 Normal    SuccessfulCreate        replicaset-controller   Created pod: hello-node-57c6b66f9c-6z64t
    1m          1m           1         hello-node.1553f1c2c6a7bcd5                    Deployment                                 Normal    ScalingReplicaSet       deployment-controller   Scaled up replica set hello-node-57c6b66f9c to 1
    
Configuration of kubectl can be seen: 

    kubectl config view
    
## Create a Service

    kubectl expose deployment hello-node --type=LoadBalancer
    
Output: 

    service "hello-node" exposed
    
View the service: 

    kubectl get services
    
Output: 

    NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
    hello-node   LoadBalancer   10.103.238.64   <pending>     8080:30918/TCP   41m
    kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP          21h

On Minikube the `LoadBalancer` type makes the service accessible through `minikube service` command: 

    minikube service hello-node
    
## View application logs

If we have sent requests to the new web service either by browser or by curl the we can see some logs: 

    kubectl logs <POD-NAME> 
    
Here POD-NAME can be got by executing: 

    kubectl get pods

