# EFK (ELK) for Hosted Azure Kubernetes Service (AKS)
# Introduction
This document contains the steps to configure EFK for AKS ie. Azure Kubernetes Service.

Pre-Requisites
1. Working Kubernetes cluster on Azure
2. Elasticsearch Server on same Azure VNet or somewhere else with accessibility 
3. Packages to be installed:

    a. docker
    
    b. kubectl 
4. dockerhub or azure container registry or any  image repository

This Repo contains:

Dockerfile:
We are using fluentd official image and we are installing necessary packages like json, es plugin, kubernetes metadata, etc...

entrypoint.sh:
This is the script which will execute the fluentd command and keeps the process running on the foreground.

Gemfile:
The Gemfile folder contains the plugins to be installed which is referred inside the dockerfile.

conf/fluent.conf
Fluent configuration has the reference to kubernetes and syslog config available in same directory. 
> Note: Please change the IP address and port number in fluent.conf (Elasticsearch IP, Port )

conf/kubernetes.conf
This has the patterns for filtering container logs

conf/syslog.conf
This has the patterns for filtering system logs.

ds.yml (daemonset)
A DaemonSet ensures that an instance of a specific pod is running on all (or a selection of) nodes in a cluster.
We are about to create a service account, Cluster role and Role binding to provide access to the containers to read the logs stored in the node machines. i.e /var/log/containers

> Note: Please mention the name of the image built using the dockerfile above.

# How to do:
1. Get into the root directory of the cloned repo , update the configuration as per your requirement,
2. Then Build the image.
> docker build -t efk/fluentd:v1 . -f Dockerfile
3. Once built, Login to dockerhub and upload the image to dockerhub
> docker login 
> docker push efk/fluentd:v1 
4. Now, we need to update the imagename inside the ds.yml file and create a daemonset
> kubectl create -f ds.yml
5. Now you can watch the pods provisioned inside k8s by,
> kubectl get pods -n kube-system -w
6. Once the pods are live, You can verify the logs in your Elasticsearch server.
    - Create index pattern for syslog as syslog* and provide @timestamp for timefilter
    - Create index pattern for kubernetes logs as logstash* or whatever you like and @timestamp for timefilter
    - You can view the logs now by using the above create index. 
