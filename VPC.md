# 01 - VPC Setup Guide

## Overview

In this guide, we'll create a complete Virtual Private Cloud (VPC) infrastructure with public and private subnets across two availability zones for high availability.

## Architecture

```
VPC (10.0.0.0/16)
├── Internet Gateway
├── Public Subnets (for ALB)
│   ├── public-subnet-1a (10.0.1.0/24) - us-east-1a
│   └── public-subnet-1b (10.0.2.0/24) - us-east-1b
├── Private Subnets (for EC2)
│   ├── private-subnet-1a (10.0.11.0/24) - us-east-1a
│   └── private-subnet-1b (10.0.12.0/24) - us-east-1b
└── Route Tables
    ├── Public Route Table (routes to Internet Gateway)
    └── Private Route Table
```

---

## Step 1: Create VPC

### 1.1 Navigate to VPC Dashboard

1. Sign in to **AWS Management Console**
2. Search for **"VPC"** in the services search bar
3. Click on **"VPC"** to open the VPC Dashboard

### 1.2 Create New VPC

1. In the left sidebar, click **"Your VPCs"**
2. Click the **"Create VPC"** button (orange button in top right)

### 1.3 Configure VPC Settings

Fill in the following details:

| Field | Value | Description |
|-------|-------|-------------|
| **Resources to create** | VPC only | We'll create subnets separately |
| **Name tag** | `my-asg-vpc` | Name for your VPC |
| **IPv4 CIDR block** | Manual input | |
| **IPv4 CIDR** | `10.0.0.0/16` | Provides 65,536 IP addresses |
| **IPv6 CIDR block** | No IPv6 CIDR block | We don't need IPv6 |
| **Tenancy** | Default | Shared hardware (cheaper) |

### 1.4 Additional Settings

Scroll down and configure DNS settings:

- **DNS resolution**: ✅ **Enabled**
- **DNS hostnames**: ✅ **Enabled**

**Why?** These settings allow instances to resolve public DNS names and get public DNS hostnames.

### 1.5 Tags (Optional but Recommended)

Click **"Add new tag"** to add additional tags:

| Key | Value |
|-----|-------|
| Project | AutoScaling-Demo |
| Environment | Development |
| ManagedBy | Manual |

### 1.6 Create VPC

1. Review all settings
2. Click **"Create VPC"** button
3. Wait for success message: "Successfully created vpc-xxxxx"
4. Click **"Close"**

**✅ Checkpoint**: Note down your VPC ID (format: vpc-xxxxxxxxxxxxx)

---

## Step 2: Create Internet Gateway

### 2.1 Navigate to Internet Gateways

1. In the left sidebar, click **"Internet Gateways"**
2. Click **"Create internet gateway"** button

### 2.2 Configure Internet Gateway

| Field | Value |
|-------|-------|
| **Name tag** | `my-asg-igw` |

Add optional tags:
| Key | Value |
|-----|-------|
| Project | AutoScaling-Demo |

### 2.3 Create and Attach

1. Click **"Create internet gateway"**
2. Success message appears
3. Click **"Actions"** dropdown (top right)
4. Select **"Attach to VPC"**

### 2.4 Attach to VPC

1. In the dropdown, select **"my-asg-vpc"**
2. Click **"Attach internet gateway"**
3. Status changes to **"Attached"**

**✅ Checkpoint**: Internet Gateway is attached to your VPC

---

## Step 3: Create Subnets

We'll create 4 subnets:
- 2 Public subnets (for Application Load Balancer)
- 2 Private subnets (for EC2 instances)

### 3.1 Navigate to Subnets

1. In the left sidebar, click **"Subnets"**
2. Click **"Create subnet"** button

### 3.2 Create Public Subnet 1

**VPC Selection:**
- Select **"my-asg-vpc"** from the dropdown

**Subnet 1 of 4 - Public Subnet 1a:**

| Field | Value |
|-------|-------|
| **Subnet name** | `public-subnet-1a` |
| **Availability Zone** | `us-east-1a` |
| **IPv4 CIDR block** | `10.0.1.0/24` |

Click **"Add new subnet"** button at the bottom

### 3.3 Create Public Subnet 2

**Subnet 2 of 4 - Public Subnet 1b:**

| Field | Value |
|-------|-------|
| **Subnet name** | `public-subnet-1b` |
| **Availability Zone** | `us-east-1b` |
| **IPv4 CIDR block** | `10.0.2.0/24` |

Click **"Add new subnet"** button

### 3.4 Create Private Subnet 1

