
# AWS Security Services Key Comparisons

---

## 1. **Threat Detection & Analysis**
| Service                | Primary Focus              | Data Source                  | Key Differentiator                     |
|------------------------|----------------------------|------------------------------|-----------------------------------------|
| **Amazon GuardDuty**   | Threat detection           | CloudTrail, VPC Flow Logs    | ML-driven anomaly detection (e.g., compromised credentials) |
| **Amazon Inspector**   | Vulnerability scanning     | EC2, ECR, Lambda             | CVE/CIS benchmarks for software/OS flaws |
| **Amazon Macie**       | Data classification        | S3 buckets                   | ML-based PII/PHI detection              |
| **AWS Detective**      | Incident investigation     | CloudTrail, VPC Flow Logs    | Visual behavior graphs for root cause analysis |

**Exam Tip**:  
- GuardDuty answers **"Is there malicious activity?"**, Inspector answers **"Are my resources vulnerable?"**, Macie answers **"Is sensitive data exposed?"**.

---

## 2. **Identity & Access**
| Service                | Scope                      | Key Function                 | Example Use Case                       |
|------------------------|----------------------------|------------------------------|-----------------------------------------|
| **IAM Policies**       | User/Role permissions      | Resource-level access rules  | Allow EC2:DescribeInstances for a dev role |
| **SCPs (Orgs)**        | Account-wide guardrails    | Deny/Allow account actions   | Block creation of IAM users in child accounts |
| **AWS SSO**            | Cross-account access       | SAML/OIDC federation         | Centralized access to multiple AWS accounts |

**Exam Tip**:  
- IAM policies grant permissions; SCPs restrict what IAM policies can do.

---

## 3. **Logging & Monitoring**
| Service                | Data Type                  | Retention       | Key Feature                          |
|------------------------|----------------------------|-----------------|---------------------------------------|
| **CloudTrail**         | API activity               | 90d (console)  | Track "who did what" for auditing     |
| **CloudWatch Logs**    | App/Service logs           | Configurable    | Real-time log analysis with Insights  |
| **VPC Flow Logs**      | Network metadata           | S3/CloudWatch   | Detect port scans/data exfiltration   |
| **AWS Config**         | Resource configurations    | Indefinite      | Track "how" resources changed over time |

**Exam Tip**:  
- Use **Athena** to query CloudTrail/VPC Flow Logs in S3 for historical analysis.

---

## 4. **Encryption & Keys**
| Service                | Key Control                | Compliance       | Use Case                              |
|------------------------|----------------------------|------------------|----------------------------------------|
| **AWS KMS**            | AWS/customer-managed CMKs  | FIPS 140-2 L2    | Encrypt S3/EBS with audit trails       |
| **CloudHSM**           | Customer-managed HSM       | FIPS 140-2 L3    | Financial data requiring dedicated HSM |
| **Secrets Manager**    | Credential storage         | -                | Rotate RDS passwords automatically    |

**Exam Tip**:  
- KMS is multi-tenant (AWS manages infrastructure); CloudHSM is single-tenant (customer-controlled).

---

## 5. **Network Security**
| Service                | Layer       | Scope            | Key Feature                          |
|------------------------|-------------|------------------|---------------------------------------|
| **Security Groups**    | L3-L4       | Instance-level   | Stateful firewall (allow rules only)  |
| **NACLs**              | L3-L4       | Subnet-level     | Stateless firewall (allow/deny rules) |
| **AWS WAF**            | L7          | Web apps         | Mitigate OWASP Top 10                 |
| **Network Firewall**   | L3-L7       | VPC-wide         | Suricata rules for deep inspection    |

**Exam Tip**:  
- WAF protects ALB/CloudFront; Network Firewall inspects all VPC traffic.

---

## 6. **Compliance & Governance**
| Service                | Focus                      | Key Feature                  | Integration               |
|------------------------|----------------------------|------------------------------|---------------------------|
| **AWS Config**         | Resource compliance        | Track configuration changes  | Auto-remediation with SSM |
| **Security Hub**       | Centralized findings       | CIS benchmarks               | Aggregates GuardDuty/Macie |
| **Audit Manager**      | Evidence collection        | Prebuilt frameworks (NIST)   | Maps to compliance controls |

**Exam Tip**:  
- **AWS Artifact** provides compliance reports; **Audit Manager** automates evidence gathering.

---

## 7. **Incident Response**
| Service                | Phase          | Key Feature                          | Example Action                    |
|------------------------|----------------|---------------------------------------|------------------------------------|
| **GuardDuty**          | Detection      | Threat alerts                        | Flag compromised EC2 instance     |
| **Lambda**             | Containment    | Automated remediation                | Quarantine instance via SG change |
| **Detective**          | Analysis       | Behavior graphs                      | Trace lateral movement            |
| **Backup**             | Recovery       | Immutable backups                    | Restore encrypted S3 data         |

**Exam Tip**:  
- Use **SSM Incident Manager** to orchestrate cross-team response workflows.

---

## 8. **Common Exam Confusions**
| Scenario                               | Correct Answer                              | Why?                                                                 |
|----------------------------------------|---------------------------------------------|----------------------------------------------------------------------|
| **Block public S3 bucket access**      | S3 Block Public Access (bucket policy)      | IAM policies control user access; S3 settings enforce bucket-level security. |
| **Prevent root account usage**         | SCPs in AWS Organizations                   | IAM policies don’t apply to root users; SCPs restrict account-wide actions. |
| **Encrypt EBS volumes by default**     | AWS Config + KMS                            | Use Config rules to enforce encryption, not just bucket policies.    |
| **Detect exposed AWS keys in GitHub**  | GuardDuty (Credential exfiltration finding) | Macie scans S3, not external sources.                                |

---

## Summary Cheat Sheet:
- **"Who did what?"** → **CloudTrail + Athena**  
- **"Is my data safe?"** → **Macie + KMS**  
- **"Am I vulnerable?"** → **Inspector + Security Hub**  
- **"Was I attacked?"** → **GuardDuty + Detective**  
- **"Is my network secure?"** → **WAF + Network Firewall**  

---

## Exam Scenarios:
1. **Unauthorized API Calls**:  
   - **Detection**: GuardDuty → **Investigation**: Detective → **Remediation**: Lambda + SSM  
2. **Data Leakage**:  
   - **Identify**: Macie → **Contain**: S3 Block Public Access → **Audit**: Config + Artifact  
3. **DDoS Attack**:  
   - **Mitigate**: Shield Advanced + WAF → **Analyze**: VPC Flow Logs + Athena  

---
