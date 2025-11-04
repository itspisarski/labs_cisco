# Lab 07 - AWS Data Protection and Disaster Recovery

**Objective:** Implement comprehensive data protection strategies using AWS Backup, cross-region replication, and disaster recovery automation.  
**Duration:** 90 minutes  
**Prerequisites:** AWS account, existing EBS volumes, S3 buckets, and RDS instances

---

## 🧩 Scenario
Your organization requires enterprise-grade data protection with automated backups, cross-region disaster recovery, and compliance with regulatory requirements for data retention and recovery time objectives (RTO/RPO).

---

## ⚙️ Lab 7.1: AWS Backup Service Configuration

### Step 1: Create IAM Roles for AWS Backup
```bash
# Create trust policy for AWS Backup service
cat > backup-trust-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "backup.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

# Create AWS Backup service role
aws iam create-role \
  --role-name AWSBackupDefaultServiceRole \
  --assume-role-policy-document file://backup-trust-policy.json

# Attach AWS managed policies
aws iam attach-role-policy \
  --role-name AWSBackupDefaultServiceRole \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSBackupServiceRolePolicyForBackup

aws iam attach-role-policy \
  --role-name AWSBackupDefaultServiceRole \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSBackupServiceRolePolicyForRestores

# Get the role ARN
BACKUP_ROLE_ARN=$(aws iam get-role --role-name AWSBackupDefaultServiceRole --query 'Role.Arn' --output text)
echo "AWS Backup Role ARN: $BACKUP_ROLE_ARN"
```

### Step 2: Create Backup Vault with Encryption
```bash
# Create KMS key for backup encryption
BACKUP_KMS_KEY_ID=$(aws kms create-key \
  --description "AWS Backup encryption key" \
  --query 'KeyMetadata.KeyId' --output text)

# Create key alias
aws kms create-alias \
  --alias-name alias/aws-backup-lab \
  --target-key-id $BACKUP_KMS_KEY_ID

echo "Backup KMS Key ID: $BACKUP_KMS_KEY_ID"

# Create backup vault
aws backup create-backup-vault \
  --backup-vault-name "lab-backup-vault" \
  --encryption-key-id $BACKUP_KMS_KEY_ID \
  --backup-vault-tags "Environment=Lab,Purpose=DataProtection"

# Create cross-region backup vault
aws backup create-backup-vault \
  --backup-vault-name "lab-backup-vault-dr" \
  --encryption-key-id $BACKUP_KMS_KEY_ID \
  --backup-vault-tags "Environment=Lab,Purpose=DisasterRecovery" \
  --region us-west-2

echo "Backup vaults created successfully"
```

