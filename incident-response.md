# AWS Incident Response Services Deep Dive

---

## 1. **AWS Systems Manager Incident Manager**  
**Description**:  
End-to-end incident management service with automated runbooks, collaboration, and post-incident analysis.  

### Key Features:  
- **Automated Runbooks**: Predefined or custom workflows (e.g., isolate EC2 instances).  
- **Incident Timeline**: Track actions taken during response.  
- **Integration**: Triggered by GuardDuty, CloudWatch Alarms, or third-party tools.  

**Use Cases**:  
- Coordinate response to security incidents across teams.  
- Automate containment steps (e.g., block malicious IPs via WAF).  

**Key Permissions**:  
```plaintext
ssm-incidents:CreateResponsePlan  
ssm-incidents:StartIncident  
```

**Exam Tips**:  
- Use **Engagement Plans** to notify stakeholders via SMS/email.  
- Integrates with **OpsCenter** for operational item tracking.  

---

## 2. **AWS Systems Manager (SSM) Automation**  
**Description**:  
Safe, scalable remediation workflows for AWS resources.  

### Key Features:  
- **Prebuilt Playbooks**: Patch instances, rotate credentials, quarantine resources.  
- **Approval Workflows**: Manual validation before critical actions.  

**Use Cases**:  
- Auto-revoke compromised IAM access keys.  
- Isolate EC2 instances by modifying security groups.  

**Key Permissions**:  
```plaintext
ssm:StartAutomationExecution  
iam:PassRole  # For SSM service roles  
```

**Exam Tips**:  
- Combine with **AWS Config** to auto-remediate non-compliant resources.  

---

## 3. **Amazon Detective**  
**Description**:  
ML-powered service to visualize and investigate security incidents using behavior graphs.  

### Key Features:  
- **Timeline Analysis**: Map events leading to a compromise.  
- **Resource Relationships**: Identify linked users, roles, and resources.  
- **12-Month Data Retention**: Historical context for forensic analysis.  

**Use Cases**:  
- Trace lateral movement post-initial breach.  
- Analyze GuardDuty findings in context.  

**Key Permissions**:  
```plaintext
detective:CreateGraph  
detective:ListInvestigations  
```

**Exam Tips**:  
- **GuardDuty must be enabled** for Detective to ingest findings.  

---

## 4. **AWS Lambda**  
**Description**:  
Serverless compute for building automated incident response workflows.  

### Key Use Cases:  
- **Containment**:  
  ```python
  def isolate_instance(event):  
      ec2.modify_instance_attribute(InstanceId=event['instance_id'], Groups=['quarantine-sg'])
  ```
- **Forensic Data Collection**: Snapshot EBS volumes for analysis.  

**Key Permissions**:  
```plaintext
lambda:CreateFunction  
ec2:CreateSnapshot  
```

**Exam Tips**:  
- Trigger via **EventBridge** rules from GuardDuty/Security Hub.  

---

## 5. **AWS Step Functions**  
**Description**:  
Orchestrate multi-step incident response workflows across AWS services.  

### Key Features:  
- **State Machines**: Visual workflow design (e.g., isolate → analyze → remediate).  
- **Error Handling**: Retries and fallback logic for unreliable operations.  

**Use Case**:  
```json
"Steps": [  
  {"IsolateInstance": "Lambda"},  
  {"CaptureMemory": "EC2"},  
  {"NotifyTeam": "SNS"}  
]  
```

**Key Permissions**:  
```plaintext
states:StartExecution  
```

**Exam Tips**:  
- Use with **AWS SDK integrations** to call services directly.  

---

## 6. **AWS CloudTrail & Athena**  
**Description**:  
- **CloudTrail**: Logs API activity for audit trails.  
- **Athena**: SQL queries to analyze CloudTrail logs in S3.  

**Use Cases**:  
- Identify who deleted critical resources.  
- Detect API calls from unauthorized regions.  

**Example Query**:  
```sql
SELECT eventSource, eventName, userIdentity.arn  
FROM cloudtrail_logs  
WHERE eventTime > '2023-01-01'  
AND errorCode = 'AccessDenied'  
```

