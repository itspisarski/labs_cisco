# Lab 2: Comparing gp3 and io2 Volumes
**Duration:** 60 minutes  
**Level:** Intermediate  

## 🎯 Learning Objectives
- Compare gp3 and io2 performance characteristics.  
- Understand over-provisioning and endurance trade-offs.  

## ⚙️ Steps
1. In the same EC2 instance, attach a 20 GB io2 EBS volume.  
2. Mount it:  
   ```bash
   sudo mkfs -t ext4 /dev/nvme2n1
   sudo mkdir /io2
   sudo mount /dev/nvme2n1 /io2
   ```
3. Benchmark gp3 vs io2:  
   ```bash
   sudo fio --name=gp3test --filename=/data/testfile --rw=randread --bs=4k --iodepth=16 --numjobs=4 --runtime=60 --time_based --group_reporting
   sudo fio --name=io2test --filename=/io2/testfile --rw=randread --bs=4k --iodepth=16 --numjobs=4 --runtime=60 --time_based --group_reporting
   ```
4. Compare latency, throughput, and IOPS results.  

## 📊 Discussion
- gp3 simulates TLC-class flash (lower cost, variable latency).  
- io2 mimics enterprise-grade MLC/SLC flash (consistent, high endurance).  
- Discuss workload mapping for each volume type.
