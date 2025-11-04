# Lab 05 - AWS Storage Gateway and Hybrid Cloud Integration

**Objective:** Implement hybrid storage solutions connecting on-premises infrastructure to AWS cloud storage.  
**Duration:** 90 minutes  
**Prerequisites:** AWS account, understanding of networking concepts, VM or on-premises server

---

## 🧩 Scenario
Your organization operates a hybrid environment with on-premises applications that need seamless integration with AWS cloud storage while maintaining low latency for frequently accessed data.

---

## ⚙️ Lab 5.1: File Gateway Deployment and Configuration

### Step 1: Deploy Storage Gateway VM
```bash
# Download Storage Gateway VM (simulated for lab - in real environment, deploy OVA)
echo "In production environment:"
echo "1. Download Storage Gateway OVA from AWS console"
echo "2. Deploy to VMware vSphere or Hyper-V"
echo "3. Configure 150GB root disk + 150GB cache disk"

# For this lab, we'll simulate with EC2 instance
# Launch Storage Gateway EC2 instance
aws ec2 run-instances \
  --image-id ami-0c02fb55956c7d316 \  # Amazon Linux 2 AMI
  --instance-type m5.xlarge \
  --key-name your-key-pair \
  --security-group-ids sg-xxxxxxxxx \
  --subnet-id subnet-xxxxxxxxx \
  --block-device-mappings '[
    {
      "DeviceName": "/dev/sda1",
      "Ebs": {"VolumeSize": 80, "VolumeType": "gp3"}
    },
    {
      "DeviceName": "/dev/sdf",
      "Ebs": {"VolumeSize": 150, "VolumeType": "gp3"}
    }
  ]' \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=StorageGateway}]' \
  --user-data file://gateway-setup.sh

# gateway-setup.sh content (create this file)
cat > gateway-setup.sh << 'EOF'
#!/bin/bash
yum update -y
yum install -y nfs-utils

# Format and mount cache disk
mkfs.ext4 /dev/nvme1n1
mkdir -p /opt/aws/storagegateway/cache
mount /dev/nvme1n1 /opt/aws/storagegateway/cache
echo "/dev/nvme1n1 /opt/aws/storagegateway/cache ext4 defaults,nofail 0 2" >> /etc/fstab

# Install and start Storage Gateway software
wget https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2021/03/11/aws-storage-gateway-installer.rpm
rpm -Uvh aws-storage-gateway-installer.rpm
systemctl enable storagegateway
systemctl start storagegateway
EOF

# Get instance IP for gateway activation
GATEWAY_INSTANCE_ID=$(aws ec2 describe-instances --filters "Name=tag:Name,Values=StorageGateway" --query 'Reservations[0].Instances[0].InstanceId' --output text)
GATEWAY_IP=$(aws ec2 describe-instances --instance-ids $GATEWAY_INSTANCE_ID --query 'Reservations[0].Instances[0].PublicIpAddress' --output text)

echo "Storage Gateway Instance IP: $GATEWAY_IP"
```

### Step 2: Activate File Gateway
```bash
# Wait for gateway to be ready (typically 2-3 minutes)
echo "Waiting for Storage Gateway to initialize..."
sleep 180

# Activate the gateway (replace with actual activation process)
echo "Activate gateway through AWS Console or CLI:"
echo "1. Navigate to Storage Gateway console"
echo "2. Choose 'Create gateway'"
echo "3. Select 'File gateway' type"
echo "4. Choose 'Amazon EC2' host platform"
echo "5. Provide gateway IP: $GATEWAY_IP"
echo "6. Complete activation wizard"

# For CLI activation (after gateway is ready)
ACTIVATION_KEY=$(curl -s "http://$GATEWAY_IP/?activationRegion=us-east-1&gatewayType=FILE_S3&no_redirect")
echo "Activation Key: $ACTIVATION_KEY"

# Create gateway (simulate - replace with actual gateway ARN after creation)
GATEWAY_ARN="arn:aws:storagegateway:us-east-1:123456789012:gateway/sgw-12345678"
echo "Gateway ARN (replace with actual): $GATEWAY_ARN"
```

