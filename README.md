# Cross-Account-EC2-to-S3-Access-Using-IAM-Role-STS
📌 Project Overview

This project demonstrates secure cross-account access where:

An EC2 instance in Account A

Uploads files to an S3 bucket in Account B

Using IAM Roles and STS (AssumeRole)

Without using static access keys

This setup follows AWS best practices and the principle of least privilege.

🏗️ Architecture
Account A (Source)

EC2 Instance

IAM Role: EC2AssumeS3Role

Account B (Target)

S3 Bucket: rehuu

IAM Role: CrossAccountS3AccessRole

Flow:
EC2 (Account A)
     ↓
STS AssumeRole
     ↓
CrossAccountS3AccessRole (Account B)
     ↓
Access S3 Bucket
🛠️ Step-by-Step Implementation
✅ Step 1: Create S3 Bucket (Account B)

Go to Amazon S3

Create bucket named: rehuu

Keep default security settings

✅ Step 2: Create IAM Role in Account B
Role Name:
CrossAccountS3AccessRole
Trusted Entity:

AWS Account

Enter Account A ID

🔹 Attach Inline Policy (S3 Permissions)
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::rehuu"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::rehuu/*"
    }
  ]
}
🔹 Trust Policy (Very Important)
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::ACCOUNT-A-ID:role/EC2AssumeS3Role"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
✅ Step 3: Create IAM Role in Account A
Role Name:
EC2AssumeS3Role
Trusted Entity:

EC2

Attach Inline Policy (Allow AssumeRole)
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sts:AssumeRole",
      "Resource": "arn:aws:iam::ACCOUNT-B-ID:role/CrossAccountS3AccessRole"
    }
  ]
}

Attach this role to EC2 instance.

✅ Step 4: Install AWS CLI on EC2
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip -y
unzip awscliv2.zip
sudo ./aws/install

Verify:

aws --version
✅ Step 5: Assume Cross-Account Role
aws sts assume-role \
--role-arn arn:aws:iam::ACCOUNT-B-ID:role/CrossAccountS3AccessRole \
--role-session-name testsession

Export credentials:

export AWS_ACCESS_KEY_ID=...
export AWS_SECRET_ACCESS_KEY=...
export AWS_SESSION_TOKEN=...

Verify identity:

aws sts get-caller-identity

Should show Account B ID.

✅ Step 6: Upload File to S3

Create file:

echo "Hello from EC2" > test.txt

Upload file:

aws s3 cp test.txt s3://rehuu/

Verify:

aws s3 ls s3://rehuu/
🔄 Automatic Sync Setup (Optional Enhancement)

To automatically reflect files created in EC2 into S3:

🥇 Method: Cron + aws s3 sync

Create folder:

mkdir /home/ubuntu/uploads

Create sync script:

nano sync-to-s3.sh

Add:

#!/bin/bash
aws s3 sync /home/ubuntu/uploads s3://rehuu/

Make executable:

chmod +x sync-to-s3.sh

Add cron job:

crontab -e

Add:

* * * * * /home/ubuntu/sync-to-s3.sh

Now any file created in /home/ubuntu/uploads will sync every minute.

❌ Errors Faced & Solutions
1️⃣ aws command not found

Cause:
AWS CLI not installed.

Solution:
Installed AWS CLI v2 manually.

2️⃣ AccessDenied during AssumeRole

Cause:
Typo in role ARN.

Fix:
Corrected ARN spelling.

3️⃣ AccessDenied during PutObject

Cause:
Still using EC2 role (not exported STS credentials).

Fix:
Exported temporary credentials from AssumeRole output.

4️⃣ ListObjectsV2 AccessDenied

Cause:
Missing s3:ListBucket permission.

Fix:
Added bucket-level permission.

🔐 Security Concepts Learned

IAM Role vs IAM User

Trust Policy vs Permission Policy

STS Temporary Credentials

Cross-Account Role Assumption

Principle of Least Privilege

Importance of correct ARN format

Object-level permissions using /*

🎯 Final Result

✔ EC2 in Account A successfully uploaded file to S3 in Account B
✔ No static access keys used
✔ Secure cross-account role-based architecture
✔ Production-ready setup
