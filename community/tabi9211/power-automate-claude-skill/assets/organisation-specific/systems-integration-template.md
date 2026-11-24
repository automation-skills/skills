# Internal Systems Integration Guide

**Organization:** [Your Organization Name]
**Last Updated:** [Date]
**Owner:** [Enterprise Architecture Team/Contact]
**Related:** [Link to API Catalog or Integration Standards]

## Overview

This guide documents integration patterns for connecting Power Automate to our organization's internal systems and APIs. Use these proven patterns to ensure consistent, secure, and maintainable integrations.

## Internal Systems Inventory

### Enterprise Applications

| System | Purpose | Integration Method | Authentication | Contact |
|--------|---------|-------------------|----------------|---------|
| HR System | Employee management | REST API | OAuth 2.0 | [HR IT Contact] |
| CRM | Customer relationship | Native Connector | Service Account | [CRM Team] |
| ERP | Financial/Operations | REST API | Certificate | [ERP Team] |
| Document Management | Document storage | SOAP API | API Key | [DMS Team] |
| [Add your systems...] | | | | |

## Common Integration Patterns

### Pattern 1: REST API with OAuth 2.0

**Use Cases:**
- Modern cloud-based internal systems
- APIs that support OAuth 2.0
- Example: Internal HR system, custom APIs

**Implementation:**

**Step 1: Register Application**
1. Register Power Automate as OAuth client in target system
2. Obtain Client ID and Client Secret
3. Store credentials in Azure Key Vault

**Step 2: Configure HTTP Connector**
```
Action: HTTP with Azure AD
Method: GET/POST/PUT (as needed)
URI: https://your-internal-api.company.com/api/endpoint
Authentication: OAuth 2.0

OAuth 2.0 Settings:
- Authority: https://login.microsoftonline.com/[tenant-id]
- Token Audience: [Resource ID]
- Client ID: [From Key Vault]
- Credential Type: Secret
- Secret: [From Key Vault]
```

**Step 3: Handle Token Refresh**
- Use managed connections (automatic refresh)
- Or implement manual refresh logic

**Example Flow:**
```
1. Initialize variables
   - apiBaseUrl
   - resourceId

2. HTTP with Azure AD - Get Data
   - URI: @{variables('apiBaseUrl')}/employees
   - Authentication: OAuth 2.0 (configured above)

3. Parse JSON
   - Content: @{body('HTTP_with_Azure_AD')}
   - Schema: [Employee schema]

4. Apply to each employee
   - Process data...
```

**Error Handling:**
- 401 Unauthorized: Check token refresh
- 403 Forbidden: Check permissions
- 429 Too Many Requests: Implement retry with exponential backoff
- 500 Server Error: Alert and retry

---

### Pattern 2: Certificate-Based Authentication

**Use Cases:**
- Legacy enterprise systems
- High-security requirements
- Systems that don't support OAuth

**Implementation:**

**Step 1: Certificate Setup**
1. Obtain client certificate from PKI team
2. Upload certificate to Azure Key Vault
3. Grant Power Automate managed identity access to certificate

**Step 2: Configure HTTP Connector**
```
Action: HTTP
Method: GET/POST/PUT
URI: https://your-internal-api.company.com/api/endpoint
Authentication: Client Certificate

Certificate Settings:
- Pfx File: [From Key Vault]
- Password: [From Key Vault if needed]
```

**Example Flow:**
```
1. Get Certificate from Key Vault
   - Use Azure Key Vault connector
   - Secret name: "ErpClientCertificate"

2. HTTP - Call ERP API
   - URI: https://erp-api.company.com/api/orders
   - Method: POST
   - Authentication: Client Certificate
   - Certificate: @{body('Get_Certificate')?['value']}

3. Parse response
4. Handle errors
```

---

### Pattern 3: API Key Authentication

**Use Cases:**
- Simple internal APIs
- Legacy systems
- Low-security requirements (Internal classification or higher)

**Implementation:**

**Step 1: Store API Key Securely**
```
Never hardcode API keys!
- Store in Azure Key Vault
- Reference via environment variables
```

