# Multiple VPC Networks

## Setup and requirements

#### (Optional) You can list the active account name with this command:
```bash
gcloud auth list
```

#### (Optional) You can list the project ID with this command:
```bash
gcloud config list project
```

## Task 1. Create custom mode VPC networks with firewall rules

#### Create the privatenet network using the Cloud Shell command line, along with firewall rules to allow SSH, ICMP, and RDP ingress traffic.
#### 1.1 Run the following command to create the privatenet network:
```bash
gcloud compute networks create privatenet --subnet-mode=custom
```

#### 1.2 Run the following command to create the privatesubnet-1 subnet:
```bash
gcloud compute networks subnets create privatesubnet-1 --network=privatenet --region=<Region_1> --range=172.16.0.0/24
```

#### 1.3 Run the following command to create the privatesubnet-2 subnet:
```bash
gcloud compute networks subnets create privatesubnet-2 --network=privatenet --region=<Region_2> --range=172.20.0.0/20
```

#### 1.4 Run the following command to list the available VPC networks:
```bash
gcloud compute networks list
```

#### 1.5 Run the following command to list the available VPC subnets (sorted by VPC network):
```bash
gcloud compute networks subnets list --sort-by=<NETWORK>
```

#### 1.6 Create the firewall rules for privatenet network using the following command to create the privatenet-allow-icmp-ssh-rdp firewall rule:
```bash
gcloud compute firewall-rules create privatenet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=privatenet --action=ALLOW --rules=icmp,tcp:22,tcp:3389 --source-ranges=0.0.0.0/0
```

#### 1.7 Run the following command to list all the firewall rules (sorted by VPC network):
```bash
gcloud compute firewall-rules list --sort-by=<NETWORK>
```

## Task 2. Create VM instances

#### 2.1 Create the privatenet-vm-1 instance using the following command to create the privatenet-vm-1 instance:
```bash
gcloud compute instances create privatenet-vm-1 --zone= --machine-type=e2-micro --subnet=privatesubnet-1
```

#### 2.2 Run the following command to list all the VM instances (sorted by zone):
```bash
gcloud compute instances list --sort-by=<ZONE>
```

## Task 3. Explore the connectivity between VM instances

#### 3.1 Ping the external IP addresses of the VM instances to determine if you can reach the instances from the public internet.
#### In the Cloud console, navigate to Navigation menu > Compute Engine > VM instances.
#### Note the external IP addresses for mynet-vm-2, managementnet-vm-1, and privatenet-vm-1.
#### For mynet-vm-1, click SSH to launch a terminal and connect.
#### To test connectivity to mynet-vm-2's external IP, run the following command, replacing mynet-vm-2's external IP:
```bash
 ping -c 3 'Enter mynet-vm-2 external IP here'
```

#### To test connectivity to managementnet-vm-1's external IP, run the following command, replacing managementnet-vm-1's external IP:
```bash
 ping -c 3 'Enter managementnet-vm-1 external IP here'
```

#### To test connectivity to privatenet-vm-1's external IP, run the following command, replacing privatenet-vm-1's external IP:
```bash
ping -c 3 'Enter privatenet-vm-1 external IP here'
```

#### 3.2 Ping the internal IP addresses of the VM instances to determine if you can reach the instances from within a VPC network.
#### In the Cloud console, navigate to Navigation menu > Compute Engine > VM instances.
#### Note the internal IP addresses for mynet-vm-2, managementnet-vm-1, and privatenet-vm-1.
#### Return to the SSH terminal for mynet-vm-1.
#### To test connectivity to mynet-vm-2's internal IP, run the following command, replacing mynet-vm-2's internal IP:
```bash
ping -c 3 'Enter mynet-vm-2 internal IP here'
```

#### To test connectivity to managementnet-vm-1's internal IP, run the following command, replacing managementnet-vm-1's internal IP:
```bash
ping -c 3 'Enter privatenet-vm-1 internal IP here'
```

## Congratulations!