# Approved Connectors List

**Organization:** [Your Organization Name]
**Last Updated:** [Date]
**Owner:** [Governance Team/Contact]
**Review Frequency:** [Quarterly/Monthly]

## Overview

This document lists all Power Automate connectors with their approval status for use within our organization. Always check this list before proposing or implementing any connector in a flow.

## Approval Status Legend

- ✅ **APPROVED** - Connector is approved for use (may have data classification restrictions)
- ⚠️ **RESTRICTED** - Requires special approval or limited to specific use cases
- 🔄 **IN REVIEW** - Currently under evaluation by governance team
- ❌ **BLOCKED** - Not permitted for use; use approved alternatives

## Quick Reference: Common Connectors

| Connector | Status | Max Classification | Notes |
|-----------|--------|-------------------|-------|
| SharePoint Online | ✅ APPROVED | Confidential | Standard collaboration |
| Microsoft Teams | ✅ APPROVED | Internal | Approved channels only |
| Outlook | ✅ APPROVED | Internal | Internal email only |
| Dataverse | ✅ APPROVED | Highly Confidential | Full access |
| Excel Online (Business) | ✅ APPROVED | Confidential | SharePoint-based only |
| OneDrive for Business | ✅ APPROVED | Confidential | Business accounts only |
| Azure Blob Storage | ⚠️ RESTRICTED | Confidential | Requires IT approval |
| HTTP | ⚠️ RESTRICTED | [Depends] | Case-by-case approval |
| SQL Server | ⚠️ RESTRICTED | Highly Confidential | Database team approval |
| [Add your connectors...] | | | |

## Microsoft 365 / Core Connectors

### SharePoint Online
- **Status:** ✅ APPROVED
- **Max Data Classification:** Confidential
- **Restrictions:**
  - Must use service accounts for production flows
  - Document libraries must follow naming conventions
  - No external sharing-enabled sites
- **Approval Process:** Self-service for approved classifications
- **Contact:** [SharePoint Team Email]

### Microsoft Teams
- **Status:** ✅ APPROVED
- **Max Data Classification:** Internal
- **Restrictions:**
  - Approved organizational channels only
  - No posting to external guests
  - No private/personal Teams
- **Approval Process:** Self-service
- **Contact:** [Teams Admin Email]

### Outlook (Office 365)
- **Status:** ✅ APPROVED
- **Max Data Classification:** Internal
- **Restrictions:**
  - Internal organization email only
  - No external recipients for automated emails
  - Must use service account mailboxes for production
- **Approval Process:** Self-service
- **Contact:** [Email Admin]

### Dataverse
- **Status:** ✅ APPROVED
- **Max Data Classification:** Highly Confidential
- **Restrictions:**
  - Must follow environment strategy (dev/test/prod)
  - Service accounts required
  - Solutions must use connection references
- **Approval Process:** Self-service
- **Contact:** [Power Platform COE]

### Excel Online (Business)
- **Status:** ✅ APPROVED
- **Max Data Classification:** Confidential
- **Restrictions:**
  - SharePoint or OneDrive for Business only
  - Must be properly formatted tables
  - No personal OneDrive
- **Approval Process:** Self-service
- **Contact:** [Power Platform COE]

### OneDrive for Business
- **Status:** ✅ APPROVED
- **Max Data Classification:** Confidential
- **Restrictions:**
  - Business accounts only
  - No external sharing
  - Service accounts preferred for production
- **Approval Process:** Self-service
- **Contact:** [IT Support]

## Azure Connectors

### Azure Blob Storage
- **Status:** ⚠️ RESTRICTED
- **Max Data Classification:** Confidential
- **Restrictions:**
  - Must use organization-managed storage accounts
  - Requires network security review
  - Must be in approved Azure region
  - Requires encryption at rest
- **Approval Process:** Submit request to [Cloud Team Email]
- **Approval Timeline:** 5-10 business days
- **Contact:** [Cloud Infrastructure Team]

### Azure SQL Database
- **Status:** ⚠️ RESTRICTED
- **Max Data Classification:** Highly Confidential
- **Restrictions:**
  - Database team must provision connection
  - Read-only access unless approved
  - Must use managed identity
  - Requires change management ticket
- **Approval Process:** Submit request to [Database Team Email]
- **Approval Timeline:** 10-15 business days
- **Contact:** [Database Team]

### Azure Key Vault
- **Status:** ✅ APPROVED
- **Max Data Classification:** Highly Confidential
- **Restrictions:**
  - Must use managed identity authentication
  - Read-only access
  - Secrets must follow naming conventions
- **Approval Process:** Self-service after Key Vault provisioning
- **Contact:** [Security Team]

## Integration Connectors

### HTTP / HTTP with Azure AD
- **Status:** ⚠️ RESTRICTED
- **Max Data Classification:** Depends on target system
- **Restrictions:**
  - Case-by-case approval required
  - Must document target system and purpose
  - API must be in organization's approved API catalog
  - Must use proper authentication (no API keys in flows)
- **Approval Process:** Submit request to [Architecture Team]
- **Contact:** [Enterprise Architecture]

