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

# How the networkin works

1. EKS Cluster in a Private Subnet

The EKS cluster, which manages your Kubernetes workloads, is primarily placed in the private subnet. This setup ensures that the nodes (EC2 instances) running your applications are not directly exposed to the internet, providing a higher level of security.

    Private Subnet: By placing the EKS nodes in a private subnet, you ensure that the applications and services running on these nodes are only accessible from within the VPC or through a controlled entry point like a bastion host or a load balancer.

2. Bastion Host for Secure Access

The Bastion Host serves as a secure gateway to access the resources in your private subnets, such as the EKS nodes.

    Public Subnet Placement: The bastion host is placed in a public subnet with an associated public IP address. This allows you to SSH into the bastion host from your home IP address or another trusted location.

    SSH Access: From the bastion host, you can securely SSH into the EKS nodes or other resources in the private subnet. This approach is considered more secure because it restricts direct access to private instances from the internet.

3. Load Balancer for Exposing Services

The Application Load Balancer (ALB) in the public subnet is responsible for directing traffic to your applications running in the EKS cluster.

    Public Subnet Placement: The ALB is placed in the public subnet to accept traffic from the internet.

    Targeting EKS Nodes: The ALB can route traffic to the EKS nodes running in the private subnet, thereby exposing your services to the outside world without directly exposing the EKS nodes.

4. RDS Instance in Private Subnet

The RDS database instance is deployed in the private subnet, ensuring that it is not directly accessible from the internet.

    Private Subnet Placement: This configuration secures the database from external threats, with access restricted to applications running within the same VPC, typically via the EKS nodes.

    Security and Management: The database credentials are securely managed using AWS Secrets Manager, and the database itself is managed by RDS, which handles backups, patching, and scaling.

5. CodeBuild, ECR, and CI/CD Integration

    ECR Repository: Stores Docker images that are built by your CI/CD pipeline using CodeBuild. These images can then be deployed on the EKS cluster.

    CodeBuild: Automates the process of building, testing, and deploying your application. It pulls the source code from a GitHub repository, builds a Docker image, and pushes it to ECR.

### How It All Ties Together

    Deployment Workflow:
        Developers push changes to the GitHub repository.
        CodeBuild triggers a build, creates a Docker image, and pushes it to the ECR repository.
        Kubernetes deployments in EKS pull the latest Docker image from ECR and deploy the application.
        ALB routes incoming traffic to the EKS nodes that are running the application.

    Access and Management:
        Use the bastion host to securely manage and access the EKS nodes.
        Use Secrets Manager to securely store and manage database credentials.

### Considerations

    Direct Interaction with Kubernetes:
        If you need to interact with the Kubernetes API directly, you would typically do this from the bastion host. The bastion host can have kubectl installed and configured with the necessary permissions to manage the cluster.

    Networking and Security:
        Ensure that security groups and network ACLs are correctly configured to allow the necessary traffic between components while keeping everything secure.