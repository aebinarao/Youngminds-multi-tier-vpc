# Multi-Tier VPC Architecture for Youngminds


📋 Table of Contents

- Overview
- Architecture
- Features
- Technical Implementation
- Security
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

Some snapshots

10.0.1.25 - Bastion host (Public Instance) - AZ1a
10.0.2.56 - Web app server (Private Instance) - AZ1a

10.0.10.42 - Bastion host (Public Instance) - AZ1b
10.0.20.73 - Web app server (Private Instance) - AZ1b


Test if Private instance Web app server can connect to S3 bucket:

<p align="center">
  <img src="/images/test1.png" width="1000" />
</p>

- Test if we can SSH from bastion host az-1b to web app server az1-b
- Test if we can access S3 Bucket from web app server az-1b

<p align="center">
  <img src="/images/test2.png" width="1000" />
</p>

- Test if we can SSH from bastion host az-1a to web app server az1-a
- Test if we can access S3 Bucket from web app server az-1a
- Test if we can access google from the web app server

<p align="center">
  <img src="/images/test3.png" width="1000" />
</p>

- Test SSH from bastion host in az-1b to web app server private instance in az-1a:

<p align="center">
  <img src="/images/test4.png" width="1000" />
</p>

- Test if we can connect to s3 bucket using the web app server in AZ-1a:

<p align="center">
  <img src="/images/test5.png" width="1000" />
</p>

- This means we can connect through different availability zones

***
**🛠️TECHNOLOGIES USED**

- Cloud Provider: Amazon Web Services (AWS)
- Networking: VPC, Subnets, Route Tables, NAT Gateway, Internet Gateway
- Security: Security Groups, VPC Flow Logs, IAM Roles
- Compute: EC2 (t2.micro instances)
- Storage: S3 with Gateway Endpoint
- Monitoring: CloudWatch Logs
- Infrastructure as Code: Terraform
- Version Control: Git, GitHub

***
👤 About the Author

Anthonie Elizaldy Binarao

- 📧 Email: hechanovanton@gmail.com
- 💼 LinkedIn: [linkedin.com/in/anthonie-binarao](https://www.linkedin.com/in/anthonie-binarao/)
- 🌐 Portfolio: yourwebsite.com
- 🎓 Certifications: CCNA Full Course, AWS Cloud Practitioner Essentials

***
🙏 Acknowledgments

- AWS Documentation and Best Practices guides
- CCNA training for foundational networking concepts
- AWS Community for troubleshooting assistance
- Terraform documentation and examples

***
**Note:**

*The files stored inside the S3 bucket were my final packet tracer project for the Network Engineering 1 class way back in college.
We were tasked to create a network topology of a hypothetical company named "Youngminds." So I referenced the hypothetical company in this cloud networking project.
This personal cloud networking project exceeds the scope of Network Engineering 1 in our computer science class, as cloud computing was not taught in our curriculum. 
I built this project through self-study by exploring official documentation and applying my foundational networking knowledge from the CCNA course.
I recognize that there are still areas for improvement, as I am continuously learning cloud and advanced networking concepts. This project reflects my willingness to learn, experiment, and apply new technologies beyond what was required in class.






