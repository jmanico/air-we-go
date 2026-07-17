# Air We Go — Architecture and Threat Model

- **Status:** Threat-modeled logical baseline; provisional until the decision gates in §9 are approved
- **Version:** 0.2
- **Threat-model assessment date:** 2026-07-17
- **Approval status:** Draft baseline; no accountable owner or approver has signed off
- **Owner / approvers:** TO BE DECIDED before implementation; approval must include product, security, privacy/legal, air-quality domain, engineering, mobile/client, accessibility, and operations representatives
- **Next review:** Before architecture freeze, and on every NFR-MAINT-007 trigger
- **v0.2 change summary:** Added explicit flows/boundaries, trust domains, invariants, consolidated threat register, STRIDE/LINDDUN analysis, attack paths, and provisional FMEA

No implementation exists. Every control in this document is therefore
**Proposed / Claimed in documentation — Not Verified**. A control becomes
verified only when deployed configuration or repeatable test evidence is
linked to it.

This document derives behavior from [REQUIREMENTS.md](./REQUIREMENTS.md),
client visual/accessibility rules from [DESIGN.md](./DESIGN.md), and
enforceable implementation rules from [SECURITY.md](./SECURITY.md). When the
documents conflict, REQUIREMENTS.md governs product behavior and the stricter
security or privacy rule governs implementation until an approved decision
resolves the conflict.

---

## 1. Architecture Objectives and Constraints

### 1.1 System purpose

Air We Go is a global public air-quality platform. It ingests observations
from approved external providers, validates and preserves provenance, derives
versioned air-quality values, stores current and historical data, and serves
web, mobile, public API, personalized, notification, and administrative use
cases.

Because users may use the output to make health-related decisions, these are
equal architecture priorities:

1. **Public-data integrity and availability** — a compromised provider,
   administrator, rule, cache, worker, or dependency must not silently turn
   untrusted, stale, replayed, or incorrectly transformed data into current
   public guidance or alerts.
2. **User and location confidentiality** — email, credentials, sessions,
   searches, saved locations, precise device location, alert rules,
   notification destinations/history, and vendor disclosures require
   minimization and least privilege.
3. **Control-plane integrity** — administrative identity, provider
   configuration, rule publication, audit evidence, secrets, CI/CD, and cloud
   management require phishing-resistant authentication, separation of
   duties, independently authorized workloads, and tamper-evident evidence.
4. **Failure containment** — expensive queries, imports, retries, backfills,
   queues, caches, providers, vendors, and regions use bounded resources and
   fail without starving trusted current data, authentication, audit, or
   incident response.

The earlier concept of treating end-user protection as a “light” investment
is rejected by this threat model. Location-linked account data and public
air-quality integrity are both high-value assets.

### 1.2 Selected architecture inputs

The following inputs were previously supplied and remain the provisional
baseline. They require dated decision records and the §9 security criteria
before implementation:

| Concern | Selected input | Remaining decision |
|---|---|---|
| Server runtime | Node.js | Framework and supported version |
| Web client | React | Rendering/session strategy and supported version |
| Mobile client | React Native for iOS and Android | Supported version, native modules, release/update process |
| API style | Versioned REST | Gateway/edge implementation and developer-access tier |
| Consumer authentication | Email/password; optional passkey as an additional or alternative authenticator | Browser and mobile session/token lifecycle |
| Administrative authentication | Mandatory WebAuthn/FIDO2 passkey with user verification | RP IDs/origins, library, authenticator assurance, enrollment, recovery, break glass |
| Authorization | Server-enforced RBAC; ABAC only for constraints that roles cannot express | Complete permission and approval matrix |
| Deployment | Microsoft Azure provisioned with Terraform | Compute, network, subscription, region, key/secret, CI and state topology |
| Scale | Global users and provider/station coverage | Launch geography, data residency, SLO/RPO/RTO and cost budgets |

Federated consumer identity and AI/ML features are out of scope for v1.
Predictive forecasting and machine-learning anomaly detection require a new
AI/ML threat-model review before introduction.

### 1.3 Architecture principles

- No component is trusted solely because it is inside the cloud network.
- Public routing, an administrative URL prefix, and a hidden client menu are
  not security boundaries.
- Provider and third-party integrations remain behind reviewed adapter
  interfaces; provider-controlled data never selects its own destination.
- A component that parses untrusted data does not receive broad cloud,
  secret, audit, or unrelated data-store privileges.
- Original provider records and administrative corrections remain
  distinguishable and reproducible; derived data never overwrites source
  evidence.
- Data stores, queues, caches, object stores, backups, and observability have
  explicit identities, classifications, readers, writers, retention, and
  failure behavior before a product is selected.
- Privileged actions cannot become unaudited during a logging outage.
- Open technology choices remain capability requirements, not guessed vendor
  or library selections.

---

## 2. Actors, Adversaries, and External Systems

### 2.1 Legitimate actors

| Actor | Trust and capabilities |
|---|---|
| Public user | Untrusted anonymous Internet client; reads licensed public data and performs bounded searches |
| Authenticated user | Untrusted client with an authenticated session; owns profile, sessions, saved locations, preferences, alerts, and notification data |
| Support Administrator | Privileged workforce identity with narrowly masked support capabilities; no provider/rule/role authority by default |
| Data Administrator | Privileged workforce identity for reviewed provider-data quality actions; cannot grant roles or administer audit controls |
| Security Administrator | Privileged workforce identity for security policy, incident response, and role approval; cannot self-approve |
| Application System Administrator | Application control-plane RBAC only; no Azure, CI, Terraform, signing, backup, key, source-control, or routine air-quality-content authority |
| Read-Only Auditor | Privileged read-only identity; technically incapable of mutation; audit access is itself audited |
| Cloud/SRE operator or incident responder | Separate operational workforce identity with time-bounded access through approved operational or break-glass workflows; never granted through an application role |
| Service workload | Non-human, individually authenticated producer/consumer with least-privilege service, queue, store, secret, and key permissions |
| CI/Terraform workload | Non-human delivery identity with short-lived, environment-scoped build or deployment permission |
| External air-quality provider | Untrusted external source of public-domain or licensed observations; may be unavailable or compromised |
| External processor | Map, geocoding, notification, CDN/edge, analytics, observability, support, or cloud provider receiving minimum approved data |

### 2.2 Threat agents

- Automated bots, scrapers, credential stuffers, denial-of-service actors, and
  malicious API users.
- A compromised or malicious air-quality provider, map/geocoding vendor,
  notification provider, dependency publisher, build action, or cloud account.
- A compromised user, administrator device/session, support account, service
  identity, CI runner, or Terraform workflow.
- A malicious or careless insider with data, security, system, audit, support,
  source-control, or cloud access.
- Malware or another application on a lost/compromised mobile device.
- An attacker who chains lower-severity flaws, such as admin recovery plus a
  provider-test SSRF, to reach the cloud control plane.

| Threat-agent profile | Motivation | Representative capability / constraint |
|---|---|---|
| Opportunistic Internet actor | Account resale, abuse, disruption | Automation, credential lists, distributed requests; no initial trust |
| Targeted criminal or stalker | Location/account disclosure, extortion | Social engineering, device/session theft, correlation across vendors |
| Provider/vendor compromise | False data, surveillance, lateral movement | Valid provider/vendor identity and plausible content; bounded by adapter scope |
| Malicious/careless insider | Fraud, concealment, operational shortcut | Legitimate narrow admin, support, data, source, or cloud access |
| Supply-chain attacker | Persistent code/cloud control | Dependency, action, runner, registry, artifact, signing, or update compromise |
| Resource-exhaustion actor | Outage or unbounded cost | Distributed queries, slow/large inputs, replay, cache busting, fan-out triggers |

### 2.3 External systems

External systems are trust boundaries even when contractually approved:

- Air-quality APIs, feeds, bulk-download hosts, and provider signing/checksum
  services.
- Map tile and geocoding services.
- Email, mobile push, and Web Push delivery services and status webhooks.
- DNS, certificate, CDN, DDoS, WAF, and edge/gateway services.
- Source control, package/container registries, CI runners, artifact stores,
  mobile app stores, and any over-the-air update channel.
- Azure management, identity, secret/key, network, compute, storage, backup,
  and observability control planes.

---

## 3. Assets and Data Classification

DATA-001–005 in REQUIREMENTS.md govern the complete field-level inventory. The
table below establishes the minimum architecture classes and blast-radius
boundaries.

| Asset / dataset | Minimum class | Integrity / availability criticality | Required separation |
|---|---|---|---|
| Approved public observations and metadata | Public after publication gate | Safety/integrity critical; high availability | Public read path cannot write the trusted dataset |
| Raw/canonical provider payloads and quarantine | At least Confidential until inspected/minimized and explicitly reclassified | Critical provenance and forensic evidence | Isolated ingestion writers; safe read-only admin viewer; no auth headers, cookies, client certificates, private keys, or bearer secrets |
| AQI rules, category thresholds, unit conversions, aggregation, source selection, freshness, health guidance | Internal | Safety/integrity critical | Versioned release path; distinct author/approver; no direct runtime edit |
| Provider endpoints, schedules, licensing and non-secret configuration | Internal | Control-plane critical | Admin control plane; change approval and audit |
| Provider, cloud, CI, signing, encryption, and service credentials | Restricted | Critical | Secret/key manager; values not returned by APIs or stored with ordinary configuration |
| User email/profile, saved/default locations, searches tied to a user, alert rules, notification destinations/history, device/push identifiers | Confidential | High confidentiality and integrity | Identity/personalization boundary; masked support access |
| Anonymous exact-coordinate searches linked to IP, device, request correlation, or another stable identifier | Restricted while linkable; otherwise classify by the approved data inventory | Physical-safety and privacy impact | Ephemeral/minimized processing; no ordinary telemetry or durable history |
| Password hashes, passkeys/recovery state, sessions, reset/verification artifacts | Restricted | Critical account security | Dedicated identity/session boundary |
| Precise device location or durable exact location history | Restricted | Physical-safety and privacy impact | Ephemeral/minimized path; no background retention in v1 |
| Audit events and security evidence | Confidential; Restricted when containing privileged/security metadata | Critical integrity and availability | Separate append-only/tamper-evident authority; audited readers |
| Logs, metrics, traces and diagnostics | Internal or Confidential by field | High detection value; disclosure risk | Redacted security/operations plane with bounded retention |
| Terraform state/plans, deployment manifests, SBOM/provenance and release artifacts | Restricted if secrets or topology are present; otherwise Internal | Critical supply-chain integrity | Separate environment state, delivery identities and protected artifacts |
| Backups and restore artifacts | Same as the most sensitive included field | Critical recovery and ransomware target | Isolated identity, encryption, immutability/versioning and restore boundary |
| Public trust, safety status, service availability, and cost budgets | Internal control state; published status may be Public | Critical safety/reputation and availability | Versioned policy, independent monitoring, bounded automation and incident authority |
| Product/admin/email/link domains, DNS, certificates, mobile app identifiers, and sending reputation | Internal; credentials/keys Restricted | Critical identity and trusted-navigation integrity | Separate ownership, renewal, monitoring, recovery and change approval |
| Source, build workflows, dependencies, registries, artifacts, signing/provenance and mobile releases | Internal; credentials and sensitive topology Restricted | Critical supply-chain integrity | Protected review, isolated build, immutable promotion, signing and revocation |
| IAM roles/policies, approval records, processor/provider contracts/licenses, notices/consent and rights-completion evidence | Confidential; secrets or sensitive policy state Restricted | Critical authorization, compliance and accountability | Versioned least privilege, separate approval, retention and tamper-evident evidence |

