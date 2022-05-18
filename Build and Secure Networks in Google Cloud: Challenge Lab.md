# Perform Foundational Infrastructure Tasks in Google Cloud: Challenge Lab

> Launch the lab [here](https://www.qwiklabs.com/focuses/12068?parent=catalog)

In this lab there is no command line instructions, you have to just use the **Google Cloud Platform (GCP)** for completing this lab. A walkthrough of each task is available below: 
  
  ->  Task 1 : Remove the overly permissive rules.<br>
  ->  Task 2 : Start the bastion host instance.<br>
  ->  Task 3 : Create a firewall rule that allows SSH (tcp/22) from the IAP service and add network tag on bastion.<br>
  ->  Task 4 : Create a firewall rule that allows traffic on HTTP (tcp/80) to any address and add network tag on juice-shop.<br>
  ->  Task 5 : Create a firewall rule that allows traffic on SSH (tcp/22) from acme-mgmt-subnet network address and add network tag on juice-shop.<br>
  ->  Task 6 : SSH to bastion host via IAP and juice-shop via bastion.<br>
 

## Solving tasks

### Task 1 : Remove the overly permissive rules

```yaml
gcloud compute firewall-rules delete open-access
```
### Task 2 : Start the bastion host instance

* Go to Compute Engine and start **Bastion** instance

### Task 3 : Create a firewall rule that allows SSH (tcp/22) from the IAP service and add network tag on bastion

* Replace the **"[NETWORK TAG]"** with the network tag provided in the lab.

```yaml
gcloud compute firewall-rules create ssh-ingress --allow=tcp:22 --source-ranges 35.235.240.0/20 --target-tags [NETWORK TAG-1] --network acme-vpc
```
```yaml
gcloud compute instances add-tags bastion --tags=[NETWORK TAG-1] --zone=us-central1-b
```

### Task 4 : Create a firewall rule that allows traffic on HTTP (tcp/80) to any address and add network tag on juice-shop

* Replace the **"[NETWORK TAG-2]"** with the network tag provided in the lab.

```yaml
gcloud compute firewall-rules create http-ingress --allow=tcp:80 --source-ranges 0.0.0.0/0 --target-tags [NETWORK TAG-2] --network acme-vpc
```
```yaml
gcloud compute instances add-tags juice-shop --tags=[NETWORK TAG-2] --zone=us-central1-b
```
### Task 5 : Create a firewall rule that allows traffic on SSH (tcp/22) from acme-mgmt-subnet network address and add network tag on juice-shop

* Replace the **"[NETWORK TAG-3]"** with the network tag provided in the lab.

```yaml
gcloud compute firewall-rules create internal-ssh-ingress --allow=tcp:22 --source-ranges 192[dot]168[dot]10[dot]0/24 --target-tags [NETWORK TAG-3] --network acme-vpc
```
```yaml
gcloud compute instances add-tags juice-shop --tags=[NETWORK TAG-3] --zone=us-central1-b
```

### Task 6 : SSH to bastion host via IAP and juice-shop via bastion

* In Compute Engine -> VM Instances page, click the SSH button for the bastion host. Then SSH to juice-shop by

```yaml
ssh [Internal IP address of juice-shop]
```
