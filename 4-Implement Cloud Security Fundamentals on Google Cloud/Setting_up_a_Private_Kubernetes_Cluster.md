# Setting up a Private Kubernetes Cluster

## Setup and requirements

#### (Optional) You can list the active account name with this command:
```bash
gcloud auth list
```

#### (Optional) You can list the project ID with this command:
```bash
gcloud config list project
```

#### Set the default zone:
```bash
gcloud config set compute/zone <ZONE>
```

#### Create a variable for region:
```bash
export REGION=<Region>
```

#### Create a variable for region:
```bash
export export ZONE=<Zone>
```

#### Learn more from the [Regions & Zones documentation](https://docs.cloud.google.com/compute/docs/regions-zones).


## Task 1. Creating a private cluster

#### 1.1 When you create a private cluster, you must specify a /28 CIDR range for the VMs that run the Kubernetes master components and you need to enable IP aliases.
#### Next you'll create a cluster named private-cluster, and specify a CIDR range of 172.16.0.16/28 for the masters. When you enable IP aliases, you let Kubernetes Engine automatically create a subnetwork for you.
#### You'll create the private cluster by using the --private-cluster, --master-ipv4-cidr, and --enable-ip-alias flags.


#### 1.2 Run the following to create the cluster:

```bash
gcloud beta container clusters create private-cluster \
    --enable-private-nodes \
    --master-ipv4-cidr 172.16.0.16/28 \
    --enable-ip-alias \
    --create-subnetwork ""
```

## Task 2. View your subnet and secondary address ranges


#### 2.1 List the subnets in the default network:
```bash
gcloud compute networks subnets list --network default
```

#### 2.2 In the output, find the name of the subnetwork that was automatically created for your cluster. For example, gke-private-cluster-subnet-xxxxxxxx. Save the name of the cluster, you'll use it in the next step.

#### 2.3 Now get information about the automatically created subnet, replacing <SUBNET_NAME> with your subnet by running:
```bash
gcloud compute networks subnets describe <SUBNET_NAME> --region=$REGION
```

## Task 3. Enable master authorized networks
#### 3.1 Create a source instance which you'll use to check the connectivity to Kubernetes clusters:

```bash
gcloud compute instances create source-instance --zone=$ZONE --scopes 'https://www.googleapis.com/auth/cloud-platform'
```

#### 3.2 Get the <External_IP> of the source-instance with:
```bash
gcloud compute instances describe source-instance --zone=$ZONE | grep natIP
```
#### Copy the <nat_IP> address and save it to use in later steps.

#### 3.3 Run the following to Authorize your external address range, replacing <MY_EXTERNAL_RANGE> with the CIDR range of the external addresses from the previous output (your CIDR range is natIP/32). With CIDR range as natIP/32, we are allowlisting one specific IP address:
```bash
gcloud container clusters update private-cluster \
    --enable-master-authorized-networks \
    --master-authorized-networks <MY_EXTERNAL_RANGE>
```
#### Now that you have access to the master from a range of external addresses, you'll install kubectl so you can use it to get information about your cluster. For example, you can use kubectl to verify that your nodes do not have external IP addresses.

#### 3.4 SSH into source-instance with:
```bash
gcloud compute ssh source-instance --zone=$ZONE
```
#### Press Y to continue. Enter through the passphrase questions.

#### 3.5 In SSH shell install kubectl component of Cloud-SDK:
```bash
sudo apt-get install kubectl
```

#### 3.6 Configure access to the Kubernetes cluster from SSH shell with:
```bash
sudo apt-get install google-cloud-sdk-gke-gcloud-auth-plugin
gcloud container clusters get-credentials private-cluster --zone=$ZONE
```

#### 3.7 Verify that your cluster nodes do not have external IP addresses:
```bash
kubectl get nodes --output yaml | grep -A4 addresses
```

#### 3.8 Here is another command you can use to verify that your nodes do not have external IP addresses:
```bash
kubectl get nodes --output wide
```

#### 3.9 Close the SSH shell by typing:
```bash
exit
```

## Task 4. Clean Up
#### 4.1 Delete the Kubernetes cluster:
```bash
gcloud container clusters delete private-cluster --zone=$ZONE
```

## Task 5. Create a private cluster that uses a custom subnetwork
#### 5.1 Create a subnetwork and secondary ranges:
```bash
gcloud compute networks subnets create my-subnet \
    --network default \
    --range 10.0.4.0/22 \
    --enable-private-ip-google-access \
    --region=$REGION \
    --secondary-range my-svc-range=10.0.32.0/20,my-pod-range=10.4.0.0/14
```

#### 5.2 Create a private cluster that uses your subnetwork:
```bash
gcloud beta container clusters create private-cluster2 \
    --enable-private-nodes \
    --enable-ip-alias \
    --master-ipv4-cidr 172.16.0.32/28 \
    --subnetwork my-subnet \
    --services-secondary-range-name my-svc-range \
    --cluster-secondary-range-name my-pod-range \
    --zone=$ZONE
```

#### 5.3 Retrieve the external address range of the source instance:
```bash
gcloud compute instances describe source-instance --zone=$ZONE | grep natIP
```
#### Copy the <nat_IP> address and save it to use in later steps.

#### 5.4 Run the following to Authorize your external address range, replacing <MY_EXTERNAL_RANGE> with the CIDR range of the external addresses from the previous output (your CIDR range is natIP/32). With CIDR range as natIP/32, we are allowlisting one specific IP address:
```bash
gcloud container clusters update private-cluster2 \
    --enable-master-authorized-networks \
    --zone=$ZONE \
    --master-authorized-networks <MY_EXTERNAL_RANGE>
```

#### 5.5 SSH into source-instance with:
```bash
gcloud compute ssh source-instance --zone=$ZONE
```

#### 5.6 Configure access to the Kubernetes cluster from SSH shell with:
```bash
gcloud container clusters get-credentials private-cluster2 --zone=$ZONE
```

#### 5.7 Verify that your cluster nodes do not have external IP addresses:
```bash
kubectl get nodes --output yaml | grep -A4 addresses
```

## Congratulations!