# Lab 04 - Amazon EFS Deep Dive and S3 Advanced Features

**Objective:** Master EFS performance modes and S3 advanced storage classes and lifecycle management.  
**Duration:** 90 minutes  
**Prerequisites:** AWS account, EFS file system from previous labs

---

## 🧩 Scenario
Your organization needs to optimize storage costs and performance for different workloads: shared file systems for development teams and object storage for data archiving and web applications.

---

## ⚙️ Lab 4.1: EFS Performance Modes and Throughput

### Step 1: Create EFS with Different Performance Modes
```bash
# Create General Purpose EFS
aws efs create-file-system \
  --creation-token lab-efs-gp-$(date +%s) \
  --performance-mode generalPurpose \
  --throughput-mode provisioned \
  --provisioned-throughput-in-mibps 100 \
  --tags Key=Name,Value=lab-efs-general-purpose

# Create Max I/O EFS for high concurrency
aws efs create-file-system \
  --creation-token lab-efs-maxio-$(date +%s) \
  --performance-mode maxIO \
  --throughput-mode provisioned \
  --provisioned-throughput-in-mibps 100 \
  --tags Key=Name,Value=lab-efs-max-io

# Get file system IDs
EFS_GP_ID=$(aws efs describe-file-systems --query 'FileSystems[?Tags[?Key==`Name` && Value==`lab-efs-general-purpose`]].FileSystemId' --output text)
EFS_MAXIO_ID=$(aws efs describe-file-systems --query 'FileSystems[?Tags[?Key==`Name` && Value==`lab-efs-max-io`]].FileSystemId' --output text)

echo "General Purpose EFS ID: $EFS_GP_ID"
echo "Max I/O EFS ID: $EFS_MAXIO_ID"
```

### Step 2: Create Mount Targets and Security Groups
```bash
# Get VPC and subnet information
VPC_ID=$(aws ec2 describe-vpcs --filters "Name=isDefault,Values=true" --query 'Vpcs[0].VpcId' --output text)
SUBNET_ID=$(aws ec2 describe-subnets --filters "Name=vpc-id,Values=$VPC_ID" --query 'Subnets[0].SubnetId' --output text)

# Create security group for EFS
EFS_SG_ID=$(aws ec2 create-security-group --group-name efs-lab-sg --description "Security group for EFS lab" --vpc-id $VPC_ID --query 'GroupId' --output text)

# Add NFS rule
aws ec2 authorize-security-group-ingress --group-id $EFS_SG_ID --protocol tcp --port 2049 --source-group $EFS_SG_ID

# Create mount targets
aws efs create-mount-target --file-system-id $EFS_GP_ID --subnet-id $SUBNET_ID --security-groups $EFS_SG_ID
aws efs create-mount-target --file-system-id $EFS_MAXIO_ID --subnet-id $SUBNET_ID --security-groups $EFS_SG_ID

# Wait for mount targets to be available
echo "Waiting for mount targets to be available..."
aws efs describe-mount-targets --file-system-id $EFS_GP_ID --query 'MountTargets[0].LifeCycleState'
```

### Step 3: Performance Testing Different EFS Modes
```bash
# SSH to EC2 instance and install EFS utils
ssh ec2-user@<instance-ip> -i your-key.pem

sudo yum install -y amazon-efs-utils

# Create mount points
sudo mkdir -p /mnt/efs-gp /mnt/efs-maxio

# Mount both EFS file systems
sudo mount -t efs $EFS_GP_ID:/ /mnt/efs-gp
sudo mount -t efs $EFS_MAXIO_ID:/ /mnt/efs-maxio

# Test single-client performance
echo "Testing General Purpose EFS single client..."
sudo fio --name=efs-gp-single --filename=/mnt/efs-gp/single-client-test --size=1G --ioengine=libaio --bs=4k --rw=randrw --direct=1 --runtime=60 --group_reporting

echo "Testing Max I/O EFS single client..."
sudo fio --name=efs-maxio-single --filename=/mnt/efs-maxio/single-client-test --size=1G --ioengine=libaio --bs=4k --rw=randrw --direct=1 --runtime=60 --group_reporting

# Test with higher concurrency (simulate multiple clients)
echo "Testing General Purpose EFS with high concurrency..."
sudo fio --name=efs-gp-multi --filename=/mnt/efs-gp/multi-client-test --size=500M --ioengine=libaio --bs=4k --rw=randrw --direct=1 --runtime=60 --numjobs=10 --group_reporting

echo "Testing Max I/O EFS with high concurrency..."
sudo fio --name=efs-maxio-multi --filename=/mnt/efs-maxio/multi-client-test --size=500M --ioengine=libaio --bs=4k --rw=randrw --direct=1 --runtime=60 --numjobs=10 --group_reporting
```

