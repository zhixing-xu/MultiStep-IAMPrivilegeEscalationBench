# AWS IAM Privilege Escalation Benchmark

A comprehensive collection of CloudFormation templates demonstrating AWS IAM privilege escalation techniques for security research, penetration testing, and defensive security training.

**WARNING: These templates create intentionally vulnerable IAM configurations.** Use only in isolated test environments with proper security controls. Never deploy these templates in production environments.

## Overview

This benchmark contains **24 privilege escalation scenarios** organized into 4 categories:

| Category | Cases | Description |
|----------|-------|-------------|
| PE1 - Direct Escalation | 7 | Single-step direct IAM abuse |
| PE2 - Transitive Escalation | 4 | Multi-step role chaining |
| PE3 - Service PassRole | 8 | PassRole + service exploitation |
| PE4 - Iterative Policy | 5 | Multiple policy modification rounds |

## Directory Structure

```
cloudformation/
├── 1-direct-privilege-escalation/      # PE1-1 to PE1-7
├── 2-transitive-privilege-escalation/  # PE2-1 to PE2-4
├── 3-service-passrole-escalation/      # PE3-1 to PE3-8
└── 4-iterative-policy-escalation/      # PE4-1 to PE4-5
```

See [cloudformation/README.md](cloudformation/README.md) for detailed documentation of each escalation scenario.

## Quick Start

```bash
# Deploy a template
aws cloudformation deploy \
  --template-file cloudformation/<category>/<template>.yaml \
  --stack-name <stack-name> \
  --capabilities CAPABILITY_NAMED_IAM \
  --region us-east-1

# Cleanup
aws cloudformation delete-stack --stack-name <stack-name>
```
