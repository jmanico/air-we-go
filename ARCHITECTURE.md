# Air We Go — Architecture

Status: **provisional / bootstrap**. This document defines architecture
inputs and a first-pass system architecture for Air We Go. It is derived from
[`REQUIREMENTS.md`](./REQUIREMENTS.md) (source of truth for behavior) and
[`DESIGN.md`](./DESIGN.md) (source of truth for client visual/UX language). No
implementation exists yet — nothing below is inferred from code. Where
REQUIREMENTS.md and the prompt do not specify a choice, it is left as an
explicit unknown rather than guessed.

---

## Required Architecture Inputs

- `Requirements source: REQUIREMENTS.md`
- **System purpose:** *(derived from REQUIREMENTS.md §1.2)* A public-facing
  platform that ingests global air-quality measurements from one or more
  public data providers, stores current and historical observations, and
  presents them through a consumer website and mobile application — with
  live conditions, map visualization, historical/trend analysis, public and
  authenticated experiences, and administrative management/monitoring.
- **Primary use cases:** *(derived from REQUIREMENTS.md §3, §4, §11, §12)*
  - View current air quality by location and on an interactive map (public)
  - Search locations by city, region, country, postal code, coordinates, or station, with disambiguation
  - View a location detail page: index, category, pollutants, source, timestamp, health guidance
  - Browse historical data by date range, pollutant, and aggregation interval; view trend summaries
  - Save locations, set preferences (index standard, units, time zone, language), and configure threshold alerts (authenticated)
  - Ingest data from pluggable providers on a schedule, validate, normalize, deduplicate, and store it with provenance
  - Detect and communicate data freshness/staleness and provider conflicts
  - Administer providers, review/correct data quality, manage users and roles, and audit administrative actions
- **Target users / actors:** *(REQUIREMENTS.md §1.3)* Public User (no
  account), Authenticated User, Administrator (roles: Support Administrator,
  Data Administrator, Security Administrator, System Administrator,
  Read-Only Auditor per ADMIN-002), and External Air-Quality Data Provider
  (system actor, not a UI user).
- **Runtime environment:** Web application and a mobile app (iOS and
  Android) *(human-provided)*.
- **Server framework:** Node.js *(human-provided)*.
- **Client framework:** React (web), React Native (iOS, Android)
  *(human-provided)*.
- **API style and integration model:** REST *(human-provided)*.
- **Authentication and session model:** Passkeys are required for
  administrators; password authentication is required for all other
  accounts, with passkey available as an optional additional/alternative
  method; non-administrative (public) air-quality data is viewable without
  an account *(human-provided)*. This resolves REQUIREMENTS.md Open
  Question §13.17 and hardens ADMIN-001/SEC-AUTH-003 ("should" require
  phishing-resistant MFA) to a firm requirement for admins. It satisfies
  WEB-005 (no-account public access) and selects one authentication path
  from the options listed in USER-002; federated identity is not selected
  for v1.
- **Data model expectations:** *(derived from REQUIREMENTS.md §5)* Core
  entities: User, UserSession, UserPreference, SavedLocation, Alert,
  Notification, Provider, ProviderConfiguration, IngestionRun,
  RawProviderRecord, MonitoringStation, GeographicLocation,
  AirQualityObservation, PollutantMeasurement, AirQualityStandard,
  DataQualityFlag, AdministrativeAnnotation, AuditEvent. Observations need a
  stable composite identity (§5.2); locations carry coordinates, admin
  region hierarchy, time zone, and provider location ID (§5.3); retention
  periods are configurable per data class and constrained by provider
  licensing (§5.4–5.5).
- **Deployment model:** Microsoft Azure, provisioned via Terraform
  *(human-provided)*.
- **Scale expectations:** Global — global user base and global
  provider/station coverage *(human-provided)*.
- **Security expectations:** The administrative surface is the primary
  hardening target — strong/phishing-resistant authentication, strict
  least-privilege RBAC, and full tamper-evident audit. End-user data
  protection is baseline for this phase — not a current investment area
  beyond what REQUIREMENTS.md states as mandatory (§7) *(human-provided,
  reconciled with REQUIREMENTS.md §7)*.

