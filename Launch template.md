# 03 - Launch Template Setup Guide

## Overview

A Launch Template defines the configuration for EC2 instances in an Auto Scaling Group. It includes the AMI, instance type, security groups, and user data script that runs when instances launch.

## What We'll Create

- **Launch Template** with:
  - Amazon Linux 2023 AMI
  - t2.micro instance type (Free Tier)
  - Automatic web server installation
  - Beautiful HTML webpage showing instance information

---

## Step 1: Navigate to Launch Templates

### 1.1 Open EC2 Dashboard

1. Go to **AWS Management Console**
2. Search for **"EC2"** and click on EC2 service
3. In the left sidebar, scroll to **"Instances"** section
4. Click **"Launch Templates"**

### 1.2 Create Launch Template

1. Click **"Create launch template"** button (orange button, top right)

---

## Step 2: Launch Template Name and Description

### 2.1 Basic Information

| Field | Value | Description |
|-------|-------|-------------|
| **Launch template name** | `web-server-template` | Name for your template |
| **Template version description** | `v1.0 - Initial web server template` | Version description |
| **Template tags** | | Optional tags |

### 2.2 Add Template Tags (Optional)

Click **"Add tag"**:

| Key | Value |
|-----|-------|
| Project | AutoScaling-Demo |
| Environment | Development |
| ManagedBy | Manual |

**Note**: These tags apply to the template itself, not the instances.

### 2.3 Auto Scaling Guidance

✅ Check **"Provide guidance to help me set up a template that I can use with EC2 Auto Scaling"**

**Why?** This enables Auto Scaling-specific options and validates your configuration.

---

## Step 3: Application and OS Images (AMI)

### 3.1 Quick Start AMI

1. Click on **"Quick Start"** tab (should be selected by default)
2. Select **"Amazon Linux"**
3. Choose **"Amazon Linux 2023 AMI"** (or Amazon Linux 2)

**AMI Details**:
- **AMI ID**: ami-xxxxxxxxx (auto-selected, latest version)
- **Architecture**: 64-bit (x86)
- **Root device type**: EBS
- **Virtualization**: HVM

**Why Amazon Linux 2023?**
- ✅ Free to use
- ✅ Optimized for AWS
- ✅ Regular security updates
- ✅ Pre-installed AWS tools
- ✅ Lightweight and fast

### 3.2 Alternative AMIs (Optional)

If you prefer a different OS:

| OS | AMI Name | Use Case |
|---|----------|----------|
| Amazon Linux 2 | Amazon Linux 2 AMI | Widely supported, stable |
| Ubuntu | Ubuntu Server 22.04 LTS | Popular, large community |
| Red Hat | RHEL 9 | Enterprise support |
| Windows | Windows Server 2022 | Windows applications |

**For this tutorial, use Amazon Linux 2023.**

---

## Step 4: Instance Type

### 4.1 Choose Instance Type

| Field | Value |
|-------|-------|
| **Instance type** | `t2.micro` |

**Click** the dropdown to see other options, but select **t2.micro**.

**Why t2.micro?**
- ✅ **Free Tier Eligible**: 750 hours/month for 12 months
- ✅ **Cost-effective**: ~$0.0116/hour ($8.50/month) after free tier
- ✅ **Sufficient for demo**: 1 vCPU, 1 GB RAM
- ✅ **Burstable performance**: Can handle traffic spikes

### 4.2 Instance Type Comparison

| Type | vCPU | RAM | Cost/hour | Best For |
|------|------|-----|-----------|----------|
| t2.micro | 1 | 1 GB | $0.0116 | Demo, low traffic |
| t2.small | 1 | 2 GB | $0.023 | Small apps |
| t3.micro | 2 | 1 GB | $0.0104 | Better performance |
| t3.small | 2 | 2 GB | $0.0208 | Medium traffic |

---

## Step 5: Key Pair (Login)

### 5.1 Select Key Pair

**Option 1: Use Existing Key Pair**
- If you have a key pair: Select it from dropdown
- You'll need the .pem file for SSH access

**Option 2: Create New Key Pair**
1. Click **"Create new key pair"**
2. Name: `my-asg-keypair`
3. Type: **RSA**
4. Format: **.pem** (for Mac/Linux) or **.ppk** (for Windows/PuTTY)
5. Click **"Create key pair"**
6. **Save the file** - you can't download it again!

