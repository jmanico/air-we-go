# Air We Go — Security Requirements

- **Status:** Threat-model-derived security baseline; provisional until the
  decision gates in ARCHITECTURE.md §9 are approved
- **Version:** 0.2
- **Threat-model assessment date:** 2026-07-17
- **Approval status:** Draft baseline; no accountable owner or approver has signed off
- **Owner / approvers:** TO BE DECIDED before implementation; approval must
  include product, security, privacy/legal, air-quality domain, engineering,
  mobile/client, accessibility, and operations representatives
- **Next review:** Before architecture freeze, and on every NFR-MAINT-007 trigger
- **v0.2 change summary:** Replaced the preliminary checklist with threat-derived, testable control and release requirements

No implementation exists. All controls are **Required by design — Not
Verified** until linked deployment or repeatable test evidence exists.

This document turns REQUIREMENTS.md §7–§9 and the threat model in
ARCHITECTURE.md §8 into enforceable rules for every contributor, service,
client, deployment, administrator, and vendor integration. It is a security
floor. A framework or ADR may make a rule more specific but may not weaken it
without a risk acceptance that SEC-IR-003 actually permits; an unresolved
Critical finding cannot be accepted for initial production.

Normative security references include the OWASP Application Security
Verification Standard, OWASP API Security Top 10, OWASP Cheat Sheet Series,
OWASP Mobile Application Security Verification Standard, WebAuthn/FIDO2
specifications, current Node.js security guidance, and primary Microsoft
Azure and HashiCorp Terraform security guidance. A project control remains
normative even when an external reference changes. Version/pin the exact
reference used in an ADR or test profile rather than relying on a mutable
landing page.

---

## 1. Threat-Model Posture

The controlling threats are TM-01–TM-24 in ARCHITECTURE.md:

- provider, rule, administrative, queue, cache, and failover corruption of
  public air-quality data;
- administrative authentication/recovery bypass, insider misuse, SSRF to
  Azure identity/control plane, and supply-chain takeover;
- account and precise/saved-location disclosure through BOLA, logs, caches,
  mobile state, notifications, exports, backups, and third parties; and
- public-query, ingestion, retry, queue, storage, notification, dependency,
  and regional resource exhaustion; and
- inaccessible, low-contrast, color-only, stale, or misleading safety status
  that causes disabled or other users to misread conditions.

The administrative surface is not the only hardening target. Public-data
integrity and user/location confidentiality receive equal protection.

### 1.1 Security invariants

- Deny by default at public, administrative, service, data, delivery, and
  cloud boundaries.
- Authenticate and authorize every human and workload action; network
  location is not identity.
- Treat all request, provider, queue, cache, file, log, URL, deep-link,
  notification, vendor, and build input as untrusted until validated for its
  destination and authority.
- Only the publication authority defined in ARCHITECTURE.md §4.5 may write
  trusted public observation or derived historical versions.
- Only the Identity service may write credential, authenticator, recovery,
  and session state.
- No public-read identity may write trusted data or read personal, secret,
  quarantine, audit, backup, or control-plane data.
- No high-impact privileged mutation succeeds without durable audit; no actor
  may approve their own elevation or high-impact change.
- No failure, retry, replay, restore, or failover bypasses authorization,
  current validation, freshness, version, suppression, deletion, or license
  policy.
- Collect and disclose the minimum personal/location data for an approved
  purpose and delete it through every derived copy and processor.
- Build once from reviewed immutable inputs, establish provenance, and
  promote the same verified artifact between environments.

### 1.2 Required decision inputs

The following remain TO BE DECIDED and block the associated implementation:

| Input | Required before |
|---|---|
| Canonical domains, admin origin, app IDs, trusted-link domains and WebAuthn RP IDs | Authentication or trusted links |
| Node.js framework/version and React rendering/version | Server/web implementation |
| Browser and mobile session/token design | Authenticated client implementation |
| WebAuthn library, admin authenticator assurance, bootstrap, recovery and break glass | Administrative implementation |
| Complete role/action/field/approval matrix | Administrative implementation |
| Provider, map/geocoder, notification and other processor selections | Corresponding integration |
| Store, queue, cache, object, audit and backup technologies | Persistence implementation |
| Azure compute/network/identity/region/key/secret topology | Infrastructure implementation |
| CI/CD, registry, Terraform state and signing/provenance tooling | Production-bound merge/deploy |
| Numeric query/import/fan-out/cost/SLO/RPO/RTO/staleness limits | Performance/topology freeze |
| Privacy jurisdiction, purpose/basis, retention, processors/transfers, minors and analytics | Personal-data collection |

Each decision must meet ARCHITECTURE.md §9 rather than merely name a product.

---

## 2. Governance and Secure Delivery

### 2.1 Change control

- REQUIREMENTS.md, ARCHITECTURE.md, SECURITY.md, DESIGN.md, authorization
  policy, AQI/freshness/guidance rules, CI workflows, Terraform, dependency
  policy, and production configuration require protected history and review
  by an authorized owner.
- Branch protection, CODEOWNERS or equivalent ownership, required reviews,
  required CI checks, deployment approvals, emergency bypass, and rollback
  must be defined before production-bound implementation is merged
  (NFR-MAINT-008, TM-04, TM-16, TM-24).
- Security-sensitive changes require an independent reviewer who did not
  author the change. Rule/config changes with global data impact also require
  air-quality-domain or designated data-policy approval.
- Direct production changes are prohibited except a declared, audited,
  time-limited break-glass action. Reconcile any emergency change back to
  version-controlled state and review it after the incident.

### 2.2 Threat-model lifecycle

- Re-run NFR-MAINT-007 before architecture freeze, production, and any new
  trust boundary, privileged action, data class, provider/processor,
  authentication method, region/topology, public calculation, incident, or
  material dependency change.
- Maintain threat ID, affected asset/flow, inherent and residual risk,
  controls, evidence, owner, due date, status, review date, and risk
  acceptance. Preserve cross-functional disagreements.
- No Critical or High item is silently deferred. SEC-IR-003 requires an
  accountable owner, independent approval, compensating controls, expiry,
  and verification plan.
- The current model is a single-analyst documentation baseline. Product,
  security, privacy/legal, air-quality domain, engineering, mobile/client,
  accessibility, and operations review are a production gate.