**Subnet 3 of 4 - Private Subnet 1a:**

| Field | Value |
|-------|-------|
| **Subnet name** | `private-subnet-1a` |
| **Availability Zone** | `us-east-1a` |
| **IPv4 CIDR block** | `10.0.11.0/24` |

Click **"Add new subnet"** button

### 3.5 Create Private Subnet 2

**Subnet 4 of 4 - Private Subnet 1b:**

| Field | Value |
|-------|-------|
| **Subnet name** | `private-subnet-1b` |
| **Availability Zone** | `us-east-1b` |
| **IPv4 CIDR block** | `10.0.12.0/24` |

### 3.6 Create All Subnets

1. Review all 4 subnets
2. Click **"Create subnet"** button at the bottom
3. Wait for all subnets to be created

**✅ Checkpoint**: You should see 4 new subnets in your subnet list

### 3.7 Enable Auto-assign Public IP for Public Subnets

For **public-subnet-1a**:
1. Select the subnet (checkbox)
2. Click **"Actions"** dropdown
3. Select **"Edit subnet settings"**
4. Check ✅ **"Enable auto-assign public IPv4 address"**
5. Click **"Save"**

Repeat for **public-subnet-1b**

**Why?** This ensures instances in public subnets get public IP addresses automatically.

---

## Step 4: Create Route Tables

### 4.1 Create Public Route Table

1. In the left sidebar, click **"Route Tables"**
2. Click **"Create route table"** button

**Route Table Details:**

| Field | Value |
|-------|-------|
| **Name** | `public-route-table` |
| **VPC** | Select `my-asg-vpc` |

**Tags** (optional):
| Key | Value |
|-----|-------|
| Project | AutoScaling-Demo |
| Type | Public |

3. Click **"Create route table"**

### 4.2 Add Internet Route to Public Route Table

1. Select **"public-route-table"** (checkbox)
2. Click on **"Routes"** tab (bottom panel)
3. Click **"Edit routes"** button

**Add Route:**

| Destination | Target | Description |
|-------------|--------|-------------|
| `0.0.0.0/0` | Internet Gateway → `my-asg-igw` | Route all traffic to internet |

4. Click **"Save changes"**

### 4.3 Associate Public Subnets

1. Still on **"public-route-table"**
2. Click **"Subnet associations"** tab
3. Click **"Edit subnet associations"** button
4. Check ✅ both:
   - `public-subnet-1a`
   - `public-subnet-1b`
5. Click **"Save associations"**

**✅ Checkpoint**: Public route table routes traffic to internet gateway

### 4.4 Create Private Route Table

1. Click **"Create route table"** button again

**Route Table Details:**

| Field | Value |
|-------|-------|
| **Name** | `private-route-table` |
| **VPC** | Select `my-asg-vpc` |

**Tags** (optional):
| Key | Value |
|-----|-------|
| Project | AutoScaling-Demo |
| Type | Private |

2. Click **"Create route table"**

### 4.5 Associate Private Subnets

1. Select **"private-route-table"**
2. Click **"Subnet associations"** tab
3. Click **"Edit subnet associations"** button
4. Check ✅ both:
   - `private-subnet-1a`
   - `private-subnet-1b`
5. Click **"Save associations"**

**Note**: Private route table has no internet route. Instances in private subnets cannot access the internet directly.

---

## Step 5: Create NAT Gateway (Optional)

**Important**: NAT Gateway costs ~$32-45/month. For this basic demo, we can skip it.

If your EC2 instances need to download packages or updates from the internet:

### 5.1 Allocate Elastic IP

1. In the left sidebar, click **"Elastic IPs"**
2. Click **"Allocate Elastic IP address"**
3. Click **"Allocate"**
4. Note the Elastic IP address

### 5.2 Create NAT Gateway

1. In the left sidebar, click **"NAT Gateways"**
2. Click **"Create NAT gateway"**

**Configuration:**

| Field | Value |
|-------|-------|
| **Name** | `my-asg-nat` |
| **Subnet** | `public-subnet-1a` (must be public) |
| **Elastic IP allocation ID** | Select the EIP you created |

3. Click **"Create NAT gateway"**

### 5.3 Update Private Route Table

1. Go to **"Route Tables"**
2. Select **"private-route-table"**
3. Click **"Routes"** tab → **"Edit routes"**
4. Click **"Add route"**:
   - Destination: `0.0.0.0/0`
   - Target: NAT Gateway → `my-asg-nat`
5. Click **"Save changes"**