---

## ⚙️ Lab 4.2: EFS Access Points and File System Policies

### Step 1: Create EFS Access Points
```bash
# Create access point for development team
DEV_ACCESS_POINT_ID=$(aws efs create-access-point \
  --file-system-id $EFS_GP_ID \
  --posix-user Uid=1001,Gid=1001 \
  --root-directory Path="/dev-team",CreationInfo='{OwnerUid=1001,OwnerGid=1001,Permissions="755"}' \
  --tags Key=Name,Value=dev-team-access-point \
  --query 'AccessPointId' --output text)

# Create access point for production team
PROD_ACCESS_POINT_ID=$(aws efs create-access-point \
  --file-system-id $EFS_GP_ID \
  --posix-user Uid=1002,Gid=1002 \
  --root-directory Path="/prod-team",CreationInfo='{OwnerUid=1002,OwnerGid=1002,Permissions="750"}' \
  --tags Key=Name,Value=prod-team-access-point \
  --query 'AccessPointId' --output text)

echo "Dev Access Point ID: $DEV_ACCESS_POINT_ID"
echo "Prod Access Point ID: $PROD_ACCESS_POINT_ID"
```

### Step 2: Test Access Point Functionality
```bash
# Create mount points for access points
sudo mkdir -p /mnt/efs-dev /mnt/efs-prod

# Mount using access points
sudo mount -t efs $EFS_GP_ID.efs.us-east-1.amazonaws.com:/ /mnt/efs-dev -o tls,accesspoint=$DEV_ACCESS_POINT_ID
sudo mount -t efs $EFS_GP_ID.efs.us-east-1.amazonaws.com:/ /mnt/efs-prod -o tls,accesspoint=$PROD_ACCESS_POINT_ID

# Test access point isolation
sudo -u ec2-user touch /mnt/efs-dev/dev-file.txt
sudo -u ec2-user touch /mnt/efs-prod/prod-file.txt

# Verify file ownership and permissions
ls -la /mnt/efs-dev/
ls -la /mnt/efs-prod/

# Test cross-access (should be isolated)
ls -la /mnt/efs-gp/  # Original mount shows both directories
```

---

## ⚙️ Lab 4.3: S3 Storage Classes and Lifecycle Management

### Step 1: Create S3 Bucket with Different Storage Classes
```bash
# Create bucket for lifecycle testing
BUCKET_NAME="aws-storage-lab-$(date +%s)-$(whoami)"
aws s3 mb s3://$BUCKET_NAME --region us-east-1

echo "Created bucket: $BUCKET_NAME"

# Upload files to different storage classes
echo "Standard storage content" > standard-file.txt
echo "Infrequent access content" > ia-file.txt
echo "Archive content that will be moved to Glacier" > archive-file.txt

# Upload to Standard (default)
aws s3 cp standard-file.txt s3://$BUCKET_NAME/data/standard-file.txt

# Upload directly to Standard-IA
aws s3 cp ia-file.txt s3://$BUCKET_NAME/data/ia-file.txt --storage-class STANDARD_IA

# Upload directly to Glacier Instant Retrieval
aws s3 cp archive-file.txt s3://$BUCKET_NAME/data/glacier-ir-file.txt --storage-class GLACIER_IR

# Verify storage classes
aws s3api list-objects-v2 --bucket $BUCKET_NAME --query 'Contents[*].[Key,StorageClass]' --output table
```

### Step 2: Create Intelligent Tiering Configuration
```bash
# Enable Intelligent Tiering for the bucket
aws s3api put-bucket-intelligent-tiering-configuration \
  --bucket $BUCKET_NAME \
  --id "EntireBucket" \
  --intelligent-tiering-configuration '{
    "Id": "EntireBucket",
    "Status": "Enabled",
    "Filter": {"Prefix": ""},
    "Tierings": [
      {
        "Days": 90,
        "AccessTier": "ARCHIVE_ACCESS"
      },
      {
        "Days": 180,
        "AccessTier": "DEEP_ARCHIVE_ACCESS"
      }
    ]
  }'

# Upload file to Intelligent Tiering
echo "This file will be automatically tiered" > intelligent-tier-file.txt
aws s3 cp intelligent-tier-file.txt s3://$BUCKET_NAME/data/intelligent-tier-file.txt --storage-class INTELLIGENT_TIERING
```

