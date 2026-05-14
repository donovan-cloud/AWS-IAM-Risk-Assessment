# AWS IAM Privilege Escalation Assessment & Remediation

## Project Overview
An automated and manual security assessment of an AWS environment to identify and remediate identity-based risks, focus on eliminating over-privileged roles and preventing resource exposure.

## Phase 1: Automated Analysis
Utilized `cloudsplaining` via AWS CloudShell to parse account authorization details.
```bash
aws iam get-account-authorization-details > iam_dump.json
cloudsplaining scan --input-file iam_dump.json --output report
```

### Finding 1: Resource Exposure via AmazonSESFullAccess (High Severity)
* **Vulnerability:** `LambdaSendEmailRole` was assigned global administrative control over Amazon SES.
* **Risk:** Potential for domain hijacking, malicious mail relaying, and configuration tampering.
* **Remediation:** Removed `AmazonSESFullAccess` and deployed a customer-managed inline policy restricting access exclusively to email transmission APIs.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ses:SendEmail",
                "ses:SendRawEmail"
            ]
            "Resource": "*"
        }
    ]
}
```
