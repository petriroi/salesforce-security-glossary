# Salesforce Security Glossary (AI-Grounding Edition)

> **Purpose.** This glossary grounds AI models (Claude, Copilot, etc.) in the *Salesforce-specific* meaning of security terms, so they don't default to general-IT definitions. Every entry follows the same shape:
>
> - **Salesforce meaning** — what the term means on the Salesforce Platform.
> - **NOT** — the general-IT interpretation the model should suppress.
> - **When it matters** — the situations where getting this wrong causes real damage.
>
> **How to use.** Drop this into a project's `CLAUDE.md` (or link to it) so it loads every session. Prune terms your team never touches; add terms the moment a model misreads one. Prioritize the `NOT` line — the contrast is what overrides the model's prior.

---

## Domain 1 — Identity & Access Management (IAM)

*Authentication: who a user is and how they prove it.*

### Org
- **Salesforce meaning:** A customer's Salesforce environment/tenant (a logical instance with its own data, config, and users). "Log into the org," "deploy to the org."
- **NOT:** An "organization" as a company or business entity.
- **When it matters:** Nearly every instruction. "Give access to the org" means the Salesforce environment, not a corporate directory.

### My Domain
- **Salesforce meaning:** A customer-specific login subdomain (e.g., `acme.my.salesforce.com`) required for SSO, branding, and modern security features.
- **NOT:** A DNS domain you register, or an Active Directory domain.
- **When it matters:** SSO/SAML config, session security, and Lightning features all assume My Domain is enabled.

### Connected App
- **Salesforce meaning:** A framework object that defines how an external application integrates with Salesforce via OAuth/SAML — including scopes, IP policies, and refresh-token behavior. **New creation is being locked down:** Winter '26 disabled Connected App creation via the UI by default for new orgs; Spring '26 disabled creation across **all** orgs (new and existing) unless Salesforce Support grants an explicit exception, and that exception path is expected to be removed entirely in a future release. **Existing Connected Apps keep functioning** — edit, deploy, and delete are unaffected; only *new creation* is blocked.
- **NOT:** A generic "app that is connected to the internet," a mobile app install, or an AppExchange managed package — a Connected App is specifically the OAuth/API integration and trust boundary.
- **When it matters:** Any new integration work should default to **External Client Apps** — creating a new Connected App now requires a Salesforce Support exception. Existing Connected App integrations need no change.

### External Client App
- **Salesforce meaning:** The newer, packageable replacement for Connected Apps — separates the app's *definition* from its *policies* for better lifecycle and second-generation packaging support.
- **NOT:** An app running outside the corporate firewall.
- **When it matters:** The default for all new integration work — as of Spring '26, creating a new Connected App requires a Salesforce Support exception, so ECAs are effectively the only unrestricted path for new builds.

### Named Credential
- **Salesforce meaning:** A named, admin-managed definition of an external endpoint URL *plus* its authentication (OAuth, JWT, mutual TLS, etc.), referenced by Apex/Flow so secrets never appear in code.
- **NOT:** A username, a saved password, or a login cookie.
- **When it matters:** Outbound callouts — using a Named Credential instead of hardcoded secrets is a core secure-coding requirement.

### MFA (Multi-Factor Authentication)
- **Salesforce meaning:** A contractually required second factor (Salesforce Authenticator, TOTP, security key) for all direct Salesforce logins; enforced by the platform.
- **NOT:** Optional, or satisfiable by SMS alone (SMS is discouraged).
- **When it matters:** Compliance — MFA is mandatory, not advisory, for Salesforce access.

### High Assurance Session / Step-Up Authentication
- **Salesforce meaning:** A session *security level* ("High Assurance") requiring stronger verification (e.g., MFA) before a user can perform sensitive actions — enforced per-action/per-policy (via Session Settings, Transaction Security, or session-level policies), not just at login.
- **NOT:** Basic login MFA — high assurance can be demanded *after* login, when the user reaches a protected action.
- **When it matters:** Gating sensitive operations (e.g., report export of PII, editing sharing) behind a stronger session level than initial login provided.

