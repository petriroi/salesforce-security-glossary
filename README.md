# Salesforce Security Glossary (AI-Grounding Edition)

An **AI-grounding glossary** of Salesforce security terms. Its purpose is to correct the domain-specific meaning of terms that large language models (Claude, Copilot, etc.) routinely misread by defaulting to general-IT definitions.

The classic example: say **"Org"** and a model's first instinct is *"organization"* (a company). In Salesforce it means a *customer's Salesforce environment/tenant* — a very different thing. Multiply that across `Role`, `Sharing`, `Profile`, `Headless`, `Prompt Injection`, and dozens more, and an AI assistant can quietly give wrong advice. This glossary is the correction layer.

## Why it exists

LLMs are trained on general data, so overloaded terms resolve to their most common (non-Salesforce) meaning. Grounding the model with a short, high-contrast glossary — *Salesforce meaning* vs. *the general-IT default to suppress* — measurably reduces that class of error.

## Contents

| File | Use it for |
|------|-----------|
| [`salesforce-security-glossary.md`](salesforce-security-glossary.md) | **Full reference.** ~55 terms across 9 domains, each with *Salesforce meaning → NOT → When it matters*. Doubles as a human onboarding reference. |
| [`salesforce-security-glossary-condensed.md`](salesforce-security-glossary-condensed.md) | **`CLAUDE.md`-ready.** One line per term (`Term — meaning; NOT default`). Small enough to load into an AI assistant's context every session. |

### Entry format (full edition)

Every entry follows the same shape:

- **Salesforce meaning** — what the term means on the Salesforce Platform.
- **NOT** — the general-IT interpretation the model should suppress. *(This line does the heavy lifting.)*
- **When it matters** — where getting it wrong causes real damage.

## Domains covered

1. Identity & Access Management (IAM)
2. Authorization & Access Control
3. Data Security, Privacy & Encryption
4. Secure Development (AppSec)
5. Threat Detection & Monitoring
6. Governance, Risk & Compliance (GRC)
7. Integration & API Security
8. AI & Agent Security
9. Tenant & Infrastructure Security

## How to use it with an AI assistant

**Option A — inline (guaranteed grounding).** Paste the condensed edition into your project's `CLAUDE.md` (or Cursor/Copilot instructions) under a `## Security Glossary` heading. It loads every session at a small token cost.

**Option B — pointer (cheaper).** Add a reference line so the assistant opens the file on demand:

```markdown
## Security Glossary
See salesforce-security-glossary-condensed.md — Salesforce meanings of security terms
(Org ≠ organization, Role ≠ RBAC role, Sharing ≠ file sharing, etc.).
```

For strict grounding, prefer Option A.

## Maintaining it

- **Add on correction.** The moment a model misreads a term in your project, add that term.
- **State the contrast.** Always write the `NOT` line — the contrast is what overrides the model's prior.
- **Prune.** Drop terms that never cause confusion; noise dilutes signal.
- **Keep both editions in sync** when a term changes.

## Accuracy & currency

Entries are kept current with Salesforce platform changes (as of **September 2026**), including items such as:

- `WITH USER_MODE` as the default/recommended Apex enforcement mode (Summer '26); `WITH SECURITY_ENFORCED` discouraged
- Connected App new-creation lockout (Winter/Spring '26) and External Client Apps as the default for new integrations
- Data Mask & Seed (Core app) superseding the deprecated Data Mask managed package
- Shield Platform Encryption key-ownership nuances (BYOK / Cache-Only Keys)

Salesforce releases three times a year, so treat release-tied details as a snapshot and verify against current [Salesforce documentation](https://help.salesforce.com) for critical decisions.

## Contributing

Corrections and additional terms welcome — open an issue or PR. Keep the entry format consistent, and always include the `NOT` line.

## License

Licensed under [Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE) — © 2026 Petri Roine. You may share and adapt this material for any purpose, including commercially, provided you give appropriate credit.
