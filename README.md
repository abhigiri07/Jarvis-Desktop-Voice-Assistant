# 🚀 Terraform + Jenkins Deployment for Jarvis Voice Assistant
![](/imgs/ReadmeHeader.gif)

## 📘 Project Description

This project showcases a fully automated deployment pipeline for the Jarvis Desktop Voice Assistant, powered by a modern DevOps stack. Using Terraform for Infrastructure-as-Code, an AWS EC2 instance for hosting, Jenkins for CI/CD automation, and GitHub Webhooks for continuous integration, the entire workflow—from infrastructure creation to application deployment—runs end-to-end without manual intervention.

The system automatically:

💠 Creates and configures AWS resources using Terraform

💠 Installs and bootstraps Jenkins on the EC2 instance

💠 Pulls the latest code from GitHub via webhook triggers

💠 Securely transfers files using Jenkins SSH credentials

💠 Deploys and updates the Jarvis Voice Assistant in real time


------------------------------------------------------------------------

## ⭐ Overview

This project automates complete deployment using:

1.  **Terraform** → Create EC2 + Security Group + Setup\
2.  **EC2 Setup** → Install Jenkins automatically\
3.  **GitHub + Jenkins** → Webhook-based CI/CD\
4.  **SSH Credentials** → Secure deployment\
5.  **Jenkins Pipeline** → Auto-deploy Jarvis on push

------------------------------------------------------------------------

## 🏗️ 1. Terraform Setup

### 📁 File Structure

-   `provider.tf` → AWS region\
-   `variables.tf` → Variables (ami, instance type, key, CIDR)\
-   `main.tf` → EC2 + SG + KeyPair\
-   `outputs.tf` → Output EC2 Public IP\
-   `user_data.sh` → Bootstrap installation

------------------------------------------------------------------------

### 🧩 provider.tf

``` hcl
provider "aws" {
  region = var.aws_region
}
```

### 🧩 variables.tf

``` hcl
variable "aws_region" { default = "ap-south-1" }
variable "ami" {}
variable "instance_type" { default = "t2.micro" }
variable "key_name" {}
variable "allowed_cidr" { default = "0.0.0.0/0" }
```

### 🧩 main.tf

``` hcl
resource "aws_key_pair" "jarvis" {
  key_name   = var.key_name
  public_key = file("~/.ssh/id_rsa.pub")
}

resource "aws_security_group" "jenkins_sg" {
  name = "jenkins_sg"

  ingress {
    from_port = 22
    to_port   = 22
    protocol  = "tcp"
    cidr_blocks = [var.allowed_cidr]
  }

  ingress {
    from_port = 8080
    to_port   = 8080
    protocol  = "tcp"
    cidr_blocks = [var.allowed_cidr]
  }

  egress {
    from_port = 0
    to_port   = 0
    protocol  = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "jarvis" {
  ami           = var.ami
  instance_type = var.instance_type
  key_name      = aws_key_pair.jarvis.key_name
  vpc_security_group_ids = [aws_security_group.jenkins_sg.id]
  user_data = file("user_data.sh")

  tags = {
    Name = "jarvis-deploy"
  }
}

output "public_ip" {
  value = aws_instance.jarvis.public_ip
}
```

------------------------------------------------------------------------

### 🛠️ user_data.sh

``` bash
#!/bin/bash
apt update -y
apt upgrade -y
apt install -y git python3 python3-venv python3-pip rsync curl openjdk-11-jdk
mkdir -p /home/ubuntu/jarvis
chown -R ubuntu:ubuntu /home/ubuntu/jarvis
```

------------------------------------------------------------------------

## ▶️ Deploy Terraform

``` bash
terraform init
terraform plan -var 'ami=ami-xxxxx' -var 'key_name=mykey'
terraform apply
```

------------------------------------------------------------------------

## 🔑 2. Jenkins Installation on EC2

SSH into instance:

``` bash
ssh -i key.pem ubuntu@PUBLIC_IP
```

Install Jenkins:

``` bash
sudo apt update
sudo apt install -y openjdk-11-jdk
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install -y jenkins
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

Access Jenkins:

    http://PUBLIC_IP:8080

Initial Password:

``` bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
------------------------------------------------------------------------

## 🔄 3. Jenkinsfile for Deployment

``` groovy
pipeline {
  agent any
  environment {
    REMOTE_USER = "ubuntu"
    REMOTE_HOST = "3.110.121.35"
    REMOTE_DIR  = "/home/ubuntu/jarvis"
    CRED_ID     = "jarvis-key"
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/<youruser>/Jarvis-Desktop-Voice-Assistant.git'
      }
    }

    stage('Package & Transfer') {
      steps {
        sshagent(credentials: ["${CRED_ID}"]) {
          sh '''
            ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} "mkdir -p ${REMOTE_DIR}"
            rsync -avz --delete --exclude='.git' ./ ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/
          '''
        }
      }
    }

    stage('Remote: Setup & Restart') {
      steps {
        sshagent(credentials: ["${CRED_ID}"]) {
          sh "ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} 'cd ${REMOTE_DIR} && ./setup_and_restart.sh'"
        }
      }
    }
  }
}
```

------------------------------------------------------------------------

## 🔔 4. GitHub Webhook Setup

Go to:\
**GitHub → Repo → Settings → Webhooks → Add Webhook**

Payload URL:

    http://JENKINS_IP:8080/github-webhook/

Content Type → `application/json`\
Trigger → **Just Push**

------------------------------------------------------------------------

## 🔐 5. Add Jenkins SSH Credentials

Jenkins → Credentials → Global → Add Credentials

-   Type → SSH Username with Private Key\
-   Username → ubuntu\
-   Private Key → Paste PEM\
-   ID → `ubuntu`
------------------------------------------------------------------------

## 🚀 6. Deployment

Create Job → Pipeline from SCM → Select Repo → Add Jenkinsfile Path

Every push = automatic deployment.
![](/imgs/Screenshot%20(144).png)
------------------------------------------------------------------------

## 🏁 Conclusion

This project demonstrates a fully automated, production-grade CI/CD workflow that combines the power of Terraform, AWS EC2, Jenkins, and GitHub Webhooks to deploy the Jarvis Desktop Voice Assistant seamlessly.
By integrating Infrastructure-as-Code with continuous deployment, this setup ensures:

🔹 Zero manual server setup — Terraform handles all infrastructure

🔹 Automatic provisioning — EC2 bootstraps itself using user_data

🔹 Secure & repeatable deployments — Jenkins manages and executes pipelines

🔹 Instant updates — Every Git push triggers a fresh deployment on the EC2 server

🔹 Scalability & maintainability — Code-driven infrastructure and pipelines

Overall, the project delivers a reliable, modern, and scalable deployment pipeline suitable for real-world DevOps environments. It not only simplifies server management but also brings the Jarvis application to life with automation, speed, and efficiency—making it a complete end-to-end DevOps solution.
