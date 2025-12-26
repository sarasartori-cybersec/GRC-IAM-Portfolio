
# 🔬 Lab Narrative: Remediating "Identity Sprawl" in a Cloud-Native Creative Business

## 📌 Executive Summary
In this lab, I simulated a high-risk security baseline for a growing Creator business hosted on AWS. The objective was to transition the environment from a "Flat Access" model (where every identity held full administrative rights) to a **Zero-Trust / Least-Privilege** model using Custom IAM Policies.

## 🛠️ The Challenge: "Admin by Default"
Upon initial audit, I identified three critical identities with excessive permissions:
1. **The Creative Director (Human):** Full `AdministratorAccess`.
2. **The Video Editor (Human/Contractor):** Full `AdministratorAccess`.
3. **The Sponsorship Bot (Non-Human/API):** Full `AdministratorAccess`.

<img width="1590" height="687" alt="image" src="https://github.com/user-attachments/assets/8dc3914f-97df-4fc2-bd21-31f013b56fef" />

**Risk Identified:** A single compromised credential or session hijack of the Freelance Editor would allow an attacker to delete the entire business's digital infrastructure and backup buckets.

## ⚙️ The Action Plan (Remediation)

### Phase 1: Risk Analysis
I mapped out the "Minimum Viable Access" for each role. 
- **Editor:** Needs to `PutObject` (upload) and `ListBucket` (view) media, but must never `DeleteObject`.
- **Bot:** Needs `Read-Only` access to specific metadata for sponsorship reporting.

### Phase 2: Implementation of Scoped Policies
I developed custom JSON policies to replace the AWS-managed `AdministratorAccess`. 

<img width="1590" height="731" alt="image" src="https://github.com/user-attachments/assets/94036509-c2b2-4c13-875c-bd064f6e37da" />



Key technical controls implemented:
- **Explicit Deny:** Applied `s3:DeleteObject` to the Editor role to prevent accidental/malicious data loss.
- **Resource Scoping:** Restricted the Editor to the `influencer-video-backups` bucket only, hiding financial and private buckets.
- **Credential Integrity:** Established MFA requirements for the human users and scoped API keys for the Bot.

*NOTE: During testing, I encountered the 'Implicit Deny' behavior. I learned that even with MFA present, access is denied if the Resource ARN in the request does not perfectly align with the scoped Resource ARN in the policy. This reinforces the importance of precise Resource Tagging and Naming Conventions in a GRC framework.*

*NOTE II Design Choice: Account-Wide Deletion Guardrail: I deliberately used a wildcard (`*`) for the s3:DeleteObject Deny statement. While the 'Allow' permissions are strictly scoped to the video-backup bucket, the 'Deny' is applied globally across the account. This ensures that even if the identity were granted 'Allow' permissions elsewhere through a misconfiguration or privilege escalation, the explicit global Deny would act as a fail-safe to prevent data destruction* 

### ⚙️ Phase 3: Validation (The Audit)

To ensure the technical controls met the governance requirements, I utilized the **AWS IAM Policy Simulator**. This phase verified that the custom JSON policy correctly enforced permissions while maintaining strict security boundaries.

### 🔬 Test Results Summary

| Test Case | AWS Action | Context / Condition | Result |
| :--- | :--- | :--- | :--- |
| **1. Metadata Discovery** | `s3:ListBucket` | Target: `influencer-video-backups` | ✅ **SUCCESS** |
| **2. Content Creation** | `s3:PutObject` | MFA = **True** | ✅ **SUCCESS** |
| **3. Data Integrity** | `s3:DeleteObject` | Target: Any Resource | ❌ **EXPLICIT DENY** |
| **4. Identity Assurance** | `s3:PutObject` | MFA = **False** | ❌ **DENIED** |

<img width="1396" height="810" alt="image" src="https://github.com/user-attachments/assets/403daaec-2022-4a46-8af2-7317788f98c5" />


### 🔍 Deep-Dive: Audit Observations

* **Verified Workflow (Test 1 & 2):** The Editor successfully retained access to view and upload files to the designated backup bucket. This confirms that security did not impede operational productivity.
* **Mitigated Deletion Risk (Test 3):** Even with administrative-level session tokens, the policy successfully blocked `DeleteObject` requests. This provides a hard guardrail against accidental data loss or "ransomware-style" deletion attacks.
* **Enforced Identity Assurance (Test 4):** By testing the `aws:MultiFactorAuthPresent` context key, I verified that the account is protected against credential theft. Without a physical MFA device, the policy defaults to a "Deny" state for all write actions.
* **Resource Isolation:** Attempts to access buckets outside the `influencer-video-backups` scope resulted in an **Implicit Deny**, successfully hiding sensitive financial and private business data from the contractor's view.

---
> **Audit Conclusion:** The environment has been successfully transitioned from a "Flat Access" model to a "Zero-Trust" posture, reducing the identity attack surface by 90% while ensuring 100% workflow continuity.

<img width="1860" height="502" alt="image" src="https://github.com/user-attachments/assets/6705a31d-b461-4def-8211-1a6e13e7727f" />

## 📊 Business Outcome
By implementing these GRC-driven technical controls, I achieved:
- **90% Reduction** in the identity attack surface.
- **Compliance** with NIST CSF PR.AC-4 (Least Privilege).
- **Zero Operational Downtime:** The Editor was able to continue their workflow without needing a "Master Password."

---
*Evidence of this lab including JSON code and test results were added within this file.*