Public status is an outcome of ING-016 and AQ-009, not a property inherited
from a provider. Quarantined, unpublished, stale, suppressed, personal, or
operational data must not be exposed through a public-data store or cache.

---

## 4. Logical Architecture

Component names describe logical security and scaling boundaries. They do not
require one deployable per box; modular-monolith versus service decomposition
remains a §9 decision.

### 4.1 System and data-flow view

~~~mermaid
flowchart LR
  subgraph U[External devices, recipients, vendors, and Internet]
    PUB[Public browser]
    USR[Authenticated browser]
    MOB[React Native app]
    ADM[Managed admin device]
    AUD[Read-only auditor device]
    OPS[Cloud / SRE / break-glass operator]
    RECIP[Email / push / lock-screen surface]
    BOT[Threat agents]
    AQP[Air-quality providers]
    VND[Map / geocoder / notification vendors]
  end

  subgraph E[Internet edge]
    EDGE[DNS / CDN / DDoS / WAF / public gateway]
    AEDGE[Separate admin access gateway and audience]
    OEDGE[Operational JIT access broker]
  end

  subgraph A[Application zone]
    PAPI[Public API]
    ID[Identity and User]
    ADMIN[Administrative Control Plane]
    NOTIF[Notification]
  end

  subgraph W[Ingestion and worker zone]
    FETCH[Provider fetch / bulk intake]
    QUAR[Quarantine and validation]
    PROC[Air-quality processing and publication gate]
    QUEUE[Queues / event bus]
  end

  subgraph D[Private data zone]
    IDS[(Identity / session store)]
    PERS[(Personalization store)]
    CUR[(Trusted current observations)]
    HIST[(Historical / analytical store)]
    RAW[(Raw / quarantine object store)]
    CACHE[(Public-data cache)]
    CONF[(Configuration / rule store)]
  end

  subgraph S[Security and operations zone]
    SECRETS[Secret / key manager]
    AUDIT[(Tamper-evident audit store)]
    OBS[(Logs / metrics / traces / alerts)]
    BACKUP[(Isolated backups)]
  end

  subgraph C[Delivery and cloud management boundary]
    SCM[Source / registries]
    CI[CI / artifact promotion]
    TF[Terraform / Azure control plane]
  end

  PUB -->|F-01 request| EDGE
  USR -->|F-01 request| EDGE
  MOB -->|F-01 request| EDGE
  BOT -->|F-01 hostile request| EDGE
  EDGE -->|F-02 public read| PAPI
  EDGE -->|F-03 auth and account| ID
  ADM -->|F-04 admin request| AEDGE
  AEDGE -->|F-05 authorized admin request| ADMIN
  ADMIN -->|F-06 admin auth and reauth| ID
  ID -->|F-07 identity state| IDS
  ID -->|F-07 preferences and rights| PERS
  ID -->|F-08 security and delivery job| NOTIF
  PAPI -->|F-09 public read only| CUR
  PAPI -->|F-09 bounded history read| HIST
  PAPI -->|F-09 public cache read or fill| CACHE
  FETCH -->|F-10 approved outbound request| AQP
  AQP -->|F-10 untrusted response| FETCH
  FETCH -->|F-11 raw intake| QUAR
  QUAR -->|F-12 immutable evidence| RAW
  QUAR -->|F-13 validated candidate| QUEUE
  QUEUE -->|F-14 authorized event| PROC
  PROC -->|F-15 eligible versions| CUR
  PROC -->|F-15 eligible versions| HIST
  PROC -->|F-15 versioned cache update| CACHE
  PROC -->|F-16 committed alert event| NOTIF
  NOTIF -->|F-17 minimal delivery| VND
  VND -->|F-17 delivered message| RECIP
  VND -->|F-18 callback| EDGE
  EDGE -->|F-18 authenticated scoped callback| NOTIF
  ADMIN -->|F-19 approved versioned config| CONF
  ADMIN -->|F-20 approved command| QUEUE
  ADMIN -->|F-21 privileged evidence| AUDIT
  ID -->|F-21 identity evidence| AUDIT
  PROC -->|F-21 publication evidence| AUDIT
  CI -->|F-21 delivery evidence| AUDIT
  PAPI -->|F-22 redacted telemetry| OBS
  ID -->|F-22 redacted telemetry| OBS
  ADMIN -->|F-22 redacted telemetry| OBS
  NOTIF -->|F-22 redacted telemetry| OBS
  FETCH -->|F-22 redacted telemetry| OBS
  QUAR -->|F-22 redacted telemetry| OBS
  PROC -->|F-22 redacted telemetry| OBS
  QUEUE -->|F-22 redacted telemetry| OBS
  ID -->|F-23 scoped secret reference| SECRETS
  FETCH -->|F-23 scoped provider secret| SECRETS
  ADMIN -->|F-23 write-only secret change| SECRETS
  NOTIF -->|F-23 scoped vendor secret| SECRETS
  IDS -->|F-24 backup snapshot| BACKUP
  PERS -->|F-24 backup snapshot| BACKUP
  CUR -->|F-24 backup snapshot| BACKUP
  HIST -->|F-24 backup snapshot| BACKUP
  RAW -->|F-24 backup snapshot| BACKUP
  CONF -->|F-24 backup snapshot| BACKUP
  AUDIT -->|F-24 backup snapshot| BACKUP
  BACKUP -->|F-25 isolated restore| IDS
  BACKUP -->|F-25 isolated restore| CUR
  BACKUP -->|F-25 isolated restore| CONF
  SCM -->|F-26 reviewed source and inputs| CI
  CI -->|F-26 promoted artifact and plan| TF
  TF -->|F-26 provision and deploy| PAPI
  TF -->|F-26 provision and deploy| FETCH
  TF -->|F-26 provision private service| IDS
  ID -->|F-27 rights and deletion command| VND
  ID -->|F-27 rights and deletion command| IDS
  ID -->|F-27 rights and deletion command| PERS
  ID -->|F-27 rights and deletion command| QUEUE
  ID -->|F-27 rights and deletion command| NOTIF
  AUD -->|F-29 audited read request| AEDGE
  AEDGE -->|F-29 authorized audit query| ADMIN
  ADMIN -->|F-29 read or export query| AUDIT
  OPS -->|F-30 time-bounded access| OEDGE
  OEDGE -->|F-30 telemetry query| OBS
  OEDGE -->|F-30 cloud operation| TF
  OEDGE -->|F-30 restore operation| BACKUP
  OEDGE -->|F-21 operational evidence| AUDIT
  NOTIF -->|F-31 current consent query| ID
  ID -->|F-31 current consent state| NOTIF
~~~

Arrows are logical flows, not blanket network permissions. Each crossing must
match the flow registry below, the trust-boundary table in §5, and an explicit
workload authorization policy. The map/geocoder direct-versus-proxied flow is
intentionally absent: F-28 remains prohibited until the §9 decision selects
and approves one topology. Restore arrows show representative control state;
F-25 applies to every backed-up store through an isolated workflow.

#### 4.1.1 Data-flow registry

Protocol/product details marked **TBD** are gates, not permission to improvise.
Retention is controlled by DATA-001/005; “bounded” below means the approved
numeric profile must exist before implementation.

