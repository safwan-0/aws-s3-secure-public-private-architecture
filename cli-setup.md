cli-setup.md — Secure Public & Private S3 Architecture (CLI)
# CLI Setup Guide — Secure Public & Private S3 Architecture

## Overview
This guide provides reproducible CLI steps to create:

- A **private S3 bucket** with encryption and blocked public access  
- A **public S3 bucket** with controlled read-only internet access  
- An **IAM role** assumed by EC2 to access the private bucket securely  

The design follows least-privilege and secure-by-default principles.

---

## Prerequisites

- AWS CLI installed
- AWS account configured:
```
aws configure
```
Environment Variables

Set your resource names once to avoid mistakes:

```
PRIVATE_BUCKET=your-private-bucket-unique-001
PUBLIC_BUCKET=your-public-bucket-unique-001
REGION=us-east-1
ROLE_NAME=S3PrivateAccessRole
PROFILE_NAME=S3PrivateAccessProfile
```
## Step 1 — Create Private Bucket (Secure by Default)

Create bucket:
```
aws s3api create-bucket \
  --bucket $PRIVATE_BUCKET \
  --region $REGION
```
 Block all public access:
 ```
aws s3api put-public-access-block \
  --bucket $PRIVATE_BUCKET \
  --public-access-block-configuration \
  BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true
```

Enable encryption at rest:
```
aws s3api put-bucket-encryption \
  --bucket $PRIVATE_BUCKET \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "AES256"
      }
    }]
  }'
```
## Step 2 — Create Public Bucket (Controlled Exposure)

Create bucket:
```

aws s3api create-bucket \
  --bucket $PUBLIC_BUCKET \
  --region $REGION
  ```

Disable public access block (policy will control exposure):
```
aws s3api put-public-access-block \
  --bucket $PUBLIC_BUCKET \
  --public-access-block-configuration \
  BlockPublicAcls=false,IgnorePublicAcls=false,BlockPublicPolicy=false,RestrictPublicBuckets=false
```

Apply public read-only bucket policy:
```
aws s3api put-bucket-policy \
  --bucket $PUBLIC_BUCKET \
  --policy file://policies/public-read-policy.json
```
Upload test file:
```
aws s3 cp test-image.png s3://$PUBLIC_BUCKET/
```

Public URL format:
```
https://$PUBLIC_BUCKET.s3.amazonaws.com/test-image.png
```
## Step 3 — Create IAM Role for Secure Access

Create role with EC2 trust relationship:
```
aws iam create-role \
  --role-name $ROLE_NAME \
  --assume-role-policy-document file://policies/trust-policy.json
```

Attach S3 access permissions to role:
```
aws iam put-role-policy \
  --role-name $ROLE_NAME \
  --policy-name PrivateS3AccessPolicy \
  --policy-document file://policies/private-s3-role-policy.json
```
## Step 4 — Create Instance Profile and Attach Role

Create instance profile:
```
aws iam create-instance-profile \
  --instance-profile-name $PROFILE_NAME
```

Add role to profile:
```
aws iam add-role-to-instance-profile \
  --instance-profile-name $PROFILE_NAME \
  --role-name $ROLE_NAME
```
Attach this instance profile when launching an EC2 instance.

## Step 5 — Verify Access from EC2

SSH into EC2 instance that has the role attached.

List private bucket contents:
```
aws s3 ls s3://$PRIVATE_BUCKET/
```

Upload file to private bucket:
```
aws s3 cp example.txt s3://$PRIVATE_BUCKET/
```

Attempt public access to private object (should fail):
```
https://$PRIVATE_BUCKET.s3.amazonaws.com/example.txt
```

Expected result: AccessDenied

## Security Design Summary

Private Bucket:

No public access allowed
Encrypted at rest
Accessible only via IAM role

Public Bucket:

Public read-only access
No public write permissions
Controlled exposure via bucket policy

IAM Role:

Temporary credentials
Least-privilege permissions
No long-term access keys

Cleanup (Optional)

Remove resources after testing to avoid charges.

Delete objects:
```
aws s3 rm s3://$PRIVATE_BUCKET --recursive
aws s3 rm s3://$PUBLIC_BUCKET --recursive
```

Delete buckets:
```
aws s3api delete-bucket --bucket $PRIVATE_BUCKET
aws s3api delete-bucket --bucket $PUBLIC_BUCKET
```
