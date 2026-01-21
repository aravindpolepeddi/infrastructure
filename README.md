# Infrastructure - AWS CloudFormation Setup

**Author:** Aravind Polepeddi  

## Overview

This repository contains AWS CloudFormation templates and configuration files for provisioning and managing cloud infrastructure. The infrastructure supports a complete cloud-native application stack including web services, serverless functions, databases, networking, and security components.

## System Architecture

This infrastructure is part of a three-repository cloud application system:

```
┌─────────────────────────────────────────────────────────────┐
│                     CLOUD APPLICATION                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌────────────────┐  ┌──────────────┐  ┌────────────────┐  │
│  │ Infrastructure │  │ Web Service  │  │   Serverless   │  │
│  │  (this repo)   │  │              │  │                │  │
│  ├────────────────┤  ├──────────────┤  ├────────────────┤  │
│  │ • VPC          │  │ • Node.js    │  │ • Lambda       │  │
│  │ • Subnets      │  │ • Express    │  │ • SNS/SQS      │  │
│  │ • Security     │  │ • REST API   │  │ • Event        │  │
│  │ • RDS          │  │ • PostgreSQL │  │   Processing   │  │
│  │ • Load Balancer│  │ • Health     │  │ • Background   │  │
│  │ • Auto Scaling │  │   Checks     │  │   Jobs         │  │
│  │ • S3           │  │ • Packer AMI │  │ • Async Tasks  │  │
│  │ • Route53      │  │ • CodeDeploy │  │                │  │
│  │ • Lambda Config│  │              │  │                │  │
│  └────────────────┘  └──────────────┘  └────────────────┘  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Detailed AWS Infrastructure Architecture

```
                                    INTERNET
                                       │
                                       │
                              ┌────────▼────────┐
                              │   Route 53      │
                              │  DNS Service    │
                              └────────┬────────┘
                                       │
                                       │
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┻━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃                              AWS REGION (us-east-1)                      ┃
┃  ┌──────────────────────────────────────────────────────────────────┐   ┃
┃  │                    VPC (10.0.0.0/16)                             │   ┃
┃  │                                                                  │   ┃
┃  │  ┌───────────────────────────────────────────────────────────┐  │   ┃
┃  │  │              AVAILABILITY ZONE 1 (us-east-1a)             │  │   ┃
┃  │  │                                                           │  │   ┃
┃  │  │  ┌──────────────────────────────────────────┐            │  │   ┃
┃  │  │  │ Public Subnet 1 (10.0.1.0/24)            │            │  │   ┃
┃  │  │  │  ┌──────────────────────┐                │            │  │   ┃
┃  │  │  │  │  Internet Gateway    │◄───────────────┼────────────┼──┼───┃
┃  │  │  │  └──────────────────────┘                │            │  │   ┃
┃  │  │  │  ┌──────────────────────┐                │            │  │   ┃
┃  │  │  │  │   NAT Gateway 1      │                │            │  │   ┃
┃  │  │  │  │   (Elastic IP)       │                │            │  │   ┃
┃  │  │  │  └──────────┬───────────┘                │            │  │   ┃
┃  │  │  │             │                            │            │  │   ┃
┃  │  │  │  ┌──────────▼───────────┐                │            │  │   ┃
┃  │  │  │  │ Application Load     │                │            │  │   ┃
┃  │  │  │  │   Balancer (ALB)     │◄───HTTPS/HTTP──┼────────────┼──┼───┃
┃  │  │  │  │   Target Group       │                │            │  │   ┃
┃  │  │  │  └──────────┬───────────┘                │            │  │   ┃
┃  │  │  └─────────────┼────────────────────────────┘            │  │   ┃
┃  │  │                │                                         │  │   ┃
┃  │  │  ┌─────────────▼──────────────────────────┐             │  │   ┃
┃  │  │  │ Private Subnet 1 (10.0.3.0/24)         │             │  │   ┃
┃  │  │  │  ┌────────────────────────────────┐    │             │  │   ┃
┃  │  │  │  │  EC2 Auto Scaling Group        │    │             │  │   ┃
┃  │  │  │  │  ┌──────────┐  ┌──────────┐    │    │             │  │   ┃
┃  │  │  │  │  │   EC2    │  │   EC2    │    │    │             │  │   ┃
┃  │  │  │  │  │ Instance │  │ Instance │    │    │             │  │   ┃
┃  │  │  │  │  │ (Node.js)│  │ (Node.js)│    │    │             │  │   ┃
┃  │  │  │  │  └────┬─────┘  └────┬─────┘    │    │             │  │   ┃
┃  │  │  │  └───────┼─────────────┼──────────┘    │             │  │   ┃
┃  │  │  │          │             │                │             │  │   ┃
┃  │  │  │  ┌───────▼─────────────▼────────┐      │             │  │   ┃
┃  │  │  │  │   Lambda Functions (VPC)     │      │             │  │   ┃
┃  │  │  │  │   - Event Processing         │      │             │  │   ┃
┃  │  │  │  │   - Email Service            │      │             │  │   ┃
┃  │  │  │  └───────┬──────────────────────┘      │             │  │   ┃
┃  │  │  │          │                              │             │  │   ┃
┃  │  │  │  ┌───────▼──────────────────────┐      │             │  │   ┃
┃  │  │  │  │   RDS PostgreSQL             │      │             │  │   ┃
┃  │  │  │  │   Primary Instance           │      │             │  │   ┃
┃  │  │  │  │   (db.t3.micro)              │      │             │  │   ┃
┃  │  │  │  └──────────────────────────────┘      │             │  │   ┃
┃  │  │  └─────────────────────────────────────────┘             │  │   ┃
┃  │  └───────────────────────────────────────────────────────────┘  │   ┃
┃  │                                                                  │   ┃
┃  │  ┌───────────────────────────────────────────────────────────┐  │   ┃
┃  │  │              AVAILABILITY ZONE 2 (us-east-1b)             │  │   ┃
┃  │  │                                                           │  │   ┃
┃  │  │  ┌──────────────────────────────────────────┐            │  │   ┃
┃  │  │  │ Public Subnet 2 (10.0.2.0/24)            │            │  │   ┃
┃  │  │  │  ┌──────────────────────┐                │            │  │   ┃
┃  │  │  │  │   NAT Gateway 2      │                │            │  │   ┃
┃  │  │  │  │   (Elastic IP)       │                │            │  │   ┃
┃  │  │  │  └──────────┬───────────┘                │            │  │   ┃
┃  │  │  │             │                            │            │  │   ┃
┃  │  │  │  ┌──────────▼───────────┐                │            │  │   ┃
┃  │  │  │  │ Application Load     │                │            │  │   ┃
┃  │  │  │  │   Balancer (ALB)     │                │            │  │   ┃
┃  │  │  │  └──────────┬───────────┘                │            │  │   ┃
┃  │  │  └─────────────┼────────────────────────────┘            │  │   ┃
┃  │  │                │                                         │  │   ┃
┃  │  │  ┌─────────────▼──────────────────────────┐             │  │   ┃
┃  │  │  │ Private Subnet 2 (10.0.4.0/24)         │             │  │   ┃
┃  │  │  │  ┌────────────────────────────────┐    │             │  │   ┃
┃  │  │  │  │  EC2 Auto Scaling Group        │    │             │  │   ┃
┃  │  │  │  │  ┌──────────┐  ┌──────────┐    │    │             │  │   ┃
┃  │  │  │  │  │   EC2    │  │   EC2    │    │    │             │  │   ┃
┃  │  │  │  │  │ Instance │  │ Instance │    │    │             │  │   ┃
┃  │  │  │  │  │ (Node.js)│  │ (Node.js)│    │    │             │  │   ┃
┃  │  │  │  │  └────┬─────┘  └────┬─────┘    │    │             │  │   ┃
┃  │  │  │  └───────┼─────────────┼──────────┘    │             │  │   ┃
┃  │  │  │          │             │                │             │  │   ┃
┃  │  │  │  ┌───────▼─────────────▼────────┐      │             │  │   ┃
┃  │  │  │  │   Lambda Functions (VPC)     │      │             │  │   ┃
┃  │  │  │  └───────┬──────────────────────┘      │             │  │   ┃
┃  │  │  │          │                              │             │  │   ┃
┃  │  │  │  ┌───────▼──────────────────────┐      │             │  │   ┃
┃  │  │  │  │   RDS PostgreSQL             │      │             │  │   ┃
┃  │  │  │  │   Standby Instance           │      │             │  │   ┃
┃  │  │  │  │   (Multi-AZ Replica)         │      │             │  │   ┃
┃  │  │  │  └──────────────────────────────┘      │             │  │   ┃
┃  │  │  └─────────────────────────────────────────┘             │  │   ┃
┃  │  └───────────────────────────────────────────────────────────┘  │   ┃
┃  │                                                                  │   ┃
┃  └──────────────────────────────────────────────────────────────────┘   ┃
┃                                                                         ┃
┃  ┌──────────────────────────────────────────────────────────────────┐   ┃
┃  │                    SUPPORTING SERVICES                           │   ┃
┃  │                                                                  │   ┃
┃  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │   ┃
┃  │  │  S3 Buckets  │  │   SNS/SQS    │  │  CloudWatch  │          │   ┃
┃  │  │  - App Data  │  │  - Events    │  │  - Logs      │          │   ┃
┃  │  │  - Backups   │  │  - Messages  │  │  - Metrics   │          │   ┃
┃  │  └──────────────┘  └──────────────┘  └──────────────┘          │   ┃
┃  │                                                                  │   ┃
┃  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │   ┃
┃  │  │  IAM Roles   │  │    Secrets   │  │  CodeDeploy  │          │   ┃
┃  │  │  - EC2       │  │   Manager    │  │  - Pipeline  │          │   ┃
┃  │  │  - Lambda    │  │  - DB Creds  │  │  - Deploy    │          │   ┃
┃  │  └──────────────┘  └──────────────┘  └──────────────┘          │   ┃
┃  └──────────────────────────────────────────────────────────────────┘   ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛

                    ┌─────────────────────────┐
                    │   CI/CD Pipeline        │
                    │  (GitHub Actions)       │
                    │  - Build AMI            │
                    │  - Deploy Lambda        │
                    │  - Update Stack         │
                    └─────────────────────────┘
