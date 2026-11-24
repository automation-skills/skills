# Changelog

All notable changes to the Power Automate Expert Skill for Claude Code will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [6.0.0] - 2025-11-24

### Added - Phase 7: Organisation-Specific Patterns

#### New Assets Structure
- **Organisation-specific knowledge base** in `assets/organisation-specific/`:
  - `approved-connectors.md` - Master connector approval list with status tracking
  - `data-classification-guide.md` - Data classification system and automation implications
  - `systems-integration.md` - Internal systems integration patterns and authentication
  - `compliance-requirements.md` - GDPR, security, and public sector compliance requirements
  - `flow-patterns/` - Directory for reusable organisation-specific flow patterns
  - `examples/` - Directory for real-world implementation examples

#### New Templates
- **Reusable templates** in `assets/templates/`:
  - `deployment-settings/` - Environment-specific deployment configurations
  - `pipelines/` - Azure DevOps pipeline templates
  - `flows/` - Common flow templates ready for adaptation
  - `scripts/` - PowerShell and other automation scripts
  - `checklists/` - Pre-deployment and compliance checklists

#### Enhanced Core Capabilities
- **New Capability #14: Organisation Context Application** - Comprehensive 5-step workflow for organisation projects:
  1. Connector Validation (check approval status before suggesting)
  2. Data Classification (verify classification compatibility)
  3. Compliance Requirements (follow compliance checklist)
  4. Leverage Organisation Patterns (use documented patterns)
  5. Templates and Assets (use pre-built templates)
- Updated Capability #5 to check organisation connector approval status
- Added organisation-specific response patterns and workflows
- Integrated compliance and security checks into all workflows

#### Critical Changes
- **Connector approval now mandatory** - All connector suggestions must verify organisation approval status first
- **Data classification required** - Must verify classification compatibility before proposing solutions
- **Compliance-first approach** - Security and compliance checks integrated into all design decisions
- **Template-driven development** - Leverage pre-built templates and patterns for consistency

#### Documentation
- Comprehensive README.md with installation instructions and usage examples
- CONTRIBUTING.md with guidelines for community contributions
- CODE_OF_CONDUCT.md for community standards
- Professional open-source repository structure

### Changed
- Reorganized knowledge base for better navigation
- Enhanced SKILL.md with organisation context workflows
- Improved response patterns for organisation-specific queries

## [5.0.0] - 2025-11-15

### Added - Phase 5: Connectors Deep Dive

#### Enhanced Reference Documentation
- **Enhanced `connectors-reference.md`** with comprehensive documentation for popular connectors:
  - SharePoint Online, Microsoft Teams, Outlook, Dataverse, HTTP
  - Excel Online, OneDrive for Business, Azure services
  - Detailed authentication patterns and setup considerations
  - Common actions and triggers with examples
  - Rate limits and throttling strategies
  - Error handling patterns and common pitfalls
  - Licensing requirements (standard vs premium)
  - Real-world usage examples

#### New Decision Framework
- **New `connector-decision-matrix.md`** with:
  - Native connector vs HTTP connector vs Custom connector decision flowcharts
  - Systematic connector selection framework
  - Cost-benefit analysis framework
  - Break-even calculations for custom connector development
  - Scenario-specific guidance
  - Comparison matrices for common scenarios

### Changed
- Updated Capability #5 (Connector Selection & Implementation) with new references
- Enhanced connector selection workflow with decision frameworks
- Expanded Azure services coverage

## [4.0.0] - 2025-11-10

### Added - Phase 4: AI & Intelligence

#### New AI Capabilities
- **AI Builder Integration** (`ai-builder-integration.md`):
  - Prebuilt AI models (OCR, sentiment analysis, form processing)
  - Custom AI models for organisation-specific needs
  - Model training and optimization guidance
  - Confidence thresholds and validation strategies
  - AI Builder credit consumption optimization

#### Copilot Integration
- **Copilot-Assisted Development** (`copilot-assisted-development.md`):
  - Natural language flow creation with Copilot
  - Effective prompt writing for flow generation
  - Expression assistant usage for complex formulas
  - When to use Copilot vs manual development
  - Balancing AI assistance with manual refinement

#### Process Intelligence
- **Process Mining** (`process-mining.md`):
  - Process and task mining methodologies
  - Event log preparation and data mapping
  - Identifying automation opportunities from process analysis
  - Task mining for desktop automation discovery
  - Creating discover → automate → monitor cycles

### Changed
- Added AI capabilities throughout core skill functionality
- Enhanced flow generation with AI Builder recommendations
- Integrated Copilot guidance into development workflows

## [3.0.0] - 2025-11-05

### Added - Phase 3: Enterprise Patterns

#### Enterprise Adoption
- **Enterprise Adoption Guide** (`enterprise-adoption.md`):
  - HEAT methodology (Help + Enable + Advocate + Train)
  - Center of Excellence (CoE) structure and roles
  - Governance frameworks and approval workflows
  - Public sector specific requirements
  - Environment strategy (dev, test, prod)