---

## Verification Checklist

Use this checklist to verify your VPC setup:

### VPC
- [ ] VPC created with CIDR `10.0.0.0/16`
- [ ] DNS resolution enabled
- [ ] DNS hostnames enabled
- [ ] VPC ID noted down

### Internet Gateway
- [ ] Internet Gateway created
- [ ] Attached to VPC
- [ ] Status shows "Attached"

### Subnets
- [ ] 4 subnets created (2 public, 2 private)
- [ ] Public subnets in different AZs (1a, 1b)
- [ ] Private subnets in different AZs (1a, 1b)
- [ ] Auto-assign public IP enabled for public subnets
- [ ] All subnet IDs noted

### Route Tables
- [ ] Public route table created
- [ ] Public route table has route to Internet Gateway (0.0.0.0/0)
- [ ] Public subnets associated with public route table
- [ ] Private route table created
- [ ] Private subnets associated with private route table

---

## Network Diagram

```
Internet
    ↓
Internet Gateway (my-asg-igw)
    ↓
┌─────────────────────────────────────┐
│   VPC (10.0.0.0/16)                 │
│                                      │
│  ┌──────────────────────────────┐  │
│  │  Public Route Table           │  │
│  │  Route: 0.0.0.0/0 → IGW      │  │
│  └──────────────────────────────┘  │
│           │              │          │
│  ┌────────┴─────┐  ┌────┴──────┐  │
│  │ public-      │  │ public-    │  │
│  │ subnet-1a    │  │ subnet-1b  │  │
│  │ 10.0.1.0/24  │  │ 10.0.2.0/24│  │
│  │ (us-east-1a) │  │(us-east-1b)│  │
│  └──────────────┘  └────────────┘  │
│                                      │
│  ┌──────────────────────────────┐  │
│  │  Private Route Table          │  │
│  │  (No internet route)          │  │
│  └──────────────────────────────┘  │
│           │              │          │
│  ┌────────┴─────┐  ┌────┴──────┐  │
│  │ private-     │  │ private-   │  │
│  │ subnet-1a    │  │ subnet-1b  │  │
│  │ 10.0.11.0/24 │  │10.0.12.0/24│  │
│  │ (us-east-1a) │  │(us-east-1b)│  │
│  └──────────────┘  └────────────┘  │
└─────────────────────────────────────┘
```

---

## Common Issues and Solutions

### Issue 1: Can't create subnets

**Error**: "The CIDR block conflicts with another subnet"

**Solution**: Make sure each subnet has a unique CIDR block that doesn't overlap.

### Issue 2: Can't attach Internet Gateway

**Error**: "Resource already has an attached gateway"

**Solution**: Each VPC can have only one Internet Gateway. Check if one is already attached.

### Issue 3: Wrong Availability Zones

**Solution**: Make sure you're using availability zones that exist in your region. Use `us-east-1a` and `us-east-1b` for US East (N. Virginia).

### Issue 4: Subnet CIDR too small

**Solution**: Use at least /24 (256 IPs) for each subnet. AWS reserves 5 IPs in each subnet.

---

## Cost Information

**VPC Components Costs:**

| Component | Cost |
|-----------|------|
| VPC | FREE |
| Subnets | FREE |
| Internet Gateway | FREE |
| Route Tables | FREE |
| NAT Gateway | $0.045/hour (~$32/month) + data transfer |
| Elastic IP (if attached) | FREE |
| Elastic IP (if NOT attached) | $0.005/hour |

**💡 Cost Tip**: Skip NAT Gateway for this demo to save ~$32/month!

---

## Next Steps

Now that your VPC is configured, proceed to:

➡️ **[02-security-groups.md](02-security-groups.md)** - Create Security Groups

---

## Quick Reference

### VPC CIDR Breakdown

| Subnet | CIDR | IPs Available | Purpose |
|--------|------|---------------|---------|
| VPC | 10.0.0.0/16 | 65,536 | Entire network |
| public-subnet-1a | 10.0.1.0/24 | 251 | ALB in AZ-1a |
| public-subnet-1b | 10.0.2.0/24 | 251 | ALB in AZ-1b |
| private-subnet-1a | 10.0.11.0/24 | 251 | EC2 in AZ-1a |
| private-subnet-1b | 10.0.12.0/24 | 251 | EC2 in AZ-1b |

**Note**: AWS reserves 5 IP addresses in each subnet (first 4 and last 1).

---

**✅ VPC Setup Complete!** Your network infrastructure is now ready for security groups and resources.
