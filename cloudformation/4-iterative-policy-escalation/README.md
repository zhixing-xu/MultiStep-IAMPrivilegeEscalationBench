# Iterative Policy Escalation Test Cases

This directory contains CloudFormation templates that demonstrate privilege escalation scenarios requiring multiple policy modification iterations to achieve the final goal.

## PE4-1: Iterative Trust Policy Modification

**File:** `PE4-1-IterativeTrustPolicyModification.yaml`

### Overview

This template demonstrates a complex privilege escalation scenario where an attacker must modify a trust policy to enable role assumption that was previously blocked.

### Architecture

```
R1 (initial) → R2 (intermediate) → R4 (trust modifier)
                ↓                      ↓
                ↓                   [modify trust policy]
                ↓                      ↓
                └─────→ R5 (data access) → S3 Bucket (sensitive data)
```

### Roles

- **R1 (pe4-1-r1-initial)**: Entry point role controlled by attacker
  - Can assume R2

- **R2 (pe4-1-r2-intermediate)**: Intermediate role with dual capabilities
  - Can assume R4 (trust policy modifier)
  - Has `sts:AssumeRole` permission for R5 (initially blocked by R5's trust policy)

- **R4 (pe4-1-r4-trustmodifier)**: Trust policy modification role
  - Has `iam:UpdateAssumeRolePolicy` permission for R5

- **R5 (pe4-1-r5-dataaccess)**: Target role with data access
  - Has `s3:GetObject` and `s3:ListBucket` permissions for the sensitive S3 bucket
  - Trust policy explicitly denies R2 from assuming it (configured post-deployment)

### Privilege Escalation Path

1. **Initial State**: R2 has `sts:AssumeRole` permission for R5, but R5's trust policy denies R2
2. **Step 1**: Attacker uses R1 to assume R2
3. **Step 2**: Using R2, assume R4
4. **Step 3**: Using R4, execute `iam:UpdateAssumeRolePolicy` to modify R5's trust policy, removing the deny statement for R2
5. **Step 4**: Using R2 again, now successfully assume R5
6. **Step 5**: Using R5, read sensitive data from the S3 bucket

### Deployment

```bash
aws cloudformation deploy \
  --template-file PE4-1-IterativeTrustPolicyModification.yaml \
  --stack-name pe4-1-iterative-trust-policy \
  --capabilities CAPABILITY_NAMED_IAM \
  --region us-east-1
```

### Post-Deployment Configuration

After stack creation, manually configure R5's trust policy to deny R2:

```bash
# Create trust policy with deny statement
cat > r5-trust-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::<ACCOUNT_ID>:root"
      },
      "Action": "sts:AssumeRole"
    },
    {
      "Effect": "Deny",
      "Principal": {
        "AWS": "arn:aws:iam::<ACCOUNT_ID>:role/pe4-1-r2-intermediate"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

# Apply the trust policy
aws iam update-assume-role-policy \
  --role-name pe4-1-r5-dataaccess \
  --policy-document file://r5-trust-policy.json
```

Replace `<ACCOUNT_ID>` with your AWS account ID.

### Testing the Escalation

1. **Verify R2 cannot directly assume R5:**
   ```bash
   # Assume R1
   aws sts assume-role --role-arn <R1_ARN> --role-session-name test

   # Assume R2
   aws sts assume-role --role-arn <R2_ARN> --role-session-name test

   # Try to assume R5 (should fail due to deny in trust policy)
   aws sts assume-role --role-arn <R5_ARN> --role-session-name test
   ```

2. **Execute the privilege escalation:**
   ```bash
   # Assume R4 via R2
   aws sts assume-role --role-arn <R4_ARN> --role-session-name test

   # Update R5's trust policy to remove the deny statement
   aws iam update-assume-role-policy \
     --role-name pe4-1-r5-dataaccess \
     --policy-document file://allow-r2-policy.json

   # Now R2 can assume R5
   aws sts assume-role --role-arn <R5_ARN> --role-session-name test

   # Read from S3 bucket
   aws s3 ls s3://pe4-1-sensitive-data-<ACCOUNT_ID>
   ```