**Step 2: Configure HTTP Connector**
```
Action: HTTP
Method: GET
URI: https://internal-api.company.com/api/endpoint
Headers:
  - X-API-Key: [From Key Vault or Environment Variable]
```

**Example Flow:**
```
1. Get API Key from Key Vault

2. HTTP - Call API
   - URI: https://dms-api.company.com/documents
   - Headers:
     - X-API-Key: @{body('Get_API_Key')?['value']}

3. Parse response
```

**Security Considerations:**
- ⚠️ API Keys should only be used for Internal classification or lower
- ⚠️ Rotate keys regularly (quarterly)
- ⚠️ Never log API keys
- ⚠️ Use HTTPS only

---

### Pattern 4: Service Account with Windows Authentication

**Use Cases:**
- On-premises systems
- Active Directory integrated systems
- SQL Server on-premises

**Implementation:**

**Step 1: Set Up Data Gateway**
```
1. Install on-premises data gateway
2. Configure gateway with service account
3. Grant service account minimum permissions
```

**Step 2: Configure Connector**
```
Action: SQL Server (on-premises)
Server: sql-server.company.local
Database: ProductionDB
Authentication: Windows
Gateway: [Your Gateway Name]
```

**Example Flow:**
```
1. Get rows (SQL Server)
   - Query: SELECT * FROM Orders WHERE Status = 'Pending'

2. Apply to each order
   - Process order data...

3. Update row (SQL Server)
   - Set Status = 'Processed'
```

**Security Considerations:**
- Use dedicated service account
- Minimum permissions (read-only when possible)
- Enable audit logging
- Regular access reviews

---

## System-Specific Integration Guides

### HR System Integration

**System Name:** [Your HR System Name]
**Base URL:** https://hr-api.company.com
**Authentication:** OAuth 2.0
**Rate Limits:** 100 requests/minute

**Common Use Cases:**
1. **Employee Onboarding**
   - Trigger: New employee in HR system
   - Actions: Create accounts, assign licenses, send welcome email

2. **Employee Offboarding**
   - Trigger: Employee status = "Terminated"
   - Actions: Revoke access, backup data, notify managers

**Example Endpoints:**

**Get Employee by ID:**
```http
GET /api/v1/employees/{employeeId}
Authorization: Bearer {token}

Response:
{
  "employeeId": "12345",
  "firstName": "John",
  "lastName": "Doe",
  "email": "john.doe@company.com",
  "department": "Engineering",
  "manager": "67890"
}
```

**Update Employee:**
```http
PUT /api/v1/employees/{employeeId}
Authorization: Bearer {token}
Content-Type: application/json

{
  "department": "Sales"
}
```

**Error Codes:**
- 400: Invalid request
- 401: Unauthorized
- 404: Employee not found
- 409: Conflict (duplicate)

**Rate Limiting:**
- Header: X-RateLimit-Remaining
- Implement exponential backoff on 429 responses

**Contact:** [HR IT Team Email]

---

### CRM Integration

**System Name:** [Your CRM System Name, e.g., Dynamics 365, Salesforce]
**Integration Method:** [Native Connector / API]
**Authentication:** [Service Account / OAuth]

**Common Use Cases:**
1. **Lead to Opportunity**
   - Trigger: Lead qualified
   - Actions: Create opportunity, assign to sales rep, notify

2. **Customer 360 View**
   - Trigger: Customer inquiry
   - Actions: Aggregate data from multiple systems, present in CRM

**Example Flow Pattern:**
```
[Provide specific pattern for your CRM]
```

**Contact:** [CRM Team Email]

---

### ERP Integration

**System Name:** [Your ERP System Name, e.g., SAP, Oracle, Dynamics 365]
**Base URL:** https://erp-api.company.com
**Authentication:** Certificate-based

**Common Use Cases:**
1. **Order Processing**
   - Trigger: New order from e-commerce
   - Actions: Validate inventory, create order, update status

2. **Inventory Sync**
   - Trigger: Schedule (hourly)
   - Actions: Sync inventory levels to other systems

