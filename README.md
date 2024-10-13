# Wordpress-website-solution-hosted-on-aws

## Goal
To create a scalable, secure, and cost-effective WordPress solution on AWS that can handle increasing traffic while ensuring high availability and fault tolerance. The project will utilize various AWS services, including VPC, EC2, Security groups, RDS, Elastic File System (EFS), and Auto Scaling, Application Loadbalancer, Route 53, Nat gateways etc.

![projectdesign](https://github.com/user-attachments/assets/601b50c8-cf18-4bfd-8825-811d3c5e07c4)

## Creating VPC 
To begin, we will build a custom Virtual Private Cloud (VPC). In this architecture, the infrastructure is divided into three distinct tiers, each handling specific responsibilities.

### First Tier (Public Subnet):

This tier contains public-facing resources, including a NAT Gateway, and Application Load Balancer (ALB).
The NAT Gateway allows instances in the private subnet to access the internet securely.
The ALB manages traffic across multiple EC2 instances.

### Second Tier (Private Subnet - Application Layer):

Hosts the web servers (EC2 instances).
These instances are isolated from the public internet but can communicate through the NAT gateway in the public subnet.

### Third Tier (Private Subnet - Database Layer):

Contains the database servers (e.g., RDS).
Isolated to enhance security and prevent direct access from external networks.

Additional Features:
Multiple Availability Zones (AZs): Each subnet is replicated across multiple AZs to ensure high availability and fault tolerance.
Internet Gateway: A route table and internet gateway will be created to enable resources within the VPC to access the internet as needed [2].

