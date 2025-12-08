# Develop your Google Cloud Network: Challenge Lab

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

#### Export varaibles 
```bash
export REGION="us-east1"
export ZONE="us-east1"
```

#### In Cloud Shell, set the default region:
```bash
gcloud config set compute/region $REGION
```

#### Set the default zone:
```bash
gcloud config set compute/zone $ZONE
```

## Task 1. Create Development VPC (griffin-dev-vpc)
#### Create a VPC called griffin-dev-vpc with the following subnets only:
``` bash
gcloud compute networks create griffin-dev-vpc --subnet-mode=custom
```

#### SSH access for griffin-dev-vpc:
``` bash
gcloud compute firewall-rules create griffin-dev-vpc-ssh --network griffin-dev-vpc --allow tcp:22
```

#### Create subnet griffin-dev-wp - IP address block: 192.168.16.0/20
``` bash
gcloud compute networks subnets create griffin-dev-wp \
  --network=griffin-dev-vpc --region=$REGION --range=192.168.16.0/20
```

#### griffin-dev-mgmt - IP address block: 192.168.32.0/20
``` bash
gcloud compute networks subnets create griffin-dev-mgmt \
  --network=griffin-dev-vpc --region=$REGION --range=192.168.32.0/20
```

## Task 2. Create production VPC manually
#### Create a VPC called griffin-prod-vpc with the following subnets only:
``` bash
gcloud compute networks create griffin-prod-vpc --subnet-mode=custom
```

#### Create subnet griffin-prod-wp IP address block: 192.168.48.0/20
``` bash
gcloud compute networks subnets create griffin-prod-wp \
  --network=griffin-prod-vpc --region=$REGION --range=192.168.48.0/20
```

#### Create subnet griffin-prod-mgmt IP address block: 192.168.64.0/20
``` bash
gcloud compute networks subnets create griffin-prod-mgmt \
  --network=griffin-prod-vpc --region=$REGION --range=192.168.64.0/20
```

#### SSH access for griffin-prod-vpc:
``` bash
gcloud compute firewall-rules create griffin-prod-vpc-ssh --network griffin-prod-vpc --allow tcp:22
```

## Task 3. Create Bastion Host with Two NICs

#### Create a bastion host with two network interfaces:

``` bash
gcloud compute instances create griffin-bastion \
  --zone=$ZONE \
  --machine-type=e2-medium \
  --tags=bastion \
  --network-interface=subnet=griffin-dev-mgmt \
  --network-interface=subnet=griffin-prod-mgmt
```

#### Testing ssh connection to griffin-bastion VM
``` bash
gcloud compute ssh griffin-bastion --zone=$ZONE
```

## Task 4. Create and configure Cloud SQL Instance
#### Create a MySQL Cloud SQL Instance called griffin-dev-db in <REGION>.
``` bash
gcloud sql instances create griffin-dev-db \
    --database-version=MYSQL_8_0 \
    --tier=db-f1-micro \
    --region=$REGION \
    --root-password="Conn123"
```

#### Get Connection Name
``` bash
export CONNECTION_NAME=$(gcloud sql instances describe griffin-dev-db --format='value(connectionName)')
```

#### Init proxy for sql
``` bash
cloud_sql_proxy -instances=$CONNECTION_NAME=tcp:3306 &
```

#### Connect as root into MySQL Intance
``` bash
mysql -u root -p -h 127.0.0.1
```

#### Prepare the WordPress environment. These SQL statements create the worpdress database and create a user with access to the wordpress database.
``` sql
CREATE DATABASE wordpress;
CREATE USER "wp_user"@"%" IDENTIFIED BY "stormwind_rules";
GRANT ALL PRIVILEGES ON wordpress.* TO "wp_user"@"%";
FLUSH PRIVILEGES;
```

