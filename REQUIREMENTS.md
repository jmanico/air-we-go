# Air Quality Platform Requirements

- **Status:** Threat-modeled baseline; provisional until the release-blocking decisions in §13 are approved
- **Version:** 0.2
- **Threat-model assessment date:** 2026-07-17
- **Approval status:** Draft baseline; no accountable owner or approver has signed off
- **Owner / approvers:** TO BE DECIDED before implementation; approval must include product, security, privacy/legal, air-quality domain, engineering, mobile/client, accessibility, and operations representatives
- **Next review:** Before architecture freeze, and on every NFR-MAINT-007 trigger
- **v0.2 change summary:** Added threat-derived safety, privacy, identity, service, supply-chain, recovery, release, and decision requirements

## 1. Project Overview

### 1.1 Name

**Air We Go**

The final product name remains to be selected.

### 1.2 Purpose

Build a public-facing air-quality platform that collects global air-quality measurements from one or more public data providers, stores current and historical observations, and presents the data through a consumer website and mobile application. Because people may use the information to make health-related decisions, the platform must preserve the authenticity, integrity, provenance, freshness, and availability of published observations without presenting the service as medical or emergency advice.

The platform must support:

* Live air-quality conditions
* Geographic map visualization
* Historical date-range queries
* Trend analysis
* Public and authenticated-user experiences
* Administrative management and monitoring

### 1.3 Primary Users

The system supports the following actors:

1. **Public User**

   * Views current air-quality information
   * Browses the air-quality map
   * Searches for locations
   * Reviews historical air-quality data
   * Does not require an account

2. **Authenticated User**

   * Has all public-user capabilities
   * Saves locations
   * Configures preferences
   * Creates alerts or subscriptions
   * Maintains a user profile
   * Accesses personalized views

3. **Administrator**

   * Manages users and administrative configuration
   * Monitors data ingestion and provider health
   * Reviews system errors and data-quality problems
   * Manages supported air-quality providers
   * Corrects, suppresses, or annotates invalid data
   * Accesses operational dashboards and audit records

4. **External Air-Quality Data Provider**

   * Supplies current and historical air-quality observations through a public API, feed, or dataset

5. **System Operator / Site Reliability Engineer**

   * Operates production infrastructure and responds to availability, security, and data-integrity incidents
   * Uses separately authorized operational access rather than application-administrator privileges
   * Does not receive Azure, CI/CD, backup, key, or Terraform authority through an application role

6. **Internal Service Workload**

   * Produces, processes, or consumes data through authenticated service-to-service interfaces
   * Receives only the data-store, queue, and API permissions required for its function

7. **External Processor or Delivery Provider**

   * Provides map/geocoding, notification delivery, hosting, or observability capabilities
   * Receives only the minimum data required under an approved contract and security review

---

## 2. Scope

### 2.1 In Scope

The initial platform includes:

* Responsive consumer website
* Consumer mobile application
* Global air-quality data ingestion
* Current air-quality map
* Historical data browsing
* Location search
* Air-quality trend visualization
* User authentication
* Saved locations
* Administrative interface
* Data-provider monitoring
* Data normalization and storage
* Public and authenticated APIs
* Role-based authorization
* Audit logging
* Operational monitoring

### 2.2 Out of Scope for Initial Release

Unless explicitly added later, the initial release does not require:

* Operation of physical air-quality sensors
* Medical diagnosis or personalized medical advice
* Government emergency-notification integration
* User-submitted sensor measurements
* Predictive pollution forecasting
* Indoor air-quality monitoring
* Commercial advertising
* Paid subscriptions
* Social-network functionality
* Carbon-footprint calculations

---

## 3. Product Goals

The platform should:

1. Make global air-quality information understandable to non-technical users.
2. Present current and historical air-quality data geographically.
3. Preserve imported observations for long-term analysis.
4. Clearly communicate data source, measurement time, and uncertainty.
5. Remain functional when an upstream data provider is temporarily unavailable.
6. Support future expansion to additional providers, pollutants, regions, and forecasting models.
7. Protect user accounts, location preferences, and administrative functions.
8. Provide traceable provenance for imported measurements.
9. Prevent unverified, stale, replayed, or improperly transformed observations from being presented as current or used to trigger alerts.
10. Minimize collection, disclosure, and linkability of precise location, saved-location, search, device, and notification data.
11. Contain provider, third-party, tenant-equivalent user, worker, and regional failures so one compromised dependency cannot silently corrupt the trusted dataset or control plane.

---

## 4. Functional Requirements

### 4.1 Public Website

#### WEB-001: Responsive Interface

Before implementation, the project must publish a browser support matrix with
minimum versions, update cadence, end-of-support rules, and the behavior shown
to an unsupported client. The website must support every browser/version in
that matrix and must not silently lose security, privacy, or accessibility
controls on an unsupported client.

#### WEB-002: Landing Page

The landing page must provide:

* Current air-quality summary
* Location search
* Access to the global map
* Explanation of the displayed air-quality index
* Data freshness indicator
* Link to historical data
* Link to methodology and data sources

#### WEB-003: Location Search

Users must be able to search by:

* City
* Region or state
* Country
* Postal code, where supported
* Geographic coordinates
* Known monitoring station

Search results must disambiguate locations with identical or similar names.

#### WEB-004: Location Detail View

The system must provide a location detail page containing:

* Current air-quality index
* Air-quality category
* Primary pollutant
* Individual pollutant measurements
* Measurement units
* Observation timestamp
* Source provider
* Monitoring-station information, where available
* Historical chart
* Data-quality or freshness warnings
* General health guidance associated with the selected index

#### WEB-005: Public Access

Current and historical air-quality information must be available without requiring account creation, subject to rate limits and provider licensing restrictions.

#### WEB-006: Observation Trust Status

Every current observation, aggregate, trend, and health-guidance display must identify its source or complete source set, applicable standard and rule version, observation or covered time range, freshness state, and data-quality state. Platform-derived values must be distinguishable from provider-supplied values. Quarantined, unverified, unavailable, or stale data must never be labeled or styled as current.

#### WEB-007: Browser Location Privacy

Browser geolocation must be requested just in time for a user-invoked nearby lookup, remain optional, and follow the same minimization, disclosure, logging, retention, and saved-location rules as MOB-003 and SEC-PRIV-006. Denial or revocation must not prevent manual search.

---

### 4.2 Mobile Application

#### MOB-001: Supported Platforms

The product must support:

* iOS
* Android

Before implementation, the project must publish minimum iOS and Android
versions, security-update and end-of-support rules, upgrade cadence, and the
behavior shown to an unsupported device. The initial implementation will use
React Native, subject to the version, native-module, signing, update, and
security decision gates in §13 and `ARCHITECTURE.md` §9.

#### MOB-002: Core Feature Parity

The mobile application must support the following website capabilities:

* Current air-quality lookup
* Interactive map
* Location search
* Location detail views
* Historical charts
* Saved locations for authenticated users
* User profile and preferences

#### MOB-003: Device Location

With explicit operating-system permission, the mobile application may use the device location to display nearby air-quality observations.

The application must remain usable when location permission is denied.

The initial release must request foreground location only when the user invokes a location-dependent feature. It must not request continuous or background location, and must not retain or associate a precise device location with an account unless the user explicitly saves that location after being told what will be stored.

#### MOB-004: Cached Data

The mobile application must provide a bounded local cache of recently viewed public observations for temporary offline viewing.

Cached data must display its observation timestamp and indicate that it may no longer be current.

Only publication-eligible Public observation payloads and the minimum coarse
display identifier may be cached. Exact coordinates, query history, account
association, and user-defined labels must not be retained unless a separately
approved feature requires them. The cache must be device-local,
non-synchronized, size- and TTL-bounded, user-clearable, and purged as
required by SEC-MOB-006. Account data, authentication material,
administrative data, and precise device-location history must not be placed
in a general-purpose or shared client cache.

#### MOB-005: Notifications

Authenticated users must be able to enable air-quality alerts for saved locations through at least one launch-approved delivery channel.

The authenticated in-application detail reached from a notification must identify:

* Location
* Air-quality level
* Trigger condition
* Observation time
* Link to additional details

Lock-screen, email-subject, and Web Push previews must follow NOTIF-008 and SEC-PRIV-010; they must not disclose a precise location or user-defined label by default.

---

### 4.3 Air-Quality Map

#### MAP-001: Interactive Map

The system must display current air-quality measurements on an interactive geographic map.

#### MAP-002: Map Navigation

Users must be able to:

* Zoom
* Pan
* Search for a location
* Select a station or geographic area
* Recenter the map
* View a legend

#### MAP-003: Map Markers

Map markers or regions must visually represent the applicable air-quality category.

The map must not rely only on color. It must also use labels, symbols, patterns, or textual descriptions for accessibility.

#### MAP-004: Marker Details

Selecting a map marker must display:

* Station or location name
* Air-quality index
* Air-quality category
* Main pollutant
* Observation time
* Data source
* Link to the full location detail view

#### MAP-005: Marker Clustering

The map must cluster or aggregate markers at lower zoom levels to avoid excessive visual density and client-side rendering load.

#### MAP-006: Geographic Aggregation

Where multiple stations exist in the same area, the platform must define and document how an area-level value is calculated.

The system must distinguish:

* Raw station measurements
* Provider-calculated regional values
* Platform-calculated aggregate values

#### MAP-007: Historical Map

Users must be able to select a historical date or date range and view available air-quality observations for that period.

The interface must indicate when historical data is incomplete or unavailable.

---

### 4.4 Historical Data and Trends

#### HIST-001: Date-Range Selection

Users must be able to select a start date and end date.

#### HIST-002: Valid Date Ranges

The system must reject:

* End dates before start dates
* Dates in the future, unless forecast data is later supported
* Date ranges exceeding configured service limits
* Unsupported dates earlier than retained data

#### HIST-003: Historical Charts

The system must display historical measurements using line charts, bar charts, heat maps, or other appropriate visualizations.

#### HIST-004: Time Aggregation

Users should be able to view data using supported aggregation intervals such as:

* Hourly
* Daily
* Weekly
* Monthly