| ID | Source → destination; direction/purpose | Data and minimum class | Authorization / protocol status | Retention, evidence, and failure behavior | Boundaries / threats |
|---|---|---|---|---|---|
| F-01 | Browser/mobile/threat agent → public edge; inbound request | Public query; Confidential account metadata; exact linked coordinate Restricted | Approved HTTPS required; edge product **TBD**; route/method/size/cost allowlist | No sensitive body logging; bounded rejection/degradation | TB-01/02/09/14; TM-06/08–10/18 |
| F-02 | Public edge → Public API; public read request/response | Public observations; query metadata by classification | Edge-authenticated route context; workload hop **TBD**; API read identity only | Public caching only; fail closed on route/schema ambiguity | TB-01/09; TM-10/13 |
| F-03 | Public edge → Identity/User; auth/account request/response | Restricted credentials/session artifacts; Confidential personal data | Approved session/auth protocol **TBD**; distinct route and audience | Private/no-store; durable security evidence; generic bounded failure | TB-02; TM-06/08/21 |
| F-04 | Managed admin device → admin gateway; inbound privileged request | Restricted authenticator/session; Confidential control data | Separate HTTPS origin/audience, managed-access defense; products/RP IDs **TBD** | No shared cache; alert/audit failed or suspicious access | TB-03; TM-03/04/09 |
| F-05 | Admin gateway → Administrative Control Plane; authorized operation | Internal–Restricted admin command/response | Admin workload path plus operation authorization; implementation **TBD** | High-impact operation fails if authorization or durable audit fails | TB-03; TM-03/04/15 |
| F-06 | Administrative Control Plane ↔ Identity/User; admin WebAuthn, reauth, role state | Restricted challenge/session/authenticator; Confidential approval context | Mandatory WebAuthn UV, exact origin/RP, separate audience, two-person lifecycle | Short-lived/single-use artifacts; audit; no weaker fallback | TB-03/08; TM-03/04/08 |
| F-07 | Identity/User → identity/personalization stores; read/write owner state | Restricted identity/session; Confidential profile/location/preference | Identity workload only for credential writes; owner/field authorization; store **TBD** | DATA-001 retention/deletion; no shared cache; fail closed | TB-02/08/15; TM-06/08/14/21 |
| F-08 | Identity/User → Notification; verified delivery/security job | Confidential destination, preference, generic content; bounded Restricted linkage | Per-topic producer/consumer identity; queue/product **TBD** | Consent checked at send; TTL/idempotency; audit without content | TB-07/10; TM-07/11/12/19 |
| F-09 | Public API → current/history/cache; read/fill only | Public eligible versions; Internal provenance pointer | Distinct read identities; only trusted loader may fill public cache; engines **TBD** | Bounded query/TTL; version/freshness; no private cache | TB-08/09; TM-02/10/13/14/24 |
| F-10 | Fetcher → approved provider, then provider → fetcher; initiated poll/response | Scoped credential Restricted; raw response at least Confidential until inspected | Approved TLS, static destination, egress/IP/DNS/redirect controls; provider **TBD** | Byte/time/retry limits; provenance; last eligible version on failure | TB-04/05; TM-01/02/05/11/23 |
| F-11 | Fetcher → quarantine; store untrusted intake | Raw response at least Confidential | Narrow fetch identity; no publication authority | Immutable/versioned intake; terminal quarantine on ambiguity | TB-05/08; TM-01/05/09/11 |
| F-12 | Quarantine → raw store; preserve evidence | Allowlisted raw/canonical payload at least Confidential | Quarantine writer; safe-view/reprocess readers only; object store **TBD** | License-bound retention; digest plus independent provenance; no active serving | TB-05/08; TM-01/09/14/23 |
| F-13 | Quarantine → queue; validated bounded candidate | Internal; Confidential if linked to non-public metadata | Per-topic producer identity and schema; broker **TBD** | TTL, idempotency, replay record, restricted DLQ | TB-06/07; TM-01/02/11/12 |
| F-14 | Queue → publication processor; authorized event delivery | Internal candidate plus provenance/version | Broker-authenticated producer scope or verified signature; consumer reauthorizes | Ordering/replay/backpressure limits; no state regression | TB-06/07; TM-01/02/11/12/24 |
| F-15 | Publication processor → trusted current/history/cache; versioned write | Public only after context-specific eligibility; Internal lineage | Publication authority alone writes current; publication/approved aggregation writes history | Atomic version/decision; correction/invalidation; last eligible state on failure | TB-06/08; TM-01/02/12–14/20/24 |
| F-16 | Publication processor → Notification; committed alert candidate | Public observation plus Confidential alert linkage downstream | Authorized topic and eligible context/version only | Ordered/idempotent/expiring event; no stale recovery | TB-07/10; TM-01/02/11/12/19/24 |
| F-17 | Notification → email/push/Web Push vendor → recipient/lock-screen surface; outbound delivery then external presentation | Minimum generic content and scoped destination, Confidential until deliberately disclosed to the recipient surface | Scoped vendor credential and approved HTTPS/API **TBD**; final device/mailbox is outside platform control | Retry/fan-out/expiry bounds; best effort; provider acceptance is not delivery; generic preview limits bystander disclosure | TB-10/14; TM-07/11/18/19 |
| F-18 | Vendor → public edge → Notification; inbound delivery/status callback | External event; Confidential delivery correlation | SEC-API-009 sender authentication, environment/type/scope binding, trusted-time replay window; protocol **TBD** | Strict schema, idempotency, rate limit; spoof/replay cannot change unrelated state | TB-01/07/10; TM-11/12/19 |
| F-19 | Administrative Control Plane → configuration/rule store; approved version | Internal policy; secret references only | Reauth, impact-matrix approval, versioned admin/release writer | Immutable version, effective time, rollback, audit; no direct secret readback | TB-03/08/11; TM-01–04/15/17/24 |
| F-20 | Administrative Control Plane → queue; approved backfill/correction/suppression command | Internal command and bounded scope; possible Confidential target | Operation authorization, reauth/approval, per-topic identity | Preview, cancellation, idempotency, current validation, audit; no direct trusted-store write | TB-03/06/07; TM-01/04/11/12/24 |
| F-21 | Privileged workloads → audit; append evidence | Confidential; Restricted security metadata where necessary | Write-only append identities; independent read/admin/retention authority; store **TBD** | Immutable/WORM plus tamper evidence and trusted ordering; high-impact action fails closed | TB-11; TM-03/04/12/15/16/22 |
| F-22 | Workloads → observability; redacted telemetry/detection | Internal/Confidential allowlisted fields | Authenticated collectors; stack **TBD**; least-privilege readers | Bounded cardinality/retention; gaps/silence/backpressure alert independently | TB-11; TM-10/11/15/22 |
| F-23 | Workloads/admin → secret/key manager; scoped read or write-only change | Restricted secrets, keys, credential metadata | Per-service/environment/purpose identity; product **TBD** | Version/rotation/access evidence; deny on uncertainty; values never logged/returned | TB-08; TM-05/14/17 |
| F-24 | Primary stores/config/audit → isolated backup; snapshot | Same class as most sensitive source | Dedicated backup identity/deletion authority; technology **TBD** | Encrypted, versioned/immutable, integrity checked, retention/deletion policy | TB-13; TM-14/20 |
| F-25 | Isolated backup → isolated restore → target; controlled recovery | Same class as source | Time-bound separate restore workflow, approval, clean environment | Test integrity/security/deletion/suppression before promotion; fail closed | TB-13; TM-14/20/22 |
| F-26 | Source/registries → CI → Terraform/Azure/resources; build/promote/provision | Internal artifacts; Restricted state/credentials/topology | Protected review, immutable pins, short-lived separate build/plan/apply/sign identities; products **TBD** | SBOM/provenance/digest, plan binding, audit, drift/rollback; no untrusted secrets | TB-12; TM-05/14/16/17 |
| F-27 | Rights/deletion orchestrator → identity/personalization stores, queue, Notification and processors; export/correct/delete | Confidential/Restricted personal data or lifecycle command | Recent owner auth, object/field scope, per-store/topic workload authority and processor-authenticated API **TBD** | Short-lived export; downstream completion/exception evidence; retention/legal/backup limits; retries must not resurrect state | TB-07/08/10/15; TM-06/07/12/14/20/21 |
| F-28 | Browser/mobile or Public API → map/geocoder; **prohibited until approved** | Viewport/search; exact coordinate becomes Restricted when linkable | Direct/proxy/on-device topology, credential scope, protocol and processor all **TBD** | Must define minimization, retention, deletion, outage and telemetry before enablement | TB-10; TM-07/10/17 |
| F-29 | Read-Only Auditor → admin gateway/control plane → audit store; read/query/export and response | Confidential/Restricted audit evidence, minimized to purpose | Separate auditor identity/audience, read-only operation/field scope, recent auth for export; client/API **TBD** | Every read/export is audited; bounded query and protected short-lived export; mutation technically denied | TB-03/11; TM-04/06/14/15/21 |
| F-30 | Cloud/SRE/break-glass operator → operational JIT broker → observability, Azure/Terraform or isolated restore workflow | Internal telemetry; Restricted cloud/restore authority and data | Separate workforce identity, phishing-resistant auth, approved purpose/scope, JIT expiry, no application-role inheritance; broker **TBD** | Session/command evidence to independent audit; alert on grant/use/expiry; fail closed without authority/evidence | TB-11–13/16; TM-03–05/14–17/20/22 |
| F-31 | Notification ↔ Identity/User; query current destination verification, consent and preference at send time | Confidential user/destination/alert state; no message content required | Authenticated workload request with owner/resource scope; service protocol **TBD** | Authoritative current read or ordered revocation; fail closed/no send on uncertainty; privacy-safe audit | TB-07/08/10; TM-07/11/12/19/21 |

### 4.2 Clients and ingress

- **Public React web client:** public current/history/map/search views. It may
  call only the documented public/authenticated REST operations. Browser
  geolocation follows WEB-007; third-party scripts or direct vendor calls are
  not assumed.
- **React Native clients:** iOS/Android feature parity, foreground
  user-invoked location, secure session storage, bounded local public-data
  cache, safe deep links, notification navigation, and signed releases.
- **Administrative client:** a distinct privileged client delivered through
  an independently enforced admin origin/access gateway and authentication
  audience. It does not accept consumer sessions, render active provider
  content, or expose raw secrets.
- **Public edge:** logical capabilities for certificate termination, DDoS and
  abuse resistance, request normalization, route/method/content-size policy,
  distributed rate/cost limits, and forwarding only to documented public or
  identity routes. Specific Azure products are not selected.
- **Administrative edge:** distinct routing and identity policy with managed
  device/network restrictions as defense in depth. It never substitutes for
  server-side operation authorization.

Server-side rendering, CDN behavior, browser session transport, map/geocoder
direct-versus-proxied flows, and third-party script use are unresolved. No
design may send precise/saved location or a stable account identifier to a
vendor without SEC-PRIV-007 review.

### 4.3 Backend logical components

1. **Public API** — current and historical observations, geographic search,
   trends, public metadata, and personalized-resource routing. Public reads
   cannot write trusted observations. Expensive operations enforce API-013
   budgets before database work.
2. **Identity and User** — registration, email verification, password and
   optional consumer passkeys, admin WebAuthn ceremonies, session/revocation,
   preferences, saved locations, alerts, privacy-rights workflows, and
   security notifications. Admin and consumer audiences remain separate.
3. **Provider Fetch / Bulk Intake** — uses approved static provider
   destinations and narrow egress; has no public-data write, broad cloud, or
   unrelated secret permission.
4. **Quarantine and Validation** — stores immutable raw/canonical evidence,
   verifies source/transport/checksum when available, applies strict schema
   and resource limits, and emits only a bounded candidate record. It cannot
   label data public.
5. **Air-Quality Processing and Publication Gate** — applies versioned unit,
   standard, aggregation, anomaly, conflict, freshness, provenance, and
   licensing rules. Only an eligible committed version can update trusted
   current/history stores, caches, or alert evaluation.
6. **Notification** — consumes committed eligible observation events, checks
   consent/preferences at send time, enforces ordering/idempotency/expiry and
   fan-out budgets, minimizes vendor payloads, and authenticates delivery
   status callbacks.
7. **Administrative Control Plane** — provider/rule/data-quality/user-role
   operations, secure previews, approvals, operational status, and audit
   review. It enforces ADMIN-001–012 and never renders raw active content or
   returns secret values.
8. **Audit and Detection** — append-only/tamper-evident privileged evidence,
   redacted logs/traces/metrics, security and data-integrity detections, and
   independently monitored alert delivery.
