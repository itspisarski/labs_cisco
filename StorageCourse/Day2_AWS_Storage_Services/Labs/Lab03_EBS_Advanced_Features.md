# Lab 03 - EBS Advanced Features and Optimization

**Objective:** Master EBS volume management, snapshots, and performance optimization techniques.  
**Duration:** 90 minutes  
**Prerequisites:** AWS account, EC2 instance running

---

## 🧩 Scenario
You're managing storage for a critical database application that requires high availability, backup capabilities, and performance optimization.

---

## ⚙️ Lab 3.1: EBS Snapshot Management

### Step 1: Create Baseline Volume with Data
```bash
# SSH to your EC2 instance
ssh ec2-user@<instance-ip> -i your-key.pem

# Create test database simulation
sudo yum install -y mysql
sudo mkdir -p /mnt/database
sudo mount /dev/xvdf /mnt/database  # Assuming gp3 volume from previous lab

# Create sample data
sudo mkdir -p /mnt/database/mysql-data
for i in {1..100}; do
    sudo bash -c "echo 'Database record $i - $(date)' >> /mnt/database/mysql-data/table_$((i%10)).dat"
done

# Verify data
ls -la /mnt/database/mysql-data/
du -sh /mnt/database/mysql-data/
```

### Step 2: Create and Manage Snapshots
```bash
# Create initial snapshot
VOLUME_ID=$(aws ec2 describe-volumes --filters "Name=tag:Name,Values=lab-gp3" --query 'Volumes[0].VolumeId' --output text)
echo "Volume ID: $VOLUME_ID"

aws ec2 create-snapshot --volume-id $VOLUME_ID --description "Database baseline snapshot" --tag-specifications 'ResourceType=snapshot,Tags=[{Key=Name,Value=baseline-snapshot},{Key=Environment,Value=lab}]'

# Get snapshot ID
SNAPSHOT_ID=$(aws ec2 describe-snapshots --owner-ids self --filters "Name=tag:Name,Values=baseline-snapshot" --query 'Snapshots[0].SnapshotId' --output text)
echo "Snapshot ID: $SNAPSHOT_ID"

# Monitor snapshot progress
aws ec2 describe-snapshots --snapshot-ids $SNAPSHOT_ID --query 'Snapshots[0].[SnapshotId,State,Progress]' --output table
```

### Step 3: Simulate Data Changes and Incremental Snapshots
```bash
# Add more data (simulate database changes)
for i in {101..200}; do
    sudo bash -c "echo 'Database record $i - $(date)' >> /mnt/database/mysql-data/table_$((i%10)).dat"
done

# Modify existing files
sudo bash -c "echo 'UPDATED - $(date)' >> /mnt/database/mysql-data/table_1.dat"

# Create incremental snapshot
aws ec2 create-snapshot --volume-id $VOLUME_ID --description "Database incremental snapshot" --tag-specifications 'ResourceType=snapshot,Tags=[{Key=Name,Value=incremental-snapshot},{Key=Environment,Value=lab}]'
```

### Step 4: Snapshot Restore Testing
```bash
# Create new volume from snapshot
RESTORE_VOLUME_ID=$(aws ec2 create-volume --snapshot-id $SNAPSHOT_ID --availability-zone us-east-1a --volume-type gp3 --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=restored-volume},{Key=Environment,Value=lab}]' --query 'VolumeId' --output text)

echo "Restored Volume ID: $RESTORE_VOLUME_ID"

# Wait for volume to be available
aws ec2 wait volume-available --volume-ids $RESTORE_VOLUME_ID

# Attach restored volume
aws ec2 attach-volume --volume-id $RESTORE_VOLUME_ID --instance-id $(curl -s http://169.254.169.254/latest/meta-data/instance-id) --device /dev/sdj

# Mount and verify data
sudo mkdir -p /mnt/restored
sleep 10  # Wait for attachment
sudo mount /dev/xvdj /mnt/restored

# Compare data
echo "Original data count:"
ls /mnt/database/mysql-data/ | wc -l
echo "Restored data count:"
ls /mnt/restored/mysql-data/ | wc -l

# Verify specific files
diff /mnt/database/mysql-data/table_1.dat /mnt/restored/mysql-data/table_1.dat || echo "Files differ (expected if incremental changes were made)"
```

---

## ⚙️ Lab 3.2: EBS Volume Encryption

