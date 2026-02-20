verification-tests.md
# Security Verification Tests

This document validates that the S3 architecture enforces correct access control, exposure boundaries, and least-privilege permissions.

All tests were performed after infrastructure creation using the AWS CLI and an EC2 instance with the IAM role attached.

---

## Test Environment

- Private bucket: stores restricted data
- Public bucket: provides controlled public read access
- IAM role: grants temporary access to private bucket
- EC2 instance: assumes IAM role

---

## Private Bucket Tests

### Test 1 — Upload via IAM Role

Command:
```
aws s3 cp example.txt s3://$PRIVATE_BUCKET/
```

Expected Result:
Access allowed

Security Meaning:
Authenticated role-based access works.

Status:
PASS

## Test 2 — List Bucket Contents

Command:
```
aws s3 ls s3://$PRIVATE_BUCKET/
```

Expected Result:
Objects listed successfully

Security Meaning:
Least-privilege permissions allow required operations only.

Status:
PASS

## Test 3 — Public Browser Access

Attempt:
```
https://$PRIVATE_BUCKET.s3.amazonaws.com/example.txt

```
Expected Result:
AccessDenied

Security Meaning:
Public access is correctly blocked.

Status:
PASS

## Test 4 — Public Write Attempt

Attempt:
Anonymous upload without credentials

Expected Result:
Denied

Security Meaning:
Bucket is not publicly writable.

Status:
PASS

### Public Bucket Tests
### Test 5 — Public Read Access

Attempt:
```
https://$PUBLIC_BUCKET.s3.amazonaws.com/test-image.png
```

Expected Result:
File loads successfully

Security Meaning:
Controlled public exposure is working.

Status:
PASS

## Test 6 — Public Write Attempt

Attempt:
Anonymous upload

Expected Result:
Denied

Security Meaning:
Public users cannot modify bucket contents.

Status:
PASS

## Security Validation Summary

Verified Controls:

Private data is not publicly accessible
Public bucket exposes read only content only
IAM role enforces identity-based access
No long-term credentials required
Least-privilege permissions confirmed

## Conclusion

Testing confirms the architecture enforces secure data isolation, controlled public exposure, and identity-based access control.

The system behaves according to secure-by-default cloud design principles.