### Step 3: Create Backup Plans with Different Retention Policies
```bash
# Create comprehensive backup plan
cat > backup-plan.json << 'EOF'
{
  "BackupPlanName": "ComprehensiveDataProtectionPlan",
  "Rules": [
    {
      "RuleName": "DailyBackups",
      "TargetBackupVaultName": "lab-backup-vault",
      "ScheduleExpression": "cron(0 5 ? * * *)",
      "StartWindowMinutes": 60,
      "CompletionWindowMinutes": 120,
      "Lifecycle": {
        "DeleteAfterDays": 30,
        "MoveToColdStorageAfterDays": 7
      },
      "RecoveryPointTags": {
        "BackupType": "Daily",
        "RetentionPolicy": "30Days"
      },
      "CopyActions": [
        {
          "DestinationBackupVaultArn": "arn:aws:backup:us-west-2:ACCOUNT-ID:backup-vault:lab-backup-vault-dr",
          "Lifecycle": {
            "DeleteAfterDays": 90,
            "MoveToColdStorageAfterDays": 30
          }
        }
      ]
    },
    {
      "RuleName": "WeeklyBackups",
      "TargetBackupVaultName": "lab-backup-vault",
      "ScheduleExpression": "cron(0 3 ? * SUN *)",
      "StartWindowMinutes": 60,
      "CompletionWindowMinutes": 180,
      "Lifecycle": {
        "DeleteAfterDays": 365,
        "MoveToColdStorageAfterDays": 30
      },
      "RecoveryPointTags": {
        "BackupType": "Weekly",
        "RetentionPolicy": "1Year"
      }
    },
    {
      "RuleName": "MonthlyBackups",
      "TargetBackupVaultName": "lab-backup-vault",
      "ScheduleExpression": "cron(0 2 1 * ? *)",
      "StartWindowMinutes": 120,
      "CompletionWindowMinutes": 240,
      "Lifecycle": {
        "DeleteAfterDays": 2555
      },
      "RecoveryPointTags": {
        "BackupType": "Monthly",
        "RetentionPolicy": "7Years"
      }
    }
  ],
  "AdvancedBackupSettings": [
    {
      "ResourceType": "EBS",
      "BackupOptions": {
        "WindowsVSS": "enabled"
      }
    }
  ]
}
EOF

# Replace ACCOUNT-ID placeholder
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
sed -i "s/ACCOUNT-ID/$ACCOUNT_ID/g" backup-plan.json

# Create backup plan
BACKUP_PLAN_ID=$(aws backup create-backup-plan \
  --backup-plan file://backup-plan.json \
  --query 'BackupPlanId' --output text)

echo "Backup Plan ID: $BACKUP_PLAN_ID"
```

### Step 4: Create Resource Assignments
```bash
# Create backup selection for EC2 instances and EBS volumes
cat > backup-selection.json << 'EOF'
{
  "SelectionName": "EC2AndEBSSelection",
  "IamRoleArn": "BACKUP_ROLE_ARN",
  "Resources": [],
  "ListOfTags": [
    {
      "ConditionType": "STRINGEQUALS",
      "ConditionKey": "Environment",
      "ConditionValue": "Production"
    }
  ],
  "Conditions": {
    "StringEquals": [
      {
        "ConditionKey": "aws:ResourceTag/Backup",
        "ConditionValue": "Required"
      }
    ]
  }
}
EOF

# Replace role ARN
sed -i "s|BACKUP_ROLE_ARN|$BACKUP_ROLE_ARN|g" backup-selection.json

# Create backup selection
aws backup create-backup-selection \
  --backup-plan-id $BACKUP_PLAN_ID \
  --backup-selection file://backup-selection.json

echo "Backup selection created for tagged resources"
```

---

## ⚙️ Lab 7.2: Test Resource Backup and Recovery

### Step 1: Create Test Resources with Backup Tags
```bash
# Create test EC2 instance with backup tags
TEST_INSTANCE_ID=$(aws ec2 run-instances \
  --image-id ami-0c02fb55956c7d316 \
  --instance-type t3.micro \
  --key-name your-key-pair \
  --tag-specifications 'ResourceType=instance,Tags=[
    {Key=Name,Value=BackupTestInstance},
    {Key=Environment,Value=Production},
    {Key=Backup,Value=Required}
  ]' \
  --query 'Instances[0].InstanceId' --output text)

# Create test EBS volume with backup tags
TEST_VOLUME_ID=$(aws ec2 create-volume \
  --size 20 \
  --volume-type gp3 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=volume,Tags=[
    {Key=Name,Value=BackupTestVolume},
    {Key=Environment,Value=Production},
    {Key=Backup,Value=Required}
  ]' \
  --query 'VolumeId' --output text)

# Create test RDS instance
RDS_INSTANCE_ID="backup-test-db"
aws rds create-db-instance \
  --db-instance-identifier $RDS_INSTANCE_ID \
  --db-instance-class db.t3.micro \
  --engine mysql \
  --master-username admin \
  --master-user-password MyTestPassword123! \
  --allocated-storage 20 \
  --vpc-security-group-ids sg-default \
  --tags Key=Environment,Value=Production Key=Backup,Value=Required

echo "Test resources created:"
echo "EC2 Instance: $TEST_INSTANCE_ID"
echo "EBS Volume: $TEST_VOLUME_ID"  
echo "RDS Instance: $RDS_INSTANCE_ID"
```

