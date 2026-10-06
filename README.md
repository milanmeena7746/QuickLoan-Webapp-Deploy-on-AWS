# QuickLoan – Scalable Web Application on AWS

A scalable cloud-hosted loan application deployed on **Amazon Web Services (AWS)** using PHP, Nginx, Amazon EC2, Amazon RDS MySQL, Amazon S3, Application Load Balancer, Auto Scaling, and VPC networking.

## 📌 Project Overview

**QuickLoan** is a cloud-hosted loan application with a PHP backend and HTML/CSS/JavaScript frontend.

The project demonstrates a scalable AWS infrastructure architecture covering:

* Networking
* Compute
* Database
* Object storage
* Load balancing
* Auto Scaling
* Secure server access
* High availability and fault tolerance

The application is deployed inside a custom **Amazon VPC** with public and private subnets. User traffic is handled by an **Application Load Balancer (ALB)** and distributed across EC2 instances managed by an **Auto Scaling Group**. Application data is stored in **Amazon RDS MySQL**, while static image assets are stored in **Amazon S3**.

---

## 🏗️ Architecture

```text
                         Internet
                            │
                            ▼
                ┌─────────────────────┐
                │   Application Load  │
                │      Balancer       │
                └──────────┬──────────┘
                           │
                 ┌─────────┴─────────┐
                 │                   │
                 ▼                   ▼
          ┌─────────────┐     ┌─────────────┐
          │  EC2 App 1  │     │  EC2 App 2  │
          │ Nginx + PHP │     │ Nginx + PHP │
          └──────┬──────┘     └──────┬──────┘
                 │                   │
                 └─────────┬─────────┘
                           │
                           ▼
                  ┌────────────────┐
                  │  Amazon RDS    │
                  │     MySQL      │
                  │ Private Subnet │
                  └────────────────┘

                           ▲
                           │
                    Application Data

                  ┌────────────────┐
                  │   Amazon S3    │
                  │ Static Assets  │
                  └────────────────┘

                    Static Images

          ┌─────────────────────────────┐
          │       Jump / Bastion        │
          │       Server (SSH)          │
          └─────────────────────────────┘
```

The documented architecture uses an ALB to forward requests to healthy EC2 instances managed by Auto Scaling. EC2 instances run Nginx and PHP, static images are served from S3, and loan application data is stored in RDS MySQL. A Jump Server is used as the secure SSH entry point.

---

## 🛠️ Technology Stack

| Layer                   | Technology / AWS Service  | Purpose                                |
| ----------------------- | ------------------------- | -------------------------------------- |
| Frontend                | HTML, CSS, JavaScript     | User interface                         |
| Backend                 | PHP 8.2, PHP-FPM          | Server-side application logic          |
| Web Server              | Nginx                     | HTTP request handling                  |
| Database                | Amazon RDS MySQL          | Persistent application data            |
| Object Storage          | Amazon S3                 | Static image assets                    |
| Networking              | Amazon VPC                | Isolated cloud network                 |
| Subnets                 | Public & Private Subnets  | Network segmentation                   |
| Internet Access         | Internet Gateway          | Public internet connectivity           |
| Private Outbound Access | NAT Gateway               | Internet access from private resources |
| Compute                 | Amazon EC2                | Application servers                    |
| Scaling                 | Auto Scaling Group        | Horizontal scaling                     |
| Image                   | Custom AMI                | Golden server image                    |
| Load Balancing          | Application Load Balancer | Traffic distribution                   |
| DNS                     | No-IP                     | Application domain                     |
| File Transfer           | WinSCP                    | Application file upload                |

The PDF specifies PHP, Nginx, MySQL and AWS as the core technology stack, with EC2, RDS, S3, VPC, ALB and Auto Scaling forming the AWS infrastructure.

---

# ☁️ AWS Infrastructure

## 1. VPC & Networking

A custom **Virtual Private Cloud (VPC)** was created for the QuickLoan application.

### Subnets

The infrastructure contains three subnets:

* Public Subnet 1 – Availability Zone A
* Public Subnet 2 – Availability Zone B
* Private Subnet 3 – Availability Zone C

### Internet Gateway

An **Internet Gateway (IGW)** is attached to the VPC to provide internet connectivity for resources in public subnets.

### NAT Gateway

