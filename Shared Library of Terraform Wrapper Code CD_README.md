# Terraform Wrapper Shared Library CD Pipeline POC

---

## Document Details

| Author | Created on | Version | Last updated by | Last edited on | Pre Reviewer | L0 Reviewer | L1 Reviewer | L2 Reviewer |
|--------|------------|---------|-----------------|----------------|--------------|-------------|-------------|-------------|
| Suraj Tripathi | 25-05-2026 | v1.0 | Suraj Tripathi | 29-05-2026 |              | Aniruddh    | Faisal      | Ashwani     |

---
# Table of Contents
* [1. Overview](#1-overview)
* [2. Objective](#2-objective)
* [3. Technologies Used](#3-technologies-used)
* [4. Architecture](#4-architecture)
* [5. Project Structure](#5-project-structure)
  * [Terraform Modules Repository](#terraform-modules-repository)
  * [Terraform Wrapper Repository](#terraform-wrapper-repository)
  * [Jenkins Shared Library Repository](#jenkins-shared-library-repository)
* [6. Terraform Module](#6-terraform-module)
  * [EC2 Module Example](#ec2-module-example)
    * [main.tf](#maintf)
    * [variables.tf](#variablestf)
    * [outputs.tf](#outputstf)
* [7. Wrapper Code](#7-wrapper-code)
  * [Dev Environment Example](#dev-environment-example)
  * [QA Environment Example](#qa-environment-example)
* [8. Jenkins Shared Library](#8-jenkins-shared-library)
  * [terraformInit.groovy](#terraforminitgroovy)
  * [terraformValidate.groovy](#terraformvalidategroovy)
  * [terraformPlan.groovy](#terraformplangroovy)
  * [terraformApply.groovy](#terraformapplygroovy)
* [9. Configure Shared Library in Jenkins](#9-configure-shared-library-in-jenkins)
* [10. Jenkins Pipeline](#10-jenkins-pipeline)
  * [Jenkinsfile](#jenkinsfile)
* [11. .gitignore](#11-gitignore)
* [12. Workflow](#12-workflow)
* [13. POC Output](#13-poc-output)
  * [Dev Environment](#dev-environment)
  * [QA Environment](#qa-environment)
* [14. Benefits of This Architecture](#14-benefits-of-this-architecture)
* [15. Practical Steps](#15-practical-steps)
  * [Step 1](#step-1)
  * [Step 2](#step-2)
  * [Step 3](#step-3)
  * [Step 4](#step-4)
  * [Step 5](#step-5)
  * [Step 6](#step-6)
  * [Step 7](#step-7)
  * [Step 8](#step-8)
  * [Step 9](#step-9)
* [16. Contact Information](#16-contact-information)
* [17. Conclusion](#17-conclusion)

---

# 1. Overview

This project demonstrates how to deploy Dev and QA infrastructure using:

* Terraform Modules
* Terraform Wrapper Code
* Jenkins Shared Library
* Jenkins CD Pipeline

This is a beginner-friendly Proof of Concept (POC) designed to understand reusable infrastructure deployment using Terraform and Jenkins.

---

# 2. Objective

The main objective of this POC is to:

* Create reusable Terraform modules
* Create separate Dev and QA environments
* Use wrapper code for environment-specific configuration
* Use Jenkins Shared Library for reusable pipeline stages
* Automate infrastructure deployment using Jenkins

---

# 3. Technologies Used

| Tool           | Purpose                         |
| -------------- | ------------------------------- |
| Terraform      | Infrastructure as Code          |
| AWS            | Cloud Provider                  |
| Jenkins        | CI/CD Automation                |
| GitHub         | Source Code Repository          |
| Shared Library | Reusable Jenkins Pipeline Logic |

---

# 4. Architecture

```text
Developer
    ↓
GitHub Repository
    ↓
Jenkins Pipeline
    ↓
Shared Library
    ↓
Terraform Wrapper Code
    ↓
Terraform Modules
    ↓
AWS Infrastructure
```

---

# 5. Project Structure

## Terraform Modules Repository

```text
terraform-modules/
│
├── ec2/
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
```

---

## Terraform Wrapper Repository

```text
terraform-wrapper/
│
├── env/
│   ├── dev/
│   │   └── ec2/
│   │       ├── main.tf
│   │       └── terraform.tfvars
│   │
│   └── qa/
│       └── ec2/
│           ├── main.tf
│           └── terraform.tfvars
│
├── Jenkinsfile
├── .gitignore
└── README.md
```

---

## Jenkins Shared Library Repository

```text
terraform-shared-library/
│
├── vars/
│   ├── terraformInit.groovy
│   ├── terraformValidate.groovy
│   ├── terraformPlan.groovy
│   └── terraformApply.groovy
```

---

# 6. Terraform Module

## EC2 Module Example

### main.tf

```hcl
resource "aws_instance" "this" {

  ami           = var.ami_id
  instance_type = var.instance_type

  tags = {
    Name = var.instance_name
  }
}
```

---

### variables.tf

```hcl
variable "ami_id" {}

variable "instance_type" {}

variable "instance_name" {}
```

---

### outputs.tf

```hcl
output "instance_id" {
  value = aws_instance.this.id
}
```

---

# 7. Wrapper Code

Wrapper code calls reusable Terraform modules.

---

## Dev Environment Example

### env/dev/ec2/main.tf

```hcl
provider "aws" {
  region = "us-east-1"
}

module "ec2" {

  source = "../../../modules/ec2"

  ami_id         = "ami-xxxxxxxx"
  instance_type  = "t2.micro"
  instance_name  = "dev-server"
}
```

---

## QA Environment Example

### env/qa/ec2/main.tf

```hcl
provider "aws" {
  region = "us-east-1"
}

module "ec2" {

  source = "../../../modules/ec2"

  ami_id         = "ami-xxxxxxxx"
  instance_type  = "t2.small"
  instance_name  = "qa-server"
}
```

---

# 8. Jenkins Shared Library

## terraformInit.groovy

```groovy
def call(String path) {

    dir(path) {
        sh 'terraform init'
    }
}
```

---

## terraformValidate.groovy

```groovy
def call(String path) {

    dir(path) {
        sh 'terraform validate'
    }
}
```

---

## terraformPlan.groovy

```groovy
def call(String path) {

    dir(path) {
        sh 'terraform plan -out=tfplan'
    }
}
```

---

## terraformApply.groovy

```groovy
def call(String path) {

    dir(path) {
        sh 'terraform apply -auto-approve tfplan'
    }
}
```

---

# 9. Configure Shared Library in Jenkins

Navigate to:

```text
Manage Jenkins
    ↓
System
    ↓
Global Pipeline Libraries
```

Add the following:

| Field          | Value                                |
| -------------- | ------------------------------------ |
| Name           | terraform-shared-library             |
| Default Branch | main                                 |
| SCM            | Git                                  |
| Repository URL | Shared Library GitHub Repository URL |

---

# 10. Jenkins Pipeline

## Jenkinsfile

```groovy
@Library('terraform-shared-library') _

pipeline {

    agent any

    parameters {

        choice(
            name: 'ENV',
            choices: ['dev', 'qa'],
            description: 'Select Environment'
        )

        choice(
            name: 'ACTION',
            choices: ['plan', 'apply'],
            description: 'Terraform Action'
        )
    }

    environment {

        TF_PATH = "env/${params.ENV}/ec2"
    }

    stages {

        stage('Checkout') {

            steps {
                checkout scm
            }
        }

        stage('Terraform Init') {

            steps {
                terraformInit(env.TF_PATH)
            }
        }

        stage('Terraform Validate') {

            steps {
                terraformValidate(env.TF_PATH)
            }
        }

        stage('Terraform Plan') {

            steps {
                terraformPlan(env.TF_PATH)
            }
        }

        stage('Terraform Apply') {

            when {
                expression {
                    params.ACTION == 'apply'
                }
            }

            steps {

                input 'Approve Terraform Apply?'

                terraformApply(env.TF_PATH)
            }
        }
    }
}
```

---

# 11. .gitignore

```gitignore
.terraform/
terraform.tfstate
terraform.tfstate.backup
```

---

# 12. Workflow

```text
Developer Push Code
        ↓
Jenkins Trigger
        ↓
Load Shared Library
        ↓
Terraform Init
        ↓
Terraform Validate
        ↓
Terraform Plan
        ↓
Approval
        ↓
Terraform Apply
        ↓
AWS Infrastructure Created
```

---

# 13. POC Output

This POC creates:

## Dev Environment

* 1 EC2 Instance
* Dev Configuration

---

## QA Environment

* 1 EC2 Instance
* QA Configuration

---

# 14. Benefits of This Architecture

* Reusable Terraform modules
* Reusable Jenkins pipeline code
* Separate environments
* Standardized deployments
* Easy maintenance
* Scalable infrastructure automation

---

# 15. Practical Steps

## Step 1

Install:

* Terraform
* Jenkins
* AWS CLI
* Git

---

## Step 2

Create GitHub repositories:

```text
terraform-modules
terraform-wrapper
terraform-shared-library
```

---

## Step 3

Create Terraform EC2 module.

---

## Step 4

Create Dev and QA wrapper code.

---

## Step 5

Run Terraform manually:

```bash
terraform init
terraform plan
terraform apply
```

---

## Step 6

Create Jenkins Shared Library.

---

## Step 7

Configure Shared Library in Jenkins.

---

## Step 8

Create Jenkins Pipeline Job.

---

## Step 9

Run pipeline using:

```text
ENV = dev
ACTION = apply
```

or

```text
ENV = qa
ACTION = apply
```

---
## 16. Contact Information

| Contact Type | Details                                                             |
| ------------ | ------------------------------------------------------------------- |
| Name         | Suraj Tripathi                                                      |
| Role         | DevOps Trainee                                                      |
| Email        | [suraj.tripathi.snaatak@mygurukulam.co](mailto:suraj.tripathi.snaatak@mygurukulam.co) |
---

# 17. Conclusion

This beginner-friendly POC demonstrates how to implement reusable infrastructure deployment using:

* Terraform Modules
* Wrapper Code
* Jenkins Shared Library
* Jenkins CD Pipeline

This architecture helps standardize deployments and reduce duplicate code in DevOps projects.

This approach is commonly used in real DevOps projects to standardize infrastructure deployments across multiple environments.
