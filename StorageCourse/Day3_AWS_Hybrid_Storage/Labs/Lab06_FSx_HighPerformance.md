# Lab 06 - Amazon FSx High-Performance File Systems

**Objective:** Deploy and configure Amazon FSx for different high-performance computing and enterprise workloads.  
**Duration:** 90 minutes  
**Prerequisites:** AWS account, understanding of file system concepts, basic networking knowledge

---

## 🧩 Scenario
Your organization needs high-performance file systems for different workloads: Windows-based applications requiring SMB shares, Linux HPC clusters needing Lustre performance, and NetApp ONTAP compatibility for enterprise applications.

---

## ⚙️ Lab 6.1: Amazon FSx for Windows File Server

### Step 1: Create FSx for Windows File Server
```bash
# Get VPC and subnet information
VPC_ID=$(aws ec2 describe-vpcs --filters "Name=isDefault,Values=true" --query 'Vpcs[0].VpcId' --output text)
SUBNET_IDS=$(aws ec2 describe-subnets --filters "Name=vpc-id,Values=$VPC_ID" --query 'Subnets[0:2].SubnetId' --output text | tr '\t' ' ')
SUBNET_ID_1=$(echo $SUBNET_IDS | cut -d' ' -f1)
SUBNET_ID_2=$(echo $SUBNET_IDS | cut -d' ' -f2)

echo "VPC ID: $VPC_ID"
echo "Subnet IDs: $SUBNET_ID_1, $SUBNET_ID_2"

# Create security group for FSx Windows
FSX_WINDOWS_SG=$(aws ec2 create-security-group \
  --group-name fsx-windows-sg \
  --description "Security group for FSx Windows File Server" \
  --vpc-id $VPC_ID \
  --query 'GroupId' --output text)

# Add required ports for FSx Windows
aws ec2 authorize-security-group-ingress --group-id $FSX_WINDOWS_SG --protocol tcp --port 445 --source-group $FSX_WINDOWS_SG  # SMB
aws ec2 authorize-security-group-ingress --group-id $FSX_WINDOWS_SG --protocol tcp --port 5985 --source-group $FSX_WINDOWS_SG  # WinRM HTTP
aws ec2 authorize-security-group-ingress --group-id $FSX_WINDOWS_SG --protocol tcp --port 5986 --source-group $FSX_WINDOWS_SG  # WinRM HTTPS
aws ec2 authorize-security-group-ingress --group-id $FSX_WINDOWS_SG --protocol udp --port 53 --source-group $FSX_WINDOWS_SG     # DNS

# Create FSx for Windows File Server
FSX_WINDOWS_ID=$(aws fsx create-file-system \
  --file-system-type Windows \
  --storage-capacity 32 \
  --storage-type SSD \
  --subnet-ids $SUBNET_ID_1 \
  --security-group-ids $FSX_WINDOWS_SG \
  --windows-configuration '{
    "ThroughputCapacity": 8,
    "WeeklyMaintenanceStartTime": "7:00:00",
    "DailyAutomaticBackupStartTime": "05:00",
    "AutomaticBackupRetentionDays": 7,
    "CopyTagsToBackups": true,
    "DeploymentType": "SINGLE_AZ_2"
  }' \
  --tags Key=Name,Value=FSx-Windows-Lab \
  --query 'FileSystem.FileSystemId' --output text)

echo "FSx Windows File System ID: $FSX_WINDOWS_ID"

# Wait for file system to be available
echo "Waiting for FSx Windows file system to be available (this may take 10-15 minutes)..."
aws fsx wait file-system-available --file-system-ids $FSX_WINDOWS_ID
```

### Step 2: Launch Windows EC2 Instance for Testing
```bash
# Launch Windows Server instance
WINDOWS_INSTANCE_ID=$(aws ec2 run-instances \
  --image-id ami-0c02fb55956c7d316 \  # Replace with Windows Server AMI
  --instance-type t3.medium \
  --key-name your-key-pair \
  --security-group-ids $FSX_WINDOWS_SG \
  --subnet-id $SUBNET_ID_1 \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=Windows-FSx-Client}]' \
  --query 'Instances[0].InstanceId' --output text)

# Wait for instance to be running
aws ec2 wait instance-running --instance-ids $WINDOWS_INSTANCE_ID

# Get instance IP
WINDOWS_INSTANCE_IP=$(aws ec2 describe-instances --instance-ids $WINDOWS_INSTANCE_ID --query 'Reservations[0].Instances[0].PrivateIpAddress' --output text)

echo "Windows Instance ID: $WINDOWS_INSTANCE_ID"
echo "Windows Instance IP: $WINDOWS_INSTANCE_IP"

# Get FSx DNS name
FSX_WINDOWS_DNS=$(aws fsx describe-file-systems --file-system-ids $FSX_WINDOWS_ID --query 'FileSystems[0].DNSName' --output text)
echo "FSx Windows DNS Name: $FSX_WINDOWS_DNS"
```

