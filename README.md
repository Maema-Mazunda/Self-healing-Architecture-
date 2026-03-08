# Self-healing-Architecture-
# Self-Healing Architecture 🏗️

A fully automated, self-healing web application infrastructure on AWS, provisioned with Terraform.

If any EC2 instance fails, the Auto Scaling Group automatically replaces it — no manual intervention needed.

---

## 📌 What This Project Does

This project spins up a highly available web server environment on AWS. Traffic from the internet flows through a Load Balancer, which distributes it across EC2 instances running in two separate Availability Zones. If an instance goes down, AWS automatically replaces it and re-registers it with the Load Balancer.

---

## 🏛️ Architecture
```
Internet
   │
   ▼
Application Load Balancer (ALB)
   │
   ▼
ALB Listener (HTTP:80)
   │
   ▼
Target Group
   │
   ▼
┌─────────────────────────────────────────┐
│              VPC (10.0.0.0/16)          │
│  ┌──────────────────────────────────┐   │
│  │        Security Group            │   │
│  │  Inbound: TCP:80  Outbound: ALL  │   │
│  │  ┌────────────────────────────┐  │   │
│  │  │     Auto Scaling Group     │  │   │
│  │  │   min:1  desired:2  max:3  │  │   │
│  │  │                            │  │   │
│  │  │  ┌──────────┐ ┌─────────┐ │  │   │
│  │  │  │ Subnet 1 │ │Subnet 2 │ │  │   │
│  │  │  │us-east-1a│ │us-east-1b│ │  │   │
│  │  │  │  EC2 💻  │ │  EC2 💻 │ │  │   │
│  │  │  │ t2.micro │ │ t2.micro│ │  │   │
│  │  │  └──────────┘ └─────────┘ │  │   │
│  │  └────────────────────────────┘  │   │
│  └──────────────────────────────────┘   │
│                                         │
│  Internet Gateway ↔ Route Table         │
└─────────────────────────────────────────┘
```

---

## 🧱 AWS Resources Provisioned

| Resource | Terraform Name | Purpose |
|---|---|---|
| VPC | `aws_vpc.main` | Isolated network — CIDR 10.0.0.0/16 |
| Public Subnet 1 | `aws_subnet.public_subnet_1` | AZ-1, 10.0.1.0/24 |
| Public Subnet 2 | `aws_subnet.public_subnet_2` | AZ-2, 10.0.2.0/24 |
| Internet Gateway | `aws_internet_gateway.main_igw` | Connects VPC to the internet |
| Route Table | `aws_route_table.public_rt` | Routes 0.0.0.0/0 to IGW |
| Security Group | `aws_security_group.web_sg` | Allows HTTP:80 in, all traffic out |
| Load Balancer | `aws_lb.app_lb` | Distributes traffic across instances |
| ALB Listener | `aws_lb_listener.app_listener` | Listens on port 80, forwards to target group |
| Target Group | `aws_lb_target_group.app_tg` | Registers and health-checks EC2 instances |
| Launch Template | `aws_launch_template.app_template` | Defines EC2 config + bootstraps Apache |
| Auto Scaling Group | `aws_autoscaling_group.app_asg` | Keeps instances healthy and scaled |

---

## ⚙️ How the Self-Healing Works

1. The Auto Scaling Group monitors EC2 instance health continuously.
2. If an instance becomes unhealthy or is terminated, ASG launches a replacement automatically using the Launch Template.
3. The new instance runs the `user_data` bootstrap script, which installs and starts Apache HTTPD.
4. Once healthy, the Target Group registers the new instance and the Load Balancer begins sending it traffic.

The minimum instance count is **1**, desired is **2**, and maximum is **3** — so the app stays online even during scaling events or instance failures.

---

## 🖥️ EC2 Bootstrap (user_data)

Every EC2 instance automatically runs this on launch:
```bash
#!/bin/bash
sudo yum update -y
sudo yum install -y httpd
echo "Hello, World" > /var/www/html/index.html
sudo systemctl start httpd
sudo systemctl enable httpd
```

This installs Apache and serves a simple HTML page on port 80.

---

## 🚀 Getting Started

### Prerequisites

- [Terraform](https://developer.hashicorp.com/terraform/install) installed
- AWS CLI configured with valid credentials (`aws configure`)
- An AWS account with permissions to create EC2, VPC, and ELB resources

### Deploy
```bash
# Clone the repo
git clone https://github.com/Maema-Mazunda/Self-healing-Architecture-.git
cd Self-healing-Architecture-

# Initialise Terraform
terraform init

# Preview what will be created
terraform plan

# Deploy the infrastructure
terraform apply
```

### Destroy
```bash
terraform destroy
```

---

## 📁 Project Structure
```
Self-healing-Architecture-/
├── main.tf          # All AWS resource definitions
├── .gitignore       # Excludes .terraform/, state files, credentials
└── README.md        # This file
```

---

## ⚠️ Important Notes

- Replace the placeholder `image_id` in `aws_launch_template` with a real AMI ID for your region.
- This setup uses **public subnets only** — suitable for learning and demos. For production, place EC2 instances in private subnets behind a NAT Gateway.
- Never commit AWS credentials or `.tfstate` files to version control.

---

## 🛠️ Built With

- [Terraform](https://www.terraform.io/) — Infrastructure as Code
- [AWS EC2](https://aws.amazon.com/ec2/) — Compute
- [AWS ALB](https://aws.amazon.com/elasticloadbalancing/) — Load Balancing
- [AWS Auto Scaling](https://aws.amazon.com/autoscaling/) — Self-healing
- [AWS VPC](https://aws.amazon.com/vpc/) — Networking