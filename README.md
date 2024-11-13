**Multi-Zone Auto Scaling and Secure Network Architecture on AWS**
---

### Project Heading:
**Highly Available and Secure Application Deployment in AWS VPC with Load Balancing and Auto Scaling**

### _Key Points_:

- **VPC Architecture**: Created a Virtual Private Cloud (VPC) with both public and private subnets to segregate application components.
- **Application Access**: Users access the application via an Internet Gateway and an Application Load Balancer located in public subnets, routing traffic securely to the application servers in private subnets.
- **Resilience and Scaling**: Set up an Auto Scaling Group across two Availability Zones to ensure high availability, with the Load Balancer distributing traffic across instances.
- **Security Layers**: Deployed instances in private subnets for added security, allowing only load balancer and Bastion Host access. Instances use NAT Gateways in each Availability Zone to access the internet.
- **Bastion Host Configuration**: Implemented a Bastion Host in the public subnet to securely connect to private subnet instances, facilitating server management.
- **Load Balancer and Target Group**: Configured an Application Load Balancer with a target group to direct HTTP traffic to instances in private subnets, enhancing traffic handling efficiency.
- **Step-by-Step Setup**: Comprehensive stepwise VPC creation, subnet and NAT Gateway configuration, EC2 Auto Scaling with launch template setup, security group rules, Bastion Host setup, and final load balancer DNS access for application deployment.

---

This README provides a complete walkthrough for creating the project. _Happy building!_

