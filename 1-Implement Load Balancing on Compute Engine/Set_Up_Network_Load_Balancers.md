
# Set Up Network Load Balancers

## Setup and requirements

#### (Optional) You can list the active account name with this command:
```bash
gcloud auth list
```

#### (Optional) You can list the project ID with this command:
```bash
gcloud config list project
```

## Task 1. Set the default region and zone for all resources

#### In Cloud Shell, set the default region:
```bash
gcloud config set compute/region <REGION>
```

#### Set the default zone:
```bash
gcloud config set compute/zone <ZONE>
```

## Task 2. Create multiple web server instances

#### Create a virtual machine, www1, in your default zone using the following code:
```bash
gcloud compute instances create www1 \
    --zone=<ZONE> \
    --tags=network-lb-tag \
    --machine-type=e2-small \
    --image-family=debian-11 \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
      apt-get update
      apt-get install apache2 -y
      service apache2 restart
      echo "
<h3>Web Server: www1</h3>" | tee /var/www/html/index.html'
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

## Task 3. Configure the load balancing service

#### Create a static external IP address for your load balancer:
```bash
gcloud compute addresses create network-lb-ip-1 \
  --region <REGION>
```

#### Add a legacy HTTP health check resource:
```bash
gcloud compute http-health-checks create basic-check
```

## Task 4. Create the target pool and forwarding rule

#### Run the following to create the target pool and use the health check, which is required for the service to function:
```bash
gcloud compute target-pools create www-pool \
  --region <REGION> --http-health-check basic-check
```

#### Add the instances you created earlier to the pool:
```bash
gcloud compute target-pools add-instances www-pool \
    --instances www1,www2,www3
```

#### Add a forwarding rule:
```bash
gcloud compute forwarding-rules create www-rule \
    --region  <REGION> \
    --ports 80 \
    --address network-lb-ip-1 \
    --target-pool www-pool
```

## Task 5. Send traffic to your instances

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

## Congratulations!