```

### Traffic Flow

1. **User Request** → Route 53 → Application Load Balancer
2. **ALB** → Health Check → EC2 Instances (via Target Group)
3. **EC2 Instance** → Process Request → PostgreSQL RDS
4. **EC2 Instance** → Publish Event → SNS Topic
5. **SNS Topic** → Trigger Lambda Function
6. **Lambda** → Process Event → RDS / S3 / Send Email
7. **Response** → ALB → User

### Related Repositories

- **Infrastructure** (this repository): [aravindpolepeddi/infrastructure](https://github.com/aravindpolepeddi/infrastructure)
- **Web Service**: [aravindpolepeddi/webservice](https://github.com/aravindpolepeddi/webservice)
- **Serverless**: [aravindpolepeddi/serverless](https://github.com/aravindpolepeddi/serverless)

## Repository Structure

```
infrastructure/
├── csye6225-infra.yml          # Main CloudFormation template
├── policies.yml                 # IAM policies and security configurations
├── config.json                  # CloudFormation stack parameters
├── prod_polepeddiaravind.me/   # SSL/TLS certificates for production domain
│   ├── prod_polepeddiaravind_me.crt
│   ├── prod_polepeddiaravind_me.ca-bundle
│   └── private.key (to be dowloaded from aws certmanager)
├── .gitignore                   # Git ignore rules
├── LICENSE                      # MIT License
└── README.md                    # This file
```

## Prerequisites

- AWS CLI installed and configured
- AWS account with appropriate permissions
- Valid SSL/TLS certificates (for HTTPS)
- Basic understanding of AWS CloudFormation
- Custom AMI created from webservice repository
- Lambda function package from serverless repository

## AWS Profile Configuration

This project supports multiple AWS profiles for different environments (dev, demo, prod).

### Configure AWS Profiles

```bash
# Configure development profile
aws configure --profile=dev