### Step 3: Mount and Test FSx Windows File Server
```powershell
# PowerShell commands to run on Windows client
# RDP to Windows instance and run these commands

# Map network drive to FSx file system
net use Z: \\$FSX_WINDOWS_DNS\share /persistent:yes

# Test file operations
New-Item -Path "Z:\test-folder" -ItemType Directory
New-Item -Path "Z:\test-folder\test-file.txt" -ItemType File -Value "Hello from FSx Windows"

# Test performance with large file
$largefile = "Z:\test-folder\large-file.dat"
$size = 1GB
$buffer = New-Object byte[] 1MB
$file = [System.IO.File]::Create($largefile)
$stopwatch = [System.Diagnostics.Stopwatch]::StartNew()
for ($i = 0; $i -lt ($size / 1MB); $i++) {
    $file.Write($buffer, 0, $buffer.Length)
}
$file.Close()
$stopwatch.Stop()
Write-Host "Write Performance: $($size / $stopwatch.Elapsed.TotalSeconds / 1MB) MB/s"

# Test read performance
$stopwatch = [System.Diagnostics.Stopwatch]::StartNew()
$content = Get-Content $largefile -Raw
$stopwatch.Stop()
Write-Host "Read Performance: $($size / $stopwatch.Elapsed.TotalSeconds / 1MB) MB/s"

# Check disk usage
Get-WmiObject -Class Win32_LogicalDisk | Where-Object {$_.DeviceID -eq "Z:"}
```

---

## ⚙️ Lab 6.2: Amazon FSx for Lustre (High-Performance Computing)

### Step 1: Create S3 Bucket for Lustre Integration
```bash
# Create S3 bucket for Lustre data repository
LUSTRE_BUCKET="fsx-lustre-lab-$(date +%s)-$(whoami)"
aws s3 mb s3://$LUSTRE_BUCKET --region us-east-1

# Upload sample HPC data
echo "Creating sample HPC dataset..."
for i in {1..100}; do
  echo "Sample HPC data file $i - $(date)" > hpc-data-$i.txt
  aws s3 cp hpc-data-$i.txt s3://$LUSTRE_BUCKET/input-data/hpc-data-$i.txt
done

echo "Created Lustre S3 bucket: $LUSTRE_BUCKET"
```

### Step 2: Create FSx for Lustre File System
```bash
# Create FSx for Lustre
FSX_LUSTRE_ID=$(aws fsx create-file-system \
  --file-system-type Lustre \
  --storage-capacity 1200 \
  --subnet-ids $SUBNET_ID_1 \
  --security-group-ids $FSX_WINDOWS_SG \
  --lustre-configuration '{
    "ImportPath": "s3://'$LUSTRE_BUCKET'/input-data",
    "ExportPath": "s3://'$LUSTRE_BUCKET'/output-data",
    "ImportedFileChunkSize": 1024,
    "DeploymentType": "SCRATCH_2",
    "PerUnitStorageThroughput": 200
  }' \
  --tags Key=Name,Value=FSx-Lustre-Lab \
  --query 'FileSystem.FileSystemId' --output text)

echo "FSx Lustre File System ID: $FSX_LUSTRE_ID"

# Wait for Lustre file system to be available
echo "Waiting for FSx Lustre file system to be available (this may take 5-10 minutes)..."
aws fsx wait file-system-available --file-system-ids $FSX_LUSTRE_ID

# Get Lustre mount name and DNS
FSX_LUSTRE_DNS=$(aws fsx describe-file-systems --file-system-ids $FSX_LUSTRE_ID --query 'FileSystems[0].DNSName' --output text)
FSX_LUSTRE_MOUNT_NAME=$(aws fsx describe-file-systems --file-system-ids $FSX_LUSTRE_ID --query 'FileSystems[0].LustreConfiguration.MountName' --output text)

echo "FSx Lustre DNS: $FSX_LUSTRE_DNS"
echo "FSx Lustre Mount Name: $FSX_LUSTRE_MOUNT_NAME"
```