**Option 3: Proceed Without Key Pair**
- Select **"Proceed without a key pair"**
- ⚠️ You won't be able to SSH into instances
- ✅ Use AWS Systems Manager Session Manager instead (recommended)

**Recommendation**: For this demo, select "Proceed without a key pair" if you don't need SSH access.

---

## Step 6: Network Settings

### 6.1 Network Configuration

**Important**: Do **NOT** include subnet in launch template.

| Field | Value | Why |
|-------|-------|-----|
| **Subnet** | **Don't include in launch template** | Auto Scaling Group will choose |
| **Firewall (security groups)** | Select existing | Choose ec2-security-group |

### 6.2 Security Groups

1. Under **"Firewall (security groups)"**, select **"Select existing security group"**
2. Click in the search box
3. Type `ec2-security-group`
4. ✅ **Check** the security group we created earlier
5. **Remove** the default security group if it's selected

**Selected Security Group**:
- `ec2-security-group (sg-xxxxxxxxxxxxx)`

### 6.3 Advanced Network Configuration (Optional)

Leave these as default:
- **Network interface**: (empty)
- **Auto-assign public IP**: (empty - controlled by subnet)

---

## Step 7: Storage (Volumes)

### 7.1 Root Volume Configuration

Keep the default settings:

| Field | Default Value | Description |
|-------|---------------|-------------|
| **Volume type** | gp3 | General Purpose SSD (newer, faster) |
| **Size** | 8 GiB | Sufficient for OS and web server |
| **IOPS** | 3000 | Input/Output Operations Per Second |
| **Throughput** | 125 MiB/s | Data transfer rate |
| **Delete on termination** | Yes | Remove volume when instance terminates |
| **Encrypted** | Not encrypted | (Can enable for production) |

### 7.2 Storage Types Comparison

| Type | Use Case | Cost |
|------|----------|------|
| **gp3** | General purpose (recommended) | $0.08/GB-month |
| **gp2** | General purpose (older) | $0.10/GB-month |
| **io1/io2** | High performance | $0.125/GB-month + IOPS cost |
| **st1** | Big data, data warehouses | $0.045/GB-month |

**For this demo, keep the default gp3 with 8 GiB.**

---

## Step 8: Resource Tags

### 8.1 Add Instance Tags

Tags help organize and identify your instances.

Click **"Add tag"** button:

| Key | Value | Tag instances | Tag volumes |
|-----|-------|---------------|-------------|
| Name | `web-server-instance` | ✅ Yes | ✅ Yes |
| Project | AutoScaling-Demo | ✅ Yes | ❌ No |
| Environment | Development | ✅ Yes | ❌ No |

**Important**: Check **"Tag instances"** and **"Tag volumes"** for the Name tag.

---

## Step 9: Advanced Details

### 9.1 Expand Advanced Details

Click **"Advanced details"** to expand this section.

### 9.2 IAM Instance Profile (Optional but Recommended)

If you need instances to access AWS services (S3, CloudWatch, etc.):

1. **IAM instance profile**: Select or create one
2. For this demo, you can leave empty or create a basic role

**To create IAM role** (if needed):
1. Open IAM console in new tab
2. Create role with `AmazonSSMManagedInstanceCore` policy
3. Come back and select it

### 9.3 Monitoring

| Field | Value |
|-------|-------|
| **Detailed monitoring** | ❌ Disabled (optional, costs extra) |

**Note**: Basic monitoring (5-minute intervals) is free. Detailed monitoring (1-minute intervals) costs $0.14 per instance per month.

### 9.4 User Data

This is where we define what happens when an instance launches.

Scroll to **"User data"** text box and paste this script:

```bash
#!/bin/bash

# ============================================
# Web Server Setup Script
# Installs Apache and creates demo webpage
# ============================================

# Enable error handling
set -e

# Update system packages
echo "Updating system packages..."
dnf update -y

# Install Apache web server
echo "Installing Apache web server..."
dnf install -y httpd

# Start Apache and enable on boot
echo "Starting Apache..."
systemctl start httpd
systemctl enable httpd

# Get instance metadata
echo "Fetching instance metadata..."
INSTANCE_ID=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)
AVAILABILITY_ZONE=$(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone)
PRIVATE_IP=$(curl -s http://169.254.169.254/latest/meta-data/local-ipv4)
REGION=$(curl -s http://169.254.169.254/latest/meta-data/placement/region)
PUBLIC_IP=$(curl -s http://169.254.169.254/latest/meta-data/public-ipv4 || echo "N/A")

# Create beautiful HTML webpage
echo "Creating webpage..."
cat > /var/www/html/index.html <<'HTMLEOF'
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="refresh" content="30">
    <title>AWS Auto Scaling Demo</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
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
            padding: 50px;
            box-shadow: 0 20px 60px rgba(0,0,0,0.3);
            max-width: 800px;
            width: 100%;
            animation: slideUp 0.5s ease-out;
        }
        
        @keyframes slideUp {
            from {
                opacity: 0;
                transform: translateY(30px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }
        
        h1 {
            color: #667eea;
            text-align: center;
            margin-bottom: 10px;
            font-size: 2.5em;
        }
        
        .subtitle {
            text-align: center;
            color: #666;
            margin-bottom: 30px;
            font-size: 1.1em;
        }
        
        .status {
            text-align: center;
            margin: 20px 0;
        }
        
        .status-badge {
            display: inline-block;
            background: #10b981;
            color: white;
            padding: 12px 40px;
            border-radius: 50px;
            font-weight: bold;
            font-size: 1.1em;
            animation: pulse 2s infinite;
        }
        
        @keyframes pulse {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.8; }
        }
        
        .info-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 20px;
            margin: 30px 0;
        }
        
        .info-box {
            background: #f8f9fa;
            padding: 25px;
            border-radius: 15px;
            border-left: 5px solid #667eea;
            transition: transform 0.3s ease, box-shadow 0.3s ease;
        }
        
        .info-box:hover {
            transform: translateY(-5px);
            box-shadow: 0 10px 30px rgba(102, 126, 234, 0.2);
        }
        
        .info-box h3 {
            color: #667eea;
            margin-bottom: 10px;
            font-size: 1.1em;
            display: flex;
            align-items: center;
            gap: 10px;
        }
        
        .info-box p {
            color: #333;
            font-size: 1em;
            font-weight: 500;
            word-break: break-all;
        }
        
        .icon {
            font-size: 1.5em;
        }
        
        .features {
            background: #f8f9fa;
            padding: 25px;
            border-radius: 15px;
            margin-top: 30px;
        }
        
        .features h3 {
            color: #667eea;
            margin-bottom: 15px;
            font-size: 1.3em;
        }
        
        .features ul {
            list-style: none;
            padding: 0;
        }
        
        .features li {
            padding: 12px;
            margin: 8px 0;
            background: white;
            border-radius: 8px;
            transition: all 0.3s ease;
        }
        
        .features li:hover {
            background: #667eea;
            color: white;
            transform: translateX(10px);
        }
        
        .features li:before {
            content: "✓ ";
            color: #10b981;
            font-weight: bold;
            margin-right: 10px;
            font-size: 1.2em;
        }
        
        .features li:hover:before {
            color: white;
        }
        
        .footer {
            text-align: center;
            margin-top: 30px;
            color: #666;
            font-size: 0.9em;
            padding-top: 20px;
            border-top: 2px solid #e0e0e0;
        }
        
        .refresh-info {
            background: #fff3cd;
            border: 1px solid #ffc107;
            color: #856404;
            padding: 10px;
            border-radius: 8px;
            text-align: center;
            margin-top: 20px;
            font-size: 0.9em;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🚀 AWS Auto Scaling Demo</h1>
        <p class="subtitle">High Availability Web Application</p>
        
        <div class="status">
            <span class="status-badge">✓ Server Running</span>
        </div>

        <div class="info-grid">
            <div class="info-box">
                <h3><span class="icon">🔧</span> Instance ID</h3>
                <p id="instance-id">Loading...</p>
            </div>

            <div class="info-box">
                <h3><span class="icon">📍</span> Availability Zone</h3>
                <p id="az">Loading...</p>
            </div>

            <div class="info-box">
                <h3><span class="icon">🌐</span> Private IP</h3>
                <p id="private-ip">Loading...</p>
            </div>

            <div class="info-box">
                <h3><span class="icon">🌍</span> Region</h3>
                <p id="region">Loading...</p>
            </div>
        </div>

        <div class="features">
            <h3>🏗️ Architecture Features:</h3>
            <ul>
                <li>Application Load Balancer distributing traffic</li>
                <li>Auto Scaling Group with 2-4 instances</li>
                <li>Multi-AZ deployment for high availability</li>
                <li>CPU-based automatic scaling policies</li>
                <li>Health checks and automatic recovery</li>
                <li>CloudWatch monitoring and alarms</li>
                <li>Secure VPC with public and private subnets</li>
            </ul>
        </div>

        <div class="refresh-info">
            ℹ️ Refresh this page multiple times to see different instances serving requests
        </div>

        <div class="footer">
            <p><strong>Powered by AWS Auto Scaling</strong></p>
            <p>This page auto-refreshes every 30 seconds</p>
        </div>
    </div>

    <script>
        // Fetch and display instance metadata
        function fetchMetadata(url, elementId) {
            fetch(url)
                .then(response => response.text())
                .then(data => {
                    document.getElementById(elementId).textContent = data;
                })
                .catch(error => {
                    document.getElementById(elementId).textContent = 'N/A';
                });
        }

        // Fetch all metadata
        fetchMetadata('http://169.254.169.254/latest/meta-data/instance-id', 'instance-id');
        fetchMetadata('http://169.254.169.254/latest/meta-data/placement/availability-zone', 'az');
        fetchMetadata('http://169.254.169.254/latest/meta-data/local-ipv4', 'private-ip');
        fetchMetadata('http://169.254.169.254/latest/meta-data/placement/region', 'region');
    </script>
</body>
</html>
HTMLEOF

# Replace placeholders with actual values (for initial load)
sed -i "s/Loading.../$INSTANCE_ID/g" /var/www/html/index.html

# Set correct permissions
echo "Setting permissions..."
chown -R apache:apache /var/www/html
chmod -R 755 /var/www/html

# Configure firewall (if firewalld is installed)
if systemctl is-active --quiet firewalld; then
    echo "Configuring firewall..."
    firewall-cmd --permanent --add-service=http
    firewall-cmd --reload
fi

# Create a health check file
echo "OK" > /var/www/html/health.html

# Log completion
echo "Web server setup completed successfully!" >> /var/log/user-data.log
echo "Instance ID: $INSTANCE_ID" >> /var/log/user-data.log
echo "Private IP: $PRIVATE_IP" >> /var/log/user-data.log

# Display status
systemctl status httpd --no-pager

echo "Setup complete! Apache is running."
```