- Generate and validate the NFR-MAINT-008 requirement → architecture
  invariant/flow → security control → release-test matrix. A new normative
  requirement, flow, threat, or control with no mapping fails documentation
  and production-bound CI rather than relying on a manual dangling-reference
  check alone.

### 2.3 Security issue and incident readiness

- Maintain a monitored private channel for vulnerability reports and a
  coordinated-disclosure process (SEC-IR-002).
- Maintain owners and runbooks for admin takeover, provider poisoning, false
  or stale data, personal-location disclosure, notification storms, leaked
  credentials/signing keys, dependency/CI compromise, denial of service,
  ransomware, and regional failure (SEC-IR-001).
- Incident access uses ADMIN-011 break glass; evidence preservation and
  public-data correction/withdrawal are part of containment.

---

## 3. Security Zones, Workloads, and Data Services

### 3.1 Zone enforcement

Implement the zones and TB-01–TB-16 in ARCHITECTURE.md:

- Internet edge, administrative ingress, application, ingestion/worker,
  private data, security/operations, delivery/cloud management, environment,
  and region boundaries use explicit default-deny policy.
- Databases, caches, queues, object/quarantine stores, secret/key stores,
  audit, observability, backups, and Terraform state have no default public
  data-plane endpoint.
- A public/admin URL prefix is routing only. The admin client/API also uses a
  distinct origin or independently enforced gateway, authentication audience,
  session namespace, workload path, and operation-level authorization.
- Production identities, secrets, state, and data are separate from
  development/test/staging. Production personal or provider data is not
  copied down without an approved minimized or synthetic process.

### 3.2 Workload identity and east-west authorization

- Each service, worker, topic producer/consumer, deployment job, backup, and
  restore workflow uses a distinct short-lived or managed identity
  (DATA-004, SEC-AUTHZ-007, TM-12/14/17).
- Grant only required action/resource scope. Public API cannot write trusted
  observations; fetch/parser workers cannot access cloud control plane or
  unrelated secrets; ordinary runtime cannot delete backups or audit.
- Internal non-loopback traffic carrying data or credentials uses approved
  authenticated encryption. A service validates the intended caller,
  audience, action, and resource even when transport or network policy also
  authenticates the peer.
- Authorization or identity-policy uncertainty fails closed for mutations
  and protected reads. Availability fallbacks must not create a bypass.
- Shared production cloud, database, queue, cache, or secret credentials are
  prohibited.

### 3.3 Queue and job security

- Producers are authorized per topic/queue; consumers are authorized per
  operation and data scope. Broker possession alone is not authority.
- Every envelope includes schema version, message type, correlation ID,
  idempotency key, creation and expiry times, and a bounded payload. Producer
  identity and authorized scope come from broker-authenticated context or a
  verified message signature; self-asserted envelope claims never confer
  authority. Do not place credentials or unnecessary PII in messages.
- Consumers validate envelope and payload independently, reject unknown
  fields where security-relevant, enforce authorization, and tolerate
  duplicate/out-of-order delivery without state regression.
- Retry attempts, total elapsed time, queue age/depth, concurrency, fan-out
  and payload size are bounded. Poison/expired messages go to a restricted
  dead-letter path with redacted diagnostics and operator alerts.
- Reprocessing repeats current validation and authorization and preserves the
  original event/outcome. No manual replay writes directly to trusted state.

### 3.4 Store, cache, backup, and key boundaries

- Apply ARCHITECTURE.md §4.5 store separation even if multiple roles share an
  engine: separate identities, schemas/namespaces, authorization, network
  policy, keys, retention, backup, and audit.
- Confidential/Restricted data is encrypted at rest and in backups, queues,
  caches, object storage, logs, exports, and Terraform state. Keys are
  environment-scoped, least-privilege, rotatable, and access-audited
  (DATA-003).
- Shared/public caches hold only explicitly Public, publication-eligible
  responses. Cache keys cover every representation dimension, including
  standard, provider/source policy, locale where meaningful, query bounds,
  and data/rule version. Authenticated/admin/recovery/personal data is
  private/no-store and never enters a shared cache.
- Only trusted loaders write public cache entries. TTL never upgrades stale
  data; correction/suppression/rule events version or invalidate every
  affected entry. Prevent cache stampedes and cap fill work.
- Backups use an identity and deletion authority separate from runtime,
  encryption, version/immutability, integrity verification, retention, and
  isolated restore. Restore tests prove that revoked credentials, deleted
  users, old roles, suppressed data, and stale alerts are not resurrected
  (REL-004–008, TM-20).

### 3.5 Trusted time and ordering

- Authentication/session expiry, WebAuthn and recovery artifacts, provider
  freshness, message/callback replay windows, notification order, audit
  order, retention, certificates, and key validity use approved synchronized
  time sources under SEC-008. Duration and timeout measurement uses a
  monotonic clock where the platform provides one.
- Define and enforce maximum skew per workflow. Detect unsynchronized nodes,
  backward/forward jumps, source loss, and cross-region disagreement. A
  component must not extend validity, publish data as newer, reorder security
  evidence, or accept a replay because its wall clock is untrusted.
- Time-source and identity/key-plane failure modes must be capacity-bounded,
  monitored independently, exercised in failover tests, and fail closed for
  protected mutations while preserving safe incident access.

---

## 4. Provider Ingestion and Public-Data Integrity

These controls address TM-01/02/05/11/23/24 and are safety requirements, not
optional data-quality improvements.

### 4.1 Provider onboarding and destination control

- Approve provider ownership, license/attribution/redistribution rights,
  endpoint ownership, current TLS behavior, authentication, schema, update
  cadence, integrity/checksum/signature capability, rate/size limits,
  incident contact, and compromise/revocation plan before publication.
- Provider endpoints come only from reviewed configuration. Provider
  payloads, request parameters, redirects, or file contents never select a
  destination or credential.
- Apply the complete SSRF/egress rules in §7.2 to scheduled polls, backfills,
  preview/test retrievals, bulk files, redirects, callback verification, and
  reprocessing.
- Fetch identities have only the provider credentials they need and no broad
  secret, data-store, audit, backup, or cloud-management permission.

### 4.2 Quarantine and immutable provenance

- Write the exact raw response or license-permitted canonical equivalent to a
  private non-executable origin before parsing; record provider identity,
  destination, retrieval time, headers needed for provenance, adapter/parser
  version, size, and cryptographic digest (ING-004).