### Step 3: Configure File Gateway Cache and S3 Bucket
```bash
# Create S3 bucket for file gateway
FILE_GW_BUCKET="file-gateway-lab-$(date +%s)-$(whoami)"
aws s3 mb s3://$FILE_GW_BUCKET --region us-east-1

echo "Created S3 bucket: $FILE_GW_BUCKET"

# Add cache disk to gateway (after activation)
# Note: In real scenario, you'd identify local disk ARN first
echo "Configure cache disk through AWS CLI:"
echo "aws storagegateway add-cache --gateway-arn $GATEWAY_ARN --disk-ids disk-12345678"

# Create NFS file share
echo "Creating NFS file share..."
echo "aws storagegateway create-nfs-file-share \\"
echo "  --client-token $(uuidgen) \\"
echo "  --gateway-arn $GATEWAY_ARN \\"
echo "  --location-arn arn:aws:s3:::$FILE_GW_BUCKET \\"
echo "  --role arn:aws:iam::123456789012:role/service-role/StorageGatewayBucketAccessRole \\"
echo "  --client-list 0.0.0.0/0 \\"
echo "  --default-storage-class S3_STANDARD"

# Simulate file share creation result
FILE_SHARE_ARN="arn:aws:storagegateway:us-east-1:123456789012:share/share-12345678"
echo "File Share ARN (simulated): $FILE_SHARE_ARN"
```

### Step 4: Test File Gateway Operations
```bash
# SSH to a client machine or use another EC2 instance
# Create test client instance
aws ec2 run-instances \
  --image-id ami-0c02fb55956c7d316 \
  --instance-type t3.medium \
  --key-name your-key-pair \
  --security-group-ids sg-xxxxxxxxx \
  --subnet-id subnet-xxxxxxxxx \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=NFS-Client}]' \
  --user-data '#!/bin/bash
yum update -y
yum install -y nfs-utils
mkdir -p /mnt/nfs-gateway'

# Get client instance IP
CLIENT_INSTANCE_ID=$(aws ec2 describe-instances --filters "Name=tag:Name,Values=NFS-Client" --query 'Reservations[0].Instances[0].InstanceId' --output text)
CLIENT_IP=$(aws ec2 describe-instances --instance-ids $CLIENT_INSTANCE_ID --query 'Reservations[0].Instances[0].PublicIpAddress' --output text)

echo "NFS Client IP: $CLIENT_IP"

# SSH to client and mount NFS share (after file share is created)
echo "On NFS client, run:"
echo "sudo mount -t nfs -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,intr,timeo=600,retrans=2 $GATEWAY_IP:/file-gateway-lab /mnt/nfs-gateway"

# Test file operations
echo "Test file operations:"
echo "sudo touch /mnt/nfs-gateway/test-file.txt"
echo "echo 'Hello from File Gateway' | sudo tee /mnt/nfs-gateway/test-file.txt"
echo "sudo mkdir /mnt/nfs-gateway/documents"
echo "sudo cp /etc/passwd /mnt/nfs-gateway/documents/"

# Verify files appear in S3
echo "Verify files in S3:"
echo "aws s3 ls s3://$FILE_GW_BUCKET/ --recursive"
```

---

## ⚙️ Lab 5.2: Volume Gateway (Stored Volumes) Configuration