9. **Privacy and deletion orchestration** — may be part of Identity/User, but
   is a distinct workflow that propagates verified access/export/correction/
   deletion to stores, caches, queues, local/client state where controllable,
   and contracted processors.

Every component validates both inbound and outbound schemas. A successful
internal network connection or queue read does not establish authority.

### 4.4 Trusted ingestion and publication path

The required state machine is:

1. **Configured** — provider ownership, license, endpoints, credentials,
   transport/checksum capability, update cadence, incident contact, and
   limits pass onboarding review.
2. **Retrieved** — the fetcher applies destination and egress policy, records
   source, timing, response metadata, adapter version, and a digest, and
   writes to isolated raw/quarantine storage.
3. **Validated** — strict schema, semantic ranges, coordinate/time/unit,
   freshness, size, unknown-field, archive/parser, duplicate, replay, and
   anomaly controls run. Failure remains quarantined with a bounded reason.
4. **Normalized** — conversion is deterministic, source values remain
   immutable, and the output references the raw digest, adapter, rules, and
   every transformation.
5. **Reconciled** — provider conflicts, ordering, prior versions,
   suppressions, corrections, licensing, and source policy are applied.
6. **Eligibility decided and published** — a versioned decision independently
   records historical-public, current-display, aggregation/guidance, and
   alert eligibility. Only the matching committed state updates its trusted
   store or emits its versioned invalidation/notification event; eligibility
   in one context never implies another.
7. **Observed** — audit/metrics make retrieval, rejection, quarantine,
   publication, supersession, suppression, replay, reconciliation failure,
   and validator silence detectable.

No retry, backfill, reprocessing, administrator preview, or failover may skip
these states. During a provider failure, only the last eligible version may be
served, under its current/delayed/stale/unavailable classification. Expired
data cannot generate a new threshold or recovery notification.

### 4.5 Data stores and access domains

Exact engines are undecided. At minimum, these are distinct logical security
principals and backup/retention policies:

| Store role | Data / class | Authorized writers | Authorized readers | Mandatory properties |
|---|---|---|---|---|
| Identity/session | Restricted credentials, authenticators, sessions, recovery state | Identity service only | Identity service; exceptional audited security workflow | Private path, encryption, revocation consistency, no public/admin query access |
| Personalization | Confidential profile, saved locations, preferences, alerts, notification metadata | Identity/User and deletion workflow | Owner-scoped API; masked approved support workflow | Object/field authorization, retention/deletion, no shared-cache exposure |
| Trusted current observations | Published Public data plus Internal provenance pointer/state | Publication gate only | Public API/cache loader | Versioned writes, source lineage, freshness, atomic correction/invalidation |
| Historical/analytical | Historical-public-eligible observations/aggregates and lineage | Publication authority or a separately authenticated approved aggregation workload operating only on eligible versions | Bounded public query workers and reconciliation | Range/cost limits, immutable version lineage, deletion/license policy; no current/alert eligibility implied |
| Raw/quarantine object storage | Internal/Confidential provider payloads, rejected records, bulk artifacts | Fetch/quarantine workers only | Safe viewer and bounded reprocessor roles | Private origin, digest/versioning, no active serving, lifecycle and license enforcement |
| Configuration/rules | Internal provider metadata, approved rule versions; secret references only | Approved admin/release workflows | Runtime components needing a specific version | Dual control for high impact, immutable versions, effective dates, rollback, audit |
| Public-data cache | Public published representations only | Trusted cache loader/public API | Public API/edge | Semantic version keys, bounded TTL, stampede control, event invalidation, never turns stale into current |
| Queue/event bus | Internal bounded jobs/events; Confidential where personalization is present | Per-topic authorized producers | Per-topic authorized consumers | Private path, encryption, schema/version/TTL, idempotency/replay, quotas, restricted DLQ |
| Audit store | Confidential privileged evidence | Write-only approved services | Separately authorized auditor/security readers | Tamper evidence, ordering/time integrity, monitored access/export, retention independence |
| Observability | Redacted Internal/Confidential telemetry | Authorized workloads/collectors | Least-privilege operations/security roles | Field allowlists, access audit, retention, alert-integrity monitoring |
| Backup/restore | Same as source data | Dedicated backup identity | Dedicated time-bound restore workflow | Encryption, version/immutability, deletion protection, isolated restore and integrity test |

Provider credentials are secret-manager references, not values in the
configuration store. Combining physical engines does not permit combining
identities, schemas, network policy, backup policy, or authorization.

### 4.6 Authentication, session, and privileged access

- Consumer accounts support email/password and may register passkeys as an
  additional factor or an alternative authentication ceremony; the account
  retains its password credential in v1. Either ceremony must preserve the
  session, notification, authenticator-lifecycle, and recovery controls.
- Browser authentication must use a server-managed, revocable, rotated
  session with a narrow Secure/HttpOnly/SameSite cookie and robust CSRF
  protection. Browser bearer credentials must not be persisted in
  JavaScript-accessible storage.
- The mobile design is not yet selected. It must provide short-lived access,
  renewable-credential rotation or reuse detection, audience/issuer checks,
  secure platform storage, server revocation, and logout/deletion cleanup.
- Administrative identities are invite-only workforce identities, separate
  from consumer self-registration and consumer recovery. WebAuthn challenges
  bind the intended admin origin, RP ID, account, ceremony, and required user
  verification.
- The first admin, added/replaced/removed authenticator, lost-device recovery,
  role increase, and ADMIN-010 action require the independent approval and
  reauthentication model in REQUIREMENTS.md. No requester can approve their
  own action.
- Break-glass access is a separate, normally disabled or restricted,
  time-limited identity/workflow that alerts immediately and cannot bypass
  durable audit.

Canonical product domains, admin origin, email/link domains, mobile
identifiers, and WebAuthn RP IDs are a release-blocking decision because they
are authentication boundaries, not branding details.

### 4.7 Notification flow

The notification system is best-effort and is not an emergency-notification
service. It must preserve the distinction between observation time,
evaluation time, queue time, and delivery time. A notification references an
eligible immutable observation/rule version; retries and regional failover
reuse its idempotency key. Recovery events require a prior committed crossing
for the same alert state. Push/lock-screen payloads are generic by default and
do not include precise coordinates, user-defined location labels, account
identifiers, or bearer links.

Inbound provider/status callbacks traverse F-18 and are not trusted merely
because they reach a callback URL. They require SEC-API-009 authentication,
environment/account/type binding, strict schema, trusted-time expiry/replay
checks, idempotency and scope-limited state transitions. Provider acceptance
must remain distinguishable from device delivery in NOTIF-010 state.

### 4.8 Delivery and cloud-management plane

Source control, CI, registries, Terraform state/plans, artifacts, signing
keys, and Azure management are a separate Critical trust domain:

- untrusted contributions do not receive production secrets or trusted
  deployment authority;
- CI jobs use isolated execution and short-lived environment-scoped workload
  identities;
- protected source and workflows produce locked dependencies, SBOM, scans,
  signed provenance, and integrity-verified artifacts;
- a reviewed immutable artifact is promoted, not rebuilt, between
  environments;
- Terraform plan and apply are bound to reviewed source/artifact and separate
  scoped authority; state is encrypted, versioned, locked, access-logged, and
  isolated by environment;
- manual production/cloud changes are audited break glass, followed by
  reconciliation and drift review;
- databases, caches, queues, object, audit, backup, secret/key, and
  observability data planes default to private access.

Specific CI, registry, Azure compute/network, identity, key, secret, state,
backup, edge, and observability products remain undecided.
Canonical domains, DNS zones, certificates, trusted email/link domains,
mobile association files, and sending reputation use explicit owners,
separate credentials, renewal/expiry monitoring, change approval, and a
tested compromise/revocation/recovery plan.

---

## 5. Trust Boundaries and Required Controls

| ID | Boundary | Data | Required controls and failure behavior | Flows / principal threats |
|---|---|---|---|---|
| TB-01 | Internet client/vendor → public edge | Public queries/callbacks; coordinates and callback metadata by classification | Approved TLS, Host/proxy normalization, DDoS/WAF, method/schema/size/cost limits, SEC-API-009 callback trust, privacy-safe logs; reject unknown routes/content | F-01/02/18; TM-06/08–10/19 |
| TB-02 | Browser/mobile → Identity/User | Credentials, sessions, email, personal resources | Separate auth/session design, CSRF where cookies apply, object/field authorization, anti-automation, no shared cache, generic failures | F-01/03/07; TM-06/08/18/21 |
| TB-03 | Managed admin device → admin edge/control plane | Roles, providers, secret references, corrections, audit queries | Distinct origin/audience, mandatory WebAuthn UV, reauth, least privilege, impact-matrix dual control, managed-access defense in depth, durable audit | F-04–06/19/20; TM-03/04/09/15 |
| TB-04 | Fetch worker → provider Internet endpoint | Provider credential, request metadata | Static approved destination, current TLS validation, DNS/IP/redirect revalidation, default-deny egress, private/link-local/metadata block, strict byte/time/retry budget | F-10; TM-01/05/11/17 |
| TB-05 | Provider/bulk artifact → quarantine/raw evidence | Untrusted payload/archive/text/URLs, at least Confidential until classified | Source metadata/authenticity, immutable digest plus independent provenance, parser isolation/limits, strict schema, safe storage/viewing; no transport credentials; never directly public | F-10–12; TM-01/05/09/11/14 |
| TB-06 | Quarantine → processing → trusted dataset | Candidate observations and provenance | Authorized producer/consumer, versioned schema, distinct historical/current/guidance/alert gates, semantic/anomaly/replay/license controls; fail closed to last eligible data | F-13–15/20; TM-01/02/11/12/24 |
| TB-07 | Service → service/queue/cache | Internal commands/events/data | Distinct workload identities, encrypted transport/storage, action/topic ACL, bounded schema/TTL, trusted-time expiry, replay/idempotency, backpressure and restricted DLQ | F-08/13/14/16/18/20; TM-02/11–13/19 |
| TB-08 | Workload → data/secret/key store | Public through Restricted data | Private endpoint/path, least-privilege identity, encryption, audited key/secret reads, separate store policies; deny on authorization uncertainty | F-06/07/09/12/15/19/23; TM-05/06/12–14/17/21 |
| TB-09 | Public API/cache → browser/mobile | Published data and provenance/status | Response schema, contextual encoding, cache version/freshness, accessible redundant status, CSP/security headers; never expose internal/quarantine fields | F-01/02/09; TM-02/09/10/13/24 |
| TB-10 | App → map/geocoder/notification processor and callback return | Query/viewport, destination token, minimal notification content/status | SEC-PRIV-007 inventory/contract, minimization, no stable account plus precise location, scoped vendor credentials, SEC-API-009 authenticated callbacks, circuit breakers | F-08/16–18/27/28; TM-07/10/11/18/19/21 |
| TB-11 | Workloads → observability/audit | Events, errors, identifiers, security evidence | Field allowlist/redaction, structured encoding, separate audit authority, immutable/WORM plus tamper evidence, trusted time/order, bounded backpressure, alert on gaps/silence; no raw secrets/payloads | F-21/22; TM-10/11/15/22 |
| TB-12 | Source/CI/registry → Terraform/Azure/resources | Code, dependencies, plans/state, artifacts, cloud authority | Protected source, authenticated/allowlisted immutable inputs, untrusted-job isolation, short-lived identity, review/approval, SBOM/provenance/signing, state controls, drift/rollback | F-26; TM-05/14/16/17 |
| TB-13 | Primary stores/config → backup/region/restore | Full classified datasets and control state | Encryption, version/immutability, separate time-bound identity, residency/retention, isolated restore, integrity and deletion/suppression preservation | F-24/25; TM-14/20/22 |
| TB-14 | Mobile app → OS/local storage/deep link/notification UI | Session, cached Public observations, device permission, navigation input | Platform secure storage, no Restricted persistence where backup exclusion fails, cache/backup/screenshot lifecycle, foreground permission, link/message validation, signed app/update, cleartext rejection | F-01/03; TM-07/08/18/19 |
| TB-15 | User → export/correction/deletion → stores/processors | Bulk personal data and lifecycle commands | Recent auth, owner scope, short-lived protected export, CSRF defense, downstream completion evidence, narrow backup/legal exceptions | F-03/07/27; TM-06/07/14/21 |
| TB-16 | Cloud/SRE/break-glass operator → JIT broker → observability/cloud/restore control planes | Internal telemetry; Restricted operational authority and full-class restore data | Separate workforce identity and phishing-resistant auth, approved JIT scope/expiry, environment/role separation, command/session evidence, independent audit/alert, no inherited application role, safe failure and termination | F-21/30; TM-03–05/14–17/20/22 |