The available intervals may depend on the selected date range and provider resolution.

#### HIST-005: Pollutant Selection

Users must be able to view historical measurements for supported pollutants, including where available:

* PM2.5
* PM10
* Ozone
* Nitrogen dioxide
* Sulfur dioxide
* Carbon monoxide

#### HIST-006: Data Gaps

Historical charts must visibly distinguish missing observations from valid zero values.

#### HIST-007: Trend Summaries

The platform should calculate understandable summaries such as:

* Average air-quality index
* Highest observed value
* Lowest observed value
* Number of unhealthy periods
* Change compared with the previous equivalent period
* Most significant pollutant

#### HIST-008: Time Zones

Historical data must be stored using UTC timestamps.

The user interface must clearly identify whether dates and times are shown in:

* The user’s local time zone
* The monitored location’s time zone
* UTC

---

### 4.5 Data Ingestion

#### ING-001: Provider Integration

The backend must retrieve air-quality data from one or more approved public data providers.

#### ING-002: Scheduled Collection

The backend must collect data at configurable intervals appropriate to each provider’s update frequency and rate limits.

#### ING-003: Provider Adapter Interface

Each provider integration must use a defined adapter interface so providers can be added, replaced, or disabled without rewriting the core platform.

#### ING-004: Raw Data Preservation

The system must preserve the raw provider response or an immutable equivalent representation, subject to provider licensing and an approved retention schedule, for troubleshooting, reconciliation, and provenance. The stored record must include a retrieval timestamp, source identity, parser/adapter version, and cryptographic digest bound to an independently protected audit/provenance record and immutable/versioned storage so later modification or substitution is detectable. A digest or unsigned checksum detects corruption but does not by itself authenticate the provider; provider identity and any available signature are verified separately under ING-015.

#### ING-005: Normalized Observations

Provider data must be converted into a normalized internal representation.

Every normalized record that can advance beyond quarantine must include:

* Provider identifier
* Provider-scoped canonical source identity and source version
* Observation timestamp and retrieval timestamp
* Station or documented geographic scope
* Pollutant, index, or other measurement type
* Finite source value and source unit
* Averaging period where applicable
* Applicable air-quality standard or an explicit not-applicable state
* Data-quality and freshness states
* Source attribution
* Raw-record digest and reference

Provider observation identifiers, latitude/longitude, country,
administrative region, locality, station display metadata, provider display
labels, and derived AQI values are optional descriptive or derived fields.
Absence or ambiguity of a publication-required field forces quarantine; an
undocumented default must not repair it.

#### ING-006: Idempotent Processing

Reprocessing the same provider record and content must produce the same canonical observation identity and must not create an unintended duplicate or repeat side effects. A corrected or changed provider record must create a traceable new version rather than silently overwriting the prior source or publication decision.

#### ING-007: Retry Handling

Transient ingestion failures must use exponential backoff with jitter and an
approved maximum attempt count, total elapsed-time/retry budget, circuit
breaker, and terminal quarantine/dead-letter outcome. Exhaustion must alert
the responsible owner without causing an unbounded retry or hiding the last
eligible data state.

#### ING-008: Failure Isolation

Failure of one provider or geographic feed must not stop ingestion from other providers.

#### ING-009: Rate-Limit Compliance

The ingestion service must enforce provider-specific rate limits and usage restrictions.

#### ING-010: Data Validation

Imported data must be validated for:

* Required fields
* Valid coordinates
* Valid timestamps
* Supported units
* Numeric measurement ranges
* Duplicate records
* Stale observations
* Malformed provider responses

#### ING-011: Invalid Records

Invalid records must be quarantined or rejected without preventing valid records in the same batch from being processed.

#### ING-012: Import Audit Trail

Each ingestion run must record:

* Provider
* Start and completion time
* Records retrieved
* Records accepted
* Records rejected
* Records updated
* Error count
* Retry count
* Processing status

#### ING-013: Backfill

Administrators must be able to initiate a historical-data backfill only after
previewing provider, date/record scope, expected cost, and publication impact.
Backfills must enforce record/date/concurrency/runtime limits, provider rate
policy, cancellation, idempotency, current validation/publication rules,
durable audit, and capacity isolation from interactive/current-data work.
The approved ADMIN-010 impact matrix determines when independent approval is
required.

#### ING-014: Provider Outage Behavior

When a provider is unavailable, the platform must continue serving the most recently stored data and clearly indicate its age.

Configured freshness limits still apply during an outage. Data older than the permitted display limit must be shown as unavailable rather than current and must not trigger new air-quality or recovery notifications.

#### ING-015: Provider Endpoint and Source Authenticity

Provider endpoints, redirect policy, credentials, expected transport security, and any available signature or checksum verification must be held in reviewed configuration, not accepted from provider payloads or request parameters. Retrieval must authenticate the configured destination using approved TLS validation, reject an unexpected destination or downgrade, and verify provider-supplied signatures or checksums when the provider supports them.

#### ING-016: Quarantine and Publication Gate

New or changed provider data must enter a quarantine/staging boundary. A
versioned decision must independently record whether the record is (1)
provenance-valid normalized data, (2) eligible for historical public display,
(3) eligible for current display, (4) eligible for aggregation or guidance,
and (5) eligible for alert evaluation. Schema, semantic range, timestamp,
unit, provenance, licensing, freshness, and configured anomaly rules govern
each transition. A stale but otherwise valid backfill may be historical-
public eligible without becoming current, guidance, or alert eligible.
Validation ambiguity, missing provenance, or unavailable policy must fail
closed for the affected transition while preserving the last known valid
dataset under its true freshness state.

#### ING-017: Replay, Ordering, and Replacement Safety

Ingestion must tolerate duplicate and out-of-order delivery without allowing an older, replayed, lower-trust, or differently scoped record to overwrite a newer trusted record silently. Publication decisions must be idempotent, preserve every source version needed for reconciliation, and record why a record was accepted, superseded, quarantined, or rejected.

#### ING-018: Ingestion Resource Isolation

Provider polling, bulk imports, test retrievals, decompression, parsing, retries, and backfills must enforce independent limits for response size, expanded size, record count, execution time, concurrency, retry budget, and queue depth. Exhaustion or malicious input from one provider must not consume the capacity reserved for other providers or interactive requests.

---

### 4.6 Air-Quality Normalization

#### AQ-001: Index Standards

The platform must identify the standard used for every displayed air-quality index.

Examples may include:

* United States AQI
* European Air Quality Index
* National or provider-specific indexes

#### AQ-002: Index Conversion

The system must not silently convert one air-quality standard into another.

Any platform-calculated conversion must:

* Use a documented algorithm
* Record the original measurement
* Identify the derived value
* Identify the conversion standard and version

#### AQ-003: Units

The system must preserve source measurement units and normalize them only through documented conversion rules.

#### AQ-004: Category Labels

Air-quality categories must be associated with configurable labels and thresholds.

#### AQ-005: Health Guidance

Any health guidance must be tied to a documented source and air-quality standard.

The platform must state that the information is educational and is not medical advice.

#### AQ-006: Data Provenance

Every observation exposed through the website, mobile application, or API must be traceable to its provider and ingestion record.

#### AQ-007: Conflicting Observations

When multiple providers report different values for the same area and time, the platform must:

* Preserve each source observation
* Avoid silently overwriting conflicting values
* Apply a documented source-selection or aggregation policy
* Expose source information to users

#### AQ-008: Rule and Guidance Change Control

Index calculations, unit conversions, freshness thresholds, category boundaries, aggregation/source-selection rules, anomaly policy, and health guidance must be immutable and versioned after approval. Each version must record its authoritative source, effective period, review/expiry date where applicable, reviewer and approver, test evidence, and rollback path. A production change must be peer reviewed, audited, tested against authoritative boundary and regression vectors, and must not rewrite the original provider measurement.

#### AQ-009: Safe Publication and Alert Eligibility

Only observations with the corresponding explicit ING-016 eligibility state
under the active versioned policy may be used for historical display, current
display, aggregation/guidance, or alert evaluation. Eligibility in one
context never implies eligibility in another. Values from different
standards, units, averaging periods, locations, or timestamps must not be
combined unless a documented and tested rule explicitly permits it.

#### AQ-010: Correction, Suppression, and Cache Invalidation

Corrections, suppressions, and annotations must preserve the original record and record the actor, reason, evidence, approval when required, effective time, and affected derived data. The change must invalidate or version all affected caches, aggregates, API responses, and pending notifications; restoration must be equally traceable.

---

### 4.7 User Accounts

#### USER-001: Registration

Users must be able to create an account using an approved authentication mechanism.

#### USER-002: Authentication

For the initial release, every non-administrative account must enroll email and password authentication and may register passkeys as an additional factor or an alternative login ceremony. A consumer passkey does not remove the v1 password/recovery credential. Administrative identities follow ADMIN-001 and must not inherit a password-only or consumer-account recovery path. Federated identity is not selected for the initial release.

#### USER-003: Email Verification

The platform must verify control of an email address before using it for notification delivery, credential recovery, or another feature that depends on email ownership. Lack of email verification must not prevent a recently authenticated owner from deleting an unverified account; sensitive export or recovery requires the approved recent-authentication and ownership proof. Changing the primary email must require a recent authenticated session, verify the new address, notify the previous address, and revoke outstanding verification/recovery artifacts affected by the change.

#### USER-004: Password Recovery

When password authentication is supported, users must be able to securely reset a forgotten password.

Recovery must provide assurance appropriate to the account, must not disclose whether an account exists, and must not be usable to bypass mandatory administrative passkey authentication. Successful recovery must atomically revoke prior recovery artifacts and every existing session/renewable credential; any newly established session must use a fresh identifier.

#### USER-005: Saved Locations

Authenticated users must be able to:

* Save a location
* Rename a saved location
* Remove a saved location
* Reorder saved locations
* Select a default location

#### USER-006: Preferences

Authenticated users must be able to configure:

* Preferred air-quality index
* Preferred units
* Time zone
* Language
* Notification settings
* Accessibility preferences

#### USER-007: Alert Configuration