### Step 3: Launch HPC Compute Instances
```bash
# Launch multiple compute instances for HPC cluster simulation
HPC_INSTANCE_IDS=()
for i in {1..3}; do
  INSTANCE_ID=$(aws ec2 run-instances \
    --image-id ami-0c02fb55956c7d316 \
    --instance-type c5n.large \
    --key-name your-key-pair \
    --security-group-ids $FSX_WINDOWS_SG \
    --subnet-id $SUBNET_ID_1 \
    --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=HPC-Node-$i}]" \
    --user-data '#!/bin/bash
yum update -y
amazon-linux-extras install -y lustre2.10
mkdir -p /mnt/fsx' \
    --query 'Instances[0].InstanceId' --output text)
  
  HPC_INSTANCE_IDS+=($INSTANCE_ID)
  echo "Created HPC instance $i: $INSTANCE_ID"
done

# Wait for all instances to be running
for instance_id in "${HPC_INSTANCE_IDS[@]}"; do
  aws ec2 wait instance-running --instance-ids $instance_id
done
```

### Step 4: Mount Lustre and Run HPC Workload
```bash
# SSH to first HPC instance and mount Lustre
HPC_NODE_IP=$(aws ec2 describe-instances --instance-ids ${HPC_INSTANCE_IDS[0]} --query 'Reservations[0].Instances[0].PublicIpAddress' --output text)

ssh ec2-user@$HPC_NODE_IP -i your-key.pem << EOF
# Mount Lustre file system
sudo mount -t lustre $FSX_LUSTRE_DNS@tcp:/$FSX_LUSTRE_MOUNT_NAME /mnt/fsx

# Verify mount and check imported data
ls -la /mnt/fsx/
wc -l /mnt/fsx/*.txt

# Install performance testing tools
sudo yum install -y ior-hpc

# Test Lustre performance with IOR benchmark
mkdir -p /mnt/fsx/ior-test
mpirun -np 4 ior -a POSIX -w -r -i 5 -g -t 1m -b 100m -o /mnt/fsx/ior-test/ior_file

# Test file system throughput
echo "Testing sequential write performance..."
time dd if=/dev/zero of=/mnt/fsx/sequential_test bs=1M count=1000 conv=fdatasync

echo "Testing sequential read performance..."  
time dd if=/mnt/fsx/sequential_test of=/dev/null bs=1M

# Test random I/O performance
sudo yum install -y fio
fio --name=lustre-random --filename=/mnt/fsx/fio_test --size=1G --ioengine=libaio --bs=4k --rw=randrw --direct=1 --runtime=60 --group_reporting

# Simulate HPC computation workload
echo "Running simulated HPC workload..."
for i in {1..10}; do
  input_file="/mnt/fsx/hpc-data-\$i.txt"
  output_file="/mnt/fsx/processed-data-\$i.txt"
  
  # Simulate data processing
  if [ -f \$input_file ]; then
    wc -w \$input_file > \$output_file
    echo "Processed: \$(date)" >> \$output_file
  fi
done

# Check processed files
ls -la /mnt/fsx/processed-*.txt
EOF
```

---

## ⚙️ Lab 6.3: Amazon FSx for NetApp ONTAP