**Example Flow Pattern:**
```
[Provide specific pattern for your ERP]
```

**Contact:** [ERP Team Email]

---

## Cross-System Integration Patterns

### Pattern: Data Synchronization

**Use Case:** Keep data consistent across multiple systems

**Implementation:**
```
1. Trigger: Scheduled (e.g., every 15 minutes)

2. Get source data (System A)
   - Query: Records modified since last sync

3. Transform data
   - Map fields from System A to System B
   - Apply business rules

4. Upsert to destination (System B)
   - Check if record exists
   - Update or create

5. Update sync timestamp
6. Log sync results
```

**Best Practices:**
- Use incremental sync (modified date filtering)
- Implement idempotency (same operation can run multiple times)
- Handle conflicts (last-write-wins or manual review)
- Log all sync operations

---

### Pattern: Event-Driven Integration

**Use Case:** Real-time response to business events

**Implementation:**
```
1. Trigger: System A webhook or event

2. Validate payload
   - Check signature
   - Validate schema

3. Transform data

4. Call System B API

5. Update System A with result

6. Error handling and retry
```

**Best Practices:**
- Validate webhook signatures
- Use queuing for reliability (e.g., Azure Queue)
- Implement idempotency
- Monitor event processing times

---

## Security Best Practices

### Secret Management

✅ **DO:**
- Store credentials in Azure Key Vault
- Use managed identities where possible
- Reference secrets via environment variables
- Rotate credentials regularly
- Use certificate authentication for high-security

❌ **DON'T:**
- Hardcode credentials in flows
- Share credentials across flows
- Store credentials in variables without encryption
- Use personal accounts

### Network Security

✅ **DO:**
- Use HTTPS for all API calls
- Verify SSL certificates
- Use approved network paths
- Document external endpoints

❌ **DON'T:**
- Use HTTP for sensitive data
- Bypass SSL verification
- Open unnecessary firewall rules

### API Security

✅ **DO:**
- Implement rate limiting in flows
- Use least-privilege service accounts
- Log all API calls
- Monitor for unusual patterns

❌ **DON'T:**
- Expose API keys in logs or responses
- Use admin accounts for automation
- Ignore rate limits

## Troubleshooting Common Issues

### Issue: 401 Unauthorized

**Causes:**
- Expired token
- Invalid credentials
- Wrong authentication method

**Solutions:**
1. Check token expiration
2. Verify credentials in Key Vault
3. Confirm authentication method with API team
4. Check service account status

### Issue: 429 Too Many Requests

**Causes:**
- Exceeded rate limits
- Too many parallel requests

**Solutions:**
1. Implement exponential backoff
2. Reduce request frequency
3. Use batching when available
4. Contact API team for rate limit increase

### Issue: 500 Internal Server Error

**Causes:**
- Backend system issue
- Invalid request payload
- Timeout

**Solutions:**
1. Check backend system status
2. Validate request payload against API spec
3. Implement retry with backoff
4. Contact API team if persists

### Issue: Certificate Errors

**Causes:**
- Expired certificate
- Wrong certificate
- Missing permissions

**Solutions:**
1. Check certificate expiration
2. Verify certificate in Key Vault
3. Confirm managed identity has Key Vault access
4. Contact PKI team

## Related Documents

- [Approved Connectors List](approved-connectors.md)
- [Data Classification Guide](data-classification-guide.md)
- [Compliance Requirements](compliance-requirements.md)
- [Internal: API Catalog] - [Link]
- [Internal: Network Security Policy] - [Link]

## Getting Help

**API Questions:**
- Enterprise Architecture: [Email]
- API Catalog Team: [Email]

**System-Specific Questions:**
- HR System: [HR IT Email]
- CRM: [CRM Team Email]
- ERP: [ERP Team Email]

**Security Questions:**
- Security Team: [Email]
- PKI Team: [Email]

## Change History

| Date | Change | Changed By |
|------|--------|------------|
| [Date] | Initial version | [Name] |

---

**Note:** This is a template. Customize it with your organization's actual systems, APIs, authentication methods, and contacts.