### Step 1: Deploy Volume Gateway
```bash
# Create Volume Gateway instance
aws ec2 run-instances \
  --image-id ami-0c02fb55956c7d316 \
  --instance-type m5.xlarge \
  --key-name your-key-pair \
  --security-group-ids sg-xxxxxxxxx \
  --subnet-id subnet-xxxxxxxxx \
  --block-device-mappings '[
    {
      "DeviceName": "/dev/sda1",
      "Ebs": {"VolumeSize": 80, "VolumeType": "gp3"}
    },
    {
      "DeviceName": "/dev/sdf",
      "Ebs": {"VolumeSize": 150, "VolumeType": "gp3"}
    },
    {
      "DeviceName": "/dev/sdg",
      "Ebs": {"VolumeSize": 500, "VolumeType": "gp3"}
    }
  ]' \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=VolumeGateway}]' \
  --user-data file://volume-gateway-setup.sh

# volume-gateway-setup.sh
cat > volume-gateway-setup.sh << 'EOF'
#!/bin/bash
yum update -y

# Format cache disk
mkfs.ext4 /dev/nvme1n1
mkdir -p /opt/aws/storagegateway/cache
mount /dev/nvme1n1 /opt/aws/storagegateway/cache

# Prepare storage disk (don't format - will be used as raw storage)
# /dev/nvme2n1 will be used as stored volume

# Install Storage Gateway
wget https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2021/03/11/aws-storage-gateway-installer.rpm
rpm -Uvh aws-storage-gateway-installer.rpm
systemctl enable storagegateway
systemctl start storagegateway

# Install iSCSI initiator utils for testing
yum install -y iscsi-initiator-utils
EOF

# Get Volume Gateway IP
VOLUME_GW_INSTANCE_ID=$(aws ec2 describe-instances --filters "Name=tag:Name,Values=VolumeGateway" --query 'Reservations[0].Instances[0].InstanceId' --output text)
VOLUME_GW_IP=$(aws ec2 describe-instances --instance-ids $VOLUME_GW_INSTANCE_ID --query 'Reservations[0].Instances[0].PublicIpAddress' --output text)

echo "Volume Gateway IP: $VOLUME_GW_IP"
```

### Step 2: Configure Stored Volumes
```bash
# Wait for gateway initialization
sleep 180

# Activate Volume Gateway (stored volumes)
echo "Activate Volume Gateway through console or API"
echo "Gateway Type: VTL (Volume Gateway - Stored Volumes)"
echo "Gateway IP: $VOLUME_GW_IP"

# Simulate gateway activation
VOLUME_GATEWAY_ARN="arn:aws:storagegateway:us-east-1:123456789012:gateway/sgw-87654321"

# Create stored volume
echo "Creating stored volume..."
echo "aws storagegateway create-stored-iscsi-volume \\"
echo "  --gateway-arn $VOLUME_GATEWAY_ARN \\"
echo "  --disk-id disk-87654321 \\"
echo "  --preserve-existing-data false \\"
echo "  --target-name stored-volume-1 \\"
echo "  --network-interface-id 192.168.1.100"

# Simulate volume creation
STORED_VOLUME_ARN="arn:aws:storagegateway:us-east-1:123456789012:volume/vol-12345678"
echo "Stored Volume ARN: $STORED_VOLUME_ARN"
```

### Step 3: Test iSCSI Connection
```bash
# SSH to Volume Gateway and configure iSCSI target
ssh ec2-user@$VOLUME_GW_IP -i your-key.pem << 'EOF'
# Configure iSCSI target (this is typically done automatically by Storage Gateway)
sudo systemctl start tgtd
sudo systemctl enable tgtd

# List available disks
lsblk

# The stored volume will be presented as an iSCSI target
# Target name will be iqn.1997-05.com.amazon:stored-volume-1
EOF

# From client machine, discover and connect to iSCSI volume
ssh ec2-user@$CLIENT_IP -i your-key.pem << 'EOF'
# Install iSCSI initiator
sudo yum install -y iscsi-initiator-utils

# Configure iSCSI initiator
sudo vi /etc/iscsi/initiatorname.iscsi
# Set: InitiatorName=iqn.1993-08.org.debian:01:client

# Discover iSCSI targets
sudo iscsiadm --mode discovery --type sendtargets --portal $VOLUME_GW_IP:3260

# Connect to the target
sudo iscsiadm --mode node --targetname iqn.1997-05.com.amazon:stored-volume-1 --portal $VOLUME_GW_IP:3260 --login

# Check connected devices
lsblk

# Format and mount the iSCSI volume
sudo mkfs.ext4 /dev/sdb
sudo mkdir -p /mnt/iscsi-volume
sudo mount /dev/sdb /mnt/iscsi-volume

# Test write operations
sudo touch /mnt/iscsi-volume/test-file.txt
echo "Data written to stored volume" | sudo tee /mnt/iscsi-volume/test-file.txt
EOF
```