### Step 1: Create FSx for NetApp ONTAP File System
```bash
# Create FSx for NetApp ONTAP
FSX_ONTAP_ID=$(aws fsx create-file-system \
  --file-system-type ONTAP \
  --storage-capacity 1024 \
  --subnet-ids $SUBNET_ID_1 $SUBNET_ID_2 \
  --security-group-ids $FSX_WINDOWS_SG \
  --ontap-configuration '{
    "DeploymentType": "MULTI_AZ_1",
    "ThroughputCapacity": 128,
    "WeeklyMaintenanceStartTime": "7:00:00",
    "FsxAdminPassword": "MyStrongPassword123!",
    "PreferredSubnetId": "'$SUBNET_ID_1'"
  }' \
  --tags Key=Name,Value=FSx-ONTAP-Lab \
  --query 'FileSystem.FileSystemId' --output text)

echo "FSx ONTAP File System ID: $FSX_ONTAP_ID"

# Wait for ONTAP file system to be available
echo "Waiting for FSx ONTAP file system to be available (this may take 20-30 minutes)..."
aws fsx wait file-system-available --file-system-ids $FSX_ONTAP_ID

# Get ONTAP management endpoints
FSX_ONTAP_MGMT_DNS=$(aws fsx describe-file-systems --file-system-ids $FSX_ONTAP_ID --query 'FileSystems[0].OntapConfiguration.Endpoints.Management.DNSName' --output text)
FSX_ONTAP_INTERCLUSTER_DNS=$(aws fsx describe-file-systems --file-system-ids $FSX_ONTAP_ID --query 'FileSystems[0].OntapConfiguration.Endpoints.Intercluster.DNSName' --output text)

echo "FSx ONTAP Management DNS: $FSX_ONTAP_MGMT_DNS"
echo "FSx ONTAP Intercluster DNS: $FSX_ONTAP_INTERCLUSTER_DNS"
```

### Step 2: Create Storage Virtual Machine (SVM)
```bash
# Create SVM
SVM_ID=$(aws fsx create-storage-virtual-machine \
  --file-system-id $FSX_ONTAP_ID \
  --name "lab-svm" \
  --svm-admin-password "SVMPassword123!" \
  --tags Key=Name,Value=Lab-SVM \
  --query 'StorageVirtualMachine.StorageVirtualMachineId' --output text)

echo "Storage Virtual Machine ID: $SVM_ID"

# Wait for SVM to be created
echo "Waiting for SVM to be created..."
aws fsx wait storage-virtual-machine-created --storage-virtual-machine-ids $SVM_ID

# Get SVM endpoints
SVM_NFS_DNS=$(aws fsx describe-storage-virtual-machines --storage-virtual-machine-ids $SVM_ID --query 'StorageVirtualMachines[0].Endpoints.Nfs.DNSName' --output text)
SVM_SMB_DNS=$(aws fsx describe-storage-virtual-machines --storage-virtual-machine-ids $SVM_ID --query 'StorageVirtualMachines[0].Endpoints.Cifs.DNSName' --output text)

echo "SVM NFS DNS: $SVM_NFS_DNS"  
echo "SVM SMB DNS: $SVM_SMB_DNS"
```

### Step 3: Create Volumes
```bash
# Create NFS volume
NFS_VOLUME_ID=$(aws fsx create-volume \
  --volume-type ONTAP \
  --name "nfs-volume" \
  --ontap-configuration '{
    "StorageVirtualMachineId": "'$SVM_ID'",
    "JunctionPath": "/nfs-vol",
    "SecurityStyle": "UNIX",
    "SizeInMegabytes": 10240,
    "StorageEfficiencyEnabled": true,
    "TieringPolicy": {
      "CoolingPeriod": 31,
      "Name": "SNAPSHOT_ONLY"
    }
  }' \
  --tags Key=Name,Value=NFS-Volume \
  --query 'Volume.VolumeId' --output text)

# Create SMB volume
SMB_VOLUME_ID=$(aws fsx create-volume \
  --volume-type ONTAP \
  --name "smb-volume" \
  --ontap-configuration '{
    "StorageVirtualMachineId": "'$SVM_ID'",
    "JunctionPath": "/smb-vol",
    "SecurityStyle": "NTFS",
    "SizeInMegabytes": 10240,
    "StorageEfficiencyEnabled": true
  }' \
  --tags Key=Name,Value=SMB-Volume \
  --query 'Volume.VolumeId' --output text)

echo "NFS Volume ID: $NFS_VOLUME_ID"
echo "SMB Volume ID: $SMB_VOLUME_ID"

# Wait for volumes to be created
aws fsx wait volume-created --volume-ids $NFS_VOLUME_ID $SMB_VOLUME_ID
```

