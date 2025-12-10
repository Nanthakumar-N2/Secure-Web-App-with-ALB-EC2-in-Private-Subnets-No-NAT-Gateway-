# 05 - Application Load Balancer Setup

## Overview
Create an internet-facing Application Load Balancer to distribute traffic across EC2 instances in multiple availability zones.

---

## Step 1: Navigate to Load Balancers
1. Go to **EC2 Dashboard** → **Load Balancers**
2. Click **"Create load balancer"**

## Step 2: Select Load Balancer Type
- Choose **"Application Load Balancer"**
- Click **"Create"**

## Step 3: Basic Configuration
| Field | Value |
|-------|-------|
| **Load balancer name** | `web-app-alb` |
| **Scheme** | Internet-facing |
| **IP address type** | IPv4 |

## Step 4: Network Mapping
- **VPC**: Select `my-asg-vpc`
- **Mappings**: Select BOTH availability zones
  - ✅ us-east-1a → `public-subnet-1a`
  - ✅ us-east-1b → `public-subnet-1b`

## Step 5: Security Groups
- Remove default security group
- Select `alb-security-group`

## Step 6: Listeners and Routing
| Field | Value |
|-------|-------|
| **Protocol** | HTTP |
| **Port** | 80 |
| **Default action** | Forward to → `web-servers-tg` |

## Step 7: Create Load Balancer
1. Review all settings
2. Click **"Create load balancer"**
3. Wait 2-3 minutes for state to become **"Active"**
4. **Note the DNS name** (e.g., web-app-alb-123456789.us-east-1.elb.amazonaws.com)

✅ **ALB Created!**

---

# 06 - Auto Scaling Group Setup

## Overview
Create an Auto Scaling Group to automatically manage EC2 instances based on demand.

---

## Step 1: Navigate to Auto Scaling Groups
1. Go to **EC2 Dashboard** → **Auto Scaling Groups**
2. Click **"Create Auto Scaling group"**

## Step 2: Choose Launch Template
| Field | Value |
|-------|-------|
| **Name** | `web-servers-asg` |
| **Launch template** | web-server-template |
| **Version** | Latest (1) |

Click **"Next"**

## Step 3: Choose Instance Launch Options
- **VPC**: `my-asg-vpc`
- **Availability Zones and subnets**: Select BOTH private subnets
  - ✅ `private-subnet-1a`
  - ✅ `private-subnet-1b`

Click **"Next"**

## Step 4: Configure Advanced Options
**Load balancing**:
- ✅ Attach to an existing load balancer
- Choose from your load balancer target groups
- Select `web-servers-tg`

**Health checks**:
- Type: ✅ ELB
- Grace period: `300` seconds

**Monitoring**:
- ✅ Enable group metrics collection within CloudWatch

Click **"Next"**

## Step 5: Configure Group Size and Scaling
**Group size**:
| Field | Value |
|-------|-------|
| Desired capacity | 2 |
| Minimum capacity | 2 |
| Maximum capacity | 4 |

**Scaling policies**:
- ✅ Target tracking scaling policy
- **Policy name**: `cpu-target-tracking`
- **Metric type**: Average CPU utilization
- **Target value**: `70`
- **Instances need**: 300 seconds warm up

Click **"Next"**

## Step 6: Add Notifications (Optional)
- Skip for now
- Click **"Next"**

## Step 7: Add Tags
| Key | Value | Tag instances |
|-----|-------|---------------|
| Name | web-server-instance | ✅ Yes |
| Project | AutoScaling-Demo | ✅ Yes |

Click **"Next"**

## Step 8: Review and Create
1. Review all settings
2. Click **"Create Auto Scaling group"**
3. Wait for instances to launch (2-3 minutes)

✅ **Auto Scaling Group Created!**

## Step 9: Verify Deployment
1. Go to **EC2** → **Instances**
2. You should see 2 new instances launching
3. Go to **Target Groups** → `web-servers-tg` → **Targets** tab
4. Wait for targets to become "Healthy" (2-3 minutes)
5. Copy **ALB DNS name**
6. Open in browser: `http://[alb-dns-name]`
7. You should see your webpage!