### Session Settings
- **Salesforce meaning:** Org-level configuration controlling session timeout, IP relaxation, required session security levels, and device activation — enforced server-side, independent of the authentication method used.
- **NOT:** Browser cookie expiration or client-side session storage — these are enforced by the platform regardless of the client.
- **When it matters:** Hardening session behavior (e.g., shorter timeouts, locking sessions to originating IP) for high-sensitivity profiles.

### Headless Identity / Headless Flows
- **Salesforce meaning:** A set of CIAM (customer identity) APIs — **Headless Registration**, **Headless Login**, **Headless Passwordless Login**, **Headless Forgot Password** — that let a custom-branded front end run auth entirely through APIs, with no redirect to a Salesforce-hosted login page. Built on OAuth 2.0 headless flows, typically for Experience Cloud digital experiences.
- **NOT:** "Headless" in the general web sense (no GUI, server-side rendering, or a headless CMS/browser).
- **When it matters:** Consumer-facing apps where the login UI must be fully custom. Misreading this leads to recommending redirect-based flows that break the "headless" requirement. (Note: "Headless 360" may also appear as an *org alias/label* in tooling — that usage is just a name, not this concept.)

---

## Domain 2 — Authorization & Access Control

*The multi-layered "who can see and do what." The richest area for AI confusion.*

### Profile
- **Salesforce meaning:** A baseline collection of permissions and settings assigned 1:1 to each user (object/field access, login hours, default record types). Being de-emphasized in favor of Permission Sets.
- **NOT:** A user's bio/profile page, a social profile, or a config "profile" file.
- **When it matters:** Access design. Every user has exactly one Profile; layering is done with Permission Sets.

### Permission Set
- **Salesforce meaning:** A grantable bundle of permissions assigned to users *in addition to* their Profile — additive only, never subtractive.
- **NOT:** A generic ACL or a single permission flag.
- **When it matters:** Modern least-privilege design assigns minimal Profiles and grants via Permission Sets.

### Permission Set Group
- **Salesforce meaning:** A container that bundles multiple Permission Sets for role-based assignment, with **Muting Permission Sets** to remove specific permissions from the group.
- **NOT:** A group of users, or a Public Group.
- **When it matters:** Scaling access administration; muting is the only way to subtract within the additive model.

### Muting Permission Set
- **Salesforce meaning:** A special Permission Set *inside a group* that removes (mutes) permissions otherwise granted by that group.
- **NOT:** Silencing notifications or disabling a user.
- **When it matters:** The one mechanism for subtraction in an otherwise additive permission model.

### OWD (Org-Wide Defaults)
- **Salesforce meaning:** The baseline record-level access for each object (Private / Public Read Only / Public Read/Write / Public Read/Write/Transfer (Lead and Case only) / Controlled by Parent) — the *most restrictive* setting, opened up by sharing.
- **NOT:** Global org preferences or default field values.
- **When it matters:** The foundation of the sharing model. "Set OWD to Private" is a data-isolation decision.

