# Contributing to Power Automate Expert Skill

Thank you for your interest in contributing to the Power Automate Expert Skill for Claude Code! This document provides guidelines for contributing to this project.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [How Can I Contribute?](#how-can-i-contribute)
- [Development Guidelines](#development-guidelines)
- [Pull Request Process](#pull-request-process)
- [Style Guide](#style-guide)

## Code of Conduct

This project adheres to a Code of Conduct that all contributors are expected to follow. Please read [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) before contributing.

## How Can I Contribute?

### Reporting Bugs

Before creating bug reports, please check existing issues to avoid duplicates. When creating a bug report, include:

- **Clear title and description**
- **Steps to reproduce** the issue
- **Expected behavior** vs actual behavior
- **Claude Code version** and operating system
- **Skill version** you're using
- **Example flow or code** that demonstrates the issue (if applicable)

Use the bug report template when creating a new issue.

### Suggesting Enhancements

Enhancement suggestions are welcome! Please include:

- **Clear title and description** of the enhancement
- **Use case** explaining why this would be valuable
- **Examples** of how the feature would work
- **Alternatives considered** (if any)

Use the feature request template when creating a new issue.

### Improving Documentation

Documentation improvements are highly valued! You can contribute by:

- **Fixing typos or clarifying existing docs**
- **Adding new reference documentation** for Power Automate features
- **Creating examples** and usage patterns
- **Improving installation instructions**
- **Adding troubleshooting guides**

### Adding Reference Content

The `references/` folder contains knowledge base files. You can contribute by:

1. **Enhancing existing references** with:
   - Real-world examples
   - Additional best practices
   - Common pitfalls and solutions
   - Performance optimization tips

2. **Creating new reference files** for:
   - New Power Automate features
   - Specific connector deep-dives
   - Advanced patterns and architectures
   - Industry-specific guidance

3. **Maintaining accuracy** by:
   - Updating deprecated information
   - Adding notes about version-specific features
   - Linking to official Microsoft documentation

### Sharing Organisation Patterns

If you have proven patterns from your organization (anonymized):

1. Add them to `assets/organisation-specific/flow-patterns/`
2. Ensure all organization-specific information is removed
3. Document the use case and benefits
4. Include any prerequisites or dependencies

## Development Guidelines

### File Structure

When adding new content, follow the existing structure:

```
references/               # General Power Automate knowledge
assets/organisation-specific/  # Customizable templates
assets/templates/        # Reusable templates
```

### Markdown Guidelines

- Use GitHub-flavored markdown
- Include a table of contents for longer documents
- Use clear heading hierarchy (H1 → H2 → H3)
- Include code examples in appropriate language blocks
- Add links to official Microsoft documentation where relevant

### Reference File Template

When creating new reference files, use this structure:

```markdown
# [Topic Name]

**Last Updated:** [Date]
**Applies to:** Power Automate [Cloud/Desktop/Both]

## Overview

Brief description of the topic and why it matters.

## Key Concepts

### Concept 1
Explanation with examples

### Concept 2
Explanation with examples

## Common Patterns

### Pattern 1: [Pattern Name]
Description, use case, implementation

## Best Practices

- Practice 1
- Practice 2

## Common Pitfalls

### Pitfall 1
Problem, why it happens, solution

## Examples

### Example 1: [Scenario Name]
Step-by-step example

## Additional Resources

- [Official Microsoft Docs](URL)
- [Related Reference Files](PATH)
```

## Pull Request Process

### Before Submitting

1. **Test your changes**: Verify the skill works with your modifications
2. **Update documentation**: Ensure README and relevant docs reflect your changes
3. **Follow style guide**: Match existing formatting and conventions
4. **Check links**: Verify all internal and external links work

### Submitting a Pull Request

1. **Fork the repository** and create your branch from `main`:
   ```bash
   git checkout -b feature/your-feature-name
   ```

2. **Make your changes** following the guidelines above

3. **Commit your changes** with clear, descriptive messages:
   ```bash
   git commit -m "Add comprehensive guide for HTTP connector authentication"
   ```

4. **Push to your fork**:
   ```bash
   git push origin feature/your-feature-name
   ```

5. **Open a Pull Request** with:
   - Clear title describing the change
   - Description of what changed and why
   - Reference to any related issues
   - Examples or screenshots (if applicable)

### Pull Request Review Process

1. Maintainers will review your PR within a few days
2. Address any feedback or requested changes
3. Once approved, maintainers will merge your PR
4. Your contribution will be included in the next release

## Style Guide

### Markdown Style

- **Headings**: Use sentence case for headings
- **Lists**: Use `-` for unordered lists, `1.` for ordered lists
- **Code blocks**: Always specify the language for syntax highlighting
- **Emphasis**: Use **bold** for important terms, *italics* for emphasis
- **Links**: Use descriptive link text, not "click here"

### Power Automate Conventions

- **Actions**: Capitalize action names (e.g., "Apply to each", "Compose")
- **Connectors**: Use official connector names (e.g., "SharePoint Online", not "Sharepoint")
- **Expressions**: Format inline expressions as `code`
- **Flow JSON**: Use proper indentation and formatting

### Code Examples

```markdown
Power Automate expression:
\`\`\`
if(greater(triggerBody()?['Amount'], 1000), 'High Value', 'Standard')
\`\`\`

PowerShell script:
\`\`\`powershell
# Example deployment script
Connect-PowerPlatform -EnvironmentUrl $envUrl
\`\`\`
```

### Terminology

- Use **"organisation"** (British spelling) consistently
- Use **"cloud flow"** not "flow" when specificity matters
- Use **"desktop flow"** not "RPA flow" or "UI flow"
- Use **"Dataverse"** not "CDS" or "Common Data Service"
- Use **"Copilot Studio"** not "Power Virtual Agents"

## Attribution

Contributors will be acknowledged in:
- Release notes for significant contributions
- The CHANGELOG.md file
- Comments in code/documentation for major additions

## Questions?

If you have questions about contributing:

1. Check existing [Issues](https://github.com/ktg0066/power-automate-claude-skill/issues)
2. Start a [Discussion](https://github.com/ktg0066/power-automate-claude-skill/discussions)
3. Review this document and the [Code of Conduct](CODE_OF_CONDUCT.md)

Thank you for contributing to make this skill better for the entire Power Automate community!
