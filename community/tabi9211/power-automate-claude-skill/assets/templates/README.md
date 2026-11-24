# Power Automate Templates

This directory contains reusable templates and assets to accelerate Power Automate development with consistent patterns.

## Directory Structure

```
templates/
├── deployment-settings/    # Environment-specific deployment configurations
├── pipelines/             # CI/CD pipeline templates
├── flows/                 # Common flow templates
├── scripts/              # PowerShell and automation scripts
└── checklists/           # Pre-deployment and quality checklists
```

## Available Templates

### Deployment Settings

Environment-specific deployment configurations for ALM (Application Lifecycle Management).

**Files to add:**
- `dev-deployment-settings.json` - Development environment settings
- `test-deployment-settings.json` - Test environment settings
- `prod-deployment-settings.json` - Production environment settings

**Usage:**
- Configure connection references per environment
- Set environment variables per environment
- Enable/disable flows per environment

**Documentation:** [Microsoft Learn - Deployment Settings Files](https://learn.microsoft.com/en-us/power-platform/alm/conn-ref-env-variables-build-tools)

---

### Pipeline Templates

CI/CD pipeline templates for Azure DevOps or GitHub Actions.

**Files to add:**
- `azure-devops-pipeline.yml` - Azure DevOps pipeline
- `github-actions-workflow.yml` - GitHub Actions workflow
- `export-solution.yml` - Solution export automation
- `import-solution.yml` - Solution import automation

**Usage:**
- Automated solution export from dev
- Automated testing (if applicable)
- Automated deployment to test/prod
- Version management

**Prerequisites:**
- Power Platform Build Tools
- Service principal or service account
- Environment URLs and credentials

---

### Flow Templates

Common flow templates ready for customization.

**Suggested templates to add:**

1. **approval-workflow-template.json** - Standard approval workflow
   - Single approver
   - Multi-stage approval
   - Conditional approval based on amount/type

2. **error-notification-template.json** - Standardized error notifications
   - Format error messages
   - Notify appropriate team
   - Include context and troubleshooting info

3. **data-sync-template.json** - Data synchronization pattern
   - Incremental sync
   - Error handling
   - Conflict resolution

4. **scheduled-cleanup-template.json** - Data cleanup/maintenance
   - Identify old/expired data
   - Archive or delete
   - Log actions

5. **http-integration-template.json** - REST API integration
   - Authentication
   - Error handling
   - Retry logic
   - Rate limiting

**Usage:**
1. Export as template or copy JSON
2. Customize for specific use case
3. Replace placeholders
4. Test in dev environment
5. Deploy through proper channels

---

### Scripts

PowerShell and other automation scripts for Power Automate management.

**Suggested scripts to add:**

1. **Export-PowerAutomateSolution.ps1**
   - Export solution from environment
   - Version management
   - Backup creation

2. **Import-PowerAutomateSolution.ps1**
   - Import solution to environment
   - Connection reference mapping
   - Environment variable setting

3. **Update-ConnectionReferences.ps1**
   - Bulk update connection references
   - Environment-specific configuration

4. **Get-FlowRunHistory.ps1**
   - Query flow run history
   - Export for analysis
   - Error reporting

5. **Enable-DisableFlows.ps1**
   - Bulk enable/disable flows
   - Maintenance windows
   - Deployment management

**Prerequisites:**
- PowerShell 7+
- Power Platform CLI or PowerShell modules
- Appropriate permissions

**Installation:**
```powershell
# Install Power Platform CLI
winget install Microsoft.PowerPlatformCLI

# Or install PowerShell modules
Install-Module -Name Microsoft.PowerApps.Administration.PowerShell
Install-Module -Name Microsoft.PowerApps.PowerShell -AllowClobber
```

---

### Checklists

Quality assurance and deployment checklists.

**Suggested checklists to add:**

1. **pre-deployment-checklist.md**
   - Solution validation
   - Testing completed
   - Documentation updated
   - Approvals obtained
   - Rollback plan ready

2. **security-review-checklist.md**
   - Authentication reviewed
   - Secrets in Key Vault
   - Least privilege applied
   - Audit logging enabled
   - Compliance requirements met

3. **performance-review-checklist.md**
   - Concurrency optimized
   - Loops minimized
   - Caching implemented
   - Rate limits considered
   - Scalability tested

4. **code-review-checklist.md**
   - Naming conventions followed
   - Error handling implemented
   - Comments added
   - Hard-coded values removed
   - Connection references used

5. **go-live-checklist.md**
   - Production deployment steps
   - Monitoring configured
   - Alerts set up
   - Team trained
   - Documentation finalized

---

## Contributing Templates

Have a great template to share? Contributions are welcome!

### Template Contribution Guidelines

1. **Generalize the template**
   - Remove organization-specific information
   - Use placeholders for variables
   - Add clear instructions

2. **Document the template**
   - Purpose and use case
   - Prerequisites
   - Customization instructions
   - Example usage

3. **Test the template**
   - Verify it works in clean environment
   - Confirm instructions are clear

4. **Submit a pull request**
   - Follow [CONTRIBUTING.md](../../CONTRIBUTING.md)
   - Include description of template value

### Template Format

**For Flow Templates:**
- Export as solution or JSON
- Include README with template explaining:
  - What it does
  - When to use it
  - How to customize it
  - Required connections

**For Scripts:**
- Include comment-based help
- Use approved verbs
- Handle errors gracefully
- Support -WhatIf where applicable

**For Checklists:**
- Markdown format
- Checkbox items
- Clear, actionable items
- Link to relevant documentation

---

## Example: Using a Flow Template

### Step 1: Get the Template
```powershell
# Clone or download the repository
git clone https://github.com/ktg0066/power-automate-claude-skill.git
cd power-automate-claude-skill/assets/templates/flows
```

### Step 2: Review the Template
- Open the template file
- Read the included README or comments
- Understand the purpose and structure

### Step 3: Customize
- Replace placeholders with your values
- Adjust logic for your use case
- Update connection references

### Step 4: Import
- Create solution in dev environment
- Import flow
- Configure connections
- Test thoroughly

### Step 5: Deploy
- Export solution
- Use deployment settings for environment variables
- Import to test/prod
- Monitor and verify

---

## Example: Using a Deployment Script

```powershell
# Connect to Power Platform
Add-PowerAppsAccount

# Export solution from dev
.\scripts\Export-PowerAutomateSolution.ps1 `
    -EnvironmentUrl "https://org-dev.crm.dynamics.com" `
    -SolutionName "MyAutomationSolution" `
    -OutputPath ".\exports"

# Import solution to test
.\scripts\Import-PowerAutomateSolution.ps1 `
    -EnvironmentUrl "https://org-test.crm.dynamics.com" `
    -SolutionPath ".\exports\MyAutomationSolution.zip" `
    -DeploymentSettingsPath ".\deployment-settings\test-deployment-settings.json"
```

---

## Related Documents

- [Solution Management Reference](../../references/solution-management.md)
- [Enterprise Adoption Guide](../../references/enterprise-adoption.md)
- [Contributing Guidelines](../../CONTRIBUTING.md)

---

**Start building your template library today!** Even simple templates can save hours of development time and ensure consistency across your organization.