- Retain only allowlisted provenance headers. Never persist provider or client
  Authorization/Cookie values, private keys, client certificates, bearer
  tokens, or other transport credentials with a raw response.
- Never render provider HTML, load remote provider assets, serve an uploaded
  object inline from an application origin, or open an archive on an admin
  workstation. Use a safe text/structured viewer and attachment/nosniff
  responses where downloads are approved.
- Strictly bound compressed/expanded bytes, records, nesting, filenames,
  memory, CPU, storage, network, duration, redirects, and concurrency.
  Reject traversal, symlink/hard-link escape, overlapping entries, external
  entity resolution, executable serialization, and unsupported formats.
- Invalid or ambiguous data remains quarantined. Quarantine access is
  least-privilege and audited; reprocessing does not skip current controls.

### 4.3 Publication eligibility

- Validate source/authenticity evidence, strict schema, type, finite numeric
  values, coordinates, timestamps, units, averaging period, pollutant,
  station/provider scope, duplicates, replay/order, freshness, semantic
  range, anomaly, conflicts, license, suppression, and provenance before
  publication (ING-010/015–017, AQ-009).
- Syntactic validity is not trust. Mandatory anomaly/reconciliation compares
  provider/source history and configured scientific policy, records
  explainable reasons, and alerts both on high-impact anomalies and unexpected
  validator silence.
- Preserve source values. Conversion/aggregation is deterministic and
  references immutable inputs, adapter/rule version, authoritative source,
  effective period, tests, reviewer, and approval. Do not combine standards,
  units, averaging periods, locations, or timestamps without an explicit
  reviewed rule.
- Only the publication authority writes trusted versions. Its current-data
  writer is exclusive; if approved, a separately authenticated aggregation
  writer may write only derived historical/analytical versions from eligible
  immutable inputs and cannot write current state or emit alert eligibility.
  Failure preserves the last eligible version under its true freshness state;
  expired data becomes unavailable.
- Corrections/suppressions keep the original, actor, reason, evidence,
  approval, effect, and restoration history and invalidate/version all
  derived data, caches, API output, and pending notifications.

### 4.4 Alerts

- Evaluate only a committed publication-eligible observation. Include
  observation identity/source, rule version, prior alert state, evaluation
  time, expiry, and idempotency key.
- Out-of-order, replayed, stale, expired, suppressed, or corrected data cannot
  create a false threshold crossing or recovery. Recovery requires a prior
  committed trigger for the same alert.
- Check recipient verification, current opt-in, destination, and preference
  at send time. Enforce per-user, destination, channel and global fan-out/
  retry budgets and isolate a failing vendor.
- Delivery-status and provider callbacks follow SEC-API-009: authenticate the
  sender, bind environment/account/type, validate schema and scope, enforce a
  trusted-time replay window plus idempotency, and never treat provider
  acceptance as proof of device delivery.
- Alerts are informational, may be delayed or unavailable, and are not a
  government/emergency service. UI exposes observation/delivery time and
  failure/delay status without making an unsafe guarantee.

---

## 5. Authentication and Session Security

### 5.1 General authentication

- Use maintained standards-based libraries and platform cryptography; no
  custom authentication, session, challenge, token, signature, or random
  generation protocol (SEC-AUTH-001, SEC-006).
- Construct verification/reset links from an approved canonical origin, never
  an untrusted Host/forwarded header. Tokens are CSPRNG-generated, single-use,
  short-lived, account/action-bound, digest-only at rest, atomically consumed,
  and invalidated when replaced.
- Externally visible login/registration/reset/verification/recovery behavior
  must not reveal whether an account exists beyond an approved workflow.
- Distributed throttles and risk signals cover account, credential,
  network/device and global patterns without permanent lockout based on a
  spoofable signal. Monitor password spraying, credential stuffing, reset
  floods, WebAuthn failures, token races, and unusual authenticator changes.

### 5.2 Passwords

- Hash passwords with an approved adaptive memory-hard algorithm using
  security-owned, benchmarked parameters; Argon2id is the provisional default
  pending the cryptographic/platform compatibility ADR. Store parameters with
  the hash and allow rehash on successful authentication. A pepper, if used,
  remains in the key/secret boundary and has a rotation plan.
- Require at least 15 characters, accept at least 64, document/test Unicode
  handling, support password-manager paste/autofill, and screen new/changed
  passwords against known-compromised values. Never silently truncate.
- Do not use security questions, arbitrary composition rules, or periodic
  expiry without compromise evidence. Online guessing controls are separate
  from password hashing.
- Password change requires recent authentication, notifies the owner, rotates
  the current session, and revokes every other session and renewable
  credential under USER-010.

### 5.3 WebAuthn/passkeys

- Every consumer and administrative WebAuthn ceremony must verify challenge,
  ceremony type, RP ID hash, exact approved origin, account/user handle,
  credential ownership/status, signature, approved algorithm, and the policy-
  required user-presence/verification flags on the server. Administrative
  WebAuthn always requires user verification; a consumer policy may not skip
  protocol checks or weaken recovery merely because passkeys are optional.
- Challenges are unpredictable, short-lived, and transaction/account bound;
  atomically consume a challenge after the first completed verification
  attempt. Rate-limit issuance and failed attempts separately. Do not trust
  client-provided credential metadata, UV flags, origin, or challenge state
  without protocol verification.
- Administrative identities are invite-only and separate from consumer
  registration/recovery. Password, email link, SMS, security question, or
  support assertion alone cannot enroll, replace, recover, or authenticate an
  admin.
- Before routine production access, an administrator must have the approved
  independent authenticator/recovery coverage so loss of one device does not
  create a weaker fallback. Exact authenticator and sync/attestation policy is
  a §1.2 decision.
- Adding, replacing, or removing an admin authenticator, recovery, role
  change, or ADMIN-010 action requires a fresh WebAuthn assertion for the
  exact admin origin, approval by a distinct authorized person, and an
  independent security notification/audit event.
- Admin bootstrap/recovery requires two authorized people; no requester
  approves their own action. Break glass follows ADMIN-011 and never becomes a
  silent normal login path.

### 5.4 Browser sessions and CSRF

- Use an opaque, unpredictable, server-revocable browser session. Rotate its
  identifier after authentication, recovery, privilege or authenticator
  change; never place it in URL, referrer, log, analytics, notification, or
  JavaScript-accessible persistent storage.