---

# 07 - Testing Guide

## Test 1: Load Balancing

### Verify Traffic Distribution
1. Open ALB URL in browser
2. Note the **Instance ID** on the page
3. Refresh the page multiple times (F5)
4. You should see different Instance IDs
5. This proves load balancing is working!

**Expected Result**:
```
Refresh 1: Instance ID: i-0abc123...
Refresh 2: Instance ID: i-0def456...
Refresh 3: Instance ID: i-0abc123...
```

---

## Test 2: High Availability

### Terminate an Instance
1. Go to **EC2** → **Instances**
2. Select one instance from your ASG
3. **Instance State** → **Terminate instance**
4. Confirm termination

### Observe Auto Scaling
1. Go to **Auto Scaling Groups** → `web-servers-asg`
2. Click **"Activity"** tab
3. Watch ASG launch a new instance
4. Time: ~3-5 minutes
5. Go to **Target Groups** → Check health
6. New instance should become healthy

**What Happened**:
- ASG detected instance count < desired (2)
- Automatically launched a replacement
- Registered with target group
- Health checks passed
- Started receiving traffic

✅ **High Availability Verified!**

---

## Test 3: Auto Scaling (Scale Out)

### Generate CPU Load

**Method 1: Using Stress Tool**

SSH into an instance:
```bash
# Install stress
sudo yum install -y stress

# Create CPU load (8 workers for 5 minutes)
stress --cpu 8 --timeout 300
```

**Method 2: Apache Bench (Load Testing)**

From your computer:
```bash
# Install Apache Bench
# Mac: Already installed
# Linux: sudo apt-get install apache2-utils
# Windows: Download from Apache website

# Send 10,000 requests with 100 concurrent
ab -n 10000 -c 100 http://[ALB-DNS-NAME]/
```

**Method 3: Multiple Browser Tabs**

Simple but effective:
1. Open ALB URL in 20+ browser tabs
2. Keep refreshing all tabs rapidly
3. Not as effective but can trigger scaling

### Watch Scaling Activity

1. Go to **Auto Scaling Groups** → `web-servers-asg`
2. Click **"Activity"** tab
3. Watch for "Launching a new EC2 instance"
4. Go to **Monitoring** tab
5. Watch CPU utilization graph
6. When CPU > 70%, ASG will add instance
7. New instance launches in 3-5 minutes

**Scaling Timeline**:
```
Time 0:   CPU hits 70%
Time 60s: Alarm triggers (1 minute wait)
Time 90s: ASG decides to scale
Time 93s: Instance launching
Time 5m:  Instance registered
Time 8m:  Instance healthy
```

✅ **Scale Out Successful!**

---

## Test 4: Auto Scaling (Scale In)

### Stop Load
1. Stop the stress test (Ctrl+C)
2. Stop Apache Bench
3. Wait for CPU to drop

### Watch Scale In
1. Wait 10-15 minutes
2. CPU drops below 30%
3. ASG will terminate one instance
4. Instance count returns to 2 (desired capacity)

**Important**: Scale in is slower to prevent "flapping"

✅ **Scale In Successful!**

---

## Test 5: Health Checks

### Make Instance Unhealthy

SSH into an instance:
```bash
# Stop Apache web server
sudo systemctl stop httpd

# Check status
sudo systemctl status httpd
```

### Observe Recovery
1. Go to **Target Groups** → **Targets** tab
2. Watch instance become "Unhealthy" (60-90 seconds)
3. ALB stops sending traffic to it
4. ASG terminates unhealthy instance
5. ASG launches replacement
6. New instance becomes healthy

✅ **Health Check and Recovery Verified!**

---

## Test 6: Multi-AZ Deployment

### Verify Multi-AZ
1. Open ALB URL
2. Refresh multiple times
3. Note the **Availability Zone** on webpage
4. You should see both zones:
   - us-east-1a
   - us-east-1b

