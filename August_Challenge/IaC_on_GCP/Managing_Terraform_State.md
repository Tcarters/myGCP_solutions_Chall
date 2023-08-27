
## Task 1. Working with backends


### Add a local backend


```bash
	touch main.tf
	
	## Get project_ID
	gcloud config list --format 'value(core.project)'
	
	## Content of main.tf
	provider "google" {
	  project     = "# REPLACE WITH YOUR PROJECT ID"
	  region      = "us-central-1"
	}
	resource "google_storage_bucket" "test-bucket-for-state" {
	  name        = "# REPLACE WITH YOUR PROJECT ID"
	  location    = "US"
	  uniform_bucket_level_access = true
	}
	
	terraform {
  	   backend "local" {
    	       path = "terraform/state/terraform.tfstate"
  	   }
	}	

```

- Examine state file

```bash
	terraform show
```


### Add a Cloud Storage backend

- Replace current local backend to gcs 

```yaml
	terraform {
  backend "gcs" {
    bucket  = "# REPLACE WITH YOUR BUCKET NAME"
    prefix  = "terraform/state"
  }
}

```

- Initialize your backend again, this time to automatically migrate the state:

```bash
	terraform init -migrate-state
	
```

### Refresh state

```bash
	terraform refresh ## update the state file by adding a label from console to file
	
	
```

### Clean up ur workspace

- revert backend to `local`

```yaml
	terraform {
  backend "local" {
    path = "terraform/state/terraform.tfstate"
  }
}

	
```

- apply changes
```bash
	terraform init -migrate-state
	
	terraform apply
	
	
```


## Task 2. Import Terraform configuration

### Create a Docker container

```bash
	docker run --name hashicorp-learn --detach --publish 8080:80 nginx:latest
```

### Import the container into Terraform

1. Cole example reposi

```bash
	git clone https://github.com/hashicorp/learn-terraform-import.git
	
	cd learn-terraform-import
	
	terraform init
	
```

> main.tf :  file configures the Docker provider.
> docker.tf file will contain the configuration necessary to manage the Docker container you created in an earlier step.


2. in `learn-terrraform-import/main.tf`

```yaml
	
```

3. Content of `docker.tf`

```yaml
	resource "docker_container" "web" {}
	
```

4. import running docker cont

```bash
	terraform import docker_container.web $(docker inspect -f {{.ID}} hashicorp-learn)
	
	### Verify container was imported
	terraform show
```

### Create configuration

```bash
	terraform plan ## will through errors
	
	## Create a new docker.tf
	
	terraform show -no-color > docker.tf
	
	
	## Run code
	
	terraform plan
```

- Update `docker.tf` with:

```bash
	# docker_container.web:
resource "docker_container" "web" {
   
    image             = "sha256:eea7b3dcba7ee47c0d16a60cc85d2b977d166be3960541991f3e6294d795ed24"
   
    name              = "hashicorp-learn"
    

    ports {
        external = 8080
        internal = 80
        ip       = "0.0.0.0"
        protocol = "tcp"
    }
}

```

- Run code: `terraform apply `


### Create image resource

```yaml
	# docker_container.web:
resource "docker_container" "web" {
   
    image             = "sha256:eea7b3dcba7ee47c0d16a60cc85d2b977d166be3960541991f3e6294d795ed24"
   
    name              = "hashicorp-learn" 

    ports {
        external = 8080
        internal = 80
        ip       = "0.0.0.0"
        protocol = "tcp"
    }
}


resource "docker_image" "nginx" {
  name   = "nginx:latest"
}


```

- Run it `terraform apply `

### Manage the container with Terraform

- New content of `docker.tf`

```yaml
	# docker_container.web:
resource "docker_container" "web" {
   
    image             = docker_image.nginx.name  #"sha256:eea7b3dcba7ee47c0d16a60cc85d2b977d166be3960541991f3e6294d795ed24"
   
    name              = "hashicorp-learn"
    
    ports {
        external = 8081
        internal = 80
        ip       = "0.0.0.0"
        protocol = "tcp"
    }
}


resource "docker_image" "nginx" {
  name   = "nginx:latest"
}
```


--------------------------------------------------------------------------🎉END-GAME🎉----------------------------------------------------------------------
