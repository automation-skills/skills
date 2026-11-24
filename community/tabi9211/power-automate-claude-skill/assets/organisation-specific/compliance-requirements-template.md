# Compliance Requirements for Power Automate

**Organization:** [Your Organization Name]
**Last Updated:** [Date]
**Owner:** [Compliance/Legal Team/Contact]
**Related:** [Link to Organization Compliance Policy]

## Overview

This document outlines compliance requirements that must be met when developing Power Automate solutions. Compliance is non-negotiable and violations can result in:

- Legal consequences
- Financial penalties
- Reputational damage
- Project shutdowns
- Disciplinary action

## Applicable Regulations

Our organization must comply with:

- [ ] **GDPR** (General Data Protection Regulation) - if processing EU personal data
- [ ] **CCPA** (California Consumer Privacy Act) - if processing California resident data
- [ ] **PCI DSS** (Payment Card Industry Data Security Standard) - if processing payment data
- [ ] **HIPAA** (Health Insurance Portability and Accountability Act) - if processing health data
- [ ] **SOX** (Sarbanes-Oxley) - if processing financial data (public companies)
- [ ] **ISO 27001** - Information security management
- [ ] **Industry-specific regulations** - [List your industry regulations]

[Check the boxes that apply to your organization]

---

## GDPR Compliance (if applicable)

### Scope

GDPR applies when processing personal data of EU residents, regardless of where the organization is located.

**Personal Data includes:**
- Names, email addresses, phone numbers
- IP addresses, cookie identifiers
- Location data
- Biometric data
- Any data that can identify an individual

### GDPR Requirements for Power Automate

#### 1. Data Protection Impact Assessment (DPIA)

**When Required:**
- Processing high-risk personal data
- Large-scale systematic monitoring
- Automated decision-making with legal effects

**Process:**
1. Complete DPIA template: [Link to template]
2. Submit to Data Protection Officer: [Email]
3. Await approval before implementation
4. Review annually

**Template includes:**
- Description of processing
- Necessity and proportionality assessment
- Risk identification
- Mitigation measures

#### 2. Lawful Basis for Processing

All flows processing personal data must have a lawful basis:

- [ ] **Consent** - Individual has given clear consent
- [ ] **Contract** - Processing necessary for a contract
- [ ] **Legal Obligation** - Required by law
- [ ] **Vital Interests** - Protecting someone's life
- [ ] **Public Task** - Performing official functions
- [ ] **Legitimate Interests** - Legitimate business need (most common)

**Document the lawful basis in flow description.**

#### 3. Purpose Limitation

- Data must only be used for the documented purpose
- Cannot repurpose data without new legal basis
- Document purpose in flow description

**Example:**
```
✅ CORRECT:
Purpose: Process customer orders
Data used: Name, email, order details
Action: Create order in ERP, send confirmation email

❌ INCORRECT:
Purpose: Process customer orders
Action: Create order AND add email to marketing list
(Marketing requires separate consent)
```

#### 4. Data Minimization

- Only collect/process necessary data
- Don't request "nice to have" data
- Filter data as early as possible in flows

**Example:**
```
❌ BAD: Get all customer columns
✅ GOOD: Get only columns needed (name, email, order_id)
```

#### 5. Storage Limitation

- Define retention periods
- Implement automatic deletion
- Document in flow description

**Example Pattern:**
```
Flow: Cleanup Old Data
Trigger: Scheduled (monthly)
Actions:
1. Get records older than [retention period]
2. Secure deletion
3. Log deletion for audit
```

#### 6. Data Subject Rights

Implement processes to support:

- **Right to Access** - Export individual's data
- **Right to Rectification** - Correct inaccurate data
- **Right to Erasure** - Delete data ("right to be forgotten")
- **Right to Portability** - Provide data in machine-readable format
- **Right to Object** - Stop processing

**Implementation:**
- Create flows to handle data subject requests
- Document process: [Link to internal process]
- Response deadline: 30 days

#### 7. Security Measures

Required security controls:
- ✅ Encryption in transit (HTTPS/TLS)
- ✅ Encryption at rest (for Highly Confidential data)
- ✅ Access controls (service accounts, least privilege)
- ✅ Audit logging
- ✅ Regular security reviews

