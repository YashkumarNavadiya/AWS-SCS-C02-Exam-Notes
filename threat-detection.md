# AWS Threat Detection Services Deep Dive

---

## 1. **Amazon GuardDuty**  
**Description**:  
A fully managed threat detection service using machine learning (ML) and threat intelligence to analyze:  
- **CloudTrail Logs**: Unusual API activity (e.g., credential compromise).  
- **VPC Flow Logs**: Anomalous network traffic (e.g., command-and-control patterns).  
- **DNS Logs**: Malicious domain resolution (e.g., domain generation algorithms).  

**Key Findings**:  
- `UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration` (EC2 keys leaked).  
- `CryptoCurrency:EC2/BitcoinTool.B!DNS` (Bitcoin mining activity).  
- `Backdoor:EC2/Spambot` (SMTP port abuse).  

**Use Cases**:  
- Detect compromised EC2 instances communicating with known malicious IPs.  
- Identify API calls from Tor exit nodes.  

**Key Permissions**:  
```plaintext
guardduty:CreateDetector       # Enable GuardDuty in a region  
guardduty:UpdateFindingsFeedback  # Mark findings as "useful" or "not useful"  
```

**Exam Tips**:  
- **Cost**: GuardDuty charges per GB of logs analyzed (no upfront cost).  
- **Multi-Account**: Enable GuardDuty across an AWS Organization for centralized management.  

---

## 2. **Amazon Inspector**  
**Description**:  
Automated vulnerability scanner for **EC2 instances**, **ECR images**, and **Lambda functions**.  

**Scan Types**:  
- **Network Reachability**: Open ports, CVSS scores for vulnerabilities.  
- **Host Assessment**: CIS benchmarks, SSH/RDP best practices.  
- **Lambda Code Scans**: Detect vulnerabilities in function dependencies.  

**Key Features**:  
- Integrates with Systems Manager (SSM Agent) for EC2 scanning.  
- Generates findings in AWS Security Hub.  

**Use Cases**:  
- Identify unpatched CVEs in Docker images (e.g., Log4j vulnerabilities).  
- Detect EC2 instances with exposed SSH port (0.0.0.0/0).  

**Key Permissions**:  
```plaintext
inspector2:ListFindings          # Retrieve vulnerability reports  
inspector2:BatchGetAccountStatus  # Check if Inspector is enabled  
```

**Exam Tips**:  
- Inspector **v2** supports Lambda and ECR (v1 is deprecated).  
- Use **EventBridge** to trigger automated remediation workflows.  

---

## 3. **Amazon Macie**  
**Description**:  
Fully managed data classification service that uses ML to detect sensitive data in **S3 buckets**.  

**Key Capabilities**:  
- **Managed Data Identifiers**: Predefined detectors for PII, PHI, credit card numbers.  
- **Custom Data Identifiers**: Regex-based patterns (e.g., employee IDs).  
- **Bucket Security Posture**: Checks for public access, encryption status.  

**Findings Examples**:  
- `SensitiveData:S3Object/Credentials` (AWS keys in S3 objects).  
- `Policy:IAMUser/S3BlockPublicAccessDisabled` (misconfigured bucket).  

**Use Cases**:  
- GDPR compliance (identify EU customer data in S3).  
- Detect accidental exposure of API keys in logs.  

**Key Permissions**:  
```plaintext
macie2:CreateClassificationJob    # Start S3 bucket scans  
macie2:GetSensitiveDataOccurrences  # List sensitive data locations  
```

**Exam Tips**:  
- Macie **does not** modify data – it only reports findings.  
- Use **Security Hub** to centralize Macie findings with GuardDuty/Inspector.  

---

## 4. **AWS Security Hub**  
**Description**:  
Centralized security findings aggregator with compliance checks (CIS, PCI DSS).  

**Key Features**:  
- **AWS Foundational Security Best Practices**: 100+ automated checks.  
- **Automated Security Findings Format (ASFF)**: Standardized JSON schema.  
- **Integrations**: 50+ AWS and third-party services (e.g., Palo Alto, Datadog).  