---

## Initial Architecture (Provisional)

### Client tier

- **Web client (React):** responsive SPA/web app implementing WEB-\*,
  MAP-\*, and HIST-\* requirements, plus authenticated account/preference/
  alert screens (USER-\*). Consumes the REST APIs directly.
  **Unknown:** rendering strategy (client-rendered SPA vs. server-rendered)
  is not specified by REQUIREMENTS.md or the prompt — deferred.
- **Mobile clients (React Native, iOS + Android):** implements MOB-001–005 —
  feature parity with the website, device-location lookup gated on OS
  permission with graceful fallback when denied, temporary offline caching
  of recently viewed data with visible staleness, and push notifications
  for alerts. Shares non-platform business logic with the web client where
  practical.
- Both clients implement the DESIGN.md system: WCAG 2.2 AA, air-quality
  categories encoded by color **+** shape **+** label **+** number (never
  color alone), chart/map data available as an accessible table, localized
  dates/units/text (NFR-I18N-\*).
- **Assumption:** no separate backend-for-frontend service — both clients
  call the same versioned REST API surface, consistent with the single
  "REST" integration model given as input.

### Backend services (Node.js)

Adapted from the provider/consumer/admin split already suggested in
REQUIREMENTS.md §10, expressed as REST-exposed Node.js services:

1. **Public API Service** — current observations, historical/trend queries,
   geographic search, public metadata (WEB-\*, MAP-\*, HIST-\*, API-001,
   API-004–010).
2. **Identity & User Service** — registration, password + passkey
   authentication, sessions, preferences, saved locations, alert
   configuration (USER-\*, SEC-AUTH-\*, SEC-SESS-\*, API-002).
3. **Ingestion Service** — provider polling and backfill behind a stable
   adapter interface, validation, normalization, idempotent processing,
   bounded-backoff retry, per-provider rate-limit compliance, quarantine of
   invalid records (ING-\*).
4. **Air-Quality Processing Service** — index calculation and documented
   standard conversions, unit normalization, geographic aggregation with
   raw/regional/platform-value distinction, trend summaries, freshness
   classification (AQ-\*, MAP-006, HIST-004/007, REL-001/002).
5. **Notification Service** — threshold evaluation on new observations,
   duplicate suppression, recovery notifications, multi-channel delivery
   logging (NOTIF-\*).
6. **Administrative Service** — provider configuration, data-quality
   review/suppression/annotation, user and role management, operational
   dashboard data, audit log access; isolated from the Public API and
   protected by explicit authorization (ADMIN-\*, API-003).
7. **Audit & Logging** — cross-cutting: every Identity, Admin, and
   Ingestion mutation emits a tamper-evident audit record (ADMIN-007/008,
   SEC-LOG-\*).

- **Assumption:** each numbered service is an independently deployable
  Node.js process (not necessarily one per requirement group), so that
  ingestion workers, historical-query paths, and notification workers can
  scale independently per NFR-SCALE-001/003. Exact service granularity
  (seven separate deployables vs. a modular monolith with the same logical
  boundaries) is **unknown** — revisit once team size and operational
  capacity are known.
- The Ingestion Service's provider adapter interface (ING-003) and a
  similarly isolated geocoding/map-tile interface keep the specific
  air-quality data provider(s) and map vendor swappable — both are open in
  REQUIREMENTS.md §13 (Q2, Q13) and must not be hardcoded into core logic.

### Data stores

REQUIREMENTS.md §10 calls for an operational store, a historical/analytical
store, a cache, a queue/event bus, and object storage. Azure is the given
hosting constraint, but no specific engine, product, or SKU is specified by
REQUIREMENTS.md or the prompt, so none is chosen here — each is expressed by
required role only:

| Role | Holds | Driven by |
|---|---|---|
| Operational store | Users, sessions, preferences, saved locations, alerts, provider configuration, current-observation snapshot | §5 Core Entities, USER-\*, ADMIN-004 |
| Historical/analytical store | Normalized long-retention observations, range/aggregation-optimized | HIST-\*, §5.2, NFR-PERF-004 |
| Cache | Current observations, geo lookups, hot historical queries | NFR-PERF-005 |
| Queue / event bus | Ingestion pipeline stages, aggregation jobs, alert evaluation, notification delivery | ING-\*, NOTIF-001, NFR-SCALE-003 |
| Object storage | Raw provider payloads, bulk datasets, reprocessing artifacts | ING-004 |
| Audit store | Append-only administrative audit trail | SEC-LOG-005, ADMIN-008 |

**Unknown:** whether the operational and historical stores are one engine
or two, and which engine family (relational, document, time-series/
columnar) fits the historical query and aggregation patterns — not
specified. Treat as a dedicated data-store-selection decision informed by
HIST-004 aggregation intervals and NFR-PERF-004 bounding rules.

### Authentication & session model

- **Public User:** unauthenticated GET access to current/historical
  endpoints (WEB-005), subject to API rate limits (API-007).
- **Authenticated User (non-admin):** password required, stored with an
  approved adaptive hash (SEC-AUTH-002); passkey optional as an additional
  or alternative method. Session behavior per SEC-SESS-001–003 (idle +
  absolute expiration, logout invalidation, revocation on security-relevant
  changes).
- **Administrator:** passkey required (phishing-resistant, per human input
  hardening ADMIN-001/SEC-AUTH-003). Administrative auth and session
  handling are isolated from the consumer auth path, consistent with
  API-003 and SEC-AUTHZ-005 (no reliance on hidden navigation/client-side
  checks).
- RBAC roles from ADMIN-002 are enforced server-side on every
  administrative operation (SEC-AUTHZ-001/002/004).
- **Unknown:** specific passkey/WebAuthn library and relying-party
  implementation — not specified. Federated identity (listed as optional in
  USER-002) is not selected for v1 by the human-provided input.

### Deployment topology (Azure + Terraform)

- All infrastructure is provisioned via Terraform *(given)*, enabling the
  logical dev/test/staging/production separation required by SEC-003 as
  distinct Terraform-managed environments.
- Backend services run as independently, horizontally scalable units on
  Azure (NFR-SCALE-001). **Unknown:** the specific Azure compute target
  (e.g., container platform vs. managed app-hosting PaaS) is not specified
  — deferred.
- "Global" scale (given) implies multi-region considerations for latency
  and the 99.9% monthly availability target (NFR-AVAIL-001). **Unknown:**
  region topology, active-active vs. active-passive, and CDN strategy for
  client assets — deferred pending recovery objectives (REL-005, §13 Q25).
- Secrets are held in an approved secret-management system, never in code,
  client bundles, logs, or images (SEC-004); an Azure-native secret store is
  assumed available given the Azure hosting decision, but the specific
  service is not chosen here.
- CI/CD must be automated and reproducible (NFR-MAINT-005); the specific
  pipeline tooling is **unknown** — deferred.

### Security posture (per human-provided emphasis)

- **Primary hardening target — administrative surface:** isolated admin
  API (API-003), mandatory phishing-resistant passkey authentication,
  least-privilege granular RBAC (SEC-AUTHZ-004), full tamper-protected audit
  trail (SEC-LOG-005) with no secrets in audit records.
- **Baseline for end-user data (light for now):** REQUIREMENTS.md §7 items
  phrased as "must" are treated as non-negotiable floor, not optional
  hardening — adaptive password hashing, TLS on all production traffic,
  parameterized data access, contextual output encoding, secure session
  cookie attributes, and the privacy minimums in §7.9. "Light" is
  interpreted as: no incremental investment *beyond* those stated musts
  (e.g., no fraud-detection ML, no advanced anomaly scoring) for this
  phase — not a waiver of them.
- Provider responses are always treated as untrusted input (SEC-INPUT-004),
  and outbound provider calls are restricted to configured destinations to
  limit SSRF exposure (SEC-INPUT-005), independent of the "light" posture
  above — these are ingestion-boundary controls, not user-data controls.