Users must be able to create alerts based on the launch-approved subset of:

* Location
* Air-quality category
* Air-quality index threshold
* Specific pollutant threshold
* Time window
* Notification channel

#### USER-008: Account Deletion

Users must be able to request deletion of their account and associated personal data, subject to legal and security retention requirements.

The request must be authenticated and protected against cross-site request forgery and accidental activation. Deletion must propagate through active stores, caches, queues, analytics, notification processors, and contracted processors within a documented time limit; retained backup or audit data must be minimized, access-restricted, and removed or rendered non-identifying when its approved retention expires.

#### USER-009: Session Management

Users must be able to review and revoke active sessions. The platform must notify them of new authenticator enrollment, password or primary-email changes, recovery completion, and material session or security-setting changes without including secrets in the notification.

#### USER-010: Sensitive Account Changes

Changing a password, primary email, passkey, recovery method, or account-deletion state must require recent authentication appropriate to the authenticator being changed. Session identifiers must rotate after authentication or privilege changes. Password change, recovery completion, primary-email change, authenticator reset/removal, or role/privilege change must invalidate all other sessions and renewable credentials; new authenticator enrollment must notify the user and provide immediate revoke-all capability.

---

### 4.8 Administration

#### ADMIN-001: Administrative Authentication

Administrative access must require a phishing-resistant WebAuthn/FIDO2 passkey with user verification. No password-only, email-link, SMS, knowledge-based, or consumer self-service recovery path may grant administrative access.

#### ADMIN-002: Role-Based Access Control

Administrative functions must be protected by role-based authorization.

At minimum, the authorization policy must distinguish the following roles (one person may hold more than one only when an approved separation-of-duties analysis permits it):

* Support Administrator
* Data Administrator
* Security Administrator
* Application System Administrator
* Read-Only Auditor

Application roles do not grant Azure, CI/CD, Terraform, signing, key/secret, backup/restore, or source-control authority. Those operational privileges use separately approved workforce identities and time-bounded workflows.

#### ADMIN-003: User Management

Authorized administrators must be able to:

* Search users
* View account status
* Disable or re-enable accounts
* Review account-related audit events
* Revoke sessions
* Modify administrative roles, when authorized

#### ADMIN-004: Provider Management

Authorized administrators must be able to:

* Add provider configuration
* Enable or disable providers
* Configure ingestion intervals
* Configure provider credentials
* Review provider status
* Initiate test retrievals
* Initiate backfills

#### ADMIN-005: Data-Quality Management

Authorized administrators must be able to:

* Review rejected records
* Review anomalous observations
* Mark observations as invalid
* Suppress invalid data from public presentation
* Add administrative annotations
* Reprocess quarantined records

#### ADMIN-006: Operational Dashboard

The administrative interface must display:

* Provider health
* Last successful ingestion time
* Ingestion lag
* Queue depth
* Error rates
* Record-processing rates
* API status
* Database status
* Cache status
* Notification-delivery status

#### ADMIN-007: Configuration Changes

Security-sensitive and data-processing configuration changes must be audited.

#### ADMIN-008: Administrative Audit Log

Administrative actions must record:

* Actor
* Action
* Target resource
* Timestamp
* Outcome
* Relevant before-and-after values
* Request or correlation identifier

Sensitive secrets must not appear in audit records.

#### ADMIN-009: Administrative Identity Lifecycle

Administrative identities must be individually assigned, invite-only, separate from consumer self-registration, and must never be shared. Enrollment, role assignment, authenticator replacement, recovery, suspension, and termination must use a documented high-assurance workflow, notify independent security personnel, and produce tamper-evident audit events. At least two authorized people must be required to recover an administrative identity or establish the first production administrator.

#### ADMIN-010: Sensitive-Action Reauthentication and Approval

Administrative actions that can grant privilege, change authentication or
recovery policy, alter provider destinations or credentials, change
publication/AQI/health-guidance rules, perform suppression or correction,
export personal data, or disable audit/security controls must require recent
phishing-resistant reauthentication and a recorded reason. A versioned impact
matrix must require a second authorized approver for privilege, authenticator,
credential, provider-destination, publication/guidance-rule, security-control,
bulk, or safety-significant corrections at any geographic scope. A requester
must never approve their own action.

#### ADMIN-011: Break-Glass Access

Any emergency administrative path must be disabled or tightly restricted during normal operation, use a separately protected identity, require a declared incident and independent approval when feasible, be time-limited and least-privileged, alert security personnel immediately, and receive retrospective review. It must not bypass immutable audit collection.

#### ADMIN-012: Administrative Data Minimization

Support, data, security, system, and audit roles must see only the personal data and secret metadata required for their task. Secret values must be write-only after entry, personal fields must be masked by default, audit-log access must itself be audited, and no administrator may approve their own role increase or high-impact data change.

---

### 4.9 Application Programming Interfaces

#### API-001: Public API

The platform must expose the versioned, documented API required by its web and mobile clients for current and historical public air-quality data. Whether third-party developer access is offered, and under what credentials and license controls, remains a separate §13 decision.

#### API-002: Authenticated API

Authenticated users must be able to access only their own personalized resources, including:

* Saved locations
* Alerts
* Preferences
* Notification history

#### API-003: Administrative API

Administrative APIs must be isolated from public APIs and protected by explicit authorization checks.

#### API-004: API Versioning

Externally consumed APIs must be versioned.

#### API-005: Pagination

Endpoints returning collections must support bounded pagination.

#### API-006: Filtering

Historical-data endpoints must support the approved subset of these filters,
and every supported combination must follow API-005 and API-013 bounds:

* Geographic area
* Station
* Pollutant
* Provider
* Start timestamp
* End timestamp
* Aggregation interval

#### API-007: Rate Limiting

Public and authenticated APIs must enforce the explicit, independently
configurable principal-, network/device-, credential-, operation-cost-, and
global limits defined by API-013 and the approved numeric limit profile.

#### API-008: Error Format

API errors must use a consistent structured error format containing:

* Error code
* Human-readable message
* Correlation identifier
* Field-level validation errors, when applicable

#### API-009: Data Freshness Metadata

Current-data responses must include:

* Observation time
* Retrieval time
* Source
* Freshness or staleness status

#### API-010: API Documentation

Public APIs must be documented using a machine-readable specification such as OpenAPI.

#### API-011: API Inventory and Contract Enforcement

Every deployed endpoint, method, request and response schema, authentication rule, authorization rule, cache policy, and deprecation state must exist in the reviewed API inventory. Unknown methods and unsupported content types must be rejected. Implementation and specification drift must fail an automated release check.

#### API-012: Cache Safety

Shared caches may store only explicitly public responses. Cache keys must include every request property that changes authorization or representation, including the air-quality standard and rule/data version. Authenticated, administrative, recovery, and personal-data responses must use `private, no-store` semantics and must never be served from a shared cache. Correction and suppression events must invalidate affected cached public data.

#### API-013: Abuse and Cost Controls

Rate, concurrency, result-size, geographic-area, date-range, alert-count, saved-location, registration, recovery, and notification limits must be explicit, independently configurable, and enforced at the earliest practical boundary. Controls must combine appropriate network, account, device, credential, and global budgets without making a single spoofable identifier a permanent denial-of-service mechanism.

---

### 4.10 Notifications

#### NOTIF-001: Threshold Evaluation

The backend must evaluate alert conditions only after a publication-eligible observation is committed under AQ-009. Retrieval, quarantine, validation, or normalization alone must not trigger evaluation.

#### NOTIF-002: Duplicate Suppression

The system must prevent repeated notifications for the same threshold event within a configurable suppression period.

#### NOTIF-003: Recovery Notifications

Users must be able to choose whether to receive a notification when air quality returns below an alert threshold, subject to NOTIF-007 ordering and freshness rules.

#### NOTIF-004: Delivery Channels

The initial release must support at least one approved and verified delivery channel selected from:

* Mobile push
* Email
* Web notifications

#### NOTIF-005: Notification Preferences

Users must be able to opt in or opt out by channel and alert.

#### NOTIF-006: Delivery Logging

The system must record notification status without storing unnecessary sensitive content.

#### NOTIF-007: Trusted and Ordered Notification Inputs

Notifications may be created only from a published observation eligible under AQ-009. Each event must carry the observation identity, source, rule version, evaluation time, and idempotency key. Delayed or out-of-order events must not create a false threshold crossing or recovery, and delivery retries must not duplicate an already accepted notification.

#### NOTIF-008: Notification Privacy

Notification providers and lock-screen payloads must receive the minimum content necessary. Precise coordinates, account identifiers, authentication material, and user-defined location names must not appear in a push payload. The default preview must use a non-sensitive location label and direct the user to an authenticated application view; users may explicitly choose a more descriptive preview after being warned about lock-screen disclosure.

#### NOTIF-009: Notification Abuse Controls

Notifications may be sent only to a verified destination with an active opt-in. Per-user, per-destination, and global fan-out/retry budgets, circuit breakers, unsubscribe handling, and provider-failure isolation must prevent ingestion replay, rule changes, or a delivery-provider outage from creating a notification storm.

#### NOTIF-010: User-Visible Delivery State

Users must be able to view the observation time, evaluation time,
provider-acceptance time when available, current delivery state, and any
failure/delay state. “Accepted by provider” must not be represented as device
delivery. Alerts must state that delivery is best-effort and is not an
emergency-notification service.

---

## 5. Data Requirements

### 5.1 Core Entities

The initial logical data model must include the following records. They need
not be separate physical tables, but their security state and lifecycle must
remain independently enforceable and auditable:

* User
* UserSession
* UserAuthenticator
* RecoveryOrVerificationArtifact
* UserPreference
* SavedLocation
* Alert
* Notification
* NotificationDestination
* DeliveryAttempt
* Provider
* ProviderConfiguration
* ProviderLicensePolicy
* IngestionRun
* RawProviderRecord
* MonitoringStation
* GeographicLocation
* AirQualityObservation
* PollutantMeasurement
* AirQualityStandard
* RuleVersion
* PublicationDecision
* DataQualityFlag
* AdministrativeAnnotation
* AdministrativeRoleBinding
* AdministrativeApproval
* DataSubjectRequestOrDeletionWorkflow
* WorkloadIdentityAuthorizationBinding
* AuditEvent

