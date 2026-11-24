# sf-datacloud-act

Activation and downstream delivery workflows for Salesforce Data Cloud.

## Use this skill for

- activations
- activation targets
- data actions
- data action targets
- downstream platform inspection before push

## Example requests

```text
"Activate this segment to the downstream platform"
"Show me what activation targets already exist"
"Create a data action target for this integration"
"Help me inspect why this activation is not ready"
```

## Common commands

```bash
sf data360 activation platforms -o myorg 2>/dev/null
sf data360 activation-target list -o myorg 2>/dev/null
sf data360 activation list -o myorg 2>/dev/null
sf data360 data-action-target list -o myorg 2>/dev/null
```

## References

- [SKILL.md](SKILL.md)
- [../sf-datacloud/assets/definitions/activation-target.template.json](https://github.com/Jaganpro/sf-skills/blob/53c9956c4279f8e9abda51164a3e9444811f0e8e/skills/sf-datacloud/assets/definitions/activation-target.template.json)
- [../sf-datacloud/assets/definitions/activation.template.json](https://github.com/Jaganpro/sf-skills/blob/53c9956c4279f8e9abda51164a3e9444811f0e8e/skills/sf-datacloud/assets/definitions/activation.template.json)
- [../sf-datacloud/assets/definitions/data-action-target.template.json](https://github.com/Jaganpro/sf-skills/blob/53c9956c4279f8e9abda51164a3e9444811f0e8e/skills/sf-datacloud/assets/definitions/data-action-target.template.json)
- [../sf-datacloud/assets/definitions/data-action.template.json](https://github.com/Jaganpro/sf-skills/blob/53c9956c4279f8e9abda51164a3e9444811f0e8e/skills/sf-datacloud/assets/definitions/data-action.template.json)
- [../sf-datacloud/references/plugin-setup.md](../sf-datacloud/references/plugin-setup.md)
- [CREDITS.md](CREDITS.md)

## License

MIT License - See [LICENSE](https://github.com/Jaganpro/sf-skills/blob/53c9956c4279f8e9abda51164a3e9444811f0e8e/skills/sf-datacloud-act/LICENSE).

<!-- Modified by Automation Skills on 2026-11-24: relative links to files that were not copied now point to the original repository. -->
