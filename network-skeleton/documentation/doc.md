

# **Documentation: Network Skeleton Terraform Module**

---

![image](https://github.com/user-attachments/assets/d166b852-e20c-42ca-b08c-810311d3df1b)

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
6. [Resources Required to Setup Network Skeleton](#6-resources-required-to-setup-network-skeleton)
7. [Inputs](#7-inputs)
8. [Outputs](#8-outputs)
9. [Conclusion](#9-conclusion)
10. [Contact Information](#10-contact-information)
11. [Resources](#11-resources)

---

## 1. Introduction

This Terraform module creates a complete network skeleton in AWS, including a VPC, subnets (public and private), route tables, internet gateway, NAT gateway, and associated components. It provides a reusable, modular foundation to build secure and scalable cloud infrastructure — essential for any multi-tier or microservice-based architecture.

---

## 2. Pre-requisites

| Tool      | Requirement                       |
| --------- | --------------------------------- |
| Terraform | >= v1.5.0                         |
| AWS CLI   | Installed and properly configured |

---

## 3. What Are Terraform Modules?

Terraform modules let you group infrastructure code into reusable components. They improve code structure, enforce standards, reduce duplication, and help manage complexity across large cloud deployments.

---

## 4. Usage

```hcl
provider "aws" {
  region                  = "us-east-1"
}

module "network_skeleton" {
  source                                               = "../"
  name                                                 = "network-skeleton"
  cidr_block                                           = "173.31.0.0/20"
  enable_dns_hostnames                                 = true
  enable_vpc_logs                                      = false
  public_subnets_cidr                                  = ["pvt_subnet_cidr"]
  pvt_zone_name                                        = "abc.xyz.in"
  private_subnets_cidr                                 = ["pub_subnet_cidr"]
  avaialability_zones                                  = ["avaialability_zones"]
  logs_bucket                                          = "ns-alb-logs"
  logs_bucket_arn                                      = "logs_bucket_arn"
  tags                                                 = "Additional tags"
  public_web_sg_name                                   = ns-web-sg
  alb_certificate_arn                                  = "ACM certificate arn"
  enable_igw_publicRouteTable_PublicSubnets_resource   = false
  enable_nat_privateRouteTable_PrivateSubnets_resource = false
  enable_public_web_security_group_resource            = false
  enable_pub_alb_resource                              = false
  enable_aws_route53_zone_resource                     = false
}
```

---

## 5. Tags

Tags are automatically applied to all resources for tracking, visibility, and cost management. Additional tags can be passed using the `tags` input variable.

### **Example**

```hcl
tags = {
  Name         = "dev-network"
  environment  = "dev"
  costcenter   = "otms-dev"
  application  = "otms"
  owner        = "rajeev"
}
```

---

## 6. Resources Required to Setup Network Skeleton

* **VPC** – Defines the isolated virtual network.
* **Public Subnets** – Allow internet-facing resources.
* **Private Subnets** – Host backend/internal resources securely.
* **Route Tables** – Handle routing within subnets.
* **Internet Gateway (IGW)** – Enables internet access for public subnets.
* **NAT Gateway** – Allows private subnets to access the internet securely.
* **Network Access Control Lists (NACLs)** – Stateless firewall rules applied at the subnet level to control inbound and outbound traffic.
* **Security Groups** – Stateful firewalls applied to individual AWS resources like EC2, ALB, RDS, etc., to control traffic based on rules.
* **Application Load Balancer (ALB)** – Distributes incoming traffic across multiple targets (e.g., EC2 instances, containers) in one or more availability zones.
* **Route 53 Hosted Zone** – Provides domain name management and internal/private DNS resolution for AWS resources.

---

## 7. Inputs

| Name                                                     | Description                                               | Type      | Default         | Required |
| -------------------------------------------------------- | --------------------------------------------------------- | --------- | --------------- | :------: |
| name                                                     | The string name appended in tags                          | `string`  | `"opstree"`     |    yes   |
| cidr\_block                                              | The CIDR block for the VPC. Default value is a valid CIDR | `string`  | `"10.0.0.0/24"` |    no    |
| instance\_tenancy                                        | A tenancy option for instances launched into the VPC      | `string`  | `"default"`     |    no    |
| enable\_dns\_support                                     | A DNS support for instances launched into the VPC         | `boolean` | `"true"`        |    no    |
| enable\_dns\_hostnames                                   | A DNS hostname for instances launched into the VPC        | `boolean` | `"false"`       |    no    |
| enable\_classiclink                                      | A classiclink option for VPC                              | `boolean` | `"false"`       |    no    |
| enable\_igw\_publicRouteTable\_PublicSubnets\_resource   | Creates IGW, public route table, and public subnets       | `boolean` | `"true"`        |    no    |
| enable\_nat\_privateRouteTable\_PrivateSubnets\_resource | Creates NAT, private route table, and private subnets     | `boolean` | `"true"`        |    no    |
| enable\_public\_web\_security\_group\_resource           | Creates Web Security Group                                | `boolean` | `"true"`        |    no    |
| enable\_pub\_alb\_resource                               | Creates Application Load Balancer                         | `boolean` | `"true"`        |    no    |
| enable\_aws\_route53\_zone\_resource                     | Creates Route 53 Hosted Zone                              | `boolean` | `"true"`        |    no    |

---

## 8. Outputs

| Name                  | Description                                 |
| --------------------- | ------------------------------------------- |
| vpc\_id               | The ID of the VPC                           |
| pub\_route\_table\_id | Public Route Table ID                       |
| pvt\_route\_table\_id | Private Route Table ID                      |
| pvt\_hosted\_zone\_id | Private Hosted Zone ID                      |
| pvt\_subnet\_ids      | Private Subnet IDs                          |
| public\_subnet\_ids   | Public Subnet IDs                           |
| web\_sg\_id           | Public Security Group ID                    |
| dns\_name             | DNS name of the Application Load Balancer   |
| aws\_lb\_arn          | ARN of the Application Load Balancer        |
| alb\_listener\_arn    | ARN of the primary ALB listener             |
| alb\_listener1\_arn   | ARN of the secondary ALB listener           |
| route53\_name         | Domain name of Route 53 private hosted zone |

---

## 9. Conclusion

This module provides a reliable and reusable base network infrastructure using Terraform on AWS. It’s ideal for teams building secure, scalable applications needing multiple subnets, zone redundancy, and flexible routing. With a few inputs, you can bootstrap a production-ready network in minutes.

---

## 10. Contact Information

| Name          | Email Address                                                                     |
| ------------- | --------------------------------------------------------------------------------- |
| Rajeev Ranjan | [rajeev.rajan.snaatak@mygurukulam.co](mailto:rajeev.rajan.snaatak@mygurukulam.co) |

---

## 11. Resources

| Name                                                                                                                   | Type     |
| ---------------------------------------------------------------------------------------------------------------------- | -------- |
| [aws\_vpc](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc)                            | Resource |
| [aws\_flow\_log](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/flow_log)                 | Resource |
| [aws\_internet\_gateway](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/internet_gateway) | Resource |
| [aws\_route53\_zone](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/route53_zone)         | Resource |

---