#### 8. Data Breach Notification

**Timeline:**
- Detect and assess breach
- Notify DPO within 24 hours: [DPO Email]
- Notify supervisory authority within 72 hours (if high risk)
- Notify affected individuals (if high risk to rights)

**Process:**
1. Stop the flow
2. Notify DPO immediately
3. Document incident
4. Follow incident response plan: [Link]

#### 9. Cross-Border Data Transfers

**Restriction:** Personal data of EU residents must stay in EU or approved countries

**Approved Transfer Mechanisms:**
- Adequacy decision (e.g., EEA, UK, Switzerland)
- Standard Contractual Clauses (SCCs)
- Binding Corporate Rules (BCRs)

**For Power Automate:**
- Verify connector data residency
- Use EU region for Dataverse
- Check third-party service locations

**Contact:** Data Protection Officer: [Email]

---

## PCI DSS Compliance (if applicable)

### Scope

PCI DSS applies when processing payment card information (credit/debit cards).

**Cardholder Data includes:**
- Card number (PAN)
- Cardholder name
- Expiration date
- Service code

**Sensitive Authentication Data (NEVER store):**
- CVV/CVC code
- PIN
- Full magnetic stripe data

### PCI DSS Requirements for Power Automate

#### 1. Do NOT Store Sensitive Authentication Data

❌ **NEVER store:**
- CVV/CVC codes
- PINs
- Full magnetic stripe data

**Even encrypted, even temporarily!**

#### 2. Protect Stored Cardholder Data

If you must store PAN (primary account number):
- ✅ Encrypt with strong encryption (AES-256)
- ✅ Store encryption keys in Azure Key Vault
- ✅ Mask PAN in logs (show only last 4 digits)
- ✅ Restrict access (need-to-know only)

**Preferred:** Don't store PAN - use tokenization from payment processor

#### 3. Use Approved Payment Processors

Only use approved payment service providers:
- [List your approved processors]
- [Contact: Finance Team Email]

Use their APIs/connectors - don't handle card data directly.

#### 4. Audit Logging

Log all access to cardholder data:
- Who accessed
- When
- What action
- Result