---

## ⚙️ Lab 5.3: Volume Gateway (Cached Volumes) Setup

### Step 1: Deploy Cached Volume Gateway
```bash
# Create another gateway instance for cached volumes
aws ec2 run-instances \
  --image-id ami-0c02fb55956c7d316 \
  --instance-type m5.xlarge \
  --key-name your-key-pair \
  --security-group-ids sg-xxxxxxxxx \
  --subnet-id subnet-xxxxxxxxx \
  --block-device-mappings '[
    {
      "DeviceName": "/dev/sda1",
      "Ebs": {"VolumeSize": 80, "VolumeType": "gp3"}
    },
    {
      "DeviceName": "/dev/sdf",
      "Ebs": {"VolumeSize": 300, "VolumeType": "gp3"}
    }
  ]' \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=CachedVolumeGateway}]'

# Get Cached Volume Gateway IP
CACHED_GW_INSTANCE_ID=$(aws ec2 describe-instances --filters "Name=tag:Name,Values=CachedVolumeGateway" --query 'Reservations[0].Instances[0].InstanceId' --output text)
CACHED_GW_IP=$(aws ec2 describe-instances --instance-ids $CACHED_GW_INSTANCE_ID --query 'Reservations[0].Instances[0].PublicIpAddress' --output text)

echo "Cached Volume Gateway IP: $CACHED_GW_IP"
```

### Step 2: Configure Cached Volumes
```bash
# Activate gateway for cached volumes
echo "Activate gateway for VTL (Volume Gateway - Cached Volumes)"
echo "Gateway IP: $CACHED_GW_IP"

# Simulate cached volume gateway ARN
CACHED_GATEWAY_ARN="arn:aws:storagegateway:us-east-1:123456789012:gateway/sgw-cached123"

# Create cached volume
echo "Creating cached volume..."
echo "aws storagegateway create-cached-iscsi-volume \\"
echo "  --gateway-arn $CACHED_GATEWAY_ARN \\"
echo "  --volume-size-in-bytes 1073741824000 \\"  # 1TB
echo "  --snapshot-id snap-12345678 \\"
echo "  --target-name cached-volume-1 \\"
echo "  --network-interface-id 192.168.1.101"

# Simulate cached volume ARN
CACHED_VOLUME_ARN="arn:aws:storagegateway:us-east-1:123456789012:volume/vol-cached123"
echo "Cached Volume ARN: $CACHED_VOLUME_ARN"
```

---

## ⚙️ Lab 5.4: Tape Gateway (VTL) Configuration

