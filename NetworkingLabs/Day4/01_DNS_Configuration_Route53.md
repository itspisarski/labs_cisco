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

1. Go to **Route 53 → Hosted Zones → Create Hosted Zone**  
2. Configuration:
   - **Domain name:** `yourdomain.com` *(use a domain you own, or a test name like `example.local`)*  
   - **Type:** Public Hosted Zone  
   - **VPC association:** none for public zones  

✅ You'll see four **NS** records and one **SOA** record automatically created.

---

## ⚙️ Step 2 – Add DNS Records

1. **A Record (Web Server)**  
   - Name: `www`  
   - Type: A  
   - Value: Web Server's public IP or ALB DNS name  
   - TTL: 300 seconds

2. **CNAME Record (Alias)**  
   - Name: `app`  
   - Type: CNAME  
   - Value: `www.yourdomain.com`

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

- From any EC2 in the same VPC:

```
nslookup app.yourdomain.com
dig db.internal
```

- From your local machine (if using a real domain):

```
ping www.yourdomain.com
```

✅ Expected:
- Public records resolve globally  
- Private records resolve only within the VPC

---

## 🔍 Verification

| Test | Expected Result |
|------|------------------|
| nslookup www.yourdomain.com | Returns Web Server IP or ALB name |
| dig db.internal | Resolves only inside VPC |
| ping www.yourdomain.com | Works if A record points to public IP |

---

## 💡 Reflection

- How does Route 53 differ from traditional on-prem DNS servers?  
- Why separate public and private zones?  
- What is the advantage of using alias records for AWS services?

---

**End of Lab 08 – Route 53 DNS**