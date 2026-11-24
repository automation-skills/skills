# Power Automate Expert Skill for Claude Code

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Version](https://img.shields.io/badge/version-6.0.0-blue.svg)](https://github.com/ktg0066/power-automate-claude-skill/releases)
[![Claude Code](https://img.shields.io/badge/Claude-Code-5C3EE8.svg)](https://claude.ai/code)

A comprehensive Claude Code skill that provides enterprise-grade Power Automate expertise for designing, building, debugging, and optimizing cloud flows and desktop flows with deep integration knowledge for Dataverse, Copilot Studio, and AI Builder.

## What This Skill Does

Transform Claude Code into a Power Automate expert consultant that can help you with every aspect of the Power Automation Platform, from initial architecture design to production deployment and optimization.

This skill provides:

- **Flow Generation**: Complete flow architectures with error handling and optimization
- **Debugging Expertise**: Systematic troubleshooting with common gotchas and resolutions
- **Performance Optimization**: Enterprise-grade strategies for scalability and efficiency
- **Connector Mastery**: Deep knowledge of 1000+ connectors with detailed implementation guidance
- **Licensing Guidance**: Critical insights into Power Automate Premium vs Copilot Studio licensing
- **Enterprise Adoption**: HEAT methodology, Center of Excellence setup, and governance frameworks
- **ALM Best Practices**: Solution management with connection references and environment variables
- **Organisation Context**: Framework for applying your specific approved connectors, data classification, and compliance requirements

## Who This Is For

- **Power Automate Developers**: Building and optimizing cloud flows and desktop flows
- **Microsoft Consultants**: Implementing enterprise automation solutions
- **Solution Architects**: Designing scalable automation architectures
- **IT Professionals**: Managing Power Platform governance and adoption
- **Business Analysts**: Understanding automation capabilities and requirements
- **Enterprise Users**: Working with organization-specific connectors and compliance requirements

## Key Features

### Comprehensive Knowledge Base

The skill covers the complete Power Automate ecosystem:

- **Cloud Flows**: Automated, instant, and scheduled flows with advanced patterns
- **Desktop Flows (RPA)**: Attended and unattended robotic process automation
- **Connectors**: 1000+ connectors including SharePoint, Teams, Outlook, Dataverse, HTTP, Azure services
- **Expressions & Functions**: Expert guidance on Power Automate expression language
- **Dataverse Integration**: Seamless work with Microsoft Dataverse
- **Copilot Studio**: Understanding when and how to integrate with conversational AI
- **AI Builder**: Prebuilt and custom AI models for intelligent automation
- **Process Mining**: Discover inefficiencies and automation opportunities
- **Work Queues**: Advanced workload management for complex automation processes
- **Machine Groups**: Traditional and hosted machine groups for enterprise scale

### Critical Licensing Guidance

Includes comprehensive licensing documentation that clarifies the often-confusing differences between:
- **Power Automate Premium** ($15/user/month): Full standalone automation capabilities
- **Copilot Studio**: LIMITED Power Automate rights (agent context only)

This knowledge helps you make informed procurement decisions and understand architecture constraints.

### Organisation-Specific Customization

The skill includes a framework for adding your organization's specific context:
- Approved connectors list with status tracking
- Data classification guide and automation implications
- Internal systems integration patterns
- Compliance requirements (GDPR, security, industry-specific)
- Reusable flow patterns and templates

## Installation

### Prerequisites

- [Claude Code](https://claude.ai/code) installed and configured
- Basic familiarity with Claude Code skills

### Installation Steps

1. **Clone this repository:**
   ```bash
   git clone https://github.com/ktg0066/power-automate-claude-skill.git
   ```

2. **Locate your Claude Code skills directory:**
   - **macOS/Linux**: `~/.claude/skills/`
   - **Windows**: `%USERPROFILE%\.claude\skills\`

3. **Copy the skill folder:**
   ```bash
   # macOS/Linux
   cp -r power-automate-claude-skill ~/.claude/skills/power-automate-expert

   # Windows
   xcopy power-automate-claude-skill %USERPROFILE%\.claude\skills\power-automate-expert /E /I
   ```

4. **Restart Claude Code** to load the skill

5. **Verify installation:**
   ```bash
   # In Claude Code, ask:
   "What Power Automate capabilities do you have?"
   ```

## Repository Structure

```
power-automate-claude-skill/
├── SKILL.md                           # Main skill file (27KB)
├── licensing-comparison.md            # Licensing guide
├── README.md                          # This file
├── LICENSE                            # MIT License
├── CONTRIBUTING.md                    # Contribution guidelines
├── CODE_OF_CONDUCT.md                # Community guidelines
├── CHANGELOG.md                       # Version history
├── .gitignore                        # Git ignore rules
│
├── references/                        # Reference documentation
│   ├── README.md                     # Reference docs overview
│   ├── connectors-reference.md       # Comprehensive connector catalog
│   ├── connector-decision-matrix.md  # Connector selection framework
│   ├── expressions-guide.md          # Expression language guide
│   ├── debugging-guide.md            # Systematic debugging approaches
│   ├── performance-optimization.md   # Performance strategies
│   ├── work-queues.md               # Work queue patterns
│   ├── best-practices.md            # Microsoft recommended patterns
│   ├── dataverse-integration.md      # Dataverse patterns
│   ├── copilot-studio-patterns.md    # Copilot Studio integration
│   ├── ai-builder-integration.md     # AI Builder guide
│   ├── process-mining.md            # Process optimization
│   ├── enterprise-adoption.md        # HEAT methodology & CoE
│   ├── solution-management.md        # ALM best practices
│   └── machine-groups.md            # Scaling strategies
│
├── assets/                           # Organisation-specific assets
│   ├── organisation-specific/        # Your organization's context
│   │   ├── README.md                # Setup instructions
│   │   ├── approved-connectors.md   # Connector approval list (template)
│   │   ├── data-classification-guide.md  # Data classification (template)
│   │   ├── systems-integration.md   # Internal systems (template)
│   │   ├── compliance-requirements.md    # Compliance (template)
│   │   ├── flow-patterns/           # Reusable patterns (your examples)
│   │   └── examples/                # Implementation examples (your examples)
│   │
│   └── templates/                    # Reusable templates
│       ├── deployment-settings/      # Environment configurations
│       ├── pipelines/               # CI/CD pipeline templates
│       ├── flows/                   # Common flow templates
│       ├── scripts/                 # Automation scripts
│       └── checklists/              # Pre-deployment checklists
│
└── .github/                          # GitHub templates
    ├── ISSUE_TEMPLATE/
    │   ├── bug_report.md
    │   ├── feature_request.md
    │   └── documentation.md
    └── PULL_REQUEST_TEMPLATE.md
```

## Customizing for Your Organisation

### Step 1: Review Organisation-Specific Assets

Navigate to `assets/organisation-specific/` and review the template files:

1. **approved-connectors.md**: List your approved connectors with status
2. **data-classification-guide.md**: Document your data classification system
3. **systems-integration.md**: Document internal API patterns
4. **compliance-requirements.md**: Document GDPR, security, regulatory requirements

### Step 2: Add Your Flow Patterns

Add your organization's proven flow patterns to:
- `assets/organisation-specific/flow-patterns/`
- `assets/organisation-specific/examples/`

### Step 3: Customize Templates

Modify templates in `assets/templates/` to match your:
- Deployment settings structure
- CI/CD pipeline configurations
- Standard flow templates

### Step 4: Keep References Updated

The `references/` folder contains general Power Automate documentation. You can:
- Add organization-specific notes
- Link to internal documentation
- Customize examples for your context

## Usage Examples

### Example Questions to Ask

#### General Power Automate:
- "Help me create a flow that processes new SharePoint documents and sends approval emails"
- "What's the best way to handle errors in a flow with multiple API calls?"
- "How do I optimize a flow that's timing out with large datasets?"
- "Explain the difference between Apply to each and Select actions"

#### Licensing and Architecture:
- "What's the difference between Power Automate Premium and Copilot Studio licenses?"
- "Can I use premium connectors with just a Copilot Studio license?"
- "Should we buy Power Automate Premium or Copilot Studio for our automation needs?"
- "What are the limitations of agent flows?"

#### Enterprise and Organisation-Specific:
- "Is the Azure Blob Storage connector approved for our organization?"
- "What data classification level do I need for processing customer contracts?"
- "Show me our organization's pattern for integrating with internal HR systems"
- "What compliance checks do I need for a GDPR-covered automation?"

#### Debugging and Optimization:
- "My flow is returning a 429 error from SharePoint, how do I fix it?"
- "How can I improve the performance of my flow that processes 10,000 items?"
- "What's causing this 'InvalidTemplate' error in my flow?"
- "How do I implement retry logic for an unreliable API?"

## Contributing

Contributions are welcome! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

### Ways to Contribute

- Add new reference documentation
- Improve existing guides with real-world examples
- Share organization-specific patterns (anonymized)
- Report issues or suggest improvements
- Enhance the skill's capabilities

## Licensing

### This Skill
This Claude Code skill is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

### Power Automate & Related Products
Power Automate, Copilot Studio, Dataverse, and AI Builder are Microsoft products subject to their respective licensing terms:
- [Power Platform Licensing Guide](https://go.microsoft.com/fwlink/?linkid=2085130)
- [Microsoft Product Terms](https://www.microsoft.com/licensing/terms/)

This skill is not affiliated with, officially maintained by, or endorsed by Microsoft Corporation.

## Version History

See [CHANGELOG.md](CHANGELOG.md) for detailed version history.

**Current Version: 6.0.0** (November 2025)
- Phase 7: Organisation-Specific Patterns
- Comprehensive organizational knowledge integration
- Enhanced connector approval workflows
- Data classification and compliance frameworks

## Author & Maintainer

Created and maintained by [Kjell Tore Guttormsen](https://github.com/ktg0066)

## Support & Community

- **Issues**: [GitHub Issues](https://github.com/ktg0066/power-automate-claude-skill/issues)
- **Discussions**: [GitHub Discussions](https://github.com/ktg0066/power-automate-claude-skill/discussions)
- **Pull Requests**: [Contributing Guidelines](CONTRIBUTING.md)

## Acknowledgments

- Built for [Claude Code](https://claude.ai/code) by Anthropic
- Inspired by the Power Platform community
- Based on Microsoft's official Power Automate documentation and best practices

## Disclaimer

This skill provides guidance based on publicly available Microsoft documentation and community best practices. Always verify current Microsoft documentation and licensing terms before making architectural or procurement decisions. The author is not responsible for any decisions made based on this skill's recommendations.

---

**Made with Claude Code** | **MIT License** | **Version 6.0.0**
