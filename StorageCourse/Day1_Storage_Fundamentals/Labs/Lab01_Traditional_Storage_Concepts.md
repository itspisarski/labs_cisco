# Lab 01 - Traditional Storage Concepts & AWS Mapping

**Objective:** Understand traditional storage architectures and how they map to AWS services.  
**Duration:** 45 minutes  
**Prerequisites:** AWS account with EC2 and storage service access

---

## 🧩 Scenario
You're a storage architect helping your company understand how traditional on-premises storage concepts translate to AWS cloud services.

---

## ⚙️ Lab 1.1: Direct Attached Storage (DAS) Simulation

### Step 1: Launch EC2 Instance with Local Storage
1. **Launch EC2 instance:**
   - Go to **EC2 → Launch Instance**
   - **Instance type:** `m5d.large` (includes NVMe SSD local storage)
   - **AMI:** Amazon Linux 2
   - **Security Group:** Allow SSH from your IP
   - **Key pair:** Use existing or create new

2. **Examine local storage:**
   ```bash
   # SSH to instance
   ssh ec2-user@<instance-ip> -i your-key.pem
   
   # List block devices
   lsblk
   
   # Check NVMe local storage
   sudo fdisk -l
   nvme list
   ```

3. **Configure local storage:**
   ```bash
   # Create file system on local NVMe
   sudo mkfs.ext4 /dev/nvme1n1
   
   # Create mount point and mount
   sudo mkdir /mnt/local-storage
   sudo mount /dev/nvme1n1 /mnt/local-storage
   
   # Test write performance
   sudo fio --name=test --ioengine=libaio --rw=write --bs=4k --size=1G --filename=/mnt/local-storage/testfile --direct=1
   ```

### Step 2: Document DAS Characteristics
**Record observations:**
- Performance characteristics
- Availability (what happens if instance stops?)
- Sharing capabilities
- Cost implications

---

## ⚙️ Lab 1.2: Network Attached Storage (NAS) with EFS

### Step 1: Create Amazon EFS File System
1. **Create EFS:**
   - Go to **EFS → Create file system**
   - **Name:** `lab-nas-efs`
   - **VPC:** Use default VPC
   - **Performance mode:** General Purpose
   - **Create**

2. **Configure security group:**
   - **EFS → Security Groups → Add rule**
   - **Type:** NFS
   - **Source:** Security group of EC2 instances

### Step 2: Mount EFS on Multiple Instances
1. **Install EFS utilities:**
   ```bash
   # On first instance
   sudo yum install -y amazon-efs-utils
   
   # Create mount point
   sudo mkdir /mnt/efs-storage
   
   # Mount EFS (replace with your EFS ID)
   sudo mount -t efs fs-xxxxxx:/ /mnt/efs-storage
   ```

2. **Launch second EC2 instance and repeat mount process**

3. **Test file sharing:**
   ```bash
   # On first instance
   echo "Hello from Instance 1" | sudo tee /mnt/efs-storage/test-file.txt
   
   # On second instance - verify file appears
   cat /mnt/efs-storage/test-file.txt
   ```

### Step 3: Performance Testing
```bash
# Test NAS performance
sudo fio --name=efs-test --ioengine=libaio --rw=write --bs=64k --size=1G --filename=/mnt/efs-storage/testfile --direct=1

# Compare to local storage performance
```

---

## ⚙️ Lab 1.3: Storage Area Network (SAN) with EBS

### Step 1: Create and Attach EBS Volumes
1. **Create EBS volume:**
   - Go to **EC2 → Volumes → Create Volume**
   - **Size:** 20 GB
   - **Type:** gp3
   - **Availability Zone:** Same as EC2 instance

2. **Attach volume to instance:**
   - Select volume → **Actions → Attach Volume**
   - **Instance:** Select your EC2 instance
   - **Device:** `/dev/sdf`

### Step 2: Configure Block Storage
```bash
# List attached volumes
lsblk

# Create partition and file system
sudo fdisk /dev/xvdf
# Create partition (follow prompts: n, p, 1, defaults, w)

sudo mkfs.ext4 /dev/xvdf1

# Mount the volume
sudo mkdir /mnt/ebs-storage
sudo mount /dev/xvdf1 /mnt/ebs-storage

# Test block storage performance
sudo fio --name=ebs-test --ioengine=libaio --rw=randwrite --bs=4k --size=1G --filename=/mnt/ebs-storage/testfile --direct=1
```

### Step 3: Demonstrate SAN Characteristics
```bash
# Create test data
sudo dd if=/dev/zero of=/mnt/ebs-storage/testfile bs=1M count=100

# Detach and reattach to different instance (via console)
# Show that data persists independently of compute
```

---

## ⚙️ Lab 1.4: Object Storage with S3

### Step 1: Create S3 Bucket
1. **Create bucket:**
   - Go to **S3 → Create bucket**
   - **Name:** `lab-object-storage-[your-initials]-[timestamp]`
   - **Region:** us-east-1
   - **Create**

### Step 2: Object Operations
```bash
# Install AWS CLI (if not installed)
sudo yum install -y awscli

# Configure AWS credentials
aws configure

# Upload objects
echo "This is object storage data" > test-object.txt
aws s3 cp test-object.txt s3://your-bucket-name/

# List objects
aws s3 ls s3://your-bucket-name/

# Download object
aws s3 cp s3://your-bucket-name/test-object.txt downloaded-object.txt
```

### Step 3: Demonstrate Object Storage Features
```bash
# Upload with metadata
aws s3 cp test-object.txt s3://your-bucket-name/metadata-test.txt --metadata purpose=demo,created=today

# Enable versioning
aws s3api put-bucket-versioning --bucket your-bucket-name --versioning-configuration Status=Enabled

# Upload new version
echo "Updated content" > test-object.txt
aws s3 cp test-object.txt s3://your-bucket-name/

# List versions
aws s3api list-object-versions --bucket your-bucket-name
```

---

## 🔍 Verification Checklist

| Storage Type | AWS Service | Key Characteristics | Use Case |
|--------------|-------------|-------------------|----------|
| DAS | EC2 Instance Store | High performance, ephemeral | ✅ Temporary processing |
| NAS | Amazon EFS | Shared, POSIX-compliant | ✅ Content sharing |
| SAN | Amazon EBS | Block-level, persistent | ✅ Database storage |
| Object | Amazon S3 | Scalable, metadata-rich | ✅ Web applications |

---

## 💡 Lab Questions

1. **Performance Comparison:** Which storage type provided the best performance for 4K random writes?
2. **Persistence:** What happens to data on each storage type when the EC2 instance is terminated?
3. **Sharing:** Which storage types allow multiple instances to access the same data simultaneously?
4. **Cost Analysis:** Compare the cost implications of each storage type for a 1TB dataset.

---

## 🧠 Key Learning Outcomes

| Traditional Concept | AWS Implementation | Key Benefits |
|-------------------|-------------------|--------------|
| **Direct Attached Storage** | EC2 Instance Store | Highest performance for temporary data |
| **Network Attached Storage** | Amazon EFS | POSIX compliance, multi-mount capability |
| **Storage Area Network** | Amazon EBS | Persistent block storage, snapshots |
| **Object Storage** | Amazon S3 | Unlimited scale, rich metadata, global access |

---

**Next Lab:** AWS Storage Services Deep Dive - EBS Volume Types and Performance Optimization