Every boundary must be represented in deployment tests and the DATA-001
inventory. A new boundary or external recipient triggers NFR-MAINT-007.

---

## 6. Deployment Security Zones and Failure Domains

The Azure topology must preserve these logical zones even if a selected
managed service implements more than one capability:

1. **Internet edge zone** — only approved public and identity routes.
2. **Privileged administration zone** — separate client, ingress,
   authentication audience, and authorization policy.
3. **Application zone** — public/user APIs with no direct provider parsing or
   cloud-control authority.
4. **Ingestion/worker zone** — narrow provider egress and queue/data
   permissions; no public ingress.
5. **Private data zone** — no default public data-plane access; store-specific
   identities and encryption.
6. **Security/operations zone** — secrets/keys, immutable audit, redacted
   observability, incident access, and isolated backup.
7. **Delivery/cloud-management zone** — protected CI/Terraform and release
   identities; never reachable with ordinary runtime credentials.
8. **Environment and region boundaries** — separate identities, secrets,
   state, data policy, and approvals; production data is not copied to lower
   environments without an approved minimized/synthetic process.

The topology must explicitly model egress mediation, Azure metadata/managed
identity protection, private data services, DDoS/resource-cost controls,
certificate/DNS ownership, secret/key rotation, patching, workload
isolation, backup deletion protection, and security telemetry. Region choice
and active/standby behavior cannot be selected before REL-005 objectives and
privacy/data-residency constraints.

Multi-region behavior must keep authorization revocation, rule versions,
observation ordering, cache invalidation, audit sequencing, suppression/
deletion state, and notification idempotency consistent. Availability
failover may not resurrect a credential, role, record, or alert state.

---

## 7. Security and Reliability Invariants

These invariants are architecture tests, not implementation suggestions:

- Only the publication gate can write a Public observation version.
- Only the Identity service can write credentials, authenticators, sessions,
  and recovery state.
- No public-read identity can write trusted data or read personal, secret,
  audit, backup, quarantine, or control-plane data.
- No ingestion/parser identity can read unrelated secrets or call the cloud
  management plane.
- No administrator can read back a provider secret, grant their own
  privilege, approve their own high-impact action, or modify their evidence.
- No message, cache entry, retry, replay, restore, or regional failover can
  bypass current validation, authorization, version, freshness, suppression,
  deletion, or licensing policy.
- No authenticated/personal/admin/recovery response enters a shared public
  cache.
- No lock-screen or vendor payload contains a precise coordinate,
  user-defined location name, account identifier, or bearer credential by
  default.
- No Critical/High finding, unsupported dependency, unsigned/unprovenanced
  artifact, failed restore, broken audit, or unresolved release gate is
  silently waived.

---

## 8. Consolidated Threat Model

### 8.1 Method and scope

This baseline adapts the analytic methods in the supplied CycloneDX Threat
Modeling Blueprint prompt suite:

- repository reconnaissance and document/architecture conflict extraction;
- STRIDE analysis across components, explicit flows, and boundaries;
- PASTA-informed business-impact and attack-path analysis;
- LINDDUN analysis of account, location, telemetry, notification, and vendor
  flows;
- attack-tree-informed path sketches for public-data corruption,
  control-plane takeover, personal-location disclosure, and resource
  exhaustion;
- provisional FMEA scoring for safety, integrity, privacy, and availability
  failures; and
- consolidation using the highest supported risk where methods differed.

This is not a claim that every prompt-suite phase was completed. No stakeholder
interviews occurred; no per-method JSON fragments, schema-conformant CycloneDX
threat-model JSON, schema validation, or standalone final-report appendices
were produced because the requested output scope was the three existing
Markdown artifacts. Those outputs remain a gate if machine-readable threat
exchange is required. The Markdown register below adopts the suite's core
asset/flow, risk, control, evidence, ownership, and status concepts.

Processed repository artifacts were: `AGENTS.md`, `CLAUDE.md`, `README.md`,
`REQUIREMENT_TEMPLATE.md`, `REQUIREMENTS.md`, `ARCHITECTURE.md`,
`SECURITY.md`, `DESIGN.md`, and all four SVG plus four PNG logo variants under
`assets/logo/`. Markdown was read directly; SVG was inspected as XML for
active/external content and accessible labels; PNG dimensions and metadata
were inspected. No source code, dependency manifest, API specification,
deployment configuration, infrastructure code, or tests exist. Material
conflicts were consolidated by keeping undecided products/limits as gates,
separating application from cloud authority, replacing unsafe/vague security
rules, and blocking known DESIGN.md contrast failures at release rather than
silently declaring them safe.

#### 8.1.1 Processed-artifact and reconciliation log

| Artifact(s) | Security-relevant evidence extracted | Reconciliation / remaining gap |
|---|---|---|
| `AGENTS.md`, `CLAUDE.md` | Repository rules designate the four primary documents, require undecided choices to remain TBD, and confirm a greenfield design | Kept product choices as capability gates; an old CLAUDE heading reference is editorial follow-up, not treated as architecture evidence |
| `README.md` | Product summary only; no implementation, deployment, security contact, or operating evidence | SEC-IR-002 now requires a monitored private intake before production; README discoverability can follow when the channel exists |
| `REQUIREMENT_TEMPLATE.md` | Generic template is narrower than this product and does not capture all privacy, accessibility, supply-chain, residual-risk, owner, or evidence fields | This baseline and NFR-MAINT-008 matrix govern this project; template hardening is a separate repository-maintenance follow-up |
| Prior `REQUIREMENTS.md` | Broad public/mobile/admin/ingestion behavior but ambiguous trust, eligibility, privacy, admin, retry, recovery, and release rules | Added explicit threat-derived requirements, context-specific publication states, decision gates and acceptance evidence |
| Prior `ARCHITECTURE.md` | Selected logical stack inputs plus unresolved products/topology; insufficient directional flows and trust separation | Replaced with the F-01–F-31 flow registry, TB-01–TB-16 boundaries, invariants, threat register and provisional FMEA |
| Prior `SECURITY.md` | Useful baseline controls mixed with vague, optional, or unsafe implementation assumptions | Replaced with capability rules, separate identity/data/control planes, precise failure behavior, verification and risk policy |
| `DESIGN.md` | Visual/accessibility rules, chart/map/status semantics, external-font/logo usage; several current token pairings fail or need contrast validation | Stricter NFR-A11Y and release gates govern; exact known ratios are recorded in §8.7, but DESIGN changes were outside requested output files |
| Four logo SVG files | Accessible title/description markup; no script, event handlers, foreign active content, external fetches, or unsafe links observed | Still require source/license/provenance/hash and pipeline sanitization under SECURITY.md §15.4 |
| Four logo PNG files | Dimensions/standard image metadata inspected; no unexpected EXIF content observed | Still require source/license/provenance/hash, deterministic generation and crop/optimization review |

No stakeholder disagreement record existed to merge. Where documents
conflicted, the stricter safety/privacy/security rule was retained and an
unselected technology or numeric limit remained an explicit gate rather than
being invented.

AI/ML analysis was excluded because v1 contains no AI/ML system. No code,
configuration, deployment, API specification, or test evidence exists, so
repository reconnaissance could verify no controls. This is an artifact-based
single-analyst baseline, not a completed cross-functional interview.

Risk scale:

- **Critical:** feasible high-impact path with no adequate independent
  control, a path to cloud/control-plane compromise, or provisional FMEA RPN
  above 200. When methods differ, the highest supported rating governs.
- **High:** significant safety, confidentiality, integrity, or availability
  impact with incomplete or single-layer planned controls.
- **Medium:** bounded impact/exploitability but requires explicit mitigation.
- **Low:** limited residual exposure with verified controls.

The pre-implementation model contains **11 Critical, 12 High, and 1 Medium**
inherent risks. Target residual risk is Medium or lower after controls are
implemented and verified; privacy/legal and safety owners may require a lower
target.

Every threat is currently **Open**, last assessed 2026-07-17 from the
repository artifacts above, with owner **TBD**, no implementation evidence,
and a due date before its affected feature or production-bound decision is
implemented. Because no control is verified, current residual risk is **Not
assessed and treated as inherent for gating**. The last column is the target
residual after independent implementation and verification, not the current
state. Where SEC-IR-003 permits acceptance, any deviation requires its
evidence and approval; a Critical finding still blocks initial production.

