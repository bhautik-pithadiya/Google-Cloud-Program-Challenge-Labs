#Still in progress

# Set Up and Configure a Cloud Environment in Google Cloud: Challenge Lab

> Launch the lab [here](https://www.cloudskillsboost.google/focuses/10603?parent=catalog)

In this lab there is no command line instructions, you have to just use the **Google Cloud Platform (GCP)** for completing this lab. A walkthrough of each task is available below: 
  
  ->  Task 1 : Create development VPC manually.<br>
  ->  Task 2 : Create production VPC manually.<br>
  ->  Task 3 : Create bastion host.<br>
  ->  Task 4 : Create and configure Cloud SQL Instance.<br>
  ->  Task 5 : Create Kubernetes cluster.<br>
  ->  Task 6 : Prepare the Kubernetes cluster.<br>
  ->  Task 7 : Create a WordPress deployment.<br>
  ->  Task 8 : Enable monitoring.<br>
  ->  Task 9 : Provide access for an additional engineer.<br>
 

### Solving tasks

## Task 1 : Create development VPC manually.

* Run the following Code in the <ins>**Cloud Shell Terminal**</ins>

```yaml
gcloud compute networks create griffin-dev-vpc --subnet-mode custom

gcloud compute networks subnets create griffin-dev-wp --network=griffin-dev-vpc --region us-east1 --range=192.168.16.0/20

gcloud compute networks subnets create griffin-dev-mgmt --network=griffin-dev-vpc --region us-east1 --range=192.168.32.0/20

```

## Task 2 : Create production VPC manually.

```yaml
gsutil cp -r gs://cloud-training/gsp321/dm .
```
```yaml
cd dm

sed -i s/SET_REGION/us-east1/g prod-network.yaml

gcloud deployment-manager deployments create prod-network \
    --config=prod-network.yaml

cd ..

```

## Task 3 : Create bastion host.

```yaml
gcloud compute instances create bastion --network-interface=network=griffin-dev-vpc,subnet=griffin-dev-mgmt  --network-interface=network=griffin-prod-vpc,subnet=griffin-prod-mgmt --tags=ssh --zone=us-east1-b

gcloud compute firewall-rules create fw-ssh-dev --source-ranges=0.0.0.0/0 --target-tags ssh --allow=tcp:22 --network=griffin-dev-vpc

gcloud compute firewall-rules create fw-ssh-prod --source-ranges=0.0.0.0/0 --target-tags ssh --allow=tcp:22 --network=griffin-prod-vpc

```

## Task 4 : Create and configure Cloud SQL Instance.

```yaml
gcloud sql instances create griffin-dev-db --root-password password --region=us-east1

gcloud sql connect griffin-dev-db --user=root
```
* Enter Password as  <ins>password</ins>   (all Lower Case)

```yaml
CREATE DATABASE wordpress;
CREATE USER "wp_user"@"%" IDENTIFIED BY "stormwind_rules";
```

```yaml
GRANT ALL PRIVILEGES ON wordpress.* TO "wp_user"@"%" IDENTIFIED BY "stormwind_rules"; 
```

* (if you are getting error here, ignore it).

```yaml
FLUSH PRIVILEGES;
```
```yaml
EXIT
```

## Task 5 : Create Kubernetes cluster.


```yaml
gcloud container clusters create griffin-dev \
  --network griffin-dev-vpc \
  --subnetwork griffin-dev-wp \
  --machine-type n1-standard-4 \
  --num-nodes 2  \
  --zone us-east1-b


gcloud container clusters get-credentials griffin-dev --zone us-east1-b

cd ~/

gsutil cp -r gs://cloud-training/gsp321/wp-k8s .

```

### Task 6 : Prepare the Kubernetes cluster.

* Click on Open Editor -> wp-k8s -> wp-env.yaml Change username and password to:

```yaml
username : wp_user
password : stormwind_rules
```

* Save.

```yaml
cd wp-k8s

kubectl create -f wp-env.yaml

gcloud iam service-accounts keys create key.json \
    --iam-account=cloud-sql-proxy@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
kubectl create secret generic cloudsql-instance-credentials \
    --from-file key.json

```
### Task - 7 : Create a WordPress deployment.

* In editor: wp-deployment.yaml -> replace YOUR_SQL_INSTANCE with griffin-dev-db.
* Save.

```yaml
kubectl create -f wp-deployment.yaml
kubectl create -f wp-service.yaml
```

### Task - 8 : Enable monitoring.

* Navigation Menu -> Kubernetes Engine -> Services and Ingress -> Copy Endpoint's address.
* Navigation Menu -> Monitoring -> Uptime Checks -> + CREATE UPTIME CHECK.

```yaml
Title : Wordpress Uptime
```

* Next -> Target

```yaml
   Hostname : {Endpoint's address} (without http...)
   Path : /
```

* Next -> Next -> Create

### Task - 9 : Provide access for an additional engineer.

* Navigation Menu -> IAM & Admin -> IAM -> ADD.

```yaml
New Member : {Username 2 from Lab instruction page}
Role : Project -> Editor
```

* Save.

## **✨Congratulations✨** you have completed Your Challenge lab Succesfully.🎉🎉
