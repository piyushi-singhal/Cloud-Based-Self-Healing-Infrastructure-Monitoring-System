Overview
The Cloud-Based Self-Healing Infrastructure Monitoring System is a cloud-native infrastructure project designed to improve the availability, reliability, scalability, and resilience of applications hosted on AWS.

Instead of relying entirely on administrators to detect infrastructure problems and manually recover failed instances, the system uses AWS monitoring and automation services to create an event-driven recovery workflow.

The infrastructure:

distributes application traffic across EC2 instances
monitors system performance using CloudWatch
detects abnormal CPU utilization and infrastructure health conditions
triggers CloudWatch alarms when configured thresholds are exceeded
sends administrator notifications through SNS
invokes Lambda to record alarm events
stores recovery events in DynamoDB
automatically scales infrastructure under increased load
replaces unhealthy or terminated EC2 instances through Auto Scaling
provides a CloudWatch dashboard for system visibility

The project was implemented in two development sprints, evolving from basic cloud infrastructure into an event-driven monitoring, logging, and recovery system.

Problem Statement

Traditional infrastructure monitoring often follows a reactive workflow:

Failure occurs
     ↓
Monitoring detects issue
     ↓
Administrator receives alert
     ↓
Administrator investigates
     ↓
Administrator performs recovery

This introduces manual intervention and can increase service downtime.

This project aims to automate as much of this workflow as possible:

Infrastructure anomaly
        ↓
CloudWatch detects condition
        ↓
CloudWatch Alarm
        ↓
 ┌──────┴────────┐
 ↓               ↓
SNS             Lambda
 ↓               ↓
Alert          Event Log
                 ↓
             DynamoDB

        ↓
Auto Scaling
        ↓
Scale / Replace Instance
        ↓
Healthy infrastructure

The project therefore focuses on automated monitoring, event-driven response, infrastructure scaling, instance replacement, and recovery logging.

Architecture
High-Level Architecture

The documented final architecture consists of an Application Load Balancer, EC2 instances managed by an Auto Scaling Group, CloudWatch monitoring, CloudWatch alarms, SNS notifications, Lambda event logging, DynamoDB recovery logs, and a CloudWatch dashboard.

Architecture Components
Component	Responsibility
Amazon EC2	Runs the application/web server instances
Launch Template	Defines configuration used to launch EC2 instances
Application Load Balancer	Distributes incoming HTTP traffic
Target Group	Maintains registered EC2 targets and health checks
Auto Scaling Group	Maintains desired capacity, scales under load, and replaces unhealthy instances
Amazon CloudWatch	Collects infrastructure metrics and monitors system behavior
CloudWatch Alarm	Detects configured threshold breaches
Amazon SNS	Sends administrator notifications
AWS Lambda	Processes alarm events and records event information
Amazon DynamoDB	Stores recovery/event logs
CloudWatch Dashboard	Visualizes system health and scaling behavior
IAM	Controls permissions for AWS resources
Failure Detection & Recovery Workflow

The central workflow of the project is event-driven.

1. Application Traffic

Users access the application through the Application Load Balancer.

User
 ↓
Application Load Balancer
 ↓
Healthy EC2 Instances

The ALB distributes requests across available healthy instances.

2. Infrastructure Monitoring

EC2 infrastructure metrics are monitored by Amazon CloudWatch.

The project primarily monitors:

CPU utilization
request count
healthy host count
instance count

The documentation specifies 1-minute monitoring frequency for CPU metrics.

3. Alarm Detection

When a configured monitoring condition is exceeded:

CloudWatch Metric
       ↓
Threshold Breached
       ↓
CloudWatch Alarm
       ↓
ALARM State

The project tested CPU-load generation to deliberately trigger the CloudWatch alarm.

4. Notification

The CloudWatch alarm triggers Amazon SNS.

CloudWatch Alarm
       ↓
      SNS
       ↓
Administrator Notification

SNS is responsible for alerting, not recovery.

5. Event Logging

The same alarm event invokes AWS Lambda.

CloudWatch Alarm
       ↓
AWS Lambda
       ↓
DynamoDB
       ↓
RecoveryLogs

Lambda records information such as:

id
timestamp
message

The RecoveryLogs DynamoDB table provides an audit trail for monitoring and analysis.

6. Automatic Infrastructure Recovery

Auto Scaling performs the actual infrastructure recovery.

Depending on the infrastructure condition, the Auto Scaling Group can:

launch additional instances under increased load
maintain desired capacity
replace terminated instances
replace instances that fail health checks
Unhealthy / Terminated Instance
              ↓
       Auto Scaling Group
              ↓
       New EC2 Instance
              ↓
       Health Check
              ↓
       Healthy Target
              ↓
       ALB Receives Traffic

This distinction is important: Lambda logs the event; Auto Scaling performs instance scaling/replacement.

Self-Healing Sequence
Sprint Evolution

The architecture evolved over two sprints.