### 8.2 Threat register

| ID | Methods / category | Attack scenario and impact | Affected assets / flows | Inherent | Required controls | Target residual |
|---|---|---|---|---|---|---|
| TM-01 | STRIDE S/T/R; PASTA-informed; FMEA | A spoofed or compromised authenticated provider submits plausible false data that passes syntax checks and becomes public guidance/alerts | Provider identity, trusted observations; F-10–16 | Critical | ING-004/010/015–017, AQ-006–010, REL-002/003, TB-04–06 | Medium |
| TM-02 | STRIDE T/D; FMEA | Replay, stale failover, trusted-time corruption, unit/standard mix-up, or cache-version error presents old/incorrect data as current or emits false recovery | Time/order, observations, alerts/cache; F-09/10/14–18/21/25 | Critical | SEC-008, WEB-006, ING-014/017, AQ-008–010, NOTIF-007, REL-001/008 | Low |
| TM-03 | STRIDE S/E/R; attack-tree-informed; FMEA | Weak admin bootstrap, authenticator enrollment/replacement/removal, recovery, or stolen session bypasses mandatory passkey and takes the control plane | Admin identity/control plane; F-04–06/19/20 | Critical | ADMIN-001/009–011, SEC-AUTH-003/006/007, SEC-SESS-005, TB-03 | Medium |
| TM-04 | STRIDE E/T/R; PASTA-informed | Overprivileged/malicious admin self-elevates, redirects providers, changes AQI/guidance, suppresses data, exports PII, or impairs audit | Roles/rules/provider config/audit; F-05/19–21 | High | ADMIN-002/010/012, SEC-AUTHZ-006, AQ-008/010, SEC-LOG-005/006 | Medium |
| TM-05 | STRIDE I/E/T; attack-tree-informed | Provider URL/test/backfill exploits redirect, DNS, alternate IP, or Azure metadata SSRF to steal workload identity and pivot to secrets/cloud | Provider credential/cloud authority; F-10/20/23/26 | Critical | SEC-INPUT-005, ING-015/018, SEC-SVC-001/003, narrow fetch identity, TB-04 | Low |
| TM-06 | STRIDE I/T/E; LINDDUN Identifiability/Disclosure; FMEA | BOLA/BFLA/mass assignment exposes or changes another user's profile, sessions, saved locations, alert destinations, export, or deletion | Personal/identity data; F-03/06/07/27 | Critical | SEC-AUTHZ-001–007, API-011/012, DATA-004, negative authorization tests | Low |
| TM-07 | LINDDUN Linkability/Identifiability/Disclosure/Unawareness/Non-compliance; FMEA | Map/geocoder/push/analytics vendors correlate precise queries, account/device/IP identifiers, and alert behavior into home/work/routine profiles | Exact/saved location and vendor records; F-01/08/17/18/27/28 | Critical | SEC-PRIV-006/007/009/010, DATA-001/005, privacy-preserving flow decision | Medium |
| TM-08 | STRIDE S/T/E | Credential stuffing, enumeration, reset races, session fixation/replay, insecure account change, or takeover of canonical domain/DNS/certificate/app-link routing takes a consumer account | Account/session/trusted domains; F-01/03/06/08 | High | USER-003/004/009/010, SEC-AUTH-004–008, SEC-SESS-001–004, domain ownership/monitoring | Low |
| TM-09 | STRIDE T/E/I | Provider text, admin annotations, raw files, URLs, logs, CSV, deep links, or a WebView triggers stored XSS/injection in a privileged client | Clients/admin/session; F-01/04/05/11/12/22 | High | SEC-INPUT-003/004/006–008, SEC-MOB-007, safe quarantine viewer, CSP | Low |
| TM-10 | STRIDE D; PASTA-informed; FMEA | Expensive history/map/search/auth operations bypass limits and exhaust database, edge, geocoder, observability, or cloud spend | Availability/cost budgets; F-01–03/09/22/28 | Critical | API-013, NFR-PERF-004, NFR-AVAIL-004/005, edge budgets, load/abuse tests | Medium |
| TM-11 | STRIDE D/T; FMEA | Provider retries, archive expansion, backfill, queue replay, rule change, notification retry, or callback flood creates a worker/storage/fan-out storm | Worker/queue/storage/vendor capacity; F-08/10–18/20/22 | Critical | ING-007/013/018, NOTIF-002/007/009, NFR-SCALE-004, queue/backpressure limits | Low |
| TM-12 | STRIDE S/T/E/R | Compromised internal service or forged/replayed queue message gains authority to publish, notify, read data, or mutate configuration | Workload identities/messages; F-08/13–16/18/20 | High | DATA-004, SEC-AUTHZ-007, SEC-SVC-002, per-topic/store IAM, TB-07/08 | Low |
| TM-13 | STRIDE T/I/D | Cache-key ambiguity, untrusted write, missing invalidation, or private-response caching leaks users or serves poisoned/stale data | Public cache/personal responses; F-02/03/09/15 | High | API-012, AQ-010, SEC-API-006, public-only cache design and version keys | Low |
| TM-14 | STRIDE I/T/E/D | Publicly exposed or overprivileged database/object/queue/backup/state leaks Restricted data or enables ransomware/tampering | Every store/backup/state; F-07/09/12–15/19/23–26 | High | DATA-003/004, SEC-SVC-001/004, SEC-SUP-004, REL-004–008 | Medium |
| TM-15 | STRIDE T/R/I/D; FMEA | Actor alters/deletes audit evidence, injects misleading logs, leaks secrets/PII through errors/diffs, or exhausts audit capacity so high-impact actions fail or go unrecorded | Audit/observability/evidence; F-21/22 | Critical | ADMIN-008/012, SEC-LOG-001–007, separate authorities, capacity/redaction tests | Low |
| TM-16 | STRIDE T/E/I; attack-tree-informed; FMEA | Malicious dependency, build action, PR, runner, registry, artifact, or Terraform workflow steals delivery authority and deploys a backdoor | Source/build/signing/cloud; F-21/23/26 | Critical | NFR-MAINT-006/008, SEC-SUP-001–004, protected promotion and short-lived identity | Medium |
| TM-17 | STRIDE I/E/D | Secret values leak through source/state/log/client/admin paths, overbroad identities enable lateral movement, or key/identity-plane failure causes unsafe fail-open behavior or prolonged outage | Secrets/keys/IAM/availability; F-06/10/17/19/23/26 | High | SEC-004/008, DATA-003/004, write-only secret UI, rotation, cached-safe failure and audit | Low |
| TM-18 | STRIDE S/I/T/E; LINDDUN Disclosure | Lost/malicious device, insecure local cache/backup, long-lived token, deep link, WebView, or unsigned update exposes account/location or invokes privileged behavior | Mobile session/location/app package; F-01/03/08/17 | High | MOB-003/004, SEC-MOB-001–008, distinct mobile session design, TB-14 | Medium |
| TM-19 | STRIDE I/T/S/E; LINDDUN Detectability/Disclosure | Lock-screen/email content, malicious bearer/deep link, spoofed callback, or sending-domain abuse reveals location, falsifies delivery state, or becomes a phishing/state-change channel | Notification content/status/trusted links; F-08/16–18 | High | NOTIF-005/007–010, SEC-API-009, SEC-PRIV-010, authenticated app resolution, safe link domains | Low |
| TM-20 | STRIDE T/D; FMEA | Restore/region failover resurrects revoked roles, credentials, suppressed/deleted records, old rules, cache, or notifications | Backup/control/data state; F-24/25 | Critical | REL-004–008, DATA-005, versioned control state, isolated restore tests | Low |
| TM-21 | STRIDE I/S/T; LINDDUN Disclosure/Non-compliance | Weak export/delete identity proof or downstream propagation discloses another user or leaves undeleted processor/cache copies | Personal data/rights evidence; F-03/07/27 | High | USER-008, DATA-005, SEC-PRIV-008, TB-15, protected short-lived exports | Low |
| TM-22 | STRIDE R/D; PASTA-informed | Missing/silent detection, unclear ownership, absent incident playbook, or loss of identity/key/alert access lets takeover, poisoning, leak, or outage persist | Detection/response/public trust; F-21/22/25 | High | OBS-004/005, SEC-LOG-007, SEC-IR-001–003, exercised response | Medium |
| TM-23 | PASTA-informed business/compliance | Scraping, cache/export, or provider-policy change violates license/attribution and forces withdrawal or legal/service disruption | Provider licenses/public data; F-09/10/12/15/27 | Medium | REQUIREMENTS.md §5.5, ING-016, AQ-009/010, API-013, SEC-007 | Low |
| TM-24 | STRIDE T/R/D; safety analysis | Compromised or erroneous AQI, aggregation, freshness, translation, source selection, health guidance, or inaccessible/low-contrast/color-only status causes users—including disabled users—to misread conditions | Rules/guidance/status/accessibility; F-02/09/14–16/19 | High | NFR-MAINT-002, NFR-A11Y-001–005, AQ-008–010, ADMIN-010, signed/provenanced release, golden/accessibility tests | Low |

### 8.3 STRIDE component coverage

| Surface | Spoofing | Tampering | Repudiation | Information disclosure | Denial of service | Elevation of privilege |
|---|---|---|---|---|---|---|
| Public/authenticated API and status UI | TM-08 | TM-06/09/13/24 | TM-08/24 | TM-06/09/13 | TM-10/13/24 | TM-06/08/09 |
| Administrative control plane | TM-03 | TM-04/09/15 | TM-03/04/15 | TM-05/09/15 | TM-15/22 | TM-03–05/09 |
| Provider ingestion/publication | TM-01 | TM-01/02/11/24 | TM-01 | TM-05 | TM-02/11/24 | TM-05 |
| Services/queues/caches | TM-12 | TM-11–13 | TM-12/15 | TM-12/13/17 | TM-11/13/15/17 | TM-12/17 |
| Stores/backups/regions | TM-21 | TM-14/15/20 | TM-15 | TM-14/15/20/21 | TM-14/15/20 | TM-14/21 |
| Mobile/notifications/vendors | TM-08/18/19 | TM-18/19 | TM-19 | TM-07/18/19 | TM-11/18/19 | TM-18/19 |
| CI/Terraform/cloud control | TM-16 | TM-14/16 | TM-15/16 | TM-14/16/17 | TM-14/16/17 | TM-05/14/16/17 |