### Step 2: Trigger Manual Backup Jobs
```bash
# Start manual backup for EC2 instance
BACKUP_JOB_ID_EC2=$(aws backup start-backup-job \
  --backup-vault-name lab-backup-vault \
  --resource-arn arn:aws:ec2:us-east-1:$ACCOUNT_ID:instance/$TEST_INSTANCE_ID \
  --iam-role-arn $BACKUP_ROLE_ARN \
  --query 'BackupJobId' --output text)

# Start manual backup for EBS volume
BACKUP_JOB_ID_EBS=$(aws backup start-backup-job \
  --backup-vault-name lab-backup-vault \
  --resource-arn arn:aws:ec2:us-east-1:$ACCOUNT_ID:volume/$TEST_VOLUME_ID \
  --iam-role-arn $BACKUP_ROLE_ARN \
  --query 'BackupJobId' --output text)

# Start manual backup for RDS instance  
BACKUP_JOB_ID_RDS=$(aws backup start-backup-job \
  --backup-vault-name lab-backup-vault \
  --resource-arn arn:aws:rds:us-east-1:$ACCOUNT_ID:db:$RDS_INSTANCE_ID \
  --iam-role-arn $BACKUP_ROLE_ARN \
  --query 'BackupJobId' --output text)

echo "Manual backup jobs started:"
echo "EC2 Backup Job: $BACKUP_JOB_ID_EC2"
echo "EBS Backup Job: $BACKUP_JOB_ID_EBS"
echo "RDS Backup Job: $BACKUP_JOB_ID_RDS"

# Monitor backup job status
echo "Monitoring backup job progress..."
while true; do
  STATUS=$(aws backup describe-backup-job --backup-job-id $BACKUP_JOB_ID_EC2 --query 'State' --output text)
  echo "EC2 Backup Status: $STATUS"
  if [ "$STATUS" == "COMPLETED" ] || [ "$STATUS" == "FAILED" ]; then
    break
  fi
  sleep 30
done
```

### Step 3: Test Point-in-Time Recovery
```bash
# List available recovery points
echo "Available recovery points:"
aws backup list-recovery-points-by-backup-vault \
  --backup-vault-name lab-backup-vault \
  --query 'RecoveryPoints[*].[RecoveryPointArn,ResourceArn,CreationDate,Status]' \
  --output table

# Get recovery point ARN for EC2 restore
RECOVERY_POINT_ARN=$(aws backup list-recovery-points-by-backup-vault \
  --backup-vault-name lab-backup-vault \
  --by-resource-arn arn:aws:ec2:us-east-1:$ACCOUNT_ID:instance/$TEST_INSTANCE_ID \
  --query 'RecoveryPoints[0].RecoveryPointArn' --output text)

echo "Recovery Point ARN: $RECOVERY_POINT_ARN"

# Test restore operation (create new instance from backup)
cat > restore-metadata.json << 'EOF'
{
  "InstanceType": "t3.micro",
  "SubnetId": "subnet-xxxxxxxxx"
}
EOF

# Start restore job
RESTORE_JOB_ID=$(aws backup start-restore-job \
  --recovery-point-arn $RECOVERY_POINT_ARN \
  --metadata file://restore-metadata.json \
  --iam-role-arn $BACKUP_ROLE_ARN \
  --query 'RestoreJobId' --output text)

echo "Restore Job ID: $RESTORE_JOB_ID"

# Monitor restore progress
echo "Monitoring restore job progress..."
aws backup describe-restore-job --restore-job-id $RESTORE_JOB_ID
```

---

