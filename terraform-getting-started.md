# Getting Started with Terraform

Terraform is an open-source tool for defining and provisioning infrastructure as code (IaC).

In this tutorial, you will:
- Install Terraform on your local machine
- Initialize a Terraform project
- Apply Terraform configuration to provision Docker infrastructure
- Destroy the infrastructure with Terraform

## Prerequisites
- Command-line interface (CLI) experience
- Unix-like environment (Linux or macOS)

## Install Terraform
Visit the [Terraform installation page](https://developer.hashicorp.com/terraform/install) and select the appropriate installation method for your system. We will use the package manager [homebrew](https://brew.sh/) to install Terraform but you may choose whichever method you like.

```shell
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
```

Verify the successful installation of Terraform your machine
```shell
$ terraform -version
Terraform v1.13.4
```

## Create your first terraform project
Create a new directory for this tutorial
```shell
$ mkdir terraform-tutorial
```
Change your current directory to the newly created directory
```shell
$ cd terraform-tutorial
```

### The Terraform Block
The Terraform block defines the version of terraform for the project.

Create a new file for the terraform block. Conventionally we recommend naming this file `terraform.tf`
```shell
$ touch terraform.tf
```

In this `terraform` block, we version lock Terraform to at least `1.13`. Terraform uses [providers](https://developer.hashicorp.com/terraform/language/providers) to interface with cloud provider and service APIs. We are configuring this project to use version `3.6.2` of the `docker` provider.

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
The `terraform init` command downloads, installs, and validates te required `providers` according to the terraform block. The command also generates a file called `.terraform.lock.hcl` to ensure integrity of providers between repeated runs.

```shell
$ terraform init
Initializing the backend...
Initializing provider plugins...
- Finding kreuzwerker/docker versions matching "~> 3.6.2"...
- Installing kreuzwerker/docker v3.6.2...
- Installed kreuzwerker/docker v3.6.2 (self-signed, key ID BD080C4571C6104C)
```

### Create the docker provider
Create a new file for the provider block. Conventionally we recommend naming this file `main.tf`.
```shell
$ touch main.tf
```

We recommend that your provider blocks be defined in a single file. In this block, we define two new `resource`, a docker container and a docker image.
```hcl
provider "docker" {
  host = "unix:///var/run/docker.sock"
}
resource "docker_container" "nginx" {
  image = docker_image.nginx.image_id
  name  = "training"
  ports {
    internal = 80
    external = 80
  }
}
resource "docker_image" "nginx" {
  name = "nginx:latest"
}
```

Initialize Terraform with the `init` command. The AWS provider will be installed. 

```shell
$ terraform init
```

You shoud check for any errors. If it ran successfully, provision the resource with the `apply` command.

```shell
$ terraform apply
```

The command will take up to a few minutes to run and will display a message indicating that the resource was created.

Finally, destroy the infrastructure.

```shell
$ terraform destroy
```

Look for a message are the bottom of the output asking for confirmation. Type `yes` and hit ENTER. Terraform will destroy the resources it had created earlier.
