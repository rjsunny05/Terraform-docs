
# **Documentation on Designing Auto-Scaling Module**

![image](https://github.com/user-attachments/assets/f029121f-7f7a-4823-9cf1-a6447b0de14f)


| Author        | Created on | Version   | Last updated by | Last edited on | Internal reviewer | L0              | L1          | L2              |
| ------------- | ---------- | --------- | --------------- | -------------- | ----------------- | --------------- | ----------- | --------------- |
| Rajeev Ranjan | 23-06-2025 | version 1 | N/A             | N/A            | Priyanshu         | Khushi Malhotra | Mukul Joshi | Piyush Upadhyay |

---

## **Table of Contents**

1. [Introduction](#1-introduction)
2. [Resources Required to Setup Auto Scaling Module](#2-resources-required-to-setup-auto-scaling-module)
3. [Inputs](#3-inputs)
4. [Outputs](#4-outputs)
5. [How it Works](#5-how-it-works)
6. [Example Usage](#6-example-usage)
7. [Tags](#7-tags)
8. [AWS Resources Overview](#8-aws-resources-overview)
9. [Conclusion](#9-conclusion)
10. [Contact Information](#10-contact-information)
11. [References](#11-references)

---

## **1. Introduction**

The **Auto-Scaling Module** dynamically manages EC2 instance counts in response to demand using **Launch Templates**, **Auto Scaling Groups**, **ALB Target Groups**, and **Scaling Policies**. This setup ensures both **high availability** and **cost-efficiency**.

---

## **2. Resources Required to Setup Auto Scaling Module**

| Resource Type                       | Count | Description                                                                                        |
| ----------------------------------- | ----- | -------------------------------------------------------------------------------------------------- |
| **Security Group**                  | 1     | Allows ingress/egress rules for EC2 (e.g., allow HTTP, restrict SSH).                              |
| **Subnet**                          | 2+    | Used to launch EC2 instances and ALB across multiple Availability Zones.                           |
| **Application Load Balancer (ALB)** | 1     | Distributes incoming traffic to target groups (and hence EC2 instances).                           |
| **Launch Template**                 | 1     | Blueprint for EC2 instance configuration (AMI, instance type, key, SG, subnet, etc.).              |
| **Auto Scaling Group**              | 1     | Automatically scales instances in/out based on performance metrics.                                |
| **Target Group**                    | 1     | Registered with Load Balancer — routes traffic only to healthy instances.                          |
| **Scaling Policies**                | 2     | One for scale-out (add EC2), one for scale-in (remove EC2).                                        |
| **ALB Listener Rule**               | 1     | Determines how traffic gets routed from ALB to Target Group based on conditions like path-pattern. |
| **IAM Role (Optional)**             | 1     | Enables EC2 to access AWS services securely if required.                                           |


---

## **3. Inputs**

| Variable Name           | Type   | Required | Description                                                     |
| ----------------------- | ------ | -------- | --------------------------------------------------------------- |
| `ami_id`                | string | Yes      | AMI ID to use for EC2 instances                                 |
| `instance_type`         | string | Yes      | Type of EC2 instance (e.g., t2.micro, t3.small)                 |
| `security_group_ids`    | list   | Yes      | List of security group IDs to associate with Launch Template    |
| `key_name`              | string | Yes      | SSH key name used to access EC2 instances                       |
| `vpc_zone_identifier`   | list   | Yes      | List of subnet IDs for Auto Scaling Group deployment            |
| `alb_subnet_ids`        | list   | Yes      | List of subnet IDs for ALB deployment across Availability Zones |
| `alb_security_group_id` | string | Yes      | Security Group ID for ALB                                       |
| `target_group_arn`      | string | Yes      | ARN of the target group for ALB to forward traffic              |
| `desired_capacity`      | number | Yes      | Desired number of EC2 instances                                 |
| `min_size`              | number | Yes      | Minimum number of EC2 instances                                 |
| `max_size`              | number | Yes      | Maximum number of EC2 instances                                 |
| `listener_arn`          | string | Yes      | ARN of the ALB listener                                         |
| `path_pattern`          | string | Yes      | Path pattern to route traffic in listener rule (e.g., /api/\*)  |
| `health_check_path`     | string | No       | Custom health check path                                        |
| `enable_autoscaling`    | bool   | No       | Toggle to enable or disable autoscaling                         |
| `env`                   | string | Yes      | Environment tag value (e.g., dev, prod)                         |
| `project_name`          | string | Yes      | Name of the project for tagging                                 |
| `application_name`      | string | Yes      | Name of the application for tagging                             |
| `costcenter`            | string | Yes      | Cost allocation tag                                             |
| `owner`                 | string | Yes      | Owner tag for traceability                                      |



---

## **4. Outputs**

| Output Name              | Description                                                                |
| ------------------------ | -------------------------------------------------------------------------- |
| `launch_template_name`   | Human-readable name of the launch template (not just ID)                   |
| `autoscaling_group_name` | Name of the Auto Scaling Group (useful for logging, monitoring, or alerts) |
| `target_group_name`      | Name of the Target Group                                                   |
| `alb_listener_arn`       | Full ARN of the ALB listener (not just rule)                               |
| `scaling_policy_names`   | List of names of the scaling policies (scale up/down)                      |


---

## **5. How it Works**

1. **Security Group** defines traffic rules for EC2.
2. **Launch Template** sets up EC2 with parameters like AMI, type, key, and network.
3. **Auto Scaling Group (ASG)** reads this template and launches instances across subnets.
4. ASG is tied to a **Target Group**, which is attached to the **ALB** for health-check–based routing.
5. **Scaling Policies** ensure scaling happens based on load — like CPU or traffic.
6. **Listener Rules** route traffic to this Target Group based on `/path-pattern` conditions.

This setup enables:

* Self-healing EC2 infrastructure
* Load-aware scaling
* Seamless routing through ALB

---

## **6. Example Usage**

```hcl
module "auto_scaling" {
  source                      = "git::https://github.com/Cloud-NInja-snaatak/terraform-modules.git//auto-scalable"
  ami_id                      = "ami-0c55b159cbfafb09"
  instance_type               = "t2.micro"
  key_name                    = "dev-otms-key"
  security_group_ids          = [module.network.sg_id]
  vpc_zone_identifier         = module.network.public_subnets
  desired_capacity            = 2
  min_size                    = 1
  max_size                    = 4
  target_group_arn            = module.alb.target_group_arn
  env                         = "dev"
  project_name                = "otms"
  application_name            = "notification"
  owner                       = "Rajeev"
  costcenter                  = "dev-otms"
}
```

---

## **7. Tags**

Used to organize resources.

```hcl
tags = {
  Name             = "dev-otms-auto-scaling"
  environment      = "dev"
  project_name     = "otms"
  application_name = "notification"
  costcenter       = "dev-otms"
  owner            = "Rajeev"
}
```

---

## **8. AWS Resources Overview**

| Name                 | Description                                                                 |
| -------------------- | --------------------------------------------------------------------------- |
| `asg_id`             | Auto Scaling Group ID – manages dynamic EC2 scaling                         |
| `launch_template_id` | Blueprint for launching EC2 instances                                       |
| `target_group_arn`   | Target Group for routing traffic via ALB                                    |
| `scaling_policy_id`  | Controls how and when scaling occurs based on traffic/CPU metrics           |
| `listener_rule_id`   | ALB listener rule that maps incoming traffic to the correct service path    |
| `security_group_id`  | Security Group controlling access to EC2 (e.g., SSH, HTTP, internal access) |

---

## **9. Conclusion**

This module simplifies setting up dynamic, scalable infrastructure in AWS. It ensures that resources grow when needed and shrink during idle periods — optimizing both **performance** and **cost**. The design promotes **fault-tolerance**, **automation**, and **clean separation of environments**.

---

## **10. Contact Information**

| Name          | Email Address                                                                     |
| ------------- | --------------------------------------------------------------------------------- |
| Rajeev Ranjan | [rajeev.rajan.snaatak@mygurukulam.co](mailto:rajeev.rajan.snaatak@mygurukulam.co) |

---

## **11. References**

| Links                                                                                                                                   |
| --------------------------------------------------------------------------------------------------------------------------------------- |
| [AWS Auto Scaling Group](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-autoscaling-autoscalinggroup.html) |
| [Attach Load Balancer to ASG](https://docs.aws.amazon.com/autoscaling/ec2/userguide/attach-load-balancer-asg.html)                      |

