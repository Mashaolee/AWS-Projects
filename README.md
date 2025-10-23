#AWS Multi-AZ Windows Application Architecture

This document describes the highly available (HA) and scalable infrastructure designed to host a stateful Windows-based web application on AWS, ensuring redundancy, shared storage access, and robust security.

1. High-Level Goal
- To deploy a Windows application across two distinct AWS Availability Zones (AZs) using shared, managed storage (FSx) to achieve sub-second failover and maintain data consistency during an outage of any single component.

2. Architecture Diagram
- The overall architecture is fully redundant, spanning two Availability Zones (AZ-01 and AZ-02).

3. Components Breakdown
3.1. Route 5: Acts as the public entry point, routing user traffic via an Alias Record directly to the Application Load Balancer (ALB).
3.2. Application Load Balancer (ALB): Distributes incoming web traffic (HTTP/S) across the two EC2 instances. Performs continuous Health Checks to ensure traffic is only sent to healthy servers.
3.3 Internet Gateway (IGW): Allows the EC2 instances in the Public Subnets to connect to the public internet for updates, patches, and external AWS service communication.
3.4. EC2: The Windows Server instances running the application (e.g., IIS). Deployed in Public Subnets for ALB accessibility and configured to automatically join the Active Directory (AD) domain.
~ FSx for Windows: Provides a fully managed, highly available, and redundant network file share (SMB protocol). It is the single source of truth for application data, ensuring data consistency regardless of which EC2 instance serves the request.
~ AWS Managed AD: Provides centralized user authentication, Kerberos tickets, and DNS resolution, which is essential for securely joining the EC2 instances and allowing them to mount the FSx file share.
~ CloudWatch: Gathers metrics from the ALB, EC2, and FSx. Alarms are configured to trigger automatic notifications or actions based on key metrics (e.g., CPU utilization, network throughput, ALB 5xx errors).
~ CloudTrail: Records all AWS API calls made within the account, storing logs securely in an S3 Bucket for security auditing and compliance.

5. Security Configuration

~ VPC NACL (Network Access Control List): A stateless firewall operating at the subnet level, providing a broad, coarse filter for traffic ~ entering and leaving the Public and Private Subnets.
~ Security Groups (SGs): Stateful firewalls applied at the instance level.
~ ALB SG: Allows inbound traffic from the internet on ports 80/443.
~ EC2 SG: Allows inbound traffic only from the ALB and allows outbound traffic only to the FSx SG on port 445 (SMB) and the AD controllers.
~ FSx SG: Allows inbound traffic only from the EC2 SG on port 445 and from the AD controllers on necessary domain ports.

5. Data Flow
~ User Request: A user enters the domain name, which is resolved by Route 53 to the ALB.
~ Traffic Routing: The ALB forwards the request to one of the two healthy EC2 Application Servers in the Public Subnets.
~ Data Access: The EC2 server processes the request and accesses any required shared application data (e.g., configuration, uploaded files) via the Z: drive (mapped to the FSx file share).
~ Response: The EC2 instance returns the response back through the ALB to the user.
