# AWS IAM Services Deep Dive

## 1. **AWS Identity and Access Management (IAM)**  
**Description**:  
The foundational service for managing **authentication** (who can log in) and **authorization** (what they can do) in AWS.  

### Key Components:  
- **Users**: Permanent identities for humans/applications.  
- **Roles**: Temporary credentials for AWS services (e.g., EC2, Lambda).  
- **Policies**: JSON documents defining permissions (e.g., `Allow S3:ListBucket`).  
- **Groups**: Collections of users with shared permissions.  

### Critical Features:  
- **Multi-Factor Authentication (MFA)**: Enforce MFA for sensitive operations.  
- **Permissions Boundaries**: Limit maximum permissions a role/user can have.  
- **Resource-Based Policies**: Attach policies directly to resources (e.g., S3 bucket policies).  
- **Access Analyzer**: Identify unintended resource exposure.  

### Use Cases:  
- Granting cross-account access via `sts:AssumeRole`.  
- Enforcing least privilege with policy conditions (e.g., `aws:SourceIP`).  
- Rotating credentials using IAM Access Keys lifecycle policies.  

### Key Permissions:  
```plaintext
iam:CreatePolicyVersion      # Modify existing policies  
iam:AttachUserPolicy         # Assign policies to users  
iam:PassRole                 # Allow a user/role to pass a role to an AWS service  
```

### Exam Tips:  
- **Policy Evaluation Logic**: Understand order of evaluation (explicit deny > allow > implicit deny).  
- **IAM vs SCPs**: IAM policies apply to users/roles; SCPs apply to entire AWS accounts (via Organizations).  

---

## 2. **AWS Organizations**  
**Description**:  
Centralized governance for multi-account AWS environments.  

### Key Features:  
- **Service Control Policies (SCPs)**: Guardrails to restrict account-level permissions.  
- **Consolidated Billing**: Single payment method for all accounts.  
- **AWS Organizations API**: Automate account creation and management.  

### Use Cases:  
- Block root account access in child accounts via SCPs.  
- Restrict regions using SCP conditions (e.g., `"Condition": {"StringNotEquals": {"aws:RequestedRegion": "us-east-1"}}`).  

### Key Permissions:  
```plaintext
organizations:EnablePolicyType  # Enable SCPs for an OU  
organizations:AttachPolicy      # Apply SCPs to accounts/OUs  
```

### Exam Tips:  
- SCPs **do not** grant permissions – they only deny or allow.  
- SCPs affect **all users/roles** in an account, including the root user.  

---

## 3. **AWS Single Sign-On (SSO)** / **IAM Identity Center**  
**Description**:  
Centralized identity management for AWS and third-party SaaS applications (e.g., Salesforce, Microsoft 365).  

### Key Features:  
- SAML 2.0 integration with identity providers (e.g., Azure AD, Okta).  
- Pre-built AWS managed permissions sets (e.g., `ReadOnlyAccess`).  
- Attribute-Based Access Control (ABAC) using user tags.  

### Use Cases:  
- Federated access to multiple AWS accounts via SAML.  
- Assign temporary credentials for CLI access via AWS SSO.  

### Key Permissions:  
```plaintext
sso:CreatePermissionSet     # Define SSO permission templates  
sso-directory:CreateUser    # Manage users in the SSO directory  
```

### Exam Tips:  
- AWS SSO replaces the legacy "AWS SSO" service – know the name change to **IAM Identity Center**.  
- SSO integrates with **AWS Managed Microsoft AD** for hybrid environments.  

---

## 4. **AWS Security Token Service (STS)**  
**Description**:  
Issues temporary credentials for IAM roles or federated users.  

### Key APIs:  
- `AssumeRole`: For cross-account access or role switching.  
- `AssumeRoleWithSAML`: For federated users via SAML providers.  
- `GetSessionToken`: For MFA-protected API access.  

### Use Cases:  
- Temporary credentials for CI/CD pipelines (e.g., GitHub Actions).  
- Assuming roles in another account for centralized logging.  

### Key Permissions:  
```plaintext
sts:AssumeRole              # Core API for role assumption  
sts:DecodeAuthorizationMessage  # Troubleshoot access denied errors  
```

### Exam Tips:  
- Temporary credentials last **15 minutes to 12 hours**.  
- Use `aws:PrincipalArn` in policies to restrict role assumption.  

---

## 5. **AWS Directory Service**  
**Description**:  
Managed Microsoft Active Directory (AD) for AWS and hybrid environments.  

### Key Flavors:  
- **AWS Managed Microsoft AD**: Fully managed AD with two domain controllers.  
- **AD Connector**: Proxy to redirect authentication requests to on-prem AD.  
- **Simple AD**: Samba-based lightweight directory (no advanced AD features).  

### Use Cases:  
- Join EC2 instances to a domain for centralized authentication.  
- Enable LDAP integration for applications like Jenkins or Grafana.  

### Key Permissions:  
```plaintext
ds:CreateDirectory          # Deploy a new directory  
ds:CreateAlias              # Assign a DNS alias (e.g., corp.example.com)  
```

### Exam Tips:  
- **AWS Managed Microsoft AD** supports **Kerberos-based authentication** for SMB file shares.  
- Use **AD Connector** to avoid storing user credentials in AWS.  

---

## 6. **AWS Resource Access Manager (RAM)**  
**Description**:  
Share AWS resources across accounts without exposing IAM roles.  

### Key Features:  
- Share resources like VPC subnets, Transit Gateways, and License Manager configurations.  
- Centrally manage sharing via AWS Organizations.  

### Use Cases:  
- Share a central inspection VPC with multiple accounts.  
- Avoid resource duplication (e.g., shared Route 53 Resolver rules).  

### Key Permissions:  
```plaintext
ram:CreateResourceShare     # Define shared resources  
ram:AssociateResourceShare  # Link shares to accounts/OUs  
```

### Exam Tips:  
- Shared resources still follow IAM policies – RAM simplifies sharing, not permissions.  
- Use **AWS RAM** with **Organizations** for large-scale sharing.  

---

## Summary Table: IAM Services Comparison

| Service              | Scope                      | Key Use Case                          |
|----------------------|----------------------------|---------------------------------------|
| **IAM**              | Account-level              | Granular user/role permissions        |
| **Organizations**    | Multi-account              | Enforce SCP guardrails                |
| **IAM Identity Center** | Cross-account SSO       | Centralized access to multiple accounts |  
| **STS**              | Temporary credentials      | Role assumption for CI/CD             |
| **Directory Service**| Hybrid identity            | EC2 domain joining                    |
| **RAM**              | Resource sharing           | Share VPC subnets across accounts     |

---

## Exam Scenarios:  
1. **Cross-Access**: Use `sts:AssumeRole` + IAM trust policies for secure cross-account access.  
2. **Emergency Lockdown**: Apply SCPs in Organizations to disable all IAM changes during a breach.  
3. **Federation**: Configure SAML with AWS SSO to grant temporary access to contractors.  