## ⚙️ Lab 7.3: Cross-Region Disaster Recovery

### Step 1: Set Up Cross-Region Replication
```bash
# Create S3 bucket for disaster recovery data
DR_BUCKET="disaster-recovery-lab-$(date +%s)-$(whoami)"
aws s3 mb s3://$DR_BUCKET --region us-east-1

# Create destination bucket in DR region
DR_BUCKET_WEST="disaster-recovery-west-$(date +%s)-$(whoami)"
aws s3 mb s3://$DR_BUCKET_WEST --region us-west-2

# Enable versioning on both buckets
aws s3api put-bucket-versioning --bucket $DR_BUCKET --versioning-configuration Status=Enabled
aws s3api put-bucket-versioning --bucket $DR_BUCKET_WEST --versioning-configuration Status=Enabled --region us-west-2

# Create replication role
cat > s3-replication-policy.json << EOF
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
        "arn:aws:s3:::$DR_BUCKET",
        "arn:aws:s3:::$DR_BUCKET/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:ReplicateObject",
        "s3:ReplicateDelete"
      ],
      "Resource": "arn:aws:s3:::$DR_BUCKET_WEST/*"
    }
  ]
}
EOF

# Create IAM role for S3 replication
cat > s3-replication-trust.json << 'EOF'
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

aws iam create-role --role-name S3ReplicationRole --assume-role-policy-document file://s3-replication-trust.json
aws iam put-role-policy --role-name S3ReplicationRole --policy-name S3ReplicationPolicy --policy-document file://s3-replication-policy.json

S3_REPLICATION_ROLE_ARN=$(aws iam get-role --role-name S3ReplicationRole --query 'Role.Arn' --output text)
```

### Step 2: Configure Multi-Region Backup Strategy
```bash
# Create disaster recovery backup plan for critical systems
cat > dr-backup-plan.json << 'EOF'
{
  "BackupPlanName": "DisasterRecoveryPlan",
  "Rules": [
    {
      "RuleName": "CriticalSystemsBackup",
      "TargetBackupVaultName": "lab-backup-vault",
      "ScheduleExpression": "cron(0 */6 * * ? *)",
      "StartWindowMinutes": 60,
      "CompletionWindowMinutes": 120,
      "Lifecycle": {
        "DeleteAfterDays": 90
      },
      "RecoveryPointTags": {
        "BackupType": "DisasterRecovery",
        "RTO": "4Hours",
        "RPO": "6Hours"
      },
      "CopyActions": [
        {
          "DestinationBackupVaultArn": "arn:aws:backup:us-west-2:ACCOUNT_ID:backup-vault:lab-backup-vault-dr",
          "Lifecycle": {
            "DeleteAfterDays": 180
          }
        }
      ]
    }
  ]
}
EOF

sed -i "s/ACCOUNT-ID/$ACCOUNT_ID/g" dr-backup-plan.json

# Create DR backup plan
DR_BACKUP_PLAN_ID=$(aws backup create-backup-plan \
  --backup-plan file://dr-backup-plan.json \
  --query 'BackupPlanId' --output text)

echo "DR Backup Plan ID: $DR_BACKUP_PLAN_ID"
```

