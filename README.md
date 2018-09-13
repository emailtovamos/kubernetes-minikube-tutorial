# kubernetes-minikube-tutorial
Basic tutorial for minikube of Kubernetes (**Only for MacOS**)

Minikube is a tool to run Kubernetes locally on your machine. It can run inside a virtual machine situated on your laptop to try out Kubernetes.

This tutorial shows how to have a simple Node.js app run on Kubernetes cluster on your laptop. The steps after the setting up part are: 

    Create Minikube cluster
    
    Create Node.js file

    Create Dockerfile

    Turn the Node.js program into a Docker container image

    Run the image on Minikube
    
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
    
