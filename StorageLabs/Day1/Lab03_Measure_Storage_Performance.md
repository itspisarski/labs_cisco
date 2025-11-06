
# ⚙️ Lab 3 – Measure Storage Performance (EBS Benchmarking)

**Related Topics:** Storage Metrics, RAID, Caching & Tiering  

---

### 🎯 Objective
Use the `fio` tool to test and analyze **storage performance** (IOPS, latency, and throughput) on AWS EBS volumes.

---

### 🕓 Estimated Time
45 minutes

---

### 🧰 Resources Required
- Same EC2 instance and EBS volume from previous labs

---

### 🪜 Steps

#### 1. Connect to Your Instance
```bash
ssh -i <your-key.pem> ec2-user@<public-ip>
```

#### 2. Install fio
```bash
sudo yum install -y fio
```

#### 3. Run Sequential Read/Write Test
```bash
cd /mnt/data
sudo fio --name=seqtest --rw=readwrite --bs=1M --size=500M --numjobs=1 --runtime=30 --group_reporting
```

#### 4. Run Random Read/Write Test
```bash
sudo fio --name=randtest --rw=randrw --rwmixread=70 --bs=4k --size=500M --numjobs=4 --runtime=30 --group_reporting
```

#### 5. Observe and Compare Results
Focus on:
- **IOPS:** operations per second  
- **BW:** bandwidth (MB/s)  
- **lat:** latency (μs or ms)

---

### 🧠 Discussion Questions
- Which test shows higher IOPS?  
- How does block size affect throughput?  
- How do these metrics relate to storage performance concepts?

---

### ✅ Expected Result
Students will visualize the impact of I/O pattern, block size, and concurrency on EBS performance.

---

### 🧹 Cleanup
```bash
aws ec2 terminate-instances --instance-ids <instance-id>
```

---
