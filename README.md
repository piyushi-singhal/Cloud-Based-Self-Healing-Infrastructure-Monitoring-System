# Cloud-Based Self-Healing Infrastructure Monitoring System

<p align="center">

  <strong>Automated Monitoring • Fault Detection • Recovery • Observability</strong>

  <br><br>

  <img src="https://img.shields.io/badge/AWS-Cloud-orange?logo=amazonaws" alt="AWS">
  <img src="https://img.shields.io/badge/Amazon%20EC2-Compute-orange?logo=amazonec2" alt="EC2">
  <img src="https://img.shields.io/badge/CloudWatch-Monitoring-blue?logo=amazoncloudwatch" alt="CloudWatch">
  <img src="https://img.shields.io/badge/AWS%20Lambda-Serverless-orange?logo=awslambda" alt="Lambda">
  <img src="https://img.shields.io/badge/Amazon%20SNS-Notifications-orange?logo=amazonsns" alt="SNS">
  <img src="https://img.shields.io/badge/DynamoDB-NoSQL-blue?logo=amazondynamodb" alt="DynamoDB">

</p>

---

## 📌 Overview

The **Cloud-Based Self-Healing Infrastructure Monitoring System** is an AWS-based cloud infrastructure project designed to improve application **availability, scalability, reliability, and operational resilience**.

The system continuously monitors infrastructure health using **Amazon CloudWatch**, detects abnormal conditions through alarms, sends notifications through **Amazon SNS**, records monitoring and recovery events using **AWS Lambda and Amazon DynamoDB**, and uses **Amazon EC2 Auto Scaling** to automatically scale or replace infrastructure when required.

The project was developed in two major stages:

- **Sprint 1:** Core cloud infrastructure and monitoring
- **Sprint 2:** Event-driven automation, recovery logging, notifications, and observability

### Core Workflow

```text
Monitor → Detect → Alert → Log → Recover → Observe

🏗️ System Architecture

The following architecture represents the complete system and its major AWS components.
flowchart TB

    USER["👤 Users"]

    ALB["Application Load Balancer"]

    ASG["Auto Scaling Group"]

    LT["Launch Template"]

    EC2A["EC2 Instance 1"]
    EC2B["EC2 Instance 2"]
    EC2C["EC2 Instance N"]

    CW["Amazon CloudWatch"]

    ALARM["CloudWatch Alarm"]

    SNS["Amazon SNS"]

    LAMBDA["AWS Lambda"]

    DB[("Amazon DynamoDB<br/>RecoveryLogs")]

    DASH["CloudWatch Dashboard"]

    USER -->|"HTTP Requests"| ALB

    ALB --> EC2A
    ALB --> EC2B
    ALB --> EC2C

    LT -->|"Instance Configuration"| ASG

    ASG -->|"Manages Capacity"| EC2A
    ASG -->|"Manages Capacity"| EC2B
    ASG -->|"Manages Capacity"| EC2C

    EC2A -.->|"CPU / Health Metrics"| CW
    EC2B -.->|"CPU / Health Metrics"| CW
    EC2C -.->|"CPU / Health Metrics"| CW

    CW --> ALARM

    ALARM -->|"Notification"| SNS
    ALARM -->|"Event Trigger"| LAMBDA

    LAMBDA -->|"Store Recovery Event"| DB

    CW --> DASH

    ALARM -.->|"Scaling / Recovery Conditions"| ASG

    ASG -->|"Scale / Replace Instances"| EC2C
