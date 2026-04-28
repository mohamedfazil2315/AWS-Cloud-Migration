# On-Premise to AWS Migration Project

## 1. Project Overview

This project demonstrates the migration of a traditional on-premise web application to AWS Cloud with a focus on:

- High availability  
- Scalability  
- Security  
- Minimal downtime  

The existing application consists of a single application server, a MySQL database, and local file storage. The migration transforms this architecture into a cloud-native, highly available system using AWS managed services.

---

## 2. Existing Architecture (On-Premise)

- Single Linux-based application server (Node.js)  
- MySQL database hosted on a separate server  
- Local file storage for static assets  
- No load balancing or auto scaling  
- Public access via static IP  

### Limitations

- Single point of failure  
- No scalability  
- Manual infrastructure management  
- Limited monitoring and security controls  

---

## 3. Target AWS Architecture

The application is migrated to AWS with the following architecture:

- Application Load Balancer in public subnets  
- Auto Scaling Group with EC2 instances in private subnets  
- Amazon RDS MySQL (Multi-AZ) in private subnets  
- Amazon S3 for file storage  
- Custom VPC with public and private subnets  
- NAT Gateway for outbound internet access  
- CloudWatch for monitoring and logging  

---

## 4. Architecture Diagram (Logical Flow)
Client → Route53 → ALB → EC2 (Auto Scaling) → RDS
↓
S3 (File Storage)


---

## 5. AWS Services Used

- Amazon EC2  
- Auto Scaling Group  
- Application Load Balancer (ALB)  
- Amazon RDS (MySQL, Multi-AZ)  
- Amazon S3  
- AWS VPC  
- Internet Gateway  
- NAT Gateway  
- AWS IAM  
- AWS CloudWatch  
- AWS Database Migration Service (DMS)  

---

## 6. Implementation Steps

### Step 1: VPC and Networking Setup

- Created a custom VPC with CIDR block `10.0.0.0/16`  
- Created two public subnets for ALB  
- Created two private subnets for EC2 and RDS  
- Attached Internet Gateway to VPC  
- Configured NAT Gateway for private subnet outbound traffic  

**Route Tables:**
- Public route table → Internet Gateway  
- Private route table → NAT Gateway  

---

### Step 2: Security Configuration

**Security Groups:**

- **ALB Security Group**
  - Allow HTTP (80) and HTTPS (443) from anywhere  

- **EC2 Security Group**
  - Allow HTTP traffic only from ALB  
  - Allow SSH from trusted IP  

- **RDS Security Group**
  - Allow MySQL (3306) only from EC2  

Applied least privilege principle.

---

### Step 3: EC2 Setup with Launch Template

- Created Launch Template:
  - Amazon Linux / Ubuntu AMI  
  - Instance type: t2.micro / t3.micro  
  - IAM role attached (S3 + CloudWatch access)  

**Installed Dependencies:**
- Python, pip, Git  
- Application dependencies (Django/Node.js)  

Deployed application code from repository.

---

### Step 4: Auto Scaling Group Configuration

- Minimum instances: 2  
- Desired instances: 2  
- Maximum instances: 4  
- Attached to ALB Target Group  

---

### Step 5: Application Load Balancer Setup

- Created ALB in public subnets  
- Configured HTTP/HTTPS listener  
- Created Target Group:
  - Registered EC2 instances  
  - Configured health check endpoint  

---

### Step 6: RDS MySQL Setup

- Engine: MySQL  
- Multi-AZ enabled  
- Private subnet deployment  
- Public access disabled  
- Configured DB security group  

---

### Step 7: Database Migration using DMS

- Created DMS Replication Instance  
- Configured endpoints:
  - Source: On-premise MySQL  
  - Target: AWS RDS  

**Migration Type:**
- Full Load + Change Data Capture (CDC)

**Migration Flow:**
1. Initial full data load  
2. Continuous replication using CDC  

---

### Step 8: File Storage Migration to S3

- Created S3 bucket  

**Command:**
```bash
aws s3 sync /local-folder s3://bucket-name

Step 9: IAM Role Configuration
Created IAM Role for EC2:
S3 access
CloudWatch logging
Avoided hardcoded credentials
Step 10: Monitoring and Logging
Enabled CloudWatch metrics
Configured CloudWatch Logs

Alarms Created:

CPU Utilization > 70%
Unhealthy EC2 instances
7. Migration Strategy
Application Migration
Manual deployment on AWS EC2
On-premise system kept running during migration
Database Migration
AWS DMS:
Full Load
CDC for real-time sync
8. Downtime Minimization Strategy
Continuous replication using CDC

Cutover Steps:

Stop writes on on-premise DB
Wait for final sync
Switch application to RDS
Update DNS to ALB

Downtime: Minimal (seconds to minutes)

9. Data Consistency Approach
CDC ensures no data loss
Writes stopped before final cutover
Post-migration validation performed
10. Rollback Plan
On-premise system kept active

If failure occurs:

Redirect DNS back to on-premise
Resume application
11. Validation and Testing
Application access via ALB
Database connectivity
File upload to S3
Load balancing test
Auto Scaling failover testing
12. Security Best Practices
Private subnets for EC2 and RDS
No public DB access
IAM roles instead of credentials
Restricted Security Groups
13. Optional Enhancements
Route53 for DNS
AWS ACM for HTTPS
Auto Scaling policies
CI/CD pipeline (Jenkins / GitHub Actions)
Terraform for Infrastructure as Code
14. Challenges and Solutions

Challenge: Minimizing downtime
Solution: DMS with CDC and controlled cutover

Challenge: Secure database access
Solution: Restricted access using Security Groups

Challenge: File storage migration
Solution: Migrated to S3 and updated application

15. Final Outcome
Highly available system (Multi-AZ)    
Scalable infrastructure (Auto Scaling)
Secure architecture (private networking + IAM)
Near-zero downtime migration
Production-ready AWS deployment