### Step 1: Create Encrypted Volume
```bash
# Create encrypted volume
ENCRYPTED_VOLUME_ID=$(aws ec2 create-volume --size 10 --volume-type gp3 --availability-zone us-east-1a --encrypted --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=encrypted-volume},{Key=Environment,Value=lab}]' --query 'VolumeId' --output text)

echo "Encrypted Volume ID: $ENCRYPTED_VOLUME_ID"

# Wait for volume creation
aws ec2 wait volume-available --volume-ids $ENCRYPTED_VOLUME_ID

# Attach encrypted volume
aws ec2 attach-volume --volume-id $ENCRYPTED_VOLUME_ID --instance-id $(curl -s http://169.254.169.254/latest/meta-data/instance-id) --device /dev/sdk

# Format and mount
sleep 10
sudo mkfs.ext4 /dev/xvdk
sudo mkdir -p /mnt/encrypted
sudo mount /dev/xvdk /mnt/encrypted
```

### Step 2: Test Encryption Features
```bash
# Create sensitive data
sudo bash -c "echo 'CONFIDENTIAL: Customer PII Data' > /mnt/encrypted/sensitive.txt"
sudo bash -c "echo 'SSN: 123-45-6789' >> /mnt/encrypted/sensitive.txt"
sudo bash -c "echo 'Credit Card: 4111-1111-1111-1111' >> /mnt/encrypted/sensitive.txt"

# Verify encryption details
aws ec2 describe-volumes --volume-ids $ENCRYPTED_VOLUME_ID --query 'Volumes[0].[Encrypted,KmsKeyId]' --output table

# Create snapshot of encrypted volume (will be encrypted)
ENCRYPTED_SNAPSHOT_ID=$(aws ec2 create-snapshot --volume-id $ENCRYPTED_VOLUME_ID --description "Encrypted volume snapshot" --tag-specifications 'ResourceType=snapshot,Tags=[{Key=Name,Value=encrypted-snapshot}]' --query 'SnapshotId' --output text)

echo "Encrypted Snapshot ID: $ENCRYPTED_SNAPSHOT_ID"

# Verify snapshot encryption
aws ec2 describe-snapshots --snapshot-ids $ENCRYPTED_SNAPSHOT_ID --query 'Snapshots[0].[Encrypted,KmsKeyId]' --output table
```

---

## ⚙️ Lab 3.3: EBS Volume Performance Optimization

### Step 1: Volume Modification (Size and Performance)
```bash
# Current volume details
aws ec2 describe-volumes --volume-ids $VOLUME_ID --query 'Volumes[0].[Size,VolumeType,Iops,Throughput]' --output table

# Modify volume - increase size and IOPS
aws ec2 modify-volume --volume-id $VOLUME_ID --size 30 --iops 4000 --throughput 250

# Monitor modification progress
aws ec2 describe-volumes-modifications --volume-ids $VOLUME_ID --query 'VolumesModifications[0].[ModificationState,Progress]' --output table

# Wait for modification to complete
aws ec2 wait volume-in-use --volume-ids $VOLUME_ID

# Extend file system to use new space
sudo resize2fs /dev/xvdf

# Verify new size
df -h /mnt/database
```

### Step 2: Performance Baseline After Optimization
```bash
# Install additional performance tools
sudo yum install -y iotop htop

# Test performance with new configuration
echo "Testing performance after optimization..."
sudo fio --name=optimized-test --filename=/mnt/database/performance-test --size=2G --ioengine=libaio --bs=4k --rw=randrw --iodepth=16 --direct=1 --runtime=60 --group_reporting

# Test sequential performance
sudo fio --name=seq-test --filename=/mnt/database/seq-test --size=5G --ioengine=libaio --bs=1M --rw=rw --direct=1 --runtime=60 --group_reporting
```

### Step 3: Multi-Attach Volumes (for compatible instance types)
```bash
# Note: Multi-Attach only works with io1 and io2 volumes on supported instances
# Create io2 volume with Multi-Attach enabled
MULTI_ATTACH_VOLUME_ID=$(aws ec2 create-volume --size 20 --volume-type io2 --iops 1000 --multi-attach-enabled --availability-zone us-east-1a --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=multi-attach-volume}]' --query 'VolumeId' --output text)

echo "Multi-Attach Volume ID: $MULTI_ATTACH_VOLUME_ID"

# Attach to current instance
aws ec2 attach-volume --volume-id $MULTI_ATTACH_VOLUME_ID --instance-id $(curl -s http://169.254.169.254/latest/meta-data/instance-id) --device /dev/sdl

# Note: In production, you would attach this to multiple instances
# and use cluster-aware file systems like GFS2 or OCFS2
```

---

## ⚙️ Lab 3.4: EBS Monitoring and Alerting

### Step 1: CloudWatch Metrics Analysis
```bash
# Get volume metrics from CloudWatch
aws cloudwatch get-metric-statistics --namespace AWS/EBS --metric-name VolumeReadOps --dimensions Name=VolumeId,Value=$VOLUME_ID --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%SZ) --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ) --period 300 --statistics Sum,Average --output table

# Get IOPS utilization
aws cloudwatch get-metric-statistics --namespace AWS/EBS --metric-name VolumeThroughputPercentage --dimensions Name=VolumeId,Value=$VOLUME_ID --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%SZ) --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ) --period 300 --statistics Average,Maximum --output table
```

