# Fmbravoglobal CloudFormation Security Modules

**Author:** Oluwafemi Alabi Okunlola | Cloud Security Engineer | DevSecOps Engineer  
**GitHub:** github.com/Fmbravoglobal  
**Stack:** CloudFormation · AWS · GitHub Actions · cfn-lint · cfn-nag · Checkov

---

## Overview

Production-grade CloudFormation modules implementing automated, policy-driven
Zero Trust cloud security controls across AWS. These modules are the CloudFormation
counterpart to the Terraform-based security framework in the Fmbravoglobal portfolio,
demonstrating IaC security enforcement across both major provisioning tools.

---

## Module Portfolio

| Module | Template | Security Focus | Compliance |
|--------|----------|---------------|------------|
| Secure S3 + KMS | `modules/secure-s3-kms-module.yaml` | Encryption, access blocking, versioning, logging, replication | CIS 2.1.x, NIST SC-28, SOC2, PCI |
| IAM Least-Privilege | `modules/iam-least-privilege-module.yaml` | OIDC federation, permission boundaries, role segmentation | CIS 1.x, NIST AC-3/AC-6, IA-5 |
| Secrets Rotation | `modules/secrets-rotation-module.yaml` | KMS CMK, zero-downtime rotation, DLQ, X-Ray | NIST IA-5, PCI Req 8, CIS |
| Observability & Threat Detection | `modules/observability-threat-detection-module.yaml` | GuardDuty, CloudTrail, Config, EventBridge, Lambda | NIST AU-2/SI-4, CIS 2.x/3.x, SOC2 |
| Zero Trust Policy Engine | `modules/zero-trust-policy-engine-module.yaml` | Risk-adaptive access, DynamoDB decision store, API | NIST 800-207, AC-2/AC-17 |
| **Master Stack** | `master-stack.yaml` | Nested stack orchestrator — deploys all modules | All frameworks |

---

## Architecture

```
master-stack.yaml
├── SecureS3Stack          → secure-s3-kms-module.yaml
├── IAMGovernanceStack     → iam-least-privilege-module.yaml
├── ObservabilityStack     → observability-threat-detection-module.yaml
├── SecretsRotationStack   → secrets-rotation-module.yaml  (conditional)
└── ZeroTrustEngineStack   → zero-trust-policy-engine-module.yaml  (conditional)
```

---

## Security Controls Enforced

### Secure S3 Module
- KMS CMK with automatic key rotation (CIS 2.8)
- All 4 public access block settings enabled (CIS 2.1.5)
- Versioning enabled (CIS 2.1.3)
- Server access logging to dedicated log bucket (CIS 2.1.4 / NIST AU-2)
- TLS-only bucket policy — denies all non-HTTPS (PCI-DSS Req 4)
- Deny unencrypted object uploads
- Optional cross-region replication with encrypted transfer
- SNS event notifications for all object operations

### IAM Least-Privilege Module
- GitHub Actions OIDC federation — eliminates long-lived static credentials (NIST IA-5)
- Permission boundaries on all roles — hard ceiling on privilege escalation (NIST AC-6)
- Least-privilege scoped policies — no admin permissions, no wildcard resources
- IAM Access Analyzer for external access detection
- CloudWatch alarm on root account usage (CIS 1.1)
- Security auditor read-only role for compliance verification

### Secrets Rotation Module
- KMS CMK encryption for all secrets (NIST SC-28)
- 30-day automated rotation schedule (NIST IA-5 / PCI Req 8.2.4)
- Zero-downtime 4-step rotation lifecycle
- SQS Dead Letter Queue for failed rotation capture
- X-Ray active tracing on Lambda
- Reserved concurrent executions (prevents abuse)
- CloudWatch alarm on rotation failures

### Observability & Threat Detection Module
- GuardDuty with S3, Kubernetes, and EBS malware protection enabled
- Multi-region CloudTrail with KMS encryption and log file validation (CIS 2.1/2.2/2.7)
- CloudTrail S3 data events for all buckets (PCI-DSS Req 10)
- EventBridge → Lambda automated response on GuardDuty findings
- DynamoDB encrypted findings store with PITR
- AWS Config continuous resource recording
- CloudWatch alarms: unauthorized API calls, console login without MFA, IAM policy changes (CIS 3.x)

