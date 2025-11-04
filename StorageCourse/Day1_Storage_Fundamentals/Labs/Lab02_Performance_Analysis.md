# Lab 02 - Block vs File vs Object Storage Performance Analysis

**Objective:** Compare performance characteristics and use cases of different storage types in AWS.  
**Duration:** 60 minutes  
**Prerequisites:** Completed Lab 01, AWS CLI configured

---

## 🧩 Scenario
Your development team needs to choose appropriate storage types for different application components. You need to benchmark and analyze the performance characteristics of each storage type.

---

## ⚙️ Lab 2.1: EBS Volume Type Performance Comparison

### Step 1: Create Different EBS Volume Types
1. **Create multiple EBS volumes:**
   ```bash
   # General Purpose SSD (gp3)
   aws ec2 create-volume --size 20 --volume-type gp3 --availability-zone us-east-1a --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=lab-gp3},{Key=Type,Value=gp3}]'
   
   # Provisioned IOPS SSD (io2)
   aws ec2 create-volume --size 20 --volume-type io2 --iops 1000 --availability-zone us-east-1a --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=lab-io2},{Key=Type,Value=io2}]'
   
   # Throughput Optimized HDD (st1)
   aws ec2 create-volume --size 500 --volume-type st1 --availability-zone us-east-1a --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=lab-st1},{Key=Type,Value=st1}]'
   ```

2. **Attach volumes to EC2 instance:**
   ```bash
   # Get volume IDs
   aws ec2 describe-volumes --filters "Name=tag:Name,Values=lab-*" --query 'Volumes[*].[VolumeId,Tags[?Key==`Name`].Value|[0],VolumeType]' --output table
   
   # Attach each volume (replace with actual volume IDs)
   aws ec2 attach-volume --volume-id vol-xxxxxxxx --instance-id i-xxxxxxxx --device /dev/sdf
   aws ec2 attach-volume --volume-id vol-yyyyyyyy --instance-id i-xxxxxxxx --device /dev/sdg
   aws ec2 attach-volume --volume-id vol-zzzzzzzz --instance-id i-xxxxxxxx --device /dev/sdh
   ```

### Step 2: Format and Mount Volumes
```bash
# SSH to EC2 instance
ssh ec2-user@<instance-ip> -i your-key.pem

# Install fio for performance testing
sudo yum install -y fio

# Create file systems
sudo mkfs.ext4 /dev/xvdf  # gp3
sudo mkfs.ext4 /dev/xvdg  # io2
sudo mkfs.ext4 /dev/xvdh  # st1

# Create mount points
sudo mkdir -p /mnt/{gp3,io2,st1}

# Mount volumes
sudo mount /dev/xvdf /mnt/gp3
sudo mount /dev/xvdg /mnt/io2
sudo mount /dev/xvdh /mnt/st1
```

### Step 3: Performance Benchmarking
```bash
# Test 1: Random 4K Read/Write (IOPS focused)
echo "Testing GP3 Random 4K..."
sudo fio --name=gp3-random --filename=/mnt/gp3/testfile --size=2G --ioengine=libaio --bs=4k --rw=randrw --direct=1 --runtime=60 --group_reporting

echo "Testing IO2 Random 4K..."
sudo fio --name=io2-random --filename=/mnt/io2/testfile --size=2G --ioengine=libaio --bs=4k --rw=randrw --direct=1 --runtime=60 --group_reporting

echo "Testing ST1 Random 4K..."
sudo fio --name=st1-random --filename=/mnt/st1/testfile --size=2G --ioengine=libaio --bs=4k --rw=randrw --direct=1 --runtime=60 --group_reporting

# Test 2: Sequential 1M Read/Write (Throughput focused)
echo "Testing GP3 Sequential 1M..."
sudo fio --name=gp3-seq --filename=/mnt/gp3/testfile-seq --size=5G --ioengine=libaio --bs=1M --rw=rw --direct=1 --runtime=60 --group_reporting

echo "Testing IO2 Sequential 1M..."
sudo fio --name=io2-seq --filename=/mnt/io2/testfile-seq --size=5G --ioengine=libaio --bs=1M --rw=rw --direct=1 --runtime=60 --group_reporting

echo "Testing ST1 Sequential 1M..."
sudo fio --name=st1-seq --filename=/mnt/st1/testfile-seq --size=5G --ioengine=libaio --bs=1M --rw=rw --direct=1 --runtime=60 --group_reporting
```

---

## ⚙️ Lab 2.2: EFS Performance Analysis

### Step 1: EFS Performance Testing
```bash
# Mount EFS if not already mounted
sudo mkdir -p /mnt/efs
sudo mount -t efs fs-xxxxxxxx:/ /mnt/efs

# Test EFS performance
echo "Testing EFS General Purpose..."
sudo fio --name=efs-test --filename=/mnt/efs/testfile --size=1G --ioengine=libaio --bs=4k --rw=randrw --direct=1 --runtime=60 --group_reporting

# Test with different block sizes
echo "Testing EFS with 64K blocks..."
sudo fio --name=efs-64k --filename=/mnt/efs/testfile-64k --size=1G --ioengine=libaio --bs=64k --rw=rw --direct=1 --runtime=60 --group_reporting
```

