# **Documentation: Network Skeleton Terraform Module**

---

| Author        | Created On | Version | Last Updated By | Last Edited On | Internal Reviewer | L0              | L1          | L2              |
| ------------- | ---------- | ------- | --------------- | -------------- | ----------------- | --------------- | ----------- | --------------- |
| Rajeev Ranjan | 23-06-2025 | v1.0    | N/A             | N/A            | Priyanshu         | Khushi Malhotra | Mukul Joshi | Piyush Upadhyay |

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Pre-requisites](#2-pre-requisites)
3. [What Are Terraform Modules?](#3-what-are-terraform-modules)
4. [Usage](#4-usage)
5. [Tags](#5-tags)
6. [Resources Created by the Module](#6-resources-created-by-the-module)
7. [Inputs](#7-inputs)
8. [Outputs](#8-outputs)
9. [Conclusion](#9-conclusion)
10. [Contact Information](#10-contact-information)
11. [Resources](#11-resources)

---

## 1. Introduction

This Terraform module provisions a fully functional AWS network skeleton including VPC, subnets (public/private), route tables, Internet/NAT gateways, NACLs, security groups, VPC peering (optional), Application Load Balancer (optional), and Route53 DNS (optional). It's designed to provide a standardized, reusable infrastructure layer for secure and scalable application deployment.

---

## 2. Pre-requisites

| Tool      | Requirement                       |
| --------- | --------------------------------- |
| Terraform | >= v1.5.0                         |
| AWS CLI   | Installed and properly configured |

---

## 3. What Are Terraform Modules?

Terraform modules are self-contained packages of Terraform configurations that are managed as a group. This module helps simplify infrastructure as code by providing consistent and reusable network skeletons for all environments (dev/stage/prod).

---

## 4. Usage

```hcl
provider "aws" {
  region = "us-east-2"
}

module "network_skeleton" {
  source             = "../network-skeleton"
  region             = "us-east-2"
  env                = "dev"
  project_name       = "otms"
  owner              = "aditya"
  app_name           = "otms"
  costcenter         = "dev-cc"

  vpc_cidr           = "192.168.0.0/24"
  subnet_names       = ["public", "frontend", "application", "database", "public-2"]
  subnet_cidrs       = ["192.168.0.0/28", "192.168.0.16/28", "192.168.0.64/27", "192.168.0.48/28", "192.168.0.32/28"]
  subnet_azs         = ["us-east-2a", "us-east-2a", "us-east-2a", "us-east-2a", "us-east-2b"]

  public_subnet_indexes = [0, 4]
  Eip_Domain             = "vpc"

  public_route_table     = "public"
  private_route_table    = "private"
  public_rt_cidr_block   = "0.0.0.0/0"
  private_rt_cidr_block  = "0.0.0.0/0"

  nacl_names              = ["public", "frontend", "application", "database"]
  create_nacl             = true

  sg_names                = ["wireguardvpn", "alb", "frontend", "attendance", "employee", "salary", "postgresql", "redis", "scylla"]
  create_sg               = true

  peering_connection      = true
  vpc_accept              = true
  manage_vpc              = "mgmt-vpc"
  public_rt_name          = "mgmt-public"
  private_rt_name         = "mgmt-private"

  alb_name                = "alb"
  create_alb              = true
  lb_internal             = true
  lb_type                 = "application"
  lb_enable_deletion      = false

  create_route53          = true
  record_name             = "www.cloudninja.live"
  record_type             = "CNAME"
}
```

---

## 5. Tags

Standard AWS tags are added to every resource for accountability and billing purposes.

### **Example:**

```hcl
tags = {
  env         = "dev"
  owner       = "aditya"
  application = "otms"
  costcenter  = "dev-cc"
}
```

---

## 6. Resources Created by the Module

* VPC with DNS support
* Subnets (public and private)
* Internet Gateway
* Elastic IP
* NAT Gateway
* Public and Private Route Tables
* NACLs with ingress/egress rules
* Security Groups and SG rules
* VPC Peering with routes to managed VPC (optional)
* Application Load Balancer (internal or public)
* Route53 DNS Record (public zone)

---

## 7. Inputs

Detailed list of inputs is available in `variables.tf`. Key parameters include:

| Name                     | Type   | Description                                  | Default                                             |
| ------------------------ | ------ | -------------------------------------------- | --------------------------------------------------- |
| env                      | string | Environment (dev/prod/test)                  | "dev"                                               |
| project\_name            | string | Project name prefix                          | "otms"                                              |
| region                   | string | AWS region                                   | "us-east-2"                                         |
| owner                    | string | Resource owner                               | "aditya"                                            |
| app\_name                | string | Application name tag                         | ""                                                  |
| costcenter               | string | Cost center tag                              | ""                                                  |
| vpc\_cidr                | string | CIDR for VPC                                 | "192.168.0.0/24"                                    |
| enable\_dns\_support     | bool   | Enable DNS support                           | true                                                |
| enable\_dns\_hostnames   | bool   | Enable DNS hostnames                         | true                                                |
| instance\_tenancy        | string | Instance tenancy                             | "default"                                           |
| subnet\_names            | list   | Names of subnets                             | -                                                   |
| subnet\_cidrs            | list   | CIDR blocks for subnets                      | -                                                   |
| subnet\_azs              | list   | Availability zones for subnets               | -                                                   |
| public\_subnet\_indexes  | list   | Indexes of public subnets                    | \[0]                                                |
| public\_route\_table     | string | Public route table name                      | "public"                                            |
| private\_route\_table    | string | Private route table name                     | "private"                                           |
| public\_rt\_cidr\_block  | string | Destination CIDR for public route            | "0.0.0.0/0"                                         |
| private\_rt\_cidr\_block | string | Destination CIDR for private route           | "0.0.0.0/0"                                         |
| Eip\_Domain              | string | Domain for Elastic IP                        | "vpc"                                               |
| nacl\_names              | list   | Names for NACLs                              | -                                                   |
| nacl\_rules              | map    | Rules for NACLs                              | -                                                   |
| create\_nacl             | bool   | Whether to create NACLs                      | true                                                |
| sg\_names                | list   | Names for Security Groups                    | -                                                   |
| security\_groups\_rule   | map    | Rules for Security Groups                    | -                                                   |
| create\_sg               | bool   | Whether to create Security Groups            | true                                                |
| alb\_name                | string | Name of the ALB                              | "alb"                                               |
| lb\_internal             | bool   | Set ALB to internal (true) or public (false) | true                                                |
| lb\_type                 | string | Type of Load Balancer                        | "application"                                       |
| lb\_enable\_deletion     | bool   | Enable ALB deletion protection               | false                                               |
| create\_alb              | bool   | Whether to create ALB                        | true                                                |
| create\_route53          | bool   | Whether to create Route53 record             | true                                                |
| record\_name             | string | DNS record name                              | "[www.cloudninja.live](http://www.cloudninja.live)" |
| record\_type             | string | DNS record type                              | "CNAME"                                             |
| peering\_connection      | bool   | Enable VPC peering                           | true                                                |
| vpc\_accept              | bool   | Whether the VPC accepts the peering          | true                                                |
| manage\_vpc              | string | Name of the VPC to peer with                 | ""                                                  |
| public\_rt\_name         | string | Name tag of the public route table           | ""                                                  |
| private\_rt\_name        | string | Name tag of the private route table          | ""                                                  |

---

## 8. Outputs

| Name            | Description                               |
| --------------- | ----------------------------------------- |
| vpc\_id         | ID of the created VPC                     |
| subnet\_ids     | Map of subnet names to subnet IDs         |
| igw\_id         | Internet Gateway ID                       |
| NAT\_Gateway    | NAT Gateway ID                            |
| public\_rt\_id  | Route Table ID for public subnets         |
| private\_rt\_id | Route Table ID for private subnets        |
| all\_sg\_ids    | Map of SG names to Security Group IDs     |
| alb\_arn        | ARN of the Application Load Balancer      |
| alb\_dns        | DNS Name of the Application Load Balancer |

---

## 9. Conclusion

This module abstracts the complexity of AWS networking and provides a clean interface for creating a production-grade network foundation. It supports modular extensions like peering, load balancing, DNS, and secure subnet isolation — ideal for modern cloud-native apps.

---

## 10. Contact Information

| Name          | Email Address                                                                     |
| ------------- | --------------------------------------------------------------------------------- |
| Rajeev Ranjan | [rajeev.rajan.snaatak@mygurukulam.co](mailto:rajeev.rajan.snaatak@mygurukulam.co) |

---

## 11. Resources

| Name                                                                                                                                | Type     |
| ----------------------------------------------------------------------------------------------------------------------------------- | -------- |
| [aws\_vpc](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc)                                         | Resource |
| [aws\_subnet](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/subnet)                                   | Resource |
| [aws\_internet\_gateway](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/internet_gateway)              | Resource |
| [aws\_nat\_gateway](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/nat_gateway)                        | Resource |
| [aws\_lb](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/lb)                                           | Resource |
| [aws\_route53\_record](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/route53_record)                  | Resource |
| [aws\_security\_group](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group)                  | Resource |
| [aws\_network\_acl](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/network_acl)                        | Resource |
| [aws\_vpc\_peering\_connection](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc_peering_connection) | Resource |

---
