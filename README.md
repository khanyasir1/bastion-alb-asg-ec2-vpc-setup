

# 🛡️ AWS VPC with Auto Scaling Group, Private Subnet EC2, Bastion Host & Application Load Balancer (ALB)

## 📌 Project Overview

This project demonstrates how to architect a secure, scalable, and web-accessible infrastructure on AWS using:

* **Custom VPC with Public & Private Subnets**
* **Auto Scaling Group (ASG) for backend EC2s**
* **Launch Template**
* **Bastion Host (Jump Server)**
* **Application Load Balancer (ALB)**
* **Private Web Server accessible via ALB**
* **Secure SSH Access via Bastion Host**

> ✅ Perfect for learning **AWS Networking**, **Scalability**, **Security**, and **Load Balancing Concepts**

---

## 📸 Architecture Diagram

> 🔧 
![VPC Architecture](https://docs.aws.amazon.com/images/vpc/latest/userguide/images/vpc-example-private-subnets.png)

```
Your Laptop
   |
   | SSH
   ↓
Bastion Host (Public Subnet)
   |
   | SSH
   ↓
EC2 (Private Subnet, Auto-Scaling Group)
   ↑
   |  HTTP 8000
ALB (Public Subnet)
   ↑
   |  HTTP 80
User Browser
```

---

## 🔧 Step-by-Step Setup Guide

---

### 🪪 Step 1: IAM User

> Create user with these policies:

* `AmazonEC2FullAccess`
* `AmazonVPCFullAccess`
* `ElasticLoadBalancingFullAccess`
* `AutoScalingFullAccess`

---

### 🌐 Step 2: Create Custom VPC

Use **"VPC and More"** option:

* **CIDR Block**: `10.0.0.0/16`
* **2 Public + 2 Private Subnets** across 2 AZs
* **NAT Gateways**: 1 per AZ
* **Enable DNS Hostnames**
* **Route Tables**:

  * Public → IGW
  * Private → NAT

---
![ VPC ](/vpc-project-images/vpc-image.png)

### 🚀 Step 3: Create Launch Template

* **AMI**: Ubuntu/Amazon Linux
* **Instance Type**: `t2.micro`
* **Key Pair**: Use your existing or create new
* **Security Group**:

  * Inbound:

    * SSH (22) from Bastion or your IP
    * TCP 8000 from ALB (internal traffic only)
  * Outbound: Allow all

---
![ Launch-Template ](/vpc-project-images/luach-template-page.png)

### ⚙️ Step 4: Create Auto Scaling Group

* **Use Launch Template**
* **VPC**: Your custom VPC
* **Subnets**: Private Subnets only
* **Desired/Min/Max**: 2 / 1 / 4
* **Skip Load Balancer** (will attach manually later)

✅ Verify: 2 EC2s in **private subnet** launched

---
![ Auto-Scaling Group - Create](/vpc-project-images/ASG-creation.png)
![ Auto-Scaling Group - Home ](/vpc-project-images/ASG-home.png)

### 🧩 Step 5: Bastion Host

* **Launch EC2** in **public subnet**
* **Auto-assign Public IP**: Enabled
* **Security Group**: SSH (22) from your IP

---
![  Bastion Host ](/vpc-project-images/jump-host.png)

### 🔐 Step 6: SSH Access Flow

1. **Local → Bastion**:

   ```bash
   ssh -i your-key.pem ubuntu@<Bastion-IP>
   ```

2. **Transfer PEM to Bastion**:

   ```bash
   scp -i your-key.pem your-key.pem ubuntu@<Bastion-IP>:/home/ubuntu
   ```

3. **Bastion → Private EC2**:

   ```bash
   ssh -i your-key.pem ubuntu@<Private-IP>
   ```

---

### 🧾 Step 7: Host HTML Page (on Private EC2s)

On each private EC2:

1. Create file:

   ```bash
   vim index.html
   ```
2. Paste the following:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Private Subnet - AWS VPC</title>
  <style>
    body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background: #f0f4f8; color: #333; }
    header { background-color: #1e3a8a; color: #fff; padding: 20px; text-align: center; }
    main { max-width: 800px; margin: 30px auto; background: #fff; padding: 30px; border-radius: 10px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); }
    .diagram { border: 2px dashed #ccc; padding: 30px; margin: 20px 0; background: #fafafa; text-align: center; }
  </style>
</head>
<body>
  <header><h1>Private Subnet - AWS VPC</h1><p>Secure your infrastructure with isolated cloud networking</p></header>
  <main>
    <h2>🌐 What is a Private Subnet?</h2>
    <p>A <strong>Private Subnet</strong> is isolated from the internet. Use it for backend apps, databases, etc.</p>
    <h2>🗺️ Network Diagram</h2>
    <div class="diagram">[Diagram Placeholder]</div>
    <h2>🔧 Use Cases</h2>
    <ul><li>Host RDS securely</li><li>Backend microservices</li><li>Job processors</li></ul>
    <h2>✅ Best Practices</h2>
    <ul><li>Use NAT Gateway for outbound traffic</li><li>Use Security Groups wisely</li><li>Enable VPC Flow Logs</li></ul>
  </main>
  <footer style="text-align:center; padding:20px;">&copy; 2025 Cloud Architecture Guide</footer>
</body>
</html>
```

3. Run web server:

   ```bash
   python3 -m http.server 8000
   ```

---

### 🧭 Step 8: Create Application Load Balancer (ALB)

#### 🔷 Basic Setup

* Go to **EC2 > Load Balancers > Create ALB**
* **Name**: `web-alb`
* **Scheme**: `Internet-facing`
* **IP Address Type**: `IPv4`

#### 🌍 Network Mapping

* **VPC**: Select your custom VPC
* **AZs & Subnets**: Select **both public subnets**
* **Security Group**: Use same SG as ASG but allow `HTTP (80)` from anywhere

#### 🔁 Listener & Routing

* **Listener**:

  * Protocol: `HTTP`
  * Port: `80`

![  Bastion Host ](/vpc-project-images/jump-host.png)

#### 🎯 Create Target Group

* **Target Type**: `Instance`
* **Name**: `private-ec2-tg`
* **Protocol**: `HTTP`
* **Port**: `8000`
* **VPC**: your custom VPC
* **Health Check Path**: `/`

✅ **Register Targets**: Select all EC2 instances in **private subnet**

---

### ✅ Final Step: Security Group Fix

If ALB shows `port 80 not reachable`, do this:

1. Go to **Security Group** attached to ALB
2. Add Inbound Rule:

   * Type: `HTTP`
   * Port: `80`
   * Source: `Anywhere (0.0.0.0/0)`

Now open ALB DNS in browser:

```
http://<your-alb-dns-name>
```

You should see the hosted **index.html** page!

---
![  private-subnet-01 ](/vpc-project-images/az-1.png)
![  private-subnet-01 ](/vpc-project-images/az-2.png)

## 📁 Project File Structure

```
├── README.md
└── vpc-project-images
    ├── 2-private-subnet-instance.png
    ├── ASG-creation.png
    ├── ASG-home.png
    ├── ASG-template.png
    ├── az-1.png
    ├── az-2.png
    ├── jump-host.png
    ├── luach-template-page.png
    ├── scp.png
    └── vpc-image.png

```

---

## 🧠 Best Practices Recap

* ❌ No internet to private EC2s directly
* 🔐 Tight Security Groups
* 🧪 ALB health checks for monitoring
* 🌍 Public web access via Load Balancer
* 📈 Auto Scaling ensures resilience

---

## 💬 Interview Tips

* Explain **private vs. public subnet** use cases
* Talk about **ALB vs NLB**
* Discuss **why port 8000** is used internally
* How ALB forwards traffic from `80 → 8000`
* Mention **scalability and secure access via Bastion**

---



