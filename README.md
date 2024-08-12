# aws-kub-iac

# Network Infrastructure Overview

## 1. VPC (Virtual Private Cloud)

The VPC (Virtual Private Cloud) serves as the foundational network structure where all your AWS resources will reside. The VPC isolates your network environment from other AWS customers, ensuring security and manageability.

    CIDR Block: The VPC is created with a CIDR block (e.g., 10.0.0.0/16), which determines the IP address range within the VPC.

## 2. Subnets

Subnets are subdivisions of the VPC that allow you to place resources in different availability zones for high availability and fault tolerance.

    Public Subnet:
        These subnets are configured to allow internet access, as they are associated with an Internet Gateway.
        Instances in public subnets can have public IPs and are accessible from the internet.
        Generally used for resources like load balancers or bastion hosts.

    Private Subnet:
        These subnets do not have direct internet access, enhancing security for resources like databases and application servers.
        Outbound internet access from these subnets is managed via a NAT Gateway.

## 3. Internet Gateway

An Internet Gateway is attached to the VPC to allow internet access to the public subnets. This enables instances in the public subnets to send and receive traffic from the internet.
## 4. NAT Gateway

The NAT Gateway is deployed in the public subnet to provide outbound internet connectivity for instances in the private subnets, while keeping them secure from inbound internet traffic.
## 5. Route Tables

    Public Route Table:
        Associated with the public subnets, routes traffic destined for the internet through the Internet Gateway.

    Private Route Table:
        Associated with the private subnets, routes internet-bound traffic through the NAT Gateway, ensuring that private resources have outbound internet access without being exposed directly to the internet.

## 6. Network ACLs and Security Groups

    Network ACLs (Access Control Lists):
        Provide a stateless layer of security at the subnet level, controlling inbound and outbound traffic for each subnet.

    Security Groups:
        Provide a stateful layer of security at the instance level, controlling the inbound and outbound traffic for specific instances or resources.

# Infrastructure Overview
## 1. EKS (Elastic Kubernetes Service) Cluster

EKS is a managed Kubernetes service that simplifies running Kubernetes on AWS without the need to manage your own Kubernetes control plane.

    EKS Cluster:
        The core of your Kubernetes infrastructure, managing containers and orchestrating workloads.
        It is deployed across the VPC’s subnets for high availability.

    Node Group:
        This is a set of EC2 instances that serve as worker nodes in your EKS cluster. The node group is autoscaled within the defined limits (MinSize, DesiredCapacity, MaxSize) to handle varying workloads.

## 2. RDS (Relational Database Service)

Amazon RDS is a managed relational database service that simplifies database management tasks such as backups, scaling, and patching.

    RDS Instance:
        A PostgreSQL database instance deployed in a private subnet to securely store and manage application data.
        The database credentials are stored securely in AWS Secrets Manager, and the database itself is not publicly accessible.

    RDS Subnet Group:
        This defines the subnets where the RDS instance will be deployed, ensuring it is placed within the private subnets of your VPC.

## 3. ECR (Elastic Container Registry)

ECR is a fully managed Docker container registry that makes it easy for developers to store, manage, and deploy Docker container images.

    ECR Repository:
        Stores Docker images built by your CI/CD pipeline. These images are versioned and scanned for vulnerabilities.

## 4. CodeBuild

CodeBuild is a fully managed continuous integration service that compiles source code, runs tests, and produces software packages.

    CodeBuild Project:
        Builds Docker images from your application's source code and pushes them to the ECR repository. This project is configured to use a GitHub repository as the source and integrates with the ECR repository.

## 5. Load Balancer

An Application Load Balancer (ALB) distributes incoming application traffic across multiple targets, such as EC2 instances, in different availability zones.

    Load Balancer:
        Deployed in the public subnet, it routes traffic to the appropriate instances in the private subnet, providing scalability and fault tolerance for your application.

## 6. Bastion Host

A Bastion Host is an EC2 instance that acts as a gateway for SSH access to the instances in your private subnet.

    Bastion Host:
        Deployed in the public subnet, it allows secure access to your private resources via SSH from a specified IP address.

## 7. CloudWatch Alarms and Monitoring

Amazon CloudWatch monitors your AWS resources and applications in real-time.

    CPU Utilization Alarm:
        Monitors the CPU utilization of the EKS node group and triggers alerts if it exceeds a defined threshold, helping you maintain optimal performance.
    SNS Topic:
        Used to send notifications when an alarm is triggered, ensuring you are alerted promptly about any critical issues.

# Outputs

    EKSClusterName:
        The name of the EKS cluster for easy reference.

    LoadBalancerDNS:
        The DNS name of the Load Balancer, which can be used to access the application deployed on the EKS cluster.

    ECRRepositoryURI:
        The URI of the ECR repository, which is used to store and pull Docker images.

    RDSDBInstanceEndpoint:
        The endpoint of the RDS database, required for connecting the application to the database.