### 5.2 Observation Identity

Every observation must have a deterministic provider-scoped canonical identity. It must include the provider and a stable provider record identifier when one is supplied. When no reliable provider record identifier exists, the documented identity rule must include enough of the following fields to be unique and stable for that provider feed:

* Provider
* Provider record identifier
* Station identifier
* Observation timestamp
* Pollutant
* Measurement type

The fallback identity must include provider plus station/location, observation timestamp, pollutant/measurement type, and any averaging period or standard needed to distinguish records. Cross-provider records must never collide or overwrite one another. Corrections, provider revisions, normalization versions, and publication decisions must be separate versioned records linked to the canonical source identity.

### 5.3 Location Model

A location should support:

* Latitude and longitude
* Country code
* Region
* City or locality
* Postal code, where available
* Provider location identifier
* Time zone
* Human-readable display name

### 5.4 Retention

Before production, an approved retention schedule must define the owner, purpose, minimum and maximum period, deletion/anonymization mechanism, legal-hold exception, backup-expiry behavior, and evidence of deletion for:

* Raw provider responses
* Normalized observations
* Ingestion logs
* Application logs
* Audit logs
* Notification records
* Deleted-user recovery periods

Normalized historical air-quality observations may be retained long-term only when the approved purpose, provider license, and applicable law allow it. Changing a retention rule must be reviewed, audited, and applied to already-held data where legally and technically required.

### 5.5 Data Licensing

The system must record provider licensing and attribution requirements.

Data must not be retained, transformed, redistributed, or commercially used in a manner prohibited by the source provider.

Licensing and attribution policy must be evaluated before publication and on every provider-policy change. Failure to determine the applicable license must fail closed for new ingestion or redistribution without deleting the last known lawful dataset.

### 5.6 Data Inventory and Classification

#### DATA-001: Inventory, Purpose, and Ownership

Every persisted, cached, queued, logged, backed-up, exported, or third-party-disclosed data field must have a documented owner, purpose, classification, retention rule, authorized consumers, and source-to-deletion lineage. The inventory must include derived data and metadata, not only database columns.

#### DATA-002: Minimum Classifications

Public observations approved for publication are **Public**. Provider configuration without secrets, internal health/quality state, and operational metadata are at least **Internal**. Unclassified raw/quarantined provider payloads are at least **Confidential** until content inspection/minimization permits a lower class. Email addresses, account and device identifiers, coarse locations, alert rules, notification destinations/history, IP addresses tied to activity, administrative annotations, and security/audit events are at least **Confidential**. Exact coordinate searches, exact saved/default or user-defined locations, precise device-location history, password hashes, authenticators, sessions, reset/verification artifacts, provider/cloud/CI credentials, cryptographic keys, and infrastructure state containing secrets are **Restricted**.

#### DATA-003: Encryption and Key Management

Confidential and Restricted data must use approved encryption in transit across external and internal trust boundaries and at rest in databases, caches, queues, object stores, logs, backups, export artifacts, and infrastructure state. Key access must be environment-scoped, least-privilege, rotatable, access-audited, and separated from the protected data. Encryption must not be treated as a substitute for authorization or minimization.

#### DATA-004: Workload and Store Access

Each service, worker, administrative tool, and delivery pipeline must use a distinct workload identity and the minimum data-store, queue, cache, object-store, secret-store, and key permissions required for its function. Shared production credentials and implicit trust based only on network location are prohibited.

#### DATA-005: Deletion and Derived-Data Propagation

Deletion, correction, suppression, consent withdrawal, and retention expiry must propagate to derived indexes, caches, queues, search indexes, notification jobs, analytics, export artifacts, and contracted processors. The system must record completion or a narrowly defined exception without retaining the deleted content in the evidence record.

---

## 6. Non-Functional Requirements

### 6.1 Availability

#### NFR-AVAIL-001

The public website and API must target at least 99.9% monthly availability. Before production, the measurement point, qualifying requests, bounded planned-maintenance exclusion, dependency treatment, error budget, reporting period, and response to an exhausted error budget must be approved and testable.

#### NFR-AVAIL-002

Provider failure alone must not make still-retained and lawfully servable
last-valid or historical data unavailable. Freshness, suppression, deletion,
licensing/withdrawal, and retention policy remain authoritative. Previously
stored data remains subject to ING-014 and REL-001: once its approved
threshold is exceeded, it may be retrieved as delayed, stale, or unavailable
but must not be presented as current or trigger a new alert/recovery event.

#### NFR-AVAIL-003

When map tiles fail, manual search and an accessible non-map current-data view
must remain available. When notification delivery fails, data ingestion and
public display must continue, retries must remain bounded, and users must see
the NOTIF-010 delayed/failed state. When an individual provider fails, other
providers continue and retained data follows ING-014 rather than being
relabeled current. Each degraded behavior must be exercised before release.

#### NFR-AVAIL-004

Every external dependency, asynchronous worker, and expensive public operation must have explicit timeouts, concurrency and memory limits, circuit breaking, retry budgets, queue/backpressure limits, and a tested degraded mode. Capacity reserved for authentication, administration, current public data, and security telemetry must not be starved by historical queries, backfills, notification fan-out, or one provider.

#### NFR-AVAIL-005

The system must define storage, queue, cache, and third-party cost budgets and alert before exhaustion. Automatic scaling and retries must have ceilings that prevent an attacker or dependency failure from causing unbounded spend.

---

### 6.2 Performance

#### NFR-PERF-001

Common current-air-quality backend queries must meet a 500-millisecond p95
target, excluding external provider calls, under an approved test profile that
specifies dataset size, query mix, concurrency, cache state, hardware/service
tier, and measurement point.

#### NFR-PERF-002

Public pages must provide meaningful initial content within two seconds at the
approved percentile under a release test profile that specifies client class,
network latency/bandwidth/loss, cache state, geography, and measurement point.

#### NFR-PERF-003

Before map implementation, an approved profile must define station/feature
count, client classes, frame/interaction latency percentile, memory ceiling,
and degraded behavior. Map interactions must meet that profile using bounded
clustering, tiling, aggregation, or server-side geographic queries.

#### NFR-PERF-004

Historical queries must enforce maximum date ranges, result sizes, and aggregation rules to prevent unbounded processing.

The maximum geographic area, spatial resolution, page size, sort/filter complexity, execution time, concurrent operations, and export size must also be bounded and covered by negative and load tests.

#### NFR-PERF-005

The platform may cache frequently requested current observations and
geographic queries only when they are explicitly Public and eligible, using
API-012 keys, freshness, invalidation, size, TTL, and stampede controls.

---

### 6.3 Scalability

#### NFR-SCALE-001

The architecture must support horizontal scaling of:

* Public API services
* Web application services
* Ingestion workers
* Historical-query workers
* Notification workers

#### NFR-SCALE-002

The system must support growth in:

* Providers
* Stations
* Observations
* Users
* Saved locations
* Alerts
* Historical-query volume

#### NFR-SCALE-003

Long-running ingestion and aggregation workloads must not block interactive user requests.

#### NFR-SCALE-004

Workers and consumers must tolerate at-least-once, duplicated, delayed, and out-of-order delivery through idempotency, version checks, bounded retries, and restricted dead-letter handling. A message may not confer authority beyond the authenticated producer's permission.

---

### 6.4 Accessibility

#### NFR-A11Y-001

The website must conform to WCAG 2.2 Level AA, and the mobile applications must conform to the applicable WCAG 2.2 Level AA success criteria plus platform accessibility guidance, for all supported user journeys. Claimed color and component combinations must be verified with automated and manual testing; a design token that fails its intended contrast or non-color communication requirement must not ship even if it appears in `DESIGN.md`.

#### NFR-A11Y-002

Air-quality categories must not be represented by color alone.

#### NFR-A11Y-003

Charts must provide accessible names/descriptions and an equivalent accessible data table; neither is a substitute for the other.

#### NFR-A11Y-004

Map information must have a non-map alternative for users who cannot interact with the map.

#### NFR-A11Y-005

Mobile applications must support platform accessibility features, including screen readers and dynamic text sizing.

---

### 6.5 Localization

#### NFR-I18N-001

User-facing text must be externalized for localization.

#### NFR-I18N-002

The system must support localized:

* Dates
* Times
* Numbers
* Measurement units
* Place names
* Air-quality category descriptions

#### NFR-I18N-003

The platform must preserve the meaning of official air-quality standards when translating category labels and health guidance.

---

### 6.6 Maintainability

#### NFR-MAINT-001

Provider integrations must be isolated behind stable interfaces.

#### NFR-MAINT-002

Business rules for index calculations, categorization, and aggregation must be versioned and covered by automated tests.

#### NFR-MAINT-003

Database changes must use controlled migrations.

#### NFR-MAINT-004

The project must include:

* Unit tests
* Integration tests
* API tests
* Data-ingestion tests
* Authorization tests
* End-to-end tests for critical workflows

#### NFR-MAINT-005

The build and deployment process must be reproducible through automated continuous integration and continuous delivery pipelines.

#### NFR-MAINT-006

Every release must be traceable to reviewed immutable source and a locked dependency graph, produce an SBOM and signed build provenance, and use signed or otherwise integrity-verified artifacts. Builds, dependencies, infrastructure code, mobile packages, and containers must pass the approved security policy before promotion.

#### NFR-MAINT-007

The threat model must be reviewed before architecture freeze, before production, at least annually, and whenever a trust boundary, privileged workflow, data category, provider/processor, authentication method, deployment topology, or public-data calculation changes. An unresolved Critical risk blocks initial production and cannot be accepted for that release. An unresolved High risk requires the named owner, due date, compensating controls, verification plan, near-term expiry, and independent acceptance in SEC-IR-003.

#### NFR-MAINT-008

