# AWS IAM Privilege Escalation Assessment, Threat Modeling & Remediation

## 📌 Project Architectural Overview
This repository documents a comprehensive identity governance assessment and targeted remediation lifecycle within a production AWS environment. By combining automated security analysis with manual verification, this project identifies systemic over-privilege risks, maps potential insider-threat privilege escalation paths, and implements zero-downtime, least-privilege customer-managed policies to eliminate high-severity blast radii.

---

## 🔍 Phase 1: Automated Identity Assessment & Scoping
Leveraged the open-source IAM security analysis framework `cloudsplaining` inside an isolated AWS CloudShell environment to parse account-wide authorization details, map complex trust relationships, and identify structural policy flaws.

```bash
# Export infrastructure authorization details to a JSON data package
aws iam get-account-authorization-details > iam_dump.json

# Parse the data package against privilege escalation signatures and generate visual HTML reports
cloudsplaining scan --input-file iam_dump.json --output report/