**Use Cases**:  
- Single pane of glass for security findings.  
- Generate compliance reports (e.g., "How many S3 buckets are unencrypted?").  

**Key Permissions**:  
```plaintext
securityhub:BatchImportFindings  # Add custom findings  
securityhub:UpdateStandardsControl  # Disable specific compliance checks  
```

**Exam Tips**:  
- Security Hub is **regional** – enable it in all relevant regions.  
- Use **insights** to create custom dashboards (e.g., "Top 10 vulnerable EC2 instances").  

---

## 5. **AWS Detective**  
**Description**:  
Visual investigation service that builds interactive graphs of AWS resource behavior.  

**Key Features**:  
- **Behavior Graphs**: Maps relationships between users, roles, resources.  
- **Timeline Analysis**: Visualize event sequences (e.g., "How did the attacker move laterally?").  
- **12-Month Data Retention**: Long-term analysis of CloudTrail/VPC Flow Logs.  

**Use Cases**:  
- Post-breach forensic analysis.  
- Identify impacted resources during an incident.  

**Key Permissions**:  
```plaintext
detective:CreateGraph              # Enable Detective for an account  
detective:ListInvestigations       # View active investigations  
```

**Exam Tips**:  
- Detective **requires GuardDuty** to be enabled.  
- No upfront cost – pay per GB of data analyzed.  

---

## 6. **AWS Network Firewall**  
**Description**:  
Managed network firewall with stateful inspection and intrusion prevention (IPS).  

**Key Features**:  
- **Suricata Rules**: Open-source rule syntax for custom threat detection.  
- **Traffic Filtering**: Block malicious IPs/domains (e.g., spamhaus.org lists).  
- **SSL/TLS Inspection**: Decrypt traffic to detect hidden threats.  

**Use Cases**:  
- Block ransomware C2 traffic at the network layer.  
- Enforce DNS filtering (e.g., prevent access to phishing domains).  

**Key Permissions**:  
```plaintext
network-firewall:CreateFirewallPolicy  # Define inspection rules  
network-firewall:AssociateFirewallPolicy  # Apply to VPCs  
```

**Exam Tips**:  
- Use **Firewall Manager** to deploy rules across multiple accounts.  

---

## Comparison Table: Threat Detection Services

| Service                | Scope                      | Detection Type              | Key Integration              |  
|------------------------|----------------------------|-----------------------------|-------------------------------|  
| **GuardDuty**          | Account/Org                | ML-driven anomalies         | CloudTrail/VPC Flow Logs      |  
| **Inspector**          | EC2/ECR/Lambda             | Vulnerability scanning      | Security Hub                  |  
| **Macie**              | S3                         | Sensitive data exposure     | Security Hub                  |  
| **Security Hub**       | Cross-service              | Findings aggregation        | GuardDuty/Inspector/Macie     |  
| **Detective**          | Account/Org                | Behavioral analysis         | GuardDuty                     |  
| **Network Firewall**   | VPC                        | Network-layer threats       | AWS Firewall Manager          |  

---

## Exam Scenarios:  
1. **Credential Compromise**:  
   - GuardDuty detects `IAMUser/AnomalousBehavior`.  
   - Security Hub aggregates findings.  
   - Detective maps API calls to identify impacted resources.  

2. **Vulnerability Response**:  
   - Inspector finds a critical CVE in an ECR image.  
   - EventBridge triggers Lambda to rebuild the image.  

3. **Data Leakage**:  
   - Macie identifies unencrypted S3 buckets with PII.  
   - Security Hub triggers an SNS alert for remediation.  

---

## Critical Permissions to Remember:  
- **GuardDuty**: `guardduty:GetFindings` (required for Security Hub integration).  
- **Security Hub**: `securityhub:EnableImportFindingsForProduct` (to ingest third-party findings).  
- **Detective**: `detective:AcceptInvitation` (for cross-account analysis).  



---
