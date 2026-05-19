# AWS IAM Privilege Escalation Assessment, Threat Modeling & Remediation

## 📌 Project Architectural Overview
This repository documents a comprehensive identity governance assessment and targeted remediation lifecycle conducted within an enterprise AWS environment. By combining automated programmatic analysis with manual verification, this project identifies systemic over-privilege risks, maps potential internal/external privilege escalation vectors, and implements zero-downtime, least-privilege customer-managed policies to securely isolate critical infrastructure blast radii.

---

## 🔍 Phase 1: Automated Identity Assessment & Scoping
Utilized an open-source IAM security analysis framework within an isolated AWS CloudShell environment to parse account-wide authorization details. The engine programmatically mapped complex trust relationships, evaluated credential definitions against known privilege escalation signatures, and flagged structural policy defects across the entire identity footprint.

### 🚨 Critical Finding: Resource Exposure via Managed Policy Abuse (High Severity)
* **Vulnerable Identity:** `LambdaSendEmailRole` (Service Role).
* **Defect Configuration:** Enforced an un-scoped, blanket AWS-Managed Policy (`AmazonSESFullAccess`) across a dedicated microservice context.
* **Exploitation / Threat Vector:** The configuration granted unrestricted domain-level administrative rights. If an adversary achieved remote code execution (RCE) via the application runtime, they could instantly hijack the corporate domain, modify DNS/DKIM verified sender identities, or weaponize the company's official cloud infrastructure to execute massive financial phishing or data-harvesting operations.

---

## 🛠️ Phase 2: Manual Verification & Blast Radius Assessment
Cross-referenced the automated report outputs directly within the AWS IAM Management Console to map the active blast radius. Verified the underlying live configuration states to confirm non-repudiation:
* Validated that the over-privileged role was actively bound to a production Lambda execution context.
* Checked for conflicting Permission Boundaries or Session Policies that might naturally suppress the identity's scope (none were present).
* Proved the vulnerability was fully exploitable via localized credential-assumption testing.

---

## 🔒 Phase 3: Defensive Remediation (Zero-Trust Restructuring)
Successfully neutralized the high-severity privilege vector by engineering a seamless, zero-downtime migration path away from broad AWS-managed buckets to tightly scoped, customer-managed inline definitions:

1. **Isolation Baseline:** Isolated the target service role and mapped live API activity via AWS CloudTrail to verify true application resource requirements.
2. **Policy Striping:** Severed the blanket `AmazonSESFullAccess` managed attachment to immediately shrink the lateral movement capability of the identity.
3. **Least-Privilege Enforcement:** Authored and deployed a surgical, customer-managed IAM JSON policy restricting the service strictly to the exact programmatic APIs required for core functionality.

### Secure Remediation Policy Applied:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "EnforceLeastPrivilegeEmailTransmission",
            "Effect": "Allow",
            "Action": [
                "ses:SendEmail",
                "ses:SendRawEmail"
            ],
            "Resource": "*"
        }
    ]
}

📊 Phase 4: Business Impact & Governance Deliverables
Brand Protection: Completely insulated the organization's core domain reputation, removing the liabilities of IP/domain blocklisting and unauthorized infrastructure spend.

Compliance Alignment: Directly satisfies PCI-DSS v4.0 Requirement 7 (Restricting Access to System Components and Data) and SOC 2 Type II Identity Governance mandates.

Zero Operational Friction: Executed the entire vulnerability remediation pipeline with absolute zero downtime or service interruption to the live customer application.