Coverage means analyzed, not controlled or verified.

### 8.4 LINDDUN privacy analysis

| LINDDUN dimension | Threat in this system | Threats / assets / flows | Required response |
|---|---|---|---|
| Linkability | Stable account/device/push/correlation identifiers link searches, saved locations, map use and alerts across services/vendors | TM-07/13/19; location/account/vendor data; F-01/03/07/08/17/18/22/28 | Minimize identifiers, isolate purposes, short retention, no stable account plus precise vendor query |
| Identifiability | Email plus exact/saved locations and alert patterns reveal home/work, identity, routines, or health-related behavior | TM-06/07/18/21; identity/location/alerts; F-03/07/08/17/27/28 | Confidential/Restricted classification, owner authorization, masked support access, vendor minimization |
| Non-repudiation | Long-lived audit/telemetry may over-retain ordinary user behavior beyond its security purpose | TM-15/21/22; audit/telemetry/rights evidence; F-21/22/27 | Audit only security-relevant user events, pseudonymize/minimize, purpose-bound retention and rights analysis |
| Detectability | Enumeration, timing, lock-screen content, delivery metadata, or vendor queries reveal that a person/account/location is monitored | TM-07/08/19; account/delivery metadata; F-01/03/08/17/18/28 | Uniform auth/privacy responses, generic previews, bounded metadata, protected delivery history |
| Disclosure | BOLA, shared cache, logs/traces, local cache, exports, backups, or processors expose personal data | TM-06/07/13/14/18/19/21; personal/backup/vendor data; F-03/07–09/17/18/22/24/25/27/28 | Object/field authorization, private cache, redaction, secure client lifecycle, export/backup/processor controls |
| Unawareness | Device permission notice omits saved/search location, map/geocoder, push, analytics, retention, or downstream deletion | TM-07/18/19/21; consent/location/processor state; F-01/03/08/17/27/28 | Layered plain-language notices and just-in-time choice mapped to the processing inventory |
| Non-compliance | Jurisdiction, lawful basis, transfer, processor, minors, retention, DPIA, licensing, or rights deadlines are unresolved | TM-07/21/23; contracts/licenses/rights evidence; F-10/12/15/17/27/28 | Privacy/legal and provider-policy approval is a launch gate, not a post-launch task |

### 8.5 High-value attack-path sketches

These sketches are attack-tree-informed prioritization aids, not full
quantified attack trees. OR/AND labels show the principal composition known
from repository artifacts; cross-functional workshops must validate and
expand preconditions and shared/common-cause nodes.

**Goal A — publish false or misleading air-quality output**

- OR compromise/impersonate an approved provider and send plausible data;
- OR change a provider endpoint/credential, then feed attacker data;
- OR exploit provider-test SSRF and obtain cloud/runtime authority;
- OR compromise an admin and alter rules, suppressions, corrections or
  source-selection policy;
- OR forge/replay a queue event or write trusted storage/cache directly;
- OR compromise the rule/release supply chain; or
- OR use stale failover, unit/time/standard ambiguity, failed invalidation,
  low contrast, or color-only/misleading status presentation.

Shared defensive nodes: source/destination assurance, quarantine publication
gate, immutable provenance, independently approved rule/admin changes,
per-workload write authority, cache versioning, redundant text/icon/pattern
status, tested contrast and assistive-technology behavior, and
audit/detection.

**Goal B — take administrative or cloud control**

- OR bypass bootstrap/recovery or add an attacker WebAuthn credential;
- OR steal/replay an admin session from a compromised device/client;
- OR exploit stored provider content in the admin viewer;
- OR self-elevate through BFLA/mass assignment/role workflow;
- OR SSRF from provider configuration into managed identity/metadata; or
- OR compromise source, CI, registry, Terraform state, plan/apply, or signing
  credentials.

Shared defensive nodes: separate workforce identity/audience, WebAuthn user
verification and two-person recovery, sensitive-action reauthentication,
operation matrix/dual control, narrow workload identities, protected
delivery provenance, and separate tamper-evident audit authority.

**Goal C — disclose or infer a user's sensitive location**

- OR BOLA through saved locations/alerts/notification history/export;
- OR correlate search/IP/account/device/push data in logs or a vendor;
- OR serve a personalized response from a shared cache;
- OR read a mobile cache, backup, task-switcher preview, or long-lived token;
- OR observe a lock-screen/email notification or malicious deep link; or
- OR access an overbroad store, backup, support/admin dashboard, or processor.

Shared defensive nodes: DATA-001 classification, owner/field authorization,
ephemeral/coarse lookup, vendor minimization, private cache policy, client
lifecycle, generic notification previews, retention/deletion propagation.

**Goal D — deny service or create unbounded cost**

- AND/OR distribute expensive historical/map/search requests and cache
  busters;
- flood registration/login/reset/verification or public API;
- deliver oversized/slow/compressed provider data and force retries;
- trigger backfill/queue poison/replay/cache stampede;
- create large alert fan-out and delivery retry storms; or
- exhaust object storage, logs, queue, database, vendor or cloud scaling
  budgets.

Shared defensive nodes: early cost-class budgets, per-principal and global
limits, bulkheads, cancellation/timeouts, retry ceilings, backpressure/DLQ,
storage/cost alerts, and graceful degradation that preserves last eligible
data without mislabeling it current.

### 8.6 Provisional FMEA

Scores use `RPN = Severity × Occurrence × Detection` on 1–10 scales:

- **Severity:** 1 is negligible; 10 is catastrophic safety, widespread
  privacy, persistent control-plane, or irrecoverable service impact.
- **Occurrence:** 1 is exceptional; 10 is expected/frequent in the intended
  environment. With no incident data, these are conservative hypotheses.
- **Detection:** 1 means the condition is almost certain to be detected and
  contained before impact; 10 means pre-impact detection is unlikely.

RPN above 200 is Critical under the supplied method. The highest rating from
FMEA or the qualitative register governs, which is why eight threats that
would otherwise be High are Critical in §8.2. Ratings are provisional because
no SRE, air-quality domain, privacy, product, or incident-history calibration
exists. The assessor was the 2026-07-17 repository-artifact security analysis;
there are **no verified existing controls** for any row.

| Component / threat | Failure mode and cause | Effect | S | O | D | RPN | Verified existing control / assessed by |
|---|---|---|---:|---:|---:|---:|---|
| Ingestion/publication — TM-01 | Compromised authenticated provider sends plausible poison that syntax/range checks accept | False current guidance and mass alerts | 10 | 5 | 8 | 400 | None; repository-artifact security analysis |
| Admin identity/control — TM-03 | Bootstrap, recovery, authenticator, or role workflow bypass | Full control-plane and public-data compromise | 10 | 4 | 8 | 320 | None; repository-artifact security analysis |
| CI/Terraform/cloud — TM-16 | Dependency, runner, registry, signing, plan/apply, or artifact compromise | Persistent cloud/data compromise | 10 | 4 | 8 | 320 | None; repository-artifact security analysis |
| Backup/region — TM-20 | Restore is unusable or promotes stale/revoked/deleted control/data state | Extended outage, ransomware loss, resurrected state | 9 | 4 | 8 | 288 | None; repository-artifact security analysis |
| Public edge/API — TM-10 | Distributed expensive query/cache-buster bypasses cost budgets | Outage, stale service and unbounded spend | 8 | 7 | 5 | 280 | None; repository-artifact security analysis |
| Vendor/privacy — TM-07 | Stable identity/IP/device is combined with exact location across processors | Stalking, profiling, breach and legal harm | 8 | 5 | 7 | 280 | None; repository-artifact security analysis |
| Authorization/rights — TM-06 | Object/field/function ownership check is missing or inconsistent | Cross-user location/account disclosure or destructive change | 9 | 5 | 6 | 270 | None; repository-artifact security analysis |
| Time/order/publication — TM-02 | Clock corruption, replay, stale failover, or version/order error | False current/recovery status and loss of trust | 9 | 5 | 6 | 270 | None; repository-artifact security analysis |
| Audit/detection — TM-15 | Writer tampering, redaction failure, shared-plane outage, or backpressure exhaustion | Attack persists or action cannot be reconstructed; privileged work halts | 8 | 4 | 8 | 256 | None; repository-artifact security analysis |
| Ingestion/queue/notification — TM-11 | Retry, expansion, replay, backfill, callback, or fan-out loop | Worker/storage/vendor exhaustion and notification storm | 8 | 6 | 5 | 240 | None; repository-artifact security analysis |

Residual scores below are targets only. They assume every listed control is
implemented, independently tested, monitored, and owned; until then current
residual remains Not assessed/inherent.

| Threat | Planned mitigation | Residual S | Residual O | Residual D | Target RPN | Validation owner |
|---|---|---:|---:|---:|---:|---|
| TM-01 | Source/destination assurance, immutable provenance, anomaly/conflict policy, context-specific publication gate and independent detection | 10 | 2 | 3 | 60 | Security + air-quality domain, TBD |
| TM-03 | Exact-origin WebAuthn UV, separate admin audience, two-person authenticator/recovery lifecycle, JIT privilege and audit | 10 | 2 | 3 | 60 | Security + identity owner, TBD |
| TM-16 | Protected source/workflows, authenticated immutable inputs, isolated CI, short-lived identities, signed provenance and plan-bound promotion | 10 | 2 | 3 | 60 | Security + delivery/cloud owner, TBD |
| TM-20 | Separate immutable backup authority, isolated restore, security/data-state validation, RPO/RTO and ransomware exercises | 9 | 2 | 3 | 54 | SRE + data/privacy owners, TBD |
| TM-10 | Early per-cost-class and global budgets, cancellation, cache/bulkheads, spend/storage alerts and degraded service | 8 | 3 | 3 | 72 | SRE + API owner, TBD |
| TM-07 | Direct/proxy/on-device privacy decision, minimization, unlinkability, contracts, retention/deletion and privacy testing | 8 | 3 | 4 | 96 | Privacy/legal + product, TBD |
| TM-06 | Central object/field/function policy, explicit DTOs and complete negative authorization matrix | 9 | 2 | 3 | 54 | Security + identity/API owner, TBD |
| TM-02 | SEC-008 trusted time/skew detection, immutable versions, freshness/order state, failover and alert-boundary tests | 9 | 2 | 2 | 36 | SRE + data-quality owner, TBD |
| TM-15 | Separate WORM and tamper-evident authority, capacity reserve, fail-closed privileged writes, gap/silence detection and redaction tests | 8 | 2 | 2 | 32 | Security + SRE, TBD |
| TM-11 | Input/expansion/retry/backfill/callback/fan-out ceilings, idempotency, DLQ/backpressure, isolation and cost alerts | 8 | 2 | 3 | 48 | SRE + ingestion/notification owners, TBD |

