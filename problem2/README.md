# Solution to problem 2:

First case (500 requests/sec, p99 < 100ms):
In regards to the cloud infra architecture to provision for the bitcoin trading platforms (which are similar to Binance trading platform), the diagram shown below should be sufficient to accommodate this requirement:

For the first case described below is the infrastructure description:
(as technical assumption, all AWS resources shall be provisioned in the AWS Singapore region)

![1st_case](https://github.com/user-attachments/assets/9833c365-5a14-48e9-9f19-5d6f2d6c3511), 

Incoming requests from either users or external systems will reach the AWS Route 53
AWS Route 53 is the DNS manager. With proper configuration, it can yield user-friendly domain names that can be invoked in a comfortable manner by users or external systems.
After that, traffic will reach the CloudFront. CloudFront is the AWS resource for content delivery network (CDN). It is also called the “edge” resource. CloudFront caches static content (images, JS, CSS) at edge locations close to users. This reduces latency because clients fetch static assets from the nearest AWS edge location rather than your origin (ALB/ECS)
For security purposes, it is backed by AWS web application firewall (WAF). WAF at CloudFront absorbs cybersecurity attacks before reaching the next contact point (ALB)
Traffic will later be forwarded to the application load balancer (ALB). This will evenly divide the incoming workloads to its backend target group.
The backend target group in this case is the ECS Task backed by EC2 instances as the compute engines. 
For this scenario (500 requests/sec and p99 < 100ms), 1 ECS with 1EC2 node of type c7i.2xlarge (8 vCPU, 16GB RAM) should be sufficient to accommodate the workloads.
But for the purpose of high availability, it is recommended to distribute the resources to at least 2 availability zones. This is to anticipate when an AZ suddenly goes down, the other resources in the other AZ can compensate immediately.
The Elasticache (Redis) acts as the caching storage necessary to support the bitcoin trading computations. Similar to ECS/EC2 resources, this Elasticache already provisions multi-AZ to compensate for any sudden downtime of the AZ (should there be any).
The same principle applies for RDS Postgresql and DynamoDB tenants.
The RDS (postgresql) is provisioned to store any relational data necessary for bitcoin trading computations (for example: users, KYC, configurations and settings, etc.)
The DynamoDB database is intentionally provisioned to store any trading computations data (bitcoin orders, tradings, financial ledgers, matching orders, etc.). This database serves as a time-series database necessary for continuous high-volume business transactions like core financial trading or bitcoins trading.


Second case (more and more requests coming in):
In regards to the cloud infra architecture to provision for the bitcoin trading platforms (which are similar to Binance trading platform), the diagram shown below should be sufficient to accommodate this further requirement:

For the second case, described below is the infrastructure description:
(again, as technical assumption, all AWS resources shall be provisioned in the AWS Singapore region)

![2nd_case](https://github.com/user-attachments/assets/88a8224a-ae1f-4064-8f67-c0d7e2030a3e)

Incoming requests from either users or external systems will reach the AWS Route 53
AWS Route 53 is the DNS manager. With proper configuration, it can yield user-friendly domain names that can be invoked in a comfortable manner by users or external systems.
After that, traffic will reach the CloudFront. CloudFront is the AWS resource for content delivery network (CDN). It is also called the “edge” resource. CloudFront caches static content (images, JS, CSS) at edge locations close to users. This reduces latency because clients fetch static assets from the nearest AWS edge location rather than your origin (ALB/ECS)
For security purposes, it is backed by AWS web application firewall (WAF). WAF at CloudFront absorbs cybersecurity attacks before reaching the next contact point (ALB)
Traffic will later be forwarded to the application load balancer (ALB). This will evenly divide the incoming workloads to its backend target group.
The backend target group in this case is the ECS Task backed by EC2 instances as the compute engines. 
For this scenario (high volumes of incoming requests), multiple nodes of ECS with multiple nodes of EC2 with type c7i.2xlarge (8 vCPU, 16GB RAM) should be sufficient to accommodate the workloads.
Alternatively, elastic kubernetes services (EKS) may also be provisioned to cover much higher incoming workloads.
But for the purpose of high availability, it is recommended to distribute the resources to at least 2 availability zones. This is to anticipate when an AZ suddenly goes down, the other resources in the other AZ can compensate immediately.
For more and more requests coming in, it is strongly recommended to implement messaging services (for example: Kafka or Amazon Managed Services for Kafka (MSK)).  This acts as the backpressure system against the high volumes of incoming traffic.
Using Kafka, incoming messages may be transformed, processed and stored into multiple database backends (RDS, NoSQL/DynamoDB, etc.)
The Elasticache (Redis) acts as the caching storage necessary to support the bitcoin trading computations. Similar to ECS/EC2 resources, this Elasticache already provisions multi-AZ to compensate for any sudden downtime of the AZ (should there be any).
The same principle applies for RDS Postgresql and DynamoDB tenants.
The RDS (postgresql) is provisioned to store any relational data necessary for bitcoin trading computations (for example: users, KYC, configurations and settings, etc.)
The DynamoDB database is intentionally provisioned to store any trading computations data (bitcoin orders, tradings, financial ledgers, matching orders, etc.). This database serves as a time-series database necessary for continuous high-volume business transactions like core financial trading or bitcoins trading.
