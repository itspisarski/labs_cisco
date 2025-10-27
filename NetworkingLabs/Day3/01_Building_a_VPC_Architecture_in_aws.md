# Lab 01 – Building a VPC Architecture in AWS


**Objective:**  
Translate a traditional two-switch LAN into a cloud-based Virtual Private Cloud (VPC) on AWS.  
You will design public and private subnets, configure routing, attach an Internet Gateway, and deploy EC2 instances to simulate a web and database tier.

---

## 🧩 Topology

### Traditional LAN
[PC1] — [Switch1] — [Router] — [Switch2] — [PC2]


### AWS Equivalent
```
             ┌──────────────────────────┐
             │         VPC              │
             │  (10.0.0.0/16)          │
             │ ┌──────────────┐ ┌──────────────┐
Internet ⇄ IGW ⇄│ Public Subnet │ │ Private Subnet│
│ (10.0.1.0/24) │ │ (10.0.2.0/24) │
│ Web Server │ │ DB Server │
└──────────────────────────┘
```

---

## ⚙️ Step 1 – Create a VPC

1. Go to **VPC Dashboard** in the AWS Management Console.  
2. Click **Create VPC** → Select **VPC only**.  
3. Configure:
   - **Name tag:** `TrainingVPC`
   - **IPv4 CIDR block:** `10.0.0.0/16`
   - Leave IPv6 unchecked for now.
4. Click **Create VPC**.  

✅ **Result:** Your logical private network is created. This is equivalent to your main LAN.

---

## ⚙️ Step 2 – Create Subnets (Public and Private)

You will create **two subnets**, each in a **different Availability Zone (AZ)** for redundancy.

| Subnet Name | CIDR Block | AZ Example | Type |
|--------------|-------------|-------------|------|
| PublicSubnetA | 10.0.1.0/24 | us-east-1a | Public |
| PrivateSubnetB | 10.0.2.0/24 | us-east-1b | Private |

### Steps
1. In the **VPC Dashboard**, go to **Subnets → Create Subnet**.  
2. Choose `TrainingVPC`.  
3. Add **PublicSubnetA** in `us-east-1a` with `10.0.1.0/24`.  
4. Add **PrivateSubnetB** in `us-east-1b` with `10.0.2.0/24`.  
5. Save both subnets.

✅ **Result:** Your VPC now has two isolated network zones.

---

## ⚙️ Step 3 – Create and Attach an Internet Gateway (IGW)

1. Go to **Internet Gateways** → **Create internet gateway**.  
   - **Name:** `TrainingIGW`
2. After creation, select it → click **Actions → Attach to VPC → TrainingVPC**.  

✅ **Result:** The VPC now has an external gateway to reach the internet (like your router’s uplink).

---

## ⚙️ Step 4 – Configure Route Tables

You will use two route tables: one for **public** and one for **private** subnets.

### Create Public Route Table
1. Go to **Route Tables → Create Route Table**.  
   - **Name:** `PublicRT`
   - **VPC:** `TrainingVPC`
2. Open the new route table → **Routes → Edit routes → Add route**.  
   - **Destination:** `0.0.0.0/0`  
   - **Target:** `TrainingIGW`
3. Save changes.  
4. Under **Subnet associations**, select **PublicSubnetA**.

### Create Private Route Table
1. Create another route table named `PrivateRT`.  
2. No IGW route for now (local only).  
3. Associate it with **PrivateSubnetB**.

✅ **Result:**  
- Public subnet can access the internet.  
- Private subnet remains isolated.

---

## ⚙️ Step 5 – Launch EC2 Instances

| Instance | Subnet | AMI | Type | Key Pair | Security Group |
|-----------|---------|-----|-------|------------|----------------|
| WebServer | PublicSubnetA | Amazon Linux 2 | t2.micro | Existing or new | Allow SSH (22), HTTP (80) |
| DBServer | PrivateSubnetB | Amazon Linux 2 | t2.micro | Existing or new | Allow SSH (22) from WebServer only |

### Launch Steps
1. Go to **EC2 → Launch Instance**.  
2. Configure as per the table above.  
3. Assign each instance to the correct subnet.  
4. For the WebServer:
   - Enable **Auto-assign Public IP**.
5. For the DBServer:
   - Disable **Auto-assign Public IP**.
6. Review and launch both.

✅ **Result:**  
- WebServer will be publicly accessible.  
- DBServer will be isolated within the private subnet.

---

## ⚙️ Step 6 – Test and Verify Connectivity

1. **Connect to WebServer** using its **Public IP** via SSH:  
```
ssh -i yourkey.pem ec2-user@<WebServer_Public_IP>
```
2. From the WebServer, check internal connectivity:
```
ping 10.0.2.10
```

✅ Expected Result:  
Ping to the DBServer **succeeds** (private connectivity within the VPC).

3. Try `curl google.com` from the **DBServer** (if no NAT configured) → should **fail**.  
This confirms that the private subnet has **no internet access**.

---

## 🔍 Verification Checklist

| Check | Expected Result |
|--------|------------------|
| VPC CIDR | 10.0.0.0/16 |
| IGW attached | Yes |
| Public Subnet route | 0.0.0.0/0 → IGW |
| Private Subnet route | Local only |
| WebServer public IP | Reachable |
| DBServer | Accessible only from WebServer |

---

## 💡 Reflection Questions

- What AWS components replace switches and routers from your Day 2 labs?  
- Why is it important to separate public and private subnets?  
- How would you enable outbound internet access for private instances (hint: NAT Gateway)?  
- How could you automate this setup using CloudFormation or Terraform?

---

## 🧠 Summary

| Traditional Network | AWS Equivalent |
|----------------------|----------------|
| Switch / VLAN | Subnet |
| Router | Route Table |
| Firewall | Security Group / NACL |
| Internet Access | Internet Gateway |
| Physical Hosts | EC2 Instances |

---

**End of Lab 05 – Building a VPC Architecture in AWS**