Sprint 1 — Core Infrastructure

The first sprint established the foundational cloud infrastructure:

EC2
 ↓
Launch Template
 ↓
Auto Scaling Group
 ↓
Application Load Balancer
 ↓
CloudWatch Monitoring

Major objectives:

EC2 deployment
IAM configuration
security groups
Apache web server deployment
Launch Template
Target Groups
Application Load Balancer
health checks
Auto Scaling
CloudWatch monitoring

The Sprint 1 testing covered EC2 deployment, IAM permissions, load balancing, target health, traffic distribution, CloudWatch alarms, CPU stress testing, and metric visibility.

Sprint 2 — Event-Driven Automation

The second sprint extended the infrastructure with:

CloudWatch
     ↓
CloudWatch Alarm
   ↙       ↘
SNS       Lambda
            ↓
        DynamoDB

Additional functionality:

SNS notifications
Lambda event logging
DynamoDB recovery logs
Auto Scaling recovery
CloudWatch dashboard
integrated failure testing

The architecture document describes Sprint 2 as the transition from basic infrastructure to an event-driven monitoring, logging, and visualization architecture.

Monitoring Dashboard

The CloudWatch dashboard provides visibility into system behavior.

Key Metrics
┌─────────────────────────────────────┐
│        CloudWatch Dashboard         │
├─────────────────────────────────────┤
│ CPU Utilization                     │
│ Request Count                       │
│ Healthy Host Count                  │
│ Instance Count                      │
└─────────────────────────────────────┘

The dashboard is intended to visualize:

workload patterns
infrastructure health
traffic
instance availability
scaling behavior

These metrics are explicitly documented as the final dashboard metrics.

Recovery Logging

Amazon DynamoDB stores system recovery events in:

RecoveryLogs
Schema
Attribute	Description
id	Unique event identifier
timestamp	Time at which the event occurred
message	Description of the event
Example
{
  "id": "event-001",
  "timestamp": "2026-04-10T12:30:00Z",
  "message": "CloudWatch alarm triggered due to high CPU utilization"
}

The database serves as an event history and audit trail for system behavior.

Testing

The system was tested under both normal and failure conditions.

Infrastructure Tests
EC2 instance launch
EC2 web-server access
User Data execution
IAM role creation
IAM policy validation
Target Group configuration
ALB health checks
ALB traffic distribution
Monitoring Tests
CloudWatch alarm creation
CPU stress testing
alarm triggering
alarm recovery
metric visibility
Automation Tests
Lambda creation
Lambda invocation through CloudWatch
manual Lambda testing
alarm-based Lambda execution
Auto Scaling creation
CPU-based scaling
instance replacement
health-check-based replacement
Logging Tests
DynamoDB table creation
event insertion
log visibility
stored attribute validation

The documented Sprint 2 functional tests report successful execution for Lambda, Auto Scaling, and DynamoDB workflows.

Example Failure Scenario

Consider an EC2 instance becoming unhealthy.

Before failure
                 ALB
               /     \
             EC2-1   EC2-2
              ✓       ✓

Both instances are healthy.

Instance failure
                 ALB
               /     \
             EC2-1   EC2-2
              ✓       ✗

The failed instance is detected through health checks / infrastructure monitoring.

Recovery
                 ALB
               /     \
             EC2-1   EC2-3
              ✓       ✓

Auto Scaling launches a replacement instance.

At the same time:

CloudWatch Alarm
      ├──→ SNS → Administrator
      │
      └──→ Lambda → DynamoDB
                       ↓
                  RecoveryLogs

This provides both automated infrastructure recovery and observability of the recovery event.

Security

The project incorporates AWS IAM and security groups as part of infrastructure access control.

The documented security approach includes:

IAM roles for AWS resources
controlled permissions
security groups
HTTP access for the application
SSH access for administration
least-privilege access as a design value

The Sprint 1 requirements specifically include IAM role configuration, security groups, and access validation.

Technology Stack
Cloud Platform

Amazon Web Services (AWS)

Compute & Networking
Amazon EC2
Application Load Balancer
Target Groups
Auto Scaling Groups
Launch Templates
Monitoring & Automation
Amazon CloudWatch
CloudWatch Alarms
CloudWatch Dashboard
AWS Lambda
Amazon SNS
Storage
Amazon DynamoDB
Security
AWS IAM
EC2 Security Groups
Application Environment
Apache HTTP Server
Linux-based EC2 environment
Project Structure

A suggested repository structure for the project:

cloud-based-self-healing-infrastructure/
│
├── README.md
│
├── lambda/
│   └── recovery_logger.py
│
├── infrastructure/
│   ├── launch-template/
│   ├── auto-scaling/
│   ├── load-balancer/
│   ├── cloudwatch/
│   └── iam/
│
├── scripts/
│   └── ec2-user-data.sh
│
├── docs/
│   ├── architecture/
│   ├── diagrams/
│   └── screenshots/
│
└── tests/
    └── test-cases.md