- Cookies use Secure, HttpOnly, no Domain, and a SameSite value justified by
  the flow. On a dedicated origin, prefer a __Host- cookie with Path=/; if a
  narrower Path is required, use a different reviewed naming/scope policy
  rather than claiming __Host- semantics.
- SameSite is defense in depth, not the sole CSRF control. State-changing
  browser requests use a framework-supported, session-bound unpredictable
  anti-CSRF token plus strict Origin validation and Fetch Metadata where
  supported. Reject state changes by GET and requests with ambiguous/missing
  origin context unless a documented non-browser protocol applies.
- Idle/absolute lifetimes, logout, device/session listing, revocation, and
  distributed propagation are mandatory. Admin sessions have a separate
  audience/namespace, shorter lifetime, no “remember me” bypass, and fresh
  WebAuthn for each high-impact approval.

### 5.5 Mobile sessions

- The mobile session/token protocol is an architecture decision. It must use
  short-lived audience/issuer-bound access, rotation or reuse detection for
  renewable credentials, server-side revocation, secure platform storage,
  and cleanup on logout/account deletion.
- Bearer credentials never enter general storage, logs, URL/deep link,
  notification payload, WebView, clipboard, crash report, or device backup.
- Users can identify and revoke a lost mobile session; recovery or a security-
  sensitive account change invalidates affected renewable credentials.

---

## 6. Authorization and Privileged Operations

### 6.1 User and resource authorization

- Deny by default on every protected operation and field. Validate the
  authenticated principal, action, object ownership/scope, field set, current
  state, and any required approval server-side.
- Owner scope covers profile/email, authenticators/recovery, sessions, saved
  locations, preferences, alerts, notification destinations/history,
  personal exports, and deletion requests (SEC-AUTHZ-003).
- Use explicit input/DTO field allowlists. Never bind user input directly to
  a persistence model or trust a client owner/role/status field.
- Collection, bulk, nested, export, search, status, and indirect identifier
  endpoints enforce the same object/field/function rules as single-resource
  operations.
- Automated negative tests enumerate every disallowed role-resource-action,
  cross-user ID, field, nested/bulk operation, state transition, and admin
  function (SEC-AUTHZ-006, TM-06).

### 6.2 Administrative authorization

- Maintain an approved operation matrix for Support, Data, Security,
  Application System Administrator, Read-Only Auditor, SRE/incident,
  approval, and break-glass identities. Cloud, CI, signing, backup, key, and
  infrastructure operators use separate workforce identities. Unmapped
  operations are denied.
- Read-Only Auditor cannot mutate. Support sees masked minimum data. Data
  administrators cannot grant roles or change audit policy. Application
  System Administrators have no routine public-data editing authority and no
  inherited cloud/CI/Terraform/signing/backup/key authority. Security
  administrators cannot approve their own request.
- Role grants, admin recovery/auth policy, provider destination/credential,
  AQI/freshness/guidance/source policy, any correction/suppression meeting the
  versioned safety-impact threshold, personal export, and security/audit-
  control changes require fresh WebAuthn, reason, bounded preview/scope, a
  distinct authorized approver, rollback, durable audit, and alerting
  (ADMIN-010). Geography alone never exempts a safety-significant action.
- SRE, cloud-management, backup/restore, signing, and break-glass privileges
  must be time-limited/JIT. If a selected platform cannot enforce this, it
  does not satisfy the architecture gate without an approved replacement
  control and SEC-IR-003 acceptance. Privilege expiry, suspension, and
  termination revoke sessions and workload/cloud grants promptly.
- Secret values are write-only after entry. Audit queries/exports and masked-
  field reveals are privileged, purpose-bound, and audited.

---

## 7. Boundary Validation, SSRF, and Content Safety

### 7.1 Schema and parser safety

- Validate body, query, path, header, cookie, provider response, queue
  message, cache entry, file/archive, deep link, native bridge, webhook, and
  API response at the first trust boundary and again before a different
  security interpretation.
- Positive schemas enforce type, length, finite range, count, nesting,
  Unicode/control-character, normalization, enum, relationship, and unknown-
  field policy. Reject prototype-pollution keys, unsafe polymorphic types,
  external entities, executable deserialization, numeric overflow/non-finite
  values, and algorithm/destination selectors not granted by policy.
- Database operations use parameter binding or an ORM interface that
  preserves parameterization. Authorization and field allowlists remain
  explicit; an ORM is not an authorization control.
- Untrusted data does not form a module path, class/type name, template,
  regular expression without complexity controls, shell command, SQL/NoSQL
  expression, filesystem path, or dynamic code.

### 7.2 Outbound request and SSRF controls

- Normalize and allowlist HTTPS scheme, hostname, port, and path where
  practical. Reject userinfo, fragments when not needed, encoded/ambiguous
  authority, mixed encodings, unsupported IP notation, and scheme/port
  downgrade.
- Resolve through controlled DNS and verify every resolved/connected IPv4 and
  IPv6 address. Block loopback, private, link-local, multicast, unspecified,
  reserved, carrier-grade NAT where policy requires, and cloud metadata/
  control endpoints.
- Disable redirects unless required. When required, cap them and repeat
  scheme/host/port/DNS/IP policy on every hop; never forward Authorization,
  cookie, provider credential, client certificate, or sensitive header to a
  changed origin.
- Enforce both application checks and default-deny network egress. Pin the
  function's allowed destinations/protocols; log destination identity and
  policy outcome without secrets.
- Bound connect/read/total time, bytes, expanded bytes, response count,
  concurrency, retries and DNS behavior. Rebinding or address changes must not
  bypass connect-time enforcement.
- Scheduled, test, preview, callback, webhook, bulk, image, map, geocoder, and
  admin-triggered requests use the same egress path. Do not create a
  privileged “test URL” exception.

### 7.3 Output, URL, and file safety

- Use context-safe APIs or maintained serializers for the final HTML,
  attribute, URL, JSON, CSS, email, push, log, terminal, CSV/spreadsheet, and
  native-view context. Do not manually construct JSON or assume that escaping
  for one context is valid for another.
- Provider/station/location names, attribution, administrative annotations,
  health guidance, filenames, links, and notification content are untrusted.
  Apply length/control-character policy and show external destinations
  clearly.
