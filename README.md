# AWS IAM Privilege Escalation Assessment & Remediation

## Project Overview
An automated and manual security assessment of an AWS environment to identify and remediate identity-based risks. This project focused on eliminating over-privileged service roles and preventing catastrophic resource exposure.

---

## Phase 1: Automated Analysis
Utilized `cloudsplaining` via AWS CloudShell to parse account authorization details and scan for security flaws.

```bash
# Export authorization details to a JSON data package
aws iam get-account-authorization-details > iam_dump.json

# Scan the data package and generate a visual HTML report
cloudsplaining scan --input-file iam_dump.json --output report
```

### Finding 1: Resource Exposure via AmazonSESFullAccess (High Severity)
* **Vulnerability:** `LambdaSendEmailRole` was assigned global administrative control over Amazon SES via an AWS-managed policy.
* **Risk:** If compromised, an attacker could hijack the corporate email domain, modify verified identities, or launch mass phishing campaigns using the client's official domain.

---

## Phase 2: Manual Verification & Blast Radius Assessment
Inspected the automated scan report and verified the policy attachments inside the AWS IAM Management Console. Confirmed that the `LambdaSendEmailRole` was actively attached to a live lambda function, validating that the vulnerability was exploitable.

---

## Phase 3: Remediation (Enforcing Least-Privilege Access)
Successfully mitigated the high-severity risk by executing a strict least-privilege migration path:

1. Isolated the `LambdaSendEmailRole` in the IAM console.
2. Stripped the blanket AWS-managed `AmazonSESFullAccess` policy to eliminate the dangerous blast radius.
3. Authored and deployed a secure customer-managed inline JSON policy restricting the Lambda function strictly to email transmission APIs.

### Secure Remediation Policy Applied:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ses:SendEmail",
                "ses:SendRawEmail"
            ],
            "Resource": "*"
        }
    ]
}
```

---

## Phase 4: Business Impact & Executive Deliverables
The remediation successfully protected the company from corporate domain blocklisting, email infrastructure spoofing, and brand reputation destruction. The entire remediation process was completed in under an hour with zero downtime to the application.