### SQL Server (On-Premises)
- **Status:** ⚠️ RESTRICTED
- **Max Data Classification:** Highly Confidential
- **Restrictions:**
  - Requires on-premises data gateway
  - Gateway must be managed by IT
  - Database team must provision connection
  - Read-only unless approved
- **Approval Process:** Submit request to [Database Team Email]
- **Approval Timeline:** 15-20 business days
- **Contact:** [Database Team]

### Custom Connectors
- **Status:** ⚠️ RESTRICTED
- **Max Data Classification:** Depends on target system
- **Restrictions:**
  - Must go through API governance
  - Requires architecture review
  - Must be documented in API catalog
  - Must follow organization security standards
- **Approval Process:** Submit request to [Architecture Team]
- **Approval Timeline:** 15-30 business days
- **Contact:** [Enterprise Architecture]

## Third-Party SaaS Connectors

### Salesforce
- **Status:** [Choose: ✅/⚠️/🔄/❌]
- **Max Data Classification:** [Classification Level]
- **Restrictions:** [List restrictions]
- **Approval Process:** [Process details]
- **Contact:** [Contact info]

### ServiceNow
- **Status:** [Choose: ✅/⚠️/🔄/❌]
- **Max Data Classification:** [Classification Level]
- **Restrictions:** [List restrictions]
- **Approval Process:** [Process details]
- **Contact:** [Contact info]

[Add other SaaS connectors your organization uses...]

## AI and Advanced Connectors

### AI Builder
- **Status:** ✅ APPROVED
- **Max Data Classification:** Confidential
- **Restrictions:**
  - Must follow AI ethics guidelines
  - Data must stay in approved regions
  - Custom models require review
- **Approval Process:** Self-service for prebuilt models; review for custom
- **Contact:** [Power Platform COE]

### Copilot Studio
- **Status:** [Choose: ✅/⚠️/🔄/❌]
- **Max Data Classification:** [Classification Level]
- **Restrictions:** [List restrictions]
- **Approval Process:** [Process details]
- **Contact:** [Contact info]

## Blocked Connectors

### Example: Dropbox
- **Status:** ❌ BLOCKED
- **Reason:** Not approved for business use; data sovereignty concerns
- **Alternatives:** OneDrive for Business, SharePoint Online

### Example: Gmail
- **Status:** ❌ BLOCKED
- **Reason:** Personal email; use Outlook for organization email
- **Alternatives:** Outlook (Office 365)

### Example: Twitter/X
- **Status:** ❌ BLOCKED
- **Reason:** Social media connectors not approved
- **Alternatives:** Approved corporate communications channels

[Add other blocked connectors...]

## Connectors Under Review

### Example: Microsoft Planner
- **Status:** 🔄 IN REVIEW
- **Review Status:** Under evaluation by COE
- **Expected Decision:** [Date]
- **Interim Guidance:** Use approved alternatives (SharePoint tasks) until reviewed
- **Contact:** [Power Platform COE]

[Add connectors currently being reviewed...]

## Premium vs Standard Connectors

**Standard Connectors** (included with all licenses):
- Most Microsoft 365 connectors
- Many popular SaaS connectors
- See [Microsoft documentation](https://learn.microsoft.com/en-us/power-platform/admin/powerapps-flow-licensing-faq)

**Premium Connectors** (require Power Automate Premium or appropriate license):
- HTTP connector
- SQL Server
- Azure SQL
- Many enterprise connectors
- See our licensing guide at [Internal Link]

## Requesting New Connector Approval

To request approval for a connector not on this list:

1. **Check Microsoft Connector Documentation:**
   - [Power Automate Connectors](https://learn.microsoft.com/en-us/connectors/connector-reference/)
   - Verify it's a legitimate Microsoft connector

2. **Submit Request:**
   - Email: [Governance Email]
   - Include: Business justification, data classification, volume estimates
   - Template: [Link to Request Template]

3. **Approval Process:**
   - Governance team review (5 business days)
   - Security review if needed (10 business days)
   - Decision communicated via email

4. **Timeline:**
   - Standard connectors: 5-10 business days
   - Premium/sensitive connectors: 15-30 business days

## Data Classification Reference

Quick reference to data classifications (see data-classification-guide.md for details):

- **Public** - No restrictions; publicly available information
- **Internal** - Organization-wide; not for external sharing
- **Confidential** - Sensitive business information; need-to-know basis
- **Highly Confidential** - Critical data; strict access controls; regulatory requirements

## Related Documents

- [Data Classification Guide](data-classification-guide.md)
- [Compliance Requirements](compliance-requirements.md)
- [Systems Integration Guide](systems-integration.md)
- [Internal: Power Platform Governance Policy] - [Link]
- [Internal: API Catalog] - [Link]

## Change History

| Date | Change | Changed By |
|------|--------|------------|
| [Date] | Initial version | [Name] |
| [Date] | Added Azure connectors | [Name] |
| [Date] | Updated approval process | [Name] |

## Questions or Issues?

- **Governance Team:** [Email]
- **Power Platform COE:** [Email]
- **Architecture Team:** [Email]
- **Security Team:** [Email]

---

**Note:** This is a template. Customize it with your organization's actual connectors, approval processes, and contacts.