Protected branches, required independent review for security-sensitive code and infrastructure, ownership rules, deployment approvals, vulnerability-remediation deadlines, rollback procedures, and required CI checks must be defined before implementation can merge production-bound changes. Untrusted pull requests must not receive production secrets or deployment authority. A generated requirement-to-architecture-invariant-to-security-control-to-release-test matrix must fail validation when a new normative requirement lacks a mapping or when a referenced ID no longer exists.

---

## 7. Security Requirements

### 7.1 General Security

#### SEC-001: Secure Development Lifecycle

The project must use a documented secure development lifecycle that includes:

* Threat modeling
* Secure coding standards
* Dependency scanning
* Static analysis
* Secret scanning
* Code review
* Security testing
* Vulnerability remediation
* Production monitoring

#### SEC-002: Secure Defaults

Security-sensitive features must use secure defaults.

#### SEC-003: Environment Separation

Development, test, staging, and production environments must be logically separated.

#### SEC-004: Secret Management

API keys, signing keys, credentials, and other secrets must be stored in an approved secret-management system.

Secrets must not be stored in:

* Source code
* Mobile application packages
* Client-side JavaScript
* Logs
* Container images
* Public configuration files

Secrets and keys must have a documented owner, purpose, permitted consumers, rotation/revocation procedure, expiry where supported, and access audit. Workload identity must be preferred over long-lived credentials.

#### SEC-005: Asset and Trust-Boundary Inventory

Before implementation of a component, its entry points, privileged operations, data flows, external dependencies, workload identities, data stores, trust boundaries, and failure modes must be documented in `ARCHITECTURE.md` and traced to enforceable controls. Undocumented routes, consumers, stores, and external data transfers are prohibited in production.

#### SEC-006: Cryptographic Policy

Cryptography must use maintained platform or approved library implementations and centrally approved algorithms, key sizes, protocols, certificate-validation behavior, and random-number generation. Custom cryptographic protocols, disabled certificate validation, algorithm downgrade, and hard-coded keys are prohibited. The design must support rotation without an outage or loss of protected data.

#### SEC-007: Third-Party Security

Before a provider, map/geocoding service, notification vendor, analytics/observability processor, library, build action, or hosted service receives data or executes in the delivery/runtime path, it must undergo documented security, privacy, availability, licensing, and exit/revocation review proportional to its access. The platform must maintain an owner and incident contact for each dependency.

#### SEC-008: Trusted Time and Ordering

Authentication expiry, WebAuthn challenges, reset/verification artifacts, freshness, message/event expiry, audit ordering, rate windows, notification ordering, retention, and certificate/key validity must use approved synchronized time sources. Duration measurement must use a monotonic clock where available. The system must define maximum tolerated skew, detect backward/forward jumps and unsynchronized nodes, fail safely when time cannot be trusted, and test clock-skew/jump behavior across services and regions.

---

### 7.2 Authentication

#### SEC-AUTH-001

Authentication must use established frameworks and protocols rather than custom cryptographic or session implementations.

#### SEC-AUTH-002

Passwords, when supported, must be stored using an approved adaptive password-hashing algorithm.

#### SEC-AUTH-003

Administrative accounts must require phishing-resistant WebAuthn/FIDO2 authentication with user verification. Administrative enrollment, replacement, recovery, and break-glass procedures must preserve equivalent assurance and must not fall back to a consumer password, email, SMS, or knowledge-based path.

#### SEC-AUTH-004

Authentication responses must not disclose whether an account exists beyond what is operationally necessary.

#### SEC-AUTH-005

Password-reset and email-verification tokens must be:

* Cryptographically random
* Single-use
* Time-limited
* Bound to the intended account and action

Only a digest of a reset or verification token may be retained at rest. Issuing a replacement must invalidate prior tokens for the same action, and consuming a token must be atomic so concurrent reuse fails.

#### SEC-AUTH-006: Authentication Abuse Resistance

Registration, login, passkey ceremony, verification, password reset, recovery, and authenticator-management endpoints must resist credential stuffing, password spraying, enumeration, replay, and automated abuse. Controls must include uniform externally visible outcomes where practical, distributed and risk-aware throttling, breached-password screening, monitoring, bounded token issuance, and a recovery path that does not let an attacker permanently lock out a victim using only a spoofable network identifier.

#### SEC-AUTH-007: Authenticator Lifecycle

Authenticator enrollment, naming, replacement, recovery, and deletion must
require recent appropriate authentication, notify the account owner, create a
security audit event, and revoke sessions and renewable credentials as
required by USER-010. WebAuthn challenges must be unpredictable, single-use,
short-lived, bound to the intended origin, relying-party ID, account,
ceremony, and user-verification policy, and verified entirely by the server.

#### SEC-AUTH-008: Password Policy

Passwords must be at least 15 characters, accept at least 64 characters,
never be silently truncated, and use documented and tested Unicode handling.
They must reject known-compromised values, allow password-manager
paste/autofill, and must not use composition rules, security questions, or
arbitrary periodic rotation. Online guessing controls must be designed and
tested separately from password hashing.

---

### 7.3 Authorization

#### SEC-AUTHZ-001

Every protected API operation must enforce authorization on the server.

#### SEC-AUTHZ-002

The system must deny access by default.

#### SEC-AUTHZ-003

Users must only access and modify their own:

* Profile and email address
* Authenticators and recovery state
* Saved locations
* Alerts
* Preferences
* Notification destinations and history
* Notification settings
* Sessions
* Personal-data exports and deletion requests

#### SEC-AUTHZ-004

Administrative permissions must be granular and based on least privilege.

#### SEC-AUTHZ-005

Administrative APIs must not rely on hidden navigation or client-side controls as an authorization mechanism.

#### SEC-AUTHZ-006: Permission Matrix and Negative Testing

Every protected role-resource-action and field-level permission must appear in an approved authorization matrix. Unmapped operations must be denied. Automated negative tests must cover cross-user identifiers, nested and bulk resources, ownership changes, mass assignment, every administrative role, and attempts to approve or grant one's own privilege.

#### SEC-AUTHZ-007: Workload Authorization

Every service-to-service call, queue publish/consume operation, cache or data-store access, secret/key read, and deployment action must authenticate a distinct workload identity and authorize the specific resource and action. Network location, a shared credential, or possession of a message alone must not confer authority; an unavailable authorization dependency must fail closed for mutations and protected reads.

---

### 7.4 Input and Output Security

#### SEC-INPUT-001

All untrusted input must be validated using positive constraints appropriate to the field.

#### SEC-INPUT-002

Database access must use parameterized queries or safe object-relational mapping APIs.

#### SEC-INPUT-003

User-controlled content must be safely encoded for its output context.

#### SEC-INPUT-004

Provider responses must be treated as untrusted input.

#### SEC-INPUT-005

Outbound requests to provider, map, geocoding, notification, webhook, and other remote endpoints must allowlist scheme, normalized host, port, path where practical, and resolved address. Application and network egress policy must block loopback, private, link-local, multicast, reserved, and cloud-metadata destinations; revalidate DNS and every redirect; reject userinfo and ambiguous URL forms; cap redirects, response bytes, expanded bytes, and time; and never forward credentials to a changed origin. Administrative test requests and preview features must use the same policy.

#### SEC-INPUT-006

File, archive, or bulk-data imports must enforce:

* File-size limits
* Decompression limits
* Content-type validation
* Processing timeouts
* Record-count limits

Archive processing must reject traversal, symlink/hard-link escape, overlapping entries, and unsupported nested formats. Each job must run with bounded CPU, memory, storage, network egress, and concurrency.

#### SEC-INPUT-007: Untrusted Content and Destinations

Provider names, station/location names, source attribution, administrative annotations, health guidance, URLs, notification content, log fields, filenames, and exported cells must be treated as untrusted. The system must use destination-specific encoding, validate URL schemes and destinations, neutralize log/CSV formula injection, and prohibit executable HTML or script content unless a separately reviewed sanitizer and isolation boundary is required.

#### SEC-INPUT-008: Parser and Schema Safety

Boundary schemas must reject type confusion, unsafe polymorphism, prototype-pollution keys, numeric overflow/non-finite values, invalid Unicode/control characters, and unknown security-relevant fields. Parsers must be configured with resource limits and must not instantiate classes, execute code, or resolve external entities from untrusted data.

---

### 7.5 API Security

#### SEC-API-001

All production API traffic must use TLS.

#### SEC-API-002

Cross-origin resource sharing must use an explicit allowlist.

#### SEC-API-003

State-changing browser requests must reject `GET`, require a framework-supported unpredictable session-bound anti-CSRF token, validate the configured Origin, and use Fetch Metadata where supported when cookie-based authentication is used. `SameSite` cookies are defense in depth and must not be the sole control.

#### SEC-API-004

API endpoints must enforce:

* Authentication where required
* Authorization
* Rate limits
* Request-size limits
* Timeouts
* Pagination limits
* Schema validation
* Response-schema and cache-policy validation
* Route-specific concurrency and cost budgets

#### SEC-API-005

Error responses must not expose:

* Stack traces
* Credentials
* Tokens
* Internal network details
* Database queries
* Sensitive configuration

#### SEC-API-006: HTTP Method, Content, and Cache Controls

Endpoints must accept only documented HTTP methods and media types, enforce safe/idempotent semantics, set appropriate security headers, and return explicit cache directives. State-changing actions must not use `GET`. Authenticated, administrative, recovery, and personal-data responses must be non-cacheable by shared intermediaries. Public cache keys and invalidation must prevent representation mix-ups and cache poisoning.

#### SEC-API-007: Administrative Boundary

The administrative API and client must use a distinct origin or equivalent independently enforceable gateway and authentication audience, must reject consumer sessions/tokens, and must not be reachable by falling through the public routing policy. Network/access policy is defense in depth and never replaces operation-level authorization.

#### SEC-API-008: Inventory and Security Regression

The reviewed API specification is the allowlist for deployed routes. CI and runtime inventory checks must detect undocumented endpoints, debug/diagnostic routes, schema drift, missing authentication or authorization, and deprecated versions that remain reachable. Production debug consoles and default sample routes are prohibited.

#### SEC-API-009: Webhook and Callback Trust

