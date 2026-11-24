---
name: n8n-credentials-security
description: Use whenever auth, API keys, OAuth, tokens, secrets, or credential setup appears in n8n work. Enforces credential system usage, least privilege, and safe HTTP auth patterns.
license: MIT
metadata:
  author: Taghi Molavi
  homepage: https://molavi.pro
  version: "1.0.0"
---

# n8n Credentials & Security

Author: [Taghi Molavi](https://molavi.pro)

## Non-negotiables

1. **No secrets in parameters, sticky notes, git, or Code strings.**
2. Use n8n **Credentials** UI/API; reference by credential attachment on the node.
3. Least privilege scopes; separate prod/staging credentials.
4. Rotate on leak; assume chat logs are semi-public.
5. For HTTP without native node: HTTP Request + proper credential type (OAuth2, Header Auth, etc.).

## Agent behavior
- Do **not** ask the user for the cosmetic display name obsessively — focus on type + scopes.
- Never echo full API keys back in summaries.
- Prefer managed/connect credentials when the user has no preference and platform supports it.

## Checklist
- [ ] Credential type matches API
- [ ] Test on staging first
- [ ] Webhook signatures verified
- [ ] No secret in workflow export shared publicly (strip credentials on export)