**What this script does:**
1. ✅ Updates system packages
2. ✅ Installs Apache web server
3. ✅ Starts and enables Apache
4. ✅ Fetches instance metadata (ID, AZ, IP)
5. ✅ Creates beautiful HTML webpage
6. ✅ Sets proper permissions
7. ✅ Configures firewall
8. ✅ Creates health check file

---

## Step 10: Review and Create

### 10.1 Review Summary

Scroll down to review all settings:

| Section | Configuration |
|---------|---------------|
| **Template name** | web-server-template |
| **AMI** | Amazon Linux 2023 |
| **Instance type** | t2.micro |
| **Key pair** | Your choice |
| **Security groups** | ec2-security-group |
| **Storage** | 8 GiB gp3 |
| **User data** | Web server setup script |

### 10.2 Create Launch Template

1. Scroll to the bottom
2. Click **"Create launch template"** button
3. Wait for success message: "Successfully created launch template"
4. Note the **Launch template ID** (lt-xxxxxxxxxxxxx)

**✅ Checkpoint**: Launch Template created successfully!

---

## Step 11: Verify Launch Template

### 11.1 View Template Details

1. Click **"View launch template"** button (or go back to Launch Templates list)
2. Select your template: `web-server-template`
3. Review:
   - **Latest version**: 1
   - **Default version**: 1
   - **Template ID**: lt-xxxxxxxxxxxxx

### 11.2 Template Actions

