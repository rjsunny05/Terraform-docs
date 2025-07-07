# Terraform Module: Standalone VM

![image](https://github.com/user-attachments/assets/8700e5e8-c19c-489d-a756-ce5d6c600d27)


---

## Metadata

| Author        | Created On | Version | Last Updated By | Last Edited On | Internal Reviewer | L0              | L1          | L2              |
| ------------- | ---------- | ------- | --------------- | -------------- | ----------------- | --------------- | ----------- | --------------- |
| Rajeev Ranjan | 23-06-2025 | v1.0    | -               | -              | Priyanshu         | Khushi Malhotra | Mukul Joshi | Piyush Upadhyay |

---

## Table of Contents

* [Introduction](#introduction)
* [Module Scope](#module-scope)
* [Pre-requisites](#pre-requisites)
* [Terraform Version](#terraform-version)
* [Providers](#providers)
* [Input Variables](#input-variables)
* [Resources Created](#resources-created)
* [Outputs](#outputs)
* [Module Usage](#module-usage)
* [Tags Structure](#tags-structure)
* [Contact Information](#contact-information)
* [References](#references)

---

## Introduction

This Terraform module is responsible for provisioning a **single EC2 instance** in AWS with:

* SSH access via Key Pair
* Custom Ingress Rules (port 22, 80, etc.)
* Optional Elastic IP
* Configurable tags

The goal is to provide an easy-to-use and production-ready base VM suitable for one-off use cases like monitoring, testing, jump-box, or lightweight services.

---

## Module Scope

The following AWS resources are provisioned:

| Resource Type        | Purpose                                                   |
| -------------------- | --------------------------------------------------------- |
| `aws_instance`       | Launches EC2 VM based on the chosen AMI and instance type |
| `aws_security_group` | Defines ingress access (e.g., port 22, 80, etc.)          |
| `aws_key_pair`       | Registers the SSH key for secure login                    |
| `aws_eip` (optional) | Associates a static public IP (only if configured)        |

---

## Pre-requisites

Before using this module, ensure the following:

| Tool         | Required | Notes                                |
| ------------ | -------- | ------------------------------------ |
| Terraform    | `>= 1.5` | Tested with Terraform 1.5+           |
| AWS CLI      | Yes      | Required for configuring credentials |
| SSH Key Pair | Yes      | Must exist in AWS or be created      |

---

## Terraform Version

```hcl
terraform {
  required_version = ">= 1.5.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

---

## Providers

```hcl
provider "aws" {
  region = "us-east-2"
}
```

---

## Input Variables

| Variable Name         | Type                  | Required | Default | Description                                                                                   |
| --------------------- | --------------------- | -------- | ------- | --------------------------------------------------------------------------------------------- |
| `ami_id`              | `string`              | Yes      | N/A     | AMI ID used to launch the EC2 instance.                                                       |
| `instance_type`       | `string`              | Yes      | N/A     | EC2 instance type (e.g., `t2.micro`, `t3.medium`).                                            |
| `key_name`            | `string`              | Yes      | N/A     | Name of SSH key pair for EC2 login.                                                           |
| `security_group_name` | `string`              | Yes      | N/A     | Name to assign to the security group created.                                                 |
| `ingress_rules`       | `list(object({...}))` | Yes      | N/A     | List of ingress rule maps (see example below).                                                |
| `associate_eip`       | `bool`                | No       | `false` | Whether to associate an Elastic IP.                                                           |
| `subnet_id`           | `string`              | No       | `null`  | Subnet ID where the EC2 instance will be launched. Required if not using default subnet.      |
| `vpc_id`              | `string`              | No       | `null`  | VPC ID to attach the security group. Useful for custom VPCs.                                  |
| `user_data`           | `string`              | No       | `null`  | User data script (bash/cloud-init) to run on instance launch.                                 |
| `tags`                | `map(string)`         | Yes      | `{}`    | Tags to apply to all resources. Use keys like `Name`, `environment`, `owner`, `project`, etc. |

---

## Resources Created

| Resource                           | Description                                                |
| ---------------------------------- | ---------------------------------------------------------- |
| `aws_instance`                     | EC2 instance created in the specified subnet.              |
| `aws_security_group`               | Security group allowing user-defined ingress rules.        |
| `aws_key_pair`                     | Registered key pair (if created using Terraform).          |
| `aws_eip` (optional)               | Elastic IP created and attached if `associate_eip = true`. |
| `aws_eip_association`              | Binds EIP to EC2 instance (if used).                       |
| `aws_network_interface` (optional) | Used if enhanced networking or explicit NIC configs exist. |

---

## Outputs

| Output Name           | Description                                                            |
| --------------------- | ---------------------------------------------------------------------- |
| `instance_id`         | Unique ID of the EC2 instance.                                         |
| `private_ip`          | Private IP address of the instance within the subnet/VPC.              |
| `public_ip`           | Public IP if Elastic IP is associated. Returns `null` if not assigned. |
| `instance_public_dns` | Public DNS name of the instance (if assigned a public IP).             |
| `security_group_id`   | ID of the security group attached to the instance.                     |
| `availability_zone`   | The AZ in which the instance is launched.                              |
| `tags_all`            | Consolidated map of all tags assigned to the EC2 instance.             |
| `instance_state`      | Current state of the instance (e.g., `running`, `stopped`).            |
| `key_pair_name`       | Name of the SSH key pair used to access the instance.                  |


---

## Module Usage

```hcl
provider "aws" {
  region = "us-east-2"
}

module "standalone_vm" {
  source              = "../../standalone_vm"
  ami_id              = "ami-0efc43a4067fe9a3e"
  instance_type       = "t3.medium"
  key_name            = "dev-otms-key"
  security_group_name = "dev-otms-standalone-sg"

  ingress_rules = [
    {
      from_port   = 22
      to_port     = 22
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    },
    {
      from_port   = 80
      to_port     = 80
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  ]

  tags = {
    Name         = "dev-otms-vm"
    environment  = "dev"
    costcenter   = "dev-otms"
    application  = "otms"
    owner        = "rajeev"
  }
}

output "instance_id" {
  value = module.standalone_vm.instance_id
}
```

---

## Tags Structure

All resources are tagged using a common structure passed via the `tags` input.

**Recommended Tag Keys:**

| Key           | Purpose                                |
| ------------- | -------------------------------------- |
| `Name`        | Resource identification                |
| `environment` | Used for environment-specific grouping |
| `owner`       | Cost accountability                    |
| `project`     | Useful for filtering & billing reports |
| `application` | Application component categorization   |

---

## Contact Information

| Name          | Contact                                                                           |
| ------------- | --------------------------------------------------------------------------------- |
| Rajeev Ranjan | [rajeev.rajan.snaatak@mygurukulam.co](mailto:rajeev.rajan.snaatak@mygurukulam.co) |

---

## References

| Title               | URL                                                                                                                                                                            |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Terraform EC2 Docs  | [https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance)   |
| Security Group Docs | [https://registry.terraform.io/modules/terraform-aws-modules/security-group/aws/latest](https://registry.terraform.io/modules/terraform-aws-modules/security-group/aws/latest) |
| Terraform Docs      | [https://developer.hashicorp.com/terraform/docs](https://developer.hashicorp.com/terraform/docs)                                                                               |

---