### Step 3: Automated Failover Testing
```bash
# Create Lambda function for automated DR testing
cat > dr-test-function.py << 'EOF'
import json
import boto3
import datetime

def lambda_handler(event, context):
    backup_client = boto3.client('backup')
    ec2_client = boto3.client('ec2')
    
    # List recovery points in DR region
    response = backup_client.list_recovery_points_by_backup_vault(
        BackupVaultName='lab-backup-vault-dr'
    )
    
    latest_recovery_point = None
    for rp in response['RecoveryPoints']:
        if rp['Status'] == 'COMPLETED':
            latest_recovery_point = rp
            break
    
    if latest_recovery_point:
        # Test restore operation
        restore_metadata = {
            'InstanceType': 't3.micro',
            'SubnetId': 'subnet-xxxxxxxxx'  # Replace with actual subnet
        }
        
        restore_job = backup_client.start_restore_job(
            RecoveryPointArn=latest_recovery_point['RecoveryPointArn'],
            Metadata=restore_metadata,
            IamRoleArn='arn:aws:iam::ACCOUNT_ID:role/AWSBackupDefaultServiceRole'
        )
        
        return {
            'statusCode': 200,
            'body': json.dumps({
                'message': 'DR test initiated',
                'restoreJobId': restore_job['RestoreJobId'],
                'recoveryPoint': latest_recovery_point['RecoveryPointArn']
            })
        }
    
    return {
        'statusCode': 404,
        'body': json.dumps('No recovery points found for DR testing')
    }
EOF

# Package Lambda function
zip dr-test-function.zip dr-test-function.py

# Create Lambda execution role
cat > lambda-trust-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

aws iam create-role --role-name DRTestLambdaRole --assume-role-policy-document file://lambda-trust-policy.json
aws iam attach-role-policy --role-name DRTestLambdaRole --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

# Create custom policy for DR testing
cat > dr-lambda-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "backup:ListRecoveryPointsByBackupVault",
        "backup:StartRestoreJob",
        "backup:DescribeRestoreJob",
        "ec2:DescribeInstances",
        "ec2:DescribeSnapshots",
        "iam:PassRole"
      ],
      "Resource": "*"
    }
  ]
}
EOF

aws iam put-role-policy --role-name DRTestLambdaRole --policy-name DRTestPolicy --policy-document file://dr-lambda-policy.json

LAMBDA_ROLE_ARN=$(aws iam get-role --role-name DRTestLambdaRole --query 'Role.Arn' --output text)

# Create Lambda function
aws lambda create-function \
  --function-name dr-test-automation \
  --runtime python3.9 \
  --role $LAMBDA_ROLE_ARN \
  --handler dr-test-function.lambda_handler \
  --zip-file fileb://dr-test-function.zip \
  --description "Automated disaster recovery testing"

echo "DR test Lambda function created"
```

---

## ⚙️ Lab 7.4: Compliance and Audit Configuration

### Step 1: Enable AWS Config for Compliance Monitoring
```bash
# Create S3 bucket for Config
CONFIG_BUCKET="aws-config-lab-$(date +%s)-$(whoami)"
aws s3 mb s3://$CONFIG_BUCKET --region us-east-1

# Create Config service role
aws iam create-service-linked-role --aws-service-name config.amazonaws.com

# Create delivery channel
aws configservice put-delivery-channel \
  --delivery-channel name=default,s3BucketName=$CONFIG_BUCKET

# Create configuration recorder
aws configservice put-configuration-recorder \
  --configuration-recorder name=default,roleARN=arn:aws:iam::$ACCOUNT_ID:role/aws-service-role/config.amazonaws.com/AWSServiceRoleForConfig,recordingGroup='{allSupported=true,includeGlobalResourceTypes=true}'

# Start configuration recorder
aws configservice start-configuration-recorder --configuration-recorder-name default

echo "AWS Config enabled for compliance monitoring"
```

### Step 2: Set Up Compliance Rules
```bash
# Create backup compliance rule
aws configservice put-config-rule \
  --config-rule '{
    "ConfigRuleName": "ec2-backup-required",
    "Description": "Checks whether EC2 instances are covered by backup plan",
    "Source": {
      "Owner": "AWS",
      "SourceIdentifier": "EC2_BACKUP_REQUIRED"
    },
    "Scope": {
      "ComplianceResourceTypes": [
        "AWS::EC2::Instance"
      ]
    }
  }'

# Create backup vault compliance rule
aws configservice put-config-rule \
  --config-rule '{
    "ConfigRuleName": "backup-vault-encryption",
    "Description": "Checks whether backup vaults are encrypted",
    "Source": {
      "Owner": "AWS",
      "SourceIdentifier": "BACKUP_VAULT_ENCRYPTED"
    }
  }'

# Create backup retention compliance rule
aws configservice put-config-rule \
  --config-rule '{
    "ConfigRuleName": "backup-recovery-point-retention",
    "Description": "Checks backup recovery point retention periods",
    "Source": {
      "Owner": "AWS",
      "SourceIdentifier": "BACKUP_RECOVERY_POINT_MINIMUM_RETENTION_CHECK"
    },
    "InputParameters": "{\"requiredRetentionDays\": \"30\"}"
  }'

echo "Compliance rules configured"
```

