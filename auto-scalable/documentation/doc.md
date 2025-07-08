
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

The **Auto-Scaling Module** provisions a complete auto-scaling setup using **Launch Templates**, **Auto Scaling Groups**, **Target Groups**, **ALB Listener Rules**, and **optional scaling policies**. It dynamically manages EC2 instance counts based on load and ensures seamless routing using an Application Load Balancer (ALB). This module supports **cost efficiency**, **resilience**, and **horizontal scalability**.

---

## **2. Resources Required to Setup Auto Scaling Module**

| Resource Type                       | Count | Description                                                                                          |
| ----------------------------------- | ----- | ---------------------------------------------------------------------------------------------------- |
| **Launch Template**                 | 1     | Blueprint for EC2 configuration including AMI, instance type, key, SG, and subnet.                  |
| **Auto Scaling Group**              | 1     | Manages dynamic EC2 provisioning using Launch Template.                                              |
| **ALB Target Group**                | 1     | Target for load balancer to forward traffic to healthy EC2 instances.                                |
| **ALB Listener Rule**               | 1     | Forwards traffic to the correct target group based on path pattern.                                  |
| **ALB Listener (Optional)**         | 1     | HTTP listener for routing traffic based on defined port/protocol.                                    |
| **Scaling Policies (Optional)**     | 1–2   | Target Tracking or Step Scaling policy to scale based on metrics.                                    |
| **Security Group**                  | 1     | For controlling access to EC2 instances.                                                             |
| **Subnet**                          | 1     | Private subnet ID where instances are deployed.                                                      |
| **IAM Role (Optional)**             | 1     | Used if EC2 instances require access to AWS services.                                                |

---

## **3. Inputs**

| Variable Name                   | Type   | Required | Description                                                       |
| ------------------------------ | ------ | -------- | ----------------------------------------------------------------- |
| `template_name`                | string | Yes      | Name prefix for the launch template                               |
| `ami_id`                       | string | Yes      | AMI ID for EC2 instances                                          |
| `instance_type`                | string | Yes      | EC2 instance type                                                 |
| `key_name`                     | string | Yes      | Key pair name for EC2 SSH access                                  |
| `subnet_id`                    | string | Yes      | Subnet ID to deploy EC2 instances                                 |
| `security_group_id`            | string | Yes      | Security group to attach to EC2                                   |
| `asg_name`                     | string | Yes      | Name of the Auto Scaling Group                                    |
| `desired_capacity`             | number | Yes      | Desired number of EC2 instances                                   |
| `min_size`                     | number | Yes      | Minimum number of EC2 instances                                   |
| `max_size`                     | number | Yes      | Maximum number of EC2 instances                                   |
| `lt_version`                   | string | Yes      | Launch template version (e.g., "$Latest")                         |
| `vpc_id`                       | string | Yes      | VPC ID where the Target Group is created                          |
| `tg_name`                      | string | Yes      | Name of the Target Group                                          |
| `port`                         | number | Yes      | Port on which target group routes traffic                         |
| `protocol`                     | string | Yes      | Protocol for target group (HTTP, HTTPS)                           |
| `target_type`                  | string | Yes      | Type of target (instance, ip)                                     |
| `listener_arn`                 | string | No       | ARN of existing ALB listener (used if listener creation is false) |
| `alb_arn`                      | string | No       | ALB ARN required to create listener (if `enable_http_listener`)   |
| `path_pattern`                 | string | No       | Path pattern for ALB listener rule (e.g., `/api/*`)               |
| `priority`                     | number | No       | Listener rule priority                                            |
| `asg_policy_name`              | string | No       | Name of the scaling policy                                        |
| `asg_policy_type`              | string | No       | Type of scaling policy (`TargetTrackingScaling`, `StepScaling`)   |
| `estimated_instance_warmup`    | number | No       | Estimated warm-up time in seconds                                 |
| `asg_policy_target_value`      | number | No       | Target value for CPU or metric utilization                        |
| `enable_asg_policy`            | bool   | No       | Whether to enable autoscaling policy                              |
| `step_adjustment_type`         | string | No       | Step scaling adjustment type                                      |
| `step_metric_interval_lower_bound` | number | No    | Step scaling interval lower bound                                 |
| `step_scaling_adjustment`      | number | No       | Adjustment value for step scaling                                 |
| `health_check_path`            | string | No       | Health check path for the target group                            |
| `health_check_interval`        | number | No       | Health check interval in seconds                                  |
| `health_check_protocol`        | string | No       | Protocol for health checks                                        |
| `health_check_timeout`         | number | No       | Timeout in seconds for health check response                      |
| `healthy_threshold`            | number | No       | Number of healthy checks before instance considered healthy       |
| `unhealthy_threshold`          | number | No       | Number of failures before instance considered unhealthy           |
| `enable_http_listener`         | bool   | No       | Whether to create HTTP listener                                   |
| `listener_port`                | number | No       | ALB listener port                                                 |
| `listener_protocol`            | string | No       | ALB listener protocol                                             |
| `asg_tags`, `lt_tags`, `tg_tags`, `lr_tags` | map/list | No | Tags for ASG, Launch Template, Target Group, and Listener Rule   |
| `launch_template_id`           | string | No       | Use an existing launch template ID instead of creating one        |
| `region`                       | string | Yes      | AWS region                                                        |
| `health_check_grace_period`    | number | No       | ASG health check grace period                                     |
| `min_healthy_percentage`       | number | No       | For instance refresh strategy                                     |

