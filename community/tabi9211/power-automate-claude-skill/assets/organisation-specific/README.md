# Organisation-Specific Assets

This directory is designed for **your organization's specific** Power Automate context, including approved connectors, data classification systems, internal integration patterns, and compliance requirements.

## Purpose

The Power Automate Expert Skill can provide much more relevant and compliant guidance when it understands your organization's specific:

- Approved and restricted connectors
- Data classification system
- Internal systems and APIs
- Compliance requirements (GDPR, industry regulations)
- Proven flow patterns and templates

## Setup Instructions

### Step 1: Review Template Files

This directory contains template files that demonstrate the structure and type of information to include:

- `approved-connectors-template.md` - Template for your connector approval list
- `data-classification-guide-template.md` - Template for data classification
- `systems-integration-template.md` - Template for internal systems
- `compliance-requirements-template.md` - Template for compliance needs

### Step 2: Create Your Organization's Files

Copy the template files and customize them with your organization's information:

```bash
# Example for approved connectors
cp approved-connectors-template.md approved-connectors.md
# Now edit approved-connectors.md with your organization's data
```

### Step 3: Add Your Patterns and Examples

Add your organization's proven flow patterns:

- `flow-patterns/` - Add reusable flow patterns as markdown files
- `examples/` - Add real-world implementation examples (anonymized)

## Files to Create

### Critical Files

1. **approved-connectors.md** (from template)
   - List all connectors with approval status
   - Include restrictions and approval contacts
   - Document licensing requirements

2. **data-classification-guide.md** (from template)
   - Your organization's data classification levels
   - Automation implications for each level
   - Connector restrictions by classification

3. **compliance-requirements.md** (from template)
   - GDPR requirements
   - Industry-specific regulations
   - Security controls and audit logging requirements

4. **systems-integration.md** (from template)
   - Internal APIs and systems
   - Authentication patterns
   - Integration examples

### Optional Files

5. **flow-patterns/** directory
   - Reusable patterns specific to your organization
   - Common approval workflows
   - Standard error handling approaches
   - Organization-specific best practices

6. **examples/** directory
   - Real implementations (anonymized)
   - Success stories
   - Lessons learned

## Important Security Note

⚠️ **Do NOT commit actual organization data to public repositories!**

The `.gitignore` file is configured to ignore the populated files while keeping the templates:

```
# These will be ignored:
approved-connectors.md
data-classification-guide.md
systems-integration.md
compliance-requirements.md

# These templates are committed:
approved-connectors-template.md
data-classification-guide-template.md
systems-integration-template.md
compliance-requirements-template.md
```

### For Internal Use

If you're using this skill within your organization's private repository:

1. Remove the relevant lines from `.gitignore`
2. Ensure the repository has appropriate access controls
3. Follow your organization's code review process
4. Regularly audit who has access

## How the Skill Uses This Information

When you ask the Power Automate Expert Skill questions, it will:

1. **Check connector approval** before suggesting any connector
2. **Verify data classification** compatibility
3. **Apply compliance requirements** to all recommendations
4. **Use your organization's patterns** when available
5. **Reference internal systems** with correct authentication

### Example Interactions

**Without organization context:**
```
User: "How do I connect to our HR system?"
Skill: "I can help with general API integration patterns..."
```

**With organization context:**
```
User: "How do I connect to our HR system?"
Skill: "Based on your systems-integration.md, your HR system uses OAuth 2.0
       with the following endpoints... Here's the proven pattern from your
       organization..."
```

## Maintaining Your Organization's Context

### Regular Updates

- **Quarterly**: Review approved connectors list
- **Monthly**: Update compliance requirements if regulations change
- **As needed**: Add new integration patterns as you develop them
- **Annually**: Full audit of all organizational context

### Version Control

Consider maintaining your organization's files in a separate, private repository and symlinking them:

```bash
# In your private org repo
git clone https://private-repo/power-automate-org-context.git

# In the skill directory
cd ~/.claude/skills/power-automate-expert/assets/organisation-specific
ln -s /path/to/private-repo/approved-connectors.md
ln -s /path/to/private-repo/data-classification-guide.md
# etc.
```

## Templates Reference

### approved-connectors-template.md

Template for documenting connector approval status with:
- Approval status (✅ APPROVED, ⚠️ RESTRICTED, 🔄 IN REVIEW, ❌ BLOCKED)
- Data classification restrictions
- Approval process and contacts
- Special considerations

### data-classification-guide-template.md

Template for your data classification system:
- Classification levels (e.g., Public, Internal, Confidential, Highly Confidential)
- Automation implications per level
- Connector restrictions
- Handling requirements

### systems-integration-template.md

Template for internal systems:
- System inventory
- API patterns
- Authentication approaches
- Code examples

### compliance-requirements-template.md

Template for compliance needs:
- GDPR/privacy requirements
- Industry regulations
- Security controls
- Audit logging requirements

## Getting Help

If you need help customizing these templates for your organization:

1. Review the examples in each template file
2. Consult your organization's governance team
3. Check Microsoft's compliance documentation
4. Open a discussion on GitHub for general questions

## Example Organizations

Some anonymized examples of what different organizations include:

**Public Sector Organization:**
- Strict connector approval process
- High data classification requirements
- GDPR + sector-specific regulations
- Extensive audit logging needs

**Enterprise Corporation:**
- Approved connector list with procurement contacts
- Three-tier data classification
- Internal SAP/Salesforce integration patterns
- Global data residency requirements

**SMB/Mid-Market:**
- Simplified connector list (approved/not approved)
- Basic data classification
- Common SaaS integrations
- Essential compliance only

---

**Ready to customize?** Start with `approved-connectors-template.md` and work through each template, adapting them to your organization's needs.