# Configure demo profile
aws configure --profile=demo
```

### Set Active Profile

```bash
# Set development profile as active
export AWS_PROFILE=dev

# Set demo profile as active
export AWS_PROFILE=demo
```

## Infrastructure Components

### Networking Layer
- **VPC:** Custom Virtual Private Cloud with configurable CIDR blocks
- **Subnets:** Public and private subnets across multiple availability zones
- **Internet Gateway:** For public internet access
- **NAT Gateway:** For private subnet internet access
- **Route Tables:** Custom routing for public/private subnets
- **Network ACLs:** Additional network security layer

### Compute Layer
- **EC2 Auto Scaling:** Dynamic scaling based on demand
- **Launch Templates:** Standardized EC2 configurations with custom AMIs
- **Application Load Balancer:** Traffic distribution across instances
- **Target Groups:** Health check and routing configurations
- **Lambda Functions:** Serverless compute for event processing

### Database Layer
- **RDS PostgreSQL:** Managed relational database
- **Multi-AZ Deployment:** High availability configuration
- **Automated Backups:** Point-in-time recovery
- **Parameter Groups:** Custom database configurations
- **Subnet Groups:** Database networking configuration

### Storage Layer
- **S3 Buckets:** Object storage for application assets
- **Versioning:** Data protection and recovery
- **Lifecycle Policies:** Cost optimization
- **Encryption:** Data security at rest

### Security Layer
- **Security Groups:** Firewall rules for EC2, RDS, and Lambda
- **IAM Roles:** EC2 instance profiles and Lambda execution roles
- **IAM Policies:** Fine-grained access control
- **SSL/TLS Certificates:** Secure HTTPS communication
- **KMS Keys:** Encryption key management

### DNS & CDN
- **Route53:** DNS management and domain routing
- **Hosted Zones:** Domain name configuration
- **Record Sets:** A, CNAME, and alias records
- **Health Checks:** DNS failover configuration

### Monitoring & Logging
- **CloudWatch:** Metrics and monitoring
- **CloudWatch Logs:** Application and system logs
- **CloudWatch Alarms:** Automated alerts
- **SNS Topics:** Notification delivery

## CloudFormation Stack Management

### Create Stack

Create a new CloudFormation stack using the template and parameters:

```bash
aws cloudformation create-stack \
  --stack-name <stack_name> \
  --capabilities CAPABILITY_NAMED_IAM \
  --profile=<profile_name> \
  --template-body file://csye6225-infra.yml \
  --parameters file://config.json
