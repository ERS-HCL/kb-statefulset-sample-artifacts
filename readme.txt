# Core Banking Transaction (TRN) Microservice - A Spring Boot Project

## Description
This project has been created to create a microservice which will expose the endpoints for basic core banking functionalities for example, deposit by cash, withdrawal by cash, fund transfer, dual bank ledger entry on transaction, enquiry and many more.

To run this microservice, the static data microservice must be running beasue it contains only trasaction part so the static data like bank and branch, customer and account must be provided by the said static microservie. 

The static data microservice named Customer Service Management (CSM) microservice. Link: https://github.com/siddhivinayak-sk/csm


## Use Case
This microservice has been designed to be part of and bank applicaiton demo which demostrate the core banking functionality with microservice based example. This microservice only contains core banking transactions with help of CSM microservice which contains the static data for processing of bank transactions.


## Technology Stack
- Java 8 or later
- Spring boot 2.x
- Maven as build tool
- Spotify docker plugin
- Docker
- Kubernetes 1.16 or later
- MySQL 5.7


## Build
Maven has been used as build tool for this project with default Spring boot build plugin.
Hence, we can use below maven code to build the runnable jar as microservice:

```
mvn clean package
```

It also has maven spotify plug in which is used for docker image creation from maven build tool by using below command:

```
mvn clean package docker:build -DpushImageTag
```
The above command will craete pacakge (jar file) after that create docker image, tag it as specifed into spotify configuration into pom.xml and push it to the registry.


## Deployment on Kubernetes - Stateless deployment
This project also contains Kubernetes declarative configuration (in form of yaml) for stateless deployment. Once, docker image created and pushed into registry, the below command can be used to create deployment into the the Kubernetes. (Pre-condidtion, there must be running Kubernetes cluster)

```
kubectl create -f trn-kb8-deployment.yaml
```

## Configuraiton
The maven project contains resource directory which contains applicaiton.yaml. This configuration file will be used to configure database and other properties.

Disclaimer: This code has been written for testing purpose, the execution and correctness depends on various envrironment parameters. It should not be used for any live usages.





KBRNET1: Demonstrate Kubernetes capabilities on. Deployment of stateful application using StatefulSet, Availability using Replication Controller, Storage management using Persistent Volume/ Persistent Volume Claim and more functionalitiesMentee should have minimum 3 month hands on experience on Docker concepts ( containers/ images/ registry/ network/ volumes/ compose)and basic understanding of Linux concepts

kubectl create -f pv-data-mysql-0.yaml -n bank-apps

kubectl create -f pv-data-mysql-1.yaml -n bank-apps

kubectl delete -f pv-data-mysql-0.yaml -n bank-apps

kubectl delete -f pv-data-mysql-1.yaml -n bank-apps

kubectl create -f mysql-statefulset.yaml -n bank-apps

kubectl delete -f mysql-statefulset.yaml -n bank-apps

kubectl get pods -n bank-apps --watch

kubectl describe pods -n bank-apps


Deployment Steps:
0. Share the drive and make a directory which has to bind with persistent-volume - Required only into Docker for Windows

1. Create storage-class

kubectl create -f pv-data-mysql-0.yaml -n bank-apps

kubectl create -f pv-data-mysql-1.yaml -n bank-apps

kubectl get sc -n bank-apps

2. Create persistent-volume

kubectl get pv -n bank-apps

3. Create persistent-volume claim

kubectl get pvc -n bank-apps

4. Create configmap

kubectl create -f mysql-configmap.yaml -n bank-apps

kubectl describe cm mysql -n bank-apps

5. Create Services

kubectl create -f mysql-services.yaml -n bank-apps
kubectl get services -n bank-apps
kubectl get all -n bank-apps
kubectl describe service -n bank-apps

6. Create Stateful

kubectl create -f mysql-statefulset.yaml -n bank-apps
kubectl get pods -n bank-apps --watch
kubectl get pods -n bank-apps
kubectl describe pod -n bank-apps

Testing:
1. Create a database named test, add a table and one insert script
kubectl run mysql-client --image=hub.docker.local:5000/mysql -i --rm --restart=Never -n bank-apps -- mysql -h mysql-0.mysql -u root < my-sqls-file.sql
CREATE DATABASE test; 
CREATE TABLE test.messages (message VARCHAR(250)); 
INSERT INTO test.messages VALUES ('hello');

2. Run the select query and see the result
kubectl run mysql-client --image=hub.docker.local:5000/mysql -i -t --rm --restart=Never -n bank-apps -- mysql -h mysql-read -u root -e "SELECT * FROM test.messages"

3. Test mysql-read in loop, it automatically change the running instace for each pod and getting same data
kubectl run mysql-client-loop --image=hub.docker.local:5000/mysql -i -t --rm --restart=Never -n bank-apps -- bash -ic "while sleep 1; do mysql -h mysql-read -u root -e 'SELECT @@server_id,NOW()'; done"

4. Pod has been forcebly stopped, and StatefulSet automatically, brings new Pod with same PVC to take place
command prompt 1
kubectl delete pod mysql-1 --force -n bank-apps

kubectl get pvc -n bank-apps

command prompt 2
kubectl get pods -n bank-apps --watch

5. Scaling up Pod
kubectl scale statefulset mysql --replicas=3 -n bank-apps

kubectl run mysql-client --image=hub.docker.local:5000/mysql -i -t --rm --restart=Never -n bank-apps -- mysql -h mysql-2.mysql -u root -e "SELECT * FROM test.messages"

6. Scaling back
kubectl scale statefulset mysql --replicas=2 -n bank-apps

kubectl get pvc -n bank-apps

Delete PVC manually
kubectl delete pvc data-mysql-2 -n bank-apps