- Do not render raw HTML. If an approved feature truly needs rich content,
  isolate one reviewed sanitizer/rendering component, disable active content
  and raw HTML by default, validate links after rendering, and test bypasses.
  Markdown alone is not a sanitizer.
- Context-specific URL policy must allow only necessary schemes and approved
  destinations. Do not generally trust external mailto or tel links; require
  a user-initiated feature and validate their contents.
- Safe file extraction uses an isolated newly created directory, rejects
  absolute/traversal/link/device entries, applies no-follow primitives where
  available, prevents overwrite/races, and never executes or serves extracted
  content from an active application origin.

---

## 8. HTTP, API, and React Web Security

### 8.1 HTTP/API boundary

- Production and non-loopback environment traffic carrying data or
  credentials uses approved TLS with current certificate/hostname validation;
  no cleartext fallback.
- Normalize and validate Host, forwarded headers, client IP and scheme only
  from explicitly trusted proxies. Use canonical configured origins for
  security links and CORS decisions.
- CORS uses the minimum exact origin/method/header allowlist; never wildcard
  with credentials and never treats CORS as authentication.
- Accept only documented methods and media types. State changes never use
  GET. Enforce request and response schema, body/header/URL size, timeout,
  pagination, geographic/date/result/cost/concurrency and output limits.
- The reviewed machine-readable API specification (OpenAPI when selected) is
  the production route allowlist.
  CI/runtime checks find undocumented, debug, sample, deprecated, or
  unprotected routes and schema/auth drift (API-011, SEC-API-008).
- Error responses are stable and generic with a correlation ID. They never
  disclose stack trace, source path, SQL, internal host/topology, dependency
  detail, credential, token, secret, raw provider payload, personal data, or
  configuration.
- Set content-type/nosniff, clickjacking/frame policy, referrer policy,
  permissions policy, and a strict CSP appropriate to the selected rendering
  strategy. HSTS is required after HTTPS readiness; includeSubDomains/preload
  requires explicit ownership and rollback review for every subdomain.

### 8.2 React client

- Use ordinary framework text interpolation for untrusted text; do not create
  element/tag names or spread untrusted props into DOM nodes.
- The only approved raw/rich-content component follows §7.3 and is located by
  an automated source check. Direct dangerous HTML sinks, dynamic code,
  string event handlers, eval, and Function construction are prohibited.
- Validate externally supplied href, src, form action, navigation, CSS and
  redirect destinations with context-specific scheme/host/path rules. New
  tabs isolate the opener.
- Validate API responses before rendering and use explicit loading/error/
  ready/unauthorized/stale states. Client validation never substitutes for
  server validation.
- postMessage verifies exact origin/source and schema; never uses wildcard
  target origin for sensitive data.
- Do not put email, precise/saved location, raw account ID, session, or other
  sensitive values in DOM IDs, analytics labels, URL, referrer, source maps,
  client logs, or reorderable list keys.
- Service workers and browser caches follow API-012. They do not cache
  authenticated/admin/recovery responses and purge versioned data after
  correction, logout or account deletion as applicable.
- Third-party runtime scripts, fonts, map SDKs and remote configuration
  require SEC-007 and SEC-SUP-001–004 review, CSP compatibility, integrity/provenance, data-
  flow documentation and a failure mode. Prefer pinned/self-hosted static
  assets when it reduces tracking and supply-chain risk.

---

## 9. React Native and Mobile Security

- Store authentication material only through approved platform-backed secure
  storage. General async/preferences storage, files, logs, crash reports,
  clipboard, WebViews and JS bundles are not secret storage.
- Classify local databases/files/caches, set a bounded retention, exclude
  Confidential/Restricted state from device/cloud backup, and purge affected
  state on logout, account deletion, permission revocation and expiry
  (SEC-MOB-006). If backup exclusion cannot be enforced for a storage
  location, Restricted data must not be persisted there.
- Recently viewed Public observations may be cached; saved labels/search
  history or association with an account are Confidential. Do not create a
  precise location history.
- Request foreground location just in time for a user action. Do not request
  background/continuous location in v1. Denial/revocation preserves manual
  search. Reduce precision/on-device process where the feature permits it.
- Validate deep/universal links, custom schemes, intents, push navigation,
  share input, native-module bridge values and WebView messages on the native
  receiving side. Allowlist scheme/host/path/action; reauthorize personal or
  privileged destinations.
- WebViews, if approved, use exact origin navigation, no arbitrary file/
  universal access, no auth token injection, strict message schema, and
  external-browser isolation for untrusted links.
- iOS and Android production network policy rejects cleartext and arbitrary
  transport exceptions. Any certificate pinning proposal requires an
  availability/rotation/recovery design; TLS validation remains mandatory.
- Protect sensitive screens from task-switcher snapshots. If a supported
  platform cannot enforce that protection, obscure or remove Restricted data
  before the app backgrounds. Do not automatically copy sensitive data. Treat
  rooted/jailbroken detection only as a risk signal, never the sole security
  boundary.
- Mobile packages and any remote code/config update are signed and
  provenance-verified for the intended app/environment and distributed
  through approved channels. Protect signing keys and test compromise,
  rotation, rollback and forced security updates.

---

## 10. Privacy and User-Location Protection

### 10.1 Processing inventory and minimization

- DATA-001 and SEC-PRIV-007 must map every personal field from collection
  through client, API, service, queue, cache, log/trace, store, backup,
  support/admin view, export, and processor to deletion.
- Exact/search/saved/default locations, custom location names, alert rules,
  notification destination/history, IP/device/push IDs, and account-location
  linkage are personal data. Precise durable location is Restricted.
- Collect only for an approved purpose. Do not add background location,
  contact access, advertising identifiers, cross-service identifiers or
  speculative analytics.
- Nearby lookup must use the approved minimum-disclosure pattern—on-device,
  reduced precision, or a privacy-preserving proxy—selected at the F-28 gate.
  Never send a vendor both a stable account identifier and precise location
  unless a separately approved purpose cannot be achieved otherwise.

### 10.2 Notice, choice, and processors

- Provide layered plain-language notice before collection and just-in-time
  permission for location, notifications and any optional analytics. State
  purpose, fields, retention, recipients, transfer, background behavior,
  revocation and deletion.
- Permission or notification denial cannot block manual public-data use.
  Consent withdrawal stops future optional processing and propagates cleanup.