```

**Example:**
```bash
aws cloudformation create-stack \
  --stack-name vpc-stack \
  --capabilities CAPABILITY_NAMED_IAM \
  --profile=demo \
  --template-body file://csye6225-infra.yml \
  --parameters file://config.json
```

### Update Stack

Update an existing stack with changes:

```bash
aws cloudformation update-stack \
  --stack-name <stack_name> \
  --template-body file://csye6225-infra.yml \
  --parameters file://config.json \
  --capabilities CAPABILITY_NAMED_IAM
```

### Delete Stack

Remove an existing CloudFormation stack:

```bash
aws cloudformation delete-stack --stack-name <stack_name>
```

**Example:**
```bash
aws cloudformation delete-stack --stack-name vpc-stack
```

### Verify Resources

List all VPCs to verify stack creation:

```bash
# List all VPCs in the current profile
aws ec2 describe-vpcs --profile=dev

# Filter VPCs by name
aws ec2 describe-vpcs --filters Name=tag:Name,Values=dev-vpc
```

## SSL/TLS Certificate Management

### Import SSL Certificate to ACM

Import your SSL/TLS certificate into AWS Certificate Manager:

```bash
aws acm import-certificate \
  --certificate fileb://prod_polepeddiaravind.me/prod_polepeddiaravind_me.crt \
  --certificate-chain fileb://prod_polepeddiaravind.me/prod_polepeddiaravind_me.ca-bundle \
  --private-key fileb://prod_polepeddiaravind.me/private.key \
  --profile=<profile_name>
```

**Note:** The `fileb://` prefix is used for binary file reading. Ensure certificate files are in PEM format.

### Certificate Requirements

- **Format:** PEM-encoded
- **Components Required:**
  - Certificate (.crt)
  - Certificate chain (CA bundle)
  - Private key
- **Validation:** Domain validation completed
- **Expiration:** Monitor and renew before expiry

## Configuration

### config.json

The `config.json` file contains CloudFormation stack parameters:

```json
{
  "Parameters": {
    "VpcCidr": "10.0.0.0/16",
    "PublicSubnet1Cidr": "10.0.1.0/24",
    "PublicSubnet2Cidr": "10.0.2.0/24",
    "PrivateSubnet1Cidr": "10.0.3.0/24",
    "PrivateSubnet2Cidr": "10.0.4.0/24",
    "InstanceType": "t2.micro",
    "KeyName": "your-key-pair",
    "AMIId": "ami-xxxxxxxxxxxxx",
    "DBInstanceClass": "db.t3.micro",
    "DBName": "webapp",
    "DBUsername": "dbadmin",
    "DomainName": "polepeddiaravind.me",
    "CertificateArn": "arn:aws:acm:region:account:certificate/xxxxx"
  }
}
```