### Simulate AZ Failure
1. Go to **Instances**
2. Terminate ALL instances in one AZ
3. ASG launches replacements in available AZ
4. Application remains available

✅ **Multi-AZ Verified!**

---

## Performance Testing

### Response Time Test
```bash
# Test response time
curl -o /dev/null -s -w "Time: %{time_total}s\n" http://[ALB-DNS]

# Expected: < 0.5 seconds
```

### Load Test with Apache Bench
```bash
# 1000 requests, 10 concurrent
ab -n 1000 -c 10 http://[ALB-DNS]/

# Review results:
# - Requests per second
# - Time per request
# - Failed requests (should be 0)
```

### Sustained Load Test
```bash
# 10,000 requests, 100 concurrent
ab -n 10000 -c 100 http://[ALB-DNS]/

# Monitor in CloudWatch:
# - CPU utilization
# - Request count
# - Target response time
```

---

# 08 - Monitoring and CloudWatch

## CloudWatch Metrics

### Key Metrics to Monitor

**Application Load Balancer**:
| Metric | Description | Good Value |
|--------|-------------|------------|
| ActiveConnectionCount | Number of active connections | Varies |
| HealthyHostCount | Healthy targets | ≥ 2 |
| UnHealthyHostCount | Unhealthy targets | 0 |
| RequestCount | Total requests | Varies |
| TargetResponseTime | Response time | < 1 second |
| HTTPCode_Target_2XX | Successful responses | > 99% |
| HTTPCode_Target_4XX | Client errors | < 1% |
| HTTPCode_Target_5XX | Server errors | < 0.1% |

**Auto Scaling Group**:
| Metric | Description | Good Value |
|--------|-------------|------------|
| GroupDesiredCapacity | Desired instance count | 2 |
| GroupInServiceInstances | Running instances | 2-4 |
| GroupMinSize | Minimum instances | 2 |
| GroupMaxSize | Maximum instances | 4 |

**EC2 Instances**:
| Metric | Description | Good Value |
|--------|-------------|------------|
| CPUUtilization | CPU usage | 30-70% |
| NetworkIn | Incoming traffic | Varies |
| NetworkOut | Outgoing traffic | Varies |
| StatusCheckFailed | Failed health checks | 0 |

---

## Setting Up CloudWatch Dashboard

### Create Dashboard
1. Go to **CloudWatch** → **Dashboards**
2. Click **"Create dashboard"**
3. Name: `Auto-Scaling-Dashboard`
4. Click **"Create dashboard"**

### Add Widgets

**Widget 1: ALB Request Count**
1. Click **"Add widget"**
2. Choose **"Line"** graph
3. Select **"Metrics"**
4. Choose **"ApplicationELB"** → **"Per AppELB Metrics"**
5. Select your ALB
6. Check `RequestCount`
7. **Period**: 1 minute
8. Click **"Create widget"**

**Widget 2: Healthy Hosts**
1. Add widget → Line graph
2. **ApplicationELB** → **"Per AppELB, per TG Metrics"**
3. Select your ALB and target group
4. Check `HealthyHostCount` and `UnHealthyHostCount`
5. Create widget

**Widget 3: CPU Utilization**
1. Add widget → Line graph
2. **EC2** → **"By Auto Scaling Group"**
3. Select your ASG
4. Check `CPUUtilization`
5. Create widget

**Widget 4: Response Time**
1. Add widget → Line graph
2. **ApplicationELB** → **"Per AppELB, per TG Metrics"**
3. Check `TargetResponseTime`
4. Create widget

Save dashboard!

---

## Creating CloudWatch Alarms

### Alarm 1: High CPU Utilization

1. Go to **CloudWatch** → **Alarms** → **"Create alarm"**
2. Click **"Select metric"**
3. **EC2** → **"By Auto Scaling Group"**
4. Select `CPUUtilization` for your ASG
5. **Period**: 5 minutes
6. **Condition**: Greater than `80`
7. **Datapoints to alarm**: 2 out of 2
8. Click **"Next"**
9. **Alarm name**: `ASG-High-CPU`
10. **Description**: "CPU utilization above 80%"
11. Skip notification (or add SNS topic)
12. Click **"Create alarm"**

