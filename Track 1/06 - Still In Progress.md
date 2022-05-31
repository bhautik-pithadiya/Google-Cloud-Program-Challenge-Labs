First Task -------
```yaml
gcloud config set compute/zone us-east1-b

git clone https://source.developers.google.com/p/$DEVSHELL_PROJECT_ID/r/sample-app

gcloud container clusters get-credentials jenkins-cd

kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=$(gcloud config get-value account)

helm repo add stable https://charts.helm.sh/stable

helm repo update

helm install cd stable/jenkins
```
```yaml
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/component=jenkins-master" -l "app.kubernetes.io/instance=cd" -o jsonpath="{.items[0].metadata.name}")
kubectl port-forward $POD_NAME 8080:8080 >> /dev/null &
printf $(kubectl get secret cd-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
```
copy the password
```yaml
cd sample-app
kubectl create ns production
kubectl apply -f k8s/production -n production
kubectl apply -f k8s/canary -n production
kubectl apply -f k8s/services -n production
```
```yaml
kubectl get svc
kubectl get service gceme-frontend -n production
```
* In Jenkins -> Manage Jenkins -> Manage Credentials -> Global -> Adding Some Credentials
* Select Google Service Account from metadata - > OK
* Go to Dashboard 
* Select New Item -> Enter sample-app -> Select Multibranch Pipeline -> OK
* Select the Branch Sources as Git.
* Copy the Repository from the Tip 3 at hte last of your challenge lab site and Change * [PROJECT_ID] * to your project id.
* Select Credentials as qwiklabs.......
* Scroll Down -> CHeck * Preiodically if ot otherwise run * and Set the interval to 1 min.
* Now Apply -> Save.

After these commands do the jenkins part then do the rest commands -

```yaml
git init
git config credential.helper gcloud.sh
git remote add origin https://source.developers.google.com/p/$DEVSHELL_PROJECT_ID/r/sample-app
```
```yaml
git config --global user.email "<user email>"
git config --global user.name "<user name>"
```
```yaml
git add .
git commit -m "initial commit"
git push origin master
```

2nd Task -----
```yaml
git checkout -b new-feature
```
* Press Open Editor on the Cloud Shell Terminal -> Open sample-app floder.
* Edit main.go file -> change * const version string *  from 1.0.0 to 2.0.1 (Check the value under the 2nd Task ) -> Save it.
* Edit html.go file -> Change * <div class="card _____"> * to * <div class="card red"> *
 * After this command make the required changes!
```yaml
git add Jenkinsfile html.go main.go
git commit -m "Version 2.0.0"
git push origin new-feature
```
3rd Task ----------

curl http://localhost:8001/api/v1/namespaces/new-feature/services/gceme-frontend:80/proxy/version
kubectl get service gceme-frontend -n production  
git checkout -b canary
git push origin canary
export FRONTEND_SERVICE_IP=$(kubectl get -o \
jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)
git checkout master
git push origin master

4rth Task -------

export FRONTEND_SERVICE_IP=$(kubectl get -o \
jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)
while true; do curl http://$FRONTEND_SERVICE_IP/version; sleep 1; done

kubectl get service gceme-frontend -n production

git merge canary
git push origin master
export FRONTEND_SERVICE_IP=$(kubectl get -o \
jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)

DONE --------
