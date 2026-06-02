# AWS IAM Privilege Escalation Assessment & Targeted Remediation

[![Provider](https://img.shields.io/badge/Provider-AWS-orange.svg)](https://aws.amazon.com/)
[![Service](https://img.shields.io/badge/Service-AWS_IAM-red.svg)](https://aws.amazon.com/iam/)
[![Category](https://img.shields.io/badge/Security-Identity_Governance-blue.svg)](https://aws.amazon.com/security/)
[![Compliance](https://img.shields.io/badge/Compliance-PCI--DSS_/_SOC_2-green.svg)](https://aws.amazon.com/security/)

## 📋 Project Overview
This repository details the end-to-end identity assessment, threat modeling, and targeted remediation lifecycle conducted within an enterprise AWS ecosystem. 

By analyzing identity trust relationships, this project isolates over-privileged AWS-Managed Policies, identifies potential lateral movement vectors, and applies zero-downtime, customer-managed least-privilege configurations to secure critical messaging infrastructure boundaries.

---

## 🔍 Vulnerability Assessment & Threat Modeling

During an infrastructure-wide scoping assessment, a critical architectural defect was isolated and validated inside the Identity Management console:

* **Vulnerable Identity:** `LambdaSendEmailRole` (Service Execution Role).
* **Defect Configuration:** Enforced an un-scoped, blanket AWS-Managed Policy (`AmazonSESFullAccess`) across a dedicated microservice context.
* **Exploitation Vector:** The default AWS policy granted unrestricted, domain-level administrative rights. If an attacker achieved Remote Code Execution (RCE) via the application runtime, they could instantly hijack the corporate email domain, modify DNS/DKIM verified sender identities, or weaponize official infrastructure to launch massive phishing or data-exfiltration campaigns.

---

## 🛠️ Least-Privilege Remediation Path

To neutralize this privilege escalation risk without causing operational downtime, the broad AWS-managed attachment was systematically replaced with a highly localized, customer-managed structure:

1. **Activity Mapping:** Tracked live service API calls via AWS CloudTrail to define the exact baseline resources required for normal application runtime.
2. **Policy Striping:** Severed the wide-open `AmazonSESFullAccess` attachment to instantly shrink the identity's lateral attack surface.
3. **Surgical Binding:** Authored and deployed a custom, customer-managed JSON policy. This configuration strips out all global infrastructure modification rights and locks execution down strictly to the specific endpoints required for transactional mail operations.

---

## 💻 Customer-Managed Least-Privilege Policy (`policy.json`)

This surgical definition explicitly restricts the application role to append-only email transmission APIs, securely binding execution parameters directly to the authorized corporate domain resource:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "RestrictToSurgicalEmailActionsOnly",
      "Effect": "Allow",
      "Action": [
        "ses:SendEmail",
        "ses:SendRawEmail"
      ],
      "Resource": "arn:aws:ses:us-east-1:123456789012:identity/yourdomain.com"
    }
  ]
}