Inbound delivery-status, provider, or other webhooks/callbacks must authenticate the sender using an approved mechanism, bind the intended environment/account and callback type, validate a strict schema, enforce timestamp/expiry and replay protection, use idempotent processing, and apply request/rate limits. Source IP alone is not authentication. A callback must not change notification, publication, account, or administrative state beyond the sender's explicitly authorized scope.

---

### 7.6 Session Security

#### SEC-SESS-001

Session identifiers must be unpredictable and protected in transit.

#### SEC-SESS-002

Browser session cookies must use appropriate:

* Secure
* HttpOnly
* SameSite

attributes.

#### SEC-SESS-003

Sessions must support:

* Idle expiration
* Absolute expiration
* Logout invalidation
* Revocation following password or security-sensitive account changes

#### SEC-SESS-004: Rotation, Binding, and Transport

Session identifiers must rotate after authentication, recovery, and privilege changes and must never appear in a URL, referrer, log, analytics event, or notification. Browser cookies must use the narrowest practical domain and path. The chosen mobile session/token design must provide short-lived access, rotation or replay detection for renewable credentials, server-side revocation, audience/issuer validation, and logout cleanup without storing bearer material in a general-purpose client cache.

#### SEC-SESS-005: Administrative Sessions

Administrative sessions must use a separate authentication audience and cookie/token namespace, shorter idle and absolute lifetimes than consumer sessions, recent passkey reauthentication for ADMIN-010 actions, immediate revocation after role or authenticator changes, and no persistent "remember me" path that bypasses user verification.

---

### 7.7 Mobile Security

#### SEC-MOB-001

Mobile applications must not embed privileged backend credentials.

#### SEC-MOB-002

Sensitive tokens must use platform-provided secure storage.

#### SEC-MOB-003

The mobile application must validate transport security and reject cleartext production traffic.

#### SEC-MOB-004

The application must minimize collection and retention of precise device location.

#### SEC-MOB-005

The mobile application must not log access tokens, refresh tokens, precise location, or other sensitive user information.

#### SEC-MOB-006: Local Data Lifecycle

The mobile application must inventory local files, databases, caches, logs,
screenshots, clipboard use, and backups by data classification. Logout,
account deletion, permission revocation, and retention expiry must remove the
affected local data and authentication material. Restricted data must be
excluded from cloud/device backups and task-switcher previews. If a supported
platform cannot enforce exclusion for a storage location, Restricted data
must not be persisted in that location.

#### SEC-MOB-007: Deep Links, WebViews, and Inter-App Input

Deep/universal links, push-notification navigation, intents, custom schemes, share-sheet input, native-module messages, and WebView messages must be treated as unauthenticated input and schema-validated against an allowlist of destinations and actions. WebViews must not receive authentication tokens, access arbitrary files/origins, or enable universal file access.

#### SEC-MOB-008: Release Integrity

Production mobile packages and any over-the-air code/configuration update must be signed, provenance-verifiable, bound to the intended application/environment, and distributed only through an approved channel. Update rollback and signing-key compromise procedures must be tested before enabling remote updates.

---

### 7.8 Logging and Monitoring

#### SEC-LOG-001

Security-relevant events must be logged, including:

* Authentication failures
* Account lockouts or throttling
* Password resets
* Session revocations
* Authorization failures
* Administrative actions
* Provider-configuration changes
* Data-suppression actions
* Abnormal API usage

#### SEC-LOG-002

Logs must not contain:

* Passwords
* Session identifiers
* Authentication tokens
* Provider secrets
* Full reset tokens
* Unnecessary precise location data

#### SEC-LOG-003

Logs must use synchronized timestamps and correlation identifiers.

#### SEC-LOG-004

Security alerts must be generated for material suspicious activity.

#### SEC-LOG-005

Audit logs must be protected against unauthorized modification and deletion.

The audit design must enforce append-only immutable/WORM retention appropriate
to the selected store **and** independently verifiable cryptographic tamper
evidence/order, use separate writer/reader/retention authorities, verify
integrity, audit all reads and exports, and alert on gaps, failed writes,
integrity failures, attempted alteration, unexpected clock movement, or
disabled collection. A high-impact administrative mutation must not succeed
without durable audit evidence; an approved emergency buffer may be used only
if it preserves ordering, integrity, and prompt alerting.

#### SEC-LOG-006: Log Safety and Access

Untrusted fields must be structured or escaped to prevent log forging. Access to logs, traces, dashboards, and exports must be least-privilege and audited. Retention, sampling, redaction, and correlation must not make account, location, authentication, administrative, or incident activity unnecessarily linkable.

#### SEC-LOG-007: Alert Ownership

Every security and data-integrity alert must have a severity, owner, response target, escalation path, runbook, and evidence-retention rule. Alert delivery and on-call access must be monitored and exercised; an alert rule that is disabled, silent, or unable to page must itself produce a signal through an independent path.

---

### 7.9 Privacy

#### SEC-PRIV-001

The platform must collect only personal data necessary for defined product functions.

#### SEC-PRIV-002

Precise location collection must require explicit user permission.

#### SEC-PRIV-003

The application must explain:

* What location data is collected
* Why it is collected
* Whether it is stored
* How long it is retained
* How users can revoke permission

#### SEC-PRIV-004

All users must be able to access, correct, export, and delete their personal data through the secure workflows in SEC-PRIV-008, subject only to narrowly documented legal, security, licensing, or immutable-audit retention exceptions. Applicable law may add stricter deadlines or rights but is not the minimum product baseline.

#### SEC-PRIV-005

Analytics and telemetry must avoid collecting unnecessary precise location or account identifiers.

#### SEC-PRIV-006: Location and Search Privacy

Exact coordinates, coordinate searches, saved/default locations, user-defined location names, alert locations, map viewports tied to an account, and location-bearing notification history are personal data. They must be private by default, minimized, excluded from logs and analytics unless essential and approved, retained only for the requested feature, and not sent to a third party with a stable account identifier. Nearby lookup must be performed locally or with reduced precision unless a documented feature test shows that the approved function cannot operate that way; any full-precision server or third-party flow requires privacy/security approval and the minimum retention.

#### SEC-PRIV-007: Processing Map and Third Parties

Before personal data is processed, the project must document the data subject, controller/processor role, fields, purpose, lawful basis where applicable, jurisdiction/transfer, retention, onward sharing, security controls, user notice/choice, and deletion path. This applies to cloud, map, geocoding, notification, analytics, observability, support, and backup providers.

#### SEC-PRIV-008: Rights Workflow Security

Access, export, correction, and deletion workflows must verify the requesting user, require recent authentication for sensitive output or destructive action, prevent cross-user disclosure, produce a short-lived protected export when needed, and notify the user. Rights evidence must record status without copying the exported or deleted personal data. Backup limitations and processor completion deadlines must be disclosed.

#### SEC-PRIV-009: Analytics and Minors Gate

Analytics using precise/saved location, account identifiers, or stable cross-session identifiers must remain disabled until a documented privacy impact assessment, legal basis/consent design, retention limit, and withdrawal mechanism are approved. Before account registration launches, the minimum-age/minors policy and any jurisdiction-specific parental-consent or child-privacy controls must be resolved; date of birth must not be collected merely to defer that decision.

#### SEC-PRIV-010: Notification Disclosure

Lock-screen, email-subject, and Web Push content must be minimized so a bystander or delivery provider cannot infer precise location, user-defined labels, account identity, or sensitive routines. Links must use an allowlisted application origin, contain no bearer credential, and reauthorize before displaying personal content.

---

### 7.10 Service, Queue, and Network Security

#### SEC-SVC-001: Security Zones and Private Data Services

Public ingress, authenticated user services, the administrative control plane, ingestion/worker processing, data services, observability/security operations, and CI/cloud management must be separate security zones with default-deny paths. Databases, caches, queues, object stores, secret/key stores, backups, and audit stores must not expose a public data-plane endpoint unless an approved threat model documents why no private path is possible.

#### SEC-SVC-002: Message Trust

Messages and jobs must carry an authenticated producer identity or be received through a producer-authorized channel, plus a schema version, bounded payload, idempotency key, correlation ID, creation/expiry time, and resource/tenant-equivalent scope. Consumers must reject unauthorized, expired, replayed, malformed, or over-budget messages and isolate poison messages in a restricted dead-letter store.

#### SEC-SVC-003: Egress Control

Runtime and build workloads must have default-deny outbound policy with destination, protocol, port, and identity restrictions appropriate to their function. A workload that parses untrusted provider data must not also possess broad cloud-control-plane, secret-store, or unrelated data-store permissions.

#### SEC-SVC-004: Store Separation

Public observations, user identity/session data, provider credentials/configuration, raw/quarantined provider payloads, security telemetry, and immutable audit/backup data must use separate logical security principals and access policies. A compromise of the public-read path must not grant write access to trusted observations or access to user, provider-secret, audit, backup, or control-plane data.

---

### 7.11 Software Supply Chain and Delivery

#### SEC-SUP-001: Source and Review Protection

Production-bound source, infrastructure, workflows, policy, AQI rules, and dependency changes must use protected branches, independent review by an authorized owner, and immutable history. Review bypass, force-push, and direct production changes must be disabled or treated as audited break-glass events.

#### SEC-SUP-002: Build and Artifact Integrity

CI must use isolated ephemeral jobs and short-lived least-privilege workload identity, must not expose secrets to untrusted contributions, and must pin third-party actions and build inputs to reviewed immutable versions. Releases must have a locked dependency graph, SBOM, scan evidence, signed provenance, integrity-verified artifacts, and promotion rather than rebuild between environments.

#### SEC-SUP-003: Vulnerability and Release Policy

The project must define risk-based remediation deadlines and a release-blocking policy for exploitable vulnerabilities, leaked secrets, malicious packages, license violations, failed tests, unsigned artifacts, and infrastructure-policy violations. Exceptions require a named owner, expiry, compensating control, and independent approval; unsupported runtimes or dependencies are prohibited.

#### SEC-SUP-004: Cloud and Terraform Control Plane

Terraform state, plans, outputs, variable files, and provider credentials must be classified and protected as potentially Restricted. State must be encrypted, versioned, access-logged, separately authorized per environment, and recoverable. Cloud/Terraform changes require policy checks, drift detection, a reviewed plan, scoped deployment identity, and a tested rollback or forward-recovery procedure.