### Role / Role Hierarchy
- **Salesforce meaning:** A record-access mechanism where users higher in the hierarchy inherit access to records owned by users below them. Purely about *record visibility*, not permissions.
- **NOT:** An RBAC "role" that bundles permissions (that's closer to a Permission Set/Profile in Salesforce).
- **When it matters:** Roles grant record access up the hierarchy; they do **not** grant object/field permissions. Conflating Role with RBAC role is the single most common AI error.

### Sharing / Sharing Rules
- **Salesforce meaning:** Mechanisms that *open up* record access beyond OWD — Sharing Rules (criteria/owner-based), Manual Sharing, Apex Managed Sharing, and implicit sharing.
- **NOT:** File sharing, screen sharing, or "sharing" data with a third party.
- **When it matters:** Record-level authorization. "Share these records" means grant record access via the sharing model.

### Implicit Sharing
- **Salesforce meaning:** Access Salesforce grants *automatically* between related records, not defined by any explicit sharing rule — e.g., access to a child record (Contact, Opportunity, Case) confers implicit read access to its parent Account, and Account owners get implicit access to child records.
- **NOT:** Explicit Sharing Rules or Manual Sharing (both user/admin-configured). Implicit sharing is built-in platform behavior you cannot turn off per-record.
- **When it matters:** Diagnosing "why can this user see this record?" — the answer is often implicit sharing, not a rule. Also a reason locking down an Account cascades to its children's visibility assumptions.

### Restriction Rule / Scoping Rule
- **Salesforce meaning:** **Restriction Rules** further *limit* which records a user can see (subtractive at the record level, unlike sharing). **Scoping Rules** set the *default* set of records a user sees (filter), without changing what they *can* access.
- **NOT:** Firewall rules or query filters in code.
- **When it matters:** Restriction Rules are one of the few ways to tighten access below what sharing grants; Scoping ≠ Restriction (default view vs. hard limit).

### FLS (Field-Level Security)
- **Salesforce meaning:** Per-field read/edit permissions, set on Profiles/Permission Sets, enforced independently of object (CRUD) access.
- **NOT:** Encryption, or form field validation.
- **When it matters:** A user with object access may still be blocked from specific fields; code must respect FLS.

### CRUD
- **Salesforce meaning:** Object-level Create/Read/Update/Delete permissions, granted via Profiles/Permission Sets — enforced separately from FLS and sharing.
- **NOT:** Just the generic database CRUD verbs; here it's a distinct permission layer.
- **When it matters:** Secure Apex must check CRUD *and* FLS *and* sharing — three separate layers.

---

## Domain 3 — Data Security, Privacy & Encryption

*Protecting data at rest, in transit, and in use.*

### Shield Platform Encryption
- **Salesforce meaning:** A premium (Shield) feature encrypting data at rest at the field/file level while preserving key platform functionality. **By default, Salesforce manages the tenant secret** via the Key Management app (you can generate/rotate it, but the underlying key material is Salesforce-managed). *Customer-held* key material applies only when you opt into **BYOK** (you supply the key material) or **Cache-Only Keys** (the key is held externally and fetched on demand). Do not assume customer-controlled keys are the default.
- **NOT:** TLS/in-transit encryption, or Classic Encrypted Text fields.
- **When it matters:** Compliance-driven encryption; has functional trade-offs (some SOQL/reporting limits). Whether the customer controls the key depends on BYOK/Cache-Only opt-in, not the base feature.

### Deterministic vs. Probabilistic Encryption
- **Salesforce meaning:** **Deterministic** encryption yields the same ciphertext for the same plaintext (allows exact-match filtering); **Probabilistic** yields different ciphertext each time (stronger, but not filterable).
- **NOT:** A general crypto strength rating.
- **When it matters:** Choosing deterministic to keep a field filterable trades some security for functionality.

### BYOK / Cache-Only Keys
- **Salesforce meaning:** **Bring Your Own Key** lets customers supply key material to Shield; **Cache-Only Keys** keep the key in an external service, fetched on demand and never stored by Salesforce.
- **NOT:** Client-side encryption or a password vault.
- **When it matters:** Regulatory requirements for customer-held key custody.

### Data Mask (Data Mask & Seed)
- **Salesforce meaning:** **Data Mask & Seed** — a Core application (no package install required, available since Summer '26) that anonymizes/obfuscates sensitive data *in sandboxes* (not production), and now adds proactive PII detection, clearer error messages, data-integrity-preserving masking, and integrated seeding (including Data Cloud and AI seeding). The older **Data Mask managed package** still exists in some orgs but is deprecated: support ended July 9, 2026 and it becomes read-only December 31, 2026 — new work should use Data Mask & Seed.
- **NOT:** UI field masking (`****`) or FLS.
- **When it matters:** Preventing real PII from leaking into dev/test environments.

### Data Classification / Compliance Categorization
- **Salesforce meaning:** Field-level metadata tagging (Data Owner, Field Usage, Data Sensitivity Level, Compliance Categorization) to inventory and govern sensitive data.
- **NOT:** Data typing, or ML classification.
- **When it matters:** Privacy governance and audit; drives downstream policy decisions.

### Field Audit Trail
- **Salesforce meaning:** A Shield feature retaining field history for up to 10 years, beyond standard history limits.
- **NOT:** Setup Audit Trail (admin config changes) or Event Monitoring.
- **When it matters:** Long-term compliance/forensic retention of *data* changes.

---

## Domain 4 — Secure Development (AppSec / Platform Security)

*Vulnerabilities and defensive coding in Apex, LWC, Flow, and integrations.*

### SOQL Injection
- **Salesforce meaning:** Injecting malicious input into a dynamically built SOQL query string, mitigated with bind variables and `String.escapeSingleQuotes()`.
- **NOT:** SQL injection specifically (SOQL is Salesforce's query language, with its own syntax and rules).
- **When it matters:** Any dynamic query built from user input.

### `with sharing` / `without sharing` / `inherited sharing`
- **Salesforce meaning:** Apex class keywords controlling whether the running user's sharing rules (record access) are enforced. Precise rules:
  - **`with sharing`** — enforces the running user's sharing rules.
  - **`without sharing`** — ignores sharing (runs in system context for record access).
  - **`inherited sharing`** — when the class is *called from another Apex class*, it runs in that caller's sharing mode; when *invoked directly* as the entry point (e.g., Aura/LWC `@AuraEnabled`, REST, Visualforce controller, batch), it **defaults to `without sharing`**. It is the secure default for reusable classes because it doesn't force a mode on its callers.
  - **No keyword at all** — the class **inherits the sharing mode of its calling class**; if there is no calling class (it's the entry point), it defaults to **`without sharing`**. This is the trap: an entry-point class with no keyword runs without sharing enforcement.
- **NOT:** File-sharing settings or code visibility modifiers.
- **When it matters:** An entry-point Apex class with no sharing keyword bypasses record-level security. Declare `with sharing` (or `inherited sharing` for reusable library classes) explicitly; never rely on the default.

### USER_MODE / SECURITY_ENFORCED / stripInaccessible
- **Salesforce meaning:** Enforcement mechanisms for CRUD/FLS in Apex: `WITH USER_MODE` (enforces CRUD/FLS *and* sharing on both SOQL **and** DML), `WITH SECURITY_ENFORCED` (SOQL-only, no DML coverage), and `Security.stripInaccessible()` (removes fields the user can't access before DML/serialization). **As of Summer '26, `USER_MODE` became the standard/default mode for Apex database operations** (replacing system mode as the default) and is the clearly recommended choice; `WITH SECURITY_ENFORCED` is still functional but discouraged (no formal deprecation announced — new code should not use it).
- **NOT:** OS user modes or database roles.
- **When it matters:** Historically Apex ran in *system mode* and ignored CRUD/FLS; from Summer '26 the default is user mode, but explicit `WITH USER_MODE` is still best practice for clarity. Use `WITH USER_MODE`, not `WITH SECURITY_ENFORCED`, in new code. Related: **Apex classes now respect sharing by default going forward, while triggers still default to `without sharing` behavior unless declared explicitly** — see the `with sharing` entry.

### Lightning Locker / Lightning Web Security (LWS)
- **Salesforce meaning:** Client-side JavaScript sandboxing that isolates components from each other and the DOM. **LWS** is the newer, less restrictive successor to **Locker Service**.
- **NOT:** A password manager, account lockout, or generic CSP.
- **When it matters:** Third-party JS libraries and cross-component DOM access are constrained by these.

### Guest User
- **Salesforce meaning:** The unauthenticated user context for public Experience Cloud sites — a well-known source of over-permissioning and data exposure.
- **NOT:** A temporary trial account or an anonymous DB connection.
- **When it matters:** Guest user access is a top security-review target. Beyond OWD and sharing, a large share of real-world guest exposure comes from **over-permissioned Guest User License / Profile object & field access** (CRUD/FLS granted to the guest profile). A proper guest security review checks **both** layers — record access (OWD/sharing) *and* object/field permissions on the guest profile.

---

## Domain 5 — Threat Detection, Monitoring & Response

*Detective and reactive controls.*

### Shield Event Monitoring / Real-Time Event Monitoring
- **Salesforce meaning:** Shield feature exposing granular event logs (logins, API calls, report exports, page views) as log files and/or streaming/storable Real-Time Events for near-live detection.
- **NOT:** Generic application logging or `System.debug`.
- **When it matters:** SIEM integration, insider-threat detection, forensic analysis.

### Transaction Security Policy (TSP)
- **Salesforce meaning:** Real-time policies that evaluate events (via Condition Builder or Apex `EventCondition`/`TxnSecurity` classes) and take action — block, require MFA, notify — e.g., on large report exports or suspicious logins.
- **NOT:** Database transactions or payment transactions.
- **When it matters:** Enforcing real-time preventive/detective controls on sensitive actions.

### Threat Detection
- **Salesforce meaning:** ML-driven Shield detections: Session Hijacking, Credential Stuffing, Report Anomaly, API Anomaly, Guest User Anomaly — surfaced as Real-Time Events.
- **NOT:** Antivirus or network IDS.
- **When it matters:** Detecting account-takeover and data-exfiltration patterns.

### Setup Audit Trail
- **Salesforce meaning:** A log of *administrative/configuration* changes (who changed what setup, when), retained ~6 months (extendable).
- **NOT:** Field Audit Trail (data changes) or Event Monitoring (user activity).
- **When it matters:** Change accountability and config-drift investigations.

### Security Center
- **Salesforce meaning:** A product providing multi-org security posture dashboards, metrics, and drift detection across an org estate.
- **NOT:** Setup > Security, or Health Check alone.
- **When it matters:** Managing security across many orgs from one pane.

---

## Domain 6 — Governance, Risk & Compliance (GRC)

*Framework and posture.*

### Health Check
- **Salesforce meaning:** A Setup tool scoring an org's security settings against a baseline (Standard or custom), with a percentage score and remediation guidance.
- **NOT:** System uptime/health monitoring or a medical concept.
- **When it matters:** Baseline posture assessment and remediation prioritization.

### Trust (trust.salesforce.com)
- **Salesforce meaning:** Salesforce's public site for real-time system status, security, and compliance information; also the "Trusted" pillar of Well-Architected.
- **NOT:** Generic trust relationships or PKI trust chains.
- **When it matters:** Incident/status communication and architectural principles.

### Well-Architected (Trusted pillar)
- **Salesforce meaning:** Salesforce's architecture framework; the **Trusted** pillar covers secure, compliant, and reliable design.
- **NOT:** AWS Well-Architected (a different, though analogous, framework).
- **When it matters:** Architecture reviews and design guidance on-platform.

### Least Privilege
- **Salesforce meaning:** On-platform, realized via minimal Profiles + additive Permission Sets + tight OWD — a specific implementation pattern, not just a principle.
- **NOT:** Only an abstract security maxim.
- **When it matters:** Access design reviews; the Salesforce mechanics differ from OS/DB privilege models.

---

## Domain 7 — Integration & API Security

*The perimeter between Salesforce and the outside world.*

### IP Allowlisting / Login IP Ranges
- **Salesforce meaning:** Restricting logins/API access by source IP, configured at the org, Profile, or Connected App level.
- **NOT:** Network firewall rules you manage (though conceptually similar).
- **When it matters:** Locking down where users and integrations can authenticate from.

### OAuth Scopes
- **Salesforce meaning:** Permissions granted to a Connected/External Client App (`api`, `refresh_token`, `full`, `web`, etc.) that bound what an access token can do.
- **NOT:** Variable scope in code, or project scope.
- **When it matters:** Least-privilege for integrations; over-scoping (`full`) is a common finding.

### CORS (in Salesforce)
- **Salesforce meaning:** An allowlist in Setup of origins permitted to call Salesforce APIs from a browser; separate from CSP Trusted Sites.
- **NOT:** Only the generic browser CORS spec — Salesforce requires explicit org config.
- **When it matters:** Browser-based apps calling Salesforce APIs cross-origin.

### Mutual TLS (mTLS)
- **Salesforce meaning:** Two-way certificate authentication for API/inbound connections, configurable via Named Credentials and org settings.
- **NOT:** Standard one-way TLS (server-only cert).
- **When it matters:** High-assurance B2B integrations requiring client-cert auth.

---

## Domain 8 — AI & Agent Security

*The fastest-growing domain — securing Agentforce, Einstein, and LLM features. Where general-IT AI definitions misfire most.*

### Einstein Trust Layer
- **Salesforce meaning:** A security/governance architecture wrapping generative AI — dynamic grounding, data masking, zero data retention, toxicity detection, and audit logging — sitting between Salesforce and the LLM.
- **NOT:** A generic "AI safety filter" or a network trust boundary.
- **When it matters:** Any Agentforce/Einstein GenAI feature; it's the built-in guardrail layer.

### Zero Data Retention (ZDR)
- **Salesforce meaning:** A contractual/architectural guarantee (via the Trust Layer) that prompts and responses sent to third-party LLMs are not stored or used for training by the model provider.
- **NOT:** Data retention *policies* in the org, or backup retention.
- **When it matters:** Compliance assurance when using external foundation models.

### Prompt Injection
- **Salesforce meaning:** Adversarial input that manipulates an LLM-powered feature (Agentforce action, prompt template) into ignoring instructions or leaking data; mitigated by grounding, guardrails, and the Trust Layer.
- **NOT:** SQL/SOQL injection (different mechanism, though analogous).
- **When it matters:** Designing/reviewing Agentforce agents and prompt templates.

### Grounding / Dynamic Grounding
- **Salesforce meaning:** Securely injecting relevant, permission-respecting org data into an LLM prompt at runtime so responses are accurate and access-controlled.
- **NOT:** Electrical grounding, or "grounding" as calming/base-lining.
- **When it matters:** Ensuring AI responses respect the running user's data access and stay factual.

### Agentforce Guardrails / Running-User Context
- **Salesforce meaning:** Controls that constrain what an autonomous agent can do, plus the principle that an agent's actions execute under a specific user's permissions and sharing.
- **NOT:** Generic "AI guardrails" or infrastructure rate limits.
- **When it matters:** An agent must not exceed its running user's authorization; over-privileged agents are a top risk.

---

## Domain 9 — Tenant & Infrastructure Security

*The platform substrate.*

### Multi-Tenancy / Tenant Isolation
- **Salesforce meaning:** The architecture where many customers (orgs) share infrastructure while being logically isolated; enforced by the platform, not the customer.
- **NOT:** VM/container isolation you configure, or a single-tenant deployment.
- **When it matters:** Governor limits and platform constraints stem from multi-tenancy; you don't manage the isolation yourself.

### Hyperforce
- **Salesforce meaning:** Salesforce's public-cloud infrastructure architecture enabling regional data residency and scalability.
- **NOT:** A generic hyperscaler account or a "force" API.
- **When it matters:** Data residency/compliance requirements and ongoing org migrations.

### Data Residency
- **Salesforce meaning:** The guarantee (via Hyperforce regions) that org data is stored in a specified geography.
- **NOT:** Where users log in from, or backup location alone.
- **When it matters:** GDPR and regional compliance mandates.

### Sandbox
- **Salesforce meaning:** A copy of a production org for dev/test; types (Developer, Developer Pro, Partial Copy, Full) differ in how much *data* they include — a data-exposure consideration.
- **NOT:** A JS/security sandbox (that's Locker/LWS) or an OS sandbox.
- **When it matters:** Full/Partial sandboxes may contain real PII — mask it (see Data Mask).

### Scratch Org
- **Salesforce meaning:** A short-lived, source-driven, disposable org for development/CI, defined by a config file.
- **NOT:** A sandbox, or a "scratch" temp directory.
- **When it matters:** Source-driven dev; contains no production data by default (safer for experimentation).

---

## Quick-Reference: Highest AI-Confusion Terms

| Term | Model's likely (wrong) default | Salesforce meaning |
|------|-------------------------------|--------------------|
| **Org** | Organization / company | Customer's Salesforce environment |
| **Role** | RBAC permission bundle | Record-access hierarchy only |
| **Sharing** | File/screen sharing | Record-access grant model |
| **Profile** | User bio page | Baseline permission set (1:1 per user) |
| **Headless** | No-GUI / headless CMS | API-driven CIAM auth flows |
| **Prompt Injection** | SQL injection variant | LLM instruction manipulation |
| **Sandbox** | JS/OS security sandbox | Copy of production org |
| **CRUD** | Generic DB verbs | Distinct object-permission layer |

---

*Maintenance: add a term the moment a model misreads it in your project; state the `NOT` contrast explicitly; prune terms that never cause confusion.*