### Alarm 2: Unhealthy Targets

1. Create alarm
2. **ApplicationELB** → **"Per AppELB, per TG Metrics"**
3. Select `UnHealthyHostCount`
4. **Condition**: Greater than `0`
5. **Period**: 1 minute
6. **Alarm name**: `ALB-Unhealthy-Targets`
7. Create alarm

### Alarm 3: All Targets Unhealthy

1. Create alarm
2. Select `HealthyHostCount`
3. **Condition**: Less than `1`
4. **Period**: 1 minute
5. **Alarm name**: `ALB-No-Healthy-Targets`
6. Create alarm

### Alarm 4: High Error Rate

1. Create alarm
2. Select `HTTPCode_Target_5XX_Count`
3. **Condition**: Greater than `10`
4. **Period**: 5 minutes
5. **Alarm name**: `ALB-High-5XX-Errors`
6. Create alarm

---

## Viewing Logs

### ALB Access Logs (Optional - Costs Extra)

Enable ALB access logs:
1. Go to **Load Balancers** → Select your ALB
2. **Actions** → **Edit attributes**
3. Under **"Monitoring"**:
   - ✅ Enable access logs
   - S3 location: Create or select bucket
4. Save changes

Logs show:
- Request time
- Client IP
- Request path
- Response code
- Response time

### CloudWatch Logs for EC2 (Optional)

Install CloudWatch Agent on instances:
```bash
# User data addition
sudo yum install -y amazon-cloudwatch-agent

# Configure agent
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config \
  -m ec2 \
  -s
```

---

## Monitoring Best Practices

### ✅ Do's
1. **Set Up Alarms**
   - High CPU
   - Unhealthy targets
   - Error rates

2. **Monitor Regularly**
   - Check dashboard daily
   - Review weekly trends
   - Investigate anomalies

3. **Track Costs**
   - Monitor instance hours
   - Track data transfer
   - Review CloudWatch costs

4. **Keep Historical Data**
   - Enable detailed monitoring for critical times
   - Export metrics for long-term analysis

### ❌ Don'ts
1. **Don't Ignore Alarms**
   - Investigate all alarms
   - Fix root causes

2. **Don't Over-Monitor**
   - Basic monitoring is usually sufficient
   - Detailed monitoring costs extra

3. **Don't Forget Cleanup**
   - Delete test instances
   - Remove unused alarms
   - Clean old dashboards

---

## Testing Checklist

### Load Balancing
- [ ] Multiple instances serving requests
- [ ] Different Instance IDs on refresh
- [ ] Traffic distributed evenly

### High Availability
- [ ] Instances in multiple AZs
- [ ] Instance termination triggers replacement
- [ ] Application remains available during replacement

### Auto Scaling
- [ ] Scales out when CPU > 70%
- [ ] Scales in when CPU < 30%
- [ ] Min/max limits respected

### Health Checks
- [ ] Unhealthy instances detected
- [ ] Unhealthy instances removed from rotation
- [ ] Unhealthy instances terminated
- [ ] Replacements launched

### Monitoring
- [ ] CloudWatch dashboard created
- [ ] Key metrics visible
- [ ] Alarms configured
- [ ] Alarm notifications working

---

## Common Issues

**Issue**: Can't access application
- Check ALB state (must be Active)
- Verify security groups
- Check target health

**Issue**: Targets unhealthy
- Check EC2 security group allows ALB
- Verify Apache is running
- Check health check path

**Issue**: Scaling not working
- Verify scaling policies exist
- Check CloudWatch alarms
- Wait full evaluation period

**Issue**: High costs
- Check instance count
- Review scaling policies
- Consider smaller instance types
- Delete unused resources

---

**✅ Complete!** Your AWS Auto Scaling infrastructure with Application Load Balancer is fully deployed, tested, and monitored!
