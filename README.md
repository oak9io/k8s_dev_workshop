# k8s_dev_workshop
![TerraOak](oak9-logo.png)
Deploying to K8's Using Argocd and Jenkins

## Table of Contents
* [Introduction](#introduction)
* [Prerequistes](#Must-Haves-Before-Starting)
* [Installation Minikube](#Getting-Started-Minikube)
* [Installation Argocd](#Getting-Started-ArgoCD)
* [Installation Jenkins](#Getting-Started-Jenkins)
* [Configuring Ngrok](#Configuring-ngrok)
* [Running Pipelne](#Blast-Off)

## Introduction 

This repo will will be used for Oak9 Dev workshop on how to get started with microservices on Kubernetes using ArgoCD and Jenkins.  We will walk through the instalation process and build a sample app that will deploy to minikube locally using jenkins and argocd. 

Before you proceed, WARNING:
All the installations steps have been performed on a Mac. 

## Prerequistes 

Github Account 
Install helm
Install Minikube
Install Jenkins 
Install Ngrok


## Intallation Minikube
Instructions can be founde here, 
https://helm.sh/docs/intro/install/

## Intallation Minikube

Let's install Minikube first, the installation instructions can be found here, https://minikube.sigs.k8s.io/docs/start/


## Installation Argocd

Now lets install argocd on our minikube cluster. Argo CD is installed within our kubernetes cluster, in a specific namespace. 

Argo CD is a pull-based deployment tool. It watches a remote Git repository for new or updated manifest files and synchronizes those changes with the cluster.


install argocd on kubernetes 
```
# create the namespace
$ kubectl create namespace argocd

# apply the installation manifest
$ kubectl apply \
    --namespace argocd \
    --filename https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

port-forward to pod to get localhost UI access 
kubectl port-forward \
    --namespace argocd \
    svc/argocd-server 8000:443

Get Password to be Used in Running with Demo 
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo

## Installation Jenkins

Now lets install Jenkins onto our minikube cluster, 

Steps: 
kubectl create namespace jenkins
kubectl get namespaces ( you should see jenkins)
helm repo add jenkinsci https://charts.jenkins.io
helm repo update
kubectl apply -f jenkins-volume.yaml
minikube ssh
mkdir -p /data/jenkins-volume
sudo chown -R 1000:1000 /data/jenkins-volume (***)
kubectl apply -f jenkins-sa.yaml
helm install jenkins -n jenkins -f jenkins-values.yaml jenkinsci/jenkins

```
Get your 'admin' user password by running:

$ jsonpath="{.data.jenkins-admin-password}"
$ secret=$(kubectl get secret -n jenkins jenkins -o jsonpath=$jsonpath)
$ echo $(echo $secret | base64 --decode)
Get the Jenkins URL to visit by running these commands in the same shell:

Get the name of the Pod running that is running Jenkins using the following command:

$ kubectl get pods -n jenkins
Use the kubectl command to set up port forwarding:

$ kubectl -n jenkins port-forward <pod_name> 8080:8080
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080

```

Visit 127.0.0.1:8080/ and log in using admin as the username and the password you retrieved earlier.

## Configuring ngrok 
head over to https://ngrok.com/ and setup an account, Its Free dont worry ! 


## Configuring your Github with Jenkins and Ngrok

You will need to fork this codebase into your github repo and do the following 

Run ngrok to create a reverse proxy 

```
./ngrok http 8000
```

configure you jenkins to send a webhook to your jenkins box for your repo, https://www.blazemeter.com/blog/how-to-integrate-your-github-repository-to-your-jenkins-project

note your url will look something like this, dont forget trailing slash !!! 
https://*****-****-***.ngrok.io/github-webhook/  

configure Jenkins to have access to your github repo

```
Create a deploy key for your repo, https://docs.github.com/en/developers/overview/managing-deploy-keys#deploy-keys


```

##Configure Argocd

1. create a project that we can link our jenkins pipeline to and push images that will get pushed to kubernetes 


## Running with Demo 

Clone this repo locally onto your workstation 