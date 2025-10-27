# Lab 03 – Troubleshooting a Broken VPC

**Objective:**  
Learn how to identify and fix common networking issues in AWS VPCs.  
You'll work with a pre-deployed but malfunctioning VPC environment and systematically restore connectivity between instances.

---

## 🧩 Topology

```
       ┌────────────────────────────┐
       │         TrainingVPC        │
       │        (10.0.0.0/16)       │
       │                            │
       │ ┌────────────┐  ┌────────────┐
       │ │ Public Sub │  │ Private Sub │
       │ │10.0.1.0/24 │  │10.0.2.0/24 │
       │ │ WebServer  │  │ DBServer   │
       │ └────────────┘  └────────────┘
       │     │                │
       │   IGW ❌ (Detached)  │
       └────────────────────────────┘
```

> The architecture looks normal, but something is **broken** — you'll find and fix it.

---

## ⚙️ Step 1 – Verify Environment Setup

1. Open the AWS Management Console.  
2. Go to **VPC → Your VPCs**.  
   - Confirm a VPC named `BrokenVPC` exists with CIDR `10.0.0.0/16`.  
3. Check **Subnets**:
   - `PublicSubnet` → 10.0.1.0/24  
   - `PrivateSubnet` → 10.0.2.0/24  
4. Two EC2 instances should exist:
   - **WebServer** in PublicSubnet  
   - **DBServer** in PrivateSubnet

✅ Both instances are running, but **connectivity is broken**.

---

## ⚙️ Step 2 – Test Connectivity Symptoms

1. Copy the **Public IP** of WebServer and attempt to SSH:

```
ssh ec2-user@<WebServer_Public_IP> -i yourkey.pem
```

❌ Connection times out.

2. From the **AWS Console**, open **EC2 → Instance → Networking → Reachability Analyzer**.  
   - Source: `WebServer`
   - Destination: `DBServer`
   - Result: **"Path not found"**

🧩 You'll now fix each possible cause one by one.

---

## ⚙️ Step 3 – Inspect Internet Gateway

1. Go to **VPC → Internet Gateways**.  
2. You should see an **IGW named `BrokenIGW`**, but it may be:
   - Detached, or  
   - Not associated with `BrokenVPC`.

✅ **Fix:**
- If detached → Select IGW → **Actions → Attach to VPC → BrokenVPC**.  
- If missing → Create a new one:
  - Name: `FixedIGW`
  - Attach to: `BrokenVPC`

---

## ⚙️ Step 4 – Check Public Route Table

1. Go to **VPC → Route Tables** → locate `PublicRT`.  
2. Open **Routes** tab and verify:

| Destination | Target |
|--------------|--------|
| 10.0.0.0/16  | local |
| 0.0.0.0/0    | igw-xxxxx |

❌ If there is no `0.0.0.0/0` → IGW route, **add it** manually.  
✅ Then associate the route table with **PublicSubnet**.

---

## ⚙️ Step 5 – Review Security Groups

1. Go to **EC2 → Security Groups** → select the one attached to **WebServer**.  
2. Ensure inbound rules:

| Type | Protocol | Port | Source |
|------|-----------|-------|--------|
| SSH | TCP | 22 | Your public IP |
| HTTP | TCP | 80 | 0.0.0.0/0 |

✅ If missing, add these and save.

---

## ⚙️ Step 6 – Review Network ACLs (NACLs)

1. Go to **VPC → Network ACLs** → locate the one for **PublicSubnet**.  
2. Check **Inbound Rules**:

| Rule # | Type | Protocol | Port Range | Source | Allow/Deny |
|---------|------|-----------|-------------|---------|------------|
| 100 | ALL Traffic | ALL | ALL | 0.0.0.0/0 | ALLOW |

3. Check **Outbound Rules**:

| Rule # | Type | Protocol | Port Range | Destination | Allow/Deny |
|---------|------|-----------|-------------|--------------|------------|
| 100 | ALL Traffic | ALL | ALL | 0.0.0.0/0 | ALLOW |

❌ If you see "DENY" rules or missing entries, correct them.

---

## ⚙️ Step 7 – Inspect Private Subnet Routing

1. Go to **Route Tables** → find the one associated with `PrivateSubnet`.  
2. Verify it only routes:

| Destination | Target |
|--------------|--------|
| 10.0.0.0/16 | local |

✅ This is expected (no internet route).  
You'll later test internal connectivity between WebServer and DBServer.

---

## ⚙️ Step 8 – Test Again

1. Try SSH to WebServer again:

```
ssh ec2-user@<WebServer_Public_IP>
```

✅ Should now connect.

2. From WebServer, ping DBServer:

```
ping 10.0.2.10
```

✅ Should succeed (private connectivity).

3. Test outbound internet from WebServer:

```
curl google.com
```

✅ Should return HTML output.

---

## ⚙️ Step 9 – Optional: Use Reachability Analyzer

1. Go to **VPC → Reachability Analyzer**.  
2. Source: WebServer  
   Destination: DBServer  
3. Run analysis again — this time it should show:

```
Path found: WebServer → Route Table → Subnet → IGW → DBServer
```

---

## 🔍 Verification Checklist

| Component | Issue Found | Action Taken | Status |
|------------|--------------|---------------|--------|
| IGW | Detached | Reattached to VPC | ✅ Fixed |
| Route Table | Missing 0.0.0.0/0 | Added route to IGW | ✅ Fixed |
| Security Group | No SSH rule | Added inbound 22 | ✅ Fixed |
| NACL | Deny rule present | Replaced with ALLOW | ✅ Fixed |

✅ Connectivity fully restored.

---

## 💡 Reflection Questions

- How can you differentiate between Security Group and NACL issues?  
- Why might the IGW be attached but still not allow outbound traffic?  
- How could AWS **VPC Flow Logs** or **Reachability Analyzer** help automate troubleshooting?  
- What are signs of asymmetric routing or overlapping CIDR blocks?

---

## 🧠 Key Takeaways

| Concept | Description |
|----------|--------------|
| Systematic Troubleshooting | Check from outermost (IGW) to innermost (instance) layer |
| Common Root Causes | Detached IGW, missing routes, restrictive SG/NACL |
| AWS Tools | Reachability Analyzer, Flow Logs, VPC Peering checks |
| Best Practice | Document your known-good VPC configuration for reuse |

---
