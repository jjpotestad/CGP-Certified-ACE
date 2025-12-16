# Build Infrastructure with Terraform on Google Cloud: Challenge Lab

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
export ZONE="us-east1-c"
```

#### In Cloud Shell, set the default region:
```bash
gcloud config set compute/region $REGION
```

#### Set the default zone:
```bash
gcloud config set compute/zone $ZONE
```

## Task 1. Create the configuration files
#### 1.1 Create your Terraform configuration files and a directory structure
```bash
touch main.tf variables.tf
mkdir -p modules/instances modules/storage
cd modules/instances
touch instances.tf outputs.tf variables.tf
cd ../storage
touch storage.tf outputs.tf variables.tf
```

#### 1.2 Fill out the variables.tf files in the root directory and within the modules. Add three variables to each file: region, zone, and project_id
``` json
variable "project_id" {
  description = "The ID of the project in which to provision resources."
  type        = string
  default     = "qwiklabs-gcp-01-74160bf2f551"
}

variable "region" {
  description = "The region in GCP."
  type        = string
  default     = "us-east1"
}

variable "zone" {
  description = "The zone in GCP."
  type        = string
  default     = "us-east1-c"
}
```

#### 1.3 Add the Terraform block and the Google Provider to the <b>main.tf</b> file. Verify the zone argument is added along with the project and region arguments in the Google Provider block.
``` json
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
      version = ">=4.64.0"
    }
  }
}

provider "google" {
  project = var.project_id
  region  = var.region
  zone    = var.zone
}
```
#### 1.4 Initialize Terraform.
```bash
terraform init
```

## Task 2. Import infrastructure
#### 2.1 In the Google Cloud Console, on the Navigation menu, click Compute Engine > VM Instances. Two instances named <b>tf-instance-1</b> and <b>tf-instance-2</b> have already been created for you.

#### 2.2 Import the existing instances into the instances module. Edit <b>modules/instances/instances.tf</b> and paste
``` json
# Config resource for tf-instance-1
resource "google_compute_instance" "tf-instance-1" {
  name         = "tf-instance-1"
  machine_type = "e2-micro"
  zone         = var.zone
  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }
  network_interface {
    network = "default"
  }
  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}

# Config resource for tf-instance-2
resource "google_compute_instance" "tf-instance-2" {
  name         = "tf-instance-2"
  machine_type = "e2-micro"
  zone         = var.zone
  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }
  network_interface {
    network = "default"
  }
  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}
```

#### 2.3 Import instances module in the <b>main.tf</b> file.
``` json
# Import instances module
module "instances" {
  source     = "./modules/instances"
  project_id = var.project_id
  region     = var.region
  zone       = var.zone
}
```
####  Initialize Terraform.
```bash
terraform init
```

#### 2.4 Use Terraform import command for the instances
```bash
terraform import module.instances.google_compute_instance.tf-instance-1 projects/$DEVSHELL_PROJECT_ID/zones/$ZONE/instances/tf-instance-1

terraform import module.instances.google_compute_instance.tf-instance-2 projects/$DEVSHELL_PROJECT_ID/zones/$ZONE/instances/tf-instance-2
```

#### Apply your changes
```bash
terraform plan
terraform apply
# Answer question always YES
```

## Task 3. Configure a remote backend
#### 3.1 Create a Cloud Storage bucket resource inside the storage module. Edit <b>modules/storage/storage.tf</b> and paste
``` json
resource "google_storage_bucket" "storage-bucket" {
  name          = "tf-bucket-297890"
  location      = "US"
  force_destroy = true
  uniform_bucket_level_access = true
}