Note: This structure is a recommended GitHub organization. The project documentation confirms the Lambda function, EC2 User Data script, Auto Scaling configuration, and CloudWatch alarm configuration as implementation artifacts, but does not establish that the original project repository used exactly this folder structure.

Key Outcomes

The project successfully integrated:

Monitoring
    +
Alerting
    +
Event Logging
    +
Auto Scaling
    +
Instance Recovery
    +
Visualization

The final system demonstrated:

automated infrastructure monitoring
load balancing
dynamic scaling
automatic instance replacement
alarm-driven notifications
event logging
recovery visibility
reduced manual intervention

The project documentation reports that all 8 planned user stories were completed — 4 in Sprint 1 and 4 in Sprint 2 — resulting in 100% completion.

What Makes It "Self-Healing"?

The important characteristic is that the system doesn't stop at detecting a failure.

A conventional monitoring system might do:

Detect → Alert → Human Responds

This project extends that to:

Detect
  ↓
Alarm
  ↓
Automated Infrastructure Response
  ↓
Instance Scaling / Replacement
  ↓
Healthy Infrastructure

Meanwhile, the event is recorded:

Alarm
  ↓
Lambda
  ↓
DynamoDB

and administrators are notified:

Alarm
  ↓
SNS
  ↓
Notification

Thus, monitoring, notification, logging, and automated infrastructure recovery work together as a closed-loop system.

Limitations

The current implementation is a baseline self-healing infrastructure system, rather than a fully autonomous SRE platform.

The documented implementation primarily relies on:

threshold-based CloudWatch alarms
predefined Auto Scaling behavior
EC2-based infrastructure
AWS-specific services
event logging rather than predictive failure analysis

It does not currently implement ML-based failure prediction or multi-cloud recovery.

This is consistent with the project's own future-enhancement section.

Future Enhancements

Potential extensions identified in the project documentation include:

Predictive Failure Detection

Introduce machine learning models to predict infrastructure failures before they occur.

Historical Metrics
       ↓
ML Model
       ↓
Failure Prediction
       ↓
Preventive Action
Multi-Cloud Support

Extend the architecture beyond AWS to support:

AWS
Azure
Google Cloud
Security Automation

Add:

automated threat detection
security-event analysis
automated mitigation
Advanced Observability

Enhance the dashboard with:

advanced analytics
custom reporting
richer real-time insights
Cost Optimization

Introduce automated resource optimization to balance:

Performance
     ↕
Availability
     ↕
Infrastructure Cost

These enhancements are directly aligned with the future-work section of the project documentation.

Project Development

The project followed an iterative sprint-based development approach.

Sprint 1
   │
   ├── EC2
   ├── IAM
   ├── ALB
   ├── Auto Scaling
   └── CloudWatch
          │
          ▼
Sprint 2
   │
   ├── SNS
   ├── Lambda
   ├── DynamoDB
   ├── Recovery Automation
   └── CloudWatch Dashboard
          │
          ▼
Final Integrated System

The architecture documentation explicitly describes this evolution from core infrastructure in Sprint 1 to event-driven automation and observability in Sprint 2.

Learning Outcomes

This project provided practical exposure to:

AWS cloud infrastructure
infrastructure monitoring
event-driven architecture
high-availability design
load balancing
auto scaling
serverless functions
NoSQL event storage
IAM and access control
cloud observability
failure simulation
automated recovery
cloud infrastructure troubleshooting
References

The implementation was based primarily on AWS documentation for:

Amazon EC2
Amazon CloudWatch
AWS Lambda
Amazon DynamoDB
Amazon EC2 Auto Scaling
Elastic Load Balancing

The project report also references research on scalable cloud availability and reliability.

Project Summary
┌──────────────────────────────────────────────────────────┐
│       CLOUD-BASED SELF-HEALING INFRASTRUCTURE            │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  Users                                                   │
│    │                                                     │
│    ▼                                                     │
│  Application Load Balancer                               │
│    │                                                     │
│    ▼                                                     │
│  Auto Scaling Group                                      │
│    │                                                     │
│    ├── EC2 Instance                                      │
│    ├── EC2 Instance                                      │
│    └── EC2 Instance                                      │
│                                                          │
│    │                                                     │
│    ▼                                                     │
│  CloudWatch ──→ Alarm ──→ SNS ──→ Administrator          │
│                    │                                     │
│                    └────→ Lambda ──→ DynamoDB            │
│                                                          │
│  Auto Scaling ──→ Scale / Replace Unhealthy Instances    │
│                                                          │
│  CloudWatch Dashboard ──→ System Visibility              │
│                                                          │
└──────────────────────────────────────────────────────────┘
Core Architecture

AWS EC2 + Application Load Balancer + Auto Scaling + CloudWatch + SNS + Lambda + DynamoDB

Core Principle

Detect → Alert → Log → Recover → Monitor
