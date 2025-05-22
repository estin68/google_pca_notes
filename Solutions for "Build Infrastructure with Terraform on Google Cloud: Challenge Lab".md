### Solutions for Build Infrastructure with Terraform on Google Cloud: Challenge Lab

#### Task 1. Create the configuration files

##### Requirements

1. In Cloud Shell, create your Terraform configuration files and a directory structure that resembles the following:

  ```plaintext
  main.tf
  variables.tf
  modules/
  └── instances
      ├── instances.tf
      ├── outputs.tf
      └── variables.tf
  └── storage
      ├── storage.tf
      ├── outputs.tf
      └── variables.tf
  ```

2. Fill out the `variables.tf` files in the root directory and within the modules. Add three variables to each file: `region`, `zone`, and `project_id`. For their default values, use , <filled in at lab start>, and your Google Cloud Project ID.

  > **Note:** You should use these variables anywhere applicable in your resource configurations.

3. Add the Terraform block and the Google Provider to the `main.tf` file. Verify the **zone** argument is added along with the **project** and **region** arguments in the Google Provider block.

4. Initialize Terraform.

##### Solutions for Task 1

###### Prerequisite

Run the following gcloud commands in Cloud Shell to set the default region and zone for your lab:

```bash
gcloud config set compute/zone "europe-west1-c"
export ZONE=$(gcloud config get compute/zone)

gcloud config set compute/region "europe-west1"
export REGION=$(gcloud config get compute/region)
```

To retrieve your Project ID, run the following command:

```bash
export MY_PROJECT_ID=$(gcloud config list --format 'value(core.project)')
```

```bash
# Create the directory structure
mkdir -p modules/instances
mkdir -p modules/storage
```

```bash
# Create root variables.tf
tee variables.tf <<EOF
variable "project_id" {
  description = "The project ID to host the network in"
  default     = "$MY_PROJECT_ID"
  type        = string

}
variable "region" {
  description = "The Google Cloud region."
  default     = "$REGION"
  type        = string
}

variable "zone" {
  description = "The Google Cloud zone."
  default     = "$ZONE"
  type        = string
}
EOF
echo "Created root variables.tf"
```

```bash
# Create main.tf and its content
tee -a main.tf  <<EOF
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "3.55.0"
    }
  }
}

provider "google" {
  project = var.project_id
  region  = var.region
  zone    = var.zone
}
EOF
```

```bash
# Create modules/instances files
touch modules/instances/instances.tf
echo "Created modules/instances/instances.tf"

touch modules/instances/outputs.tf
echo "Created modules/instances/outputs.tf"

tee modules/instances/variables.tf <<EOF
variable "project_id" {
  description = "The Google Cloud Project ID."
  type        = string
  default     = "$MY_PROJECT_ID"
}

variable "region" {
  description = "The Google Cloud region for the instances."
  type        = string
  default     = "$REGION"
}

variable "zone" {
  description = "The Google Cloud zone for the instances."
  type        = string
  default     = "$ZONE"
}
EOF
echo "Created modules/instances/variables.tf"
```

```bash
# Create modules/storage files
touch modules/storage/storage.tf
echo "Created modules/storage/storage.tf"

touch modules/storage/outputs.tf
echo "Created modules/storage/outputs.tf"

tee modules/storage/variables.tf <<EOF
variable "project_id" {
  description = "The Google Cloud Project ID."
  type        = string
  default     = "$MY_PROJECT_ID"
}

variable "region" {
  description = "The Google Cloud region for the storage resources."
  type        = string
  default     = "$REGION"
}

variable "zone" {
  description = "The Google Cloud zone (might be relevant for regional/zonal buckets or disks)."
  type        = string
  default     = "$ZONE"
}
EOF
echo "Created modules/storage/variables.tf"
```

```bash
echo "Verifying directory structure:"
ls -R

echo "Initializing Terraform..."
terraform init
```

#### Task 2. Import infrastructure

##### Solutions for Task 2