A **NAT Gateway** is deployed in Public Subnet 1 and allows resources in private subnets to access the internet for outbound traffic.

### Route Tables

Two routing configurations are used:

```text
Public Route Table
        │
        └── Internet Gateway

Private Route Table
        │
        └── NAT Gateway
```

These components provide network segmentation between public-facing infrastructure and private resources.

---

# 🔐 Security Groups

Two major security groups are configured.

### Application & Jump Server Security Group

Allows:

* SSH – Port 22
* HTTP – Port 80

### Database Security Group

Allows:

* SSH – Port 22
* MySQL – Port 3306

Database access is restricted to the application server security group.

```text
Internet
   │
   ▼
Port 80
   │
   ▼
Application Server
   │
   │ Port 3306
   ▼
RDS MySQL
```

---

# 🖥️ EC2 Infrastructure

Three EC2 instances were deployed using **Amazon Linux 2023**:

| Server          | Network | Purpose                 |
| --------------- | ------- | ----------------------- |
| Jump Server     | Public  | Secure SSH entry point  |
| App Server      | Public  | QuickLoan application   |
| Database Server | Private | Database infrastructure |

Administrative access follows the SSH jump/bastion-host pattern.

---

# 🌐 Application Deployment

The QuickLoan application was deployed using:

* Nginx
* PHP 8.2
* PHP-FPM
* PHP MySQL driver

Application files were uploaded to EC2 using **WinSCP** and deployed to:

```text
/usr/share/nginx/html
```

Appropriate ownership and permissions were configured for the Nginx web root.

---

# 🪣 Amazon S3

Amazon S3 is used to store static image assets.

The application source files were updated to reference the S3 bucket for static assets.

```text
QuickLoan Application
        │
        └──────► Amazon S3
                    │
                    └── Static Images
```

The documented S3 bucket was created in the `us-east-2` region.

---

# 🗄️ Amazon RDS MySQL

Amazon RDS is used as the managed relational database service.

The RDS MySQL database was deployed in a **private subnet**.

The project database contains:

```text
quickloan_db
└── applications
```

The application server was configured with the RDS database connection, and database connectivity was verified through the Jump Server.

---

# 🌍 Domain Configuration

The application was configured with a free dynamic DNS hostname using **No-IP**.

Configured hostname:

```text
quickloan.hopto.org
```

Nginx was configured with the corresponding virtual host/domain configuration.

---

# 💿 Custom AMI

After configuring and testing the application server, a custom Amazon Machine Image was created:

```text
app-server-image
```

This AMI serves as the **golden image** used by the Auto Scaling Launch Template.

```text
Configured EC2 App Server
          │
          ▼
    Custom AMI
          │
          ▼
 Launch Template
          │
          ▼
 Auto Scaling Group
```

---

# ⚖️ Application Load Balancer

An **Internet-facing Application Load Balancer (ALB)** was configured.

The ALB forwards HTTP traffic on:

```text
Port 80
```

to the configured Target Group.

The ALB spans multiple Availability Zone subnets to distribute traffic across application instances.

---

# 📈 Auto Scaling

An **Auto Scaling Group (ASG)** was configured using the custom AMI through a Launch Template.

Configuration documented in the project:

```text
Minimum Instances: 2
Maximum Instances: 5
```

The Auto Scaling Group is attached to the ALB Target Group.

```text
                 ALB
                  │
          ┌───────┴───────┐
          ▼               ▼
       EC2 App 1       EC2 App 2
          │               │
          └───────┬───────┘
                  │
            Auto Scaling
             Min: 2
             Max: 5
```

---

# 🧪 End-to-End Verification

The deployed application was tested end-to-end.

### Application Test

The QuickLoan homepage was successfully loaded through the configured domain.

### Loan Application Test

A test loan application was submitted through the web form.

### Database Verification

The submitted application data was verified directly in the RDS MySQL database.

This confirmed the application-to-database communication path was working correctly.

---

# 🔄 Application Request Flow

```text
                    User
                      │
                      ▼
              No-IP Domain
                      │
                      ▼
            Application Load
                Balancer
                      │
              ┌───────┴───────┐
              ▼               ▼
           EC2 App 1       EC2 App 2
              │               │
          Nginx + PHP     Nginx + PHP
              │               │
              └───────┬───────┘
                      │
                      ▼
                RDS MySQL
                      │
                      ▼
              Loan Application
                  Database


Static Images
      │
      ▼
   Amazon S3
```

