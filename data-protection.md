

# AWS Data Protection Services Deep Dive

---

## 1. **AWS Key Management Service (KMS)**  
**Description**:  
Managed service for creating and controlling encryption keys used to protect data at rest and in transit.  

### Key Features:  
- **Customer Master Keys (CMKs)**: Symmetric (AES-256) and asymmetric (RSA/ECC) keys.  
- **Automatic Key Rotation**: Yearly for AWS-managed keys, customizable for customer-managed keys.  
- **Cross-Account Access**: Share keys via key policies and grants.  
- **Integration**: Works with 140+ AWS services (S3, EBS, RDS, etc.).  

**Use Cases**:  
- Encrypt S3 buckets using SSE-KMS.  
- Generate data keys for client-side encryption.  

**Key Permissions**:  
```plaintext
kms:Encrypt                   # Encrypt data with a CMK  
kms:CreateGrant               # Delegate key usage to other accounts/roles  
kms:ScheduleKeyDeletion       # Initiate key deletion (7-30 day waiting period)  
```

**Exam Tips**:  
- **Envelope Encryption**: Use `GenerateDataKey` to encrypt large data with a data key (encrypted by CMK).  
- **Key Policies** override IAM policies – use `kms:ViaService` to restrict usage to specific services.  

---

## 2. **AWS CloudHSM**  
**Description**:  
Dedicated Hardware Security Modules (HSMs) for FIPS 140-2 Level 3 compliance.  

### Key Features:  
- **Single-Tenant HSMs**: Isolated from AWS shared infrastructure.  
- **Custom Key Management**: Full control over key lifecycle (AWS can’t access keys).  
- **PKCS#11 / Java Cryptography Extensions (JCE)**: Integrate with on-prem apps.  

**Use Cases**:  
- Financial data encryption requiring strict compliance.  
- Legacy applications needing HSM-backed encryption.  

**Key Permissions**:  
```plaintext
cloudhsm:CreateCluster        # Deploy HSM cluster  
cloudhsm:InitializeCluster    # Set first admin credentials  
```

**Exam Tips**:  
- **KMS vs CloudHSM**: KMS is multi-tenant and managed; CloudHSM is single-tenant and customer-controlled.  
- CloudHSM is **not regional** – deploy clusters across Availability Zones for HA.  

---

## 3. **Amazon Macie**  
**Description**:  
Machine learning-powered service to discover and classify sensitive data (PII, PHI, credentials) in **S3**.  

### Key Features:  
- **Managed Data Identifiers**: Prebuilt detectors for 100+ sensitive data types.  
- **Bucket Security Reports**: Public access, encryption status, and sharing analysis.  
- **Findings Integration**: Export to Security Hub for centralized monitoring.  

**Use Cases**:  
- GDPR compliance (identify EU customer data).  
- Detect exposed AWS credentials in S3 logs.  

**Key Permissions**:  
```plaintext
macie2:CreateClassificationJob  # Scan S3 buckets for sensitive data  
macie2:GetSensitiveDataOccurrences  # List exact locations of PII  
```

**Exam Tips**:  
- Macie **cannot** scan EFS, EBS, or RDS – S3 only.  
- Use **Custom Data Identifiers** (regex) for organization-specific data patterns.  

---

## 4. **AWS Secrets Manager**  
**Description**:  
Securely store, rotate, and retrieve credentials (database passwords, API keys).  

### Key Features:  
- **Automatic Rotation**: Built-in support for RDS, Redshift, and custom integrations (Lambda).  
- **Resource Policies**: Restrict access to secrets (e.g., `secretsmanager:GetSecretValue`).  
- **Cross-Account Access**: Share secrets via key policies.  

**Use Cases**:  
- Rotate RDS master passwords every 90 days.  
- Replace hardcoded credentials in Lambda functions.  

**Key Permissions**:  
```plaintext
secretsmanager:RotateSecret   # Trigger credential rotation  
secretsmanager:GetResourcePolicy  # Check sharing permissions  
```

**Exam Tips**:  
- Secrets Manager charges **per secret/month** – use Parameter Store (free) for non-sensitive configs.  
- **Version Stages**: Track `AWSCURRENT` (active) and `AWSPREVIOUS` versions during rotation.  

---

## 5. **AWS Certificate Manager (ACM)**  
**Description**:  
Managed SSL/TLS certificates for AWS services (ALB, CloudFront, API Gateway).  

### Key Features:  
- **Auto-Renewal**: Free public certificates with automatic renewal.  
- **Private Certificates**: Issue internal PKI certificates for IoT/microservices.  
- **Export Restrictions**: Certificates cannot be downloaded (except for EC2/on-prem).  

