# Lift-and-Shift Multi-Tier Web Application to AWS (V-Profile)

> **Project Type:** Cloud Migration (Lift & Shift)  
> **Platform:** AWS  
> **App Stack:** V-Profile (multi-tier Java web app)  
> **Outcome:** Migrated a locally hosted VM-based stack to AWS with load balancing, private DNS, and autoscaling.

---

## 1) Project Summary

This project migrates a multi-tier web application stack (V-Profile) that previously ran on local virtual machines (Vagrant-based) into AWS using a **lift-and-shift** strategy.

Instead of managing physical/virtual infrastructure in a local datacenter (high ops overhead, difficult scaling, manual processes), the stack is deployed onto AWS IaaS components for:

- Pay-as-you-go cost model  
- Better scalability and availability  
- Cleaner networking and security segmentation  
- Easier automation and standardized deployment flow  

---

## 2) Problem and Motivation

### Problems with the original (local/datacenter) model

- Complex infrastructure management across multiple teams  
- Scaling up/down was slow and manual  
- High upfront CapEx + ongoing OpEx  
- Hard to automate reliably  
- Time-consuming troubleshooting and maintenance  

### Why AWS lift-and-shift

- Move the same architecture to cloud infrastructure quickly  
- Reduce operational burden  
- Prepare the foundation for future re-architecture (cloud-native modernization)  

---

## 3) Target Architecture on AWS

### High-level architecture flow

1. User accesses the application via a public URL (DNS hosted at domain registrar).
2. Traffic hits an **AWS Application Load Balancer (ALB)** over HTTPS.
3. ALB forwards requests to **Tomcat EC2 instances** managed by an **Auto Scaling Group (ASG)**.
4. The app connects to backend services (MySQL, Memcached, RabbitMQ) using **Route 53 Private Hosted Zone DNS names**.
5. Application build artifact is stored in **S3** and pulled to the Tomcat server for deployment.

✅ **AWS services used**

- EC2 (Tomcat, MySQL, Memcached, RabbitMQ)
- Application Load Balancer (replacing NGINX)
- Auto Scaling Group
- Route 53 Private Hosted Zone (internal DNS)
- S3 (artifact storage)
- IAM (user keys + instance role)
- ACM (TLS certificate for HTTPS)
- EBS (instance volumes)

---

## 4) Architecture Diagrams

### 4.1 Previous (Local VM-Based) Architecture

![alt text](img/56.old-architecture.png)

### 4.2 Target (AWS) Architecture

![alt text](img/55.architecturediagramreview.png)

---

## 5) Implementation Steps (What I Did)

### Step 1: Create Key Pair

Created an EC2 key pair for SSH access to instances.

![alt text](img/keypair.png)

---

### Step 2: Create Security Groups (Network Segmentation)

Created 3 security groups:

#### A) Load Balancer SG
- Inbound: HTTP 80 (initially), HTTPS 443 (final)
- Source: Internet

#### B) App (Tomcat) SG
- Inbound: 8080 from Load Balancer SG
- Inbound: 22 from my IP (SSH)

#### C) Backend SG (MySQL, Memcached, RabbitMQ)
Inbound:
- 3306 from App SG (MySQL)
- 11211 from App SG (Memcached)
- 5672 from App SG (RabbitMQ)
- 22 from my IP (SSH)

Plus:
- Allow all traffic within Backend SG (so backend services can talk to each other)

![alt text](img/3-SG-created.png)

![alt text](img/ELB-sg-inbound-rules.png)

![alt text](img/backend-inbound-rules.png)

![alt text](img/backend-inbound-rules-between-servers.png)


### Step 3: Launch EC2 Instances with User Data Scripts

Provisioned the following EC2 instances using user data scripts:

- `vprofile-db01` (Amazon Linux 2023) -> MariaDB/MySQL
- `vprofile-mc01` (Amazon Linux 2023) -> Memcached
- `vprofile-rmq01` (Amazon Linux 2023) -> RabbitMQ
- `vprofile-app01` (Ubuntu 24) -> Tomcat 10

Validated services using `systemctl status` and basic checks (DB login, service health).

