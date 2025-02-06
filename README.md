# Terraform-Deploying-Secure-Highly-Available-Fault-Tolerant-Cloud-Infrastructure-


" Terraform Auto Scaling"

# Technical Architecture

![image alt](https://github.com/Tatenda-Prince/Terraform-Deploying-Secure-Highly-Available-Fault-Tolerant-Cloud-Infrastructure-/blob/40ce4c252fc5e172587eadb5f41bc5b3c278a48d/images/Screenshot%202025-02-06%20093756.png)

# Introduction

Having a robust and reliable cloud infrastructure is crucial for businesses to deliver seamless services to their customers. High availability and fault tolerance are two essential components of such an infrastructure. In order to achieve this, many organizations turn to infrastructure automation tools like Terraform to deploy and manage their cloud resources.


We’ll explore how to use Terraform to deploy a high availability and fault-tolerant cloud environment on AWS, complete with an Auto Scaling Group (ASG) spanning two Availability Zones in private subnets of a custom VPC. We’ll also look at how to front the ASG with an Application Load Balancer (ALB) placed in public subnets, with appropriate gateway and route table configurations.

By the end, you’ll have a solid understanding of how to leverage Terraform to build a reliable and scalable cloud infrastructure that can handle high traffic loads and maintain uptime, even in the event of failures in individual components.

## What is Terraform

Terraform is an open-source infrastructure as code (IaC) tool that allows developers to define and manage cloud infrastructure resources in a declarative way.

Declarative is a programming concept where a program describes what should be accomplished, rather than how to accomplish it. In the context of Terraform, a declarative approach means that you define the desired state of your infrastructure and Terraform takes care of figuring out the necessary actions to achieve that state.

In Terraform, you define the desired state of your infrastructure using a configuration file, which specifies the resources you want to create, their properties and any dependencies between them. You do not need to write procedural code that specifies how to create and manage those resources. Instead, Terraform compares the desired state with the current state of the infrastructure and determines the necessary actions to achieve the desired state.

Terraform is also cloud agnostic. This means it supports multiple cloud providers such as Amazon Web Services (AWS), Microsoft Azure, and Google Cloud Platform (GCP), as well as various other services, such as databases, DNS, and monitoring tools.

## Benefits of using Terraform

1.Infrastructure as code: Terraform provides a way to define infrastructure as code, which means you can version control and manage your infrastructure just like you would with application code.

2.Reusability: Terraform allows you to reuse infrastructure code across different environments, making it easier to manage and deploy consistent infrastructure.

3.Cloud-agnostic: Terraform can manage resources across multiple cloud providers and services, making it a great tool for multi-cloud deployments.

4.Consistency: Terraform ensures that the infrastructure deployed is consistent across all environments, making it easier to manage and maintain.

Now let me give you some background information to understand the more specific components of Terraform.

# Background

## Terraform Providers

A provider is a plugin that enables Terraform to interact with a particular infrastructure platform, service, or system. Providers are responsible for translating Terraform configuration files into API calls to create and manage resources.

## Resource Type

A resource type is a category or class of infrastructure resources that can be created, updated, or destroyed using Terraform.

## Resource Name

A resource name is a unique identifier assigned to a specific resource within a given Terraform configuration.

## Resource Arguments

Resource arguments are the parameters or attributes that are passed to a resource block in Terraform. They provide the necessary information for Terraform to provision and manage a specific resource.

## Variables

Variables are used to define values that can be passed into a Terraform configuration at runtime. These variables can be easily changed or customized in your configurations to make them more dynamic and reusable.

# Main Terraform Commands

## Terraform init

The “terraform init” command initializes a new or existing Terraform working directory by downloading and installing the necessary plugins and modules for the Terraform configuration in that directory specified in the configuration files.

## Terraform validate

The “terraform validate” command checks the syntax and structure of the Terraform configuration files in the current working directory and validates the Terraform code for any errors or mistakes before deploying it to the target environment.

## Terraform plan

The “terraform plan” command creates an execution plan and generates a list of all the changes that Terraform will make to the infrastructure resources to bring it to the desired state defined in the Terraform configuration files.

## Terraform apply

The “terraform apply” command applies the changes by deploying the infrastructure resources defined in the configuration files to the target environment.

## Terraform destroy

The “terraform destroy” command removes all the resources that Terraform has created in the target environment based on the configuration files.

## Amazon EC2 Auto Scaling Group (ASG)

Amazon EC2 Auto Scaling Group is a service provided by AWS that automatically adjusts the number of EC2 instances in a group based on demand.

It helps to ensure that the desired number of instances are available to handle the workload, and can automatically add or remove instances as needed. This allows applications to scale up or down seamlessly, depending on demand, and helps to optimize cost by only running the necessary number of instances at any given time.

EC2 Auto Scaling Groups can be configured with various scaling policies, launch configurations, and instance types to ensure efficient and reliable application performance.

## Application Load Balancer (ALB)

AWS’s Application Load Balancer (ALB) enables the distribution of incoming traffic to multiple targets, such as EC2 instances, based on application-level content.

ALB can intelligently route traffic to different targets based on advanced routing rules, such as path-based routing, host-based routing, and HTTP header-based routing.

ALB provides various features such as SSL/TLS offloading, connection draining, and sticky sessions, and can be used with AWS’s Auto Scaling and Elasticity features to ensure optimal performance and scalability.

# Prerequisites

1.Basic knowledge and understanding of Terraform concepts and commands

2.Basic Linux command line knowledge

3.AWS Account with an IAM user with administrative privileges

4.Visual Code Studio

4.Amazon EC2 Key pair


## Use Case

Up The Chels TECH Corp, an e-commerce company, needs to handle a surge in traffic during the holiday season. The company wants to ensure that their website remains available and responsive to customers even during high traffic periods.

Your manager has assigned you to deploy a cloud infrastructure on AWS that will ensure high availability and fault tolerance. You decide to utilize Terraform for infrastructure automation to deploy an EC2 Auto Scaling Group (ASG) in private subnets fronted by an Application Load Balancer (ALB) in public subnets, which will automatically scale up or down based on traffic, ensuring that the website remains responsive to customers at all times.

## Objectives

1.Create a custom Virtual Private Network (VPC) with 2 public subnets, 2 private subnets in two separate Availability Zones.

2.Utilize a NAT Gateway in the public subnet, and an Internet Gateway so there is outbound internet traffic.

3.Configure a public route table and private route table.

4.Launch the ALB in the public subnets.

5.Launch the Auto Scaling group in the private subnets

6.Output the public DNS of the ALB and then verify reachability to the web servers using its URL.

## Step 0: Install latest Visual Code Studio environment

To enhance your understanding and follow this demonstration more effectively, please clone the project’s repository from my GitHub by clicking on the link below. As we move forward, feel free to edit the files as necessary.

https://github.com/Tatenda-Prince/Terraform-Deploying-Secure-Highly-Available-Fault-Tolerant-Cloud-Infrastructure-

## Step 1: Creating and Configuring a custom AWS VPC

Our initial objective is to establish a custom VPC within our AWS environment, where all our resources can be deployed. To achieve this, we must define a logically isolated CIDR block and carefully select the appropriate subnets and corresponding availability zones (AZ) to deploy them in.

Let’s review the code below of a Terraform file to create and configure the custom VPC —

```yaml
# Create an AWS VPC
resource "aws_vpc" "terraform-vpc" {
  cidr_block       = var.vpc-cidr
  instance_tenancy = "default"

  tags = {
    Name = var.vpc_name
  }
}

# Create first public subnet in the VPC
resource "aws_subnet" "pub-sub1" {
  vpc_id                  = aws_vpc.terraform-vpc.id
  cidr_block              = var.pub_sub1_cidr
  availability_zone       = var.availability_zone-1
  map_public_ip_on_launch = true

  tags = {
    Name = var.pub-sub1-name
  }
}

# Create second public subnet in the VPC
resource "aws_subnet" "pub-sub2" {
  vpc_id                  = aws_vpc.terraform-vpc.id
  cidr_block              = var.pub_sub2_cidr
  availability_zone       = var.availability_zone-2
  map_public_ip_on_launch = true

  tags = {
    Name = var.pub-sub2-name
  }
}

# Create first private subnet in the VPC
resource "aws_subnet" "priv-sub1" {
  vpc_id                  = aws_vpc.terraform-vpc.id
  cidr_block              = var.priv_sub1_cidr
  availability_zone       = var.availability_zone-1
  map_public_ip_on_launch = true

  tags = {
    Name = var.priv-sub1-name
  }
}

# Create second private subnet in the VPC
resource "aws_subnet" "priv-sub2" {
  vpc_id                  = aws_vpc.terraform-vpc.id
  cidr_block              = var.priv_sub2_cidr
  availability_zone       = var.availability_zone-2
  map_public_ip_on_launch = true

  tags = {
    Name = var.priv-sub2-name
  }
}
```

## Code Explanation

Here we create an AWS VPC with a given CIDR block and instance tenancy. Then we create four subnets, two public and two private. Each subnet has a given CIDR block and is associated with the VPC created earlier and is defined to be deployed in specified availability zones.

The public subnets have the map_public_ip_on_launch attribute set to true, which means that instances launched in those subnets will be assigned a public IP address automatically. The private subnets have that attribute set to false, so instances launched in those subnets won’t be reachable from the internet unless through a NAT gateway which we will create in the next step. Each subnet has a name assigned to it using a tag.

Now that we have created and configured a custom VPC, let’s jump to Step 2 and create an Internet Gateway and NAT Gateway.

## Step 2: Create an Internet Gateway and NAT Gateway in public subnet.

When deploying resources in the VPC, it is crucial to ensure that they are connected to the internet in a secure and efficient manner. One way to accomplish this is by creating an Internet Gateway and NAT Gateway in a public subnet.

An Internet Gateway is a VPC component that allows communication between instances in a VPC and the internet. Meanwhile, a NAT Gateway is used to enable instances in private subnets to connect to the internet or other AWS services, while also providing additional security for outbound traffic.

Let’s explore the code below of a Terraform file to create and configure an Internet Gateway and NAT Gateway —

```yaml
# Create an internet gateway and associate it with the VPC
resource "aws_internet_gateway" "terraform-igw" {
  vpc_id = aws_vpc.terraform-vpc.id

  tags = {
    Name = var.igw-name
  }
}

# Create an Elastic IP address
resource "aws_eip" "ngw-eip" {
  vpc = true
}

# Create a NAT gateway and associate it with an Elastic IP and a public subnet
resource "aws_nat_gateway" "terraform-ngw" {
  allocation_id = aws_eip.ngw-eip.id     # Associate the NAT gateway with the Elastic IP
  subnet_id     = aws_subnet.pub-sub1.id # Associate the NAT gateway with a public subnet

  tags = {
    Name = var.nat-gw-name
  }

  depends_on = [aws_internet_gateway.terraform-igw] # Make sure the internet gateway is created before creating the NAT gateway
}
```
## Code Explanation

Firstly we create an Internet Gateway and associate with the custom VPC. We also create an Elastic IP address along with a NAT Gateway and associate it with the Elastic IP and a public subnet. We also ensure that the Internet Gateway is created before creating the NAT gateway.

Great! We’ve got our Internet Gateway and NAT Gateway created, on to Step 3 — Configuring our public and private route tables.

## Step 3: Configure a public route table and private route table.

At this point, we have successfully created a customized VPC and the gateways. However, the challenge we face now is to manage the communication among the services and other components deployed in the VPC. This is where the configuration of the network traffic flow within the VPC becomes crucial. One effective way to achieve this is by setting up routing tables to direct the network traffic between the subnets.

Let’s review the code below on how we can create and configure our respective route tables —

```yaml
# Creates a public route table with a default route to the internet gateway
resource "aws_route_table" "pub-rt" {
  vpc_id = aws_vpc.terraform-vpc.id

  # Create a default route for the internet gateway with destination 0.0.0.0/0
  route {
    cidr_block = var.pub_rt_cidr
    gateway_id = aws_internet_gateway.terraform-igw.id
  }

  tags = {
    Name = var.pub-rt-name
  }
}

# Creates a private route table with a default route to the NAT gateway
resource "aws_route_table" "priv-rt" {
  vpc_id = aws_vpc.terraform-vpc.id

  route {
    cidr_block = var.priv_rt_cidr
    gateway_id = aws_nat_gateway.terraform-ngw.id
  }

  tags = {
    Name = var.priv-rt-name
  }
}

# Associates the public route table with the public subnet 1
resource "aws_route_table_association" "pub-sub1-rt-ass" {
  subnet_id      = aws_subnet.pub-sub1.id
  route_table_id = aws_route_table.pub-rt.id
}

# Associates the public route table with the public subnet 2
resource "aws_route_table_association" "pub-sub2-rt-ass" {
  subnet_id      = aws_subnet.pub-sub2.id
  route_table_id = aws_route_table.pub-rt.id
}

# Associates the private route table with the private subnet 1
resource "aws_route_table_association" "priv-sub1-rt-ass" {
  subnet_id      = aws_subnet.priv-sub1.id
  route_table_id = aws_route_table.priv-rt.id

  # Wait for the private route table to be created before creating this association
  depends_on = [aws_route_table.priv-rt]
}

# Associates the private route table with the private subnet 2
resource "aws_route_table_association" "priv-sub2-rt-ass" {
  subnet_id      = aws_subnet.priv-sub2.id
  route_table_id = aws_route_table.priv-rt.id

  # Wait for the private route table to be created before creating this association
  depends_on = [aws_route_table.priv-rt]
}
```

## Code Explanation

Here we are creating two route tables — a public route table and a private route table. The public route table has a default route to the Internet Gateway, and the private route table has a default route to the NAT Gateway.

The public route table is associated with the two public subnets, and the private route table is associated with the two private subnets. The private resource associations are dependent on the private route table being created before they are established. The route tables have also been assigned names using tags.

Now, on to Step 4 we go! Let’s launch a new ALB in the public subsets.

## Step 4: Launch the ALB in the public subnets.

Let’s explore the process of launching an ALB in the public subnets of the custom VPC. Creating an ALB involves several steps, including creating a Security Group, setting up Target Groups, listeners and routing rules.

Review the the following Terraform code, which creates a Security Group to be associated with our ALB —

```yaml
# Create security group for ALB
resource "aws_security_group" "alb-sg" {
  # Set name and description of the security group
  name        = var.alb_sg_name
  description = var.alb_sg_name 

  # Set the VPC ID where the security group will be created
  vpc_id     = aws_vpc.terraform-vpc.id
  depends_on = [aws_vpc.terraform-vpc]

  # Inbound Rule
  # HTTP access from anywhere
  ingress {
    description = "Allow HTTP Traffic"
    from_port   = var.http_port
    to_port     = var.http_port
    protocol    = "tcp"
    cidr_blocks = var.alb_sg_ingress_cidr_blocks
  }

  # SSH access from anywhere
  ingress {
    description = "Allow SSH Traffic"
    from_port   = var.ssh_port
    to_port     = var.ssh_port
    protocol    = "tcp"
    cidr_blocks = var.alb_sg_ingress_cidr_blocks
  }

  # Inbound Rule
  # Allow all egress traffic
  egress {
    from_port   = "0"
    to_port     = "0"
    protocol    = "-1"
    cidr_blocks = var.alb_sg_egress_cidr_blocks
  }

  # Set tags for the security group
  tags = {
    Name = "ALB SG"
  }
}
```
## Code Explanation

In this Terraform code, we create a Security Group that is intended to be used with the ALB. The Security Group is defined with a name and description, and is associated with the custom VPC.

It has ingress rules allowing HTTP and SSH traffic from a specified CIDR block, and an egress rule allowing all traffic. We also tag the Security Group with a name for easy of identification.

We can now proceed to review the code to create and configure the ALB —

```yaml

# Create a new load balancer
resource "aws_lb" "pub-sub-alb" {
  name            = var.load_balancer_name
  subnets         = [aws_subnet.pub-sub1.id, aws_subnet.pub-sub2.id]
  security_groups = [aws_security_group.alb-sg.id]

  tags = {
    Name = "Pub-Sub-ALB"
  }
}

# Create a target group for the load balancer
resource "aws_lb_target_group" "alb-tg" {
  name     = var.target_group_name
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.terraform-vpc.id

  # Set the health check configuration for the target group
  health_check {
    interval = 60
    path     = "/"
    port     = 80
    timeout  = 45
    protocol = "HTTP"
    matcher  = "200,202"
  }
}


# Create ALB listener
resource "aws_lb_listener" "alb-listener" {
  load_balancer_arn = aws_lb.pub-sub-alb.arn
  port              = "80"
  protocol          = "HTTP"

  # Set the default action for the listener
  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.alb-tg.arn
  }
}
```

## Code Explanation

Here we create an ALB and its associated resources. Firstly, we create the ALB, then specifying the name, subnets, and Security Groups to be used.

We proceed by creating a Target Group for the ALB to direct traffic to, specifying the name, port, protocol, and health check configuration.

Finally, we create a listener for the ALB, specifying the port and protocol to listen on, and set the default action to forward traffic to the previously created Target Group. This allows traffic to be directed to the appropriate resources based on the listener rules and the Target Group’s configuration.

Now, let’s recap, we have reviewed the Terraform configuration files to a create custom VPC and configure its appropriate subnets, along with an Internet and NAT gateway and a public and private route table. With the ALB’s configurations now set and ready, we can head over to Step 5 — Launching the ASG in private subnets.

## Step 5: Launch the Auto Scaling Group in the private subnets

While Auto Scaling Groups (ASG) are commonly deployed in public subnets, there are many cases where private subnets are preferred, such as when dealing with sensitive data or complying with regulatory requirements.

Let’s explore the code below that goes through the steps involved in launching an EC2 ASG across two Availability Zones (AZ) in private subnets —

```yaml
# Creating Security Group for ASG Launch Template
resource "aws_security_group" "lt-sg" {
  name   = var.lt_sg_name
  vpc_id = aws_vpc.terraform-vpc.id

  # Inbound Rules
  # HTTP access from anywhere
  ingress {
    from_port       = var.http_port
    to_port         = var.http_port
    protocol        = "tcp"
    security_groups = [aws_security_group.alb-sg.id]
  }

  # SSH access from anywhere
  ingress {
    from_port       = var.ssh_port
    to_port         = var.ssh_port
    protocol        = "tcp"
    security_groups = [aws_security_group.alb-sg.id]
  }

  # Outbound Rules
  # Internet access to anywhere
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = var.lt_sg_egress_cidr_blocks
  }
}
```
## Code Explanation

Firstly, we need to create a Security Group for the ASG Launch Template and associate it with the previously created custom VPC.

The Security Group allows inbound traffic for HTTP and SSH protocols strictly from the ALB’s Security Group defined in a separate resource. This provides security as access to our ASG can only come from the ALB. As Security Groups are stateful, we also allows outbound traffic to anywhere with a CIDR block specified in a defined variable.

## Bash Script

To prepare for the creation of our ASG, we will first develop a bash script that will be incorporated into the ASG’s Launch Template. This script will serve as the EC2 instance’s user data, automating the installation and initiation of the Apache web service, as well as the creation of a personalized web page.

Let’s review the bash script below —

```bash
#!/bin/bash
# Use this for your user data (script from top to bottom)
# install httpd (Linux 2 version)
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1> Hello! Up The Chels from $(hostname -f)</h1>" > /var/www/html/index.html
```
## Code Explanation

In this script, we first update all the yum package repositories, then install the Apache Web Server package using yum. The we start and enable the Apache service to ensure that it starts on boot.

The script also installs the Extra Packages for Enterprise Linux (EPEL) repository which allows for easy installation of commonly used software packages.

Additionally, the script installs the stress package, which can be used to test the instance under heavy workloads. Finally, the script adds a custom webpage with a Hello, Up The Chelsea TECH! message to the default index.html file located in the Apache document root directory /var/www/html/.

## ASG Creation and Configuration

With our ASG’s Security Group and Bash Script created we can now go ahead and now create and configure our ASG —

```yaml

# Create an autoscaling group with the specified configurations
resource "aws_autoscaling_group" "asg" {
  name                = var.asg_name
  min_size            = var.asg_min
  max_size            = var.asg_max
  desired_capacity    = var.asg_des_cap
  vpc_zone_identifier = [aws_subnet.priv-sub1.id, aws_subnet.priv-sub2.id]
  launch_template {
    id = aws_launch_template.lt-asg.id
  }

  # Tag the autoscaling group for easier identification
  tag {
    key                 = "Name"
    value               = "Private Sub ASG"
    propagate_at_launch = true
  }
}

# Create a launch template with the specified configurations
resource "aws_launch_template" "lt-asg" {
  name                   = var.lt_asg_name
  image_id               = var.lt_asg_ami
  instance_type          = var.lt_asg_instance_type
  key_name               = var.lt_asg_key
  vpc_security_group_ids = [aws_security_group.lt-sg.id]
  user_data              = filebase64("${path.root}/install-apache.sh")
}

# Attach the autoscaling group to the target group of the ALB
resource "aws_autoscaling_attachment" "asg-tg-attach" {
  autoscaling_group_name = aws_autoscaling_group.asg.id
  lb_target_group_arn    = aws_lb_target_group.alb-tg.arn
}
```

## Code Explanation

Upon creating the ASG, we need to set a minimum, maximum, and desired capacities. Additionally, the ASG is launched in two private subnets that are identified by their subnet IDs.

A Launch Template is also created with configurations such as the Amazon Machine Image (AMI), instance type, and key name. Here we also specify our previously created Bash Script as a user data file that is encoded as base64. This runs a script to install and start the Apache Web Server and create the custom web page.

Finally, the ASG is attached to the Target Group of our ALB using an autoscaling attachment. This enables the ASG to automatically balance the load across instances in response to incoming traffic, ensuring optimal performance and availability.

On to Step 6 — Utilizing the output.tf file to obtain public DNS of ALB.

## Step 6: Utilize output.tf file to obtain public DNS of ALB

After creating the ALB, it can be tedious to manually access the ALB’s public DNS required to send traffic to it. Fortunately, we cam create an output.tf file in Terraform which provides an easy solution to instantly obtain the public DNS of the ALB and ensure that incoming traffic is correctly routed to the instances in the target group.

Let’s review this output.tf file below —

```yaml
# Output
output "vpc-id" {
  value = aws_vpc.terraform-vpc.id
}

output "pub-sub1-id" {
  value = aws_subnet.pub-sub1.id
}

output "pub-sub2-id" {
  value = aws_subnet.pub-sub2.id
}

output "priv-sub1-id" {
  value = aws_subnet.priv-sub1.id
}

output "priv-sub2-id" {
  value = aws_subnet.priv-sub2.id
}

output "terraform-igw" {
  value = aws_internet_gateway.terraform-igw.id
}

output "alb-dns" {
  value = aws_lb.pub-sub-alb.dns_name
}
```

## Code Explanation

Outputs allow you to see the values of the resources created in your Terraform infrastructure. Here we create several outputs for our Terraform infrastructure, each output refers to a different resource.

Our outputs include the IDs of the VPC, the public and private subnets, the Internet Gateway and the DNS name of the ALB. These outputs will be used to provide us information about the our infrastructure.

Now, let’s hop to Step 7 — creating variables for dynamic of resource arguments.

## Step 7: Create Variables for dynamic values of resource arguments

When creating Infrastructure as Code (IaC) using Terraform it’s common to use dynamic values for resource arguments. These dynamic values can be based on user input, environment variables, or outputs from other resources.

To manage these dynamic values, it’s important to use variables. Variables help to ensure consistency across deployments, make code easier to read and maintain, and enable reusability of code.

Let’s review the code below on the many variables we have defined —

```yaml

variable "aws_region" {
  default = "us-west-1"
  type    = string
}

variable "vpc_name" {
  default = "Two Tier VPC"
  type    = string
}

variable "availability_zone-1" {
  default = "us-west-1a"
  type    = string
}

variable "availability_zone-2" {
  default = "us-west-1b"
  type    = string
}

variable "auto-assign-ip" {
  default = "true"
  type    = bool
}

variable "pub-sub1-name" {
  default = "Public Subnet 1"
  type    = string
}

variable "pub-sub2-name" {
  default = "Public Subnet 2"
  type    = string
}

variable "priv-sub1-name" {
  default = "Private Subnet 1"
  type    = string
}

variable "priv-sub2-name" {
  default = "Private Subnet 2"
  type    = string
}

variable "igw-name" {
  default = "Internet Gatway"
  type    = string
}

variable "nat-gw-name" {
  default = "Nat Gateway"
  type    = string
}

variable "pub-rt-name" {
  default = "Public Route Table"
  type    = string
}

variable "priv-rt-name" {
  default = "Private Route Table"
  type    = string
}

variable "vpc-cidr" {
  default = "10.0.0.0/16"
  type    = string
}

variable "alb_sg_name" {
  type        = string
  description = "Name of the App Load Balancer security group"
  default     = "alb-sg"
}


variable "http_port" {
  type        = number
  description = "Port for HTTP traffic"
  default     = 80
}

variable "ssh_port" {
  type        = number
  description = "Port for SSH traffic"
  default     = 22
}

variable "alb_sg_ingress_cidr_blocks" {
  type        = list(string)
  description = "List of CIDR blocks to allow inbound traffic to the App Load Balancer security group"
  default     = ["0.0.0.0/0"]
}

variable "alb_sg_egress_cidr_blocks" {
  type        = list(string)
  description = "List of CIDR blocks to allow outbound traffic from the App Load Balancer security group"
  default     = ["0.0.0.0/0"]
}

variable "load_balancer_name" {
  type        = string
  description = "Name of the load balancer"
  default     = "pub-sub-alb"
}

variable "target_group_name" {
  type        = string
  description = "Name of the target group"
  default     = "alb-tg"
}

variable "lt_sg_name" {
  type        = string
  description = "Name of the ASG security group"
  default     = "Security Group for ASG"
}

variable "lt_sg_egress_cidr_blocks" {
  type        = list(string)
  description = "List of CIDR blocks to allow outbound traffic from the ASG security group"
  default     = ["0.0.0.0/0"]
}

variable "asg_name" {
  type        = string
  description = "Name of the ASG "
  default     = "ASG"
}

variable "lt_asg_name" {
  type        = string
  description = "Name of the Launch Template "
  default     = "lt-asg"
}

variable "lt_asg_ami" {
  type        = string
  description = "Amazon Linux 2 Ami ID"
  default     = "ami-0aa117785d1c1bfe5"
}

variable "lt_asg_instance_type" {
  type        = string
  description = "T2 Micro Instance Type"
  default     = "t2.micro"
}

variable "lt_asg_key" {
  type        = string
  description = "Key Pair"
  default     = "tatendaKeypair"
}

variable "asg_min" {
  type        = number
  description = "ASG Min Size"
  default     = 2
}

variable "asg_max" {
  type        = number
  description = "ASG Max Size"
  default     = 5
}

variable "asg_des_cap" {
  type        = number
  description = "ASG Desired Capacity"
  default     = 2
}


variable "pub_rt_cidr" {
  type        = string
  description = "CIDR block to route taffic from internet gateway to anywhere"
  default     = "0.0.0.0/0"
}

variable "priv_rt_cidr" {
  type        = string
  description = "CIDR block to route taffic from private subnet to natgateway"
  default     = "0.0.0.0/0"
}


variable "pub_sub1_cidr" {
  default = "10.0.1.0/24"
  type    = string
  
}

variable "pub_sub2_cidr" {
  default = "10.0.2.0/24"
  type    = string
}

variable "priv_sub1_cidr" {
  default = "10.0.3.0/24"
  type    = string
}

variable "priv_sub2_cidr" {
  default = "10.0.4.0/24"
  type    = string
}
```

## Code Explanation

Here we define a series of variables that allows for dynamic values to be used in this Terraform project.

Each variable has a name, a default value, and a type. The default value is what the variable will be set to if no other value is specified when running the Terraform code. The type specifies the expected data type of the variable.

For example, the aws_region variable has a default value of us-east-1 and a type of string, which means that it expects a string value. Similarly, the auto-assign-ip variable has a default value of true and a type of bool, which means that it expects a boolean value.

Some of the variables are related to the networking components of the infrastructure, which specify values like the names of the VPC, public subnets, and private subnets.

By defining variables, the values can be easily changed without modifying the main configuration file.

Alright! Now that we’ve reviewed all our Terraform configuration files, we can proceed to Step 8 — Running though Terraform workflow to deploy our defined infrastructure

## Step 8: Run Terraform workflow to initialize, validate, plan then apply

In your Visual Code Studio terminal, to initialize the necessary providers, execute the following command in your Visual Code Studio terminal —

```command
terraform init
```

Upon completion of the initialization process, a successful prompt will be displayed, as shown below.

![image alt](https://github.com/Tatenda-Prince/Terraform-Deploying-Secure-Highly-Available-Fault-Tolerant-Cloud-Infrastructure-/blob/f3bfcb4e7b86d7963611df79c0d25ac5c5cfaa8f/images/Screenshot%202025-01-05%20160457.png)

Next, let’s ensure that our code does not contain any syntax errors by running the following command —

```command
terraform validate
```
The command should generate a success message, confirming that it is valid, as demonstrated below.

![image alt](https://github.com/Tatenda-Prince/Terraform-Deploying-Secure-Highly-Available-Fault-Tolerant-Cloud-Infrastructure-/blob/9967954009418a96d55045b16acb73aaaa117528/images/Screenshot%202025-01-05%20160533.png)


Let’s now execute the following command to generate a list of all the modifications that Terraform will apply. —

```command
terraform plan
```
The list of changes that Terraform is anticipated to apply to the infrastructure resources should be displayed. The “+” sign indicates what will be added, while the “-” sign indicates what will be removed.

![image alt](https://github.com/Tatenda-Prince/Terraform-Deploying-Secure-Highly-Available-Fault-Tolerant-Cloud-Infrastructure-/blob/3a49fb9f4d7ec4da95309ba91563e16a14cf19c1/images/Screenshot%202025-01-05%20160623.png)

Now, let’s deploy this infrastructure! Execute the following command to apply the changes and deploy the resources.

Note — Make sure to type “yes” to agree to the changes after running this command

```command
terraform apply
```
Terraform will initiate the process of applying all the changes to the infrastructure. Kindly wait for a few seconds for the deployment process to complete.

![image](https://github.com/Tatenda-Prince/Terraform-Deploying-Secure-Highly-Available-Fault-Tolerant-Cloud-Infrastructure-/blob/3dacb794d4c12fe33df01b4610f67145b6e1a311/images/Screenshot%202025-01-05%20160737.png)

## Success!

The process should now conclude with a message indicating “Apply complete”, stating the total number of added, modified, and destroyed resources, accompanied by several resource outputs.

Please copy and save the ALB’s DNS URL, which will be required to access the web page from the browser.

![image alt](https://github.com/Tatenda-Prince/Terraform-Deploying-Secure-Highly-Available-Fault-Tolerant-Cloud-Infrastructure-/blob/0c8a6d41164cb0660c71b8401c28a9b71bd6b2eb/images/Screenshot%202025-01-05%20161205.png)

Now, let’s verify that our resources have been created from reviewing them in the Management Console.

## Step 9: Verify creation of EC2 Instance from ASG and Target group’s health status

In the AWS Management Console, head to the EC2 dashboard and verify that the two servers were launched from the ASG.

![image alt](https://github.com/Tatenda-Prince/Terraform-Deploying-Secure-Highly-Available-Fault-Tolerant-Cloud-Infrastructure-/blob/6d59c9902334270b17742913d9011a3754aa6f8c/images/Screenshot%202025-01-05%20155701.png)

Also, navigate to the left pane, scroll down and select Target groups. Select the created Target group, scroll down, then verify that the instances Health status is healthy, as shown below.

![image alt](https://github.com/Tatenda-Prince/Terraform-Deploying-Secure-Highly-Available-Fault-Tolerant-Cloud-Infrastructure-/blob/30b076da01e71cfab5435c473829b47461e1ff73/images/Screenshot%202025-01-05%20155740.png)

Great! We’ve successfully confirmed that our ASG has generated the expected EC2 instances, and all of our Target groups’ health statuses are displaying as healthy. Let’s now verify that we can access our ASG’s web servers through our ALB.

## Step 10: Verify reachability to the Web server using its URL in browser

Open up your desired browser and paste the ALB’s DNS URL in your browser.

Note — Make sure to use the “http://” protocol and not https:// to reach the ALB.

![image alt](https://github.com/Tatenda-Prince/Terraform-Deploying-Secure-Highly-Available-Fault-Tolerant-Cloud-Infrastructure-/blob/a74c35d71163d3407b39982f2369aa9e10bced5d/images/Screenshot%202025-01-05%20155833.png)

## Congratulations! 

ou’ve successfully completed “Terraform Auto Scaling”. You’ve learned how to leverage Terraform to build a reliable and scalable cloud infrastructure that can handle high traffic loads and maintain uptime, even in the event of failures in individual components.

# Clean up

## Destroy infrastructure

Run the follow command to remove/delete/tear down all the resources previously provisioned from Terraform —

```command
terraform destroy
```

Wait for it to complete. At the end, you should receive a prompt stating Destroy complete along with how many resources were destroyed.





















