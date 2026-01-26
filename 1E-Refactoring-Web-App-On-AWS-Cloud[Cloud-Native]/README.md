# Re-architecting Web Apps on AWS Cloud (Cloud Native)  
## Project: VProfile Refactoring with AWS Managed Services

> **Project Type:** Re-architecture / Refactoring (from Lift-and-Shift to Managed Services)  
> **App Stack:** VProfile (Java/Tomcat)  
> **Goal:** Reduce ops overhead, improve scalability + reliability, adopt AWS managed services (PaaS/SaaS).

---
## Table of Contents

- [1. Project Overview](#1-project-overview)
- [2. Why Refactor](#2-why-refactor)
- [3. Target Architecture](#3-target-architecture)
  - [High-Level Flow](#high-level-flow)
  - [Architecture Diagram](#architecture-diagram)
  - [Flowchart Diagram](#flowchart-diagram)
- [4. Services Used](#4-services-used)
  - [Frontend / App Hosting](#frontend--app-hosting)
  - [Backend](#backend)
  - [Networking + Delivery](#networking--delivery)
  - [Service Mapping (Old vs New)](#service-mapping-old-vs-new)
- [5. Key Design Decisions](#5-key-design-decisions)
- [6. Prerequisites](#6-prerequisites)
  - [AWS](#aws)
  - [Local Tools](#local-tools)
  - [Domain + Certificate](#domain--certificate)
- [7. Implementation Phases](#7-implementation-phases)
  - [Phase 1: Security Group + Key Pair](#phase-1-security-group--key-pair)
    - [1. Create Backend Security Group](#1-create-backend-security-group)
    - [2. Add Inbound Rule: Allow All Traffic Within Itself](#2-add-inbound-rule-allow-all-traffic-within-itself)
    - [3. Create Key Pair](#3-create-key-pair)
  - [Phase 2: RDS Setup](#phase-2-rds-setup)
    - [1. Create RDS Parameter Group (MySQL 8)](#1-create-rds-parameter-group-mysql-8)
    - [2. Create DB Subnet Group](#2-create-db-subnet-group)
    - [3. Create RDS Instance](#3-create-rds-instance)
  - [Phase 3: ElastiCache Setup](#phase-3-elasticache-setup)
    - [1. Create Cache Parameter Group (Memcached 1.6)](#1-create-cache-parameter-group-memcached-16)
    - [2. Create Cache Subnet Group](#2-create-cache-subnet-group)
    - [3. Create Memcached Cluster](#3-create-memcached-cluster)
  - [Phase 4: Amazon MQ Setup](#phase-4-amazon-mq-setup)
    - [1. Create Amazon MQ RabbitMQ Broker](#1-create-amazon-mq-rabbitmq-broker)
  - [Phase 5: Database Initialization](#phase-5-database-initialization)
    - [1. Launch Temporary EC2 (MySQL Client)](#1-launch-temporary-ec2-mysql-client)
    - [2. Allow EC2 → RDS Access](#2-allow-ec2--rds-access)
    - [3. Install Tools on EC2](#3-install-tools-on-ec2)
    - [4. Login to RDS](#4-login-to-rds)
    - [5. Clone Source Code + Switch Branch](#5-clone-source-code--switch-branch)
    - [6. Import DB Schema](#6-import-db-schema)
  - [Phase 6: Elastic Beanstalk Setup](#phase-6-elastic-beanstalk-setup)
    - [1. Create IAM Role for Elastic Beanstalk EC2 Instances](#1-create-iam-role-for-elastic-beanstalk-ec2-instances)
    - [2. Create Elastic Beanstalk Application + Environment](#2-create-elastic-beanstalk-application--environment)
    - [3. Configure Service Access (Roles + Key Pair)](#3-configure-service-access-roles--key-pair)
    - [4. Networking Configuration](#4-networking-configuration)
    - [5. Instance Settings (Root Volume + Monitoring)](#5-instance-settings-root-volume--monitoring)
    - [6. Capacity, Scaling, and Load Balancer](#6-capacity-scaling-and-load-balancer)
    - [7. Enable Stickiness (Session Affinity)](#7-enable-stickiness-session-affinity)
    - [8. Health Check Configuration](#8-health-check-configuration)
    - [9. Deployment Policy (Rolling Updates)](#9-deployment-policy-rolling-updates)
    - [10. Create the Environment](#10-create-the-environment)
    - [11. Update Backend Security Group to Allow Beanstalk Access (Later Step)](#11-update-backend-security-group-to-allow-beanstalk-access-later-step)
  - [Phase 7: Build + Deploy Artifact](#phase-7-build--deploy-artifact)
    - [1. Collect Backend Endpoints](#1-collect-backend-endpoints)
    - [2. Clone Repo Locally + Switch Branch](#2-clone-repo-locally--switch-branch)
    - [3. Update `application.properties`](#3-update-applicationproperties)
    - [4. Build Artefact](#4-build-artefact)
    - [5. Upload + Deploy to Beanstalk](#5-upload--deploy-to-beanstalk)
  - [Phase 8: HTTPS + Custom Domain](#phase-8-https--custom-domain)
    - [1. Add HTTPS Listener (443) on ALB via Beanstalk](#1-add-https-listener-443-on-alb-via-beanstalk)
    - [2. Create DNS Record (GoDaddy)](#2-create-dns-record-godaddy)
  - [Phase 9. CloudFront CDN](#phase-9-cloudfront-cdn)
    - [1. Create CloudFront Distribution](#1-create-cloudfront-distribution)
    - [2. Add Alternate Domain Name (CNAME)](#2-add-alternate-domain-name-cname)
    - [3. Update DNS to Point to CloudFront](#3-update-dns-to-point-to-cloudfront)
  - [Phase 10: Validation](#phase-10-validation)
    - [1. Functional Testing](#1-functional-testing)
  - [Phase 11: Cleanup](#phase-11-cleanup)
    - [1. CloudFront](#1-cloudfront)
    - [2. DNS Record](#2-dns-record)
    - [3. RDS](#3-rds)
    - [4. ElastiCache](#4-elasticache)
    - [5. Amazon MQ](#5-amazon-mq)
    - [6. Security Groups](#6-security-groups)
    - [7. Elastic Beanstalk](#7-elastic-beanstalk)
    - [8. Key Pair](#8-key-pair)
- [8. Troubleshooting Tips](#8-troubleshooting-tips)
  - [1. App not loading after deploy](#1-app-not-loading-after-deploy)
  - [2. DB connection fails](#2-db-connection-fails)
  - [3. RabbitMQ connection fails](#3-rabbitmq-connection-fails)
  - [4. Sticky session issues](#4-sticky-session-issues)
  - [5. CloudFront shows old content](#5-cloudfront-shows-old-content)


---

## 1. Project Overview
In the previous project, VProfile was deployed using a **Lift-and-Shift** approach (EC2-focused, manual service management).

In this project, we **refactor** the stack to AWS-managed services so we can:
- scale with less effort
- reduce operational overhead
- improve uptime and agility
- stop babysitting servers all day

This is what “cloud native” starts to feel like. Not perfect, but way better.

---

## 2. Why Refactor
When services run on physical servers/VMs/EC2 (DB, cache, broker, app servers, DNS, etc.), you typically need:
- sysadmins
- monitoring team
- virtualization team
- cloud team
- ops team
…and still everyone is tired.

Refactoring shifts responsibilities to AWS managed services:
- less patching
- fewer manual backups
- easier scaling
- “pay as you go”
- automation becomes practical (IaC-friendly)


---

## 3. Target Architecture

### High-Level Flow
1. User enters application URL (custom domain)
2. DNS resolves domain to CloudFront
3. CloudFront serves cached content from nearest edge location
4. CloudFront forwards request to ALB (origin)
5. ALB forwards to Beanstalk EC2 instances in ASG
6. Application interacts with:
   - RDS (MySQL) for database
   - ElastiCache (Memcached) for caching
   - Amazon MQ (RabbitMQ) for messaging

### Architecture Diagram

![alt text](img/1.ARchitecture-Diagram.png)

### Flowchart Diagram

![alt text](img/archi2.png)

---
## 4. Services Used

### Frontend / App Hosting
- **Elastic Beanstalk (Tomcat platform)**
  - Creates EC2 instances automatically
  - Creates ALB automatically
  - Creates Auto Scaling automatically
  - Manages deployments + health checks
- **S3**
  - Stores build artifacts (WAR)

### Backend
- **Amazon RDS (MySQL 8.x)** for DB
- **ElastiCache (Memcached 1.6)** for caching
- **Amazon MQ (RabbitMQ)** for messaging

### Networking + Delivery
- **Route 53** (optional) or external registrar (GoDaddy) for DNS
- **CloudFront** for CDN + caching + global edge delivery
- **ACM** for SSL certificates (HTTPS)

### Service Mapping (Old vs New)
| Layer | Lift-and-Shift / Traditional | Refactored (Cloud Native) |
|---|---|---|
| App Hosting | EC2 + Tomcat install | Elastic Beanstalk (Tomcat) |
| Load Balancing | ALB manually configured | Beanstalk-managed ALB |
| Scaling | Auto Scaling manually configured | Beanstalk-managed ASG |
| Artifact Storage | SCP/manual or self S3 | S3 (Beanstalk uses S3) |
| Database | MySQL on EC2 | RDS MySQL |
| Cache | Memcached on EC2 | ElastiCache Memcached |
| Messaging | RabbitMQ on EC2 | Amazon MQ RabbitMQ |
| DNS | Manual | Route 53 or GoDaddy |
| CDN | None | CloudFront |

---

## 5. Key Design Decisions

- **RDS is private** (no public access). DB initialization happens from a temporary EC2 instance in the same VPC.
- **Single backend security group** for RDS + MQ + ElastiCache.
- **Backend SG allows internal traffic within itself** so backend services can talk to each other.
- **Beanstalk SG access is added later** once Beanstalk creates its own security group.
- **Sticky sessions enabled** at ALB target group level (app requires session stickiness).
- **Health check path updated to `/login`** (because app defaults there).
- **Root volume set to GP3** (avoids older Beanstalk launch configuration issues).
- **Rolling deployments, batch size 50%** (safe enough for small ASG).
---

## 6. Prerequisites
### AWS
- AWS account with permissions for:
  - Elastic Beanstalk
  - EC2 / VPC / Security Groups
  - RDS
  - ElastiCache
  - Amazon MQ
  - CloudFront
  - ACM
  - IAM

### Local Tools
- Git
- Maven `3.9.x`
- Java `17+` (Corretto 21 compatible)
- IDE (VS Code recommended)

### Domain + Certificate
- Domain registered (GoDaddy or Route 53)
- ACM certificate:
  - For ALB: certificate must be in the same region as Beanstalk
  - For CloudFront: certificate must be in **us-east-1**


---

## 7. Implementation Phases

### Phase 1: Security Group + Key Pair

#### 1. Create Backend Security Group
Create one SG for backend services (RDS, ElastiCache, Amazon MQ):
- Name: `vprofile-rearch-backend-sg`

#### 2. Add Inbound Rule: Allow All Traffic Within Itself
- Inbound rule:
  - Type: `All traffic`
  - Source: **same SG ID** (`backend-sg-id`)
- Purpose: Allow RDS, MQ, Cache to communicate internally

![alt text](img/2.backend-security-group.png)

![alt text](img/2a.SG-inboundrule.png)

#### 3. Create Key Pair
Create a key pair for potential troubleshooting into Beanstalk EC2 instances:
- Name: `vprofile-rearch-key.pem`

![alt text](img/3.key-pair-created.png)

---

### Phase 2: RDS Setup

#### 1. Create RDS Parameter Group (MySQL 8)
- Engine: MySQL
- Family: MySQL 8.0
- Name: `vprofile-rds-rearch-paramgroup`

![alt text](img/4.create-parametergrp.png)

#### 2. Create DB Subnet Group
- VPC: default (or your custom)
- Subnets: select across AZs
- Name: `vprofile-rds-rearch-subnetgroup`

![alt text](img/5.create-subnetgrp1.png)

![alt text](img/5.create-subnetgrp2.png)

![alt text](img/5.subnetgroup-created.png)

#### 3. Create RDS Instance
- Engine: MySQL `8.0.x`
- Template: Free tier
- Identifier: `vprofile-rearch-db`
- Username: `admin`
- Password: auto-generate (save it)
- Public access: **No**
- Security group: `vprofile-rearch-backend-sg`
- Initial DB name: `accounts`
- Port: `3306`

**Capture:**
- RDS endpoint
- DB username/password

![alt text](img/5.create-subnetgrp1.png)

![alt text](img/6.create-RDS-DB2.png)

![alt text](img/6.create-RDS-DB3.png)

![alt text](img/6.create-RDS-DB4.png)

![alt text](img/6.create-RDS-DB5.png)

![alt text](img/6.create-RDS-DB6.png)

![alt text](img/6.create-RDS-DB7.png)

![alt text](img/6.create-RDS-DB8.png)

![alt text](img/6.create-RDS-DB9.png)

![alt text](img/6.create-RDS-DB10.png)

![alt text](img/6.DB-created.png)

---

### Phase 3: ElastiCache Setup

#### 1. Create Cache Parameter Group (Memcached 1.6)
- Family: `memcached1.6`
- Name: `vprofile-rearch-cache-paramgroup`

![alt text](img/7.create-parametergrp-elasticcache.png)

#### 2. Create Cache Subnet Group
- VPC: default
- Subnets: select across AZs
- Name: `vprofile-rearch-cache-subnetgroup`

![alt text](img/8.create-subnetgrp-elasticcache.png)

![alt text](img/8.subnetgroup-created-elasticcache.png)

#### 3. Create Memcached Cluster
- Engine: Memcached `1.6`
- Node type: `cache.t2.micro` (smallest)
- Nodes: `1`
- Port: `11211`
- Security group: `vprofile-rearch-backend-sg`
- Subnet group: created above

**Capture:**
- Memcached configuration endpoint
- port

![alt text](img/9.create-elasticcache.png)

![alt text](img/9.create-elasticcache2.png)

![alt text](img/9.create-elasticcache3.png)

![alt text](img/9.create-elasticcache4.png)

---

### Phase 4: Amazon MQ Setup

#### 1. Create Amazon MQ RabbitMQ Broker
- Broker: RabbitMQ
- Deployment mode: Single instance
- Instance type: `mq.t3.micro`
- Access type: Private
- VPC: default
- Security group: `vprofile-rearch-backend-sg`
- Username: `rabbit`
- Password: xxxxxxxxxx

**Capture:**
- Broker endpoint (host)
- Port: usually `5671` (TLS)

![alt text](img/10.create-rabbitMQ.png)

![alt text](img/10.create-rabbitMQ1.png)

![alt text](img/10.create-rabbitMQ2.png)

![alt text](img/10.create-rabbitMQ3.png)

![alt text](img/10.create-rabbitMQ4.png)

---

### Phase 5: Database Initialization
Because RDS is private, use a temporary EC2 instance in the same VPC to apply the schema.

#### 1. Launch Temporary EC2 (MySQL Client)
- Ubuntu (t2.micro)
- SSH allowed from your IP only
- Key pair: use existing or new

#### 2. Allow EC2 → RDS Access
Edit backend SG inbound rules:
- MySQL (3306)
- Source: EC2 Client SG

#### 3. Install Tools on EC2
SSH to EC2:

```bash
sudo apt update && sudo apt install mysql-client git -y
```

#### 4. Login to RDS

```
mysql -h <RDS_ENDPOINT> -u admin -p accounts

``` 

#### 5. Clone Source Code + Switch Branch

```
git clone <REPO_URL>
cd vprofile-project
git checkout aws-refactor
```

#### 6. Import DB Schema

```
mysql -h <RDS_ENDPOINT> -u admin -p accounts < src/main/resources/db_backup.sql

```

Verify:
```
mysql -h <RDS_ENDPOINT> -u admin -p accounts
show tables
```
Remember to terminate temporary instance.

![alt text](img/11.create-ec2instance-sql1.png)

![alt text](img/11.create-ec2instance-sql2.png)

![alt text](img/11.create-ec2instance-sql3.png)

![alt text](<img/12. SSH-into-sqlinstance.png>)

![alt text](<img/12. SSH-into-sqlinstance1.png>)

![alt text](<img/12. SSH-into-sqlinstance2.png>)

![alt text](img/12.db-ssh-login-successful.png)

![alt text](img/12.getendpointdetails.png)

![alt text](img/12.inboundrulemodify.png)

![alt text](img/13.cloneVprofileapptoLM.png)

![alt text](img/14.dbinitialisationcompleted.png)

![alt text](img/14.mySQLclient-terminated.png)


### Phase 6: Elastic Beanstalk Setup

Elastic Beanstalk will provision and manage the full “frontend” stack for the application, including:

- EC2 instances running Tomcat
- Auto Scaling Group (ASG)
- Application Load Balancer (ALB)
- Security Groups (for ALB + EC2)
- Deployment orchestration (rolling, immutable, etc.)
- Health monitoring + logs (CloudWatch integrations)

![alt text](img/EBS.png)

---

#### 1. Create IAM Role for Elastic Beanstalk EC2 Instances

Beanstalk needs an IAM role (instance profile) so EC2 instances can perform required operations during deployment and runtime.

**Path:**
- AWS Console → **IAM** → **Roles** → **Create role**
- Trusted entity: **AWS service**
- Use case: **EC2**

**Attach required policies (as per the course/project):**
- `AdministratorAccess-AWSElasticBeanstalk`
- `AWSElasticBeanstalkCustomPlatformforEC2Role`
- `AWSElasticBeanstalkRole`
- `AWSElasticBeanstalkWebTier`

**Role name:**
- `vprofile-rearch-beanstalk-role`

> Note: Policy names can vary slightly by AWS updates, but the intent remains the same:  
> allow Beanstalk web tier + EC2 platform permissions.

![alt text](img/15.createIamrole1.png)

![alt text](img/15.createIamrole2.png)

---

#### 2. Create Elastic Beanstalk Application + Environment

**Path:**
- AWS Console → **Elastic Beanstalk** → **Create application**

**Configuration:**
- Environment tier: **Web server environment**
- Platform: **Tomcat**
- Platform branch: **Tomcat 10 + Corretto 21** (Corretto 17/21 works depending on your build)
- Configuration presets: **Custom configuration**
- Sample app: **Do not use** (you’ll deploy your WAR later)

**Naming (example):**
- Application name: `vprofile-rearch`
- Environment name: `vprofile-rearch-env`
- Domain (unique): choose something like `vprorearch`

![alt text](img/16.createbeanstalk1.png)

![alt text](img/16.createbeanstalk2.png)

![alt text](img/16.createbeanstalk3.png)


---

#### 3. Configure Service Access (Roles + Key Pair)

**Service role (Beanstalk service role):**
- If missing, use “Create role” from Beanstalk UI (it auto-generates the service role)

**EC2 instance profile:**
- Select: `vprofile-rearch-beanstalk-role`

**EC2 Key pair:**
- Select: `vprofile-rearch-key.pem`

![alt text](img/17.create-service-role.png)

![alt text](17.create-service-role1.png)

![alt text](17.create-service-role2.png)

![alt text](17.create-service-role3.png)
---

#### 4. Networking Configuration

**VPC:**
- Default VPC (or your project VPC)

**Instance public IP:**
- Enabled (for troubleshooting access, optional in production)

**Subnets:**
- Select appropriate subnets (often all available in default VPC)

**Database (RDS option inside Beanstalk):**
- Leave **empty**
- We created RDS separately to keep it decoupled and manageable independently

![alt text](img/18.createbeanstalkenviro.png)

![alt text](img/18.createbeanstalkenviro2.png)


---

#### 5. Instance Settings (Root Volume + Monitoring)

**Root volume:**
- Set to `gp3`

> Important: This avoids older “launch configuration” behaviour and ensures modern launch template usage.

**Monitoring:**
- CloudWatch interval: `5 minutes` (good for lab/free-tier friendly setup)

![alt text](img/18.createbeanstalkenviro3.png)

---

#### 6. Capacity, Scaling, and Load Balancer

**Environment type:**
- `Load balanced`

**Auto Scaling:**
- Min: `2`
- Max: `4`
- Instance type: `t2.micro` (or `t3.micro`)

**Load Balancer:**
- Visibility: `Public`
- Type: `Application Load Balancer`
- Listener: `HTTP : 80`

![alt text](img/18.createbeanstalkenviro4.png)

![alt text](img/18.createbeanstalkenviro5.png)

![alt text](img/18.createbeanstalkenviro6.png)

![alt text](img/18.createbeanstalkenviro7.png)
---

#### 7. Enable Stickiness (Session Affinity)

VProfile requires stickiness to avoid session/login issues when traffic is routed across instances.

**Path:**
- Beanstalk → Configuration → Load Balancer → Processes → Edit

**Enable:**
- Stickiness: **ON**

![alt text](img/18.createbeanstalkenviro8.png)

---

#### 8. Health Check Configuration

- Set to Enhanced (deployment safety, health visibility)

![alt text](img/18.createbeanstalkenviro9.png)

![alt text](img/18.createbeanstalkenviro10.png)

---

#### 9. Deployment Policy (Rolling Updates)

Beanstalk supports multiple deployment strategies. For this project:

**Deployment policy:**
- Rolling

**Batch size:**
- `50%`

This means:
- With 2 instances, it updates 1 at a time
- With 4 instances, it updates 2 at a time

![alt text](img/18.createbeanstalkenviro11.png)

![alt text](img/18.createbeanstalkenviro12.png)

---

#### 10. Create the Environment

Review all configuration selections, then launch:

- Click **Submit**
- Wait for environment health to become **Green / OK**

After successful provisioning:
- Note the Beanstalk environment URL (temporary testing URL)

![alt text](img/18.cbeanstalkcreated.png)
---

#### 11. Update Backend Security Group to Allow Beanstalk Access (Later Step)

Once Beanstalk is created, it will create security groups for:
- Beanstalk EC2 instances
- Beanstalk ALB

You must update **backend SG** to allow inbound access *from Beanstalk EC2 SG* for:
- MySQL `3306` (RDS): Source = Beanstalk EC2 SG
- Memcached `11211` (ElastiCache): Source = Beanstalk EC2 SG
- RabbitMQ `5671` (Amazon MQ): Source = Beanstalk EC2 SG
- Keep the existing “all traffic within itself” rule

![alt text](img/33.backend-sg-updated.png)
---

### Phase 7: Build + Deploy Artifact

#### 1. Collect Backend Endpoints

You will need:

- RDS endpoint + username + password

- ElastiCache endpoint + port 11211

- Amazon MQ endpoint + port (often 5671) + credentials

#### 2. Clone Repo Locally + Switch Branch

I used VScode to clone my repo.

![alt text](22.switch-branch-awsrefactor.png)

#### 3. Update `application.properties`

Update values:

- JDBC URL: jdbc:mysql://<RDS_ENDPOINT>:3306/accounts

- Username: admin

- Password: <RDS_PASSWORD>

- Memcached endpoint: <ELASTICACHE_ENDPOINT>:11211

- RabbitMQ endpoint: <AMAZON_MQ_ENDPOINT>:5671 (note port change)

![alt text](img/23.App-prop-file-update.png)

#### 4. Build Artefact

```
mvn -version
mvn install
```
Output artifact:
`target/vprofile-v2.war`

![alt text](img/24.AppbuildMVN.png)

![alt text](img/24.AppbuildMVN1.png)

#### 5. Upload + Deploy to Beanstalk

Beanstalk environment → Upload and Deploy

- Choose WAR file

- Version label: something meaningful

- Deploy

Watch:

- Beanstalk Events

- Target group health (instances rotate unhealthy → healthy during rolling deploy)

![alt text](img/25.deploy-artifact.png)

![alt text](img/26.Env_updated1.png)

![alt text](img/26.Env_updated2.png)
---

### Phase 8: HTTPS + Custom Domain

#### 1. Add HTTPS Listener (443) on ALB via Beanstalk
Beanstalk → Configuration → Load balancer listeners:

Add listener:

- Protocol: HTTPS

- Port: 443

- SSL cert: ACM certificate

Apply changes.

![alt text](28.addACMcertificate.png)

![alt text](29.add-https-listeners.png)

#### 2. Create DNS Record (GoDaddy)
Use a CNAME:

- Name: vprorearch

- Value: Beanstalk environment endpoint (ALB DNS)

Wait for propagation.

Verify:

- Open HTTPS URL

- Certificate is valid

- Login works

![alt text](img/31.test-completed-sc.png)

![alt text](img/31.test-completed-sc1.png)

- Login failed:

![alt text](img/32.login_failed.png)

- Backend SG updated to fix

![alt text](img/33.backend-sg-updated.png)

- Login works

![alt text](img/34.access-approved.png)

![alt text](img/34.access-approved1.png)
---

### Phase 9. CloudFront CDN

Goal: Serve global users faster by caching content in edge locations.

#### 1. Create CloudFront Distribution
- Origin: ALB (Beanstalk load balancer)

- Origin protocol policy: Match viewer (common)

- Allowed methods: include what your app needs (GET/POST etc.)

- Viewer protocol policy: redirect HTTP to HTTPS (recommended), or allow both for learning

Optional:
- WAF (recommended in real projects, skipped here)

![alt text](img/35.CF.png)

#### 2.  Add Alternate Domain Name (CNAME)

Add your domain:

`*.xdctest.xyz`

Attach ACM certificate that matches that domain.

#### 3. Update DNS to Point to CloudFront

CNAME:

- Name: vprorearch

- Value: CloudFront distribution domain (remove https://)

Wait until distribution is Deployed.

### Phase 10: Validation

#### 1. Functional Testing

- Open the app URL

- Login:

- - username: admin_vp

- - password: admin_vp

Confirm app loads dashboard

Confirm cache test works (if app has cache demo page)

![alt text](img/49.https-app-check-working.png)

![alt text](img/49.logged-in-db-connection-verified.png)

![alt text](img/49.memcache-working.png)

![alt text](img/49.rabbitmq-working.png)

### Phase 11: Cleanup

Delete in this order (less pain, fewer dependency errors):

#### 1. CloudFront
1. Disable the distribution
2. Wait until status changes (fully disabled)
3. Delete the distribution

#### 2. DNS Record
- Remove the CloudFront CNAME in GoDaddy / Route 53

#### 3. RDS
1. Delete the DB instance  
2. Skip final snapshot (for project cleanup)
3. Disable retention / automated backups

#### 4. ElastiCache
- Delete the cluster

#### 5. Amazon MQ
- Delete the broker

#### 6. Security Groups
1. Remove the inbound rule that allows **Beanstalk SG → Backend SG**
2. Delete the backend SG (only if it’s not needed)

#### 7. Elastic Beanstalk
- Terminate the environment

#### 8. Key Pair
- Optional: delete the key pair (only if you won’t reuse it)

---

### 8. Troubleshooting Tips

#### 1. App not loading after deploy
- Check **Elastic Beanstalk → Events**
- Verify health check path is **`/login`**
- Confirm backend SG allows Beanstalk EC2 SG on required ports

#### 2. DB connection fails
- Confirm DB name is **`accounts`**
- Confirm RDS endpoint + credentials are correct
- Confirm port **3306** is allowed from Beanstalk SG

#### 3. RabbitMQ connection fails
- Double-check port (in this setup it’s often **5671**, not 5672)
- Confirm credentials match what you created in Amazon MQ

### 4. Sticky session issues
- Ensure stickiness is enabled on the **ALB target group / Beanstalk process**

#### 5. CloudFront shows old content
- Clear browser cache
- Invalidate CloudFront cache (optional)
- Confirm CloudFront origin is the **ALB DNS name**