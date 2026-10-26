# CIS Azure Foundations Benchmark — Compliance Automation

**Benchmark:** CIS Azure Foundations Benchmark v2.0  
**Environment:** Nexus Financial Group — Microsoft Entra ID + Azure  
**Author:** DeM White | **Date:** September 2026

This repository documents an automated compliance assessment of the Nexus Financial Group Azure environment against the CIS Azure Foundations Benchmark. The assessment is executed via PowerShell against the Azure Management API and Microsoft Graph API — no portal clicking, no manual checklists.

---

## Why This Exists

Running a compliance assessment manually against a checklist is what auditors do. Building a script that queries your environment, evaluates each control programmatically, and produces a structured report is what security engineers do. This project does the latter.

The script queries the live Nexus Financial Group environment — the same tenant built in Project 1 and monitored in Project 3 — and produces an objective pass/fail/NA result for each applicable CIS control. Every finding has evidence. Every NA has a documented business justification. Every failure has a remediation path.

This is also the project where the identity controls built in Project 1 get independently verified. The CA policies, PIM configuration, named locations, and admin role counts all show up as PASS here — not because we said they were configured correctly, but because the script proved it.

---

## Assessment Results

**Date:** September 7, 2026  
**Total controls evaluated:** 34  
**Applicable controls:** 11  
**Passing:** 11  
**Failing:** 0  
**Not Applicable:** 23  
**Compliance score (applicable controls):** 100%

---

## Results By Section

### Section 1 — Identity and Access Management (7/7 applicable passing)

| Control | Title | Status |
|---|---|---|
| 1.1 | Security Defaults disabled when CA active | ✅ PASS |
| 1.2 | MFA enabled for all users | ✅ PASS |
| 1.3 | MFA enabled for admin roles | ✅ PASS |
| 1.4 | Fewer than 5 permanent Global Admins | ✅ PASS |
| 1.5 | Named locations configured | ✅ PASS |
| 1.6 | Guest invitations restricted | ✅ PASS |
| 1.7 | Azure AD Password Protection configured | ⚪ NA |
| 1.8 | PIM used for privileged roles | ✅ PASS |

Section 1 validates the entire identity architecture built in Project 1. Every applicable identity control passes independently — this is verification, not self-assertion.

**1.7 NA justification:** Azure AD Password Protection requires an on-premises agent deployed to domain controllers. The NFG hybrid environment has AD Connect configured but the Password Protection agent is not deployed in the lab. This is documented as a production requirement.

---

### Section 2 — Microsoft Defender for Cloud (1/1 applicable passing)

| Control | Title | Status |
|---|---|---|
| 2.1–2.8 | Defender plans for various resource types | ⚪ NA |
| 2.9 | Security contact email configured | ✅ PASS |

Defender plans for VMs, SQL, AppServices, Storage, Containers, Key Vault, DNS, and ARM are NA because NFG is an identity-first environment — none of those resource types are deployed. Defender plans are documented as production requirements when those resource types are provisioned.

---

### Sections 3–4 — Storage and Database (All NA)

No storage accounts or database services deployed in NFG lab environment. All controls documented as NA with production provisioning guidance.

---

### Section 5 — Logging and Monitoring (3/3 passing)

| Control | Title | Status |
|---|---|---|
| 5.1 | Diagnostic settings capture appropriate logs | ✅ PASS |
| 5.2 | Log Analytics workspace configured | ✅ PASS |
| 5.3 | Microsoft Sentinel deployed | ✅ PASS |

Full logging and monitoring stack confirmed — subscription diagnostic settings routing Administrative, Security, Alert, and Policy logs to the law-nexus-sentinel workspace.

---

### Sections 6–9 — Networking, VMs, Key Vault, AppService (All NA)

No IaaS workloads, NSGs, VNets, VMs, Key Vault, or App Services deployed. NFG is an identity-focused cloud environment. All controls documented as NA with production provisioning guidance.

---

## Remediation Actions Taken During Assessment

Three controls initially failed and were remediated during the assessment:

**1.6 — Guest invitations:** Guest invite policy had been set to "everyone" during Project 1 Sentinel setup to add a guest user. Tightened to `adminsAndGuestInviters` after Sentinel workspace was established.

**2.9 — Security contact:** No security contact was configured in Defender for Cloud. Configured via Azure Management API — alertNotifications enabled for High severity, notifications to Owner role.

**5.1 — Diagnostic settings:** No subscription-level diagnostic settings existed. Created `NFG-Subscription-DiagSettings` routing Administrative, Security, Alert, and Policy logs to law-nexus-sentinel Log Analytics workspace.

---

## How The Script Works

The assessment script uses two authentication contexts:

**Azure Management API** via `Invoke-AzRestMethod` — queries subscription-level resources including Defender for Cloud, diagnostic settings, and Log Analytics workspaces.

**Microsoft Graph API** via `Invoke-MgGraphRequest` — queries Entra ID controls including Security Defaults, Conditional Access policies, named locations, PIM assignments, and authorization policies.

Running both in the same session against separate tenants (Gmail Azure subscription + Nexus Financial Group tenant) requires explicit tenant targeting for Graph — `Connect-MgGraph -TenantId {nexus-tenant-id}`. Without this, Graph defaults to the subscription's home tenant and returns empty results for all identity controls.

The script outputs a CSV with Section, Control, Title, Status, Finding, and Remediation columns — structured for import into any GRC tool or spreadsheet.

---

## NA Controls — Production Roadmap

| Resource type | Controls | Production action |
|---|---|---|
| Virtual Machines | 2.1, 6.1, 6.2, 7.1, 7.2 | Enable Defender for Servers, restrict RDP/SSH, enable disk encryption |
| Storage Accounts | 2.4, 3.1, 3.2, 3.3 | Enforce secure transfer, disable public access, rotate keys |
| Database Services | 2.2, 4.1, 4.2 | Enable Defender for SQL, enable audit, enable TDE |
| App Services | 2.3, 9.1, 9.2 | Enable Defender for App, enforce authentication and HTTPS |
| Key Vault | 2.6, 8.1, 8.2 | Enable soft delete, purge protection, and diagnostic logging |
| Networking | 6.3 | Enable Network Watcher when VNet resources are provisioned |
| Password Protection | 1.7 | Deploy Azure AD Password Protection agent to on-premises DCs |

---

## Repository Contents

```
cis-azure-benchmark-automation/
├── README.md
└── NFG-CIS-Assessment-Final-20260907.csv    — Full assessment results, all 100+ controls
```

---

## Relationship To Other Projects

**Project 1 — IAM Architecture:** The identity controls evaluated in Section 1 (CA policies, PIM, named locations, admin counts) were built in Project 1. This assessment independently verifies they are correctly configured.

**Project 3 — Sentinel SIEM:** The logging controls in Section 5 (diagnostic settings, Log Analytics, Sentinel) were built in Project 3. This assessment confirms the logging stack is complete.

**Project 5 — Identity Threat Landscape Report:** The NA controls in this assessment become the residual risk documented in Project 5 — controls that are designed but not yet deployed represent accepted risk pending production provisioning.

---

*Part of a six-project cloud security portfolio. All company names, users, and data are fictional.*