---

## **4. Outputs**

| Output Name              | Description                                                                      |
| ------------------------ | -------------------------------------------------------------------------------- |
| `asg_name`               | Name of the Auto Scaling Group                                                   |
| `launch_template_id`     | ID of the Launch Template                                                        |
| `launch_template_name`   | Name of the Launch Template                                                      |
| `listener_rule_arn`      | ARN of the ALB Listener Rule                                                     |
| `listener_rule_id`       | ID of the ALB Listener Rule                                                      |
| `asg_policy_name`        | Name of the ASG scaling policy                                                   |
| `asg_policy_type`        | Type of the ASG scaling policy                                                   |
| `asg_policy_target_value`| Target metric value used by the scaling policy                                   |
| `target_group_arn`       | ARN of the Target Group                                                          |
| `target_group_name`      | Name of the Target Group                                                         |
| `target_group_id`        | ID of the Target Group                                                           |
| `alb_lr_arn`             | ARN of the created ALB listener (if `enable_http_listener` is true)             |

---

## **5. How it Works**

1. A **Launch Template** is created with the desired AMI, instance type, subnet, and security group.
2. The **Auto Scaling Group** launches EC2 instances based on this template across the provided subnet.
3. A **Target Group** is either created or reused and registered with the ASG to perform health checks.
4. Optionally, an **ALB Listener** is created to forward HTTP traffic.
5. A **Listener Rule** is created to forward traffic based on a path pattern to the Target Group.
6. **Scaling Policies** can be configured using either target tracking or step scaling.

---

## **6. Example Usage**

```hcl
module "auto_scaling" {
  source                  = "git::https://github.com/Cloud-NInja-snaatak/terraform-modules.git//auto-scalable"
  template_name           = "dev-otms-asg-template"
  ami_id                  = "ami-0c55b159cbfafb09"
  instance_type           = "t2.micro"
  key_name                = "dev-otms-key"
  subnet_id               = module.vpc.private_subnet_id
  security_group_id       = module.sg.instance_sg_id
  asg_name                = "dev-otms-asg"
  desired_capacity        = 2
  min_size                = 1
  max_size                = 4
  lt_version              = "$Latest"
  vpc_id                  = module.vpc.vpc_id
  tg_name                 = "dev-otms-tg"
  port                    = 80
  protocol                = "HTTP"
  target_type             = "instance"
  listener_arn            = module.alb.listener_arn
  path_pattern            = "/api/*"
  enable_asg_policy       = true
  asg_policy_name         = "dev-otms-cpu-policy"
  asg_policy_type         = "TargetTrackingScaling"
  asg_policy_target_value = 60
  region                  = "us-east-1"
}
````

---

## **7. Tags**

```hcl
lt_tags = {
  Name        = "lt-dev-otms"
  environment = "dev"
  owner       = "aditya"
}

asg_tags = [
  {
    key                 = "Name"
    value               = "asg-dev-otms"
    propagate_at_launch = true
  },
  {
    key                 = "environment"
    value               = "dev"
    propagate_at_launch = true
  }
]

tg_tags = {
  Name        = "tg-dev-otms"
  environment = "dev"
}

lr_tags = {
  environment = "dev"
}
```

---

## **8. AWS Resources Overview**

| Resource                 | Description                                                        |
| ------------------------ | ------------------------------------------------------------------ |
| `aws_launch_template`    | Launch template for EC2 instance bootstrapping                     |
| `aws_autoscaling_group`  | Maintains EC2 instance count based on desired, min, and max values |
| `aws_lb_target_group`    | Routes traffic to ASG-managed EC2 instances                        |
| `aws_lb_listener`        | Optional HTTP listener on ALB                                      |
| `aws_lb_listener_rule`   | Path-based forwarding rule that maps to the target group           |
| `aws_autoscaling_policy` | Optional scaling policy (target tracking or step scaling)          |

---

## **9. Conclusion**

This Auto Scaling module is built to ensure scalable, fault-tolerant, and cost-efficient infrastructure. It abstracts complex ALB-ASG configurations into reusable Terraform code, with built-in support for rolling updates, dynamic scaling, and detailed tagging for visibility and governance.

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

```
