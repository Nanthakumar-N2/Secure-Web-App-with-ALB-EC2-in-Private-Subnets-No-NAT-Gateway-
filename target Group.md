# 04 - Target Group Setup Guide

## Overview

A Target Group is a logical grouping of targets (EC2 instances) that receive traffic from a load balancer. It performs health checks to ensure only healthy instances receive traffic.

## What We'll Create

- **Target Group** with:
  - HTTP protocol on port 80
  - Health checks every 30 seconds
  - Automatic registration via Auto Scaling Group

---

## Step 1: Navigate to Target Groups

### 1.1 Open EC2 Dashboard

1. Go to **AWS Management Console**
2. Navigate to **EC2 Dashboard**
3. In the left sidebar, scroll to **"Load Balancing"** section
4. Click **"Target Groups"**

### 1.2 Create Target Group

1. Click **"Create target group"** button (orange button, top right)

---

## Step 2: Choose Target Type

### 2.1 Select Target Type

You'll see three options:

| Type | Description | Use Case |
|------|-------------|----------|
| **Instances** | Register EC2 instances as targets | ✅ Use this for our setup |
| **IP addresses** | Register by IP address | Containers, on-premises servers |
| **Lambda function** | Invoke Lambda function | Serverless applications |

1. Select **"Instances"** (should be selected by default)
2. Click **"Next"**

---

## Step 3: Target Group Configuration

### 3.1 Basic Configuration

| Field | Value | Description |
|-------|-------|-------------|
| **Target group name** | `web-servers-tg` | Name for your target group |
| **Protocol** | HTTP | Communication protocol |
| **Port** | 80 | Port number |
| **IP address type** | IPv4 | IP version |
| **VPC** | Select `my-asg-vpc` | Your VPC from dropdown |

### 3.2 Protocol Version

| Field | Value |
|-------|-------|
| **Protocol version** | HTTP1 |

**Protocol Options**:
- **HTTP1**: Standard HTTP (recommended for our demo)
- **HTTP2**: Newer protocol, better performance
- **gRPC**: For gRPC applications

**For this tutorial, select HTTP1.**

---

## Step 4: Health Check Settings

Health checks determine if targets are healthy and can receive traffic.

### 4.1 Health Check Protocol

| Field | Value | Description |
|-------|-------|-------------|
| **Health check protocol** | HTTP | Protocol for health checks |
| **Health check path** | `/` | URL path to check |

**Why `/`?**
- Our homepage is at root path
- Simple and reliable
- If page loads, server is healthy

### 4.2 Advanced Health Check Settings

Click **"Advanced health check settings"** to expand.

| Field | Value | Description |
|-------|-------|-------------|
| **Port** | Traffic port | Use same port as traffic (80) |
| **Healthy threshold** | `2` | Consecutive successful checks to mark healthy |
| **Unhealthy threshold** | `2` | Consecutive failed checks to mark unhealthy |
| **Timeout** | `5` seconds | Time to wait for response |
| **Interval** | `30` seconds | Time between health checks |
| **Success codes** | `200` | HTTP status codes that indicate success |

### 4.3 Health Check Settings Explained

**Healthy Threshold (2)**:
- Target must pass 2 consecutive health checks
- Takes 60 seconds (2 × 30) to become healthy
- Prevents false positives

**Unhealthy Threshold (2)**:
- Target must fail 2 consecutive health checks
- Takes 60 seconds to be marked unhealthy
- Allows for temporary issues

**Timeout (5 seconds)**:
- If response takes longer than 5 seconds, check fails
- Balances between patience and responsiveness

**Interval (30 seconds)**:
- Check every 30 seconds
- Shorter intervals = faster detection, more requests
- 30 seconds is good balance

**Success Codes (200)**:
- HTTP 200 = OK, successful response
- Can add other codes: `200,301,302`

### 4.4 Health Check Timeline

```
Time: 0s    30s   60s   90s   120s
      │     │     │     │     │
Check │ ✓   │ ✓   │ X   │ X   │
      │     │     │     │     │
Status│Healthy │   │Unhealthy│
```

---

## Step 5: Tags

### 5.1 Add Tags (Optional)

Tags help organize and track resources.

Click **"Add tag"**:

| Key | Value |
|-----|-------|
| Name | web-servers-tg |
| Project | AutoScaling-Demo |
| Environment | Development |

---

## Step 6: Register Targets

### 6.1 Skip Registration

**Important**: Do NOT register any targets now.

The **Auto Scaling Group** will automatically register instances.

1. On the **"Register targets"** page
2. **Skip** this step - don't select any instances
3. Click **"Next"** button

**Why skip?**
- ASG automatically manages target registration
- Manual registration would conflict with ASG
- ASG adds/removes targets dynamically

---

## Step 7: Review and Create

### 7.1 Review Configuration

Review all settings on the summary page:

