# Implement Cloud Security Fundamentals on Google Cloud: Challenge Lab

## Setup and requirements

#### (Optional) You can list the active account name with this command:
```bash
gcloud auth list
```

#### (Optional) You can list the project ID with this command:
```bash
gcloud config list project
```

## Challenge scenario

#### In Cloud Shell, set the default region:
```bash
gcloud config set compute/region <REGION>
```

#### Set the default zone:
```bash
gcloud config set compute/zone <ZONE>
```

#### Export varaibles 
```bash
export REGION="us-central1"
export ZONE="us-central1-c"
export ORCA_VPC="orca-build-vpc"
export MGMT_SUBNET_CIDR="192.168.10.2/32"
export MASTER_SUBNET_CIDR="172.16.0.0/28"
export JUMPHOST_TAG="orca-jumphost"

```
## Task 1. Create a custom security role
#### 1.1 Time to get started! Create your role definition YAML file by running:
```bash
nano orca-security-role.yaml
```

#### 1.2 Add this custom role definition to the YAML file and save:
```yaml
title: "Orca security role "
description: "Provide the Google Cloud storage bucket and object permissions required to be able to create and update storage objects"
stage: "ALPHA"
includedPermissions:
- storage.buckets.get
- storage.objects.get
- storage.objects.list
- storage.objects.update
- storage.objects.create
```

#### 1.3 Execute the following gcloud command:
```bash
gcloud iam roles create orca_storage_editor_338 --project $DEVSHELL_PROJECT_ID --file orca-security-role.yaml
```

#### 1.4 To view the role metadata, use command below
```bash
gcloud iam roles describe orca_storage_editor_338 --project $DEVSHELL_PROJECT_ID
```

## Task 2. Create a service account
#### To create a service account, run the following command in Cloud Shell:
```bash
gcloud iam service-accounts create orca-private-cluster-452-sa --display-name "service account for GKE orca-cluster-471"
```

## Task 3. Bind a custom security role to a service account
#### Run the following in Cloud Shell to grant roles to the service account you just made:
#### add custome-role
```bash
gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID \
    --member serviceAccount:orca-private-cluster-452-sa@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com \
    --role projects/$DEVSHELL_PROJECT_ID/roles/orca_storage_editor_338 
```
#### add custome-role 2
```bash
gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID \
    --member serviceAccount:orca-private-cluster-452-sa@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com \
    --role roles/monitoring.viewer 
```
#### add custome-role 3
```bash
gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID \
    --member serviceAccount:orca-private-cluster-452-sa@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com \
    --role roles/monitoring.metricWriter 
```
#### add custome-role 4
```bash
gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID \
    --member serviceAccount:orca-private-cluster-452-sa@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com \
    --role roles/logging.logWriter
```
 

## Task 4. Create and configure a new Kubernetes Engine private cluster


#### 4.3 Create a private cluster that uses your subnetwork:
```bash
gcloud container clusters create orca-cluster-471 \
    --zone=$ZONE \
    --network=$ORCA_VPC \
    --subnetwork=orca-build-subnet \
    --service-account=orca-private-cluster-452-sa@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com \
    --master-authorized-networks=$MGMT_SUBNET_CIDR \
    --enable-ip-alias \
    --enable-private-nodes \
    --enable-private-endpoint \
    --enable-master-authorized-networks \
    --num-nodes=1
```

#### 4.4 Create a instance for orca-jumphost
```bash
gcloud compute instances create orca-jumphost \
    --zone=$ZONE \
    --machine-type=e2-small \
    --network-interface subnet=orca-mgmt-subnet,private-network-ip=192.168.10.2 \
    --tags=$JUMPHOST_TAG
```

## Task 5. Deploy an application to a private Kubernetes Engine cluster
#### 5.1 Run command to ssh connection to VM jumphost
```bash
gcloud compute ssh orca-jumphost --zone=$ZONE
```

#### 5.2 Install gcloud-auth-plugin, kubectl and get cluster credentials
```bash
sudo apt-get install google-cloud-sdk-gke-gcloud-auth-plugin
echo "export USE_GKE_GCLOUD_AUTH_PLUGIN=True" >> ~/.bashrc
source ~/.bashrc
sudo apt-get install kubectl
gcloud container clusters get-credentials orca-cluster-471 --internal-ip --project=qwiklabs-gcp-02-500f5a668c93 --zone us-central1-c
```

#### 5.3 Deploy an application to the GKE cluster
```bash
kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0
```

#### 5.4 Expose an application on port 8080
```bash
kubectl expose deployment hello-server --type=LoadBalancer --port 80 --target-port 8080
```

## Congratulations!

## TIPS

#### Fix ssh error connection to VM
```bash
gcloud compute instances add-metadata bigquery-instance \
    --metadata enable-oslogin=FALSE \
    --zone us-central1-c
```

#### Run command to ssh connection to VM
```bash
gcloud compute ssh bigquery-instance --zone us-central1-c
```