---

### 7.12 Incident and Vulnerability Response

#### SEC-IR-001: Incident Response Plan

Before production, the project must maintain and exercise a plan covering security, privacy, provider compromise, false or stale public data, notification storms, admin takeover, secret/signing-key compromise, dependency compromise, denial of service, ransomware, and regional failure. The plan must assign severity, decision authority, containment, evidence preservation, credential/key rotation, public-data correction, recovery, third-party coordination, and legally required communication.

#### SEC-IR-002: Vulnerability Intake and Remediation

The project must publish or otherwise maintain a monitored private vulnerability-reporting channel, validate and prioritize reports, prevent retaliation against good-faith research, track remediation and disclosure, and verify fixes with regression tests. Internet-exposed and privileged surfaces require an independent penetration test before initial production and after material authentication, authorization, admin, or trust-boundary changes.

#### SEC-IR-003: Risk Acceptance

An unresolved Critical threat-model, test, or vulnerability finding blocks
initial production and must not be risk accepted for that release. A High
finding may receive a temporary acceptance only when it states the affected
assets, likelihood and impact, compensating controls, owner, near-term expiry,
verification plan, and approval by an accountable person independent of the
implementer. Emergency continuation of an already-running service follows
SEC-IR-001 incident authority and does not silently convert the finding into
accepted release risk.

---

## 8. Reliability and Data Quality

### REL-001: Freshness Classification

Observations must be classified as:

* Current
* Delayed
* Stale
* Unavailable

Thresholds must be configurable by provider and observation type.

### REL-002: Anomaly Detection

The ingestion pipeline must identify and quarantine suspicious observations before they are eligible for public presentation or alerts, including:

* Impossible coordinates
* Implausible pollutant values
* Sudden extreme changes
* Repeated identical values
* Future timestamps
* Excessively old timestamps

Anomaly policy must combine deterministic checks with provider/source context, preserve explainable reasons, and must not silently discard a record or auto-correct the source value. High-impact anomalies and unexpected drops in anomaly detection volume must alert an operator.

### REL-003: Reconciliation

The system must support deterministic reconciliation from every normalized or derived observation to the immutable raw/canonical provider record, adapter and rule versions, validation outcomes, administrative changes, and publication decision. Reconciliation failures for published data must alert and remove the affected value from current/alert eligibility until resolved.

### REL-004: Backup and Recovery

The platform must maintain encrypted, integrity-checked, versioned, and access-isolated backups of critical data, configuration, rule versions, audit evidence, and infrastructure state. Backup deletion/retention authority must be separate from normal runtime writers. Restores must occur in an isolated environment, validate data and security-control integrity, and must not make an expired credential or deleted personal record active again.

### REL-005: Recovery Objectives

Numeric recovery-point and recovery-time objectives for each critical data class and service, plus maximum tolerable public-data staleness and notification delay, must be approved before storage, queue, and regional-topology decisions are finalized and before production deployment.

### REL-006: Disaster Recovery

The project must document procedures for:

* Database restoration
* Provider credential rotation
* Regional outage recovery
* Cache reconstruction
* Reprocessing raw observations
* Restoring immutable audit evidence and cryptographic/key references
* Re-establishing service identities and authorization without restoring compromised credentials
* Correcting or withdrawing false public data and queued notifications
* Preserving deletion, suppression, and licensing state during restore

### REL-007: Recovery Exercises

Backup restoration, regional failover, provider compromise, administrator compromise, ransomware, signing/key compromise, queue replay, and stale/false-data withdrawal must be exercised on an approved cadence. Evidence must show that REL-005 objectives and data-integrity/privacy requirements were met; a failed exercise must create a tracked release risk.

### REL-008: Consistency During Failover

Multi-instance and multi-region designs must preserve authorization revocation, rule/configuration version, observation ordering, audit continuity, deletion/suppression state, and notification idempotency. A failover must not resurrect an old role, credential, record, cache entry, or alert state.

---

## 9. Observability

### OBS-001: Metrics

The system must collect metrics for:

* Request volume
* Error rate
* Response latency
* Cache performance
* Database performance
* Ingestion success rate
* Provider latency
* Data freshness
* Queue depth
* Notification delivery
* Authentication failures

### OBS-002: Distributed Tracing

Requests and background jobs must use correlation identifiers or distributed tracing across service boundaries without propagating credentials, raw session identifiers, precise location, or unnecessary personal data in trace context.

### OBS-003: Health Endpoints

Services must provide health and readiness information without exposing sensitive details.

Public liveness output must reveal only coarse status. Readiness, dependency, metrics, queue, build/version, topology, and diagnostic details must be private, authenticated, and authorized.

### OBS-004: Alerting

Operational alerts must be configured for:

* Provider failures
* Stale data
* Elevated API errors
* High latency
* Queue backlog
* Database failures
* Authentication anomalies
* Exhausted storage
* Failed backups
* Audit gaps or integrity failures
* Provider endpoint, credential, or trust-policy changes
* Publication of anomalous, stale, or unreconciled data
* Administrative recovery, break-glass, or privilege changes
* Security-control disablement or telemetry silence
* Unexpected cost or quota exhaustion

### OBS-005: Detection Validation

Security, privacy, and data-integrity detections must be mapped to the threat IDs in `ARCHITECTURE.md`, have an owner and runbook, and be tested before production and on an approved cadence. Monitoring must include both suspicious events and absence of expected signals so a disabled ingestion validator, audit writer, backup, or alert pipeline is detectable.

---

## 10. Required Logical Architecture Boundaries

The implementation must preserve the security authorities and failure domains below. They are logical boundaries and do not by themselves require separate deployables or products; `ARCHITECTURE.md` defines their data flows and trust boundaries.

1. **Public Web and Mobile Clients** — public/personalized presentation, accessible maps/charts, user-invoked location, bounded local public-data cache, and safe notification navigation.
2. **Separate Administrative Client and Ingress** — privileged origin/audience and managed-access defense in depth; never a hidden route within the consumer client.
3. **Public Edge and API** — documented current/history/search metadata, validation, DDoS/abuse/cost controls, public-cache safety, and no trusted-data write authority.
4. **Identity and User Boundary** — credentials, authenticators, recovery, sessions, profiles, saved locations, preferences, alerts, exports, deletion, and security notifications.
5. **Provider Fetch Boundary** — reviewed destinations, narrow egress/credentials, polling, bulk intake, retries, and no trusted-publication or cloud-control authority.
6. **Quarantine and Validation Boundary** — immutable raw/canonical evidence, parser/resource isolation, strict validation, anomaly/replay handling, and safe administrative viewing.
7. **Air-Quality Processing and Publication Gate** — versioned normalization, calculations, source/conflict policy, aggregation, freshness, licensing, reconciliation, and exclusive authority to publish a trusted observation version.
8. **Notification Boundary** — eligible event evaluation, consent/preferences, ordering/idempotency/expiry, bounded fan-out, privacy-minimized delivery, and authenticated status processing.
9. **Administrative Control Plane** — approved provider/rule/data-quality/user-role operations, reauthentication, dual control, masked data, write-only secrets, and durable audit.
10. **Private Data Boundaries** — separate identities and policy for credential/session, personalization, trusted current, historical/analytical, raw/quarantine, configuration/rules, queue, public cache, audit, observability, and backup data.
11. **Security and Operations Boundary** — secret/key management, tamper-evident audit, redacted telemetry/detection, incident access, and isolated backup/restore.
12. **Delivery and Cloud-Management Boundary** — protected source, dependencies, CI, registries, provenance/signing, Terraform state/plan/apply, Azure identity/control plane, and environment/region separation.

No physical consolidation may give a public-read, provider-parser, ordinary runtime, administrator, CI, backup, or auditor identity permissions that violate DATA-004, SEC-SVC-001–004, or the invariants in `ARCHITECTURE.md` §7.

---

## 11. User Stories

### Public User

* As a public user, I want to see current air quality near me so that I can decide whether to spend time outside.
* As a public user, I want to search for another city so that I can understand conditions there.
* As a public user, I want to view air quality on a map so that I can compare nearby areas.
* As a public user, I want to review historical conditions so that I can identify recurring patterns.
* As a public user, I want to understand which pollutant is driving unhealthy air quality.
* As a public user, I want to know how recent the data is so that I do not mistake stale data for current conditions.

### Authenticated User

* As an authenticated user, I want to save locations that matter to me.
* As an authenticated user, I want alerts when air quality becomes unhealthy.
* As an authenticated user, I want to configure my preferred air-quality standard.
* As an authenticated user, I want to manage notification preferences.
* As an authenticated user, I want to delete my account and personal data.

### Administrator

* As an administrator, I want to know when provider ingestion fails.
* As an administrator, I want to review rejected observations.
* As an administrator, I want to disable a malfunctioning provider.
* As an administrator, I want to backfill missing historical data.
* As an administrator, I want to suppress invalid observations.
* As an auditor, I want to review security-sensitive administrative actions.

---

## 12. Acceptance Criteria for Initial Release

The initial release is acceptable when:

1. The backend ingests data from at least one provider that passed ING-015 and SEC-007 review.
2. Adversarial ingestion tests prove that malformed, oversized, decompression-bomb, tested forgery classes, replayed, out-of-order, anomalous, suppressed, unreconciled, and stale records cannot enter or overwrite trusted-current/publication eligibility or trigger an alert; any permitted quarantine or versioned historical retention preserves its non-current state and provenance. Plausible data from a compromised authenticated provider remains a documented residual risk governed by OQ-045 and TM-01.
3. The public website displays current air-quality data on a map.
4. Users can search for supported locations.
5. Users can select a historical date range and view available trends.
6. Every displayed observation and derived value satisfies WEB-006 and is reproducible from immutable provenance and rule versions.
7. Tests at every freshness boundary prove that delayed, stale, unavailable, quarantined, and conflicting data are distinguished and that expired data is withdrawn from current and alert eligibility.
8. Users can register, authenticate, and sign out.
9. Authenticated users can save and remove locations.
10. Authenticated users can create an alert delivered through at least one launch-approved verified channel, opt out, and see the NOTIF-010 delivery state; spoofed callbacks and duplicate, stale, reordered, replayed, or over-budget events do not cause a false state or notification storm.
11. Administrators can review ingestion status and failures.
12. Administrators can enable or disable a provider.
13. Admin login, enrollment, authenticator replacement, recovery, role change, provider change, data correction/suppression, approval, and break-glass tests prove ADMIN-001–012, including mandatory phishing-resistant user verification and no weaker fallback.
14. Object-, field-, and function-level negative authorization tests cover every public, authenticated, workload, and administrative role-resource-action combination, including cross-user and self-elevation attempts.
15. Automated contrast/component checks and manual keyboard, screen-reader, zoom/text-scaling, reduced-motion, non-color, web, and mobile testing demonstrate NFR-A11Y-001–005; known failing design tokens are not shipped.
16. The API enforces the reviewed contract, schema, cache policy, method/content-type rules, pagination, rate/concurrency/cost limits, geographic/date/result bounds, and request/response-size limits under bypass and distributed-load tests; webhook/callback tests cover authentication, environment/account/type binding, strict schema, trusted-time expiry, replay, idempotency, scope, and rate limits under SEC-API-009.
17. The public edge/API, identity/session path, ingestion/quarantine/publication path, trusted-current store/cache, notification pipeline, audit/detection plane, key/identity dependencies, and backup/restore path expose approved private operational metrics and coarse health status without leaking topology or secrets.
18. Approved numeric SLO/RPO/RTO/staleness objectives are met in load, dependency-failure, retry-storm, backup-restore, regional-failover, and ransomware recovery exercises.
19. Provider attribution and licensing requirements are displayed and enforced.
20. Critical user, mobile, workload, ingestion, notification, and administrative workflows are covered by positive, negative, concurrency, and abuse-case automated tests.
21. SSRF tests cover redirects, DNS rebinding/change, IPv4/IPv6 private/link-local/loopback/reserved and cloud-metadata destinations, ambiguous URLs, alternate schemes/ports, credential forwarding, and oversized/slow responses at application and network-egress layers.
22. Deployed configuration proves encryption, key separation, private data-service access, distinct workload identities, least privilege, secret rotation/revocation, safe Terraform state, queue trust, cache isolation, and immutable/tamper-evident audit controls.
23. Credential stuffing, password spraying, account enumeration, reset/verification flooding, token replay/races, session fixation, cross-site request forgery, and authenticator downgrade tests pass without creating a trivial victim lockout.
24. Web and mobile tests prove location permission denial/revocation, no unapproved background collection, secure local storage, cache/backup/screenshot lifecycle, deep-link validation, cleartext rejection, logout, session revocation, export, and deletion propagation through processors.
25. Supply-chain evidence includes a locked dependency graph, SBOM, scan/policy results, signed provenance and artifacts, protected promotion approval, rollback evidence, and no finding that violates SEC-SUP-003; every permitted exception is current, scoped, independently approved, and unexpired.
26. Audit-integrity, log/trace redaction, alert-delivery, telemetry-silence, privileged-access, trusted-time skew/jump/source-loss, and cross-region ordering tests demonstrate SEC-008, SEC-LOG-001–007, and OBS-005.
27. Independent penetration testing of Internet-exposed, authentication, personalized-resource, and administrative surfaces has no unresolved Critical finding and no unresolved High finding without a current SEC-IR-003 exception.
28. Incident-response exercises cover administrator takeover, provider/data poisoning, false/stale public data, location disclosure, dependency/CI compromise, notification storm, and key/credential compromise.
29. Provider licenses, launch geography, privacy jurisdictions and transfers, processor contracts, retention/deletion schedules, minors policy, administrative permission/approval matrix, alert basis/channel, query/export limits, product domains/RP IDs, recovery objectives, and OQ-045 authoritative test vectors/confidence/withdrawal policy are approved rather than unresolved.
30. The threat model has been validated by product, security, privacy/legal, air-quality domain, engineering, mobile/client, accessibility, and operations representatives; disagreements, unverified controls, accepted residual risks, owners, and review dates are recorded.

---

## 13. Open Questions

Each item has a stable identifier and an explicit gate. Every item is **Open**
with owner and ADR/evidence **TBD** unless a later decision record says
otherwise. A decision is complete only when it records an accountable owner,
date, rationale, threat impact, testable security/privacy/safety criteria,
rollback/exit plan where relevant, and approval. **Deferred** and **Feature
introduction** items do not block v1 unless their feature is brought into
scope.

- **OQ-001 — Release / product identity:** What product name should replace the working title?
- **OQ-002 — Ingestion architecture:** Which public air-quality provider or providers will be used?
- **OQ-003 — Ingestion architecture:** What licensing, attribution, storage, and redistribution restrictions apply to each provider?
- **OQ-004 — Release / geography:** Which countries and regions must be supported at launch?
- **OQ-005 — Publication architecture:** Which air-quality index is the default?
- **OQ-006 — Publication architecture:** May users switch between national air-quality standards?
- **OQ-007 — Publication architecture:** Will the platform calculate AQI from pollutant measurements?
- **OQ-008 — Publication architecture:** How are conflicting provider observations presented?
- **OQ-009 — Publication architecture:** How are multiple stations within one city or map region aggregated?
- **OQ-010 — Publication architecture:** What age qualifies an observation as delayed, stale, or unavailable?
- **OQ-011 — Release / data:** How much historical data must be imported before launch?
- **OQ-012 — API architecture:** What maximum historical date range is supported per query?
- **OQ-013 — Map feature introduction:** Which map provider or on-device alternative will be used?
- **OQ-014 — Map feature introduction:** What map-tile licensing and cost constraints apply?
- **OQ-015 — Mobile architecture:** React Native is selected; which supported version, native modules, release-signing process, deep-link domains, and over-the-air update policy are approved?
- **OQ-016 — Authentication architecture:** Password plus optional passkey is selected for consumers; what distinct browser-cookie and mobile-session/token lifecycles apply?
- **OQ-017 — Administrative architecture:** Passkeys are mandatory for administrators; which WebAuthn library, RP IDs/origins, authenticator-assurance policy, bootstrap, replacement, recovery, and break-glass design is approved?
- **OQ-018 — Alert feature introduction:** Which notification channels are required at launch?
- **OQ-019 — Alert feature introduction:** Are alerts based on provider AQI, platform-calculated AQI, pollutant measurements, or an approved combination?
- **OQ-020 — Privacy release gate:** Is anonymous or account-linked usage analytics required at launch?
- **OQ-021 — Developer-API feature introduction:** Will the public API be offered to third parties?
- **OQ-022 — Developer-API feature introduction:** Will API keys or developer accounts be required?
- **OQ-023 — Deferred:** Are paid plans or commercial API access expected later?
- **OQ-024 — Privacy release gate:** Which privacy jurisdictions apply?
- **OQ-025 — Reliability architecture:** What numeric availability, recovery-time, and recovery-point objectives apply?
- **OQ-026 — Cloud architecture:** Azure/Terraform is selected; what compute, subscription/environment, ingress/egress, private-network, key/secret, region, backup, and CI/state topology is approved?
- **OQ-027 — Storage architecture:** How long are raw provider responses retained?
- **OQ-028 — Export feature introduction:** May users export historical data?
- **OQ-029 — Comparison feature introduction:** Will multiple-location comparison be supported?
- **OQ-030 — Deferred:** Is predictive air-quality forecasting planned later?
- **OQ-031 — Deferred:** Will wildfire, pollen, weather, or smoke-plume overlays be added later?
- **OQ-032 — Release / localization:** Which languages are required at launch?
- **OQ-033 — Administrative architecture:** What complete ADMIN-002 permission, JIT, separation-of-duties, approval, support-access, and termination matrix is approved?
- **OQ-034 — Publication governance:** Who may suppress or correct public observations, at which impact thresholds?
- **OQ-035 — Release / data assurance:** Is formal regulatory, government, or research-grade validation required?
- **OQ-036 — Identity architecture:** Which canonical product/admin/email/link domains, mobile app identifiers, DNS/certificate owners, and RP IDs bind passkeys and trusted navigation?
- **OQ-037 — Privacy/data release gate:** What exact retention/deletion values apply to every DATA-001 category, including logs, traces, caches, raw records, notifications, audit, backups, and processor copies?
- **OQ-038 — Processor release gate:** Which map, geocoding, notification, analytics, observability, support, edge, and cloud processors receive which fields, where, and under which contracts?
- **OQ-039 — Privacy release gate:** What minimum-age/minors and child-privacy policy applies?
- **OQ-040 — Analytics feature introduction:** If OQ-020 approves analytics, what purpose, lawful basis/consent, identifiers, sampling, retention, processor, and opt-out design applies?
- **OQ-041 — Security release gate:** What numeric response, remediation, breach-assessment, and legally required notification targets apply?
- **OQ-042 — Governance release gate:** Who owns and approves security architecture, privacy, publication policy, incident response, on-call operation, and residual risk?
- **OQ-043 — Security architecture:** Which products/patterns satisfy edge protection, workload identity, egress, private data, audit, keys/secrets, and detection without a single fail-open dependency?
- **OQ-044 — Developer-API feature introduction:** Which tiers, credentials, quotas, redistribution controls, and abuse response comply with provider licensing?
- **OQ-045 — Publication release gate:** Which independent authoritative vectors, corroboration, confidence policy, and withdrawal/escalation thresholds govern public-data and alert integrity?

---

## 14. Future Enhancements

Potential future capabilities include:

* Predictive air-quality forecasts
* Wildfire and smoke-plume overlays
* Weather overlays
* Pollen and allergen data
* Indoor sensor integration
* Personal exposure tracking
* Route planning based on air quality
* School and workplace dashboards
* Public data exports
* Developer API subscriptions
* Machine-learning anomaly detection
* Community sensor integrations
* Government alert integrations
* Comparative city reports
* Long-term climate and pollution analysis
* Researcher data-access tools
* Widget and embeddable map support
