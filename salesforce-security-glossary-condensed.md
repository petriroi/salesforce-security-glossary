# Salesforce Security Glossary (Condensed — CLAUDE.md)

> Grounds domain terms to their Salesforce meaning. Format: **Term** — meaning; **NOT** general-IT default. Full reference: `salesforce-security-glossary.md`.

## Identity & Access Management
- **Org** — a customer's Salesforce environment/tenant; **NOT** an "organization"/company.
- **My Domain** — customer login subdomain (`acme.my.salesforce.com`) required for SSO/modern features; **NOT** a DNS or AD domain.
- **Connected App** — OAuth/SAML integration config (scopes, IP, policies); new creation locked down as of Spring '26 (Support exception required), existing apps unaffected; **NOT** a generic or AppExchange app.
- **External Client App** — packageable successor to Connected Apps and the default for new integrations (Connected App creation now requires a Support exception); **NOT** an app running outside the firewall.
- **Named Credential** — admin-managed endpoint URL + auth, so secrets stay out of code; **NOT** a saved username/password.
- **MFA** — contractually required second factor for all logins; **NOT** optional or SMS-only.
- **High Assurance Session / Step-Up Auth** — stronger per-action session level (e.g., MFA before a sensitive action); **NOT** just login MFA.
- **Session Settings** — server-side session timeout/IP/security-level controls; **NOT** browser cookie expiration.
- **Headless Identity** — API-driven CIAM auth flows (register/login/passwordless) with no Salesforce login page; **NOT** headless CMS/browser or "no-GUI." ("Headless 360" may also just be an org alias.)