![alt text](img/vprofile-project-sourcecode-git.png)

![alt text](img/1.switch-branch-aws-lift-and-shift.png)

![alt text](img/2.launch-mysql-instance.png)

![alt text](img/3.Ec2-instances-up.png)

![alt text](img/4.DB-instance-mariadb.png)

![alt text](img/6.Memcache-server-active-check.png)

![alt text](img/7.RabbitMQ-active-running.png)





---

### Step 4: Configure Private DNS with Route 53 (Private Hosted Zone)

Created a Route 53 **Private Hosted Zone** (e.g., `vprofile.in`) and added A-records mapping:

- `db01.vprofile.in` -> private IP of DB instance
- `mc01.vprofile.in` -> private IP of Memcached instance
- `rmq01.vprofile.in` -> private IP of RabbitMQ instance

Tested DNS resolution from the app server using `ping`.

![alt text](img/8.create-hosted-zone.png)

![alt text](img/9.create-hosted-zonea.png)

![alt text](img/10.create-record-get-private-ip.png)

![alt text](img/11.Name-2-IP-mappinga.png)

![alt text](img/12.create-record-get-private-ip-mc01.png)

![alt text](img/13.Name-2-IP-mapping-mc01.png)

![alt text](img/14.create-record-get-private-ip-rmq01.png)

![alt text](img/15.Name-2-IP-mapping-rmq01.png)

![alt text](img/16.created-records.png)

![alt text](img/17.error-occured.png)

![alt text](<img/18.added-ICMP-to fix-the-issue.png>)

![alt text](img/19.check-db-connection.png)

---

### Step 5: Build Application Artifact Locally and Upload to S3

On local machine:

- Confirmed toolchain:
  - Java 17
  - Maven 3.9.x
  - AWS CLI
- Updated `application.properties` to use internal DNS names (`db01.vprofile.in`, etc.)
- Built artifact using:
  - `mvn install`
- Uploaded artifact (WAR) to S3 bucket using AWS CLI.

![alt text](img/20.artefact-build-flow-chart.png)

![alt text](img/21.creates3bucketforartefact.png)

![alt text](img/22.create-user-give-s3-full-access.png)

![alt text](img/23.create-access-keys.png)

![alt text](img/24.Iamrolesfors3.png)

![alt text](img/25.apply-IAMrole-to-appserver.png)

![alt text](img/26.update-Iam-role.png)

![alt text](img/27.modify-application.propertiesfile.png)

![alt text](img/28.check-requirements.png)

![alt text](img/29.build-the-app.png)

![alt text](img/30.artefact-generated.png)

---

### Step 6: Deploy Artifact to Tomcat EC2 Instance

On `vprofile-app01`:

- Installed AWS CLI (snap on Ubuntu)
- Pulled artifact from S3
- Stopped Tomcat, removed default ROOT app, deployed WAR as ROOT
- Restarted Tomcat and verified app responds

![alt text](img/31.awsconfigured-with-accesskeys.png)

![alt text](img/32.upload-artefact-to-s3.png)

![alt text](img/34.s3-content-list.png)

![alt text](img/35.snap-install-aws-cli.png)

![alt text](img/36.copy-artefact.png)

![alt text](img/37.artefact-extracted.png)

![alt text](img/38.add-additional-rule.png)

![alt text](<img/39. public-ip-app-server-to-test.png>)

![alt text](img/40.website-running.png)

---

### Step 7: Create ALB + HTTPS using ACM

- Created Target Group on port `8080`
- Created Application Load Balancer
- Added HTTPS listener (`443`) using ACM certificate
- Routed traffic to target group
- Optional: mapped custom domain using registrar DNS (CNAME to ALB DNS name)

Validated:
- Target is healthy
- HTTP/HTTPS works
- Certificate valid on browser (HTTPS)

![alt text](img/41.create-targrt-groupa.png)

![alt text](img/41.create-targrt-groupb.png)

![alt text](img/41.create-targrt-groupc.png)

![alt text](img/41.create-targrt-groupd.png)

![alt text](img/41.create-targrt-groupe.png)