You can:
- **Create new version**: Update configuration
- **Set default version**: Choose which version to use
- **Delete template**: Remove template (careful!)
- **Launch instance**: Test launch manually

---

## Testing the Launch Template (Optional)

### Test Launch an Instance

1. Select your launch template
2. Click **"Actions"** → **"Launch instance from template"**
3. Configure:
   - **Name**: `test-instance`
   - **Number of instances**: 1
   - **Network settings** → **Subnet**: Select any public subnet
4. Click **"Launch instance"**
5. Wait 2-3 minutes for instance to launch
6. Go to **Instances** → Find your test instance
7. Copy **Public IP address**
8. Open in browser: `http://[public-ip]`
9. You should see your beautiful webpage!

**Cleanup**:
- Terminate the test instance after verification
- We'll use Auto Scaling Group for actual deployment

---

## Troubleshooting Launch Template

### Issue 1: User data script failed

**Check logs**:
```bash
# SSH into instance
ssh -i your-key.pem ec2-user@instance-ip

# Check user data log
sudo cat /var/log/cloud-init-output.log

# Check Apache status
sudo systemctl status httpd

# Check error logs
sudo cat /var/log/httpd/error_log
```

### Issue 2: Can't access webpage

**Verify**:
1. Instance is running
2. Security group allows port 80
3. Apache is running: `sudo systemctl status httpd`
4. Using HTTP not HTTPS: `http://` not `https://`

### Issue 3: Metadata not loading

**Reason**: Metadata service is blocked or instance role missing

**Fix**:
- Ensure IMDSv2 is not required (or update script)
- Check network settings

---

## Launch Template Best Practices

### ✅ Do's

1. **Use Latest AMI**
   - Regularly update to latest AMI version
   - Includes security patches

2. **Version Control**
   - Create new versions instead of editing
   - Makes rollback easy

3. **Test User Data**
   - Test scripts on a single instance first
   - Check logs for errors

4. **Use IAM Roles**
   - Never put credentials in user data
   - Use instance profiles for AWS access

5. **Document Changes**
   - Add version descriptions
   - Note what changed

### ❌ Don'ts

1. **Don't Include Subnets**
   - Let Auto Scaling Group choose
   - Allows multi-AZ deployment

2. **Don't Hardcode Values**
   - Use metadata service
   - Use parameter store for configs

3. **Don't Include Sensitive Data**
   - Use AWS Secrets Manager
   - Use Systems Manager Parameter Store

4. **Don't Skip Testing**
   - Always test user data scripts
   - Verify all components work

---

## Launch Template Checklist

Use this checklist to verify your launch template:

### Basic Configuration
- [ ] Template name: `web-server-template`
- [ ] Version description added
- [ ] Auto Scaling guidance enabled

### Compute
- [ ] AMI: Amazon Linux 2023
- [ ] Instance type: t2.micro
- [ ] Key pair selected (or proceeding without)

### Network
- [ ] Subnet NOT included in template
- [ ] Security group: ec2-security-group selected
- [ ] Default SG removed

### Storage
- [ ] Volume type: gp3
- [ ] Size: 8 GiB
- [ ] Delete on termination: Yes

### Tags
- [ ] Name tag configured
- [ ] Tag instances enabled
- [ ] Additional tags added

### Advanced
- [ ] User data script added
- [ ] IAM instance profile (if needed)
- [ ] Monitoring configured

### Verification
- [ ] Template created successfully
- [ ] Template ID noted
- [ ] Test launch successful (optional)

---

## Next Steps

Launch Template is now ready! Proceed to:

➡️ **[04-target-group.md](04-target-group.md)** - Create Target Group for Load Balancer

---

## Quick Reference

### Launch Template Details

| Component | Value |
|-----------|-------|
| **Template Name** | web-server-template |
| **Template ID** | lt-xxxxxxxxxxxxx |
| **AMI** | Amazon Linux 2023 |
| **Instance Type** | t2.micro |
| **Security Group** | ec2-security-group |
| **Storage** | 8 GiB gp3 |

### User Data Script Purpose

1. System updates
2. Apache installation
3. Service configuration
4. Webpage creation
5. Health check setup
6. Permissions configuration

---

**✅ Launch Template Setup Complete!** Your EC2 instance configuration is ready for Auto Scaling.
