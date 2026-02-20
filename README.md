
# Secure S3 Public & Private Architecture (CLI-Based)

## Project Overview

This project demonstrates a secure Amazon S3 architecture implementing:

- Private data storage with blocked public access
- Controlled public read-only bucket exposure
- IAM role-based access using temporary credentials
- Least-privilege permission design
- CLI-based reproducible infrastructure setup

The goal is to showcase practical cloud security principles used in production environments.

---

## Architecture Summary

The system separates public and private data using identity-based access control.

Public Access Path:
Internet Users → Public S3 Bucket (read-only)

Secure Access Path:
EC2 Instance → IAM Role → Private S3 Bucket

Security boundaries are enforced through IAM policies, bucket policies, and public access block settings.

---

## Security Principles Implemented

- Least privilege access control
- Secure-by-default configuration
- Identity-based authorization
- No long-term credentials
- Controlled public exposure
- Encryption at rest
- Layered access enforcement

---

## Repository Structure



aws-s3-secure-public-private-architecture/
│
├── README.md
├── cli-setup.md
├── verification-tests.md
└── policies/
├── trust-policy.json
├── private-s3-role-policy.json
└── public-read-policy.json


---

## Key Components

### Private S3 Bucket
- Blocks all public access
- Encrypted at rest
- Accessible only via IAM role

### Public S3 Bucket
- Public read-only access
- No public write permissions
- Exposure controlled via bucket policy

### IAM Role
- Assumed by EC2 instance
- Temporary credentials
- Scoped access to private bucket only

---

## Reproducible Setup

Follow the full CLI implementation guide:



cli-setup.md


This document includes:
- Bucket creation
- Policy application
- IAM role configuration
- Access verification

---

## Security Verification

All access control behaviors were validated through functional testing.

See:


verification-tests.md


Tests confirm:
- Private bucket is not publicly accessible
- Public bucket allows read-only internet access
- IAM role provides scoped access to private data
- Unauthorized actions are denied

---

## Learning Outcomes

This project demonstrates understanding of:

- AWS identity and access management
- S3 exposure risk management
- Policy-based authorization
- Temporary credential model
- Secure infrastructure design
- Cloud security verification practices

---

## License

This project is licensed under the MIT License.