### Assumptions (consolidated)

1. No backend-for-frontend layer; web and mobile share one REST API surface.
2. Backend decomposes into independently deployable Node.js services along
   the boundaries in REQUIREMENTS.md §10, exact granularity TBD.
3. Provider, map-tile, and geocoding vendors sit behind internal adapter
   interfaces and are not assumed to be any specific vendor.
4. An Azure-native secret-management service and CI/CD platform are
   available under the Azure/Terraform decision; the specific services are
   unselected.
5. Federated identity login is out of scope for v1 (human input selects
   password + passkey only).

### Open / deferred decisions

- Specific air-quality data provider(s) — REQUIREMENTS.md §13 Q2
- Specific map/tile provider — §13 Q13
- Default air-quality index standard and cross-standard switching UX — §13 Q5–Q7
- Operational and historical data-store engine(s) — not specified
- Queue/event-bus technology on Azure — not specified
- Azure compute hosting target (containers vs. PaaS) — not specified
- CI/CD pipeline tooling — NFR-MAINT-005 requires it; tool unspecified
- Recovery point/time objectives and region topology — §13 Q25, REL-005/006
- Passkey/WebAuthn library and relying-party setup — not specified
- Observability stack (metrics/tracing/alerting backend) — OBS-\* requires
  capability, tool unspecified

---

## Requirement Traceability

| Architecture component | Requirement groups | Status |
|---|---|---|
| Web client (React) | WEB-\*, MAP-\*, HIST-\*, NFR-A11Y-\*, NFR-I18N-\* | Defined |
| Mobile clients (React Native) | MOB-\*, NOTIF-004, NFR-A11Y-005 | Defined |
| Public API Service | API-001, API-004–010, WEB-005 | Defined |
| Identity & User Service | USER-\*, SEC-AUTH-\*, SEC-SESS-\*, API-002 | Defined |
| Ingestion Service | ING-\*, SEC-INPUT-004/005/006 | Defined |
| Air-Quality Processing Service | AQ-\*, MAP-006, HIST-004/007, REL-001/002, NFR-MAINT-002 | Defined |
| Notification Service | NOTIF-\*, MOB-005 | Defined |
| Administrative Service | ADMIN-\*, API-003, SEC-AUTHZ-004/005 | Defined |
| Audit & Logging | ADMIN-007/008, SEC-LOG-\*, OBS-\* | Defined |
| Operational data store | §5 Core Entities, USER-\*, ADMIN-004 | **TO BE DECIDED** (engine) |
| Historical/analytical store | HIST-\*, §5.2, NFR-PERF-004 | **TO BE DECIDED** (engine) |
| Cache | NFR-PERF-005 | **TO BE DECIDED** (technology) |
| Queue / event bus | ING-\*, NOTIF-001, NFR-SCALE-003 | **TO BE DECIDED** (technology) |
| Object storage | ING-004 | **TO BE DECIDED** (technology) |
| Audit store | SEC-LOG-005, ADMIN-008 | **TO BE DECIDED** (technology) |
| Passkey / password auth | USER-002, ADMIN-001, SEC-AUTH-003 | Policy defined — **TO BE DECIDED** (implementation) |
| Provider & map vendor selection | ING-001, MAP-001 | **TO BE DECIDED** (§13 Q2, Q13) |
| Deployment topology (Azure/Terraform) | NFR-AVAIL-\*, NFR-SCALE-\*, SEC-003 | Partially defined — region topology **TO BE DECIDED** |
| Observability stack | OBS-\* | **TO BE DECIDED** (technology) |
| Disaster recovery / backups | REL-004/005/006 | **TO BE DECIDED** (objectives + tooling) |
| Localization pipeline | NFR-I18N-\* | Client-side requirement defined — externalization tooling **TO BE DECIDED** |

---

*Architecture v0.1 (provisional) — derived from `REQUIREMENTS.md`,
`DESIGN.md`, and explicit human-provided inputs. Update this document as
open questions in REQUIREMENTS.md §13 are resolved and vendor/technology
decisions are made.*
