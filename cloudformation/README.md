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

**Examples:**
- `iam:CreatePolicyVersion` - Create new versions of managed policies
- `iam:SetDefaultPolicyVersion` - Set existing policy versions as default
- `iam:AttachUserPolicy` - Attach policies directly to users
- `iam:PutUserPolicy` - Create inline policies on users

### 2. Transitive Privilege Escalation (`2-transitive-privilege-escalation/`)

Templates that require role assumption or user creation as an intermediate step to achieve privilege escalation.

**Characteristics:**
- Multi-step escalation process
- Requires role assumption or user manipulation
- Indirect privilege acquisition

**Examples:**
- `iam:AssumeRole` with overprivileged roles
- `iam:CreateUser` + `iam:AttachUserPolicy`
- `iam:AddUserToGroup` with privileged groups

### 3. Service PassRole Privilege Escalation (`3-service-passrole-escalation/`)

Templates that abuse AWS service integration with IAM roles through the `iam:PassRole` permission combined with service-specific actions.

**Characteristics:**
- Requires `iam:PassRole` permission
- Leverages AWS service functionality
- Service acts on behalf of the attacker

**Examples:**
- Lambda function creation with privileged execution role
- EC2 instance launch with privileged instance profile
- Glue job creation with administrative role
- CodeBuild project with elevated permissions

### 4. Iterative Policy Escalation (`4-iterative-policy-escalation/`)

Templates that require multiple rounds of policy updates or modifications to achieve full privilege escalation.

**Characteristics:**
- Multiple policy modification steps
- Gradual privilege acquisition
- Complex escalation chains

**Examples:**
- Multiple `iam:CreatePolicyVersion` iterations
- Combination of policy attachments and modifications
- Resource constraint bypasses through multiple updates

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
aws cloudformation create-stack \
  --stack-name privilege-escalation-test \
  --template-body file://template-name.yaml \
  --parameters ParameterKey=AssumeRoleArn,ParameterValue=arn:aws:iam::123456789012:user/testuser \
  --capabilities CAPABILITY_NAMED_IAM
```

### Cleanup

Always clean up test resources after testing:

```bash
aws cloudformation delete-stack --stack-name privilege-escalation-test
```
