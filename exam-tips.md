# AWS Security Specialty (SCS-C02) Exam Prep Tips

---

## 1. **Core Concepts to Master**
### Shared Responsibility Model
- **AWS Responsibility**: Physical infrastructure, hypervisor, managed services (e.g., RDS encryption).  
- **Customer Responsibility**: IAM policies, OS patching (EC2), data encryption (client-side).  
- **Key Exam Focus**: Know which security tasks fall on the customer for each service (e.g., S3 SSE-KMS vs. SSE-S3).

### Service Integration
- **Key Combos**:  
  - CloudTrail → Athena → QuickSight = Audit dashboards.  
  - GuardDuty → EventBridge → Lambda = Auto-remediation.  
  - Security Hub → Config → Macie = Unified compliance reporting.  

---

## 2. **Permissions & Policies**
- **Hierarchy**:  
  1. **SCPs** (AWS Organizations): Account-wide guardrails.  
  2. **IAM Policies**: User/role permissions (cannot override SCPs).  
  3. **Resource Policies** (e.g., S3 bucket policies): Directly attached to resources.  
- **Key Permissions**:  
  - `kms:Decrypt` (cross-account access).  
  - `iam:PassRole` (critical for service roles).  
  - `sts:AssumeRole` (cross-account federation).  

---

## 3. **Scenario-Based Thinking**
### Common Exam Patterns:
1. **"Most secure" vs. "Least privilege"**:  
   - Always favor **IAM roles over hardcoded credentials**.  
   - Use **permissions boundaries** for role delegation.  
2. **Compliance Requirements**:  
   - HIPAA/GDPR: Macie + KMS + Config.  
   - PCI DSS: WAF + GuardDuty + Encryption.  
3. **Cost vs. Security Trade-offs**:  
   - CloudTrail Lake (pay per query) vs. S3/Athena (long-term storage).  
   - Inspector (per-instance pricing) vs. third-party tools (fixed cost).  

---

## 4. **Key Services & Features**
### High-Yield Services:
| Service                | Must-Know Features                                  |  
|------------------------|-----------------------------------------------------|  
| **GuardDuty**          | Finding types (e.g., `CryptoCurrency:EC2`), multi-account setup |  
| **KMS**                | Key policies, grants, cross-account sharing         |  
| **IAM**                | Permission boundaries, MFA enforcement, AssumeRole  |  
| **Security Hub**       | ASFF format, custom actions, compliance standards    |  
| **Macie**              | Managed/custom data identifiers, S3 security reports |  

### Often Overlooked:
- **AWS Artifact**: Download SOC/PCI reports (no IAM permissions needed).  
- **VPC Flow Logs**: Capture `REJECT` events for NACL troubleshooting.  
- **Secrets Manager vs. Parameter Store**: Use Secrets Manager for rotation.  

---

## 5. **Troubleshooting & Analysis**
### Logging Flowchart:
1. **"Who did what?"**: Query **CloudTrail** with Athena.  
2. **"Why was access denied?"**: Check IAM policy `Deny` statements with **IAM Simulator**.  
3. **"Is data encrypted?"**: Use **AWS Config** + **KMS key policies**.  
4. **"Is traffic blocked?"**: Analyze **VPC Flow Logs** in CloudWatch Insights.  

---

## 6. **Practice Strategies**
### Lab Focus Areas:
1. **Auto-Remediation**:  
   - Build a Lambda function to revoke compromised IAM keys (triggered by GuardDuty).  
2. **Cross-Account Security**:  
   - Configure SCPs to disable root account actions in child accounts.  
3. **Encryption Deep Dive**:  
   - Rotate KMS keys + test S3 SSE-KMS bucket policies.  

### Mock Exam Tips:
- **Flag Long Questions**: Skip and return later.  
- **Eliminate Wrong Answers**: Look for absolutes like "all" or "never".  
- **Time Management**: Allocate ~2 mins per question (75 questions / 170 mins).  

---

## 7. **Last-Minute Cheat Sheet**
### Acronyms:
- **SCP**: Service Control Policy (Organizations).  
- **ASFF**: AWS Security Findings Format (Security Hub).  
- **CVE**: Common Vulnerabilities and Exposures (Inspector).  

### Key Numbers:
- **KMS Key Deletion**: 7-30 day waiting period.  
- **GuardDuty Findings**: Retained for 90 days.  
- **CloudTrail Logs**: 90 days in console (indefinite in S3).  

---

## 8. **Exam Day Checklist**
1. **Focus Areas**:  
   - Incident response workflows (Detective + Lambda).  
   - Encryption (KMS key policies, cross-account grants).  
   - Logging (CloudTrail vs. VPC Flow Logs vs. Config).  
2. **Avoid Pitfalls**:  
   - Confusing **Security Groups** (stateful) with **NACLs** (stateless).  
   - Overlooking **resource-based policies** (e.g., S3 bucket policies).  
3. **Mindset**:  
   - Think like a **Security Architect**: Prioritize prevention, detection, remediation.  
   - AWS’s **Well-Architected Framework** pillars (Security, Reliability, etc.) are implicit in many questions.  

---

## Final Tip:  
**"Security is Job Zero"**: AWS’s mantra means questions often prioritize **proactive controls** (e.g., encryption, least privilege) over reactive fixes.  