**Key Permissions**:  
```plaintext
cloudtrail:LookupEvents  
athena:StartQueryExecution  
```

**Exam Tips**:  
- Enable **CloudTrail Lake** for cheaper long-term log retention.  

---

## 7. **AWS Config & Remediation**  
**Description**:  
- **AWS Config**: Tracks resource changes for root cause analysis.  
- **Automated Remediation**: Fix non-compliant resources via SSM/Lambda.  

**Use Cases**:  
- Roll back security group rules post-incident.  
- Enforce encryption for new S3 buckets.  

**Key Permissions**:  
```plaintext
config:PutRemediationConfigurations  
```

**Exam Tips**:  
- Use **Conformance Packs** to enforce baseline configurations.  

---

## 8. **VPC Flow Logs & Traffic Mirroring**  
**Description**:  
- **Flow Logs**: Metadata for network traffic analysis.  
- **Traffic Mirroring**: Capture full packet data for deep inspection.  

**Use Cases**:  
- Detect C2 traffic patterns.  
- Replay malicious traffic for analysis.  

**Key Permissions**:  
```plaintext
ec2:CreateTrafficMirrorSession  
```

**Exam Tips**:  
- Use **Traffic Mirroring Targets** (ENI/NLB) with third-party IDS tools.  

---

## 9. **AWS Backup & Restore**  
**Description**:  
Centralized backup management for disaster recovery.  

**Key Features**:  
- **Cross-Region Backups**: Isolate backups from ransomware.  
- **Legal Hold**: Prevent deletion of forensic evidence.  

**Use Cases**:  
- Restore encrypted data from untainted backups.  
- Maintain chain of custody for audit trails.  

**Key Permissions**:  
```plaintext
backup:StartRestoreJob  
```

**Exam Tips**:  
- Use **Backup Vault Lock** for immutable backups.  

---

## 10. **AWS Security Hub**  
**Description**:  
Aggregates findings from GuardDuty, Inspector, Macie, and third-party tools.  

**Key Features**:  
- **Automated Response Actions**: Trigger Lambda/SSM via EventBridge.  
- **Compliance Dashboards**: Track post-incident remediation progress.  

**Key Permissions**:  
```plaintext
securityhub:BatchUpdateFindings  # Change finding statuses  
```

**Exam Tips**:  
- Use **Custom Actions** to automate containment steps.  

---

## Incident Response Workflow Example:  
1. **Detection**: GuardDuty flags `UnauthorizedAccess:IAMUser/ConsoleLogin`.  
2. **Containment**:  
   - Lambda revokes temporary credentials via `sts:AssumeRole`.  
   - SSM Automation isolates the affected EC2 instance.  
3. **Analysis**:  
   - Detective maps API calls to identify compromised roles.  
   - Athena queries CloudTrail for anomalous events.  
4. **Recovery**:  
   - Restore data from AWS Backup.  
   - Rotate KMS keys used by compromised roles.  
5. **Post-Mortem**:  
   - Security Hub generates compliance reports.  
   - Incident Manager documents lessons learned.  

---

## Critical IAM Permissions:  
- **Containment**: `ec2:ModifyInstanceAttribute`, `iam:UpdateAccessKey`.  
- **Forensics**: `s3:PutObject` (store logs), `ec2:CreateSnapshot`.  
- **Automation**: `states:StartExecution`, `lambda:InvokeFunction`.  

---

## Exam Scenarios:  
1. **Credential Compromise**:  
   - Use Lambda to call `iam:DeleteAccessKey` and `iam:UpdateLoginProfile`.  
   - Analyze API calls with Detective.  
2. **Ransomware Attack**:  
   - Restore data from immutable backups (Backup Vault Lock).  
   - Block malicious IPs via WAF/Network Firewall.  
3. **Data Exfiltration**:  
   - Use VPC Flow Logs + Athena to identify large outbound transfers.  
   - Revoke S3 permissions via SSM Automation.  


---
