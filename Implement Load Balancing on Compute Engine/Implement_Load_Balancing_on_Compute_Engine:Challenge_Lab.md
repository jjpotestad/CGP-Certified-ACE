# Implement Load Balancing on Compute Engine: Challenge Lab

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

## Task 1. Create multiple web server instances

#### Create a virtual machine, www1, in your default zone using the following code:
##### <b>VM web1</b>
```bash
gcloud compute instances create web1 \
    --zone=<ZONE> \
    --tags=network-lb-tag \
    --machine-type=e2-small \
    --image-family=debian-12 \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
      apt-get update
      apt-get install apache2 -y
      service apache2 restart
      echo "
<h3>Web Server: web1</h3>" | tee /var/www/html/index.html'
```

##### <b>VM web2</b>
```bash
gcloud compute instances create web2 \
    --zone=<ZONE> \
    --tags=network-lb-tag \
    --machine-type=e2-small \
    --image-family=debian-12 \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
      apt-get update
      apt-get install apache2 -y
      service apache2 restart
      echo "
<h3>Web Server: web2</h3>" | tee /var/www/html/index.html'
```

##### <b>VM web3</b>
```bash
gcloud compute instances create web3 \
    --zone=<ZONE> \
    --tags=network-lb-tag \
    --machine-type=e2-small \
    --image-family=debian-12 \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
      apt-get update
      apt-get install apache2 -y
      service apache2 restart
      echo "
<h3>Web Server: web3</h3>" | tee /var/www/html/index.html'
```
#### Create a firewall rule to allow external traffic to the VM instances:
```bash
gcloud compute firewall-rules create www-firewall-network-lb \
    --target-tags network-lb-tag --allow tcp:80
```

### Run the following to list your instances. You'll see their IP addresses in the EXTERNAL_IP column:
```bash
gcloud compute instances list
```

### Verify that each instance is running with curl
```bash
curl http://[IP_ADDRESS]
```

## Task 2. Configure the load balancing service
### Task 2.1 Create a static external IP address for your load balancer:
```bash
gcloud compute addresses create network-lb-ip-1 \
  --region <REGION>
```

#### Add a legacy HTTP health check resource:
```bash
gcloud compute http-health-checks create basic-check
```

### Task 2.2 Create the target pool and forwarding rule. 
#### Run the following to create the target pool and use the health check, which is required for the service to function:
```bash
gcloud compute target-pools create www-pool \
  --region <REGION> --http-health-check basic-check
```

#### Add the instances you created earlier to the pool:
```bash
gcloud compute target-pools add-instances www-pool \
    --instances web1,web2,web3
```

#### Add a forwarding rule:
```bash
gcloud compute forwarding-rules create www-rule \
    --region  <REGION> \
    --ports 80 \
    --address network-lb-ip-1 \
    --target-pool www-pool
```

### Task 2.3 Send traffic to your instances
#### Enter the following command to view the external IP address of the www-rule forwarding rule used by the load balancer:
```bash
gcloud compute forwarding-rules describe www-rule --region <REGION>
```

#### Access the external IP address:
```bash
IPADDRESS=$(gcloud compute forwarding-rules describe www-rule --region <REGION> --format="json" | jq -r .IPAddress)
```

#### Show the external IP address:
```bash
echo $IPADDRESS
```

#### Use the curl command to access the external IP address, replacing IP_ADDRESS with an external IP address from the previous command:
```bash
while true; do curl -m1 $IPADDRESS; done
```

## Task 3. Create an HTTP load balancer
#### First, create the load balancer template:
```bash
gcloud compute instance-templates create lb-backend-template \
   --region=<REGION> \
   --network=default \
   --subnet=default \
   --tags=allow-health-check \
   --machine-type=e2-medium \
   --image-family=debian-12 \
   --image-project=debian-cloud \
   --metadata=startup-script='#!/bin/bash
     apt-get update
     apt-get install apache2 -y
     a2ensite default-ssl
     a2enmod ssl
     vm_hostname="$(curl -H "Metadata-Flavor:Google" \
     http://169.254.169.254/computeMetadata/v1/instance/name)"
     echo "Page served from: $vm_hostname" | \
     tee /var/www/html/index.html
     systemctl restart apache2'
```

#### Create a managed instance group based on the template:
```bash
gcloud compute instance-groups managed create lb-backend-group \
   --template=lb-backend-template --size=2 --zone=<ZONE>
```

#### Create the fw-allow-health-check firewall rule.
```bash
gcloud compute firewall-rules create fw-allow-health-check \
  --network=default \
  --action=allow \
  --direction=ingress \
  --source-ranges=130.211.0.0/22,35.191.0.0/16 \
  --target-tags=allow-health-check \
  --rules=tcp:80
```

#### Now that the instances are up and running, set up a global static external IP address that your customers use to reach your load balancer:

```bash
gcloud compute addresses create lb-ipv4-1 \
  --ip-version=IPV4 \
  --global
```
#### Notice that the IPv4 address that was reserved:
```bash
gcloud compute addresses describe lb-ipv4-1 \
  --format="get(address)" \
  --global
```
#### Create a health check for the load balancer (to ensure that only healthy backends are sent traffic):
```bash
gcloud compute health-checks create http http-basic-check \
  --port 80
```
#### Create a backend service:
```bash
gcloud compute backend-services create web-backend-service \
  --protocol=HTTP \
  --port-name=http \
  --health-checks=http-basic-check \
  --global
```
#### Add your instance group as the backend to the backend service:
```bash
gcloud compute backend-services add-backend web-backend-service \
  --instance-group=lb-backend-group \
  --instance-group-zone=<ZONE> \
  --global
```
#### Create a URL map to route the incoming requests to the default backend service:
```bash
gcloud compute url-maps create web-map-http \
    --default-service web-backend-service
```
#### Create a target HTTP proxy to route requests to your URL map:
```bash
gcloud compute target-http-proxies create http-lb-proxy \
    --url-map web-map-http
```
#### Create a global forwarding rule to route incoming requests to the proxy:
```bash
gcloud compute forwarding-rules create http-content-rule \
   --address=lb-ipv4-1\
   --global \
   --target-http-proxy=http-lb-proxy \
   --ports=80    
```

## Task 4. Test traffic sent to your instances

#### 1- On the Google Cloud console in the Search field type Load balancing, then choose Load balancing from the search results
#### 2- Click on the load balancer that you just created, <b>web-map-http</b>.
#### 3- In the <b>Backend</b> section, click on the name of the backend and confirm that the VMs are <b>Healthy</b>. If they are not healthy, wait a few moments and try reloading the page.
#### 4- When the VMs are healthy, test the load balancer using a web browser, going to http://IP_ADDRESS/, replacing IP_ADDRESS with the load balancer's IP address that you copied previously. Your browser should render a page with content showing the name of the instance that served the page, along with its zone (for example, Page served from: lb-backend-group-xxxx).

## Congratulations!