### Step 1: Deploy Tape Gateway
```bash
# Create Tape Gateway instance
aws ec2 run-instances \
  --image-id ami-0c02fb55956c7d316 \
  --instance-type m5.xlarge \
  --key-name your-key-pair \
  --security-group-ids sg-xxxxxxxxx \
  --subnet-id subnet-xxxxxxxxx \
  --block-device-mappings '[
    {
      "DeviceName": "/dev/sda1",
      "Ebs": {"VolumeSize": 80, "VolumeType": "gp3"}
    },
    {
      "DeviceName": "/dev/sdf",
      "Ebs": {"VolumeSize": 300, "VolumeType": "gp3"}
    },
    {
      "DeviceName": "/dev/sdg",
      "Ebs": {"VolumeSize": 150, "VolumeType": "gp3"}
    }
  ]' \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=TapeGateway}]'

# Get Tape Gateway IP
TAPE_GW_INSTANCE_ID=$(aws ec2 describe-instances --filters "Name=tag:Name,Values=TapeGateway" --query 'Reservations[0].Instances[0].InstanceId' --output text)
TAPE_GW_IP=$(aws ec2 describe-instances --instance-ids $TAPE_GW_INSTANCE_ID --query 'Reservations[0].Instances[0].PublicIpAddress' --output text)

echo "Tape Gateway IP: $TAPE_GW_IP"
```

### Step 2: Create Virtual Tapes
```bash
# Activate Tape Gateway (VTL)
echo "Activate gateway for VTL (Virtual Tape Library)"
echo "Gateway IP: $TAPE_GW_IP"

# Simulate tape gateway ARN
TAPE_GATEWAY_ARN="arn:aws:storagegateway:us-east-1:123456789012:gateway/sgw-tape123"

# Create virtual tapes
echo "Creating virtual tapes..."
for i in {1..5}; do
  echo "aws storagegateway create-tapes \\"
  echo "  --gateway-arn $TAPE_GATEWAY_ARN \\"
  echo "  --tape-size-in-bytes 107374182400 \\"  # 100GB
  echo "  --client-token tape-$i-$(date +%s) \\"
  echo "  --num-tapes-to-create 1 \\"
  echo "  --tape-barcode-prefix TAPE00$i"
done

# List virtual tapes
echo "Listing virtual tapes:"
echo "aws storagegateway list-tapes --gateway-arn $TAPE_GATEWAY_ARN"
```

### Step 3: Test Backup Software Integration
```bash
# Install backup software (simulated with tar for demonstration)
ssh ec2-user@$TAPE_GW_IP -i your-key.pem << 'EOF'
# Install backup tools
sudo yum install -y mt-st tar

# List tape devices (provided by VTL)
ls -la /dev/st*
ls -la /dev/nst*

# Test tape operations
sudo mt -f /dev/st0 status
sudo mt -f /dev/st0 rewind

# Create test backup
echo "Creating test backup to virtual tape..."
sudo tar -cvf /dev/st0 /etc/

# Check tape position
sudo mt -f /dev/st0 tell

# Rewind tape
sudo mt -f /dev/st0 rewind

# Test restore
mkdir -p /tmp/restore
cd /tmp/restore
sudo tar -xvf /dev/st0

# Eject tape
sudo mt -f /dev/st0 offline
EOF
```

---

## ⚙️ Lab 5.5: Monitoring and Performance Optimization

### Step 1: CloudWatch Metrics for Storage Gateway
```bash
# Enable detailed monitoring
aws logs create-log-group --log-group-name /aws/storagegateway/gateway

# Create CloudWatch alarms for key metrics
aws cloudwatch put-metric-alarm \
  --alarm-name "StorageGateway-HighCacheUtilization" \
  --alarm-description "Cache utilization is high" \
  --metric-name CachePercentUsed \
  --namespace AWS/StorageGateway \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=GatewayName,Value=FileGateway \
  --evaluation-periods 2

# Monitor file share metrics
aws cloudwatch put-metric-alarm \
  --alarm-name "FileShare-HighReadLatency" \
  --alarm-description "File share read latency is high" \
  --metric-name ReadTime \
  --namespace AWS/StorageGateway \
  --statistic Average \
  --period 300 \
  --threshold 100 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2

# Get gateway metrics
echo "Retrieving Storage Gateway metrics:"
aws cloudwatch get-metric-statistics \
  --namespace AWS/StorageGateway \
  --metric-name CachePercentUsed \
  --dimensions Name=GatewayName,Value=FileGateway \
  --statistics Average \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 3600
```

