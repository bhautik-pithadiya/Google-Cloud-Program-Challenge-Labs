#Still in progress

# Perform Foundational Infrastructure Tasks in Google Cloud: Challenge Lab

> Launch the lab [here](https://www.qwiklabs.com/focuses/12068?parent=catalog)

In this lab there is no command line instructions, you have to just use the **Google Cloud Platform (GCP)** for completing this lab. A walkthrough of each task is available below: 
  
  ->  Task 1 : Create a Docker image and store the Dockerfile.<br>
  ->  Task 2 : Test the created Docker image.<br>
  ->  Task 3 : Push the Docker image into the Container Repository.<br>
  ->  Task 4 : Create and expose a deployment in Kubernetes.<br>
  ->  Task 5 : Update the deployment with a new version of valkyrie-app.<br>
  ->  Task 6 : Create a pipeline in Jenkins to deploy your app.<br>
 

### Solving tasks

## Task 1 : Create a Docker image and store the Dockerfile

* Run the following Code in the **Cloud Shell Terminal**

```yaml
gcloud auth list
gsutil cat gs://cloud-training/gsp318/marking/setup_marking_v2.sh | bash
gcloud source repos clone valkyrie-app
cd valkyrie-app
cat > Dockerfile <<EOF
FROM golang:1.10
WORKDIR /go/src/app
COPY source .
RUN go install -v
ENTRYPOINT ["app","-single=true","-port=8080"]
EOF
```
* Change **Docker Image** and **Tag Name**

```yaml
docker build -t <Docker Image>:<Tag Name> .
cd ..
cd marking
./step1_v2.sh
```


## Task 2 : Test the created Docker image.

```yaml
cd ..
cd valkyrie-app
```

* Change **Docker Image** and **Tag Name**

```yaml
docker run -p 8080:8080 <Docker Image>:<Tag Name> &
cd ..
cd marking
./step2_v2.sh

```

## Task 3 : Push the Docker image into the Container Repository.

```yaml
cd ..
cd valkyrie-app
```

* Change **Docker Image** and **Tag Name**

```yaml
docker tag <Docker Image>:<Tag Name> gcr.io/$GOOGLE_CLOUD_PROJECT/<Docker Image>:<Tag Name>
docker push gcr.io/$GOOGLE_CLOUD_PROJECT/<Docker Image>:<Tag Name>
```

## Task 4 : Create and expose a deployment in Kubernetes.

* Replace the **"[NETWORK TAG-2]"** with the network tag provided in the lab.

```yaml
gcloud compute firewall-rules create http-ingress --allow=tcp:80 --source-ranges 0.0.0.0/0 --target-tags [NETWORK TAG-2] --network acme-vpc
```
```yaml
gcloud compute instances add-tags juice-shop --tags=[NETWORK TAG-2] --zone=us-central1-b
```
## Task 5 : Create a firewall rule that allows traffic on ***SSH (tcp/22)*** from acme-mgmt-subnet network address and add network tag on juice-shop

* Replace the **"[NETWORK TAG-3]"** with the network tag provided in the lab.

```yaml
gcloud compute firewall-rules create internal-ssh-ingress --allow=tcp:22 --source-ranges 192.168.10.0/24 --target-tags [NETWORK TAG-3] --network acme-vpc
```
```yaml
gcloud compute instances add-tags juice-shop --tags=[NETWORK TAG-3] --zone=us-central1-b
```

#### Task 6 : SSH to bastion host via IAP and juice-shop via bastion

* In Compute Engine -> VM Instances page, click the SSH button for the **Bastion** host.
* Write the below code in **SSH** terminal
```yaml
gcloud compute ssh juice-shop --intenal-ip
```
* If asked for **Passphrase**  -->   Hit *Enter* twice, then Y.


## **✨Congratulations✨** you have completed Your Challenge lab Succesfully.🎉🎉
