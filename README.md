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
Internet Gateway: A route table and internet gateway will be created to enable resources within the VPC to access the internet as needed

# AWS VPC Setup Guide

Creating and configuring a Virtual Private Cloud (VPC) in AWS. The setup includes creating public and private subnets, configuring route tables, and establishing NAT gateways for secure internet access.

## Table of Contents

## Step 1: Create a VPC

1. **Access the VPC Console**:
   - Log in to the AWS Management Console.
   - Search for and select **VPC**.

2. **Create the VPC**:
   - Click on **Create VPC**.
   - Choose the **VPC only** option.
   - Provide a name for your VPC and specify the CIDR block as `10.0.0.0/16`.
   - Click on the **Create VPC** button.

3. **Enable DNS Hostnames**:
   - Click on the VPC ID link.
   - Select **Actions**, then choose **Edit VPC settings**.
   - Check the box to enable **DNS hostname**, scroll down, and click **Save**.

## Step 2: Create and Attach an Internet Gateway

1. **Navigate to Internet Gateways**:
   - On the VPC dashboard, click on **Internet gateways** in the left sidebar.

2. **Create Internet Gateway**:
   - Click on **Create Internet gateway**.
   - Provide a name for the internet gateway and click on **Create internet gateway**.

3. **Attach Internet Gateway to VPC**:
   - Select the created internet gateway.
   - Click on **Actions**, then select **Attach to VPC**.
   - Choose your previously created VPC and click **Attach internet gateway**.

## Step 3: Configure Subnets within the VPC

1. **Create Public Subnets**:
   - Navigate to the **Subnets** section in the left sidebar and click on it.
   - Click on **Create subnet**.
     - For the first public subnet:
       - Select your VPC ID.
       - Name it and choose availability zone (e.g., `us-east-1a`).
       - Specify IPv4 CIDR as `10.0.0.0/24`.
       - Click on **Add new subnet**.
     - Repeat for a second public subnet with availability zone `us-east-1b` and CIDR `10.0.1.0/24`.

2. **Enable Auto-Assign Public IPs**:
   - Select each public subnet, click on **Actions**, then select **Edit subnet settings**.
   - Enable **auto-assign public IPv4 address**, then click **Save**.

## Step 4: Create Route Table for Connectivity

1. **Create Route Table**:
   - Navigate to the **Route tables** section.
   - Click on **Create route table**, enter a name (e.g., `public-route-table`), select your VPC, and click on **Create route table**.

2. **Add Public Route to Route Table**:
   - Select your route table, go to the **Routes** tab, and click on **Edit routes**.
   - Add a route with Destination `0.0.0.0/0` (to allow all IPv4 addresses) and Target as your internet gateway, then click **Save changes**.

3. **Associate Subnets with Route Table**:
   - Go to the Subnet associations tab, click on **Edit subnet associations**, select both public subnets, and click on **Save association**.

## Step 5: Create Private Subnets

1. Repeat the process of creating subnets for four private subnets (two for applications and two for databases).
2. Use CIDR blocks such as `10.0.2.0/24` for one private subnet in `us-east-1a` and similar for others.

## Step 6: Create NAT Gateways

1. For each public subnet, create a NAT gateway:
   - Go to NAT gateways in the dashboard, click on Create NAT gateway, provide a name, select the public subnet, allocate an Elastic IP, and create it.

2. Create corresponding route tables for private subnets that route traffic through their respective NAT gateways.

3. Associate these route tables with their corresponding private subnets.

## Step 7: Create Security Groups

1. Navigate to Security Groups in your VPC dashboard.
2. Create security groups as needed (e.g., web server, database).
3. Set inbound rules for HTTP (port 80) and HTTPS (port 443) with source set to `0.0.0.0/0`.

By following these steps meticulously, you will have set up a robust AWS VPC architecture that supports both public and private subnets with secure internet access through NAT gateways and appropriate security groups configured for your applications and databases.


## PART 2: RDS, EFS, EC2, and WordPress Setup

### Step 6: Create RDS Database

1. Search for "RDS" in the AWS Management Console and select it.
2. In the RDS dashboard, go to the left sidebar, select **Subnet groups**, then click on "**Create db subnet group**".
3. Provide a name for the database subnets, give a description, then select your VPC.
4. Under "Add subnet", select availability zones `us-east-1a` and `us-east-1b`.
5. Under Subnets, select the private data subnets with CIDR blocks `10.0.4.0/24` and `10.0.5.0/24`.
6. Scroll down and click "**Create**".

### Next Steps to Create Database

1. In the left sidebar of RDS dashboard, select "**Databases**" then click "**Create database**".
2. Choose "**Standard create**" under database creation method.
3. Select MySQL under Engine options; choose the latest version available.
4. Under Templates, select "**Dev/Test**".
5. In Settings section, type a name for DB instance identifier; under Credential settings, input your username and password.
6. Under DB instance class, select Burstable classes; enable previous generation classes if needed.
7. Under Connectivity (VPC), select your VPC; under DB subnet group, choose the database subnet group created earlier.
8. Under VPC security group, choose existing security group named "database security group".
9. Select availability zone "us-east-1b", scroll down to expand "Additional configuration", type a name for initial database name.
10. Finally, scroll down and click "**Create database**". The RDS instance is now successfully created.