```bash
# Open Editor and append at the bottom of main.tf

# Add this module block to call your instances module
module "instances" {
  source     = "./modules/instances"
  project_id = var.project_id
  region     = var.region
  zone       = var.zone
}
# Then init againb
echo "Initializing Terraform..."
terraform init
```

```bash
# modules/instances/instances.tf

resource "google_compute_instance" "tf-instance-1" {
  zone         = var.zone
  name         = "tf-instance-1"
  machine_type = "n1-standard-1"

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

resource "google_compute_instance" "tf-instance-2" {
  zone         = var.zone
  name         = "tf-instance-2"
  machine_type = "n1-standard-1"

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

```bash
# Ensure your shell variables are set, or replace them with 
echo "Importing with Project ID: $MY_PROJECT_ID and Zone: $ZONE"

# These command worked but lab check failed
terraform import "module.instances.google_compute_instance.tf_instance_1" "$MY_PROJECT_ID/$ZONE/tf-instance-1"
terraform import "module.instances.google_compute_instance.tf_instance_2" "$MY_PROJECT_ID/$ZONE/tf-instance-2"
# These command with instance id did not work
terraform import module.instances.google_compute_instance.tf-instance-1 1293257390812098623
terraform import module.instances.google_compute_instance.tf-instance-2 591810302761300000
```

```bash
terraform plan
terraform apply
```

#### Task 3. Configure a remote backend

##### Solutions for Task 3

```bash
# Add following code to modules/storage/storage.tf file

resource "google_storage_bucket" "storage-bucket" {
  name          = "tf-bucket-804640"
  location      = "US"
  force_destroy = true
  uniform_bucket_level_access = true
}
```

```bash
# Add the following to the main.tf file:
module "storage" {
  source     = "./modules/storage"
}
```

```bash
terraform init
terraform apply
```

```bash
#  Update the main.tf file so that the terraform block 
terraform {
  backend "gcs" {
    bucket  = "tf-bucket-804640"
    prefix  = "terraform/state"
  }
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "3.55.0"
    }
  }
}
```

```bash
terraform init
terraform apply
```

#### Task 4. Modify and update infrastructure

##### Solutions for Task 4

```bash
# Update modules/instances/instances.tf 
# modify machine_type to "e2-standard-2" on both tf-instance-1 and tf-instance-2
# Add new instance with same setting as previous two

resource "google_compute_instance" "tf-instance-1" {
  zone         = var.zone
  name         = "tf-instance-1"
  machine_type = "e2-standard-2"

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

resource "google_compute_instance" "tf-instance-2" {
  zone         = var.zone
  name         = "tf-instance-2"
  machine_type = "e2-standard-2"

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

resource "google_compute_instance" "tf-instance-775247" {
  zone         = var.zone
  name         = "tf-instance-775247"
  machine_type = "e2-standard-2"

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

```bash
terraform init
terraform apply
```

#### Task 5: Taint and destroy resources

##### Solutions for Task 5

```bash
terraform taint module.instances.google_compute_instance.tf-instance-775247
```

```bash
terraform init
terraform apply
```

```bash
# Navigate to modules/instances/instances.tf remove tf-instance-3
terraform apply
```

#### TASK 6: Use a module from the Registry

##### Solutions for Task 6

```bash
module "vpc" {
    source  = "terraform-google-modules/network/google"
    version = "~> 6.0.0"

    project_id   = var.project_id
    network_name = "tf-vpc-064599"
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
            subnet_private_access = "true"
            subnet_flow_logs      = "true"
            description           = "This subnet has a description"
        }
    ]
}
```

```bash
terraform init
terraform apply
```

#### TASK 7: Configure a firewall

##### Solutions for Task 7

```bash
resource "google_compute_firewall" "tf-firewall"{
  name    = "tf-firewall"
 network = "projects/PROJECT_ID/global/networks/tf-vpc-064599"

  allow {
    protocol = "tcp"
    ports    = ["80"]
  }

  source_tags = ["web"]
  source_ranges = ["0.0.0.0/0"]
}

```