- Before any map/geocoder/push/email/analytics/observability/support/cloud
  processor receives personal data, document controller/processor role,
  minimum fields, purpose/basis, jurisdiction/transfer, retention, onward use,
  security, incident terms, deletion and exit plan. Execute required
  agreements before transfer.
- Resolve applicable privacy jurisdictions, DPIA/impact assessment, minors/
  minimum-age policy and analytics design before collection. Global intent
  makes deferral until a “confirmed” user base unsafe.

### 10.3 Rights and deletion

- Access/export/correction/deletion requires authenticated ownership and
  recent authentication for sensitive or destructive action. Prevent CSRF,
  BOLA, enumeration, duplicate/racing completion and export-link leakage.
- Exports are minimal, encrypted or otherwise protected as appropriate,
  short-lived, single-account, non-indexable, and access-audited. Do not email
  a bulk personal-data attachment or bearer URL without approved protection.
- Deletion propagates to active credentials/sessions, profile, saved
  locations, alerts, notification destinations/history, caches, queues,
  indexes, analytics, exports, local state under app control, and processors
  within the approved deadline.
- Audit/legal/backup exceptions are purpose-limited, access-restricted and
  disclosed; expiry removes or irreversibly de-identifies the subject. Rights
  evidence records outcome without copying exported/deleted content.

### 10.4 Notification privacy and anti-phishing

- Default lock-screen/push/email-subject content is generic and uses a
  non-sensitive label. Never include precise coordinates, user-defined
  labels, account ID, token, password/reset prompt, or sensitive routine.
- Links use a canonical allowlisted HTTPS/universal/app-link origin, contain
  no bearer credential or sensitive query data, display their destination,
  and authenticate/authorize inside the app before personal content.
- Users may opt into more descriptive previews only after a clear bystander/
  lock-screen disclosure warning.
- Email channels must use the approved sending domains and authentication
  controls (including SPF, DKIM and DMARC policy appropriate to rollout).
  Messages never ask users to disclose a password, passkey, code or secret.

---

## 11. Cryptography, Secrets, and Configuration

- Use platform or maintained approved cryptographic implementations and a
  security-owned algorithm/protocol policy. Custom crypto, disabled
  certificate validation, weak fallback/downgrade and hard-coded keys are
  prohibited (SEC-006).
- Generate tokens, identifiers requiring unpredictability, keys and nonces
  with the platform CSPRNG. Do not use a general-purpose pseudo-random
  function for security values.
- Prefer managed/workload identity over secrets. Remaining provider,
  database, signing, encryption and API credentials live in the approved
  secret/key service, referenced by identifier rather than copied into
  configuration.
- Define owner, purpose, consumers, creation, delivery, version, rotation,
  revocation, expiry, access review, compromise response, recovery and
  destruction for every secret/key. Test rotation without outage or loss.
- Scope secrets/keys per service, environment and purpose. Log reads and
  administrative changes, alert unusual access, and prevent secret reveal
  after entry.
- Source, client bundles, mobile packages, logs/traces, crash reports, images,
  artifacts, Terraform plans/state/output, test fixtures and support exports
  must pass secret-leak tests. Marking a Terraform value sensitive does not
  remove it from state.
- Security-sensitive configuration uses strict schema, safe defaults,
  immutable/versioned release, review/approval, integrity/provenance, audit,
  rollback and drift detection. Ordinary feature flags cannot disable
  authentication, authorization, audit, encryption, privacy enforcement,
  validation, or publication gates. A separately modeled emergency override
  requires break-glass authority, expiry, alerting, rollback, and durable
  evidence and must never bypass authorization or evidence collection.

---

## 12. Logging, Audit, Detection, and Error Handling

### 12.1 Safe telemetry

- Log security and data-integrity events in SEC-LOG-001 plus authenticator
  lifecycle, admin recovery/break glass, role/approval, provider endpoint/
  credential, rule/guidance, quarantine/publication/replay, export/deletion,
  CI/deployment, key/secret and backup/restore events.
- Use structured fields and escape untrusted values to prevent log forging.
  Reject or neutralize control characters and bound field size/cardinality.
- Never log passwords, passkey assertions/challenges where replay or privacy
  risk exists, session/refresh/reset/verification tokens, cookies/
  Authorization headers, secrets/keys, precise location, personal export,
  raw provider payload, full request body, Terraform secret/state, or
  unsanitized exception objects.
- Do not “log the full error.” Build a sanitized diagnostic object with
  allowlisted fields. Correlation may use an opaque non-secret ID; never a raw
  session or stable unnecessary user/location identifier.
- Apply classification, least-privilege access, retention, sampling,
  deletion/pseudonymization, export monitoring and regional policy to logs,
  metrics, traces, baggage and dashboards.

### 12.2 Audit integrity

- Administrative and other high-impact events include actor/workload,
  authenticated session/approval context, action, target/scope, reason,
  outcome, safe allowlisted before/after diff, trusted timestamp/order and
  correlation ID. Never copy a secret or unnecessary PII into a diff.
- Application writers have append-only/write-only authority. Audit read,
  export, retention and administration use separate identities; the actor
  performing an operation cannot alter its evidence.
- Use immutable/WORM retention appropriate to the selected store **and**
  cryptographic tamper evidence anchored outside the ordinary writer's
  authority. Verify immutability, integrity, and order on an approved
  schedule.
- Audit every read/export and alert on gaps, failed writes, clock anomalies,
  integrity failures, attempted alteration/deletion, disabled collection and
  unusual bulk access.
- A high-impact mutation fails closed if durable audit cannot be committed.
  A reviewed emergency durable buffer may preserve availability only when it
  maintains integrity/order and immediately alerts for reconciliation.

### 12.3 Detection and response

- Every TM-01–TM-24 High/Critical path maps to preventive controls and at
  least one detection or explicit explanation of why detection is infeasible.
- Each alert has severity, owner, response target, runbook, escalation path
  and evidence rule. Monitor alert delivery and on-call access; silence or a
  disabled rule emits through an independent path.
- Health/readiness endpoints expose coarse liveness publicly at most.
  Dependency, topology, version/build, queue, storage and diagnostic status is
  private, authenticated and authorized.
- Exercise detection and incident runbooks before production and
  periodically. Record mean detection/response evidence without claiming an
  unmeasured guarantee.

