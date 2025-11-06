# Lab 1: AWS Flash Storage Fundamentals
**Duration:** 60 minutes  
**Level:** Beginner (Cloud Fundamentals)  

## 🎯 Learning Objectives
- Understand how AWS implements flash storage concepts.  
- Explore EBS SSDs and NVMe interfaces.  
- Measure IOPS, throughput, and latency using fio.  

## ⚙️ Steps
1. Create a `t3.large` EC2 instance with Amazon Linux 2.  
2. Attach a 20 GB gp3 EBS volume.  
3. Connect via SSH and install tools:  
   ```bash
   sudo yum install -y fio nvme-cli
   ```
4. Check and mount storage:  
   ```bash
   lsblk
   sudo mkfs -t ext4 /dev/nvme1n1
   sudo mkdir /data
   sudo mount /dev/nvme1n1 /data
   ```
5. Run benchmark:  
   ```bash
   sudo fio --name=baseline --filename=/data/testfile --rw=randrw --bs=4k --size=1G --iodepth=16 --numjobs=4 --runtime=60 --time_based --group_reporting
   ```

## 📊 Discussion
- Analyze IOPS, throughput, and latency results.  
- Relate findings to SSD controller and flash architecture concepts.
