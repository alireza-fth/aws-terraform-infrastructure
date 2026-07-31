# AWS Infrastructure with Terraform

Infrastructure as Code (IaC) project for deploying a public web server on AWS using Terraform.

The project provisions a complete AWS networking environment, launches an EC2 instance, installs Nginx automatically, and exposes the web server to the internet.

## Architecture

```text
                         Internet
                            │
                            ▼
                  ┌──────────────────┐
                  │ Internet Gateway │
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │   Route Table    │
                  │  0.0.0.0/0 → IGW │
                  └────────┬─────────┘
                           │
                           ▼
              ┌─────────────────────────┐
              │      Public Subnet      │
              │       10.0.1.0/24       │
              │                         │
              │   ┌─────────────────┐   │
              │   │ EC2 t3.micro    │   │
              │   │ Amazon Linux    │   │
              │   │                 │   │
              │   │ Nginx           │   │
              │   └─────────────────┘   │
              └─────────────────────────┘
                           │
                    Security Group
                    ┌──────┴──────┐
                    │             │
                  SSH :22      HTTP :80
```

## Project Overview

This project demonstrates how to provision and manage AWS infrastructure using Terraform instead of manually creating resources through the AWS Management Console.

The infrastructure includes:

* AWS VPC
* Public Subnet
* Internet Gateway
* Public Route Table
* Route Table Association
* Security Group
* EC2 Instance
* AWS Key Pair
* Nginx Web Server
* Terraform Outputs

The EC2 instance is automatically configured using Terraform `user_data`.

## Technologies

* **AWS**
* **Terraform**
* **Amazon Linux 2023**
* **Nginx**
* **AWS CLI**
* **Git & GitHub**
* **PowerShell**

## AWS Region

The infrastructure is deployed in:

```text
eu-central-1
```

AWS Region:

```text
Frankfurt
```

## Terraform Resources

### VPC

```text
CIDR: 10.0.0.0/16
```

### Public Subnet

```text
CIDR: 10.0.1.0/24
Availability Zone: eu-central-1a
```

The subnet is configured to assign public IPv4 addresses to launched instances.

### Internet Gateway

The Internet Gateway provides connectivity between the VPC and the public internet.

### Route Table

The public route table contains:

```text
Destination: 10.0.0.0/16
Target: local

Destination: 0.0.0.0/0
Target: Internet Gateway
```

### Security Group

Inbound traffic:

| Protocol | Port | Source    | Purpose |
| -------- | ---: | --------- | ------- |
| TCP      |   22 | 0.0.0.0/0 | SSH     |
| TCP      |   80 | 0.0.0.0/0 | HTTP    |

Outbound traffic is allowed.

> **Security note:** SSH access from `0.0.0.0/0` is acceptable for this learning project, but production environments should restrict SSH access to trusted IP addresses or use more secure access mechanisms.

### EC2

The project launches:

```text
Instance Type: t3.micro
Operating System: Amazon Linux 2023
```

The AMI is dynamically selected using a Terraform data source rather than hardcoding an AMI ID.

## Automatic Nginx Deployment

Nginx is installed automatically using EC2 `user_data`.

Terraform executes:

```bash
dnf update -y
dnf install -y nginx
systemctl enable nginx
systemctl start nginx
```

This means the server is ready to serve HTTP traffic immediately after provisioning.

## Project Structure

```text
aws-terraform-infrastructure/
│
├── terraform/
│   ├── main.tf
│   ├── provider.tf
│   ├── variables.tf
│   ├── outputs.tf
│   ├── key.tf
│   └── .terraform.lock.hcl
│
└── README.md
```

## Prerequisites

Before deploying the infrastructure, install and configure:

* AWS CLI
* Terraform
* Git
* An AWS account
* An SSH key pair

Verify AWS CLI authentication:

```powershell
aws sts get-caller-identity
```

Verify Terraform:

```powershell
terraform version
```

## Deployment

Navigate to the Terraform directory:

```powershell
cd terraform
```

Initialize Terraform:

```powershell
terraform init
```

Format the configuration:

```powershell
terraform fmt
```

Validate the configuration:

```powershell
terraform validate
```

Create an execution plan:

```powershell
terraform plan
```

Apply the infrastructure:

```powershell
terraform apply
```

Confirm with:

```text
yes
```

## Terraform Outputs

After deployment, Terraform provides useful infrastructure information:

```powershell
terraform output
```

For example:

```text
ec2_public_ip
ec2_public_dns
ec2_instance_id
vpc_id
public_subnet_id
security_group_id
internet_gateway_id
```

The EC2 public IP can also be retrieved with:

```powershell
terraform output ec2_public_ip
```

## Verification

After deployment, the Nginx web server can be tested from the local machine:

```powershell
curl.exe -v http://<EC2_PUBLIC_IP>
```

A successful deployment returns:

```text
HTTP/1.1 200 OK
Server: nginx
```

The response contains the default Nginx welcome page.

## SSH Access

The EC2 instance can be accessed using the configured SSH private key:

```powershell
ssh -i "$HOME\.ssh\id_ed25519" ec2-user@<EC2_PUBLIC_DNS>
```

Once connected, Nginx can be verified with:

```bash
sudo systemctl status nginx
```

Local HTTP connectivity can be tested with:

```bash
curl localhost
```

## Infrastructure Verification

The project was tested at multiple network layers:

### Security Group

Port 80 is allowed:

```text
TCP 80 → 0.0.0.0/0
```

### Route Table

```text
0.0.0.0/0 → Internet Gateway
```

### Network ACL

The default Network ACL allows inbound and outbound traffic.

### EC2

```text
Instance State: Running
Status Checks: Passed
```

### Nginx

```text
Active: active (running)
```

### HTTP

```text
HTTP/1.1 200 OK
```

## Troubleshooting

During deployment, connectivity was tested using:

```powershell
Test-NetConnection <EC2_PUBLIC_IP> -Port 80
```

A successful result:

```text
TcpTestSucceeded : True
```

The HTTP service was also tested directly:

```powershell
curl.exe -v http://<EC2_PUBLIC_IP>
```

This confirmed that the complete path from the local machine to Nginx was operational.

## Cleanup

To remove all infrastructure created by Terraform:

```powershell
terraform destroy
```

Confirm with:

```text
yes
```

This prevents unnecessary AWS charges after completing the project.

## Key Cloud Engineering Concepts Demonstrated

This project demonstrates practical experience with:

* Infrastructure as Code
* AWS networking
* VPC architecture
* Public and private IP addressing
* Subnets
* Internet Gateways
* Route Tables
* Security Groups
* Network ACLs
* EC2
* Linux server administration
* SSH
* Nginx
* Automated server provisioning
* Terraform state management
* Terraform data sources
* Terraform outputs
* AWS CLI
* Cloud troubleshooting
* Git and GitHub

## Future Improvements

Planned improvements for future versions include:

* Restrict SSH access to a specific IP
* Use Terraform variables for configurable values
* Create reusable Terraform modules
* Add an Application Load Balancer
* Deploy multiple EC2 instances
* Implement Auto Scaling
* Add CloudWatch monitoring
* Add CI/CD with GitHub Actions
* Containerize the application with Docker
* Deploy containers using AWS ECS

## Author

**Alireza Fathihafshejani**

Master's Student in Software Engineering

Germany

---

## License

This project is intended for educational and portfolio purposes.
