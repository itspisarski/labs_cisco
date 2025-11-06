# Lab 3: Flash Monitoring and Optimization in AWS
**Duration:** 60 minutes  
**Level:** Intermediate  

## 🎯 Learning Objectives
- Use monitoring tools to assess flash health and performance.  
- Tune EBS performance to optimize latency and throughput.  

## ⚙️ Steps
1. View SMART and NVMe metrics:  
   ```bash
   sudo nvme smart-log /dev/nvme0n1
   sudo smartctl -a /dev/nvme0n1
   ```
2. Simulate workload stress:  
   ```bash
   sudo fio --name=stress --filename=/data/testfile --rw=randwrite --bs=4k --size=2G --iodepth=32 --numjobs=4 --runtime=120 --time_based --group_reporting
   ```
3. Open CloudWatch > Metrics > EBS and observe IOPS, latency, and queue depth.  
4. Increase gp3 provisioned IOPS in the console, rerun fio, and compare performance.  

## 📊 Discussion
- Relate metrics to wear-leveling and write amplification.  
- Explain how AWS abstracts predictive failure analysis.  
- Discuss balancing cost vs performance using IOPS tuning.