![alt text](img/41.create-targrt-groupf.png)

![alt text](img/41.TG-created.png)

![alt text](img/42.aws-certificate-manager-request.png)

![alt text](img/42.aws-certificate-manager-setup.png)

![alt text](img/42.purchased-domain-godaddy.png)

![alt text](img/43.load-balancer-creationa.png)

![alt text](img/43.load-balancer-creationb.png)

![alt text](img/43.load-balancer-creationc.png)

![alt text](img/43.load-balancer-creationd.png)

![alt text](img/43.load-balancer-creatione.png)

![alt text](img/43.load-balancer-provisioning-completed.png)

![alt text](img/45.DNS-record-for-elb.png)

![alt text](img/47.targetgrp-health-check-completed.png)

![alt text](img/48.ELB-DNScheck-on-browser-working.png)

![alt text](img/49.certificate-check.png)

![alt text](img/49.https-app-check-working.png)

![alt text](img/49.logged-in-db-connection-verified.png)

![alt text](img/49.memcache-working.png)

![alt text](img/49.rabbitmq-working.png)
---

### Step 8: Configure Auto Scaling Group (ASG)

- Created AMI from the working Tomcat app instance
- Created Launch Template using the AMI + App SG
- Created Auto Scaling Group:
  - Desired capacity: 1 (or more as needed)
  - Max capacity: 4
  - Scaling policy: CPU utilization target (e.g., 50%)
- Enabled load balancer health checks

#### Sticky sessions (important for this app)

Enabled **stickiness** on the Target Group to avoid session logout issues across multiple instances.

![alt text](img/50.create-image-for-appserver-instance.png)

![alt text](img/50.image-created.png)

![alt text](img/51.create-launch-templatea.png)

![alt text](img/51.create-launch-templateb.png)

![alt text](img/51.create-launch-templatec.png)

![alt text](img/51.create-launch-templated.png)

![alt text](img/52.create-ASG.png)

![alt text](img/52.create-ASGa.png)

![alt text](img/52.create-ASGb.png)

![alt text](img/52.create-ASGc.png)

![alt text](img/52.create-ASGd.png)

![alt text](img/52.create-ASGe.png)

![alt text](img/52.create-ASGf.png)

![alt text](img/52.ASG-created.png)

![alt text](img/53.turn-on-stickiness.png)

![alt text](img/54.ASG-launced-instance.png)
---

## 6) Verification and Testing

Validation steps performed:

- ✅ DB login works (app can authenticate)
- ✅ RabbitMQ queue creation works (connection verified)
- ✅ Memcached caching works:
  - first hit: data from DB + cached
  - second hit: data served from cache
- ✅ ALB health checks show targets as healthy
- ✅ HTTPS certificate is valid in browser
- ✅ ASG maintains desired instance count

**ADD IMAGE HERE**  
Suggested filename: `images/18-app-login-and-cache-check.png`

![App login and cache verification](images/18-app-login-and-cache-check.png)

---

## 7) Deliverables

- Working AWS-hosted V-Profile application behind ALB
- Secure segmentation using Security Groups
- Private service discovery via Route 53 Private Hosted Zone
- Artifact storage and delivery through S3
- Autoscaling-ready architecture (ASG + Launch Template + AMI)
- Optional custom domain + HTTPS via ACM

---

## 8) Lessons Learned (Reflection)

A few things really stood out:

- Private DNS (Route 53 Hosted Zone) keeps configs stable even when instances change.
- Security groups need to be designed from traffic flow, not guesswork.
- Stickiness can be a lifesaver for simple apps that do not handle distributed sessions well.
- Lift-and-shift is fast, but it also shows where cloud-native improvements can reduce instance sprawl.

---

## 9) What I’d Improve Next (Future Work)

If I revisit this project, I’d modernize it by:

- Moving database to RDS
- Using ElastiCache for Memcached
- Using Amazon MQ or SQS for messaging patterns
- Containerizing the app (ECS/EKS)
- CI/CD pipeline for automated deployments (instead of manual WAR pulls)
- Secrets management via SSM Parameter Store or Secrets Manager