**Use Cases**:  
- Enable HTTPS for CloudFront distributions.  
- Secure API Gateway endpoints with mutual TLS (mTLS).  

**Key Permissions**:  
```plaintext
acm:RequestCertificate       # Request a new certificate  
acm:DeleteCertificate        # Remove unused certificates  
```

**Exam Tips**:  
- ACM certificates are **regional** except for CloudFront (global).  
- Use **AWS Private CA** for custom certificate hierarchies.  

---

## 6. **AWS Backup**  
**Description**:  
Centralized backup management for EBS, RDS, DynamoDB, EFS, and Storage Gateway.  

### Key Features:  
- **Cross-Region/Account Backups**: Replicate backups for disaster recovery.  
- **Backup Vault Lock**: Immutable backups to prevent ransomware deletion.  
- **Lifecycle Policies**: Transition backups to cold storage (e.g., Glacier).  

**Use Cases**:  
- Meet RPO/RTO requirements for compliance.  
- Automate backup retention (e.g., delete after 7 years).  

**Key Permissions**:  
```plaintext
backup:CreateBackupPlan       # Define backup schedules  
backup:StartRestoreJob        # Initiate data recovery  
```

**Exam Tips**:  
- **Backup Vault Lock** requires a **cool-off period** (72 hours) before activation.  
- Use **AWS Backup Audit Manager** to track compliance with backup policies.  

---

## 7. **AWS Artifact**  
**Description**:  
Self-service portal for AWS compliance reports (SOC, PCI, ISO) and agreements.  

### Key Features:  
- **On-Demand Reports**: Download SOC 2/3, ISO 27001, etc.  
- **Artifact Agreements**: Accept terms for HIPAA/BAA or GDPR DPA.  

**Use Cases**:  
- Provide auditors with AWS compliance documentation.  
- Verify AWS infrastructure compliance for internal policies.  

**Access**:  
- No permissions needed – available to all AWS account root users.  

**Exam Tips**:  
- **Artifact ≠ Audit Manager**: Artifact provides reports; Audit Manager automates evidence collection.  

---

## 8. **Amazon S3 Encryption**  
**Methods**:  
- **SSE-S3**: S3-managed keys (AES-256).  
- **SSE-KMS**: KMS CMKs with audit trails.  
- **SSE-C**: Customer-provided keys.  
- **Client-Side Encryption**: Encrypt data before uploading.  

**Use Cases**:  
- Enforce bucket-level encryption using S3 Bucket Policies.  
- Use **S3 Bucket Keys** to reduce KMS costs for large objects.  

**Exam Tips**:  
- **Default Encryption** ≠ Enforcement – use bucket policies to block unencrypted uploads (`s3:x-amz-server-side-encryption`).  

---

## 9. **AWS Audit Manager**  
**Description**:  
Automates evidence collection for compliance frameworks (e.g., GDPR, HIPAA).  

**Key Features**:  
- Prebuilt frameworks (NIST, PCI DSS).  
- Continuous monitoring with AWS Config rules.  

**Use Cases**:  
- Map KMS key usage to encryption compliance controls.  
- Generate audit-ready reports for SOC 2.  

**Key Permissions**:  
```plaintext
auditmanager:CreateAssessment        # Define compliance scope  
auditmanager:GetEvidenceByEvidenceFolder  # Export evidence  
```

---

## Comparison Table: Data Protection Services

| Service                | Scope                      | Key Feature                     | Compliance Use Case          |  
|------------------------|----------------------------|---------------------------------|-------------------------------|  
| **KMS**                | Multi-service              | Centralized key management      | Encryption audits             |  
| **CloudHSM**           | Dedicated HSM              | FIPS 140-2 Level 3              | Financial/Government data     |  
| **Macie**              | S3                         | Sensitive data discovery        | GDPR/HIPAA                    |  
| **Secrets Manager**    | Credentials                | Automatic rotation              | PCI DSS (password security)   |  
| **Backup**             | Multi-service              | Immutable backups               | Disaster recovery compliance  |  
| **Audit Manager**      | Cross-service              | Evidence collection             | SOC 2 reporting               |  

---

## Exam Scenarios:  
1. **Data Breach Response**:  
   - Use Macie to identify exposed PII in S3.  
   - Rotate KMS CMKs and revoke grants.  
   - Enable S3 Block Public Access and bucket encryption.  

2. **Compliance Audit**:  
   - Use Audit Manager to collect KMS/S3 encryption evidence.  
   - Download AWS SOC 3 report via Artifact.  

3. **Disaster Recovery**:  
   - Configure AWS Backup with Vault Lock for immutable snapshots.  
   - Restore RDS from cross-region backups during DR testing.  

---