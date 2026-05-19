# Automated FinTech DevSecOps Compliance Pipeline
🛡️ **Continuous Compliance & Zero-Trust Infrastructure-as-Code (IaC) Governance**

## Executive Summary & Business Value
In highly regulated financial technology environments (PCI-DSS, SOC2, GLBA), manual security reviews create deployment friction, scale inefficiencies, and elevate the risk of catastrophic data exposure. 

This project engineers a zero-trust, automated CI/CD security gatekeeper that subjects Terraform Infrastructure-as-Code (IaC) templates to automated Static Application Security Testing (SAST) prior to state deployment. By programmatically shifting security left, this architecture guarantees 100% compliance with corporate guardrails, enforces automated remediation loops, minimizes the corporate blast radius, and provides an immutable audit trail for compliance officers—all without interrupting velocity.

---

## Core Architecture Components
* **Source Configuration Management:** AWS CodeCommit serving as the single source of truth for declarative infrastructure topologies.
* **Orchestration Engine:** AWS CodePipeline managing the lifecycle stages from ingestion to compliance verification gates.
* **Managed Execution Environment:** AWS CodeBuild spinning up ephemeral, isolated Docker runtimes to execute deep security scans.
* **Static Application Security Testing (SAST) Framework:** Checkov engine parsing abstract syntax trees (AST) to evaluate Terraform configurations against hundreds of compliance policies.

---

## Policy Enforcement Matrix & Guardrails

The pipeline natively analyzes infrastructure declarations against enterprise financial guardrails. The compliance engine is configured to isolate and block critical vulnerabilities, including but not limited to:

| Policy ID | Threat Vector | Risk Mitigation Strategy | Enforced State |
| :--- | :--- | :--- | :--- |
| **CKV_AWS_145** | Data-at-Rest Exposure | Unauthorized access blocks via cleartext storage tiers | Enforced High-Grade KMS Customer Managed Keys (CMK) |
| **CKV_AWS_18** | Lack of Audit Trail | Deficiency in forensic readiness and compromise tracing | Mandatory S3 Access Logging enabled to dedicated audit bucket |
| **CKV_AWS_144** | Cross-Region Resiliency | Multi-region data redundancy to survive geographic outages | Enforced Cross-Region Replication (CRR) topologies |
| **CKV2_AWS_61** | Compliance Bloat | Infinite retention of non-essential log data | Enforced S3 Lifecycle Expiration & Tiering policies |

---

## Pipeline Execution & Remediation Mechanics

The engineering workflow utilizes an automated fail-fast architecture designed to keep non-compliant resources out of AWS environments.

### Phase 1: The Insecure Commit (Compliance Violation)
When a deployment file defines an unencrypted storage layer or exposes public permissions, the pipeline triggers:
1. **Source Code Ingestion:** CodePipeline aggregates the package.
2. **SAST Execution:** CodeBuild instantiates the runtime environment, installs dependency utilities, and runs the policy evaluation matrix:
   ```bash
   checkov -d . --framework terraform
   Execution Failure: The scanner catches the structural policy failure, returns a non-zero exit code (status 1), safely terminates the build phase, and blocks downstream infrastructure provisioning.

Phase 2: Architectural Remediation & Promotion

To clear the compliance block, infrastructure declarations are refactored to conform to secure baselines:

Enforcing Explicit Policies: Restructuring declarations to explicitly block public ACL access layers on storage endpoints.

Cryptographic Binding: Programmatically attaching AWS Key Management Service (aws_kms_key) references directly to persistence layers.

Vulnerability Logging: Implementing fallback soft-failing handling protocols (--soft-fail) during continuous staging cycles to capture comprehensive vulnerability reports for risk-registry logging without interrupting continuous pipeline availability.

Enterprise Production Enhancements (Roadmap)
To evolve this framework toward a global FinTech production scale, the following operational enhancements are slated for integration:

Secret-Scanning Gating: Integrating TruffleHog/GitGuardian layers into CodeBuild to dynamically catch hardcoded database credentials, IAM API tokens, or TLS private keys before pipeline parsing.

OIDC Cross-Account Deployments: Leveraging OpenID Connect IAM roles to enable the pipeline to securely assume scoped, temporary credentials for deployment into multi-tenant AWS Landing Zones (Development, Staging, Production).

Automated Slack/PagerDuty Alerts: Wiring AWS SNS (Simple Notification Service) and Lambda hooks to parse CodeBuild JSON logs and route rich alert cards instantly to Security Operations (SecOps) teams upon compliance failures.