### Step 3: Create Audit Reports
```bash
# Create CloudWatch dashboard for backup monitoring
aws cloudwatch put-dashboard \
  --dashboard-name "BackupComplianceDashboard" \
  --dashboard-body '{
    "widgets": [
      {
        "type": "metric",
        "x": 0,
        "y": 0,
        "width": 12,
        "height": 6,
        "properties": {
          "metrics": [
            ["AWS/Backup", "NumberOfBackupJobsCompleted"],
            [".", "NumberOfBackupJobsFailed"],
            [".", "NumberOfRestoreJobsCompleted"],
            [".", "NumberOfRestoreJobsFailed"]
          ],
          "period": 3600,
          "stat": "Sum",
          "region": "us-east-1",
          "title": "Backup Job Status"
        }
      }
    ]
  }'

# Generate compliance report
echo "=== Backup Compliance Report ===" > compliance-report.txt
echo "Generated on: $(date)" >> compliance-report.txt
echo "" >> compliance-report.txt

# Get backup job statistics
echo "Backup Job Statistics:" >> compliance-report.txt
aws backup describe-backup-vault --backup-vault-name lab-backup-vault \
  --query '[VaultName,NumberOfRecoveryPoints]' --output text >> compliance-report.txt

# Get compliance status
echo "" >> compliance-report.txt
echo "Config Compliance Status:" >> compliance-report.txt
aws configservice get-compliance-summary-by-config-rule \
  --query 'ComplianceSummary' --output table >> compliance-report.txt

cat compliance-report.txt
```

---

## ⚙️ Lab 7.5: Recovery Time and Point Objectives Testing

### Step 1: Measure Recovery Time Objectives (RTO)
```bash
# Script to test RTO
cat > test-rto.sh << 'EOF'
#!/bin/bash

echo "=== RTO Testing Started: $(date) ==="

# Start restore job
RECOVERY_POINT_ARN=$(aws backup list-recovery-points-by-backup-vault \
  --backup-vault-name lab-backup-vault \
  --query 'RecoveryPoints[0].RecoveryPointArn' --output text)

START_TIME=$(date +%s)
echo "Starting restore at: $(date)"

RESTORE_JOB_ID=$(aws backup start-restore-job \
  --recovery-point-arn $RECOVERY_POINT_ARN \
  --metadata '{"InstanceType":"t3.micro"}' \
  --iam-role-arn $BACKUP_ROLE_ARN \
  --query 'RestoreJobId' --output text)

# Monitor restore completion
while true; do
  STATUS=$(aws backup describe-restore-job --restore-job-id $RESTORE_JOB_ID --query 'Status' --output text)
  echo "Current status: $STATUS"
  
  if [ "$STATUS" == "COMPLETED" ]; then
    END_TIME=$(date +%s)
    RTO_SECONDS=$((END_TIME - START_TIME))
    echo "RTO achieved: $RTO_SECONDS seconds ($(($RTO_SECONDS / 60)) minutes)"
    break
  elif [ "$STATUS" == "FAILED" ]; then
    echo "Restore failed"
    break
  fi
  
  sleep 30
done

echo "=== RTO Testing Completed: $(date) ==="
EOF

chmod +x test-rto.sh
```