## Table of Contents
1. [Create the VPC](#create-the-vpc)
2. [Configure Subnets](#configure-subnets)
3. [Launch EC2 Instances and Auto-Scaling](#launch-ec2-instances-and-auto-scaling)
4. [Create the Bastion Host](#create-the-bastion-host)
5. [Configure Load Balancer and Target Group](#configure-load-balancer-and-target-group)
6. [Access Load Balancer](#access-load-balancer)
---

## Step 1: Create the VPC

**Virtual Private Cloud (VPC)** on AWS is a logically isolated section of the cloud where you can launch AWS resources in a virtual network. VPC allows you to control IP addressing, subnets, route tables, and security.

1. Go to the **VPC Dashboard** and select **Create VPC**.
2. **Configure VPC Settings:**
   - **Name tag:** Enter `Name_of_Your_VPC` for easy identification.
   - **IPv4 CIDR Block:** Use `10.0.0.0/24`. CIDR (Classless Inter-Domain Routing) is used to define IP address ranges, and in AWS, five IP addresses are reserved for internal AWS use.
   - **IPv6 CIDR Block:** Choose **No** (optional unless you need IPv6 support).
   - **Tenancy:** Select **Default**.
   - **Availability Zones (AZs):** Choose **2** for redundancy.
   - **Number of Public Subnets:** Set to **2**.
   - **Number of Private Subnets:** Set to **2**.
   - **NAT Gateways:** Choose **1 per AZ** (helps maintain secure internet access for private instances).
3. Click **Create VPC**.

---

## Step 2: Configure Subnets

After the VPC is created, subnets are necessary to logically separate instances. In this architecture, we’re using both **public** and **private subnets** to improve security.

1. Go to **VPC** > **Subnets** > **Create Subnet**.
2. **Configure Public Subnet:**
   - **Name tag:** `test_public_subnet`
   - **VPC ID:** Select the VPC created above.
   - **Availability Zone:** Select `us-east-2a`.
   - **IPv4 CIDR Block:** Choose a specific range within the VPC’s CIDR for the public subnet.
3. **Repeat** the process for the other subnets:
   - One more public subnet in a different AZ.
   - Two private subnets (one in each AZ).

---

## Step 3: Launch EC2 Instances and Configure Auto Scaling Group

EC2 (Elastic Compute Cloud) instances are the virtual servers for running applications, and an **Auto Scaling Group** (ASG) will help to automatically adjust capacity based on demand.

### How to Create an Auto Scaling Group
1. Go to the **EC2 Dashboard**, scroll down to **Auto Scaling Groups**, and choose **Create Auto Scaling Group**.
2. **Choose Launch Template** (required for auto scaling):
   - Go to **Instances** > **Launch Templates** > **Create Launch Template**.
   - **Template Name**: Enter a descriptive name.
   - **Application and OS Image (AMI)**: Select an AMI from **Recents** or **Quick Start**.
   - **Instance Type**: Select a **Free tier** type.
   - **Key Pair (login)**: Choose or create a new key pair for secure SSH access.
   - **Network Settings**:
     - **Subnet:** Do not assign; this is handled by the ASG.
     - **Firewall (Security Group)**:
       - **Create a Security Group**:
         - **Group Name**: `Aws_prod_Example`
         - **Description**: Short description for the group.
         - **VPC**: Select the VPC created in Step 1.
         - **Inbound Rules**:
           - Type **SSH** | Source: **Anywhere** | Port **22**.
           - Type **Custom TCP** | Source: **Anywhere** | Port **8000** (for application access).
3. Return to the **Auto Scaling Group** setup:
   - **VPC**: Select the VPC created earlier.
   - **Availability Zone and Subnets**: Choose **both private subnets**.
   - **Load Balancing**: Set to **No load balancer** (we’ll set up an external load balancer later).
   - **Group Size and Scaling Policies**:
     - **Desired Capacity**: 2 (initial instance count).
     - **Minimum Capacity**: 2.
     - **Maximum Capacity**: 4.
   - Press **Create Auto Scaling Group**.

---

## Step 4: Create the Bastion Host

A **Bastion Host** acts as a secure entry point to access instances in the private subnet.

1. Go to **EC2 Dashboard** > **Launch Instance** > **Launch Instance**.
2. **Instance Configuration**:
   - **AMI**: Choose Ubuntu.
   - **Instance Type**: Select `t2.micro` (free tier).
   - **Network Settings**:
     - **VPC**: Select the VPC created in Step 1.
     - **Auto-assign Public IP**: Enable (for external access).
   - **Firewall (Security Group)**:
     - **Create Security Group** with the following rules:
       - **Allow SSH** access from **anywhere** (IP range).
3. Click **Launch Instance** to create the bastion host.
4. **Access the Bastion Host**:
   - Copy the public IP of the bastion host.
   - Use **scp** to transfer the **.pem** key pair to the bastion host for accessing private instances:
     ```bash
     scp -i /path/to/aws_login.pem /path/to/aws_login.pem ubuntu@<Bastion_HOST_IP_ADDRESS>:/home/ubuntu
     ```
5. **Login to Private Instances**:
   - **ssh** into the bastion host:
     ```bash
     ssh -i aws_login.pem ubuntu@<Bastion_HOST_IP_ADDRESS>
     ```
   - From the bastion host, **ssh** into private instances:
     ```bash
     ssh -i aws_login.pem ubuntu@<Private_IP_Address>
     ```

---

## Step 5: Configure Load Balancer and Target Group

A **Load Balancer** distributes incoming traffic across multiple instances for better performance and redundancy.

1. Go to **EC2 Dashboard** > **Load Balancers** > **Create Load Balancer**.
2. **Load Balancer Configuration**:
   - **Name**: `aws-prod-example`.
   - **Scheme**: Internet-facing.
   - **IP Address Type**: IPv4.
   - **VPC**: Select the previously created VPC.
   - **Availability Zones**: Select both **public subnets**.
3. **Security Groups**: Select the security group configured earlier for the load balancer.
4. **Listeners and Routing**:
   - **Protocol**: HTTP | Port **80**.
   - **Target Group**: Create a new target group:
     - **Target Type**: Instance.
     - **Protocol**: HTTP | Port **8000**.
     - **VPC**: Select the previously created VPC.
     - **Targets**: Select the private instances and add them as targets.
5. **Security Group Rules for Load Balancer**:
   - **Inbound Rule**: Add an HTTP rule for **anywhere**.
6. **Create Load Balancer**.

---

## Step 6: Access Load Balancer

1. **Copy the DNS name** of the load balancer from the load balancer dashboard.
2. **Access the application** via the DNS name:
   - Open a browser and go to `http://<Load_Balancer_DNS_Name>`.

---
## Project Webshot
![Screenshot 2024-10-26 2357222](https://github.com/user-attachments/assets/2887cf54-1c37-4b38-87c1-805c8c635461)

### Mistakes what we occur during the implementation

- Error during Process the making Private Instance n bastion public instance
    - Select existing security group
        - Chose recently created VPC
    - Auto assign ip address : Enable
- While connecting Bastion Host with Private instance subnet, before switching bastion host to private instance ip:
    
    we should give permission to private key by putting command:
    
    > chmod 400 *private_key_name.pem*
    > 
- Select only one Security group during making of load balancer(Choose your created one)
- After successfully completing your project, you have to close instance by following process:

## Conclusion

Following these steps will result in a secure, scalable VPC setup on AWS with public/private subnet separation, a bastion host for secure access, and load balancing for redundancy. This environment supports high availability, secure management of private instances, and load-distributed application access.

> **Note**: Adjust configurations based on specific application needs or compliance requirements. 

---
