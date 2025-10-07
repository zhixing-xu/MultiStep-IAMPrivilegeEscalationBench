# AWS IAM Privilege Escalation CloudFormation Templates

This directory contains CloudFormation templates that contains various AWS IAM privilege escalation risks. These templates are designed for security research, penetration testing, and defensive security training purposes.
**These templates create intentionally vulnerable IAM configurations.** Use only in isolated test environments with proper security controls. Never deploy these templates in production environments.

## Directory Structure

The templates are organized into four categories based on the privilege escalation methodology:

### 1. Direct Privilege Escalation (`1-direct-privilege-escalation/`)

Templates that demonstrate direct privilege escalation without requiring transitive access or role assumptions. These scenarios involve IAM permissions that directly allow an attacker to escalate their privileges.

**Characteristics:**
- Single-step escalation
- No role assumption required
- Direct IAM action abuse

| File | Description | Tested Actions |
|------|-------------|----------------|
| `PE1-1-DirectInlinePermissionMutation.yaml` | IAM principals that can add inline policies to themselves | `iam:PutRolePolicy`, `iam:PutUserPolicy` |
| `PE1-2-DirectManagedPermissionMutation.yaml` | IAM principals that can escalate via managed policy mutations | `iam:AttachRolePolicy`, `iam:AttachUserPolicy`, `iam:AttachGroupPolicy`, `iam:CreatePolicyVersion` |

### 2. Transitive Privilege Escalation (`2-transitive-privilege-escalation/`)

Templates that require role assumption or user creation as an intermediate step to achieve privilege escalation.

**Characteristics:**
- Multi-step escalation process
- Requires role assumption or user manipulation
- Indirect privilege acquisition

| File | Description | Escalation Path |
|------|-------------|-----------------|
| `PE2-1-TransitiveAdminRoleAssumption.yaml` | Role chain leading to admin privileges | r1 → AssumeRole(r2) → AssumeRole(r3-admin) |
| `PE2-2-TransitiveRolePermissionMutation.yaml` | Role chain enabling permission mutation on initial role | r1 → AssumeRole(r2) → AssumeRole(r3/r4) → PutRolePolicy/AttachRolePolicy(r1) |

### 3. Service PassRole Privilege Escalation (`3-service-passrole-escalation/`)

Templates that abuse AWS service integration with IAM roles through the `iam:PassRole` permission combined with service-specific actions.

**Characteristics:**
- Requires `iam:PassRole` permission
- Leverages AWS service functionality
- Service acts on behalf of the attacker

| File | Description | Escalation Path |
|------|-------------|-----------------|
| `PE3-1-EC2PassRoleEscalation.yaml` | EC2 instance launch with admin instance profile | r1 (PassRole + RunInstances) → Launch EC2 with r2 profile → Access instance metadata → Retrieve r2 (admin) credentials |

### 4. Iterative Policy Escalation (`4-iterative-policy-escalation/`)

Templates that require multiple rounds of policy updates or modifications to achieve full privilege escalation.

**Characteristics:**
- Multiple policy modification steps
- Gradual privilege acquisition
- Complex escalation chains

| File | Description | Escalation Path |
|------|-------------|-----------------|
| `PE4-1-IterativeTrustPolicyModification.yaml` | Trust policy modification to enable blocked role assumption | r1 → assume r2 → assume r4 → update r5 trust policy → r2 assume r5 → access S3 bucket |

## Usage Instructions

### Prerequisites

- AWS CLI configured with appropriate credentials
- CloudFormation deployment permissions
- Isolated AWS account or sandbox environment

### Deployment

1. Choose the appropriate category directory
2. Select a template based on your testing scenario
3. Deploy using AWS CLI:

```bash
aws cloudformation deploy \
  --template-file template-name.yaml \
  --stack-name privilege-escalation-test \
  --capabilities CAPABILITY_NAMED_IAM \
  --region us-east-1
```

### Cleanup

Always clean up test resources after testing:

```bash
aws cloudformation delete-stack --stack-name privilege-escalation-test
```