### Zero Trust Policy Engine Module
- Risk-adaptive access decisions (ALLOW / INVESTIGATE / ESCALATE/DENY)
- Risk scoring: identity context + device posture + behavioral signals + geo-risk
- AUDIT vs ENFORCE mode — safe rollout without disruption
- Continuous trust verification every 15 minutes via EventBridge
- API Gateway with AWS_IAM authorization (no unauthenticated access)
- DynamoDB encrypted decision audit trail with TTL
- Permission boundary on all Lambda roles

---

## CI/CD Security Pipeline

```
Push / Pull Request
       │
       ▼
┌─────────────┐
│  cfn-lint   │  CloudFormation template linting and best-practice validation
└──────┬──────┘
       │
       ▼
┌─────────────┐    ┌─────────────┐
│  cfn-nag    │    │  checkov    │  Run in parallel after lint
│  security   │    │  policy-    │  Checkov uploads SARIF to GitHub
│  scan       │    │  as-code    │  Security tab
└──────┬──────┘    └──────┬──────┘
       │                  │
       └────────┬─────────┘
                ▼
       Validation Summary
       PASS → merge allowed
       FAIL → merge blocked
```

---

## Compliance Framework Mapping

| Framework | Version | Controls Validated |
|-----------|---------|-------------------|
| CIS AWS Benchmark | v1.5 | 1.x IAM, 2.1.x S3, 2.7 CloudTrail, 3.x Monitoring |
| NIST 800-53 | Rev5 | AC-2, AC-3, AC-6, AU-2, AU-9, IA-2, IA-5, IR-4, SC-8, SC-28, SI-4 |
| NIST SP 800-207 | Zero Trust | Continuous verification, least privilege, identity-centric |
| PCI-DSS | v3.2 | Req 3 (Encryption), Req 4 (TLS), Req 7 (Access Control), Req 8 (Credentials), Req 10 (Logging) |
| SOC 2 Type II | 2017 | CC6.1, CC6.3, CC6.6, CC6.7, CC7.2, CC7.3, A1.2 |

---

## Deployment — Individual Module

```bash
# Deploy Secure S3 module
aws cloudformation deploy \
  --template-file modules/secure-s3-kms-module.yaml \
  --stack-name fmbravoglobal-secure-s3-dev \
  --parameter-overrides \
      ProjectName=fmbravoglobal-secure-s3 \
      Environment=dev \
      AlertEmail=security@example.com \
  --capabilities CAPABILITY_NAMED_IAM \
  --region us-east-1

# Deploy IAM Governance module
aws cloudformation deploy \
  --template-file modules/iam-least-privilege-module.yaml \
  --stack-name fmbravoglobal-iam-governance-dev \
  --parameter-overrides \
      ProjectName=fmbravoglobal-iam \
      Environment=dev \
      GitHubOrg=Fmbravoglobal \
  --capabilities CAPABILITY_NAMED_IAM \
  --region us-east-1
```

## Deployment — Full Master Stack

```bash
# 1. Upload templates to S3
aws s3 sync modules/ s3://YOUR-TEMPLATES-BUCKET/cfn-templates/

# 2. Deploy master stack (all modules)
aws cloudformation deploy \
  --template-file master-stack.yaml \
  --stack-name fmbravoglobal-zt-security-dev \
  --parameter-overrides \
      ProjectName=fmbravoglobal-zt-security \
      Environment=dev \
      AlertEmail=security@example.com \
      TemplatesBucket=YOUR-TEMPLATES-BUCKET \
      TemplatesPrefix=cfn-templates \
      LambdaS3Bucket=YOUR-LAMBDA-BUCKET \
      GitHubOrg=Fmbravoglobal \
      PolicyEvaluationMode=AUDIT \
  --capabilities CAPABILITY_NAMED_IAM \
  --region us-east-1
```

---

## Repository Structure

```
cfn-security-modules/
├── master-stack.yaml                              # Nested stack orchestrator
├── modules/
│   ├── secure-s3-kms-module.yaml                 # S3 + KMS + logging + replication
│   ├── iam-least-privilege-module.yaml           # IAM + OIDC + permission boundaries
│   ├── secrets-rotation-module.yaml              # Secrets Manager + Lambda rotation
│   ├── observability-threat-detection-module.yaml # GuardDuty + CloudTrail + Config
│   └── zero-trust-policy-engine-module.yaml      # Risk-adaptive ZT policy evaluation
├── pipelines/
│   └── cfn-security-pipeline.yml                 # GitHub Actions: cfn-lint + cfn-nag + Checkov
└── README.md
```
