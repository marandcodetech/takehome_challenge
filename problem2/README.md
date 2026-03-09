# Solution for problem no.2

# First case (500 requests/sec, p99 < 100ms):
In regards to the cloud infra architecture to provision for the bitcoin trading platforms (which are similar to Binance trading platform), the diagram shown below should be sufficient to accommodate this requirement:

For the first case described below is the infrastructure description:
(as technical assumption, all AWS resources shall be provisioned in the AWS Singapore region)

![1st_case](https://github.com/user-attachments/assets/9833c365-5a14-48e9-9f19-5d6f2d6c3511), 

# AWS Architecture Flow for Bitcoin Trading Platform (first case)

## Request Flow Overview

Incoming requests from **users or external systems** follow the architecture below.

---

## 1. DNS Layer – AWS Route 53

Requests first reach **AWS Route 53**, which acts as the **DNS manager**.

Its responsibilities include:

- Mapping domain names to infrastructure endpoints
- Providing **user-friendly domain names**
- Allowing external systems and users to access services easily

Example:

```
api.example.com
```

Route 53 resolves the domain and forwards traffic to the next layer.

---

## 2. Edge Layer – Amazon CloudFront (CDN)

After DNS resolution, traffic reaches **Amazon CloudFront**.

CloudFront is AWS's **Content Delivery Network (CDN)** and serves as the **edge layer**.

### Responsibilities

- Caches **static assets** such as:
  - Images
  - JavaScript files
  - CSS files
- Stores these assets in **edge locations close to users**

### Benefits

- Lower latency
- Faster page load time
- Reduced load on origin servers (ALB / ECS)

Instead of contacting the backend directly, users retrieve static content from the **nearest AWS edge location**.

---

## 3. Security Layer – AWS WAF

CloudFront is protected by **AWS Web Application Firewall (WAF)**.

WAF provides:

- Protection against common **web exploits**
- Filtering of malicious requests
- Blocking of suspicious IP traffic
- Mitigation of bot attacks

This layer **absorbs cyber attacks before they reach the backend infrastructure**.

---

## 4. Load Balancing – Application Load Balancer (ALB)

Validated traffic is then forwarded to the **Application Load Balancer (ALB)**.

The ALB performs:

- **Traffic distribution**
- **Health checks**
- **Layer 7 routing**

Requests are evenly distributed across the **backend target group**.

---

## 5. Compute Layer – ECS with EC2

The backend target group consists of **ECS tasks running on EC2 instances**.

### Example Capacity

For the workload:

```
500 requests / second
p99 latency < 100 ms
```

A possible configuration:

```
1 ECS cluster
1 EC2 instance
Instance type: c7i.2xlarge
8 vCPU
16 GB RAM
```

This setup should be capable of handling the workload.

---

## High Availability Design

For production systems, it is recommended to deploy resources across **multiple Availability Zones (AZs)**.

Example design:

```
AZ-1
 └ ECS Task
 └ EC2 Instance

AZ-2
 └ ECS Task
 └ EC2 Instance
```

### Benefits

- Protects against **AZ-level failures**
- Ensures **service continuity**
- Allows automatic load redistribution if one AZ fails

---

## 6. Caching Layer – ElastiCache (Redis)

**Amazon ElastiCache (Redis)** provides the caching layer required for **high-speed bitcoin trading computations**.

### Role

- Stores frequently accessed data
- Reduces database load
- Accelerates trading calculations

### High Availability

ElastiCache is deployed with **Multi-AZ support**, allowing automatic failover if a node or AZ fails.

---

## 7. Relational Storage – Amazon RDS (PostgreSQL)

**Amazon RDS PostgreSQL** stores **relational data** required by the system.

Typical data stored includes:

- Users
- KYC records
- System configurations
- Platform settings

This database ensures **data consistency and relational integrity**.

---

## 8. Trading Data Storage – DynamoDB

**Amazon DynamoDB** is used to store **high-volume trading transaction data**.

Examples of stored records:

- Bitcoin orders
- Trade executions
- Financial ledgers
- Order matching records

### Why DynamoDB?

Trading systems require:

- Extremely high throughput
- Low latency
- Horizontal scalability

DynamoDB behaves effectively as a **time-series datastore** capable of handling continuous high-frequency financial transactions.

---

# Final Architecture Flow

```
Client / External Systems
        │
        ▼
AWS Route 53 (DNS)
        │
        ▼
CloudFront (CDN + Edge Caching)
        │
        ▼
AWS WAF (Security Protection)
        │
        ▼
Application Load Balancer (Traffic Distribution)
        │
        ▼
ECS Tasks running on EC2 (Compute Layer)
        │
        ├── ElastiCache Redis (Caching Layer)
        │
        ├── RDS PostgreSQL (Relational Data)
        │
        └── DynamoDB (High-volume Trading Data)
```

This architecture provides:

- **Low latency**
- **High scalability**
- **High availability**
- **Security protection**
- **Optimized storage for trading workloads**


# Second case (more and more requests coming in):
In regards to the cloud infra architecture to provision for the bitcoin trading platforms (which are similar to Binance trading platform), the diagram shown below should be sufficient to accommodate this further requirement:

For the second case, described below is the infrastructure description:
(again, as technical assumption, all AWS resources shall be provisioned in the AWS Singapore region)

![2nd_case](https://github.com/user-attachments/assets/88a8224a-ae1f-4064-8f67-c0d7e2030a3e)


# AWS Architecture Flow for High-Volume Bitcoin Trading Platform (second case)

## Request Flow Overview

Incoming requests from **users or external systems** follow the architecture path described below.

---

## 1. DNS Layer – AWS Route 53

Incoming traffic first reaches **AWS Route 53**, which serves as the **DNS manager**.

Its main responsibilities include:

- Mapping domain names to infrastructure endpoints  
- Providing **user-friendly domain names**  
- Allowing users or external systems to access services easily  

Example:

```
api.example.com
```

Route 53 resolves the domain and directs traffic to the next layer.

---

## 2. Edge Layer – Amazon CloudFront (CDN)

After DNS resolution, traffic is routed to **Amazon CloudFront**.

CloudFront is AWS’s **Content Delivery Network (CDN)** and functions as the **edge layer** of the architecture.

### Responsibilities

CloudFront caches static assets such as:

- Images  
- JavaScript files  
- CSS files  

These assets are stored in **edge locations close to users**.

### Benefits

- Reduced latency  
- Faster response times  
- Lower load on backend services  

Users retrieve static content from the **nearest edge location** rather than contacting the backend infrastructure directly.

---

## 3. Security Layer – AWS Web Application Firewall (WAF)

CloudFront is protected by **AWS WAF (Web Application Firewall)**.

WAF provides:

- Protection against common web exploits  
- Filtering of malicious traffic  
- Bot detection and mitigation  
- Blocking suspicious IP addresses  

This layer absorbs potential **cybersecurity attacks before they reach the backend infrastructure**, specifically before traffic reaches the Application Load Balancer.

---

## 4. Load Balancing Layer – Application Load Balancer (ALB)

Validated traffic is forwarded to the **Application Load Balancer (ALB)**.

The ALB performs:

- Intelligent **Layer 7 traffic routing**
- **Health checks**
- **Load distribution**

Incoming workloads are evenly distributed across the **backend target group**.

---

## 5. Compute Layer – ECS with EC2

The backend target group consists of **ECS tasks running on EC2 instances**.

For high incoming request volumes, multiple nodes are recommended:

Example compute configuration:

```
ECS Cluster
 ├─ EC2 Instance (c7i.2xlarge)
 ├─ EC2 Instance (c7i.2xlarge)
 └─ EC2 Instance (c7i.2xlarge)

Instance type: c7i.2xlarge
CPU: 8 vCPU
Memory: 16 GB RAM
```

This setup allows the system to handle **high levels of concurrent requests**.

### Alternative: Kubernetes

For extremely large workloads, **Amazon Elastic Kubernetes Service (EKS)** may be used instead of ECS.

Benefits include:

- Advanced container orchestration  
- Higher scaling flexibility  
- More sophisticated workload management  

---

## 6. High Availability Strategy

To ensure resilience, infrastructure resources should be distributed across **multiple Availability Zones (AZs)**.

Example:

```
Availability Zone A
 └ ECS Tasks
 └ EC2 Instances

Availability Zone B
 └ ECS Tasks
 └ EC2 Instances
```

### Benefits

- Protects against **single-AZ failure**
- Ensures **continuous service availability**
- Allows immediate workload redistribution

---

## 7. Messaging and Backpressure Layer – Kafka / Amazon MSK

For very high traffic environments, a **messaging layer** should be introduced.

Recommended technologies:

- Apache Kafka  
- Amazon Managed Streaming for Kafka (MSK)

### Role of Messaging Layer

Kafka acts as a **backpressure system** to manage bursts of incoming requests.

Benefits:

- Decouples producers and consumers  
- Buffers high traffic spikes  
- Enables asynchronous processing  
- Improves overall system stability

Incoming events can be:

- Transformed  
- Processed  
- Routed to multiple downstream systems  

---

## 8. Caching Layer – Amazon ElastiCache (Redis)

**Amazon ElastiCache using Redis** provides high-speed caching to support bitcoin trading computations.

### Role

- Stores frequently accessed data  
- Reduces database load  
- Speeds up trading operations  

ElastiCache is deployed with **Multi-AZ support**, ensuring availability if one zone becomes unavailable.

---

## 9. Relational Storage – Amazon RDS (PostgreSQL)

**Amazon RDS PostgreSQL** stores relational data used by the trading platform.

Examples of stored data:

- User accounts  
- KYC records  
- Platform configurations  
- System settings  

RDS ensures:

- Data integrity  
- ACID transactions  
- Structured relational storage

---

## 10. Trading Data Storage – Amazon DynamoDB

**Amazon DynamoDB** stores high-volume trading and transaction data.

Examples include:

- Bitcoin orders  
- Trade executions  
- Financial ledgers  
- Order matching records  

DynamoDB is ideal for this workload because it provides:

- Extremely high throughput  
- Low latency reads and writes  
- Automatic scaling  

It effectively behaves as a **time-series datastore**, suitable for continuous financial trading workloads.

---

# Final Architecture Flow

```
Client / External Systems
        │
        ▼
AWS Route 53 (DNS)
        │
        ▼
CloudFront (CDN + Edge Caching)
        │
        ▼
AWS WAF (Security Protection)
        │
        ▼
Application Load Balancer
        │
        ▼
ECS / EC2  (or EKS)
        │
        ├── Kafka / MSK (Message Streaming Layer)
        │
        ├── ElastiCache Redis (Caching Layer)
        │
        ├── RDS PostgreSQL (Relational Data)
        │
        └── DynamoDB (High-volume Trading Data)
```

This architecture provides:

- **High scalability**
- **Low latency**
- **Fault tolerance**
- **Security protection**
- **Efficient processing for financial trading workloads**