```

#### 3.2 Add the module reference to the <b>main.tf</b> file. Initialize the module and apply the changes to create the bucket using Terraform.
``` json
# Import the storage module
module "storage" {
    source     = "./modules/storage"
    project_id = var.project_id
    region     = var.region
    zone       = var.zone
}
```
####  Initialize Terraform. Check plan and apply your changes
```bash
terraform init
terraform plan
terraform apply
# Answer question always YES
```
#### 3.3 Configure this storage bucket as the remote backend inside the <b>main.tf</b> file. Be sure to use the prefix <b>terraform/state</b> so it can be graded successfully.
``` json
terraform {
  # Import the terraform bucket state
  backend "gcs" {
    bucket  = "tf-bucket-297890"
    prefix  = "terraform/state"
  }
// ...
}
```
#### After write the configuration, upon init. Terraform will ask whether you want to copy the existing state data to the new backend
```bash
terraform init
# Answer question always YES
```

## Task 4. Modify and update infrastructure
#### 4.1 Edit <b>modules/instances/instances.tf</b> to change machine_type and add third instances resource.
``` json
# Config resource for tf-instance-1
resource "google_compute_instance" "tf-instance-1" {
  name         = "tf-instance-1"
  machine_type = "e2-standard-2"
  zone         = var.zone
  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }
  network_interface {
    network = "default"
  }
  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}

# Config resource for tf-instance-2
resource "google_compute_instance" "tf-instance-2" {
  name         = "tf-instance-2"
  machine_type = "e2-standard-2"
  zone         = var.zone
  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }
  network_interface {
    network = "default"
  }
  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}

# Config resource for the third instance
resource "google_compute_instance" "tf-instance-597586" {
  name         = "tf-instance-597586"
  machine_type = "e2-standard-2"
  zone         = var.zone
  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }
  network_interface {
    network = "default"
  }
  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}
```
#### 4.1 Apply your changes
```bash
terraform apply
# Answer question always YES
```

## Task 5. Destroy resources
#### Destroy the third instance by removing the resource from the configuration file. After removing it, initialize terraform and apply the changes.
#### 5.1 Edit <b>modules/instances/instances.tf</b> to remove third instances resource.
#### 5.2 Apply your changes
```bash
terraform apply
# Answer question always YES
```

## Task 6. Use a module from the Registry
#### 6.1 Add the [Network Module](https://registry.terraform.io/modules/terraform-google-modules/network/google/10.0.0) module to your <b>main.tf</b> file
``` json

# Network Module
  module "vpc" {
    source  = "terraform-google-modules/network/google"
    version = "~> 10.0.0"

    project_id   = var.project_id
    network_name = "tf-vpc-761569"
    routing_mode = "GLOBAL"

    subnets = [
        {
            subnet_name           = "subnet-01"
            subnet_ip             = "10.10.10.0/24"
            subnet_region         = var.region
        },
        {
            subnet_name           = "subnet-02"
            subnet_ip             = "10.10.20.0/24"
            subnet_region         = var.region
        }
    ]
  }
// ...
```

#### 6.2 Once you've written the module configuration, initialize Terraform and run an apply to create the networks.
```bash
terraform init
terraform apply
# Answer question always YES
```

#### 6.3 Next, navigate to the <b>modules/instances/instances.tf</b>  file and update the configuration resources to connect <b>tf-instance-1</b> to <b>subnet-01</b> and <b>tf-instance-2</b> to <b>subnet-02</b>.
``` json
# Config resource for tf-instance-1
resource "google_compute_instance" "tf-instance-1" {
  name         = "tf-instance-1"
  machine_type = "e2-standard-2"
  zone         = var.zone
  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }
  network_interface {
    network = "tf-vpc-761569"
    subnetwork = "subnet-01"
  }
  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}

# Config resource for tf-instance-2
resource "google_compute_instance" "tf-instance-2" {
  name         = "tf-instance-2"
  machine_type = "e2-standard-2"
  zone         = var.zone
  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }
  network_interface {
    network = "tf-vpc-761569"
    subnetwork = "subnet-02"
  }
  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}
```

#### 6.4 Apply your changes
```bash
terraform apply
# Answer question always YES
```

## Task 7. Configure a firewall
#### 7.1 Create a firewall rule resource in the main.tf file, and name it tf-firewall.
``` json
# Add firewall rule for the tf-vpc
resource "google_compute_firewall" "tf-firewall" {
  name    = "tf-firewall"
  network = "projects/${var.project_id}/global/networks/tf-vpc-761569"

  allow {
    protocol = "tcp"
    ports    = ["80"]
  }

  source_ranges = ["0.0.0.0/0"]
}
```
#### 7.2 Apply your changes
```bash
terraform apply
# Answer question always YES
```
