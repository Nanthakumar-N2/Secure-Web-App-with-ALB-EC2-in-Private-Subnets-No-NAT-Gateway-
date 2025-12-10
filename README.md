
# 🚀 AWS Auto Scaling with Application Load Balancer
## Manual Setup Guide - AWS Console Only

![AWS](https://img.shields.io/badge/AWS-Console-orange.svg)
![Project](https://img.shields.io/badge/Project-Beginner%20Friendly-green.svg)
![License](https://img.shields.io/badge/license-MIT-blue.svg)

> A complete step-by-step guide to set up Auto Scaling Groups (ASG) with Application Load Balancer (ALB) using AWS Management Console only. Perfect for learning and understanding AWS infrastructure.

---

## 📁 Repository Structure

```
aws-autoscaling-loadbalancer/
├── README.md
├── LICENSE
├── docs/
│   ├── 01-vpc-setup.md
│   ├── 02-security-groups.md
│   ├── 03-launch-template.md
│   ├── 04-target-group.md
│   ├── 05-load-balancer.md
│   ├── 06-auto-scaling.md
│   ├── 07-testing.md
│   └── 08-monitoring.md
├── screenshots/
│   ├── step1-vpc/
│   ├── step2-security/
│   ├── step3-ec2/
│   ├── step4-alb/
│   ├── step5-asg/
│   └── step6-testing/
├── resources/
│   ├── user-data.sh
│   ├── sample-webpage.html
│   └── architecture-diagram.png
└── troubleshooting/
    └── common-issues.md
```

---

## 📖 Table of Contents

1. [Project Overview](#project-overview)
2. [Prerequisites](#prerequisites)
3. [Architecture](#architecture)
4. [Step-by-Step Setup](#step-by-step-setup)
5. [Testing](#testing)
6. [Monitoring](#monitoring)
7. [Cost Estimation](#cost-estimation)
8. [Troubleshooting](#troubleshooting)
9. [Cleanup](#cleanup)

---

## 🎯 Project Overview

This project demonstrates how to build a **highly available, auto-scaling web application** on AWS using:

- **EC2 Instances**: Running web servers (Apache/Nginx)
- **Application Load Balancer (ALB)**: Distributing traffic across instances
- **Auto Scaling Group (ASG)**: Automatically adjusting capacity based on demand
- **CloudWatch**: Monitoring and alerting

### What You'll Learn

✅ Creating VPC and Subnets  
✅ Configuring Security Groups  
✅ Launching EC2 Instances  
✅ Setting up Application Load Balancer  
✅ Creating Auto Scaling Groups  
✅ Configuring Scaling Policies  
✅ Monitoring with CloudWatch  
✅ Load Testing

---

## 📋 Prerequisites

### AWS Account
- Active AWS Account
- IAM user with Administrator access or these permissions:
  - EC2 Full Access
  - VPC Full Access
  - CloudWatch Full Access
  - IAM (for creating roles)

### Knowledge Requirements
- Basic understanding of AWS Console
- Basic knowledge of web servers
- Understanding of IP addresses and networking (helpful but not required)

### Tools (Optional for Testing)
- Web Browser
- Command line tool (for load testing)

---

## 🏗️ Architecture

```
                           Internet
                              |
                    Internet Gateway
                              |
                  ┌───────────┴───────────┐
                  │                       │
            Public Subnet           Public Subnet
           (us-east-1a)            (us-east-1b)
                  │                       │
                  └───────────┬───────────┘
                              │
                   Application Load Balancer
                              │
                  ┌───────────┴───────────┐
                  │                       │
            Private Subnet          Private Subnet
           (us-east-1a)            (us-east-1b)
                  │                       │
              EC2 Instance            EC2 Instance
           (Auto Scaling)          (Auto Scaling)
```

### Key Components

| Component | Purpose | Configuration |
|-----------|---------|---------------|
| **VPC** | Isolated network | 10.0.0.0/16 |
| **Public Subnets** | For ALB | 2 AZs for high availability |
| **Private Subnets** | For EC2 instances | 2 AZs for high availability |
| **ALB** | Load distribution | Application Load Balancer |
| **ASG** | Auto scaling | Min: 2, Max: 4, Desired: 2 |
| **Security Groups** | Firewall rules | HTTP, HTTPS, SSH |

---

## 🔧 Step-by-Step Setup

### STEP 1: Create VPC and Subnets

#### 1.1 Create VPC

1. Navigate to **VPC Dashboard**
2. Click **"Create VPC"**
3. Configure VPC:
   - **Name**: `my-asg-vpc`
   - **IPv4 CIDR block**: `10.0.0.0/16`
   - **Tenancy**: Default
   - **Enable DNS hostnames**: ✅ Yes
   - **Enable DNS resolution**: ✅ Yes
4. Click **"Create VPC"**

#### 1.2 Create Internet Gateway

1. In VPC Dashboard, go to **"Internet Gateways"**
2. Click **"Create internet gateway"**
   - **Name**: `my-asg-igw`
3. Click **"Create"**
4. Select the IGW → **Actions** → **"Attach to VPC"**
5. Select `my-asg-vpc` → **"Attach"**

#### 1.3 Create Subnets

**Public Subnet 1:**
- Name: `public-subnet-1a`
- VPC: `my-asg-vpc`
- Availability Zone: `us-east-1a`
- IPv4 CIDR: `10.0.1.0/24`
- Auto-assign Public IP: ✅ Enable

**Public Subnet 2:**
- Name: `public-subnet-1b`
- VPC: `my-asg-vpc`
- Availability Zone: `us-east-1b`
- IPv4 CIDR: `10.0.2.0/24`
- Auto-assign Public IP: ✅ Enable

**Private Subnet 1:**
- Name: `private-subnet-1a`
- VPC: `my-asg-vpc`
- Availability Zone: `us-east-1a`
- IPv4 CIDR: `10.0.11.0/24`

**Private Subnet 2:**
- Name: `private-subnet-1b`
- VPC: `my-asg-vpc`
- Availability Zone: `us-east-1b`
- IPv4 CIDR: `10.0.12.0/24`

#### 1.4 Create and Configure Route Tables

**Public Route Table:**
1. Go to **"Route Tables"** → **"Create route table"**
   - Name: `public-route-table`
   - VPC: `my-asg-vpc`
2. Select the route table → **Routes** tab → **"Edit routes"**
3. Add route:
   - Destination: `0.0.0.0/0`
   - Target: Internet Gateway (`my-asg-igw`)
4. **Subnet associations** tab → **"Edit subnet associations"**
5. Select both public subnets → **Save**

**Private Route Table:**
1. Create route table:
   - Name: `private-route-table`
   - VPC: `my-asg-vpc`
2. Associate with both private subnets
3. **Note**: For this basic setup, private instances will use NAT Gateway or NAT Instance (optional)

---

### STEP 2: Create Security Groups

#### 2.1 ALB Security Group

1. Navigate to **EC2 Dashboard** → **Security Groups**
2. Click **"Create security group"**
3. Configure:
   - **Name**: `alb-security-group`
   - **Description**: Security group for Application Load Balancer
   - **VPC**: `my-asg-vpc`

4. **Inbound Rules**:
   
   | Type | Protocol | Port | Source | Description |
   |------|----------|------|--------|-------------|
   | HTTP | TCP | 80 | 0.0.0.0/0 | Allow HTTP from anywhere |
   | HTTPS | TCP | 443 | 0.0.0.0/0 | Allow HTTPS from anywhere |

5. **Outbound Rules**: Keep default (All traffic)
6. Click **"Create security group"**

#### 2.2 EC2 Security Group

1. Create another security group:
   - **Name**: `ec2-security-group`
   - **Description**: Security group for EC2 instances
   - **VPC**: `my-asg-vpc`

2. **Inbound Rules**:
   
   | Type | Protocol | Port | Source | Description |
   |------|----------|------|--------|-------------|
   | HTTP | TCP | 80 | alb-security-group | Allow HTTP from ALB |
   | SSH | TCP | 22 | Your IP | SSH access (optional) |

3. **Outbound Rules**: Keep default (All traffic)
4. Click **"Create security group"**

---

### STEP 3: Create Launch Template

#### 3.1 Create Launch Template

1. Navigate to **EC2 Dashboard** → **Launch Templates**
2. Click **"Create launch template"**

3. **Launch template details**:
   - **Name**: `web-server-template`
   - **Description**: Template for web server instances
   - **Template version description**: v1.0

4. **Application and OS Images (AMI)**:
   - Click **"Quick Start"**
   - Select **"Amazon Linux 2023"** (or Amazon Linux 2)
   - AMI ID: (latest Amazon Linux 2023 AMI)

5. **Instance type**:
   - Select `t2.micro` (Free tier eligible)

6. **Key pair (login)**:
   - Select existing key pair or create new one
   - **Note**: If you don't need SSH access, select "Proceed without a key pair"

7. **Network settings**:
   - **Subnet**: Don't include in launch template
   - **Firewall (security groups)**: Select `ec2-security-group`

8. **Advanced details** → **User data**:

```bash
#!/bin/bash
# Update system
yum update -y

# Install Apache web server
yum install -y httpd

# Start Apache
systemctl start httpd
systemctl enable httpd

# Get instance metadata
INSTANCE_ID=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)
AVAILABILITY_ZONE=$(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone)
PRIVATE_IP=$(curl -s http://169.254.169.254/latest/meta-data/local-ipv4)

# Create a simple webpage
cat > /var/www/html/index.html <<EOF
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AWS Auto Scaling Demo</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        body {
            font-family: 'Arial', sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 20px;
        }
        .container {
            background: white;
            border-radius: 20px;
            padding: 40px;
            box-shadow: 0 20px 60px rgba(0,0,0,0.3);
            max-width: 600px;
            width: 100%;
        }
        h1 {
            color: #667eea;
            text-align: center;
            margin-bottom: 30px;
            font-size: 2.5em;
        }
        .info-box {
            background: #f8f9fa;
            padding: 20px;
            border-radius: 10px;
            margin: 15px 0;
            border-left: 4px solid #667eea;
        }
        .info-box h3 {
            color: #667eea;
            margin-bottom: 10px;
            font-size: 1.2em;
        }
        .info-box p {
            color: #666;
            font-size: 1em;
            word-break: break-all;
        }
        .status {
            text-align: center;
            margin: 20px 0;
        }
        .status-badge {
            display: inline-block;
            background: #10b981;
            color: white;
            padding: 10px 30px;
            border-radius: 50px;
            font-weight: bold;
            font-size: 1.1em;
        }
        .footer {
            text-align: center;
            margin-top: 30px;
            color: #666;
            font-size: 0.9em;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🚀 AWS Auto Scaling Demo</h1>
        
        <div class="status">
            <span class="status-badge">✓ Server Running</span>
        </div>

        <div class="info-box">
            <h3>Instance ID</h3>
            <p>$INSTANCE_ID</p>
        </div>

        <div class="info-box">
            <h3>Availability Zone</h3>
            <p>$AVAILABILITY_ZONE</p>
        </div>

        <div class="info-box">
            <h3>Private IP Address</h3>
            <p>$PRIVATE_IP</p>
        </div>

        <div class="info-box">
            <h3>Load Balancer</h3>
            <p>Traffic distributed by Application Load Balancer</p>
        </div>

        <div class="footer">
            <p>Powered by AWS Auto Scaling Group | Refresh to see different instances</p>
        </div>
    </div>
</body>
</html>
EOF

# Set permissions
chmod 644 /var/www/html/index.html
```

9. Click **"Create launch template"**

---

### STEP 4: Create Target Group

1. Navigate to **EC2 Dashboard** → **Target Groups**
2. Click **"Create target group"**

3. **Choose target type**:
   - Select **"Instances"**

4. **Target group details**:
   - **Name**: `web-servers-tg`
   - **Protocol**: HTTP
   - **Port**: 80
   - **VPC**: `my-asg-vpc`
   - **Protocol version**: HTTP1

5. **Health checks**:
   - **Health check protocol**: HTTP
   - **Health check path**: `/`
   - **Advanced health check settings**:
     - Healthy threshold: 2
     - Unhealthy threshold: 2
     - Timeout: 5 seconds
     - Interval: 30 seconds
     - Success codes: 200

6. Click **"Next"**
7. **Skip** registering targets (ASG will do this automatically)
8. Click **"Create target group"**

---

### STEP 5: Create Application Load Balancer

1. Navigate to **EC2 Dashboard** → **Load Balancers**
2. Click **"Create load balancer"**

3. **Select load balancer type**:
   - Choose **"Application Load Balancer"**
   - Click **"Create"**

4. **Basic configuration**:
   - **Name**: `web-app-alb`
   - **Scheme**: Internet-facing
   - **IP address type**: IPv4

5. **Network mapping**:
   - **VPC**: `my-asg-vpc`
   - **Mappings**: Select both availability zones
     - ✅ us-east-1a → `public-subnet-1a`
     - ✅ us-east-1b → `public-subnet-1b`

6. **Security groups**:
   - Select `alb-security-group`
   - Remove default security group

7. **Listeners and routing**:
   - **Protocol**: HTTP
   - **Port**: 80
   - **Default action**: Forward to → `web-servers-tg`

8. **Summary**: Review configuration
9. Click **"Create load balancer"**

10. **Wait** for the load balancer state to become **"Active"** (2-3 minutes)

---

### STEP 6: Create Auto Scaling Group

#### 6.1 Create Auto Scaling Group

1. Navigate to **EC2 Dashboard** → **Auto Scaling Groups**
2. Click **"Create Auto Scaling group"**

3. **Step 1: Choose launch template**:
   - **Name**: `web-servers-asg`
   - **Launch template**: Select `web-server-template`
   - **Version**: Latest
   - Click **"Next"**

4. **Step 2: Choose instance launch options**:
   - **VPC**: `my-asg-vpc`
   - **Availability Zones and subnets**: Select both private subnets
     - ✅ `private-subnet-1a`
     - ✅ `private-subnet-1b`
   - Click **"Next"**

5. **Step 3: Configure advanced options**:
   - **Load balancing**: 
     - ✅ Attach to an existing load balancer
     - Choose from your load balancer target groups
     - Select `web-servers-tg`
   - **Health checks**:
     - ✅ Turn on Elastic Load Balancing health checks
     - Health check grace period: 300 seconds
   - **Monitoring**:
     - ✅ Enable group metrics collection within CloudWatch
   - Click **"Next"**

6. **Step 4: Configure group size and scaling**:
   - **Group size**:
     - Desired capacity: `2`
     - Minimum capacity: `2`
     - Maximum capacity: `4`
   
   - **Scaling policies**:
     - Select **"Target tracking scaling policy"**
     - **Scaling policy name**: `cpu-target-tracking`
     - **Metric type**: Average CPU utilization
     - **Target value**: `70`
     - **Instances need**: 300 seconds warm up

   - Click **"Next"**

7. **Step 5: Add notifications** (Optional):
   - Skip for now
   - Click **"Next"**

8. **Step 6: Add tags**:
   - Add tag:
     - Key: `Name`
     - Value: `web-server-instance`
   - Click **"Next"**

9. **Step 7: Review**:
   - Review all settings
   - Click **"Create Auto Scaling group"**

#### 6.2 Additional Scaling Policies (Optional)

To add more specific scaling policies:

1. Select your Auto Scaling Group
2. Go to **"Automatic scaling"** tab
3. Click **"Create dynamic scaling policy"**

**Scale Out Policy (Add Instances)**:
- Policy type: Simple scaling
- Scaling policy name: `scale-out-policy`
- CloudWatch alarm: Create new alarm
  - Metric: CPU Utilization > 70%
  - Period: 5 minutes
- Take action: Add 1 instance
- Wait: 300 seconds

**Scale In Policy (Remove Instances)**:
- Policy type: Simple scaling
- Scaling policy name: `scale-in-policy`
- CloudWatch alarm: Create new alarm
  - Metric: CPU Utilization < 30%
  - Period: 5 minutes
- Take action: Remove 1 instance
- Wait: 300 seconds

---

### STEP 7: Verify Deployment

#### 7.1 Check Auto Scaling Group

1. Go to **Auto Scaling Groups**
2. Select `web-servers-asg`
3. Check:
   - **Activity**: Should show instances launching
   - **Instance management**: Should show 2 instances running
   - **Monitoring**: CPU and other metrics

#### 7.2 Check Target Group

1. Go to **Target Groups**
2. Select `web-servers-tg`
3. Go to **Targets** tab
4. Verify:
   - 2 targets registered
   - Health status: **"healthy"** (wait 2-3 minutes)

#### 7.3 Access Application

1. Go to **Load Balancers**
2. Select `web-app-alb`
3. Copy **DNS name** (e.g., `web-app-alb-1234567890.us-east-1.elb.amazonaws.com`)
4. Open in web browser: `http://[DNS-name]`
5. You should see your webpage with instance details
6. **Refresh** multiple times to see different instances serving requests

---

## 🧪 Testing

### Test 1: High Availability

1. Open ALB URL in browser
2. Note the Instance ID
3. Refresh multiple times
4. You should see different Instance IDs (load balancing)

### Test 2: Auto Scaling (Scale Out)

#### Method 1: Manual Load Test
1. SSH into one instance (if you have SSH access)
2. Run CPU stress:
```bash
# Install stress tool
sudo yum install -y stress

# Create CPU load
stress --cpu 8 --timeout 300s
```

#### Method 2: Terminate Instance
1. Go to **EC2 Dashboard** → **Instances**
2. Select one instance from your ASG
3. **Instance State** → **Terminate**
4. Watch ASG launch a new instance automatically

### Test 3: Health Check

1. SSH into an instance
2. Stop Apache:
```bash
sudo systemctl stop httpd
```
3. Wait 2-3 minutes
4. Check Target Group health status
5. Instance should become "unhealthy"
6. ASG will terminate and replace it

### Test 4: Scaling Policies

1. Go to **CloudWatch** → **Alarms**
2. Check alarms created by Auto Scaling
3. Manually trigger alarm or wait for actual load
4. Watch ASG scale out/in automatically

---

## 📊 Monitoring

### CloudWatch Dashboards

1. Navigate to **CloudWatch** → **Dashboards**
2. Create a new dashboard:
   - Name: `ASG-Monitoring`

3. Add widgets:
   - **ALB Metrics**:
     - Request Count
     - Target Response Time
     - HTTP 4xx/5xx errors
     - Healthy/Unhealthy Host Count
   
   - **ASG Metrics**:
     - Group Desired Capacity
     - Group In-Service Instances
     - Group Min/Max Size
   
   - **EC2 Metrics**:
     - CPU Utilization
     - Network In/Out
     - Disk Read/Write

### Set Up Alarms

1. Go to **CloudWatch** → **Alarms** → **Create alarm**

**High CPU Alarm**:
- Metric: EC2 → By Auto Scaling Group → CPUUtilization
- Condition: Greater than 80%
- Period: 5 minutes
- Action: SNS notification (optional)

**Unhealthy Targets Alarm**:
- Metric: ApplicationELB → UnHealthyHostCount
- Condition: Greater than 0
- Period: 1 minute
- Action: SNS notification

---

## 💰 Cost Estimation

### Monthly Costs (us-east-1 region)

| Service | Configuration | Estimated Cost |
|---------|---------------|----------------|
| EC2 Instances | 2-4 × t2.micro | $7.30 - $14.60 |
| Application Load Balancer | 1 ALB | $16.20 |
| Data Transfer | ~10 GB | $0.90 |
| **Total** | | **~$24.40 - $31.70/month** |

**Note**: 
- t2.micro is Free Tier eligible (750 hours/month for 12 months)
- ALB has no Free Tier
- Actual costs may vary based on usage

### Cost Optimization Tips

✅ Use t2.micro instances (Free Tier)  
✅ Set appropriate min/max instance counts  
✅ Delete unused resources  
✅ Enable AWS Cost Explorer  
✅ Set up billing alerts  

---

## 🐛 Troubleshooting

### Issue 1: Can't Access Load Balancer

**Symptoms**: Timeout when accessing ALB URL

**Solutions**:
1. Check ALB security group allows port 80 from 0.0.0.0/0
2. Verify ALB is in "Active" state
3. Check ALB is in public subnets
4. Verify public subnets have route to Internet Gateway

### Issue 2: Targets Unhealthy

**Symptoms**: Target group shows unhealthy targets

**Solutions**:
1. Check EC2 security group allows port 80 from ALB security group
2. Verify Apache is running: `sudo systemctl status httpd`
3. Check health check path is accessible
4. Review target group health check settings
5. Check EC2 instance logs: `/var/log/httpd/error_log`

### Issue 3: Auto Scaling Not Working

**Symptoms**: Instances not launching/terminating

**Solutions**:
1. Check CloudWatch alarms are in "OK" state
2. Verify scaling policies are configured
3. Review ASG activity history
4. Check if you've reached max capacity
5. Verify IAM role permissions

### Issue 4: User Data Script Failed

**Symptoms**: Webpage not loading, showing default page

**Solutions**:
1. SSH into instance
2. Check user data execution logs:
```bash
sudo cat /var/log/cloud-init-output.log
```
3. Manually run user data commands
4. Verify Apache is installed and running

### Issue 5: Can't SSH into Instance

**Symptoms**: Connection timeout on SSH

**Solutions**:
1. Instances in private subnets need NAT Gateway or bastion host
2. Use AWS Systems Manager Session Manager instead
3. Check security group allows SSH from your IP
4. Verify key pair is correct

---

## 🧹 Cleanup (Important!)

### To Avoid Charges, Delete in This Order:

#### Step 1: Delete Auto Scaling Group
1. Go to **Auto Scaling Groups**
2. Select `web-servers-asg`
3. **Actions** → **Delete**
4. Confirm deletion
5. Wait for all instances to terminate

#### Step 2: Delete Load Balancer
1. Go to **Load Balancers**
2. Select `web-app-alb`
3. **Actions** → **Delete load balancer**
4. Confirm deletion

#### Step 3: Delete Target Group
1. Go to **Target Groups**
2. Select `web-servers-tg`
3. **Actions** → **Delete**

#### Step 4: Delete Launch Template
1. Go to **Launch Templates**
2. Select `web-server-template`
3. **Actions** → **Delete template**

#### Step 5: Delete Security Groups
1. Delete `ec2-security-group`
2. Delete `alb-security-group`

#### Step 6: Delete VPC
1. Go to **VPC Dashboard**
2. **Actions** → **Delete VPC**
3. This will delete all associated resources (subnets, route tables, IGW)

#### Step 7: Verify CloudWatch
1. Check no alarms are left
2. Delete custom dashboards

---

## 📚 Additional Resources

### AWS Documentation
- [Auto Scaling User Guide](https://docs.aws.amazon.com/autoscaling/)
- [Application Load Balancer Guide](https://docs.aws.amazon.com/elasticloadbalancing/)
- [EC2 User Guide](https://docs.aws.amazon.com/ec2/)

### Learning Resources
- AWS Free Tier: [https://aws.amazon.com/free/](https://aws.amazon.com/free/)
- AWS Well-Architected Framework
- AWS Architecture Center

---

## 🤝 Contributing

Found an issue or want to improve this guide?

1. Fork the repository
2. Create your feature branch
3. Commit your changes
4. Push to the branch
5. Open a Pull Request

---

## 📝 License

This project is licensed under the MIT License.

---

## 📧 Support

If you have questions:
- Open an issue on GitHub
- Check troubleshooting section
- Review AWS documentation

---

## ✅ Checklist

Use this checklist to track your progress:

- [ ] AWS Account created
- [ ] VPC and subnets configured
- [ ] Security groups created
- [ ] Launch template created
- [ ] Target group created
- [ ] Load balancer deployed
- [ ] Auto Scaling Group configured
- [ ] Application accessible via ALB
- [ ] Scaling policies tested
- [ ] Monitoring configured
- [ ] Documentation reviewed
- [ ] All resources cleaned up (if done learning)

---

## 🎯 Next Steps

After completing this project, you can:

1. Add HTTPS/SSL certificate using ACM
2. Implement Multi-Region deployment
3. Add RDS database backend
4. Implement CloudFront CDN
5. Add WAF for security
6. Implement CI/CD pipeline
7. Use AWS Systems Manager for management
8. Implement Container-based deployment (ECS/EKS)

---

**⭐ Star this repository if you found it helpful!**

**Happy Learning! 🚀**