### Step 3: Configure Lifecycle Policies
```bash
# Create lifecycle configuration
cat > lifecycle-config.json << 'EOF'
{
  "Rules": [
    {
      "ID": "DataLifecycleRule",
      "Status": "Enabled",
      "Filter": {"Prefix": "data/"},
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "STANDARD_IA"
        },
        {
          "Days": 90,
          "StorageClass": "GLACIER"
        },
        {
          "Days": 365,
          "StorageClass": "DEEP_ARCHIVE"
        }
      ],
      "Expiration": {
        "Days": 2555
      }
    },
    {
      "ID": "LogsLifecycleRule",
      "Status": "Enabled",
      "Filter": {"Prefix": "logs/"},
      "Transitions": [
        {
          "Days": 7,
          "StorageClass": "STANDARD_IA"
        },
        {
          "Days": 30,
          "StorageClass": "GLACIER"
        }
      ],
      "Expiration": {
        "Days": 90
      }
    }
  ]
}
EOF

# Apply lifecycle configuration
aws s3api put-bucket-lifecycle-configuration --bucket $BUCKET_NAME --lifecycle-configuration file://lifecycle-config.json

# Verify lifecycle configuration
aws s3api get-bucket-lifecycle-configuration --bucket $BUCKET_NAME --query 'Rules[*].[ID,Status,Filter.Prefix]' --output table
```

### Step 4: Test Multipart Upload for Large Objects
```bash
# Create large test file (100MB)
dd if=/dev/urandom of=large-file.bin bs=1M count=100

# Configure multipart threshold
aws configure set default.s3.multipart_threshold 50MB
aws configure set default.s3.multipart_chunksize 10MB

# Upload large file with multipart
time aws s3 cp large-file.bin s3://$BUCKET_NAME/large-objects/large-file.bin

# Check upload parts (if still in progress)
aws s3api list-multipart-uploads --bucket $BUCKET_NAME

# List objects and verify size
aws s3api list-objects-v2 --bucket $BUCKET_NAME --prefix "large-objects/" --query 'Contents[*].[Key,Size,StorageClass]' --output table
```

---

## ⚙️ Lab 4.4: S3 Cross-Region Replication and Versioning

### Step 1: Enable Versioning
```bash
# Enable versioning on source bucket
aws s3api put-bucket-versioning --bucket $BUCKET_NAME --versioning-configuration Status=Enabled

# Verify versioning is enabled
aws s3api get-bucket-versioning --bucket $BUCKET_NAME
```

### Step 2: Create Destination Bucket for Replication
```bash
# Create destination bucket in different region
DEST_BUCKET_NAME="aws-storage-lab-replica-$(date +%s)-$(whoami)"
aws s3 mb s3://$DEST_BUCKET_NAME --region us-west-2

# Enable versioning on destination bucket
aws s3api put-bucket-versioning --bucket $DEST_BUCKET_NAME --versioning-configuration Status=Enabled --region us-west-2
```

### Step 3: Set Up Cross-Region Replication
```bash
# Create IAM role for replication
cat > replication-trust-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "s3.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

# Create replication policy
cat > replication-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObjectVersionForReplication",
        "s3:GetObjectVersionAcl",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::SOURCE_BUCKET",
        "arn:aws:s3:::SOURCE_BUCKET/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:ReplicateObject",
        "s3:ReplicateDelete"
      ],
      "Resource": "arn:aws:s3:::DEST_BUCKET/*"
    }
  ]
}
EOF

# Replace bucket names in policy
sed -i "s/SOURCE_BUCKET/$BUCKET_NAME/g" replication-policy.json
sed -i "s/DEST_BUCKET/$DEST_BUCKET_NAME/g" replication-policy.json

# Create IAM role
aws iam create-role --role-name S3ReplicationRole --assume-role-policy-document file://replication-trust-policy.json
aws iam put-role-policy --role-name S3ReplicationRole --policy-name S3ReplicationPolicy --policy-document file://replication-policy.json

# Get account ID for ARN
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

# Configure replication
cat > replication-config.json << EOF
{
  "Role": "arn:aws:iam::${ACCOUNT_ID}:role/S3ReplicationRole",
  "Rules": [
    {
      "ID": "ReplicateEverything",
      "Status": "Enabled",
      "Priority": 1,
      "Filter": {"Prefix": ""},
      "Destination": {
        "Bucket": "arn:aws:s3:::${DEST_BUCKET_NAME}",
        "StorageClass": "STANDARD_IA"
      }
    }
  ]
}
EOF

# Apply replication configuration
aws s3api put-bucket-replication --bucket $BUCKET_NAME --replication-configuration file://replication-config.json

# Test replication by uploading new objects
echo "This file will be replicated" > replication-test.txt
aws s3 cp replication-test.txt s3://$BUCKET_NAME/replication-test.txt

# Check if replication occurred (may take a few minutes)
echo "Checking source bucket:"
aws s3 ls s3://$BUCKET_NAME/

echo "Checking destination bucket:"
aws s3 ls s3://$DEST_BUCKET_NAME/ --region us-west-2
```

