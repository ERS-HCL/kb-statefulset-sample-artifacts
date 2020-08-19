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

