# Lab 01 – Configuring DNS with Amazon Route 53

**Objective:**  
Understand how to manage DNS in AWS by creating hosted zones and records in Route 53.  
You'll configure a custom domain to point to a web server and test resolution using the AWS DNS infrastructure.

---

## 🧩 Topology

```
Browser ⇄ Route 53 ⇄ ALB ⇄ Web Servers in VPC
```

---

## ⚙️ Step 1 – Create a Hosted Zone

### Option A: If You Own a Domain
1. Go to **Route 53 → Hosted Zones → Create Hosted Zone**  
2. Configuration:
   - **Domain name:** `yourdomain.com` *(your actual domain)*  
   - **Type:** Public Hosted Zone  
   - **VPC association:** none for public zones  

### Option B: If You Don't Own a Domain (Recommended for Lab)
1. Go to **Route 53 → Hosted Zones → Create Hosted Zone**  
2. Configuration:
   - **Domain name:** `training-lab-[YOUR-INITIALS].example` *(e.g., `training-lab-jd.example`)*  
   - **Type:** Public Hosted Zone  
   - **VPC association:** none for public zones

> 💡 **Note:** Using `.example` domains won't work for real internet resolution, but you can still practice Route 53 configuration and test within AWS services like ALB.

### Option C: Focus on Private Hosted Zone Only
Skip the public zone and go directly to **Step 3** to work with private DNS resolution within your VPC.

✅ You'll see four **NS** records and one **SOA** record automatically created.

---

## ⚙️ Step 2 – Add DNS Records

### For Your Chosen Domain (Replace with your domain from Step 1)

1. **A Record (Web Server)**  
   - **Name:** `www`  
   - **Type:** A  
   - **Value:** Web Server's public IP from Day 3 labs (e.g., `3.85.123.45`)
   - **TTL:** 300 seconds

2. **CNAME Record (Alias)**  
   - **Name:** `app`  
   - **Type:** CNAME  
   - **Value:** `www.training-lab-[YOUR-INITIALS].example` *(or your actual domain)*

### Examples Based on Step 1 Choice:
- **If using example domain:** `www.training-lab-jd.example` → Your WebServer IP
- **If using real domain:** `www.yourdomain.com` → Your WebServer IP
- **If skipping public:** Go to Step 3 for private DNS only

---

## ⚙️ Step 3 – Optional: Create Private Hosted Zone

1. Go to **Route 53 → Create Hosted Zone** again.  
2. Select **Private Hosted Zone**.  
3. Choose your **TrainingVPC** from previous labs.  
4. Add internal records:
   - Name: `db.internal`
   - Type: A  
   - Value: DBServer private IP

✅ This enables name resolution only **within the VPC**.

---

## ⚙️ Step 4 – Test DNS Resolution

### Option A: Testing with Real Domain
From your local machine:
```
nslookup www.yourdomain.com
ping www.yourdomain.com
```

### Option B: Testing with Example Domain (Lab Environment)
**Note:** `.example` domains won't resolve from your local machine, but you can test within AWS.

**From EC2 instances in your VPC:**
```
# Test using AWS DNS resolver
nslookup www.training-lab-[YOUR-INITIALS].example

# Test direct IP connectivity 
curl http://[WebServer-IP]
```

### Option C: Testing Private DNS (Works for Everyone)
**From any EC2 in the same VPC:**
```
# Test private DNS resolution
nslookup db.internal
dig db.internal

# Test connectivity to private resources
ping db.internal
```

### Alternative Testing Methods (No Domain Required)

1. **Test ALB DNS Name Directly:**
   ```
   # Use the ALB DNS name from Day 4 Lab 2
   curl http://your-alb-dns-name.elb.amazonaws.com
   ```

2. **Test Route 53 Alias Records:**
   - Create an **Alias record** pointing to your ALB
   - This works without owning a domain

✅ **Expected Results:**
- **Real domain:** Records resolve globally  
- **Example domain:** Records visible in Route 53 console, testable within AWS
- **Private records:** Resolve only within the VPC

---

## 🔍 Verification

### For Real Domain Users
| Test | Expected Result |
|------|------------------|
| `nslookup www.yourdomain.com` | Returns Web Server IP |
| `ping www.yourdomain.com` | Successful pings to WebServer |
| `dig db.internal` (from EC2) | Resolves only inside VPC |

### For Example Domain Users (Lab Only)
| Test | Location | Expected Result |
|------|----------|----------------|
| Route 53 Console | AWS Console | Records visible and configured correctly |
| `nslookup www.training-lab-xx.example` | From EC2 in VPC | May resolve to WebServer IP |
| `curl http://[WebServer-IP]` | From anywhere | Direct access to web server |
| `dig db.internal` | From EC2 in VPC | Resolves to private IP |

### For Private DNS Only Users
| Test | Location | Expected Result |
|------|----------|----------------|
| `nslookup db.internal` | From EC2 in VPC | Resolves to DBServer private IP |
| `ping db.internal` | From WebServer | Successful connection |
| Direct IP access | Any EC2 in VPC | Works as fallback |

### Universal Verification (Works for Everyone)
```bash
# From WebServer EC2 instance
curl http://[ALB-DNS-NAME]    # Test load balancer access
ping 10.0.2.10               # Test private connectivity  
nslookup db.internal         # Test private DNS
```

---

## 💡 Reflection

- How does Route 53 differ from traditional on-prem DNS servers?  
- Why separate public and private zones?  
- What is the advantage of using alias records for AWS services?

---

**End of Lab 08 – Route 53 DNS**