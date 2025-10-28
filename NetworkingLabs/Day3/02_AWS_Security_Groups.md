# Lab 02 – Understanding and Configuring AWS Security Groups

**Objective:**  
Learn how to configure and test Security Groups (SGs) in AWS to control inbound and outbound traffic at the **instance level**.  
You'll secure a web server and a database server within a VPC and observe how SGs enforce Layer 4 rules.

---

## 🧩 Topology

```
      Internet
         │
    [IGW – TrainingIGW]
         │
   ┌───────────────────────┐
   │  VPC (10.0.0.0/16)   │
   │  ┌────────────┐ ┌────────────┐
   │  │PublicSubnet│ │PrivateSubnet│
   │  │Web Server  │ │DB Server   │
   └──────────────────────────────┘
```

---

## ⚙️ Step 1 – Verify Existing VPC Setup

If you completed **Lab 01**, use the same VPC.  
Confirm:
- PublicSubnetA → 10.0.1.0/24  
- PrivateSubnetB → 10.0.2.0/24  
- WebServer (public) and DBServer (private) are running  

If not, create these resources as per **Lab 01**.

---

## ⚙️ Step 2 – Create Security Groups

### a. WebServerSG
1. Go to **EC2 → Security Groups → Create Security Group**.  
2. Name: `WebServerSG`  
3. VPC: `TrainingVPC`
4. Add inbound rules:
   | Type | Protocol | Port | Source |
   |------|-----------|-------|--------|
   | SSH | TCP | 22 | Your public IP |
   | HTTP | TCP | 80 | 0.0.0.0/0 |

5. Outbound rule: Keep default (Allow All).

### b. DBServerSG
1. Create another SG named `DBServerSG`.  
2. Add inbound rules:
   | Type | Protocol | Port | Source |
   |------|-----------|-------|--------|
   | SSH | TCP | 22 | WebServerSG |
   | MySQL/Aurora | TCP | 3306 | WebServerSG |

3. Outbound rule: Keep default (Allow All).

---

## ⚙️ Step 3 – Assign Security Groups to Instances

| Instance | Security Group |
|-----------|----------------|
| WebServer | WebServerSG |
| DBServer | DBServerSG |

You will need to create 2 instances in EC2.  

To assign:
- In EC2 console, select the instance → **Actions → Security → Change Security Groups** → choose accordingly.



---

## ⚙️ Step 4 – Test Security Group Rules

1. SSH into the WebServer (public IP).  

```
ssh -i yourkey.pem ec2-user@<WebServer_Public_IP>
```

2. From WebServer, try to SSH into DBServer:  

```
ssh ec2-user@10.0.2.10
```

✅ Should **succeed**.

3. From your **local machine**, try:  

```
ssh ec2-user@<DBServer_Private_IP>
```

❌ Should **fail** (not accessible directly).

4. Try opening the WebServer's public IP in a browser:  

```
http://<WebServer_Public_IP>
```

✅ Should load if HTTP (port 80) is open.

---

## 🔍 Verification Checklist

| Test | Expected Result |
|------|------------------|
| SSH to WebServer | ✅ Allowed from your IP |
| HTTP to WebServer | ✅ Allowed from anywhere |
| SSH to DBServer from WebServer | ✅ Allowed |
| SSH to DBServer from Internet | ❌ Blocked |

---


---

## 🧠 Key Takeaways

| Concept | Description |
|----------|--------------|
| Security Group | Virtual firewall at instance level |
| Stateful | Return traffic automatically allowed |
| Default Behavior | Deny all inbound, allow all outbound |
| Best Practice | One SG per function (web, db, bastion, etc.) |

---
