# Youngminds-multi-tier-vpc
Youngminds AWS Multi-tier VPC Architecture


📋 Table of Contents

- Overview
- Architecture
- Features
- Technical Implementation
- Security
- Cost Optimization
- Deployment
- Testing


***
🎯 Overview

This project demonstrates enterprise-level cloud networking practices by implementing a secure, scalable, multi-tier VPC architecture on AWS. The design emphasizes security through defense-in-depth, high availability through multi-AZ deployment, and cost efficiency through strategic use of AWS services.


Project Objectives

Design and implement a production-ready VPC architecture
Demonstrate network segmentation and security best practices
Implement high availability across multiple Availability Zones
Optimize costs through VPC endpoints and resource selection
Document infrastructure for team collaboration and knowledge transfer

***
🏗️ Architecture

Network Topology
<p align="center">
  <img src="/images/Youngminds Multi-Tier VPC.drawio.png" width="1000" />
</p>

***
Traffic Flow Scenarios

Scenario 1: Administrative Access

Administrator → Internet → IGW → Public Subnet → Bastion Host → Private Subnet → Web Server


Scenario 2: Private Instance Internet Access

Web Server → Private Subnet → NAT Gateway → IGW → Internet

***
✨ Features
High Availability

- ✅ Multi-AZ deployment across 2 Availability Zones
- ✅ Redundant public and private subnets
- ✅ Fault-tolerant design (single AZ failure doesn't impact availability)

Security

- ✅ Network Segmentation: Public/private subnet isolation
- ✅ Bastion Host Architecture: Controlled administrative access
- ✅ Defense-in-Depth: Multiple security layers
- ✅ Least Privilege: Security groups follow principle of least privilege
- ✅ No Direct Internet Access: Private instances have no public IPs
- ✅ VPC Flow Logs: Comprehensive network traffic monitoring

Cost Optimization

- ✅ S3 Gateway Endpoint: Eliminates NAT charges for S3 traffic (~$4.50/month savings)
- ✅ Free Tier Resources: t3.micro instances, optimized storage
- ✅ Single NAT Gateway: Appropriate for non-production (HA would require 2)
- ✅ Efficient Log Retention: 7-day retention balances visibility and cost

Monitoring & Logging

- ✅ VPC Flow Logs integrated with CloudWatch
- ✅ Real-time traffic analysis capability
- ✅ Security event detection
- ✅ Compliance audit trail

***
🔧 Technical Implementation

Infrastructure Components

<table>
  <tr>
    <th>Component</th>
    <th>Configuration</th>
    <th>Purpose</th>
  </tr>
  <tr>
    <td>VPC</td>
    <td>10.0.0.0/16</td>
    <td>Isolated network environment</td>
  </tr>
  <tr>
    <td>Public Subnet</td>
    <td>10.0.1.0/24, 10.0.10.0/24</td>
    <td>Internet-facing resources</td>
  </tr>
  <tr>
    <td>Private Subnets</td>
    <td>10.0.2.0/24, 10.0.20.0/24</td>
    <td>Internal application servers</td>
  </tr>
  <tr>
    <td>Internet Gateway</td>
    <td>1x IGW</td>
    <td>Public subnet internet connectivity</td>
  </tr>
  <tr>
    <td>NAT Gateway</td>
    <td>1x NAT-GW</td>
    <td>Private subnet outbound internet</td>
  </tr>
  <tr>
    <td>Route Tables</td>
    <td>2x (Public, Private)</td>
    <td>Traffic routing control</td>
  </tr>
  <tr>
    <td>Security Groups</td>
    <td>2x (Bastion, Web-App))</td>
    <td>Instance-level firewall</td>
  </tr>
  <tr>
    <td>S3 Gateway Endpoint</td>
    <td>1x VPC Endpoint</td>
    <td>Cost-optimized S3 access</td>
  </tr>
  <tr>
    <td>VPC Flow Logs</td>
    <td>Enabled (All traffic)</td>
    <td>Network monitoring</td>
  </tr>
  <tr>
    <td>CloudWatch Logs</td>
    <td>Log Group + IAM Role</td>
    <td>VPC Flow Logs storage & analysis</td>
  </tr>
  <tr>
    <td>S3 Bucket</td>
    <td>Standard tier, encrypted</td>
    <td>Application data storage</td>
  </tr>
</table>

***
Routing Configuration

Public Route Table:

<table>
  <tr>
    <th>Destination</th>
    <th>Target</th>
    <th>Purpose</th>
  </tr>
   <tr>
    <td>10.0.0.0/16</td>
    <td>local</td>
    <td>Internal VPC traffic</td>
  </tr>
   <tr>
    <td>0.0.0.0/0</td>
    <td>igw-xxxxx</td>
    <td>Internet access</td>
  </tr>
</table>


Private Route Table:

<table>
  <tr>
    <th>Destination</th>
    <th>Target</th>
    <th>Purpose</th>
  </tr>
   <tr>
    <td>10.0.0.0/16</td>
    <td>local</td>
    <td>Internal VPC traffic</td>
  </tr>
   <tr>
    <td>0.0.0.0/0</td>
    <td>nat-xxxxx</td>
    <td>Outbound internet via NAT</td>
  </tr>
  <tr>
    <td>pl-xxxxx</td>
    <td>vpce-xxxxx</td>
    <td>S3 access via endpoint</td>
  </tr>
</table>

***
🔒 Security
Security Architecture

This implementation follows AWS security best practices and implements a defense-in-depth strategy:


**Layer 1:** Network Isolation

- Private subnets have no direct internet access
- No public IP addresses assigned to private instances
- Network segmentation enforced through route tables

**Layer 2:** Bastion Host (Jump Server)

- Single controlled entry point for administrative access
- Security group restricts SSH to authorized IPs only
- All administrative access logged and auditable
- Key-based authentication required

**Layer 3:** Security Groups (Stateful Firewall)

Inbound:
  - SSH (22) from Admin IP only

Outbound:
  - All traffic (for SSH to private instances)
    

Web Application Security Group:

Inbound:
  - SSH (22) from Bastion Security Group only
  - Future: HTTP/HTTPS from ALB Security Group
    
Outbound:
  - HTTP (80) to 0.0.0.0/0 (package updates)
  - HTTPS (443) to 0.0.0.0/0 (package updates, API calls)

**Layer 4:** IAM Roles

- EC2 instances use IAM roles (no hardcoded credentials)
- Least privilege permissions for S3 access
- Role-based access control for AWS services


**Layer 5:** Monitoring & Detection

- VPC Flow Logs capture all network traffic
- CloudWatch integration for real-time monitoring
- Audit trail for security investigations

***
🚀 Deployment

**Prerequisites:**

- AWS Account with appropriate permissions (VPC, EC2, S3, IAM, CloudWatch)
- AWS CLI configured (optional but recommended)
- SSH key pair for EC2 access
- Basic understanding of networking and AWS services

***
**TESTING**

Test if Private instance Web app server can connect to S3 bucket:

<p align="center">
  <img src="/images/test1.png" width="1000" />
</p>

- Test if we can SSH from bastion host az-1b to web app server az1-b
- Test if we can access S3 Bucket from web app server az-1b

<p align="center">
  <img src="/images/test2.png" width="1000" />
</p>

*The files inside the S3 bucket was my Final packet tracer project in my Network Engineering 1 class
We were tasked to create a network topology of a hypothetical company named "Youngminds".






This project was done for approximately 5 hours (including testing of components)