### Step 7: Create Elastic File System (EFS)

1. Search for "EFS" in your console services and select it; then click "**Create file system**".
2. Provide a name under File system settings; select Standard storage class; uncheck "encryption".
3. Give it a tag name; click Next.
4. Under Network access:
   - Select your VPC,
   - For Mount targets in availability zones, choose `us-east-1a` and `us-east-1b`.
5. Under Security groups section:
   - Delete default security group,
   - Select EFS security group for both availability zones respectively.
6. Click Next until you reach create page; then click "**Create file system**".

### Step 8: Launch EC2 Instance Server

1. Search for "EC2" in your console; select it then click "**Launch instance**".
2. Provide a name for your EC2 instance; choose your machine image; create key pair if necessary.
3. Under Network settings:
   - Select your VPC,
   - Choose public subnet az1 from dropdown menu.
4. Under Firewall settings:
   - Select Existing security group,
   - Choose SSH, ALB (Application Load Balancer), and web server security groups from dropdown list.
5. Click "**Launch instance**".

### Install WordPress on EC2 Instance

SSH into the server after launching it:

```bash
# Update mount information
sudo su
yum update -y
mkdir -p /var/www/html
sudo mount -t nfs4 \
-o nfsvers=4.1,rsize=1048576,wsize=1048576,\
hard,timeo=600,retrans=2,noresvport \
fs-03c9b3354880b36a6.efs.us-east-1.amazonaws.com:/ /var/www/html

# Install Apache
sudo yum install -y httpd httpd-tools mod_ssl
sudo systemctl enable httpd 
sudo systemctl start httpd

# Install PHP 7.x
sudo amazon-linux-extras enable php7.x
sudo yum clean metadata
sudo yum install php php-common php-{cgi,curl,mysqlnd,gettext,json} php-mbstring php-gd php-intl php-zip php-fpm php-json 

# Install MySQL
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql2022
sudo yum install mysql-community-server
sudo systemctl enable mysqld
sudo systemctl start mysqld

# Set permissions
sudo usermod -aG apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d \
-exec sudo chmod 2775 {} \;
sudo find /var/www -type f \
-exec sudo chmod 0664 {} \;
chown apache:apache /var/www/html 

# Download WordPress files
wget https://wordpress.org/latest.tar.gz
tar xzf latest.tar.gz
cp -r wordpress/* /var/www/html/

# Configure WordPress
cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
nano /var/www/html/wp-config.php # Update DB credentials here

# Restart web server
service httpd restart

Copy the public IP address of your EC2 server to view it in a browser.

Step 9: Create Application Load Balancer
Set up EC2 instances in each of the private app subnets:

#!/bin/bash
yum update -y
sudo yum install httpd httpd-tools mod_ssl 
sudo systemctl enable httpd 
sudo systemctl start httpd 
# Install other required packages as before...
echo "fs-03c9b3354880b36a6...:/ /var/www/html nfs4 nfsvers=4..." >> /etc/fstab 
mount -a 
chown apache:apache /var/www/html 
service httpd restart

Launch instances using this script in both private app subnets.
Step 10: Create Target Group
In EC2 dashboard:
Go to Target groups section,
Click on "Create target group",
Place both instances in this target group so that ALB can route traffic correctly.
Step 11: Configure Route 53 Record Set
To access your website via domain:
Search for Route 53 in AWS console,
On Route 53 dashboard, go to Hosted zones,
Click "Create hosted zone",
Select your domain,
Click "Create record set, type 'www', toggle Alias ON,
Select Alias to Application Load Balancer,
7.Select load balancer created earlier,
8.Click "**Create record set".

SSL Certificate Configuration
Request an SSL certificate via AWS Certificate Manager:
1 Follow instructions in ACM dashboard until status shows "Pending",
2 Validate it through Route 53 as instructed,
3 Once issued status shows success.

HTTPS Listener Setup
In Load Balancer dashboard:
1.Create HTTPS listener using SSL certificate,
2.Modify HTTP listener to redirect traffic to HTTPS.

Launch Bastion Host
To SSH into instances in private subnets:
1.Launch an instance in public subnet as Bastion host,
2.Copy its IPv4 public address,
3.SSH into it using command:

ssh ec2-user@<Bastion_Public_IP>

Then SSH into any web server using its private IP:
bash

ssh ec2-user@<Private_Web_Server_IP>

Auto Scaling Group Creation
Terminate manually created web servers before creating Auto Scaling Group:
1.Follow steps provided in screenshots to create launch template,
2.Paste earlier configuration commands into User Data box ensuring EFS mount info is updated,
3.Create Auto Scaling Group using this launch template.

Final Steps: Install WordPress Theme & Template
Log into your WordPress site at /wp-admin, install Astra theme:
1.Select "Appearance" > "Add New",
2.Install & activate theme,
3.Follow prompts to build professional website using Elementor template named "Love Nature".

Congratulations! You have successfully set up an AWS environment with WordPress running behind an Application Load Balancer secured by SSL!


