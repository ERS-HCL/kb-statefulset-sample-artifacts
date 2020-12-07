# Replicated StatefulSet deployment Artifact for Kubernetes - with Core Banking appliaction example

## Description
This repository contains the declarative Kubernetes deployment configuration for replicated StatefulSet with persistence volume, persistence volume claim and storage class. The deployment has been chiseled in respect of creating a core banking applicaiton which involves following:

1. Customer Service Management (CSM) Microservice - This is the first microservice which provides endpoints to store static data for bank & branch, customer and account. It will be used by transaction microservice.

URL: https://github.com/ERS-HCL/k8s-csm

Deployment Highlights:
Pod: Single Container
Stateful: No
Replica: 2
Service: Loadbalancer type to expose outside
Volume: NA

2. Core Banking Transaction (TRN) Microservice - This is the second microservice which provides endpoints for basic core banking operations like deposit by cash, widhdrawal by case, fund transfer, transaction enquiry and so on.

URL: https://github.com/ERS-HCL/k8s-trn

Deployment Highlights:
Pod: Single Container
Stateful: No
Replica: 2
Service: Loadbalancer type to expose outside
Volume: NA

Both microservice will be using MySQL database as backend data store which also will be deployed on Kubernetes cluster. 

The database deployement will be a replicated StatefulSet deployment with following properties:
Pod: Two Containers (1. mysql:5.7, 2. xtrabackup to replicate the database logs to other instaces)
Stateful: Yes
Replica: 2
Service: Headless (as it will be used by deployed applicaiton microservice)
Volume: Yes (for each replica)

Once database will be deployed, both microservices also need to be started to form a complete application.

For sake of testing, have also included stateless deployment of mysql configuration with persistent volume.

To read full article visit URL: https://medium.com/@siddhivinayak.sk/kubernetes-and-replicated-statefulset-with-real-world-application-9bf21fca9eb


## Use Case
The Kubernetes artifacts is used to create deployment of replicated statefulset with persistent volume claim and persistent volume. Once deployment completed, it becomes a complete example for replicated statefulset applicability in real world application.  


## Technology Stack
- Docker
- Kubernetes 1.16 or later
- MySQL 5.7


## Pre-Condition
It is being assumed that there is already a running Kubernetes cluster availble with Docker.

The microservices code has been built, docker image created and pushed into the registry.


## Deployment on Kubernetes - Stateless deployment
Create Storage Class, Persistent volume and Persistent Volume Claim:

```
kubectl create -f pv-data-mysql-0.yaml -n bank-apps

kubectl create -f pv-data-mysql-1.yaml -n bank-apps
```

Creating headless service:
```
kubectl create -f mysql-services.yaml -n bank-apps
```

Creating configmap use into other configuration:
```
kubectl create -f mysql-configmap.yaml -n bank-apps
```

Create replicated stateful deployment of mysql server:
```
kubectl create -f mysql-statefulset.yaml -n bank-apps
```

Disclaimer: This code has been written for testing purpose, the execution and correctness depends on various envrironment parameters. It should not be used for any live usages.
