# Data Classification Guide for Power Automate

**Organization:** [Your Organization Name]
**Last Updated:** [Date]
**Owner:** [Data Governance Team/Contact]
**Related Policy:** [Link to Organization's Data Classification Policy]

## Overview

This guide explains how our organization's data classification system applies to Power Automate development. Understanding data classification is critical for:

- Selecting approved connectors
- Applying appropriate security controls
- Meeting compliance requirements
- Avoiding data breaches and violations

## Data Classification Levels

### Level 1: Public
**Definition:** Information that is already public or approved for public release.

**Examples:**
- Published press releases
- Public website content
- Marketing materials
- General product information

**Automation Requirements:**
- ✅ Any approved connector can be used
- ✅ No special security controls required
- ✅ Can be stored/processed anywhere
- ℹ️ Still follow basic security best practices

**Flow Naming Convention:** `[PUBLIC]` prefix

---

### Level 2: Internal
**Definition:** Information intended for internal use within the organization but not sensitive.

**Examples:**
- Internal announcements
- General policies and procedures
- Non-sensitive employee information
- Internal meeting schedules

**Automation Requirements:**
- ⚠️ Must use approved connectors only
- ⚠️ No external sharing
- ⚠️ Service accounts required for production
- ⚠️ Basic audit logging

**Approved Connectors:**
- SharePoint Online
- Microsoft Teams (internal channels)
- Outlook (internal email)
- Dataverse
- [Add your organization's approved connectors...]

**Flow Naming Convention:** `[INTERNAL]` prefix

---

### Level 3: Confidential
**Definition:** Sensitive business information that could harm the organization if disclosed.

**Examples:**
- Financial data
- Customer information
- Strategic plans
- Vendor contracts
- Employee personal data (non-PII)

**Automation Requirements:**
- 🔒 Must use approved connectors with Confidential classification
- 🔒 Encryption in transit (HTTPS/TLS only)
- 🔒 Service accounts with least privilege
- 🔒 Comprehensive audit logging
- 🔒 No external storage/processing
- 🔒 Data Loss Prevention (DLP) policies enforced
- 🔒 Must document in flow description

**Approved Connectors:**
- SharePoint Online (internal sites only)
- Dataverse
- OneDrive for Business
- Excel Online (Business)
- Azure Key Vault
- SQL Server (with approvals)
- [Add your organization's approved connectors...]

**Restricted:**
- No consumer services (personal OneDrive, Gmail, etc.)
- No third-party storage
- No email to external recipients

**Flow Naming Convention:** `[CONFIDENTIAL]` prefix

**Required Controls:**
1. **Access Control:**
   - Service account with minimum permissions
   - No personal accounts in production

2. **Audit Logging:**
   - Enable flow run history
   - Log all data access
   - Retain logs per policy

3. **Documentation:**
   - Data processed must be documented in flow description
   - Data retention period must be specified
   - Purpose must be clearly stated

---

### Level 4: Highly Confidential
**Definition:** Critical data subject to regulatory requirements or severe consequences if disclosed.

**Examples:**
- Personal Identifiable Information (PII) under GDPR
- Payment Card Information (PCI)
- Healthcare information (where applicable)
- Trade secrets
- Highly sensitive customer data
- Legal documents

**Automation Requirements:**
- 🔐 Must use approved connectors with Highly Confidential classification
- 🔐 Encryption in transit AND at rest
- 🔐 Managed identities or certificates (no passwords)
- 🔐 Comprehensive audit logging with SIEM integration
- 🔐 Data Loss Prevention (DLP) policies enforced
- 🔐 Data must stay in approved geographic region
- 🔐 Requires security review before deployment
- 🔐 Requires Data Protection Impact Assessment (DPIA) if GDPR applies
- 🔐 Must document retention period and deletion process

**Approved Connectors:**
- Dataverse (with row-level security)
- Azure Key Vault
- Azure SQL (with approvals)
- HTTP with Azure AD (approved APIs only)
- [Add your organization's approved connectors...]

**Strictly Prohibited:**
- Any consumer services
- Any third-party storage
- Any external processing
- Any email to external recipients
- Any logs outside organization

**Flow Naming Convention:** `[HIGHLY-CONFIDENTIAL]` prefix

**Required Controls:**
1. **Access Control:**
   - Managed identity or certificate authentication
   - Absolutely no passwords or API keys in flows
   - Row-level security where applicable

2. **Audit Logging:**
   - Comprehensive logging to SIEM
   - Real-time alerts on data access
   - Log retention per regulatory requirements

3. **Encryption:**
   - TLS 1.2+ in transit
   - AES-256 at rest (or equivalent)
   - Key management via Azure Key Vault

4. **Documentation:**
   - Complete data flow diagram
   - DPIA if processing personal data
   - Security review sign-off
   - Incident response plan

5. **Compliance:**
   - GDPR compliance (if applicable)
   - Industry-specific compliance (PCI, HIPAA, etc.)
   - Regular compliance audits

---

## Determining Data Classification

### Classification Decision Tree

```
1. Is this data subject to regulatory requirements (GDPR, PCI, HIPAA)?
   YES → Highly Confidential
   NO → Continue

2. Does this data contain Personal Identifiable Information (PII)?
   YES → Highly Confidential or Confidential (depending on sensitivity)
   NO → Continue

3. Would disclosure cause significant harm to the organization or individuals?
   YES → Confidential
   NO → Continue

4. Is this information intended only for internal organization use?
   YES → Internal
   NO → Public
```

### Classification Examples by Data Type

| Data Type | Classification | Notes |
|-----------|---------------|-------|
| Employee email addresses | Confidential | Internal use only |
| Employee Social Security Numbers | Highly Confidential | PII under GDPR |
| Customer names and addresses | Highly Confidential | PII under GDPR |
| Customer order history | Highly Confidential | Personal data |
| Public product catalog | Public | Already published |
| Internal meeting notes | Internal | Non-sensitive |
| Financial statements | Confidential | Business sensitive |
| Payment card data | Highly Confidential | PCI requirements |
| Marketing materials | Public | Approved for release |
| Strategic plans | Confidential | Competitive advantage |

[Customize this table with your organization's common data types]

## Connector Restrictions by Classification

Quick reference table:

| Connector | Public | Internal | Confidential | Highly Confidential |
|-----------|--------|----------|--------------|-------------------|
| SharePoint Online | ✅ | ✅ | ✅ (internal) | ⚠️ (with controls) |
| Microsoft Teams | ✅ | ✅ | ⚠️ (internal) | ❌ |
| Outlook | ✅ | ✅ (internal) | ⚠️ (internal) | ❌ (external) |
| Dataverse | ✅ | ✅ | ✅ | ✅ (with RLS) |
| OneDrive (Business) | ✅ | ✅ | ✅ | ⚠️ (with approval) |
| Excel Online | ✅ | ✅ | ✅ | ⚠️ (with approval) |
| Azure Blob Storage | ✅ | ✅ | ✅ | ✅ (encrypted) |
| HTTP | ✅ | ⚠️ | ⚠️ | ⚠️ (approved APIs) |
| SQL Server | ⚠️ | ⚠️ | ⚠️ | ✅ (with controls) |
| OneDrive (Personal) | ⚠️ | ❌ | ❌ | ❌ |
| Gmail | ⚠️ | ❌ | ❌ | ❌ |
| Dropbox | ❌ | ❌ | ❌ | ❌ |

✅ = Approved for this classification
⚠️ = Requires approval/restrictions
❌ = Not permitted for this classification

See [approved-connectors.md](approved-connectors.md) for detailed connector information.

## Data Sovereignty and Regional Requirements

### Approved Data Regions
[List your organization's approved regions for data storage]

Examples:
- ✅ EU (European Union)
- ✅ US (United States)
- ❌ Non-approved regions

### Regional Restrictions by Classification

**Highly Confidential:**
- Must remain in [Primary Region, e.g., EU]
- No cross-border transfers without approval

**Confidential:**
- Must remain in approved regions
- Cross-border transfers require documentation

**Internal & Public:**
- Any approved region

## Handling Mixed Classifications

### Rule: Use the Highest Classification

If a flow processes data of multiple classifications, use the most restrictive:

**Example:**
- Flow processes Public product catalog + Confidential customer orders
- Classification: **Confidential** (most restrictive applies)

### Separating Classifications

When possible, separate flows by classification:

**Better Approach:**
- Flow 1: Public product lookup (Public classification)
- Flow 2: Customer order processing with product references (Confidential)

## Data Retention and Deletion

| Classification | Retention Period | Deletion Requirements |
|----------------|------------------|----------------------|
| Public | [Your policy] | Standard deletion |
| Internal | [Your policy] | Standard deletion |
| Confidential | [Your policy] | Secure deletion + audit log |
| Highly Confidential | [Your policy] | Secure deletion + audit log + verification |

### Implementing Retention in Flows

**Pattern: Data Retention Check**
```
1. Check data age
2. If older than retention period:
   - Secure deletion
   - Log deletion action
   - Verify deletion
```

See [flow-patterns/data-retention.md](flow-patterns/data-retention-pattern.md) for implementation.

## Compliance Implications

### GDPR (if applicable)

**Personal Data = Highly Confidential or Confidential**

Requirements:
- Data Protection Impact Assessment (DPIA)
- Purpose limitation
- Data minimization
- Right to erasure implementation
- Data portability
- Consent management

### Industry-Specific Regulations

[Customize based on your industry]

**PCI DSS (Payment Card Industry):**
- Payment data = Highly Confidential
- No storage of CVV/PIN
- Encryption required
- Regular audits

**HIPAA (Healthcare):**
- PHI = Highly Confidential
- Additional access controls
- Breach notification procedures

## Related Documents

- [Approved Connectors List](approved-connectors.md)
- [Compliance Requirements](compliance-requirements.md)
- [Systems Integration Guide](systems-integration.md)
- [Internal: Data Classification Policy] - [Link]
- [Internal: GDPR Compliance Guide] - [Link]

## Getting Help

**Questions about data classification:**
- Data Governance Team: [Email]
- Privacy Officer: [Email]
- Security Team: [Email]

**Questions about automation:**
- Power Platform COE: [Email]
- Architecture Team: [Email]

## Change History

| Date | Change | Changed By |
|------|--------|------------|
| [Date] | Initial version | [Name] |

---

**Note:** This is a template. Customize it with your organization's actual classification system, requirements, and contacts.
