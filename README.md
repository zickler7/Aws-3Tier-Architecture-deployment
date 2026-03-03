# Aws-3Tier-Architecture-deployment
Highly available 3-tier web appliaction using AWS VPC, EC2 and AURORA MYSQL
This project demonstrates a production-ready, scalable architecture on AWS. It separates the Presentation, Application, and Data layers to ensure maximum security and independent scalability.
## The Architecture at a Glance 
Tier,Technology,Function
Presentation,React.js / Nginx,Public-facing UI handled by an external ALB.
Application,Node.js,Private API layer processing logic via an internal ALB.
Data,Aurora MySQL,Multi-AZ database for high availability and persistence.
## Key Features
High Availability: Multi-AZ deployment with Auto Scaling Groups (ASG) to ensure zero downtime.

Security: Private subnets isolate the App and Data tiers; Security Groups (like 'DBSG') enforce strict traffic rules.

Scalability: Decoupled design allows each tier to scale horizontally based on demand.
## AWS Services & Technical Tools
Compute: EC2 (Web & App Tiers), Auto Scaling Groups (ASG).

Networking: VPC, Public/Private Subnets, NAT Gateway, Internet Gateway, Application Load Balancers (ALB).

Database: Amazon Aurora MySQL (Multi-AZ).

Security: IAM Roles, Security Groups (specifically 'DBSG' for database isolation).

Dev Tools & CLI: MariaDB Client (for DB connectivity testing), SSH (for instance access).
## Networking & Security Infrastructure
To ensure a "Defense in Depth" posture, the network was designed with strict isolation between the public internet and the sensitive data layers.

Custom VPC ('3tier-vpc'): A dedicated network environment with a specific CIDR block to house all project resources.

Subnet Strategy: Six subnets were deployed across two Availability Zones (AZs) to ensure high availability and fault tolerance.

Public Subnets: Hosted the external Application Load Balancer and NAT Gateways.

Private App Subnets: Hosted the Node.js application instances, accessible only via the internal Load Balancer.

Private Data Subnets: Hosted the Aurora MySQL database, with no direct route to the internet.

Security Groups (Firewalls):

Web-SG: Allowed HTTP/HTTPS traffic from the public internet.

App-SG: Restricted access to only allow traffic from the Web-SG.

DB-SG ('DBSG'): Configured to only accept MySQL/Aurora (port 3306) connections originating from the App-SG.
## Database Tier & Connection Validation
The Data Tier was built using Amazon Aurora MySQL in a Multi-AZ configuration for high availability and automated failover.

Instance Type: db.t3.medium (or whatever you used).

Security: Managed by the 'DBSG' group, allowing inbound traffic on port 3306 only from the Application Tier Security Group.

Connectivity Test: To validate the 3-tier communication, I accessed a private App instance and successfully connected to the Aurora cluster using the Writer Endpoint:
mariadb -h [your-aurora-endpoint] -u admin -p
This confirmed that the Security Group rules and VPC Routing were correctly configured for cross-tier communication.
## Challenges & Solutions
Challenge: Connectivity issue between App Tier and RDS.

Problem: After deploying the database, the Node.js application couldn't establish a connection, resulting in a "Connection Timed Out" error.

Troubleshooting: I used Reachability Analyzer and checked the Security Group rules for the 'DBSG' group. I discovered that the inbound rule was incorrectly set to the Web-SG instead of the App-SG.

Solution: I updated the 'DBSG' security group to specifically allow traffic on port 3306 only from the Application Tier’s Security Group ID, successfully enforcing the Principle of Least Privilege.

Challenge: Ensuring High Availability during a simulated outage.

Problem: Needed to verify if the Auto Scaling Group (ASG) would actually trigger a replacement if an instance failed.

Action: Manually terminated an instance in one Availability Zone.

Result: The ASG health checks identified the "unhealthy" state and automatically provisioned a new instance in the second AZ, maintaining the desired capacity of 2.
## Contact Info: 