### policies.yml

Contains IAM policies and security configurations:

**EC2 Instance Role Policies:**
- CloudWatch Logs write access
- S3 bucket access for application data
- SNS publish permissions
- RDS connection permissions
- Secrets Manager read access

**Lambda Execution Role Policies:**
- CloudWatch Logs write access
- SNS/SQS access for event processing
- S3 read/write access
- RDS connection via VPC
- Secrets Manager access

## Deployment Workflow

### Complete Deployment Process

1. **Infrastructure Setup** (this repository):
```bash
# Deploy networking and base infrastructure
aws cloudformation create-stack \
  --stack-name base-infrastructure \
  --template-body file://csye6225-infra.yml \
  --parameters file://config.json \
  --capabilities CAPABILITY_NAMED_IAM
```

2. **AMI Creation** (webservice repository):
```bash
# Build custom AMI with Packer
cd ../webservice
packer build -var-file='vars.json' ami.pkr.hcl
```

3. **Update Infrastructure with AMI**:
```bash
# Update config.json with new AMI ID
# Update stack to use new AMI
cd ../infrastructure
aws cloudformation update-stack \
  --stack-name base-infrastructure \
  --template-body file://csye6225-infra.yml \
  --parameters file://config.json \
  --capabilities CAPABILITY_NAMED_IAM
```

4. **Deploy Lambda Functions** (serverless repository):
```bash
# Package and deploy Lambda
cd ../serverless
npm run deploy
```

### CI/CD Integration

The complete deployment is automated through GitHub Actions across all three repositories:

**Infrastructure Repository:**
- Validates CloudFormation templates
- Runs security scans
- Creates/updates stacks on merge

**Webservice Repository:**
- Builds and tests application
- Creates AMI with Packer
- Triggers infrastructure update

**Serverless Repository:**
- Packages Lambda functions
- Deploys to Lambda service
- Updates function configuration

## Network Architecture

### VPC Design

```
VPC (10.0.0.0/16)
├── Public Subnet 1 (10.0.1.0/24) - AZ-1
│   ├── NAT Gateway
│   ├── Application Load Balancer
│   └── Bastion Host (optional)
├── Public Subnet 2 (10.0.2.0/24) - AZ-2
│   ├── NAT Gateway
│   └── Application Load Balancer
├── Private Subnet 1 (10.0.3.0/24) - AZ-1
│   ├── EC2 Auto Scaling Group
│   ├── Lambda Functions (VPC)
│   └── RDS Primary
└── Private Subnet 2 (10.0.4.0/24) - AZ-2
    ├── EC2 Auto Scaling Group
    ├── Lambda Functions (VPC)
    └── RDS Standby
```

### Security Group Configuration

**Load Balancer Security Group:**
- Inbound: 443 (HTTPS) from 0.0.0.0/0
- Inbound: 80 (HTTP) from 0.0.0.0/0 (redirect to HTTPS)
- Outbound: All traffic

**Application Security Group:**
- Inbound: 3000 (Node.js) from Load Balancer SG
- Inbound: 22 (SSH) from Bastion SG (optional)
- Outbound: All traffic

**Database Security Group:**
- Inbound: 5432 (PostgreSQL) from Application SG
- Inbound: 5432 (PostgreSQL) from Lambda SG
- Outbound: None required

**Lambda Security Group:**
- Inbound: None
- Outbound: All traffic (for RDS, SNS, S3 access)

## Auto Scaling Configuration

### Scaling Policies

**Scale Up:**
- Metric: CPU Utilization > 70%
- Action: Add 1 instance
- Cooldown: 300 seconds

**Scale Down:**
- Metric: CPU Utilization < 30%
- Action: Remove 1 instance
- Cooldown: 300 seconds

### Capacity Limits

- **Minimum:** 1 instance
- **Desired:** 2 instances
- **Maximum:** 5 instances

## Monitoring & Alerts

### CloudWatch Alarms

**Application Health:**
- Unhealthy host count > 0
- HTTP 5xx errors > threshold
- Response time > 1000ms

**Database Performance:**
- CPU utilization > 80%
- Free storage space < 10GB
- Read/Write latency > 100ms

**Lambda Performance:**
- Error rate > 5%
- Throttles > 0
- Duration > timeout threshold