#### Application Lifecycle Management
- **Solution Management** (`solution-management.md`):
  - ALM best practices for Power Automate
  - Connection references and when to use them
  - Environment variables setup and deployment
  - Deployment settings file examples
  - CI/CD pipeline configurations

#### Scaling Infrastructure
- **Machine Groups** (`machine-groups.md`):
  - Traditional machine groups for on-premises scale
  - Hosted machine groups for cloud scale
  - Auto-scaling configuration
  - Capacity planning based on workload patterns
  - Cost optimization while meeting SLA requirements

### Changed
- Enhanced skill with enterprise deployment patterns
- Added governance and compliance considerations
- Integrated ALM workflows into solution guidance

## [2.0.0] - 2025-11-01

### Added - Phase 2: Performance & Scale

#### Performance Optimization
- **Performance Optimization Guide** (`performance-optimization.md`):
  - Enterprise-grade optimization strategies
  - Concurrency opportunities (parallel branches, Apply to each settings)
  - Data operations optimization (Select vs Apply to each, early filtering)
  - Caching patterns for repeated operations
  - Optimization checklists and anti-patterns

#### Work Queue Architecture
- **Work Queues Guide** (`work-queues.md`):
  - Dispatcher-Performer pattern for complex workflows
  - Priority-based processing strategies
  - SLA management for time-sensitive work
  - Machine group scaling for capacity
  - Queue monitoring and optimization

### Changed
- Enhanced expressions guide with performance considerations
- Updated flow generation with optimization by default
- Added performance analysis to debugging workflows

## [1.0.0] - 2025-10-25

### Added - Foundation Release

#### Core Power Automate Expertise
- Flow generation capabilities with error handling
- Debugging assistance with systematic approaches
- Connector knowledge for common scenarios
- Expression language guidance
- Dataverse integration patterns
- Copilot Studio integration basics
- Best practices from Microsoft documentation

#### Initial Reference Documentation
- `connectors-reference.md` - Basic connector catalog
- `expressions-guide.md` - Power Automate expression language
- `dataverse-integration.md` - Dataverse patterns
- `copilot-studio-patterns.md` - Copilot Studio integration
- `flow-generation.md` - Common flow templates
- `debugging-guide.md` - Troubleshooting approaches
- `best-practices.md` - Microsoft recommended patterns

#### Skill Structure
- SKILL.md with core capabilities
- Basic knowledge base organization
- Response patterns for common queries

---

## Release Notes Format

Each release includes:
- **Version number** following Semantic Versioning (MAJOR.MINOR.PATCH)
- **Release date**
- **Added**: New features and capabilities
- **Changed**: Changes to existing functionality
- **Deprecated**: Features that will be removed in future releases
- **Removed**: Features that have been removed
- **Fixed**: Bug fixes
- **Security**: Security updates and fixes

## Upgrade Notes

### Upgrading to 6.0.0
1. Review new `assets/organisation-specific/` structure
2. Customize templates for your organization
3. Add approved connectors list
4. Document data classification system
5. Review compliance requirements

### Upgrading to 5.0.0
1. Review enhanced `connectors-reference.md` for your commonly-used connectors
2. Use `connector-decision-matrix.md` for architecture decisions

### Upgrading to 4.0.0
1. Explore AI Builder integration capabilities
2. Try Copilot-assisted flow development
3. Consider process mining for optimization projects

### Upgrading to 3.0.0
1. Review enterprise adoption guide for CoE setup
2. Implement connection references for new solutions
3. Plan environment strategy

### Upgrading to 2.0.0
1. Review performance optimization guide
2. Consider work queues for complex processes
3. Apply optimization patterns to existing flows

---

## Future Roadmap

### Planned for 7.0.0
- Advanced error handling patterns
- Integration with Azure Monitor and Application Insights
- Custom connector development deep-dive
- Power Fx integration guidance

### Under Consideration
- Specific industry verticals (healthcare, finance, manufacturing)
- Low-code/no-code citizen developer guidance
- Power Apps integration patterns
- Advanced security and threat modeling

---

[6.0.0]: https://github.com/ktg0066/power-automate-claude-skill/releases/tag/v6.0.0
[5.0.0]: https://github.com/ktg0066/power-automate-claude-skill/releases/tag/v5.0.0
[4.0.0]: https://github.com/ktg0066/power-automate-claude-skill/releases/tag/v4.0.0
[3.0.0]: https://github.com/ktg0066/power-automate-claude-skill/releases/tag/v3.0.0
[2.0.0]: https://github.com/ktg0066/power-automate-claude-skill/releases/tag/v2.0.0
[1.0.0]: https://github.com/ktg0066/power-automate-claude-skill/releases/tag/v1.0.0
