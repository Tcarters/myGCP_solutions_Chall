# Automating Infrastructure on Google Cloud with Terraform: Challenge Lab



## Task 1. Create the configuration files


```bash
	mkdir -p modules/instances modules/storage
	
	touch main.tf variables.tf 
	
	for file in instances.tf outputs.tf variables.tf; do touch modules/instances/$file; done
	
	for file in storage.tf outputs.tf variables.tf; do touch modules/storage/$file; done
```

- Content of `main.tf`

```yaml
	terraform {
 required_providers {
        google = {
         source = "hashicorp/google"
        }
 }
}


provider "google" {
 version = "3.5.0"
 project = var.project_id
 region  = var.region
 zone    = var.zone

}

```

- Content of `variables.tf`

```yaml
	variable "project_id" {
  description = "The project ID to host the network in"
  default = "qwiklabs-gcp-04-86647ed50ccb"
}

variable "region" {
 default = "us-east1"
}

variable "zone" {
 default = "us-east1-c"
}
```


## Task 2. Import infrastructure

- Add modules `instances ` in `main.tf`

```yaml
	terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "3.55.0"
    }
  }
}

provider "google" {
  project     = var.project_id
  region      = var.region
  zone        = var.zone
}


module "instances" {
  source     = "./modules/instances"
}
```

- apply changes

` terraform init `

- Define content of `modules/instances/instances.tf `

```yaml
	resource "google_compute_instance" "tf-instance-1" {
 name           = "tf-instance-1"
 machine_type   = "e2-medium"
 
 boot_disk {
   initialize_params {
     image =    "debian-10-buster-v20230809"
     #labels
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
 name           = "tf-instance-2"
 machine_type   = "e2-medium"

 boot_disk {
   initialize_params {
     image =    "debian-10-buster-v20230809"
     #labels
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

- Import first & second instance

```bash
	 terraform import module.instances.google_compute_instance.tf-instance-1 "3767519532487765762"

	 terraform import module.instances.google_compute_instance.tf-instance-2 "2987647673051868930"