---

# 🔑 Key AWS Services Used

```text
Amazon VPC
   │
   ├── Subnets
   ├── Route Tables
   ├── Internet Gateway
   └── NAT Gateway

Amazon EC2
   │
   ├── Jump Server
   ├── Application Server
   └── Database Infrastructure

Amazon RDS
   └── MySQL

Amazon S3
   └── Static Assets

Application Load Balancer
   └── Target Group

Auto Scaling
   └── Launch Template + AMI
```

---

# 👨‍💻 Project Responsibilities

The project documentation records completion of the following tasks:

* Designed and configured VPC networking
* Created subnets and route tables
* Configured Internet Gateway and NAT Gateway
* Configured Security Groups
* Deployed PHP application on EC2
* Configured Nginx and PHP-FPM
* Created custom AMI
* Configured S3 for static assets
* Provisioned RDS MySQL
* Created database schema
* Verified database connectivity
* Configured Launch Template
* Configured Auto Scaling Group
* Configured Application Load Balancer
* Performed end-to-end application testing
* Verified submitted application data in the database

---

# 📂 Suggested Repository Structure

```text
QuickLoan/
│
├── README.md
│
├── frontend/
│   ├── index.html
│   ├── css/
│   ├── js/
│   └── images/
│
├── backend/
│   ├── *.php
│   └── config/
│
├── nginx/
│   └── quickloan.conf
│
├── database/
│   └── schema.sql
│
└── docs/
    └── AWS-Architecture.pdf
```
---

# 🚀 Deployment Summary

The complete deployment process can be summarized as:

```text
1. Create VPC
       ↓
2. Create Public & Private Subnets
       ↓
3. Configure IGW + NAT Gateway
       ↓
4. Configure Route Tables
       ↓
5. Configure Security Groups
       ↓
6. Launch EC2 Instances
       ↓
7. Install Nginx + PHP
       ↓
8. Deploy QuickLoan Application
       ↓
9. Create S3 Bucket
       ↓
10. Create RDS MySQL
       ↓
11. Configure Database Connection
       ↓
12. Configure No-IP Domain
       ↓
13. Create Custom AMI
       ↓
14. Create Target Group
       ↓
15. Create Application Load Balancer
       ↓
16. Create Launch Template
       ↓
17. Create Auto Scaling Group
       ↓
18. Perform End-to-End Testing
```

---

# 📊 Project Highlights

| Feature               | Implementation            |
| --------------------- | ------------------------- |
| Cloud Platform        | Amazon Web Services       |
| Application           | QuickLoan                 |
| Frontend              | HTML / CSS / JavaScript   |
| Backend               | PHP 8.2                   |
| Web Server            | Nginx                     |
| Database              | Amazon RDS MySQL          |
| Object Storage        | Amazon S3                 |
| Compute               | Amazon EC2                |
| Network               | Amazon VPC                |
| Load Balancer         | Application Load Balancer |
| Scaling               | Auto Scaling Group        |
| Server Image          | Custom AMI                |
| DNS                   | No-IP                     |
| OS                    | Amazon Linux 2023         |
| Minimum EC2 Instances | 2                         |
| Maximum EC2 Instances | 5                         |
| Database Location     | Private Subnet            |
| Static Assets         | Amazon S3                 |

---

# 👤 Author

**Milan Meena**

### Project

**QuickLoan – Scalable Web Application Deployment on AWS**

---

## 📄 Project Documentation

The complete technical project documentation contains the detailed AWS deployment steps, architecture, configuration, and database verification.

**Technology:** PHP + Nginx + MySQL + AWS

**Cloud Platform:** Amazon Web Services

**Region documented:** Virginia

---

## ⭐ AWS Architecture Concepts Demonstrated

This project demonstrates practical implementation of:

* AWS VPC networking
* Public and private subnet architecture
* Internet Gateway
* NAT Gateway
* Route Tables
* Security Groups
* EC2
* Nginx
* PHP-FPM
* Amazon S3
* Amazon RDS MySQL
* Bastion / Jump Server
* Custom AMI
* Launch Templates
* Application Load Balancer
* Target Groups
* Auto Scaling
* DNS configuration
* End-to-end application and database testing

---

**QuickLoan — Scalable Web Application Deployment on Amazon Web Services**
