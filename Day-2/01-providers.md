# Providers 

A provider in Terraform is a plugin that enables interaction with an API. 
This includes cloud providers, SaaS providers, and other APIs. The providers are specified in the Terraform configuration code. They tell Terraform which services it needs to interact with.

For example, if you want to use Terraform to create a virtual machine on AWS, you would need to use the aws provider. The aws provider provides a set of resources that Terraform can use to create, manage, and destroy virtual machines on AWS.

Here is an example of how to use the aws provider in a Terraform configuration:

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "example" {
  ami = "ami-0123456789abcdef0" # Change the AMI 
  instance_type = "t2.micro"
}
```

In this example, we are first defining the aws provider. We are specifying the region as us-east-1. Then, we are defining the `aws_instance` resource. We are specifying the `AMI ID` and the `instance type`.

When Terraform runs, it will first install the aws provider. Then, it will use the aws provider to create the virtual machine.

Here are some other examples of providers:

- `azurerm` - for Azure
- `google` - for Google Cloud Platform
- `kubernetes` - for Kubernetes
- `openstack` - for OpenStack
- `vsphere` - for VMware vSphere

There are many other providers available, and new ones are being added all the time.

Providers are an essential part of Terraform. They allow Terraform to interact with a wide variety of cloud providers and other APIs. This makes Terraform a very versatile tool that can be used to manage a wide variety of infrastructure.


## Different ways to configure providers in terraform

There are three main ways to configure providers in Terraform:

### In the root module(Root module

The root module is the directory where you run terraform init/plan/apply. All the .tf files directly in that folder together form the root module. Files in subfolders are not included.

It is where you:

configure providers
configure the backend, i.e. where the state file lives
call child modules and supply their inputs (often from .tfvars or TF_VAR_*)

Each root module has its own state file.) 

This is the most common way to configure providers. The provider configuration block is placed in the root module of the Terraform configuration. This makes the provider configuration available to all the resources in the configuration.

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "example" {
  ami = "ami-0123456789abcdef0"
  instance_type = "t2.micro"
}
```

### In a child module(A child module is any module that a root module (or another module) calls with a module block. It’s a reusable package of resources, much like a function:

Inputs are variable blocks (the function’s parameters).
Outputs are output blocks (its return values).
It has no provider config and no backend. It uses whatever provider the caller passes in (yesterday’s providers = {...}).
Its resources are stored in the caller’s state, under addresses like module.cp_nodes.vsphere_virtual_machine.this["cp1"].)
iac-lab/terraform/
├── modules/
│   └── vm/                  ← CHILD module (never run terraform here)
│       ├── main.tf            vsphere_virtual_machine resource
│       ├── variables.tf       name, cpu, memory, ip, template, network...
│       └── outputs.tf         ip, vm_id
└── envs/
    └── lab/                 ← ROOT module (cd here, run terraform)
        ├── versions.tf        required_providers, backend
        ├── providers.tf       provider "vsphere" {}
        ├── main.tf            module "cp_nodes" { source = "../../modules/vm" ... }
        └── lab.tfvars         actual values (not committed if it holds secrets)

You can also configure providers in a child module. This is useful if you want to reuse the same provider configuration in multiple resources.

```hcl
module "aws_vpc" {
  source = "./aws_vpc"
  providers = {
    aws = aws.us-west-2
  }
}

resource "aws_instance" "example" {
  ami = "ami-0123456789abcdef0"
  instance_type = "t2.micro"
  depends_on = [module.aws_vpc]
}
```

### In the required_providers block

You can also configure providers in the required_providers block. This is useful if you want to make sure that a specific provider version is used.

```hcl
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "~> 3.79"
    }
  }
}

resource "aws_instance" "example" {
  ami = "ami-0123456789abcdef0"
  instance_type = "t2.micro"
}
```

The best way to configure providers depends on your specific needs. If you are only using a single provider, then configuring it in the root module is the simplest option. If you are using multiple providers, or if you want to reuse the same provider configuration in multiple resources, then configuring it in a child module is a good option. And if you want to make sure that a specific provider version is used, then configuring it in the required_providers block is the best option.