### Step 2: Measure Recovery Point Objectives (RPO)
```bash
# Script to test RPO
cat > test-rpo.sh << 'EOF'
#!/bin/bash

echo "=== RPO Testing Started: $(date) ==="

# Get latest recovery point
LATEST_BACKUP=$(aws backup list-recovery-points-by-backup-vault \
  --backup-vault-name lab-backup-vault \
  --query 'RecoveryPoints[0].CreationDate' --output text)

CURRENT_TIME=$(date -u +%Y-%m-%dT%H:%M:%S)
BACKUP_TIME=$(date -d "$LATEST_BACKUP" +%s)
CURRENT_TIME_SECONDS=$(date +%s)

RPO_SECONDS=$((CURRENT_TIME_SECONDS - BACKUP_TIME))
echo "Latest backup: $LATEST_BACKUP"
echo "Current time: $CURRENT_TIME"
echo "RPO: $RPO_SECONDS seconds ($(($RPO_SECONDS / 60)) minutes)"

if [ $RPO_SECONDS -le 21600 ]; then  # 6 hours
  echo "RPO Status: ✓ COMPLIANT (< 6 hours)"
else
  echo "RPO Status: ✗ NON-COMPLIANT (> 6 hours)"
fi

echo "=== RPO Testing Completed: $(date) ==="
EOF

chmod +x test-rpo.sh
./test-rpo.sh
```

---

## 🔍 Verification Checklist

- [ ] AWS Backup service configured with encryption
- [ ] Multi-tier backup plans created (daily, weekly, monthly)
- [ ] Cross-region backup replication working
- [ ] Manual backup and restore operations tested
- [ ] Disaster recovery automation implemented
- [ ] Compliance monitoring with AWS Config enabled
- [ ] RTO and RPO measurements completed
- [ ] Audit reports generated and reviewed

---

## 📊 Data Protection Metrics

### Backup Performance Results
| Resource Type | Backup Time | Restore Time | Backup Size | RTO Target | RPO Target |
|---------------|-------------|--------------|-------------|------------|------------|
| EC2 Instance | _____ min | _____ min | _____ GB | 4 hours | 6 hours |
| EBS Volume | _____ min | _____ min | _____ GB | 2 hours | 4 hours |
| RDS Database | _____ min | _____ min | _____ GB | 1 hour | 1 hour |
| S3 Data | N/A | Immediate | _____ GB | Immediate | Real-time |

### Cost Analysis (Monthly)
| Protection Level | Backup Storage | Cross-Region Copy | Total Cost | Use Case |
|------------------|----------------|-------------------|------------|----------|
| Basic (30 days) | $0.05/GB | $0.022/GB | $_____ | Development |
| Standard (1 year) | $0.05/GB + $0.01/GB cold | $0.022/GB | $_____ | Production |
| Compliance (7 years) | $0.05/GB + $0.004/GB cold | $0.022/GB | $_____ | Regulated |

---

## 💡 Key Learning Outcomes

### Data Protection Strategy:
1. **3-2-1 Rule**: 3 copies of data, 2 different media types, 1 offsite
2. **Tiered Retention**: Different retention periods for different backup types
3. **Cross-Region Replication**: Geographic disaster recovery protection
4. **Encryption**: Data protection at rest and in transit

### RTO/RPO Optimization:
1. **RTO Factors**: Backup size, restore method, network bandwidth
2. **RPO Factors**: Backup frequency, change rate, business requirements
3. **Cost vs. Performance**: Balance between protection level and cost
4. **Automation**: Reduce manual intervention and human error

### Compliance Requirements:
1. **Backup Verification**: Regular restore testing and validation
2. **Audit Trails**: Complete logging and monitoring of all operations
3. **Retention Policies**: Meet regulatory and business requirements
4. **Access Controls**: Proper IAM and encryption key management

---

**Next Lab:** Kubernetes Storage Integration and Container Persistent Volumes