### Step 4: Test ONTAP Features
```bash
# Launch Linux client for NFS testing
LINUX_CLIENT_ID=$(aws ec2 run-instances \
  --image-id ami-0c02fb55956c7d316 \
  --instance-type t3.medium \
  --key-name your-key-pair \
  --security-group-ids $FSX_WINDOWS_SG \
  --subnet-id $SUBNET_ID_1 \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=Linux-ONTAP-Client}]' \
  --user-data '#!/bin/bash
yum update -y
yum install -y nfs-utils' \
  --query 'Instances[0].InstanceId' --output text)

aws ec2 wait instance-running --instance-ids $LINUX_CLIENT_ID
LINUX_CLIENT_IP=$(aws ec2 describe-instances --instance-ids $LINUX_CLIENT_ID --query 'Reservations[0].Instances[0].PublicIpAddress' --output text)

# Test NFS mount
ssh ec2-user@$LINUX_CLIENT_IP -i your-key.pem << EOF
# Mount NFS volume
sudo mkdir -p /mnt/ontap-nfs
sudo mount -t nfs -o nfsvers=4.1,rsize=1048576,wsize=1048576 $SVM_NFS_DNS:/nfs-vol /mnt/ontap-nfs

# Test NFS operations
sudo touch /mnt/ontap-nfs/nfs-test-file.txt
echo "Hello from ONTAP NFS" | sudo tee /mnt/ontap-nfs/nfs-test-file.txt

# Create test data for deduplication
sudo mkdir -p /mnt/ontap-nfs/dedup-test
for i in {1..10}; do
  echo "This is duplicate data for deduplication testing" | sudo tee /mnt/ontap-nfs/dedup-test/file-\$i.txt
done

# Test snapshots (via ONTAP CLI)
echo "Snapshot operations would be performed via ONTAP CLI or REST API"
EOF

echo "Linux ONTAP Client IP: $LINUX_CLIENT_IP"
```

---

## ⚙️ Lab 6.4: Performance Testing and Monitoring

### Step 1: Comprehensive Performance Testing
```bash
# Test all FSx file systems performance
echo "=== FSx Performance Testing Results ==="

# FSx Windows performance (run on Windows client)
echo "FSx Windows File Server Performance:"
echo "Run CrystalDiskMark or similar tool on Windows client"

# FSx Lustre performance (already tested in previous steps)
ssh ec2-user@$HPC_NODE_IP -i your-key.pem << 'EOF'
echo "FSx Lustre Performance Results:"
echo "Sequential Write: $(dd if=/dev/zero of=/mnt/fsx/perf_test_write bs=1M count=1000 conv=fdatasync 2>&1 | grep -o '[0-9.]* MB/s')"
echo "Sequential Read: $(dd if=/mnt/fsx/perf_test_write of=/dev/null bs=1M 2>&1 | grep -o '[0-9.]* MB/s')"

# Test parallel access
echo "Testing parallel Lustre access..."
for i in {1..4}; do
  (dd if=/dev/zero of=/mnt/fsx/parallel_$i bs=1M count=250 conv=fdatasync 2>&1 | grep -o '[0-9.]* MB/s') &
done
wait
EOF

# FSx ONTAP performance
ssh ec2-user@$LINUX_CLIENT_IP -i your-key.pem << 'EOF'
echo "FSx ONTAP NFS Performance:"
echo "Sequential Write: $(dd if=/dev/zero of=/mnt/ontap-nfs/perf_test_write bs=1M count=500 conv=fdatasync 2>&1 | grep -o '[0-9.]* MB/s')"  
echo "Sequential Read: $(dd if=/mnt/ontap-nfs/perf_test_write of=/dev/null bs=1M 2>&1 | grep -o '[0-9.]* MB/s')"
EOF
```

### Step 2: CloudWatch Monitoring Setup
```bash
# Create CloudWatch alarms for FSx metrics
aws cloudwatch put-metric-alarm \
  --alarm-name "FSx-Lustre-HighThroughputUtilization" \
  --alarm-description "FSx Lustre throughput utilization is high" \
  --metric-name ThroughputUtilization \
  --namespace AWS/FSx \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=FileSystemId,Value=$FSX_LUSTRE_ID \
  --evaluation-periods 2

# Monitor storage capacity
aws cloudwatch put-metric-alarm \
  --alarm-name "FSx-Windows-HighStorageUtilization" \
  --alarm-description "FSx Windows storage utilization is high" \
  --metric-name StorageUtilization \
  --namespace AWS/FSx \
  --statistic Average \
  --period 300 \
  --threshold 85 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=FileSystemId,Value=$FSX_WINDOWS_ID \
  --evaluation-periods 2

# Get current metrics
echo "Current FSx Metrics:"
aws cloudwatch get-metric-statistics \
  --namespace AWS/FSx \
  --metric-name DataReadBytes \
  --dimensions Name=FileSystemId,Value=$FSX_LUSTRE_ID \
  --statistics Sum \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 3600
```