## Task 5. Create Kubernetes cluster
#### Create a 2 node cluster (e2-standard-4) called griffin-dev, in the griffin-dev-wp subnet, and in zone <ZONE>.
``` bash
gcloud container clusters create griffin-dev \
  --zone=$ZONE \
  --num-nodes=2 \
  --machine-type=e2-standard-4 \
  --network=griffin-dev-vpc \
  --subnetwork=griffin-dev-wp
```

## Task 6. Prepare the Kubernetes cluster
#### 6.1 From Cloud Shell copy all files from gs://spls/gsp321/wp-k8s
``` bash
gsutil -m cp -r gs://spls/gsp321/wp-k8s .
cd wp-k8s
```

#### 6.2 Use the command below to create the Service Account Key:
``` bash
gcloud iam service-accounts keys create key.json \
    --iam-account=cloud-sql-proxy@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
```

#### 6.3 Use the command below to Secret for Service Account Key, and then add the key to the Kubernetes environment:
``` bash
kubectl create secret generic cloudsql-instance-credentials \
    --from-file key.json
```

#### 6.4 Add the following secrets and volume to the cluster using <b>wp-env.yaml</b>.
#### Get Connection Name de Cloud SQL
``` bash
DB_CONN=$(gcloud sql instances describe griffin-dev-db --format='value(connectionName)')
echo $DB_CONN
```

#### 6.5 Edit wp-env.yaml using SED
#### Reemplaza username por el base64 correcto
``` bash
sed -i 's/username: .*/username: d3BfdXNlcg==/' wp-env.yaml
```
#### Reemplaza password por el base64 correcto
``` bash
sed -i 's/password: .*/password: c3Rvcm13aW5kX3J1bGVz/' wp-env.yaml
```
#### Reemplaza YOUR_SQL_INSTANCE por el Connection Name real
``` bash
sed -i "s/YOUR_SQL_INSTANCE/$CONNECTION_NAME/g" wp-env.yaml
```

#### 6.5 Apply environment configuration:
``` bash
kubectl apply -f wp-env.yaml
```

#### 6.6 Veirfy secrets and PVC
``` bash
kubectl get secrets \
kubectl get pvc
```

## Task 7. Create WordPress Deployment
#### 7.1 Get instance connection name:

``` bash
DB_CONN=$(gcloud sql instances describe griffin-dev-db --format='value(connectionName)')
echo "Connection Name: $DB_CONN"
```

#### 7.2 Edit <b>wp-deployment.yaml</b> and replace <YOUR_SQL_INSTANCE>:
``` bash
sed -i "s/YOUR_SQL_INSTANCE/$CONNECTION_NAME/g" wp-deployment.yaml
```

#### 7.3 Apply deployment configuration:
``` bash
kubectl apply -f wp-deployment.yaml
```

#### 7.4 Create the Service for External Exposure. This file defines a LoadBalancer service to expose the WordPress
``` bash
kubectl apply -f wp-service.yaml
```

#### 7.4 Verify and Access the Site
``` bash
kubectl get service wordpress
```
#### Use a browser to access the URL http://[EXTERNAL-IP]


## Task 8. Enable monitoring
#### Create an uptime check for your WordPress development site.

#### 8.1 Get the Public IP Address of the Load Balancer
``` bash
EXTERNAL_IP=$(kubectl get service wordpress --format='jsonpath="{.status.loadBalancer.ingress[0].ip}"')
echo "IP Pública de WordPress: $EXTERNAL_IP"
```

#### 8.2 Create the Uptime Check
``` bash
gcloud alpha monitoring uptime-checks create http griffin-dev-wordpress-check \
    --display-name="WordPress Dev Uptime Check" \
    --resource-type=uptime_url \
    --resource-labels=host=$EXTERNAL_IP \
    --port=80 \
    --validate-ssl=false
```

## Task 9. Provide Access for an Additional Engineer
#### Provide access for an additional engineer:
``` bash
gcloud projects add-iam-policy-binding $GOOGLE_CLOUD_PROJECT \
   --member="user:SECOND_USER_EMAIL" \
   --role="roles/editor"
```

## Congratulations!