## Authorization & Access Control
- **Profile** — baseline permissions, 1:1 per user; **NOT** a bio/profile page.
- **Permission Set** — additive grantable permission bundle; **NOT** a generic ACL.
- **Permission Set Group** — bundle of permission sets, assignable as a unit; **NOT** a group of users.
- **Muting Permission Set** — subtracts permissions *within its group only*; **NOT** an org-wide deny ACL.
- **OWD (Org-Wide Defaults)** — most-restrictive baseline record access per object; **NOT** global prefs or default field values.
- **Role / Role Hierarchy** — record-access inheritance up the tree; **NOT** an RBAC permission bundle or org chart.
- **Sharing / Sharing Rules** — mechanisms that *open up* record access beyond OWD; **NOT** file/screen sharing.
- **Implicit Sharing** — automatic built-in parent/child record access; **NOT** explicit or manual sharing (and can't be turned off).
- **Restriction Rule / Scoping Rule** — Restriction *limits* records seen; Scoping sets the *default* filter; **NOT** firewall/query filters.
- **FLS (Field-Level Security)** — per-field read/edit permissions; **NOT** encryption or masking.
- **CRUD** — object-level Create/Read/Update/Delete permission layer; **NOT** just generic DB verbs.

## Data Security, Privacy & Encryption
- **Shield Platform Encryption** — field/file encryption at rest; tenant secret is Salesforce-managed by default (customer-held keys only via BYOK/Cache-Only); **NOT** TLS/in-transit or Classic encrypted text.
- **Deterministic vs. Probabilistic Encryption** — deterministic is filterable (same ciphertext), probabilistic is stronger but not; **NOT** a generic strength rating.
- **BYOK / Cache-Only Keys** — customer-supplied / externally-held key material; **NOT** client-side encryption or a password vault.
- **Data Mask (Data Mask & Seed)** — Core app (Summer '26) that anonymizes data *in sandboxes* + PII detection/seeding; legacy managed package deprecated; **NOT** UI masking or FLS.
- **Data Classification / Compliance Categorization** — field metadata tagging sensitivity/ownership; **NOT** data typing or ML classification.
- **Field Audit Trail** — long-term (≤10yr) field *history* retention; **NOT** Setup Audit Trail or Event Monitoring.

## Secure Development
- **SOQL Injection** — injection into dynamic SOQL; **NOT** SQL injection (SOQL is its own language).
- **with / without / inherited sharing** — Apex keywords for record-access enforcement (`inherited` = secure default); **NOT** file-sharing or code visibility.
- **USER_MODE / SECURITY_ENFORCED / stripInaccessible** — enforce CRUD/FLS in Apex; `USER_MODE` is the default as of Summer '26 (SOQL+DML) and recommended; `WITH SECURITY_ENFORCED` is SOQL-only and discouraged; **NOT** OS user modes. (Classes respect sharing by default going forward; triggers still default to `without sharing` unless declared.)
- **Lightning Locker / Lightning Web Security** — client-side JS sandboxing (LWS is the newer successor); **NOT** a password manager or account lockout.
- **Guest User** — unauthenticated Experience Cloud context; a top over-permissioning risk; **NOT** a trial account or anonymous DB connection.

## Threat Detection & Monitoring
- **Shield / Real-Time Event Monitoring** — granular event logs & streaming events for detection; **NOT** app logging or `System.debug`.
- **Transaction Security Policy (TSP)** — real-time policies on events (block/MFA/notify); **NOT** DB or payment transactions, or a WAF rule.
- **Threat Detection** — ML detections (session hijacking, credential stuffing, anomalies); **NOT** antivirus or network IDS.
- **Setup Audit Trail** — log of admin/config changes; **NOT** Field Audit Trail (data) or Event Monitoring (activity).
- **Security Center** — multi-org posture dashboards & drift; **NOT** Setup > Security or Health Check alone.

## Governance, Risk & Compliance
- **Health Check** — scores org security settings vs. a baseline; **NOT** uptime/system health.
- **Trust (trust.salesforce.com)** — Salesforce status/security/compliance site; also the Well-Architected "Trusted" pillar; **NOT** PKI trust chains.
- **Well-Architected (Trusted pillar)** — Salesforce's architecture framework; **NOT** AWS Well-Architected.
- **Least Privilege** — realized via minimal Profiles + additive Permission Sets + tight OWD; **NOT** just an abstract maxim.

## Integration & API Security
- **IP Allowlisting / Login IP Ranges** — restrict logins/API by source IP (org/profile/app); **NOT** your own network firewall.
- **OAuth Scopes** — permissions bounding an access token (`api`, `refresh_token`, `full`…); **NOT** variable/project scope.
- **CORS (Salesforce)** — Setup allowlist of browser origins allowed to call APIs; **NOT** only the generic browser spec (needs org config).
- **Mutual TLS (mTLS)** — two-way cert auth for connections; **NOT** standard one-way TLS.

## AI & Agent Security
- **Einstein Trust Layer** — GenAI guardrail layer (grounding, masking, ZDR, toxicity, audit); **NOT** a generic AI filter or network boundary.
- **Zero Data Retention (ZDR)** — LLM provider doesn't store/train on prompts; **NOT** org data-retention or backup policies.
- **Prompt Injection** — adversarial input manipulating an LLM feature; **NOT** SQL/SOQL injection.
- **Grounding / Dynamic Grounding** — securely injecting permission-respecting org data into prompts; **NOT** electrical/emotional grounding.
- **Agentforce Guardrails / Running-User Context** — constraints + agents act under a user's permissions/sharing; **NOT** generic AI guardrails or rate limits.

## Tenant & Infrastructure Security
- **Multi-Tenancy / Tenant Isolation** — shared infra, logically isolated orgs, enforced by the platform; **NOT** VM/container isolation you configure.
- **Hyperforce** — Salesforce's public-cloud infra enabling regional residency; **NOT** a generic hyperscaler account.
- **Data Residency** — guarantee data is stored in a specified geography; **NOT** where users log in from.
- **Sandbox** — a copy of a production org (types differ in data included); **NOT** a JS/OS security sandbox.
- **Scratch Org** — short-lived, source-driven, disposable dev org; **NOT** a sandbox or temp directory.