### Step 2: Performance Tuning
```bash
# Optimize cache disk performance
ssh ec2-user@$GATEWAY_IP -i your-key.pem << 'EOF'
# Check current cache disk performance
sudo iostat -x 1 5

# Optimize cache disk mount options
sudo umount /opt/aws/storagegateway/cache
sudo mount -o noatime,data=writeback /dev/nvme1n1 /opt/aws/storagegateway/cache

# Update /etc/fstab for persistent optimization
sudo sed -i 's/defaults,nofail/defaults,nofail,noatime,data=writeback/' /etc/fstab

# Monitor network performance
sudo iftop -i eth0

# Check gateway logs
sudo tail -f /opt/aws/storagegateway/logs/StorageGateway-sgw-*.log
EOF

# Test performance improvements
echo "Testing file operations after optimization..."
time ssh ec2-user@$CLIENT_IP -i your-key.pem << 'EOF'
# Large file test
dd if=/dev/zero of=/mnt/nfs-gateway/largefile bs=1M count=1000
sync
time ls -la /mnt/nfs-gateway/largefile
EOF

# Monitor S3 upload patterns
aws s3api list-multipart-uploads --bucket $FILE_GW_BUCKET
```

---

## 🔍 Verification Checklist

- [ ] File Gateway successfully deployed and mounted via NFS
- [ ] Files written to NFS share appear in S3 bucket
- [ ] Volume Gateway (Stored) configured with iSCSI connectivity
- [ ] Volume Gateway (Cached) deployed for large dataset access
- [ ] Tape Gateway (VTL) created virtual tapes for backup
- [ ] CloudWatch monitoring configured for all gateways
- [ ] Performance optimization applied to cache disks
- [ ] Backup and restore operations tested

---

## 📊 Performance Comparison Results

### Gateway Types Performance
| Gateway Type | Use Case | Latency | Bandwidth | Storage Location |
|-------------|----------|---------|-----------|------------------|
| File Gateway | File shares | _____ ms | _____ MB/s | S3 |
| Stored Volumes | Primary storage | _____ ms | _____ MB/s | Local + S3 snapshots |
| Cached Volumes | Archive access | _____ ms | _____ MB/s | S3 + Local cache |
| Tape Gateway | Backup/Archive | _____ ms | _____ MB/s | S3 Glacier/Deep Archive |

### Cost Analysis
| Solution | Monthly Cost (1TB) | Backup Cost | Use Case |
|----------|-------------------|-------------|----------|
| File Gateway + S3 Standard | $23 | N/A | Active file sharing |
| Stored Volumes + Snapshots | $40 + snapshot costs | Included | Primary storage |
| Cached Volumes | $23 (S3) + cache costs | N/A | Large datasets |
| VTL + Glacier | $4 (Glacier) | $0.05/GB | Backup/Archive |

---

## 💡 Key Learning Outcomes

### Hybrid Storage Benefits:
1. **Seamless Integration**: On-premises applications access cloud storage transparently
2. **Cost Optimization**: Frequently accessed data cached locally, all data stored in cloud
3. **Scalability**: Unlimited cloud storage without hardware investments
4. **Disaster Recovery**: Built-in backup and replication to AWS

### Gateway Selection Criteria:
1. **File Gateway**: For file shares, content distribution, backup to cloud
2. **Stored Volumes**: For primary storage with cloud backup
3. **Cached Volumes**: For frequently accessed subset of large datasets
4. **Tape Gateway**: For replacing physical tape infrastructure

### Performance Optimization:
1. **Cache Sizing**: 1.5x frequently accessed data
2. **Network Bandwidth**: Dedicated connection for large transfers
3. **Disk Performance**: High-IOPS SSDs for cache
4. **Monitoring**: CloudWatch metrics for proactive management

---

**Next Lab:** Amazon FSx for High-Performance File Systems