---

## 13. Azure, Terraform, CI/CD, and Supply Chain

### 13.1 Source, dependencies, and builds

- Protected source and workflow definitions require independent owner review.
  Untrusted pull requests/forks run without production secrets, signing keys,
  trusted cache poisoning ability or deployment identity.
- Allowlist and authenticate package/container registries and other artifact
  sources. Pin third-party build actions, packages where the ecosystem permits
  immutable resolution, images, Terraform providers/modules, and toolchains
  to reviewed immutable versions/digests. A source that cannot provide
  immutable, integrity-verifiable resolution requires an explicit replacement
  control or must not enter the production build. Detect dependency confusion,
  typosquatting, lockfile tampering, and unexpected lifecycle scripts.
- Build in isolated ephemeral jobs from a locked dependency graph. Produce an
  SBOM, security/license scan evidence, signed provenance/attestation and
  integrity-verifiable artifact. Protect build cache and registry write
  permissions.
- Promote the same verified artifact between environments. Production
  approval binds exact source, artifact digest, Terraform plan and target
  environment; do not rebuild after approval.
- Use short-lived federated/workload deployment identity, not long-lived
  general cloud credentials. Separate build, plan, apply, signing, promotion,
  runtime, backup and break-glass authority.

### 13.2 Terraform and Azure controls

- Terraform state, plans, outputs and variable files are potentially
  Restricted. Encrypt, version, lock, access-log, back up and isolate them by
  environment; do not expose them to untrusted CI or ordinary runtime.
- A production apply requires policy/security checks, reviewed immutable plan,
  protected environment approval, scoped identity and target validation.
  Detect drift and reconcile approved emergency changes.
- Default-deny public data-plane access and use private paths for data,
  queue/cache, object, audit, backup and secret/key services unless a specific
  approved threat model proves a public path necessary.
- Enforce ingress and egress policy, managed identity protection, Azure
  metadata-path defense, least-privilege roles, separate environments/
  subscriptions or equivalent boundaries, resource locks where appropriate,
  encryption, patching, diagnostics, cost quotas and required ownership/
  classification tags.
- Backups, keys/secrets, audit and security telemetry use authority separate
  from the workload and from a single general cloud administrator.
- Before any destructive infrastructure or data change, preview exact scope,
  require approval proportional to impact, preserve recovery, and audit.

### 13.3 Vulnerability and exception policy

- Choose dependencies by supported lifecycle, publisher/provenance,
  maintenance/security process, license, dependency graph, platform fit,
  exploitability/reachability and tested known-good version—not arbitrary
  “latest major,” release-age or dependency-count rules.
- Prefer mature security libraries for authentication, cryptography, parsing,
  sanitization and protocol handling over ad hoc first-party code.
  Dependency minimization must not encourage custom security mechanisms.
- Define release-blocking severity/exploitability policy and remediation
  deadlines. Scan direct/transitive dependencies, runtime/container/mobile
  packages, IaC, source, secrets and artifacts continuously and on release.
- An exception identifies the finding, affected path/data, exploitability,
  controls, owner, expiry and independent approver. Re-evaluate on dependency
  or threat change.
- Unsupported runtimes/frameworks/dependencies and known malicious packages
  are prohibited.

---

## 14. Availability, Abuse, and Recovery

- Classify endpoints/jobs by authentication, data sensitivity, computational/
  vendor cost and safety criticality. Enforce request, result, range,
  geographic area/resolution, page, filter/sort, concurrency, memory, CPU,
  queue, storage, retry, fan-out and global cost budgets at the earliest
  practical boundary.
- Combine account/token, abuse-resistant network/device and global controls.
  A spoofable IP or username alone must not permanently lock out a victim.
- Bound provider retry attempts and elapsed time, use jitter/circuit breakers,
  isolate providers/workloads, and reserve capacity for current trusted data,
  authentication, administration, audit and security response.
- Limit backfill scope/concurrency, provide cancellation, enforce provider
  rates and idempotency, and prevent starvation of interactive/current paths.
- Protect cache fills from stampede, queue/storage/logs from exhaustion,
  notifications from fan-out/retry storms, and auto-scaling from unbounded
  cost. Alert before capacity and spend limits.
- Degraded operation exposes true freshness/delivery status. It never labels
  stale data current, issues alerts from expired data, silently disables
  authorization/audit, or exposes a more privileged fallback.
- Approve numeric SLO, RPO, RTO, maximum public-data age and alert delay before
  topology selection. Exercise provider failure, dependency latency, DDoS/
  cost abuse, retry storms, regional loss, ransomware, restore, and false-data
  withdrawal (NFR-AVAIL-001–005, REL-004–008).

---

## 15. Stack-Specific Coding Baseline

These are capability rules. Framework/library-specific profiles are selected
only after the corresponding ADR and must not weaken them.

### 15.1 Node.js / TypeScript (conditional)

- These rules apply only if the Node.js/TypeScript ADR in ARCHITECTURE.md §9
  is approved. Use a supported Node.js release and strict TypeScript
  configuration. Avoid unsafe any or unchecked type assertions at boundaries;
  runtime schema validation remains mandatory.
- Do not merge untrusted keys into ordinary object prototypes. Reject
  __proto__, prototype and constructor manipulation; use Map or null-
  prototype dictionaries when accepting dynamic keys.
- Do not evaluate untrusted code or dynamic modules. eval, Function,
  dangerous VM execution, string-built shell commands and require/import from
  input are prohibited.
- When an OS process is unavoidable, invoke a fixed executable with an
  argument array, minimal environment/working directory, no shell, resource
  limits and least-privilege identity.
- Configure server proxy/Host behavior, body/header/URL limits, parsing,
  request/response timeouts, cancellation, error mapping, security headers,
  CORS, CSRF, rate/cost limits and graceful shutdown centrally and test that
  routes cannot bypass them.
- Use structured logging with recursive field/header/body redaction and test
  error paths. Production console output is not an unreviewed bypass.
- Files/archive handling follows §7.3; a path.resolve prefix check alone is
  not sufficient against symlink/race attacks.
- Use current platform CSPRNG and constant-time comparison where secret-value
  comparison is required by a reviewed protocol.

### 15.2 React web

- Follow §8.2. Add automated rules for dangerous HTML/dynamic code, untrusted
  navigation/props, browser storage of auth material, wildcard postMessage,
  missing opener isolation and accidental sensitive logging/analytics.
