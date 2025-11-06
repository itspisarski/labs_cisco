# 🧩 Lab 1 – Explore Block Storage (EBS Volume Management)

**Related Topics:** DAS, iSCSI, Block Storage, SAN  

---

### 🎯 Objective
Learn how **block storage** works by creating and attaching an **EBS volume** to an EC2 instance, formatting it, and testing write operations.

---

### 🕓 Estimated Time
45–60 minutes

---

### 🧰 Resources Required
- AWS account (Free Tier)
- 1 EC2 instance (Amazon Linux 2023, t2.micro)
- 1 EBS volume (5 GB, gp3 type)

---

### 🪜 Steps

#### 1. Launch an EC2 Instance
1. Sign in to the [AWS Management Console](https://aws.amazon.com/console/).
2. Go to **EC2 → Launch Instance**.
3. Configure:
   - **Name:** `storage-lab-instance`
   - **AMI:** Amazon Linux 2023
   - **Instance type:** t2.micro (Free Tier)
   - **Key pair:** Create or choose existing
   - **Security Group:** Allow SSH (port 22)
4. Click **Launch Instance**.

---

#### 2. Create and Attach an EBS Volume
1. In the EC2 menu, go to **Volumes → Create Volume**.
2. Choose:
   - **Size:** 5 GB  
   - **Type:** gp3  
   - **Availability Zone:** same as your instance
3. Click **Create Volume**.
4. After creation → **Actions → Attach Volume** → Select your instance → attach as `/dev/xvdf`.

---

#### 3. Connect to the Instance
```bash
ssh -i <your-key.pem> ec2-user@<public-ip>
```

---

#### 4. Identify and Format the Volume
```bash
lsblk
sudo mkfs -t ext4 /dev/xvdf
```

---

#### 5. Mount the Volume
```bash
sudo mkdir /mnt/data
sudo mount /dev/xvdf /mnt/data
```

---

#### 6. Verify and Test
```bash
df -h
sudo dd if=/dev/zero of=/mnt/data/testfile bs=1M count=100
```

---

#### 7. Make Mount Persistent
```bash
echo "/dev/xvdf /mnt/data ext4 defaults,nofail 0 2" | sudo tee -a /etc/fstab
```

---

#### 8. Cleanup
- Detach and delete the EBS volume.  
- Terminate the EC2 instance.

---

### ✅ Expected Result
- `/mnt/data` is mounted and functional.
- You understand **block storage behavior** and its use in SANs.

---
