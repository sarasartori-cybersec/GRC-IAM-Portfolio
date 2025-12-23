
# 🔬 Lab Narrative: Remediating "Identity Sprawl" in a Cloud-Native Creative Business

## 📌 Executive Summary
In this lab, I simulated a high-risk security baseline for a growing Creator business hosted on AWS. The objective was to transition the environment from a "Flat Access" model (where every identity held full administrative rights) to a **Zero-Trust / Least-Privilege** model using Custom IAM Policies.

## 🛠️ The Challenge: "Admin by Default"
Upon initial audit, I identified three critical identities with excessive permissions:
1. **The Creative Director (Human):** Full `AdministratorAccess`.
2. **The Video Editor (Human/Contractor):** Full `AdministratorAccess`.
3. **The Sponsorship Bot (Non-Human/API):** Full `AdministratorAccess`.

**Risk Identified:** A single compromised credential or session hijack of the Freelance Editor would allow an attacker to delete the entire business's digital infrastructure and backup buckets.

## ⚙️ The Action Plan (Remediation)

### Phase 1: Risk Analysis
I mapped out the "Minimum Viable Access" for each role. 
- **Editor:** Needs to `PutObject` (upload) and `ListBucket` (view) media, but must never `DeleteObject`.
- **Bot:** Needs `Read-Only` access to specific metadata for sponsorship reporting.

### Phase 2: Implementation of Scoped Policies
I developed custom JSON policies to replace the AWS-managed `AdministratorAccess`. 



Key technical controls implemented:
- **Explicit Deny:** Applied `s3:DeleteObject` to the Editor role to prevent accidental/malicious data loss.
- **Resource Scoping:** Restricted the Editor to the `influencer-video-backups` bucket only, hiding financial and private buckets.
- **Credential Integrity:** Established MFA requirements for the human users and scoped API keys for the Bot.

### Phase 3: Validation (The Audit)
I utilized the **AWS IAM Policy Simulator** to verify that the controls were working as intended.
- **Test 1 (Upload):** Result: **SUCCESS**
- **Test 2 (Delete):** Result: **EXPLICIT DENY**

## 📊 Business Outcome
By implementing these GRC-driven technical controls, I achieved:
- **90% Reduction** in the identity attack surface.
- **Compliance** with NIST CSF PR.AC-4 (Least Privilege).
- **Zero Operational Downtime:** The Editor was able to continue their workflow without needing a "Master Password."

---
*Evidence of this lab including JSON code and test results can be found in the accompanying files within this folder.*
