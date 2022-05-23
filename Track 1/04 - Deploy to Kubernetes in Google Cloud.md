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

* Run the following Code in the <ins>**Cloud Shell Terminal**</ins>

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

* Change <ins>**Docker Image**</ins> and <ins>**Tag Name**</ins>

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

* Change <ins>**Docker Image**</ins> and <ins>**Tag Name**</ins>

```yaml
docker run -p 8080:8080 <Docker Image>:<Tag Name> &
```
* Once you have your container running, and before clicking **Check my progress**, run *step2_v2.sh* to perform the local check of your work. 
* After you get a successful response from the local marking you can check your progress.

```yaml
cd ..
cd marking
./step2_v2.sh

```

## Task 3 : Push the Docker image into the Container Repository.

```yaml
cd ..
cd valkyrie-app
```

* Change <ins>**Docker Image**</ins> and <ins>**Tag Name**</ins>

```yaml
docker tag <Docker Image>:<Tag Name> gcr.io/$GOOGLE_CLOUD_PROJECT/<Docker Image>:<Tag Name>
```
```yaml
docker push gcr.io/$GOOGLE_CLOUD_PROJECT/<Docker Image>:<Tag Name>
```

## Task 4 : Create and expose a deployment in Kubernetes.

* Change <ins>**Docker Image**</ins> and <ins>**Tag Name**</ins>

```yaml
sed -i s#IMAGE_HERE#gcr.io/$GOOGLE_CLOUD_PROJECT/<Docker Image>:<Tag Name>#g k8s/deployment.yaml
gcloud container clusters get-credentials valkyrie-dev --zone us-east1-d
kubectl create -f k8s/deployment.yaml
kubectl create -f k8s/service.yaml
```

## Task 5 : Update the deployment with a new version of valkyrie-app.


```yaml
git merge origin/kurt-dev
kubectl edit deployment valkyrie-dev
```
* Press <ins>**i**</ins> to edit in the editor then
* change replicas from 1 to **Replicas Count** in two places
* change <ins>**Tag Name**</ins> to <ins>**Updated Version**</ins> in two places
* After changing, press <ins>**ESC**</ins> than type <ins>**:wq**</ins> to close the editor

```yaml
docker build -t gcr.io/$GOOGLE_CLOUD_PROJECT/<Docker Image>:<Updated Version> .
docker push gcr.io/$GOOGLE_CLOUD_PROJECT/<Docker Image>:<Updated Version>
```

### Task 6 : Create a pipeline in Jenkins to deploy your app.

```yaml
docker ps
```
```yaml
docker kill <take container_id from above command>
```
```yaml
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/component=jenkins-master" -l "app.kubernetes.io/instance=cd" -o jsonpath="{.items[0].metadata.name}")
kubectl port-forward $POD_NAME 8080:8080 >> /dev/null &
printf $(kubectl get secret cd-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo

```

* Note the Output.
* Open Jenkins -- Web PreView -> Preview on port 8080.

```yaml
  Username : admin
  Password : {Code output from previous command} 
```
* Go through the following:

* Manage Jenkins -> Manage Credentials -> Jenkins -> Global credentials (unrestricted) -> Add credentials -> Kind: Google Service Account from metadata -> OK

* Jenkins -> New Item -> Name : valkyrie-app -> Pipeline -> Pipeline script from SCM -> Set SCM to git -> OK

* Pipeline -> Script: Pipeline script from SCM -> SCM: Git

* Repository URL: {find it using command} -> Credentials : {Select Project id(eg qwiklab.......)} 
```yaml 
gcloud source repos list  #Run in the Cloud Shell Terminal
```

* Apply -> Save

* Run the Following Code in Cloud Shell Terminal.

```yaml
sed -i "s/green/orange/g" source/html.go
sed -i "s/YOUR_PROJECT/$GOOGLE_CLOUD_PROJECT/g" Jenkinsfile
```

* Enter Your Cloud registered Email.
* Enter Username -> <ins>**netstudent-02-8db47dd8c98a**</ins>  from <ins>**netstudent-02-8db47dd8c98a@qwiklabs.net**</ins>

```yaml
git config --global user.email "you@example.com"              # Email
git config --global user.name "student..."                     # Username
```

```yaml
git add .
git commit -m "built pipeline init"
git push

```
* Go to Jenkins Tab, click Build and wait to get score.
* It will take Approx <ins>5 - 10</ins> mins to build.




## **✨Congratulations✨** you have completed Your Challenge lab Succesfully.🎉🎉