```

- Update instances

`` terraform plan && terraform apply ``



## Task 3. Configure a remote backend


## Task 4. Modify and update infrastructure

```yaml
	resource "google_compute_instance" "tf-instance-1" {
 name           = "tf-instance-1"
 machine_type   = "e2-standard-2" #"e2-medium"
 
 boot_disk {
   initialize_params {
     image =    "debian-10-buster-v20230809"
     #labels
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
 name           = "tf-instance-2"
 machine_type   = "e2-standard-2"  #"e2-medium"

 boot_disk {
   initialize_params {
     image =    "debian-10-buster-v20230809"
     #labels
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


resource "google_compute_instance" "tf-instance-958901" {
  name         = "tf-instance-958901"
  machine_type = "e2-standard-2"
  zone         = "us-east1-d"
  allow_stopping_for_update = true

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
    network = "default"
  }
  metadata_startup_script = <<-EOT
        #!/bin/bash
   EOT

}
```

- RUn : `` terraform init && terraform apply ``


## Task 5. Destroy resources


```bash
	terraform taint module.instances.google_compute_instance.tf-instance-958901
	
	terraform init
	
	terraform apply 
	
```

## Task 6. Use a module from the Registry

- New content of `main.tf`:

```yaml
	terraform {
  backend "gcs" {
        bucket = "tf-bucket-398916"
  prefix       =  "terraform/state"
  }

  required_providers {
    google = {
      source = "hashicorp/google"
      version = "4.53.0"
    }
  }
}

provider "google" {
  project     = var.project_id
  region      = var.region
  zone        = var.zone
}


module "instances" {
  source     = "./modules/instances"
}

module "storage" {
  source     = "./modules/storage"
}


module "vpc" {
    source  = "terraform-google-modules/network/google"
    version = "~> 6.0.0"

    project_id   = "qwiklabs-gcp-01-8fd4576ed87c"
    network_name = "tf-vpc-575433"
    routing_mode = "GLOBAL"

    subnets = [
        {
            subnet_name           = "subnet-01"
            subnet_ip             = "10.10.10.0/24"
            subnet_region         = "us-east1"
        },
        {
            subnet_name           = "subnet-02"
            subnet_ip             = "10.10.20.0/24"
            subnet_region         = "us-east1"
            subnet_private_access = "true"
            subnet_flow_logs      = "true"
            description           = "This subnet has a description"
        }
    ]
}
```

- Updating of `instance.tf` :

```yaml
	student_03_7fe766fbd02a@cloudshell:~ (qwiklabs-gcp-01-8fd4576ed87c)$ cat modules/instances/instances.tf 
	
resource "google_compute_instance" "tf-instance-1" {
 name           = "tf-instance-1"
 machine_type   = "e2-standard-2" #"e2-medium"
 
 boot_disk {
   initialize_params {
     image =    "debian-10-buster-v20230809"
     #labels
   }
 }

 network_interface {
   network = "tf-vpc-575433" #"default"
        subnetwork = "subnet-01"
 }

 metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
 allow_stopping_for_update = true

}

resource "google_compute_instance" "tf-instance-2" {
 name           = "tf-instance-2"
 machine_type   = "e2-standard-2"  #"e2-medium"

 boot_disk {
   initialize_params {
     image =    "debian-10-buster-v20230809"
     #labels
   }
 }

 network_interface {
   network = "tf-vpc-575433"
        subnetwork = "subnet-02"  #"default"
 }

 metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
 allow_stopping_for_update = true

}
```

## Task 7. Configure a firewall

- new content of `main.tf`

```yaml
	terraform {
  backend "gcs" {
        bucket = "tf-bucket-398916"
  prefix       =  "terraform/state"
  }

  required_providers {
    google = {
      source = "hashicorp/google"
      version = "4.53.0"
    }
  }
}

provider "google" {
  project     = var.project_id
  region      = var.region
  zone        = var.zone
}


module "instances" {
  source     = "./modules/instances"
}

module "storage" {
  source     = "./modules/storage"
}


module "vpc" {
    source  = "terraform-google-modules/network/google"
    version = "~> 6.0.0"

    project_id   = "qwiklabs-gcp-01-8fd4576ed87c"
    network_name = "tf-vpc-575433"
    routing_mode = "GLOBAL"

    subnets = [
        {
            subnet_name           = "subnet-01"
            subnet_ip             = "10.10.10.0/24"
            subnet_region         = "us-east1"
        },
        {
            subnet_name           = "subnet-02"
            subnet_ip             = "10.10.20.0/24"
            subnet_region         = "us-east1"
            subnet_private_access = "true"
            subnet_flow_logs      = "true"
            description           = "This subnet has a description"
        }
    ]
}



resource "google_compute_firewall" "tf-firewall" {
  name    = "tf-vpc-575433"
 network = "projects/qwiklabs-gcp-01-8fd4576ed87c/global/networks/tf-vpc-575433"

  allow {
    protocol = "tcp"
    ports    = ["80"]
  }

  source_tags = ["web"]
  source_ranges = ["0.0.0.0/0"]
}
```
- RUn

```bash
	terraform init && terraform apply
```

#### Final file

- Content of `main.tf`

```yaml
	
terraform {
backend "gcs" {
    bucket = "tf-bucket-280121"
prefix     = "terraform/state"
}
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "4.53.0"
    }
  }
}

provider "google" {
  project     = var.project_id
  region      = var.region
  zone        = var.zone
}

module "instances" {
  source     = "./modules/instances"
}

module "storage" {
  source     = "./modules/storage"
}

module "vpc" {
    source  = "terraform-google-modules/network/google"
    version = "~> 6.0.0"

    project_id   = "qwiklabs-gcp-02-809dddb5aefc"
    network_name = "tf-vpc-422986"
    routing_mode = "GLOBAL"

    subnets = [
        {
            subnet_name           = "subnet-01"
            subnet_ip             = "10.10.10.0/24"
            subnet_region         = "us-east1"
        },
        {
            subnet_name           = "subnet-02"
            subnet_ip             = "10.10.20.0/24"
            subnet_region         = "us-east1"
            subnet_private_access = "true"
            subnet_flow_logs      = "true"
            description           = "Network for tf-vpc-422986"
        },
    ]
}

resource "google_compute_firewall" "tf-firewall"{
  name    = "tf-firewall"
 network = "projects/qwiklabs-gcp-02-809dddb5aefc/global/networks/tf-vpc-422986"

  allow {
    protocol = "tcp"
    ports    = ["80"]
  }

  source_tags = ["web"]
  source_ranges = ["0.0.0.0/0"]
}

```

- COntent of `instances.tf`

```yaml
	resource "google_compute_instance" "tf-instance-1" {
  name         = "tf-instance-1"
  machine_type = "e2-standard-2"
  zone         = "us-east1-c"

  boot_disk {
    initialize_params {
      image = "debian-10-buster-v20230809" #"debian-cloud/debian-10"
    }
  }

  network_interface {
    network = "tf-vpc-422986" #"default"
    subnetwork = "subnet-01"
  }
  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}

resource "google_compute_instance" "tf-instance-2" {
  name         = "tf-instance-2"
  machine_type = "e2-standard-2"
  zone         =  "us-east1-c"

  boot_disk {
    initialize_params {
      image = "debian-10-buster-v20230809" #"debian-cloud/debian-10"
    }
  }

  network_interface {
    network = "tf-vpc-422986" # "default"
    subnetwork = "subnet-02"
  }
  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}




/* ############# COMMENT THIS AFTER TAINTED THE INSTANCE BEFORE DESTROYING..


resource "google_compute_instance" "tf-instance-631630" {
  name         = "tf-instance-631630"
  machine_type = "e2-standard-2"
  zone         =  "us-east1-c"

  boot_disk {
    initialize_params {
      image = "debian-10-buster-v20230809" #"debian-cloud/debian-10"
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

*/
```

- Content of `storage.tf`

```yaml
	resource "google_storage_bucket" "bucket" {
  name          = "tf-bucket-280121"
  location      = "us"
  force_destroy = true
  uniform_bucket_level_access = true
}

```
