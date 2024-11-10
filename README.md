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