| Section | Value |
|---------|-------|
| **Target group name** | web-servers-tg |
| **Protocol** | HTTP:80 |
| **VPC** | my-asg-vpc |
| **Health check path** | / |
| **Healthy threshold** | 2 |
| **Unhealthy threshold** | 2 |
| **Timeout** | 5 seconds |
| **Interval** | 30 seconds |

### 7.2 Create Target Group

1. Click **"Create target group"** button at the bottom
2. Wait for success message: "Successfully created target group"
3. **Note down the Target Group ARN** (very long identifier)

**✅ Checkpoint**: Target Group created successfully!

---

## Step 8: Verify Target Group

### 8.1 View Target Group Details

1. You should be redirected to the target group details page
2. If not, go to **Target Groups** and select `web-servers-tg`

### 8.2 Check Tabs

**Details Tab**:
- Target group ARN
- Load balancer: (none yet - we'll add this next)
- Protocol: HTTP:80
- VPC: my-asg-vpc

**Targets Tab**:
- Should be empty (0 registered targets)
- Message: "No targets are registered with this target group"
- This is expected! ASG will add them later.

**Health checks Tab**:
- Review health check settings
- Confirm all values are correct

**Monitoring Tab**:
- No data yet (will show metrics after targets are added)

**Tags Tab**:
- Your tags should be visible

---

## Understanding Target Groups

### Target Group Architecture

```
                Application Load Balancer
                          │
                          ↓
                    Target Group
                    (web-servers-tg)
                          │
            ┌─────────────┼─────────────┐
            │             │             │
            ↓             ↓             ↓
        Instance 1    Instance 2    Instance 3
        (Healthy)     (Healthy)     (Unhealthy)
            ✓             ✓             X
    
    Traffic sent only to healthy targets
```

### Health Check States

| State | Description | Traffic |
|-------|-------------|---------|
| **Initial** | New target, being checked | No traffic |
| **Healthy** | Passed health checks | Receives traffic ✓ |
| **Unhealthy** | Failed health checks | No traffic |
| **Draining** | Being deregistered | Existing connections only |
| **Unused** | No load balancer attached | N/A |

### Health Check Process

```
1. Target registered
   ↓
2. Initial health check
   ↓
3. Wait for 2 consecutive successes (60 seconds)
   ↓
4. Target becomes "Healthy"
   ↓
5. Load balancer sends traffic
   ↓
6. Continuous health checks every 30 seconds
   ↓
7. If 2 consecutive failures → "Unhealthy"
```

---

## Advanced Configuration (Optional)

### Stickiness (Session Affinity)

Make users stick to same instance:

1. Select target group
2. Click **"Actions"** → **"Edit attributes"**
3. Under **"Stickiness"**:
   - Enable: ✅ Stickiness
   - Type: Load balancer generated cookie
   - Duration: 1 hour (3600 seconds)
4. Click **"Save changes"**

**When to use**:
- Session-based applications
- Shopping carts
- User preferences stored in memory

**When NOT to use**:
- Stateless applications (recommended)
- Can cause uneven distribution

### Slow Start Mode

Gradually increase traffic to new targets:

1. Edit attributes
2. Under **"Slow start duration"**:
   - Enable slow start
   - Duration: 30-300 seconds
3. Save changes

**Why use slow start**:
- Warm up new instances
- Prevent overwhelming new targets
- Better for applications with caching

### Deregistration Delay

Time to complete in-flight requests:

1. Edit attributes
2. **Deregistration delay**: 300 seconds (default)
3. Save changes

**What it does**:
- Allows existing connections to complete
- New connections go to other targets
- Prevents abrupt disconnections

---

## Target Group Attributes

### Default Attributes

| Attribute | Default | Description |
|-----------|---------|-------------|
| **Deregistration delay** | 300 seconds | Time to drain connections |
| **Slow start** | 0 seconds (disabled) | Gradual traffic ramp-up |
| **Stickiness** | Disabled | Session affinity |
| **Load balancing algorithm** | Round robin | How traffic is distributed |
| **HTTP/2** | Enabled | HTTP/2 support |

---

## Monitoring Target Groups

### Key Metrics

| Metric | Description | Good Value |
|--------|-------------|------------|
| **HealthyHostCount** | Number of healthy targets | ≥ 2 |
| **UnHealthyHostCount** | Number of unhealthy targets | 0 |
| **TargetResponseTime** | Average response time | < 1 second |
| **RequestCount** | Total requests | Varies |
| **HTTPCode_Target_2XX** | Successful responses | > 99% |
| **HTTPCode_Target_4XX** | Client errors | < 1% |
| **HTTPCode_Target_5XX** | Server errors | < 1% |

### CloudWatch Alarms

Set up alarms for:
1. **Unhealthy targets** > 0
2. **All targets unhealthy**
3. **High response time** > 3 seconds
4. **High error rate** > 5%

---

## Troubleshooting Target Groups

### Issue 1: No targets registered

**Symptoms**: Target group shows 0 targets

**Possible Causes**:
1. Auto Scaling Group not created yet
2. ASG not attached to target group
3. Instances failed to launch

**Solution**:
- Wait for ASG creation (next step)
- Check ASG configuration
- Review ASG activity history

### Issue 2: Targets unhealthy

**Symptoms**: All targets show "Unhealthy" status

**Possible Causes**:
1. Security group doesn't allow traffic from ALB
2. Web server not running
3. Health check path incorrect
4. Health check timeout too short

**Solutions**:
```
1. Check EC2 security group:
   - Allows port 80 from ALB security group

2. Check web server:
   - SSH into instance
   - Run: sudo systemctl status httpd
   - Check logs: /var/log/httpd/error_log

3. Verify health check path:
   - Try accessing: http://instance-ip/
   - Should return HTTP 200

4. Check health check settings:
   - Timeout ≤ Interval
   - Path returns quickly
```

### Issue 3: Targets constantly flip between healthy/unhealthy

**Symptoms**: Status changes frequently

**Possible Causes**:
1. Application intermittently fails
2. High load causing timeouts
3. Health check too strict

**Solutions**:
- Increase timeout to 10 seconds
- Increase healthy threshold to 3
- Check application logs
- Add more instances to handle load

### Issue 4: Can't delete target group

**Error**: "Target group is currently in use"

**Cause**: Target group attached to load balancer or ASG

**Solution**:
1. Remove from load balancer listener
2. Detach from Auto Scaling Group
3. Wait a few minutes
4. Try deleting again

---

## Target Group Best Practices

### ✅ Do's

1. **Use Appropriate Health Checks**
   - Check actual application health
   - Use dedicated health endpoint
   - Keep checks simple and fast

2. **Set Reasonable Thresholds**
   - Balance between responsiveness and stability
   - Consider application startup time
   - Account for temporary failures

3. **Monitor Target Health**
   - Set up CloudWatch alarms
   - Review unhealthy targets regularly
   - Investigate failures

4. **Use Descriptive Names**
   - Include purpose in name
   - Add comprehensive tags
   - Document configurations

5. **Test Health Checks**
   - Manually test health check URL
   - Verify response codes
   - Check response time

### ❌ Don'ts

1. **Don't Use Short Intervals**
   - < 10 seconds can overwhelm targets
   - Increases costs
   - Rarely necessary

2. **Don't Ignore Unhealthy Targets**
   - Always investigate
   - Fix root cause
   - Don't just restart instances

3. **Don't Manually Register Targets**
   - Let ASG manage targets
   - Manual registration causes conflicts
   - Defeats purpose of auto scaling

4. **Don't Use Complex Health Checks**
   - Keep checks simple
   - Fast response time
   - Avoid database queries in health checks

---

## Target Group Checklist

Use this checklist to verify your target group setup:

### Basic Configuration
- [ ] Target group name: `web-servers-tg`
- [ ] Type: Instances
- [ ] Protocol: HTTP
- [ ] Port: 80
- [ ] VPC: my-asg-vpc

### Health Checks
- [ ] Protocol: HTTP
- [ ] Path: /
- [ ] Healthy threshold: 2
- [ ] Unhealthy threshold: 2
- [ ] Timeout: 5 seconds
- [ ] Interval: 30 seconds
- [ ] Success codes: 200

### Configuration
- [ ] No targets manually registered
- [ ] Tags added
- [ ] Target group ARN noted

### Verification
- [ ] Target group created successfully
- [ ] Details page accessible
- [ ] Health check settings correct
- [ ] Ready for load balancer attachment

---

## Target Group Information Summary

### Configuration Details

| Setting | Value |
|---------|-------|
| **Name** | web-servers-tg |
| **ARN** | arn:aws:elasticloadbalancing:region:account:targetgroup/... |
| **Type** | instance |
| **Protocol** | HTTP:80 |
| **VPC** | my-asg-vpc (vpc-xxxxx) |
| **Health Check** | HTTP:80 / every 30s |

### Health Check Logic

```
Success = HTTP 200 response in < 5 seconds

Healthy:   ✓ → ✓ (2 consecutive successes)
Unhealthy: X → X (2 consecutive failures)

Time to healthy:   60 seconds (2 × 30s)
Time to unhealthy: 60 seconds (2 × 30s)
```

---

## Next Steps

Target Group is now configured! Proceed to:

➡️ **[05-load-balancer.md](05-load-balancer.md)** - Create Application Load Balancer

---

## Quick Reference

### Target Group Components

```
Target Group (web-servers-tg)
│
├── Protocol: HTTP:80
├── VPC: my-asg-vpc
├── Health Checks
│   ├── Path: /
│   ├── Interval: 30s
│   ├── Timeout: 5s
│   └── Success: 200
├── Registered Targets: (managed by ASG)
└── Attributes
    ├── Deregistration delay: 300s
    ├── Stickiness: Disabled
    └── Slow start: Disabled
```

---

**✅ Target Group Setup Complete!** Your target group is ready to receive traffic from the load balancer.
