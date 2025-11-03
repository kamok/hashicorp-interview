# Getting Started with Terraform

Terraform is an open-source tool for defining and provisioning infrastructure as code (IaC).

In this tutorial, you will:
- Install Terraform on your local machine
- Initialize a Terraform project
- Apply Terraform configuration to provision Docker infrastructure

## Prerequisites
- Command-line interface (CLI) experience
- Unix-like environment (Linux or macOS)
- Docker (version 24.0 or greater)

## Install Terraform
Visit the [Terraform installation page](https://developer.hashicorp.com/terraform/install) and select the appropriate installation method for your system. We use [Homebrew](https://brew.sh/) to install Terraform. You can use any method from the installation page.

```shell
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
```

Verify Terraform is installed correctly
```shell
$ terraform -version
Terraform v1.13.4
```

## Create your first terraform project
Create a new directory for this tutorial
```shell
$ mkdir terraform-tutorial
```
Change to the new directory
```shell
$ cd terraform-tutorial
```

### The Terraform Block
The Terraform block defines the version of terraform for the project.

Create a new file for the `terraform` block. Name the file `terraform.tf` to follow community convention.
```shell
$ touch terraform.tf
```

In the `terraform` block, version lock Terraform to at least `1.13`. Terraform uses [providers](https://developer.hashicorp.com/terraform/language/providers) to interface with cloud provider and service APIs. Configure this project to use version `3.6.2` of the `docker` provider.

```shell
terraform {
  required_providers {
    docker = {
      source = "kreuzwerker/docker"
      version = "3.6.2"
    }
  }
  required_version = ">= 1.13"
}
```

### Initialize terraform
The `terraform init` command downloads and sets up the required `providers` defined in the terraform block. The command generates a file called `.terraform.lock.hcl` to ensure integrity of providers between repeated runs.

```shell
$ terraform init
Initializing the backend...
Initializing provider plugins...
- Finding kreuzwerker/docker versions matching "~> 3.6.2"...
- Installing kreuzwerker/docker v3.6.2...
- Installed kreuzwerker/docker v3.6.2 (self-signed, key ID BD080C4571C6104C)
```

### Configure the Docker Provider and Resources

Create a file named `main.tf` with the following configuration:
```hcl
# Configure the Docker provider
provider "docker" {
  host = "unix:///var/run/docker.sock"
}

# Create an nginx container and its image
resource "docker_container" "nginx" {
  image = docker_image.nginx.image_id
  name  = "terraform_tutorial_container"
}

resource "docker_image" "nginx" {
  name = "nginx:latest"
}
```

The `provider` block configures Terraform to use your local Docker daemon. The `resource` blocks define the infrastructure you want to create: a Docker container running nginx and its corresponding image. The docker_container resource references the docker_image resource using `docker_image.nginx.image_id`.

### Apply your infrastructure change
Run `terraform apply` to begin the process. Terraform displays a plan and prompts you to confirm. Enter yes to apply the changes after you have reviewed them.

```shell
$ terraform apply
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

...

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

docker_image.nginx: Creating...
docker_image.nginx: Creation complete after 3s [id=sha256:46fabdd7f288c91a57f5d5fe12a02a41fbe855142469fcd50cbe885229064797nginx:latest]
docker_container.nginx: Creating...
docker_container.nginx: Creation complete after 0s [id=337134c2a268afcd058f628b8ca20409c86fddcfcafb58a1efe8cb423c5baa85]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

### Validate your infrastructure
List your running Docker containers.
```shell
$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS     NAMES
337134c2a268   nginx:latest   "nginx -g daemon off;"   1 minutes ago   Up 1 minutes             terraform_tutorial_container
```

## Next steps
In this tutorial, you learned how to manage infrastructure with Terraform by creating Docker resources on your local machine. Infrastructure as Code (IaC) increases deployment speed and efficiency through automation, while improving consistency and reliability by eliminating manual errors. In the next tutorial, you will learn how to safely destroy infrastructure managed by Terraform.