- A sanitizer, Markdown engine, map SDK, chart renderer, service worker or
  remote font/script is a security dependency and requires explicit
  configuration and bypass tests.
- Source maps and debug bundles are not publicly deployed unless a reviewed
  access/use case exists and no secret/internal source metadata is exposed.

### 15.3 React Native

- Follow §9. Native code and bridge receivers validate independently of
  TypeScript types. Hermes/minification/obfuscation does not protect secrets.
- Do not hard-code privileged backend/provider credentials or security-
  critical authorization logic in the shipped client.
- Test iOS and Android separately for secure storage, backup, screenshot/
  task-switcher, network policy, universal/app links, WebView, push, logout,
  deletion, permission revocation and signed update behavior.

### 15.4 Static assets

- Treat every shipped SVG, font, image, and generated design artifact as a
  supply-chain input, including assets already present when the pipeline is
  introduced. SVG must be sanitized to prohibit script, event handlers,
  foreign active content, external fetches and unsafe links before serving.
- Serve exact safe MIME types with nosniff and a CSP that does not make SVG or
  uploaded content executable in an application origin.
- Record source/license/provenance and deterministic generation settings;
  strip unnecessary metadata, optimize/crop raster variants, and verify
  expected hashes in the asset pipeline.
- Fonts are pinned/licensed and preferably self-hosted when that avoids
  tracking or runtime dependency; downloaded font data is not trusted code.

---

## 16. Verification and Release Gates

The initial release must produce repeatable evidence for REQUIREMENTS.md §12.
At minimum:

| Gate | Required evidence |
|---|---|
| Architecture/threat | Approved F-01–F-31 flows and TB-01–TB-16 boundaries, DATA-001 inventory, TM-01–24 owners/controls/residual risks, cross-functional validation |
| Provider/data integrity | Forged/malformed/oversized/archive/replay/order/anomaly/stale/conflict/license/correction tests; immutable provenance and golden AQI/rule vectors |
| SSRF/egress | IPv4/IPv6 private/link-local/loopback/reserved/metadata, DNS change/rebinding, redirects, ambiguous URLs, ports/schemes, credential forwarding, slow/large responses |
| Authentication | Enumeration, credential stuffing/spraying, breached password, reset races/replay, WebAuthn challenge/RP/origin/UV, admin bootstrap/recovery/break glass, session fixation/rotation/revocation |
| Authorization | Complete positive and negative object/field/function/workload/admin matrix, cross-user/bulk/nested/self-elevation/approval tests |
| Web/API | Machine-readable API-specification drift, methods/media/schema, webhook/callback authentication, environment binding, timestamp/expiry/replay/idempotency, CSRF/CORS/CSP/security headers, cache isolation/poisoning/invalidation, error/redaction, DAST/fuzz/abuse/load |
| Mobile | Secure storage, session replay/revocation, cache/backup/screenshot cleanup, permission denial/revoke, deep links/WebViews/native bridge, TLS/cleartext, signed package/update |
| Privacy | Notice/choice, processor/transfer approval, exact retention, data export/delete identity and downstream completion, analytics/minors gate, notification preview/link privacy |
| Service/data | Distinct workload identities, least privilege, internal encryption, queue schema/replay/DLQ, private stores, encryption/keys, secret rotation, cache public-only |
| Audit/detection/IR | Tamper/immutability/integrity/gap/read-export tests, trusted-time skew/jump/unsynchronized-node behavior, log/error/trace redaction and injection tests, alert delivery/silence, runbook/tabletop evidence |
| Supply chain/cloud | Protected source/workflows, untrusted PR isolation, locked graph, SBOM/scans, signed provenance/artifacts, static-asset source/license/hash/sanitization evidence, plan/apply binding, state controls, policy/drift, rollback |
| Resilience | Numeric SLO/RPO/RTO/staleness/cost approval; retry/fan-out/storage exhaustion, provider/vendor/region failure, ransomware, isolated restore/failover and state-resurrection tests |
| Accessibility/safe design | Automated token/component contrast plus manual web/mobile keyboard, reader, zoom/text scaling, motion and non-color tests; known failing DESIGN.md tokens blocked |

Independent penetration testing is required before initial production and
after material authentication, authorization, admin, or trust-boundary
changes. An unresolved Critical finding blocks initial production and cannot
be risk accepted. An unresolved High finding blocks release unless the
time-limited SEC-IR-003 evidence is current and the release authority
explicitly accepts it.

---

## 17. Threat-to-Control Summary

| Threats | Primary control families |
|---|---|
| TM-01/02/24 public-data corruption, ordering, or inaccessible/misleading presentation | §4 ingestion/publication, SEC-008, AQ-008–010, REL-001–003, NFR-A11Y-001–005, redundant non-color status, contrast/accessibility evidence, rule release provenance |
| TM-03/04 admin takeover/abuse | §5.3–5.4, §6.2, ADMIN-001/009–012, audit separation |
| TM-05 SSRF/cloud pivot | §3.2, §4.1, §7.2, SEC-SVC-003, narrow fetch identity |
| TM-06/08 account/BOLA | §5, §6.1, API contract/cache, negative authorization testing |
| TM-07/18/19/21 privacy/mobile/notification/callback | §8–10, SEC-API-009, DATA-001/005, authenticated callback and privacy release evidence |
| TM-09 content/injection | §4.2, §7, §8.2, §9, static/raw safe viewers |
| TM-10/11 availability/cost | §3.3, §4.4, §14, API-013, NFR-AVAIL-004/005 |
| TM-12/13 internal/cache trust | §3.2–3.4, queue envelope, workload IAM, cache policy |
| TM-14/17 stores/secrets | §3.4, §11, §13.2, DATA-003/004, private services |
| TM-15/22 audit/detection/response | §2.3, §12, SEC-LOG-001–007, SEC-IR-001–003 |
| TM-16 supply chain | §2.1, §13, NFR-MAINT-006/008, release provenance |
| TM-20 recovery state resurrection | §3.4, §6.2, §14, REL-004–008 |
| TM-23 licensing | §4.1/4.3, provider governance, publication and API tiers |

---

*Security baseline v0.2. Update it with each NFR-MAINT-007 trigger and attach
verified evidence before changing a control from Not Verified.*
