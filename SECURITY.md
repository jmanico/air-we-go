# Air We Go — Security Requirements

Status: **provisional / bootstrap**. No implementation exists yet. This
document translates [`REQUIREMENTS.md`](./REQUIREMENTS.md) §7 (Security
Requirements) and §8 (Reliability and Data Quality), plus the stack decisions
in [`ARCHITECTURE.md`](./ARCHITECTURE.md), into concrete, enforceable rules
for Claude Code and every contributor. It is grounded in:

- [OWASP Top 10 Proactive Controls](https://owasp.org/www-project-proactive-controls/)
- [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/)
- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)
- [OWASP REST Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html)

Per `ARCHITECTURE.md`, Air We Go is a web + mobile + REST API platform (Node.js
backend, React web client, React Native mobile client, Azure/Terraform
deployment). This document therefore prioritizes, in order: **HTTP boundary
security, authentication and authorization, input validation, secret
handling, logging and error handling, deployment/CI-CD safety, and privacy
(GDPR-aligned) compliance.** The administrative surface is the primary
hardening target per `ARCHITECTURE.md` §"Security posture."

Nothing below overrides a `REQUIREMENTS.md` §7 "must." Where this document
adds detail, it narrows implementation choices — it does not loosen the
underlying requirement.

---

## Required Security Inputs

These are the facts a security review or Claude Code needs and where they
currently stand. Items marked **TO BE DECIDED** are open in
`ARCHITECTURE.md` and must be resolved before the corresponding rule below
can move from provisional to final.

| Input | Status | Source |
|---|---|---|
| Runtime / language | Node.js (server), React (web), React Native (mobile) | ARCHITECTURE.md |
| API style | REST, versioned | ARCHITECTURE.md, API-004 |
| Specific Node framework (Express / NestJS / Fastify / other) | **TO BE DECIDED** | ARCHITECTURE.md "Unknown" |
| Auth model | Password (argon2id) + optional passkey for regular users; **mandatory** phishing-resistant passkey for admins | ARCHITECTURE.md |
| WebAuthn/passkey library & relying-party config | **TO BE DECIDED** | ARCHITECTURE.md |
| Authorization model | Server-enforced RBAC (ADMIN-002 roles) as the default; attribute-based rules only where a role can't express the constraint | ARCHITECTURE.md, ADMIN-002 |
| Deployment | Microsoft Azure, provisioned via Terraform | ARCHITECTURE.md |
| Azure compute target (containers vs. PaaS) | **TO BE DECIDED** | ARCHITECTURE.md |
| Secret store | Azure-native secret manager assumed, product **TO BE DECIDED** | ARCHITECTURE.md |
| CI/CD tooling | **TO BE DECIDED** (must satisfy NFR-MAINT-005) | ARCHITECTURE.md |
| Operational / historical data-store engines | **TO BE DECIDED** | ARCHITECTURE.md |
| Observability stack (metrics/tracing/alerting) | **TO BE DECIDED** (must satisfy OBS-\*) | ARCHITECTURE.md |
| Air-quality / map / geocoding providers | **TO BE DECIDED** (§13 Q2, Q13) — affects the SSRF allowlist and third-party data-processing agreements | REQUIREMENTS.md §13 |
| Privacy jurisdictions in scope | **TO BE DECIDED** (§13 Q24) — assume GDPR applies given global/EU scope until ruled out | REQUIREMENTS.md §13 |
| Data retention periods per class | **TO BE DECIDED** (§5.4) | REQUIREMENTS.md §5.4 |
| RPO / RTO and region topology | **TO BE DECIDED** (§13 Q25, REL-005) | REQUIREMENTS.md, ARCHITECTURE.md |

---

## Provisional Security Rules

### 1. HTTP Boundary Security

- All production traffic over TLS only; no cleartext fallback (SEC-API-001).
- CORS is an explicit allowlist of known origins — never `*` with credentials
  (SEC-API-002).
- Every state-changing request from a browser session requires CSRF
  protection when cookie-based auth is used (SEC-API-003) — e.g. double-submit
  token or `SameSite=strict` plus origin check.