**Retention:** 1 year minimum (check your organization's policy)

#### 5. Network Security

- ✅ Use HTTPS for all payment API calls
- ✅ Verify SSL/TLS certificates
- ✅ Use approved network paths
- ✅ No card data over email

#### 6. Incident Response

Payment data breach = severe consequences

**Immediate Actions:**
1. Stop the flow
2. Notify security team: [Email]
3. Notify payment processor
4. Follow PCI incident response plan: [Link]

**Contact:** PCI Compliance Team: [Email]

---

## Industry-Specific Regulations

### [Your Industry Regulation 1]

[Document your industry-specific requirements]

**Scope:**
[When does this apply?]

**Requirements:**
- Requirement 1
- Requirement 2
- Requirement 3

**Implementation in Power Automate:**
[How to comply]

**Contact:** [Team Email]

---

### [Your Industry Regulation 2]

[Document additional regulations]

---

## General Security Requirements

### 1. Authentication and Authorization

**Requirements:**
- ✅ Use service accounts (never personal accounts)
- ✅ Implement least privilege
- ✅ Use managed identities where possible
- ✅ Multi-factor authentication for admin accounts
- ✅ Regular access reviews (quarterly)

### 2. Secret Management

**Requirements:**
- ✅ Store secrets in Azure Key Vault
- ✅ Rotate secrets regularly (quarterly minimum)
- ✅ Never hardcode credentials
- ✅ Never log secrets

### 3. Audit Logging

**Required Logging:**
- All data access
- All modifications
- All authentication events
- All errors and exceptions

**Log Retention:**
- Standard: 1 year
- Compliance-related: [Check your policy, often 7 years]

**SIEM Integration:**
- Forward logs to: [Your SIEM]
- Real-time alerting for: [Define critical events]

### 4. Change Management

**Requirements:**
- ✅ Document all changes
- ✅ Peer review before production
- ✅ Test in non-production environment
- ✅ Approval from [Role] before production deployment
- ✅ Rollback plan documented

**Process:**
1. Create change ticket: [Link to system]
2. Peer review
3. Test in dev/test environment
4. Get approval
5. Deploy to production
6. Verify and monitor

### 5. Data Loss Prevention (DLP)

**Power Platform DLP Policies:**
- [List your organization's DLP policies]
- [Link to DLP policy documentation]

**Enforcement:**
- DLP policies are enforced at environment level
- Flows violating DLP will not save/run
- Contact Power Platform COE for exceptions: [Email]

---

## Compliance Checklist for New Flows

Use this checklist for every new flow:

### Data Assessment
- [ ] Data classification determined
- [ ] Personal data identified (if any)
- [ ] Sensitive data identified (if any)
- [ ] DPIA completed (if required)
- [ ] Lawful basis documented (if processing personal data)

### Security Controls
- [ ] Service account configured
- [ ] Secrets stored in Key Vault
- [ ] Encryption in transit (HTTPS)
- [ ] Encryption at rest (if required)
- [ ] Access controls configured
- [ ] Audit logging enabled

### Compliance Requirements
- [ ] GDPR requirements met (if applicable)
- [ ] PCI requirements met (if applicable)
- [ ] Industry-specific requirements met
- [ ] Data retention period defined
- [ ] Deletion process implemented

### Documentation
- [ ] Flow description includes purpose
- [ ] Data processed documented
- [ ] Compliance considerations documented
- [ ] Incident response plan identified

### Approval and Deployment
- [ ] Peer review completed
- [ ] Testing completed
- [ ] Change ticket created
- [ ] Approval obtained
- [ ] Rollback plan documented

### Post-Deployment
- [ ] Monitoring configured
- [ ] Alerts configured
- [ ] Documentation updated
- [ ] Team trained (if needed)

---

## Incident Response

### Types of Incidents

1. **Data Breach** - Unauthorized access or disclosure
2. **Data Loss** - Accidental deletion or corruption
3. **Compliance Violation** - Breach of requirements
4. **Security Incident** - Attempted or successful attack

### Response Process

**Step 1: Identify and Contain**
- Stop the flow immediately
- Assess scope of incident
- Document initial findings

**Step 2: Notify**
- Security Team: [Email] - Immediate
- Compliance Team: [Email] - Within 1 hour
- DPO: [Email] - Within 24 hours (if personal data)
- Management: [Email] - Per escalation policy

**Step 3: Investigate**
- Review audit logs
- Identify root cause
- Assess impact
- Document findings

**Step 4: Remediate**
- Fix the issue
- Test the fix
- Implement preventive measures

**Step 5: Report**
- Internal incident report
- Regulatory notification (if required)
- Affected individuals notification (if required)

**Step 6: Review and Learn**
- Post-incident review
- Update procedures
- Train team

### Emergency Contacts

- **Security Operations Center:** [Phone/Email]
- **Data Protection Officer:** [Phone/Email]
- **Compliance Team:** [Phone/Email]
- **Legal Team:** [Phone/Email]
- **Management:** [Phone/Email]

---

## Training and Awareness

### Required Training

All Power Automate developers must complete:
- [ ] GDPR awareness training (if applicable)
- [ ] Security awareness training
- [ ] Compliance training
- [ ] Power Platform governance training

**Training links:** [Link to training platform]
**Frequency:** Annual refresher required

### Staying Current

- Subscribe to compliance updates: [Link]
- Attend quarterly compliance reviews
- Review this document regularly (quarterly)

---

## Related Documents

- [Data Classification Guide](data-classification-guide.md)
- [Approved Connectors List](approved-connectors.md)
- [Systems Integration Guide](systems-integration.md)
- [Internal: Organization Compliance Policy] - [Link]
- [Internal: Incident Response Plan] - [Link]
- [Internal: GDPR Compliance Guide] - [Link]

## Getting Help

**Compliance Questions:**
- Compliance Team: [Email]
- Data Protection Officer: [Email]
- Legal Team: [Email]

**Technical Questions:**
- Security Team: [Email]
- Power Platform COE: [Email]

## Change History

| Date | Change | Changed By |
|------|--------|------------|
| [Date] | Initial version | [Name] |

---

**Note:** This is a template. Customize it with your organization's actual compliance requirements, regulations, processes, and contacts. Consult with your legal and compliance teams before finalizing.
