
## Task 1. Build infrastructure

1. Create the main.tf file:


```bash
	touch main.tf
	
		terraform {
	  required_providers {
	    google = {
	      source = "hashicorp/google"
	    }
	  }
	}
	provider "google" {
	  version = "3.5.0"
	  project = "<PROJECT_ID>"
	  region  = "us-central1"
	  zone    = "us-central1-c"
	}
	resource "google_compute_network" "vpc_network" {
	  name = "terraform-network"
	}
```

2. Initialize new Terraform configuration

```bash
	terraform init
```

3. Creating resources

```bash
	terraform apply
```

4. Inspect current state:

```bash
	terraform show
```


## Task 2. Change infrastructure

### Adding resources

- Add new content to `main.tf` file:

```bash
	resource "google_compute_instance" "vm_instance" {
  name         = "terraform-instance"
  machine_type = "e2-micro"
  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }
  network_interface {
    network = google_compute_network.vpc_network.name
    access_config {
    }
  }
}
```

- Applying resources `terraform apply`

- Changing resources 

```bash
	resource "google_compute_instance" "vm_instance" {
  name         = "terraform-instance"
  machine_type = "e2-micro"
  tags         = ["web", "dev"]
  # ...
}
```


- Destructive changes

> A destructive change is a change that requires the provider to replace the existing resource rather than updating it. This usually happens because the cloud provider doesn't support updating the resource in the way described by your configuration.

> Changing the disk image of your instance is one example of a destructive change.

```bash
	  boot_disk {
    initialize_params {
      image = "cos-cloud/cos-stable"
    }
  }
```

- Destroy infrastructure

```bash
	terraform destroy
```


## Task 3. Create resource dependencies


- Assigning a static IP address

```bash
	resource "google_compute_address" "vm_static_ip" {
  name = "terraform-static-ip"
}

```

- Creating a "google_compute_address" resource type. This resource type allocates a reserved IP address to your project.

```bash
	## will only show what would be changed, and never actually apply the changes directly.
	
	terraform plan 
```

- Update the `network_interface` configuration for your instance like so

```bash
	  network_interface {
    network = google_compute_network.vpc_network.self_link
    access_config {
      nat_ip = google_compute_address.vm_static_ip.address
    }
  }
```

- Save plan 

```bash
	terraform plan -out static_ip
```

- Run terraform apply "static_ip" to see how Terraform plans to apply this change:

```bash
	terraform apply "static_ip"
	
```

### Implicit and explicit dependencies

1. Add a Cloud Storage bucket and an instance with an explicit dependency on the bucket by adding the following to main.tf

```bash
	# New resource for the storage bucket our application will use.
resource "google_storage_bucket" "example_bucket" {
  name     = "<UNIQUE-BUCKET-NAME>"
  location = "US"
  website {
    main_page_suffix = "index.html"
    not_found_page   = "404.html"
  }
}
# Create a new instance that uses the bucket
resource "google_compute_instance" "another_instance" {
  # Tells Terraform that this VM instance must be created only after the
  # storage bucket has been created.
  depends_on = [google_storage_bucket.example_bucket]
  name         = "terraform-instance-2"
  machine_type = "e2-micro"
  boot_disk {
    initialize_params {
      image = "cos-cloud/cos-stable"
    }
  }
  network_interface {
    network = google_compute_network.vpc_network.self_link
    access_config {
    }
  }
}
```


## Task 4. Provision infrastructure

### Defining a provisioner

1. define a provisioner, modify the resource block defining the first vm_instance in your configuration to look like the following:

```bash
	resource "google_compute_instance" "vm_instance" {
  name         = "terraform-instance"
  machine_type = "e2-micro"
  tags         = ["web", "dev"]
  provisioner "local-exec" {
    command = "echo ${google_compute_instance.vm_instance.name}:  ${google_compute_instance.vm_instance.network_interface[0].access_config[0].nat_ip} >> ip_address.txt"
  }
  # ...
}


	terraform apply 
```

2. Use terraform taint to tell Terraform to recreate the instance

```bash
	terraform taint google_compute_instance.vm_instance
	
	## A tainted resource will be destroyed and recreated during the next apply...
```




--------------------------------------------------------------------------🎉END-GAME🎉----------------------------------------------------------------------