### Step 3: Cost Analysis and Optimization
```bash
# Calculate FSx costs
echo "=== FSx Cost Analysis ==="

# Get file system details for cost calculation
FSX_WINDOWS_CAPACITY=$(aws fsx describe-file-systems --file-system-ids $FSX_WINDOWS_ID --query 'FileSystems[0].StorageCapacity' --output text)
FSX_LUSTRE_CAPACITY=$(aws fsx describe-file-systems --file-system-ids $FSX_LUSTRE_ID --query 'FileSystems[0].StorageCapacity' --output text)
FSX_ONTAP_CAPACITY=$(aws fsx describe-file-systems --file-system-ids $FSX_ONTAP_ID --query 'FileSystems[0].StorageCapacity' --output text)

echo "Monthly FSx Costs (estimated):"
echo "FSx Windows (${FSX_WINDOWS_CAPACITY}GB): \$$(echo "scale=2; $FSX_WINDOWS_CAPACITY * 0.13" | bc)"
echo "FSx Lustre (${FSX_LUSTRE_CAPACITY}GB): \$$(echo "scale=2; $FSX_LUSTRE_CAPACITY * 0.145" | bc)"  
echo "FSx ONTAP (${FSX_ONTAP_CAPACITY}GB): \$$(echo "scale=2; $FSX_ONTAP_CAPACITY * 0.163" | bc)"

# Backup costs
echo "Backup costs (per GB/month):"
echo "FSx Windows automatic backups: \$0.05/GB"
echo "FSx ONTAP snapshots: \$0.05/GB"
echo "FSx Lustre (no automatic backups): \$0/GB"
```

---

## 🔍 Verification Checklist

- [ ] FSx for Windows File Server deployed and mounted via SMB
- [ ] FSx for Lustre integrated with S3 and mounted on HPC nodes
- [ ] FSx for NetApp ONTAP with SVM and volumes created
- [ ] Performance testing completed for all FSx types
- [ ] CloudWatch monitoring configured with alarms
- [ ] Cost analysis completed for different FSx options
- [ ] HPC workload simulation executed on Lustre
- [ ] ONTAP enterprise features tested (deduplication, snapshots)

---

## 📊 Performance Comparison Results

### FSx File System Performance
| FSx Type | Sequential Read | Sequential Write | Random 4K IOPS | Latency | Use Case |
|----------|----------------|------------------|-----------------|---------|----------|
| Windows | _____ MB/s | _____ MB/s | _____ IOPS | _____ ms | SMB file shares |
| Lustre | _____ MB/s | _____ MB/s | _____ IOPS | _____ ms | HPC workloads |
| ONTAP | _____ MB/s | _____ MB/s | _____ IOPS | _____ ms | Enterprise NAS |

### Cost Comparison (per GB/month)
| FSx Type | Storage Cost | Throughput Cost | Backup Cost | Total Cost |
|----------|-------------|-----------------|-------------|------------|
| Windows | $0.13 | $2.20/MBps | $0.05 | $_____ |
| Lustre Scratch | $0.145 | Included | N/A | $0.145 |
| Lustre Persistent | $0.145 | $2.20/MBps | $0.05 | $_____ |
| ONTAP | $0.163 | $1.17/MBps | $0.05 | $_____ |

---

## 💡 Key Learning Outcomes

### FSx Selection Criteria:
1. **FSx for Windows**: SMB protocol, Active Directory integration, Windows applications
2. **FSx for Lustre**: HPC workloads, high throughput computing, S3 integration
3. **FSx for NetApp ONTAP**: Multi-protocol (NFS/SMB/iSCSI), enterprise features, data management

### Performance Optimization:
1. **Throughput Capacity**: Choose based on workload requirements
2. **Storage Type**: SSD for performance, HDD for capacity
3. **Deployment Type**: Single-AZ vs Multi-AZ for availability requirements
4. **Client Configuration**: Optimize mount options and client-side caching

### Enterprise Features:
1. **Backup and Restore**: Automatic backups, point-in-time recovery
2. **Data Deduplication**: Reduce storage costs (ONTAP)  
3. **Compression**: Improve storage efficiency (ONTAP)
4. **Snapshots**: Fast backup and restore operations
5. **Data Tiering**: Automatic movement to lower-cost storage

---

**Next Lab:** Data Protection and Disaster Recovery Strategies