### Step 2: Create CloudWatch Alarms
```bash
# Create alarm for high IOPS utilization
aws cloudwatch put-metric-alarm \
  --alarm-name "EBS-High-IOPS-Utilization" \
  --alarm-description "Alert when EBS IOPS utilization is high" \
  --metric-name VolumeThroughputPercentage \
  --namespace AWS/EBS \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --dimensions Name=VolumeId,Value=$VOLUME_ID

# Create alarm for queue depth
aws cloudwatch put-metric-alarm \
  --alarm-name "EBS-High-Queue-Depth" \
  --alarm-description "Alert when EBS queue depth is high" \
  --metric-name VolumeQueueLength \
  --namespace AWS/EBS \
  --statistic Average \
  --period 300 \
  --threshold 10 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --dimensions Name=VolumeId,Value=$VOLUME_ID

# List created alarms
aws cloudwatch describe-alarms --alarm-names "EBS-High-IOPS-Utilization" "EBS-High-Queue-Depth" --query 'MetricAlarms[*].[AlarmName,StateValue,Threshold]' --output table
```

---

## ⚙️ Lab 3.5: EBS Lifecycle Management

### Step 1: Automated Snapshot Lifecycle Policy
```bash
# Create IAM role for DLM (Data Lifecycle Manager)
cat > dlm-trust-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "dlm.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

# Create DLM service role
aws iam create-role --role-name DLMServiceRole --assume-role-policy-document file://dlm-trust-policy.json

# Attach policy
aws iam attach-role-policy --role-name DLMServiceRole --policy-arn arn:aws:iam::aws:policy/service-role/AWSDataLifecycleManagerServiceRole

# Create lifecycle policy
cat > lifecycle-policy.json << 'EOF'
{
  "ResourceTypes": ["VOLUME"],
  "TargetTags": [
    {
      "Key": "Environment",
      "Value": "lab"
    }
  ],
  "Schedules": [
    {
      "Name": "Daily snapshots",
      "CreateRule": {
        "Interval": 24,
        "IntervalUnit": "HOURS",
        "Times": ["03:00"]
      },
      "RetainRule": {
        "Count": 7
      },
      "TagsToAdd": [
        {
          "Key": "CreatedBy",
          "Value": "DLM"
        }
      ],
      "CopyTags": true
    }
  ]
}
EOF

# Get account ID
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

# Create lifecycle policy
aws dlm create-lifecycle-policy \
  --description "Daily snapshots for lab volumes" \
  --state ENABLED \
  --execution-role-arn "arn:aws:iam::${ACCOUNT_ID}:role/DLMServiceRole" \
  --policy-details file://lifecycle-policy.json

# List lifecycle policies
aws dlm get-lifecycle-policies --query 'Policies[*].[PolicyId,Description,State]' --output table
```

---

## 🔍 Verification Checklist

- [ ] Created and restored from EBS snapshots
- [ ] Tested incremental snapshot functionality
- [ ] Configured volume encryption and verified security
- [ ] Optimized volume performance (size, IOPS, throughput)
- [ ] Set up CloudWatch monitoring and alarms
- [ ] Implemented automated snapshot lifecycle management
- [ ] Documented performance improvements after optimization

---

## 📊 Results Documentation

### Snapshot Performance Impact
| Operation | Time (seconds) | Data Size | Notes |
|-----------|----------------|-----------|-------|
| Initial Snapshot | _____ | _____ | First snapshot of volume |
| Incremental Snapshot | _____ | _____ | After adding 50% more data |
| Restore from Snapshot | _____ | _____ | Time to create volume |

### Performance Optimization Results
| Metric | Before Optimization | After Optimization | Improvement |
|--------|-------------------|-------------------|-------------|
| 4K Random IOPS | _____ | _____ | _____% |
| Sequential Throughput | _____ MB/s | _____ MB/s | _____% |
| Volume Size | _____ GB | _____ GB | _____ GB added |

---

## 💡 Key Learning Outcomes

### EBS Best Practices Learned:
1. **Snapshots are incremental** - only changed blocks are stored
2. **Encryption is transparent** - no performance impact for encryption
3. **Performance can be modified** without downtime
4. **Automated lifecycle management** reduces operational overhead
5. **CloudWatch integration** provides comprehensive monitoring

### Production Recommendations:
- Use **encrypted volumes** for sensitive data
- Implement **automated snapshot policies** for backup
- Monitor **performance metrics** and set appropriate alarms
- Size volumes appropriately with **burst vs baseline** performance consideration
- Use **appropriate volume types** for workload characteristics

---

**Next Lab:** Amazon EFS Advanced Configuration and Performance Tuning