---

## ⚙️ Lab 4.5: Performance Monitoring and Cost Analysis

### Step 1: S3 Request Metrics and CloudWatch
```bash
# Enable request metrics for the bucket
aws s3api put-bucket-metrics-configuration \
  --bucket $BUCKET_NAME \
  --id "EntireBucket" \
  --metrics-configuration '{
    "Id": "EntireBucket",
    "Filter": {"Prefix": ""}
  }'

# Generate some S3 operations for metrics
for i in {1..20}; do
  echo "Test file $i content" > test-file-$i.txt
  aws s3 cp test-file-$i.txt s3://$BUCKET_NAME/metrics-test/test-file-$i.txt
done

# Read operations
for i in {1..10}; do
  aws s3 cp s3://$BUCKET_NAME/metrics-test/test-file-$i.txt downloaded-file-$i.txt
done
```

### Step 2: Cost Analysis and Optimization
```bash
# Get storage analysis for cost optimization
aws s3api put-bucket-analytics-configuration \
  --bucket $BUCKET_NAME \
  --id "EntireBucketAnalytics" \
  --analytics-configuration '{
    "Id": "EntireBucketAnalytics",
    "Filter": {"Prefix": ""},
    "StorageClassAnalysis": {
      "DataExport": {
        "OutputSchemaVersion": "V_1",
        "Destination": {
          "S3BucketDestination": {
            "Format": "CSV",
            "Bucket": "arn:aws:s3:::'$BUCKET_NAME'",
            "Prefix": "analytics-reports/"
          }
        }
      }
    }
  }'

# List current storage usage by storage class
aws s3api list-objects-v2 --bucket $BUCKET_NAME --query 'Contents[*].[Key,Size,StorageClass]' --output table

# Calculate estimated monthly costs (simplified)
echo "Calculating estimated monthly storage costs..."
TOTAL_SIZE=$(aws s3api list-objects-v2 --bucket $BUCKET_NAME --query 'sum(Contents[].Size)' --output text)
TOTAL_SIZE_GB=$((TOTAL_SIZE / 1024 / 1024 / 1024))

echo "Total storage size: ${TOTAL_SIZE_GB} GB"
echo "Estimated costs per month:"
echo "  Standard: \$$(echo "scale=2; $TOTAL_SIZE_GB * 0.023" | bc)"
echo "  Standard-IA: \$$(echo "scale=2; $TOTAL_SIZE_GB * 0.0125" | bc)"
echo "  Glacier Instant Retrieval: \$$(echo "scale=2; $TOTAL_SIZE_GB * 0.004" | bc)"
```

---

## 🔍 Verification Checklist

- [ ] Tested EFS General Purpose vs Max I/O performance modes
- [ ] Configured EFS access points for team isolation
- [ ] Implemented S3 lifecycle policies for cost optimization
- [ ] Set up cross-region replication for disaster recovery
- [ ] Enabled S3 Intelligent Tiering for automatic cost optimization
- [ ] Analyzed storage costs across different classes
- [ ] Configured monitoring and metrics for both EFS and S3

---

## 📊 Performance Comparison Results

### EFS Performance Modes
| Mode | Single Client IOPS | Multi-Client IOPS | Latency | Use Case |
|------|-------------------|-------------------|---------|----------|
| General Purpose | _____ | _____ | Lower | Low latency apps |
| Max I/O | _____ | _____ | Higher | High concurrency |

### S3 Storage Class Costs (per GB/month)
| Storage Class | Cost | Retrieval Cost | Use Case |
|---------------|------|----------------|----------|
| Standard | $0.023 | None | Frequently accessed |
| Standard-IA | $0.0125 | $0.01/GB | Monthly access |
| Glacier IR | $0.004 | $0.03/GB | Quarterly access |
| Glacier Flexible | $0.0036 | Minutes-hours | Archive |
| Deep Archive | $0.00099 | 12 hours | Long-term archive |

---

## 💡 Key Learning Outcomes

### EFS Best Practices:
1. **General Purpose mode** for most applications (better latency)
2. **Max I/O mode** for highly concurrent workloads (>7,000 files/second)
3. **Provisioned Throughput** for predictable performance requirements
4. **Access Points** provide secure, application-specific access

### S3 Optimization Strategies:
1. **Lifecycle policies** automatically optimize costs
2. **Intelligent Tiering** removes manual decision-making
3. **Multipart uploads** improve performance for large objects
4. **Cross-region replication** provides disaster recovery
5. **Request metrics** help optimize application patterns

---

**Next Lab:** AWS Storage Gateway and Hybrid Cloud Integration