Common-cause failures must be tested, not scored as independent: a malicious
CI/config release can affect every region; domain/DNS/certificate takeover can
affect authentication and callbacks; trusted-time failure can affect sessions,
freshness, replay and audit; key/identity-plane loss can disable control and
recovery; replicated stale state can corrupt failover; and a shared audit/
observability dependency can both hide an attack and block safe operations.

### 8.7 Cross-functional coverage and limitations

| Perspective | Coverage in this baseline | Required validation |
|---|---|---|
| Security architecture | Assessed from documents; single analyst | Independent security review and penetration-test plan |
| Software/data engineering | Partial, inferred from provisional design | Validate feasibility, schemas, consistency, and testability |
| Air-quality / public-health domain | Not represented | Validate ranges, source confidence, aggregation, freshness, guidance and alert hazards |
| Privacy/legal | Not represented | Validate jurisdiction, purpose/basis, processors/transfers, retention, minors, DPIA and rights |
| SRE/Azure/operations | Not represented | Calibrate availability/cost/FMEA, topology, identity, backup, monitoring and incident response |
| Mobile/client/accessibility | Partial documentation only | Validate sessions, local lifecycle, notifications, deep links, vendor flows, WCAG and known token contrast issues |
| Product/business/support | Not represented | Validate impact, abuse, support access, notification promises and acceptable residual risk |

The artifact review found unresolved DESIGN.md contrast hazards: white on
AQI green `#00A65A` is approximately 3.18:1, blue link `#1C7FB8` on white
4.40:1, success text `#3E9E4F` on white 3.38:1, and muted text `#8A99A8` on
white 2.92:1. The broad roles assigned to `#5FBB46`, `#E0A400`, and
`#1FA7C4` also require context-specific text/graphical contrast checks. These
are not accepted controls: REQUIREMENTS.md §12 and SECURITY.md §16 block the
affected use until token/component testing and accessible redundant status
pass. DESIGN.md itself was not changed because the requested output scope was
limited to the three threat-model artifacts.

No cross-functional disagreements were available to preserve. This absence is
a coverage gap, not consensus. NFR-MAINT-007 and the final cross-functional
acceptance item in REQUIREMENTS.md §12 require
validation and recording of disagreements before production. The prompt
suite supports professional work but does not replace accountable domain,
legal, privacy, engineering, operations, or security sign-off.

---

## 9. Open and Deferred Decision Gates

No item below may be chosen solely by a coding agent. Each requires an owner,
dated ADR/rationale, threat impact, testable criteria, rollback/exit plan, and
approval. “Architecture freeze” means before code depends on the choice;
“production gate” means before any production data or user is exposed.

| Decision | Gate | Minimum security/privacy/safety criteria |
|---|---|---|
| Canonical product domains, admin origin, email/link domains, app IDs and WebAuthn RP IDs | Before auth implementation | Ownership/certificate plan, origin/RP binding, trusted links, no open redirect, domain-compromise response |
| Node.js framework/version | Architecture freeze | Supported lifecycle; centralized schema/auth/error/security headers; safe proxy/Host/body/time defaults; update record |
| Web rendering and browser session | Architecture freeze | Cookie/token decision, robust CSRF, CSP nonce/hash support, SSR/CDN private-cache isolation, no persistent browser bearer token |
| Mobile session and secure storage | Architecture freeze | Short access lifetime, renewal rotation/replay detection, revocation, device/logout cleanup, deep links, backup/screenshot policy |
| WebAuthn library/admin authenticator policy | Before admin implementation | Standards conformance, UV enforcement, challenge/RP/origin checks, bootstrap, multiple authenticators, replacement/recovery, break glass |
| Admin permission/approval/JIT model | Before admin implementation | Complete operation matrix, SoD, no self-elevation/approval, support masking, reauth, dual control, termination and audit |
| Provider(s) and launch geography | Before ingestion implementation | Ownership/authenticity, license, schemas, integrity/checksum, rate/size, incidents, compromise/revocation, privacy/residency |
| AQI/default standard/conversion/conflict/aggregation/freshness/guidance | Before publication path | Authoritative source and golden vectors, immutable versions, domain review, uncertainty/confidence, approval/rollback, display/alert policy |
| Map/geocoder and flow pattern | Before map/search implementation | Direct/proxy/on-device privacy analysis, field/precision minimization, license/cost, key scope, destination/egress, outage/deletion |
| Notification channels/provider | Before alert implementation | Verified recipients, minimal payload, link domains, credential scope, callbacks, fan-out/retry/expiry, consent/deletion, SPF/DKIM/DMARC for email |
| Service granularity | Architecture freeze | Workload identities, east-west policy, blast radius, operational ownership, transactional/outbox needs, deployment/rollback capacity |
| Operational/identity/historical engines | Architecture freeze | Data-class separation, private access, encryption/key policy, least privilege, PITR, consistency, deletion/TTL, audit and restore |
| Queue/event bus | Architecture freeze | Producer/consumer IAM, private encrypted path, schema/size/TTL, ordering/dedup/replay, DLQ, quotas and failover |
| Cache | Architecture freeze | Public-only or strict isolation, semantic keys, trusted writers, TTL/freshness, invalidation, stampede control, regional behavior |
| Raw/quarantine object storage/viewer | Architecture freeze | Private origin, digest/versioning, safe rendering/download, archive isolation, malware/content policy, lifecycle/license and reprocessing role |
| Audit/observability stack | Architecture freeze | Separate authority, tamper evidence, trusted time/order, field redaction, private access, retention, alert/runbook and silence detection |
| Azure compute/network/subscription/region | Architecture freeze | Edge/DDoS, private data services, egress/metadata protection, workload identity, patch/isolation, environment separation, residency, cost and failover |
| Secret/key manager | Architecture freeze | Managed identity, per-service/environment scope, rotation/revocation/versioning, audit, recovery, state/plan leakage prevention |
| CI/CD/registry/Terraform state | Before production-bound merge | Protected refs/workflows/environments, untrusted PR isolation, short-lived identity, plan/apply binding, SBOM/provenance/signing, state security, rollback |
| SLO/RPO/RTO/staleness/cost budgets | Before storage/region choice | Numeric objectives per critical data/service, measurement/error budget, restore/failover evidence, maximum alert delay and public-data age |
| Privacy jurisdictions/processors/retention/minors/analytics | Before collection | Data map, purpose/basis, DPIA decision, contracts/transfers, exact retention/deletion, consent/rights, age policy, breach response |
| Public/developer API tier | Before external developer launch | Anonymous/keyed scopes, hashed credentials, quotas/abuse attribution, license/redistribution enforcement, privacy-safe telemetry and revocation |

---

## 10. Requirement Traceability

| Architecture element | Principal requirements | Status |
|---|---|---|
| Public web client | WEB-001–007, MAP-*, HIST-*, NFR-A11Y-*, SEC-API-006 | Logical boundary defined; rendering/session/vendor choices TBD |
| React Native clients | MOB-001–005, SEC-MOB-001–008, NOTIF-008 | Logical boundary defined; session/modules/release choices TBD |
| Administrative client/edge | ADMIN-001–012, SEC-API-007, SEC-SESS-005 | Required boundary defined; access product and role matrix TBD |
| Public edge/API and callbacks | API-001–013, SEC-API-001–009, SEC-008, NFR-AVAIL-004/005 | Capabilities and F-18 callback boundary defined; Azure products/protocols/limits TBD |
| Identity/User | USER-001–010, SEC-AUTH-*, SEC-SESS-*, SEC-PRIV-* | Policies defined; browser/mobile implementation TBD |
| Ingestion/quarantine | ING-001–018, SEC-INPUT-004–008, SEC-SVC-003 | Trusted state machine defined; providers/storage/egress TBD |
| Processing/publication | AQ-001–010, REL-001–003, NFR-MAINT-002 | Authority and gates defined; domain policy decisions TBD |
| Notification | NOTIF-001–010, SEC-API-009, SEC-PRIV-010, NFR-SCALE-004 | Trust/failure/callback rules defined; launch channel/vendor TBD |
| Service/queue boundary | DATA-004, SEC-AUTHZ-007, SEC-SVC-001–004 | Mandatory model defined; products/granularity TBD |
| Data/cache/object stores | DATA-001–005, API-012, REL-004–008 | Logical separation and properties defined; engines TBD |
| Audit/observability/trusted time | ADMIN-008, SEC-008, SEC-LOG-001–007, OBS-001–005 | Separate plane and time/order failure behavior required; products/sources/retention TBD |
| CI/Terraform/Azure | NFR-MAINT-005/006/008, SEC-SUP-001–004 | Trust boundary and gates defined; tooling/topology TBD |
| Backup/region/recovery | REL-004–008, DATA-003/005 | Invariants defined; objectives/topology/tooling TBD |
| Threat/security governance | NFR-MAINT-007, SEC-IR-001–003 | Baseline created; owners and cross-functional validation TBD |

---

## 11. Assumptions and Explicit Non-Assumptions

Assumptions retained:

1. Web and mobile use the same documented business API contracts, but may use
   distinct session/transport adapters.
2. Logical component boundaries may be deployed as services or a modular
   monolith if workload/store authorization and failure isolation remain
   enforceable.
3. Providers and vendors remain replaceable behind adapters.
4. Azure and Terraform remain selected inputs.

Not assumed:

- Internal network placement, managed hosting, a secret manager, TLS, a
  passkey, an allowlist, encryption, or append-only storage automatically
  provides correct authorization, integrity, recovery, or privacy.
- A provider is trustworthy merely because it is approved.
- A configured hostname is safe from redirect, DNS, private-address, metadata
  or credential-forwarding SSRF.
- Syntax/range validation establishes scientific correctness.
- A URL base path establishes administrative isolation.
- Backups are recoverable, audit is immutable, alerts are delivered, caches
  are public-only, or CI is trusted until verified by tests/evidence.
- GDPR or another privacy regime can be deferred until after global users
  arrive.
- A health-information disclaimer permits misleading current/stale labels or
  unsafe notification behavior.

---

*Architecture and threat-model baseline v0.2. Re-run and cross-functionally
validate the model when any NFR-MAINT-007 trigger occurs and before
production.*