### SNS Notifications

Configure SNS topics for:
- Critical infrastructure alerts
- Scaling events
- Deployment notifications
- Security incidents

## Best Practices

### Infrastructure as Code
1. **Version Control:** All templates in Git
2. **Code Review:** Peer review for changes
3. **Testing:** Validate templates before deployment
4. **Documentation:** Comment complex configurations
5. **Modularity:** Use nested stacks for large deployments

### Security
1. **Least Privilege:** Minimal IAM permissions
2. **Encryption:** Enable for data at rest and in transit
3. **Secrets Management:** Use Secrets Manager/Parameter Store
4. **Network Segmentation:** Public/private subnet isolation
5. **Security Groups:** Restrictive inbound rules

### Cost Optimization
1. **Right Sizing:** Match instance types to workload
2. **Auto Scaling:** Scale based on demand
3. **Reserved Instances:** For predictable workloads
4. **S3 Lifecycle:** Archive old data
5. **Spot Instances:** For non-critical workloads

### High Availability
1. **Multi-AZ:** Deploy across availability zones
2. **Load Balancing:** Distribute traffic
3. **Health Checks:** Automated instance replacement
4. **Database Failover:** RDS Multi-AZ
5. **Backup Strategy:** Regular automated backups

## Troubleshooting

### Stack Creation Fails

1. **Check CloudFormation Events:**
```bash
aws cloudformation describe-stack-events \
  --stack-name <stack-name> \
  --max-items 20
```

2. **Common Issues:**
   - Invalid parameter values
   - Insufficient IAM permissions
   - Resource quotas exceeded
   - AMI not available in region
   - Certificate ARN incorrect

### Resource Access Issues

1. **Verify Security Groups:**
```bash
aws ec2 describe-security-groups \
  --filters Name=group-name,Values=application-sg
```

2. **Check Route Tables:**
```bash
aws ec2 describe-route-tables \
  --filters Name=vpc-id,Values=<vpc-id>
```

### Application Not Accessible

1. Check Load Balancer health checks
2. Verify DNS records in Route53
3. Confirm SSL certificate is valid
4. Check target group registrations
5. Review application logs in CloudWatch

### Database Connection Issues

1. Verify RDS instance status
2. Check security group rules
3. Validate connection string
4. Confirm subnet group configuration
5. Review database logs

## Cost Estimation

### Monthly Cost Breakdown (Approximate)

- **VPC & Networking:** $30-50 (NAT Gateways, data transfer)
- **EC2 Instances:** $15-100 (t2.micro to t3.medium, 2-5 instances)
- **Load Balancer:** $20-25
- **RDS PostgreSQL:** $15-50 (db.t3.micro to db.t3.small)
- **S3 Storage:** $5-20 (depends on usage)
- **Lambda:** $0-10 (1M free tier requests)
- **CloudWatch:** $5-15 (logs and metrics)
- **Route53:** $0.50 per hosted zone
- **Data Transfer:** Variable

**Total Estimated Monthly Cost:** $90-270 (depending on usage)

## Disaster Recovery

### Backup Strategy

1. **RDS Automated Backups:**
   - Daily automated snapshots
   - 7-day retention period
   - Point-in-time recovery

2. **S3 Versioning:**
   - Object version history
   - Accidental deletion protection

3. **Infrastructure as Code:**
   - CloudFormation templates in Git
   - Easy recreation of entire stack

### Recovery Procedures

**Database Recovery:**
```bash
# Restore from snapshot
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier restored-db \
  --db-snapshot-identifier <snapshot-id>
```

**Infrastructure Recovery:**
```bash
# Recreate stack from template
aws cloudformation create-stack \
  --stack-name recovered-stack \
  --template-body file://csye6225-infra.yml \
  --parameters file://config.json
```

## Related Repositories

- **Web Service:** [aravindpolepeddi/webservice](https://github.com/aravindpolepeddi/webservice)
- **Serverless:** [aravindpolepeddi/serverless](https://github.com/aravindpolepeddi/serverless)

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make infrastructure changes
4. Test in dev environment
5. Update documentation
6. Submit pull request
7. Wait for review and approval

## Support

For issues or questions:
- **Infrastructure Issues:** Open an issue in this repository
- **Application Code:** See webservice repository
- **Lambda Functions:** See serverless repository
- **AWS Service Issues:** Contact AWS Support

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
