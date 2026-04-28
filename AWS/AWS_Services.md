# ☁️ AWS Services – Complete Reference Guide

> **Date:** 15/04/26 | **Topic:** Top 10–12 AWS Services for Interviews & Real Projects

---

## Table of Contents

1. [Amazon EC2](#1-amazon-ec2-elastic-compute-cloud)
2. [AWS Lambda](#2-aws-lambda)
3. [Amazon S3](#3-amazon-s3-simple-storage-service)
4. [Amazon RDS](#4-amazon-rds-relational-database-service)
5. [Amazon DynamoDB](#5-amazon-dynamodb)
6. [Amazon API Gateway](#6-amazon-api-gateway)
7. [Amazon CloudFront](#7-amazon-cloudfront)
8. [Amazon VPC](#8-amazon-vpc-virtual-private-cloud)
9. [AWS IAM](#9-aws-iam-identity-and-access-management)
10. [Amazon SQS](#10-amazon-sqs-simple-queue-service)
11. [Amazon SNS](#11-amazon-sns-simple-notification-service)
12. [Amazon CloudWatch](#12-amazon-cloudwatch)

---

## 1. Amazon EC2 (Elastic Compute Cloud)

> **Type:** Stateful Compute | **Category:** Infrastructure

### 🔍 What is it?
EC2 provides virtual machines (instances) in the cloud. You get full control over the OS, software, and configuration — just like a physical server but hosted on AWS.

### 📌 Key Concepts
| Concept | Description |
|---|---|
| **Instance Types** | t2 (small/dev), t3 (general), m5 (production) |
| **Security Groups** | Firewall rules — control who can access the server |
| **Key Pairs** | SSH login using private key instead of password |
| **Auto Scaling** | Automatically adds/removes instances under high load |

### ✅ When to Use EC2
- You need a **long-running server** (backend API, game server, cron jobs)
- You need **full OS-level control** (custom software, specific libraries)
- Your app is **heavy on computation** (video processing, ML training)
- You need to run **containers** or custom runtime environments
- Your app is **always running** and you want to pay for uptime

### ❌ When NOT to Use EC2
- For short, event-triggered tasks → Use **Lambda**
- For simple static websites → Use **S3 + CloudFront**
- You don't want to manage servers → Use **Lambda or ECS/Fargate**

### ⚡ EC2 vs Lambda (Quick Comparison)
| EC2 | Lambda |
|---|---|
| Long-running servers | Short tasks, event-driven |
| Full control | No server management |
| Good for heavy backend | Good for APIs, triggers |
| Always running | Runs only when called |
| Pay for uptime | Pay per execution |

### 🔗 How to Connect with Node.js (Concept)
1. Launch an EC2 instance (Ubuntu/Amazon Linux)
2. Open port 22 (SSH) and port 3000/80 in Security Group
3. SSH into the server using your key pair
4. Install Node.js on the instance
5. Upload your app (via SCP, Git, or CodeDeploy)
6. Run your Node server (use PM2 to keep it alive)
7. Point your domain/load balancer to the instance's public IP

```
Flow: Your Code → SSH → EC2 Instance → Node.js App Runs
```

### 🔄 Alternatives
| Alternative | Use Case |
|---|---|
| **AWS Fargate / ECS** | Run containers without managing EC2 |
| **AWS Lambda** | Serverless, event-driven compute |
| **Google Compute Engine** | GCP equivalent |
| **Azure Virtual Machines** | Azure equivalent |
| **Railway / Render / Heroku** | Managed PaaS, simpler deployments |

---

## 2. AWS Lambda

> **Type:** Serverless Compute | **Category:** Function-as-a-Service

### 🔍 What is it?
Lambda lets you run backend code **without managing any servers**. You just upload your function, and AWS runs it in response to events. It is stateless and short-lived.

### 📌 Key Concepts
| Concept | Description |
|---|---|
| **Event-Driven** | Runs only when triggered by an event |
| **Triggers** | API Gateway, S3, SQS, DynamoDB Streams, CloudWatch |
| **Stateless** | No memory between invocations |
| **Cold Starts** | First invocation may be slow (container spin-up) |

### ✅ When to Use Lambda
- Handling **API requests** via API Gateway
- Processing **S3 uploads** (resize images, parse files)
- Consuming **SQS messages** in the background
- Running **scheduled jobs** (cron-like with CloudWatch Events)
- Building **microservices** without infrastructure management
- Any task that is **short, fast, and event-triggered**

### ❌ When NOT to Use Lambda
- Long-running tasks (Lambda max timeout = 15 minutes)
- Apps that need persistent connections (WebSockets, long polling)
- Heavy computation needing full OS access → Use EC2

### 🔗 How to Connect with Node.js (Concept)
1. Write your function as a Node.js handler
2. Deploy via AWS Console, CLI (`aws lambda`), or frameworks like **Serverless Framework** or **AWS SAM**
3. Set a trigger (e.g., API Gateway for HTTP, S3 for file upload)
4. Lambda runs your function and returns a response

```
Flow: Event (HTTP/S3/SQS) → Lambda Trigger → Node.js Handler Runs → Response
```

### 🔄 Alternatives
| Alternative | Use Case |
|---|---|
| **Google Cloud Functions** | GCP serverless |
| **Azure Functions** | Azure serverless |
| **Cloudflare Workers** | Edge serverless, ultra-low latency |
| **Vercel / Netlify Functions** | Frontend-focused serverless |

---

## 3. Amazon S3 (Simple Storage Service)

> **Type:** Object Storage | **Category:** Storage

### 🔍 What is it?
S3 is AWS's file storage service. Every application uses it — for images, videos, documents, backups, static websites, and more. Files are stored as **objects** inside **buckets**.

### 📌 Key Concepts
| Concept | Description |
|---|---|
| **Buckets** | Containers for your files (like a folder) |
| **Objects** | Individual files stored in buckets |
| **Public vs Private** | Control whether files are accessible publicly or privately |
| **Pre-signed URLs** | Temporary links to private files (expire after a set time) |
| **Static Hosting** | Serve HTML/CSS/JS directly from S3 |

### ✅ When to Use S3
- Storing **user-uploaded files** (profile pictures, documents)
- Serving **static frontend** (React, Vue build files)
- Storing **backups** and logs
- Sharing files via **temporary pre-signed URLs**
- Storing **data for ML** pipelines or ETL jobs
- Archiving large amounts of data cheaply

### 🔗 How to Connect with Node.js (Concept)
1. Install `@aws-sdk/client-s3` (AWS SDK v3)
2. Configure AWS credentials (via IAM role or environment variables)
3. Use SDK methods to upload, download, list, or delete files
4. For private files, generate pre-signed URLs for temporary access

```
Flow: Node App → AWS SDK → S3 API → Bucket/Object
```

### 🔄 Alternatives
| Alternative | Use Case |
|---|---|
| **Google Cloud Storage** | GCP equivalent |
| **Azure Blob Storage** | Azure equivalent |
| **Cloudflare R2** | S3-compatible, no egress fees |
| **Backblaze B2** | Cheaper S3 alternative |
| **Supabase Storage** | S3-backed, developer-friendly |

---

## 4. Amazon RDS (Relational Database Service)

> **Type:** Managed SQL Database | **Category:** Database

### 🔍 What is it?
RDS is a fully managed **relational database** service. AWS handles backups, patches, and failover for you. Supports MySQL, PostgreSQL, MariaDB, Oracle, and SQL Server.

### 📌 Key Concepts
| Concept | Description |
|---|---|
| **Multi-AZ** | Standby instance in another zone for failover |
| **Read Replicas** | Extra read-only copies to scale read traffic |
| **Automated Backups** | Point-in-time recovery included |
| **Managed** | AWS handles OS patching, upgrades |

### ✅ When to Use RDS
- Your app needs a **structured relational database** (SQL)
- You want **managed backups and failover** without manual ops
- Building apps with **complex joins and transactions**
- When data integrity is critical (e-commerce, finance, healthcare)

### ❌ When NOT to Use RDS
- Massive scale with unpredictable traffic → Use **DynamoDB**
- Simple key-value lookups at high speed → Use **DynamoDB** or **ElastiCache**

### 🔗 How to Connect with Node.js (Concept)
1. Create RDS instance in a **private subnet** (VPC)
2. Allow inbound traffic from your EC2/Lambda Security Group on port 5432 (Postgres) or 3306 (MySQL)
3. Use connection libraries like `pg` (PostgreSQL) or `mysql2` (MySQL) or an ORM like **Prisma / Sequelize**
4. Store DB credentials in **AWS Secrets Manager** (never hardcode)

```
Flow: Node App → pg/mysql2/Prisma → RDS Endpoint → Database
```

### 🔄 Alternatives
| Alternative | Use Case |
|---|---|
| **DynamoDB** | NoSQL, massive scale |
| **PlanetScale** | Serverless MySQL |
| **Neon / Supabase** | Serverless PostgreSQL |
| **Google Cloud SQL** | GCP managed SQL |
| **Azure Database** | Azure managed SQL |

---

## 5. Amazon DynamoDB

> **Type:** NoSQL Database | **Category:** Database

### 🔍 What is it?
DynamoDB is AWS's fully managed **NoSQL** key-value and document database. It's built for **massive scale** and **single-digit millisecond latency** at any traffic volume.

### 📌 Key Concepts
| Concept | Description |
|---|---|
| **Key-Based Access** | Data is accessed by Partition Key (and optionally Sort Key) |
| **Ultra-Low Latency** | Consistent performance at any scale |
| **Auto Scaling** | Automatically adjusts capacity based on traffic |
| **No Schema** | Each item can have different attributes |

### ✅ When to Use DynamoDB
- Massive scale with **unpredictable or spiky traffic**
- Simple **key-value lookups** (user sessions, cart data)
- **Real-time apps** (leaderboards, gaming, IoT)
- Apps that need to scale to **millions of requests/sec**
- When you don't need complex joins or SQL queries

### ❌ When NOT to Use DynamoDB
- Complex queries with joins → Use **RDS**
- Analytics and reporting → Use **Redshift** or **Athena**

### 🔗 How to Connect with Node.js (Concept)
1. Install `@aws-sdk/client-dynamodb` or `@aws-sdk/lib-dynamodb`
2. Use IAM role or credentials for authentication
3. Perform operations: `PutItem`, `GetItem`, `Query`, `Scan`, `DeleteItem`
4. Use **DynamoDB DocumentClient** for easier JSON-style access

```
Flow: Node App → AWS SDK → DynamoDB API → Table → Item
```

### 🔄 Alternatives
| Alternative | Use Case |
|---|---|
| **MongoDB / Atlas** | Flexible document store |
| **Firebase Firestore** | Real-time NoSQL for mobile/web |
| **Redis / ElastiCache** | In-memory key-value, ultra-fast |
| **Cassandra** | Open-source wide-column store |

---

## 6. Amazon API Gateway

> **Type:** API Management | **Category:** Networking / API Layer

### 🔍 What is it?
API Gateway is the **front door** to your backend. It receives HTTP requests from clients (React, mobile) and routes them to your backend (Lambda, EC2, or other services). Think of it as a managed reverse proxy + API manager.

### 📌 Key Concepts
| Concept | Description |
|---|---|
| **REST/HTTP APIs** | Define routes and methods (GET, POST, etc.) |
| **Authentication** | Built-in JWT auth, Cognito, Lambda authorizers |
| **Rate Limiting** | Throttle requests per user/API key |
| **Throttling & Protection** | Prevent abuse, DDoS protection |

### ✅ When to Use API Gateway
- You're building a **serverless backend** (Lambda + API Gateway)
- You need a **managed API layer** with auth, rate limiting, and throttling
- Building **microservices** that need a single entry point
- You want to **version your API** easily

### Typical Flow
```
React App → API Gateway → Lambda → DynamoDB/RDS
```

### 🔗 How to Connect with Node.js (Concept)
1. Create an API in API Gateway console
2. Define routes (e.g., `GET /users`, `POST /orders`)
3. Set integration target → Lambda function or EC2 endpoint
4. Enable API key / Cognito auth if needed
5. Deploy to a stage (dev/prod) — you get a public URL
6. Call this URL from your Node.js frontend or other services

```
Flow: HTTP Request → API Gateway URL → Lambda/EC2 → Response
```

### 🔄 Alternatives
| Alternative | Use Case |
|---|---|
| **AWS App Runner** | Simpler container-based API deployment |
| **Kong / Nginx** | Self-hosted API gateway |
| **Apigee (Google)** | Enterprise API management |
| **Cloudflare Workers** | Edge API gateway |
| **Express.js on EC2** | Simple self-managed API |

---

## 7. Amazon CloudFront

> **Type:** CDN (Content Delivery Network) | **Category:** Networking / Performance

### 🔍 What is it?
CloudFront is AWS's **global CDN**. It caches your content (images, HTML, JS, videos) at **edge locations** around the world, making your app fast for users regardless of where they are.

### 📌 Key Concepts
| Concept | Description |
|---|---|
| **Edge Locations** | 400+ global points of presence to cache content |
| **Origin** | Your source (S3 bucket, EC2, ALB, API Gateway) |
| **Caching** | Static content served from nearest edge, not origin |
| **HTTPS** | Free SSL/TLS certificates via ACM |

### ✅ When to Use CloudFront
- You want your **static frontend** (React/Next.js build) to load fast globally
- Serving **large files** (videos, images) at scale
- Reducing load on your origin server
- Adding **HTTPS and custom domain** to S3-hosted sites
- Protecting against basic DDoS via AWS Shield

### Common Setup
```
User in India → CloudFront Edge (Mumbai) → Cached S3 Content
                                          ↗ (Cache miss) → S3 Origin (us-east-1)
```

### 🔗 How to Connect with Node.js (Concept)
1. Upload your built frontend to **S3**
2. Create a CloudFront distribution pointing to the S3 bucket
3. Set cache behaviors (cache JS/CSS aggressively, never cache HTML)
4. Attach an **ACM SSL certificate** for HTTPS
5. Point your domain's DNS (Route 53) to the CloudFront URL
6. Use `aws cloudfront create-invalidation` to clear cache after deploy

```
Flow: Deploy → S3 Upload → CloudFront Invalidation → Users Get Fresh Content
```

### 🔄 Alternatives
| Alternative | Use Case |
|---|---|
| **Cloudflare CDN** | Popular, easy-to-use CDN with free tier |
| **Fastly** | Developer-friendly CDN |
| **Akamai** | Enterprise CDN |
| **Vercel / Netlify** | Automatic CDN + deployment for frontend |
| **Azure CDN / GCP CDN** | Cloud-native alternatives |

---

## 8. Amazon VPC (Virtual Private Cloud)

> **Type:** Networking | **Category:** Infrastructure / Security

### 🔍 What is it?
VPC is your **private, isolated network** inside AWS. Every resource you create (EC2, RDS, Lambda) lives inside a VPC. It gives you full control over IP ranges, routing, and access.

### 📌 Key Concepts
| Concept | Description |
|---|---|
| **Public Subnet** | Resources accessible from the internet (EC2, ALB) |
| **Private Subnet** | Resources NOT accessible from internet (RDS, internal services) |
| **Internet Gateway (IGW)** | Allows public subnet to connect to the internet |
| **NAT Gateway** | Allows private subnet to make outbound internet calls (e.g., for software updates) |
| **Security Groups** | Instance-level firewall (control inbound/outbound traffic) |

### ✅ When to Configure VPC
- Every production app should use a properly configured VPC
- When you need to isolate your database from public internet
- When multiple services need to communicate **privately** (without public IPs)
- For compliance (HIPAA, PCI-DSS) requiring network isolation

### Typical Architecture
```
Internet
   ↓
Internet Gateway (IGW) → Public Subnet → EC2 (Backend)
                          Private Subnet → RDS (Database)
NAT Gateway → Private Subnet → Outbound internet (for updates)
```

### 🔗 How to Connect with Node.js (Concept)
- VPC is infrastructure-level; your Node.js app doesn't directly use VPC
- Ensure your EC2/Lambda is in the **right subnet** with correct security group rules
- For RDS: ensure your app's security group is allowed to access RDS port
- Use **VPC Endpoints** to access S3/DynamoDB privately without going through internet

```
Flow: Node App (EC2 in Public Subnet) → Security Group → RDS (Private Subnet)
```

### 🔄 Alternatives
| Alternative | Use Case |
|---|---|
| **Google VPC** | GCP equivalent |
| **Azure VNet** | Azure equivalent |
| **Tailscale / WireGuard** | Simpler private networking for small teams |

---

## 9. AWS IAM (Identity and Access Management)

> **Type:** Security & Permissions | **Category:** Security

### 🔍 What is it?
IAM controls **who can do what on which AWS service**. It's the security backbone of every AWS account. Without IAM, everything is either fully open or fully closed.

### 📌 Key Concepts
| Concept | Description |
|---|---|
| **User** | A person (you, a developer, an admin) |
| **Role** | A permission set assumed by a service (Lambda, EC2) |
| **Policy** | JSON document defining what is allowed/denied |
| **Least Privilege** | Give only the minimum permissions needed |

### ✅ When to Use IAM
- **Always** — every AWS resource interaction requires IAM
- Give your Lambda function a role to access S3 or DynamoDB
- Give developers limited access (e.g., only S3 read, no EC2 access)
- Cross-account access between AWS accounts

### Example Policy Concept
```
"Allow Lambda to read from S3 bucket named 'my-app-uploads'"
→ Create IAM Role → Attach S3:GetObject Policy → Assign Role to Lambda
```

### 🔗 How to Connect with Node.js (Concept)
1. Create an **IAM Role** with required permissions (e.g., S3 access)
2. Attach the role to your EC2 instance or Lambda function
3. AWS SDK automatically uses this role — no credentials needed in code
4. For local development, use `aws configure` with IAM user credentials
5. **Never hardcode AWS keys** in your code — use roles or environment variables

```
Flow: Node App → AWS SDK → IAM Role Auth → Allowed Service (S3/DynamoDB/etc)
```

### 🔄 Alternatives
| Alternative | Use Case |
|---|---|
| **AWS Organizations SCP** | Org-level permission boundaries |
| **Google Cloud IAM** | GCP equivalent |
| **Azure RBAC** | Azure role-based access |
| **HashiCorp Vault** | Secrets + access management for multi-cloud |

---

## 10. Amazon SQS (Simple Queue Service)

> **Type:** Message Queue | **Category:** Messaging / Decoupling

### 🔍 What is it?
SQS is a **managed message queue** for decoupling services. A producer sends messages to the queue, and consumers process them independently. This prevents one slow service from blocking another.

### 📌 Key Concepts
| Concept | Description |
|---|---|
| **Producer/Consumer** | Producer sends messages; Consumer reads and processes |
| **Visibility Timeout** | Message hidden from other consumers while being processed |
| **Retry Handling** | Built-in retries if processing fails |
| **Dead Letter Queue (DLQ)** | Messages that keep failing go here for inspection |

### ✅ When to Use SQS
- **Decoupling microservices** (Order service sends to queue, Inventory service reads)
- **Background jobs** (send email, resize image, generate PDF)
- **Handling traffic spikes** — queue absorbs burst, consumers process at their pace
- Ensuring **at-least-once delivery** for critical tasks
- When you need **retry logic** for failed operations

### ❌ When NOT to Use SQS
- For real-time pub/sub broadcasting → Use **SNS**
- For immediate response APIs → Use API Gateway + Lambda directly

### 🔗 How to Connect with Node.js (Concept)
1. Create an SQS queue (Standard or FIFO)
2. Use `@aws-sdk/client-sqs` in Node.js
3. Producer: Call `SendMessage` to add a job to the queue
4. Consumer: Poll with `ReceiveMessage`, process, then call `DeleteMessage`
5. Alternatively, trigger a **Lambda** automatically when a message arrives

```
Flow: Order Service → SQS Queue → Lambda/Worker → Process Order → Delete Message
```

### 🔄 Alternatives
| Alternative | Use Case |
|---|---|
| **AWS SNS** | Fan-out/broadcast messaging |
| **RabbitMQ** | Open-source message broker |
| **Apache Kafka** | High-throughput event streaming |
| **Google Pub/Sub** | GCP equivalent |
| **BullMQ / Bull** | Redis-based job queue for Node.js |

---

## 11. Amazon SNS (Simple Notification Service)

> **Type:** Pub/Sub Messaging | **Category:** Messaging / Events

### 🔍 What is it?
SNS is a **publish/subscribe** messaging service. One event can be **broadcast to many subscribers** simultaneously. It's perfect for fan-out patterns where one action needs to notify multiple systems.

### 📌 Key Concepts
| Concept | Description |
|---|---|
| **Topic** | A named channel for publishing messages |
| **Publisher** | The service that sends an event |
| **Subscriber** | Services that receive the event (SQS, Lambda, Email, HTTP) |
| **Fan-Out** | One message → many consumers at the same time |

### ✅ When to Use SNS
- **Fan-out notifications** (one event, many listeners)
- Sending **emails or SMS** to users (transactional notifications)
- Triggering **multiple SQS queues** from a single event
- **Event-driven microservices** architecture
- Mobile **push notifications** (iOS, Android)

### Common Flow
```
Order Placed
     ↓
  SNS Topic
 ↙    ↓    ↘
SQS   SQS   Email
(Inventory)(Shipping)(Customer)
```

### 🔗 How to Connect with Node.js (Concept)
1. Create an SNS Topic
2. Add subscriptions (SQS queues, Lambda functions, email endpoints)
3. Use `@aws-sdk/client-sns` to publish a message from Node.js
4. Call `Publish` with your topic ARN and message payload
5. All subscribers receive the message simultaneously

```
Flow: Node App → SNS Publish → Topic → Multiple Subscribers Notified
```

### 🔄 Alternatives
| Alternative | Use Case |
|---|---|
| **SQS** | Single consumer queue (not broadcast) |
| **Apache Kafka** | High-throughput event streaming |
| **Google Pub/Sub** | GCP equivalent |
| **Firebase Cloud Messaging** | Mobile push notifications |
| **Twilio / SendGrid** | Email/SMS as managed service |

---

## 12. Amazon CloudWatch

> **Type:** Monitoring & Observability | **Category:** Operations

### 🔍 What is it?
CloudWatch is AWS's **monitoring and observability** platform. It collects logs, metrics, and events from your AWS resources and lets you set alarms and dashboards to stay on top of your system's health.

### 📌 Key Concepts
| Concept | Description |
|---|---|
| **Logs** | EC2, Lambda, and application logs stored and searchable |
| **Metrics** | CPU usage, memory, error rates, latency, custom metrics |
| **Alarms** | Set thresholds and trigger actions when breached |
| **Dashboards** | Visual graphs of your metrics over time |

### ✅ When to Use CloudWatch
- Monitoring **EC2 CPU, memory, and disk** usage
- Viewing **Lambda invocation errors** and durations
- Setting **alarms** to auto-trigger scaling or notifications
- Debugging production issues via **centralized logs**
- Tracking **custom business metrics** (orders/sec, failed logins)

### Example Alarm Setup
```
CPU Usage > 80% for 5 minutes
        ↓
  CloudWatch Alarm Triggered
        ↓
   SNS Notification → Email/Slack Alert
        ↓
   Auto Scaling → Add EC2 Instance
```

### 🔗 How to Connect with Node.js (Concept)
1. **Logs:** EC2 and Lambda automatically send logs to CloudWatch. Use `aws-sdk` or `winston-cloudwatch` to send custom app logs
2. **Custom Metrics:** Use `PutMetricData` API to send business metrics
3. **Alarms:** Set up in Console or via SDK/Terraform — no Node.js code needed
4. **Log Insights:** Query logs with SQL-like syntax directly in CloudWatch console

```
Flow: Node App → CloudWatch SDK → Log Group → CloudWatch Dashboard/Alarm
```

### 🔄 Alternatives
| Alternative | Use Case |
|---|---|
| **Datadog** | Full-stack monitoring, great UI, expensive |
| **New Relic** | APM + infrastructure monitoring |
| **Grafana + Prometheus** | Open-source monitoring stack |
| **Sentry** | Error tracking for frontend/backend |
| **ELK Stack** | Open-source logging (Elasticsearch, Logstash, Kibana) |
| **Google Cloud Monitoring** | GCP equivalent |

---

## 🏗️ How These Services Work Together

Here's a real-world app architecture using all 12 services:

```
User (Browser/Mobile)
        ↓
  CloudFront (CDN) ← S3 (Static Frontend)
        ↓
  API Gateway (API Layer)
        ↓
  Lambda (Business Logic) ← IAM (Permissions)
     ↙        ↘
  RDS       DynamoDB
(SQL DB)   (NoSQL DB)
     ↓
   SQS Queue (Decoupled Jobs)
     ↓
  Lambda Worker → SNS (Notifications)
        ↓
  CloudWatch (Monitoring + Alarms)
        ↓
  VPC (Everything lives inside this private network)
  EC2 (For any long-running processes)
```

---

## 📋 Quick Reference – When to Use What

| Need | Use |
|---|---|
| Virtual server, full control | EC2 |
| Serverless function | Lambda |
| File/image storage | S3 |
| SQL relational database | RDS |
| NoSQL, massive scale | DynamoDB |
| HTTP API routing | API Gateway |
| Global CDN, fast delivery | CloudFront |
| Private network isolation | VPC |
| Permissions and access | IAM |
| Async job queue | SQS |
| Broadcast notifications | SNS |
| Logs, metrics, alerts | CloudWatch |

---

*Generated from handwritten notes dated 15/04/26 | AWS Services Interview Prep*