### Step 2: Multi-Client EFS Testing
```bash
# On first instance
echo "Starting multi-client test from Instance 1..."
sudo fio --name=efs-client1 --filename=/mnt/efs/client1-testfile --size=500M --ioengine=libaio --bs=4k --rw=write --direct=1 --runtime=120 --group_reporting &

# Launch second instance and run simultaneously
# ssh to second instance and run:
# sudo fio --name=efs-client2 --filename=/mnt/efs/client2-testfile --size=500M --ioengine=libaio --bs=4k --rw=write --direct=1 --runtime=120 --group_reporting
```

---

## ⚙️ Lab 2.3: S3 Performance Analysis

### Step 1: S3 Transfer Performance
```bash
# Create test files of different sizes
dd if=/dev/urandom of=small-file.dat bs=1M count=10   # 10MB
dd if=/dev/urandom of=large-file.dat bs=1M count=100  # 100MB

# Single-threaded upload test
echo "Testing S3 single-threaded upload..."
time aws s3 cp small-file.dat s3://your-bucket-name/performance-test/
time aws s3 cp large-file.dat s3://your-bucket-name/performance-test/

# Multi-threaded upload test
echo "Testing S3 multi-threaded upload..."
time aws s3 cp large-file.dat s3://your-bucket-name/performance-test/multipart-test --storage-class STANDARD

# Download performance
echo "Testing S3 download..."
time aws s3 cp s3://your-bucket-name/performance-test/large-file.dat downloaded-large-file.dat
```

### Step 2: S3 Multipart Upload
```bash
# Configure AWS CLI for multipart uploads
aws configure set default.s3.multipart_threshold 64MB
aws configure set default.s3.multipart_chunksize 16MB

# Create larger test file
dd if=/dev/urandom of=very-large-file.dat bs=1M count=500  # 500MB

# Upload with multipart
time aws s3 cp very-large-file.dat s3://your-bucket-name/performance-test/multipart/
```

---

## ⚙️ Lab 2.4: Performance Monitoring with CloudWatch

### Step 1: Enable Detailed Monitoring
```bash
# Enable detailed monitoring for EBS volumes
aws ec2 modify-volume-attribute --volume-id vol-xxxxxxxx --dry-run
aws ec2 describe-volumes --volume-ids vol-xxxxxxxx --query 'Volumes[0].State'
```

### Step 2: View CloudWatch Metrics
1. **Go to CloudWatch console:**
   - **Metrics → EC2 → EBS Volume Metrics**
   - Select your volumes and observe:
     - `VolumeReadOps` / `VolumeWriteOps`
     - `VolumeThroughputPercentage`
     - `VolumeQueueLength`

2. **View EFS metrics:**
   - **Metrics → EFS → File System Metrics**
   - Observe:
     - `TotalIOBytes`
     - `DataReadIOBytes` / `DataWriteIOBytes`

---

## 📊 Lab 2.5: Results Analysis and Documentation

### Performance Results Table
Create a table documenting your results:

| Storage Type | 4K Random IOPS | 1M Sequential (MB/s) | Latency | Cost/GB/Month | Best Use Case |
|--------------|----------------|---------------------|---------|---------------|---------------|
| GP3 | _____ | _____ | _____ | $0.08 | General purpose |
| IO2 | _____ | _____ | _____ | $0.125 + IOPS | High IOPS databases |
| ST1 | _____ | _____ | _____ | $0.045 | Big data, log processing |
| EFS | _____ | _____ | _____ | $0.30 | Shared file access |
| S3 | N/A (object) | _____ | _____ | $0.023 | Web apps, backup |

### Analysis Questions
1. **Which storage type performed best for database-like workloads (random 4K)?**
2. **Which storage type is most cost-effective for archival data?**
3. **When would you choose EFS over EBS?**
4. **How does S3 performance compare to block storage?**

---

## 🔍 Verification Checklist

- [ ] Tested all EBS volume types (gp3, io2, st1)
- [ ] Measured EFS performance with single and multiple clients
- [ ] Analyzed S3 upload/download performance
- [ ] Documented performance characteristics in results table
- [ ] Viewed CloudWatch metrics for storage monitoring
- [ ] Identified appropriate use cases for each storage type

---

## 💡 Key Learning Outcomes

### Performance Patterns:
- **IO2:** Highest IOPS, consistent performance, premium cost
- **GP3:** Balanced performance and cost, configurable IOPS/throughput
- **ST1:** High sequential throughput, low cost, poor random performance
- **EFS:** NFS performance, scales with aggregate usage
- **S3:** Optimized for large objects, not suitable for random access

### Decision Framework:
- **High IOPS requirement:** IO2 or GP3 with provisioned IOPS
- **High throughput requirement:** ST1 or GP3 with provisioned throughput
- **Shared access requirement:** EFS
- **Cost optimization:** GP3 for block, S3 for object
- **Durability requirement:** S3 (99.999999999%) > EBS (99.999%)

---

**Next Lab:** AWS Storage Optimization and Cost Management