- Enforce on every endpoint: authentication where required, authorization,
  rate limiting, request-size limits, timeouts, pagination limits, and schema
  validation (SEC-API-004, API-005/006/007).
- Public API vs. authenticated API vs. administrative API are physically or
  logically isolated; the admin surface never shares a base path or trust
  boundary with the public API (API-003).
- Historical/query endpoints enforce maximum date ranges and result sizes to
  prevent unbounded processing (NFR-PERF-004).
- Outbound calls to air-quality providers, map tiles, and geocoding services
  are restricted to a configured destination allowlist — reduces SSRF risk
  from provider-supplied redirects or malicious feed URLs (SEC-INPUT-005).

### 2. Authentication

- No custom cryptography or session protocols — use established, audited
  libraries only (SEC-AUTH-001).
- Passwords (where used) are hashed with `argon2id`; never a fast hash
  (SHA-256/MD5) (SEC-AUTH-002).
- **Administrators must authenticate with a phishing-resistant passkey
  (WebAuthn/FIDO2)** — no password-only path for admin accounts
  (ADMIN-001, SEC-AUTH-003). See [FIDO Alliance passkeys](https://fidoalliance.org/passkeys/):
  passkeys use per-service key pairs, no shared secret ever leaves the
  device, and device unlock provides built-in user verification.
- Non-admin users authenticate with password + optional passkey as an
  additional/alternative factor. Federated identity is out of scope for v1.
- Auth failure responses must not reveal whether an account exists beyond
  what's operationally necessary (SEC-AUTH-004).
- Password-reset and email-verification tokens: cryptographically random,
  single-use, time-limited, and bound to the specific account + action
  (SEC-AUTH-005).
- Sessions: unpredictable IDs, `Secure`/`HttpOnly`/`SameSite` cookies, idle +
  absolute expiration, logout invalidation, and forced revocation on
  password or security-sensitive changes (SEC-SESS-001–003).

### 3. Authorization

- Deny by default; every protected operation checks authorization
  server-side — never trust hidden navigation or client-side checks
  (SEC-AUTHZ-001/002/005).
- Users may only read/write their own saved locations, alerts, preferences,
  notification settings, and sessions (SEC-AUTHZ-003).
- Administrative permissions are granular, least-privilege, and mapped to
  the roles in ADMIN-002 (Support/Data/Security/System Administrator,
  Read-Only Auditor) (SEC-AUTHZ-004).
- Default authorization model is **RBAC**, enforced server-side on every
  admin operation. Introduce attribute-based (ABAC) rules only for
  constraints a role can't express (e.g., "only during an active incident
  window," "only for providers this admin manages"); when you do, follow
  deny-overrides combination, treat unverified request headers as
  untrusted attributes, and fail closed (deny) on a missing attribute or an
  unreachable policy engine — never fail open.
- Every administrative action is audited: actor, action, target, timestamp,
  outcome, before/after values, and a correlation ID; no secrets in audit
  records (ADMIN-007/008). Audit logs are append-only / tamper-evident
  (SEC-LOG-005).

### 4. Input Validation

- All untrusted input — request bodies, query params, headers, path params,
  and **provider API responses** — is validated with positive (allowlist)
  constraints at the trust boundary; provider responses are never trusted
  implicitly (SEC-INPUT-001, SEC-INPUT-004, ING-010).
- Database access is exclusively parameterized queries or a safe ORM — no
  string-built SQL (SEC-INPUT-002).
- Output is encoded for its destination context (HTML, JSON, URL, attribute)
  before rendering (SEC-INPUT-003).
- Bulk/file/archive imports (e.g., provider backfills) enforce file-size
  limits, decompression limits, content-type validation, processing
  timeouts, and record-count caps (SEC-INPUT-006, ING-013).
- Invalid ingestion records are quarantined, never dropped silently or
  allowed to block a valid batch (ING-011).

### 5. Secret Handling

- Secrets (API keys, signing keys, DB/provider credentials) live only in an
  approved secret-management system — never in source, mobile app packages,
  client-side JS, logs, container images, or config files committed to the
  repo (SEC-004, SEC-MOB-001).
- Generate tokens/secrets with a CSPRNG (`node:crypto.randomBytes`), never
  `Math.random()`.
- Mobile apps store tokens only in platform-provided secure storage
  (Keychain / Keystore via `react-native-keychain`), never in `AsyncStorage`
  or plain files (SEC-MOB-002).
- Dev, test, staging, and production are logically separated environments
  with separate secrets and no shared credentials (SEC-003).

### 6. Logging and Error Handling

- Log authentication failures, lockouts/throttling, password resets, session
  revocations, authorization failures, admin actions, provider-config
  changes, data-suppression actions, and abnormal API usage (SEC-LOG-001).
- Never log passwords, session identifiers, auth tokens, provider secrets,
  full reset tokens, or unnecessary precise location (SEC-LOG-002,
  SEC-MOB-005).
- Structured logs with synchronized timestamps and correlation/request IDs
  for cross-service tracing (SEC-LOG-003, OBS-002).
- Error responses to clients never include stack traces, credentials,
  tokens, internal network details, DB queries, or config — return a generic
  message plus correlation ID; log the full error only server-side
  (SEC-API-005).
- Generate security alerts for material suspicious activity (SEC-LOG-004)
  and operational alerts per OBS-004 (provider failures, stale data,
  elevated errors, latency, queue backlog, DB failures, auth anomalies,
  storage exhaustion, failed backups).

### 7. Deployment and CI/CD Safety

Azure + Terraform is the given deployment target
([Wiz: Terraform security best practices](https://www.wiz.io/academy/application-security/terraform-security-best-practices/)):

- All infrastructure changes go through Terraform, version control, and PR
  review — no manual console changes to shared or production environments.
- Store Terraform state remotely and encrypted, with access restricted to
  approved CI runners; never commit state files.
- Separate Terraform state/workspace per environment (dev/test/staging/
  production), matching SEC-003's logical separation requirement.
- Run policy-as-code and static analysis (e.g., `tfsec`/Checkov, or
  OPA/Sentinel) on every PR; block merge/apply on violations.
- Grant least-privilege managed identities per service/pipeline — no broad
  or shared cloud credentials.
- Enforce mandatory resource tagging (owner, environment, data
  classification) via policy for accountability and incident response.
- Pin module and provider versions from a reviewed internal registry; run
  scheduled drift detection between declared and actual infrastructure.
- CI/CD pipelines are reproducible and automated end-to-end (NFR-MAINT-005),
  and never bypass required checks (tests, security scans) to ship faster.
- Dependency and secret scanning, static analysis, and security testing run
  in CI on every change, feeding a documented vulnerability-remediation
  process (SEC-001).

### 8. Privacy / GDPR-Aligned Compliance

`REQUIREMENTS.md` §13 Q24 leaves privacy jurisdiction open, but the product
is explicitly global-scale and may serve EU users, so this document adopts
GDPR-aligned defaults as the safe baseline until legal scoping narrows it:

- Collect only personal data necessary for a defined product function — no
  speculative collection (SEC-PRIV-001, data minimization).
- Precise device location requires explicit, revocable user permission; the
  app must remain usable when denied (SEC-PRIV-002, MOB-003).
- Disclose, in plain language, what location data is collected, why,
  whether/how long it's stored, and how to revoke it (SEC-PRIV-003).
- Support user rights to access, correct, export, and delete personal data,
  including full account deletion, subject to legal/security retention
  needs (SEC-PRIV-004, USER-008).
- Minimize precise location and account identifiers in analytics/telemetry
  (SEC-PRIV-005).
- Treat a confirmed EU user base as triggering: a documented lawful basis
  per processing purpose, data processing agreements with any third-party
  provider/processor (including map, geocoding, and notification vendors),
  a 72-hour breach-notification capability, and a record of processing
  activities. Confirm applicability with legal before launch.

---

## 9. Stack-Specific Coding Rules

### 9.1 Backend — Node.js

- Use `Object.create(null)` or `Map` for dictionary/key-value objects; never
  merge untrusted JSON into a plain object (prototype pollution). Block
  `__proto__`/`constructor`/`prototype` keys on any merge.
- Never build a shell command from input — use `execFile`/`spawn` with an
  argument array, never `exec()` with concatenated input.
- Use the `node:` import prefix for built-ins; never `require(userInput)`.
- Always `===`; never `==`.
- All tokens/secrets via `node:crypto.randomBytes()`, never `Math.random()`.
- Validate every trust boundary (body, query, headers, provider payloads,
  queue messages) with Zod/Ajv, `additionalProperties: false` to block mass
  assignment.
- Ban `eval()`, `new Function()`, and `vm.runInNewContext()` on any value
  that originated outside the process.
- Path handling: `path.resolve(baseDir, userPath)` then verify the result
  stays under `baseDir` (traversal/zip-slip); check for symlinks before
  reading archive members.
- `helmet()` as the first middleware; strict CSP (no `unsafe-inline`/
  `unsafe-eval`); HSTS with `includeSubDomains` + `preload`.
- Rate-limit with a distributed limiter (e.g. Redis-backed); cap request
  body size; set upstream/server timeouts.
- Structured logging (Pino/Winston) with automatic redaction of
  `authorization`/`cookie` headers; never `console.log` in production.
- `npm ci` with a committed lockfile; automated dependency/secret scanning
  in CI (Snyk/Socket or equivalent); TypeScript `strict: true`, avoid `any`.
- Specific framework hardening (Express/NestJS/Fastify/etc.) is **TO BE
  DECIDED** pending the framework choice in `ARCHITECTURE.md` — apply the
  framework-specific secure-developer prompt once selected.

### 9.2 Frontend — React (Web)

- Default to JSX interpolation (`{value}`) — it escapes automatically.
  Never construct JSX or tag names from untrusted strings.
- Rich/untrusted content renders via Markdown (escaped by default) or,
  if HTML is unavoidable, DOMPurify inside one dedicated `<SafeHtml>`
  wrapper — the only place `dangerouslySetInnerHTML` may appear anywhere in
  the codebase.
- Validate any externally sourced URL against an `https:`/`mailto:`/`tel:`
  allowlist before it reaches `href`, `src`, `formAction`, or `navigate()`;
  a Zod `.url()` check alone is insufficient (it accepts `javascript:`).
  `target="_blank"` requires `rel="noopener noreferrer"`.
- Validate all API responses with Zod before rendering; use an explicit
  loading/error/ready state machine so the UI never reads fields off
  `undefined`.
- For `postMessage`: verify `event.origin` against an allowlist and
  schema-validate `event.data`; never send with target origin `"*"`.
- Never spread untrusted props onto DOM elements; use stable, non-sensitive
  keys for list items (not raw emails/usernames, not array index for
  reorderable lists).
- No inline event-handler strings, `eval`, or `new Function` — the app must
  run under a strict CSP with no `unsafe-inline`.

### 9.3 Mobile — React Native (iOS / Android)

- Never store tokens, session data, or PII in `AsyncStorage` — it is
  unencrypted on disk on both platforms. Use `react-native-keychain` with
  platform-backed storage (`SECURE_HARDWARE` on Android, `THIS_DEVICE_ONLY`
  accessibility on iOS) for anything sensitive (SEC-MOB-002).
- Validate every native-module bridge parameter on the native side —
  bridge serialization to JSON allows type confusion regardless of the
  JS/TypeScript type declared on the calling side.
- WebViews: allowlist origins, set `allowFileAccess`/
  `allowFileAccessFromFileURLs`/`allowUniversalAccessFromFileURLs` to
  `false`, validate every `onMessage` payload against a strict schema, and
  never pass an auth token into a WebView.
- Hermes bytecode is not obfuscation — it is trivially decompiled; never
  embed API keys, signing keys, or sensitive logic in JS/TS shipped to the
  device.
- Deep links are attacker-controlled input from any app or browser —
  validate scheme, host, path, and every parameter against an allowlist
  before using them for navigation or API calls.
- iOS: no blanket `NSAllowsArbitraryLoads`; Android:
  `usesCleartextTraffic="false"` plus a `network_security_config.xml` — no
  cleartext production traffic (SEC-MOB-003).
- Minimize precise-location collection and retention; never log tokens or
  precise location (SEC-MOB-004/005).
- Never auto-copy sensitive values to the OS clipboard; blur/obscure
  sensitive screens when the app enters the background (task-switcher
  snapshot protection).

---

## 10. Third-Party Dependency Rules

- Do not add a dependency when the standard library or a few lines of
  first-party code will do.
- Prefer zero new dependencies. If a library is required, justify it in the
  PR description.
- Only use libraries that are actively maintained (a commit or release
  within the last 12 months).
- Only use the latest stable major version — no deprecated, abandoned, or
  pre-release packages.
- Reject any library with a known unpatched CVE. Check before adding and on
  every update.
- Audit transitive dependencies, not just direct ones — a small direct
  dependency with a large or unvetted tree is a rejection.
- Pin exact versions with a committed lockfile. No floating ranges in
  production.
- Prefer libraries with narrow scope, minimal dependencies of their own,
  and a clear security track record.

---

## 11. Prompt Placeholders To Resolve

Resolution status for each placeholder, based on the stack decisions already
recorded in `ARCHITECTURE.md`. Unresolved sub-parts are marked
**TO BE DECIDED** rather than guessed.

| Placeholder | Status | Resolution |
|---|---|---|
| `{{CODE_QUALITY_PROMPT}}` | **Resolved** | Low cyclomatic complexity, low cognitive complexity, and separation of concerns — applied uniformly across backend, web, and mobile code. |
| `{{API_SECURITY_PROMPT}}` | **Resolved** | REST confirmed (ARCHITECTURE.md). Apply [OWASP API Security Top 10](https://owasp.org/www-project-api-security/) and [OWASP REST Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html); compressed into §1 (HTTP Boundary Security) above. |
| `{{BACKEND_FRAMEWORK_PROMPT}}` | **Partially resolved** | Node.js confirmed → `platform/context/prompts/code security/Backend Frameworks/NodeJS/00 Secure Node.js Developer`, compressed into §9.1. The specific framework (Express/NestJS/Fastify/other) is **TO BE DECIDED**; apply the matching framework-specific prompt in that same directory once chosen. |
| `{{FRONTEND_FRAMEWORK_PROMPT}}` | **Resolved** | Web: React → `.../Client Side Frameworks/ReactJS/00 React19 Secure Generator (JS)`, compressed into §9.2. Mobile: React Native → `.../Mobile/02 Secure Mobile React Native Developer`, compressed into §9.3. |
| `{{AUTH_PROMPT}}` | **Resolved** | [FIDO Alliance passkeys](https://fidoalliance.org/passkeys/) for mandatory admin authentication (§2), plus `.../Authorization/02 ABAC Architect` for authorization-policy design once RBAC alone is insufficient (§3). RBAC (ADMIN-002) remains the default authorization model for v1. |
| `{{DEPLOYMENT_PROMPT}}` | **Resolved** | Azure + Terraform confirmed → [Wiz: Terraform security best practices](https://www.wiz.io/academy/application-security/terraform-security-best-practices/), compressed into §7. |

### Selected prompt imports summary

- **Architecture decision → REST, isolated admin API:** OWASP API Security
  Top 10 + REST Security Cheat Sheet (§1, §11).
- **Backend framework choice → Node.js (framework TBD):** Secure Node.js
  Developer prompt (§9.1); framework-specific supplement pending.
- **Frontend framework choice → React (web) + React Native (mobile):**
  React 19 secure-generator prompt (§9.2) and Secure Mobile React Native
  Developer prompt (§9.3).
- **Auth model → password + optional passkey (users), mandatory passkey
  (admins):** FIDO passkeys reference (§2) + ABAC Architect prompt for
  future policy design (§3).
- **Deployment model → Azure + Terraform:** Wiz Terraform security
  best-practices reference (§7).

---

*Security v0.1 (provisional) — derived from `REQUIREMENTS.md` §7–§9,
`DESIGN.md`, `ARCHITECTURE.md`, and the OWASP references above. Update this
document as the open questions in `REQUIREMENTS.md` §13 and the
**TO BE DECIDED** items in `ARCHITECTURE.md` are resolved.*
