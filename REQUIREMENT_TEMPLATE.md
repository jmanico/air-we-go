# Requirement Template — Web and API Applications

> Use this template to define atomic, unambiguous, traceable, and independently testable requirements for web applications and APIs. Normative terms such as **MUST**, **MUST NOT**, **SHOULD**, and **MAY** are to be interpreted as described in BCP 14 (RFC 2119 and RFC 8174).

## Metadata

- **Requirement ID**: `REQ-[MODULE]-[###]`  
  Example: `REQ-AUTH-001`
- **Title**: [Concise, descriptive requirement name]
- **Version**: [Semantic version, such as `1.0.0`]
- **Status**: [Draft | In Review | Approved | Implemented | Deprecated]
- **Author**: [Name or team]
- **Owner**: [Team or role accountable for the requirement]
- **Reviewers**: [Required technical, security, compliance, or product reviewers]
- **Created**: [ISO 8601 date: `YYYY-MM-DD`]
- **Last Updated**: [ISO 8601 date: `YYYY-MM-DD`]
- **Priority**: [Critical | High | Medium | Low]
- **Classification**: [Functional | Security | Privacy | Compliance | Performance | Reliability | Operational]

## Requirement

- **Normative Statement**: [A single, atomic, implementation-independent statement describing what the system MUST or MUST NOT do.]
- **Rationale**: [Business, security, privacy, regulatory, operational, or threat-driven justification for the requirement.]
- **Source**: [Business objective, policy, regulation, threat model, architecture decision, defect, incident, or parent requirement.]
- **Design Reference**: [Reference the relevant section of `DESIGN.md`, architecture decision record, data-flow diagram, or interface specification. Update the design documentation when the requirement introduces a material architectural change.]
- **Assumptions**: [Conditions assumed to be true for this requirement.]
- **Out of Scope**: [Related behavior explicitly excluded from this requirement.]

## Scope

- **Applies To**: [Web Application | API | Both]
- **Environment(s)**: [Development | Test | Staging | Production | All]
- **Components**: [Examples: Authentication Service, Transaction API, Account Portal]
- **Interfaces**: [Endpoints, event topics, queues, user interfaces, protocols, or trust-boundary crossings]
- **Actors**: [Examples: Authenticated Customer, Service Account, Internal Administrator, Third-Party Aggregator]
- **Assets Protected**: [Data, credentials, transactions, services, models, configuration, or other protected resources]
- **Data Classification**: [Public | Internal | Confidential | Restricted/PII | Regulated]
- **Applicable Data Types**: [Examples: credentials, access tokens, PAN, account data, audit records, prompts, model output]

## Security and Privacy Context

- **Security Objective(s)**: [Confidentiality | Integrity | Availability | Authenticity | Authorization | Accountability | Non-Repudiation | Privacy]
- **Defense Layer(s)**: [Architecture | Authentication | Authorization | Input Validation | Business Logic | Output Encoding | Sanitization | Cryptography | Monitoring]
- **Threats Addressed**: [Reference specific threats or weaknesses, such as an OWASP Top 10 category, CWE identifier, CAPEC pattern, or STRIDE category.]
- **Abuse Cases**: [Describe realistic malicious, unauthorized, or unintended uses that the requirement prevents or limits.]
- **Trust Boundary**: [Identify where the requirement enforces or crosses a trust boundary, such as an API gateway, service mesh, browser-to-server boundary, or third-party integration.]
- **Zero Trust Considerations**: [Describe how identity, device state, input, context, and authorization are independently verified rather than implicitly trusted.]
- **Least Privilege Considerations**: [Identify the minimum permissions required and how excessive privilege is prevented.]
- **Privacy Considerations**: [Describe data minimization, purpose limitation, retention, consent, disclosure, and data-subject implications.]
- **Logging Restrictions**: [Identify data that MUST NOT be logged, such as credentials, access tokens, raw PAN, secrets, or unnecessary personal data.]

## Standards and Control Alignment

Use exact control identifiers and versions whenever possible. Enter `N/A` when a framework does not apply.

- **OWASP ASVS**: [Version and control ID, such as `ASVS 5.0 V2.1.1`]
- **OWASP API Security Top 10**: [Category and version, if applicable]
- **OWASP AISVS**: [Version and control ID, if AI or LLM components are involved]
- **CWE**: [Weakness ID and title]
- **NIST SP 800-53**: [Revision, control family, and control ID, such as `Rev. 5 SI-10`]
- **NIST SP 800-207**: [Applicable Zero Trust principle]
- **Regulatory**: [Applicable PCI DSS, GLBA, Regulation P, SOX, OCC, FFIEC, GDPR, CCPA, or other requirement]
- **Other Standards**: [Examples: OAuth security BCP, FIDO2/WebAuthn, ISO/IEC 27001, ISO/IEC 42001, or organization-specific policy]
- **Control Mapping Notes**: [Explain how the requirement satisfies or contributes to each referenced control.]

## Acceptance Criteria

Each acceptance criterion MUST be objective, independently testable, and traceable to the normative requirement. Use Given/When/Then syntax or explicit pass/fail conditions. Include positive, negative, boundary, and abuse-case tests as applicable.

1. **AC-01 — Expected Behavior**  
   Given [precondition], when [authorized or valid action], then [observable expected result].

2. **AC-02 — Invalid or Malicious Input**  
   Given [precondition], when [invalid, malformed, unexpected, or malicious input is submitted], then [the system rejects the input, performs no unauthorized processing, and returns the specified response].

3. **AC-03 — Authorization or Trust-Boundary Failure**  
   Given [precondition], when [an actor lacks the required identity, permission, context, or trust], then [access is denied and no protected data or operation is exposed].

4. **AC-04 — Boundary Conditions**  
   Given [minimum, maximum, empty, duplicate, expired, replayed, or otherwise relevant boundary condition], when [action], then [expected result].

5. **AC-05 — Prohibited Behavior**  
   [Explicitly state what MUST NOT occur, including side effects, data disclosure, partial processing, insecure fallback behavior, or control bypass.]

6. **AC-06 — Auditability**  
   Given [security-relevant or compliance-relevant event], when [event occurs], then [the required audit event is recorded with the approved fields and without prohibited sensitive data].

## Failure and Recovery Behavior

- **Invalid Input**: [Specify rejection behavior, HTTP status or protocol error, response schema, correlation identifier, and confirmation that no prohibited processing occurs.]
- **Authentication Failure**: [Specify externally observable behavior, throttling, logging, and information-disclosure constraints.]
- **Authorization Failure**: [Specify denial behavior and confirm that protected resource existence or internal authorization logic is not unnecessarily disclosed.]
- **Dependency Failure**: [Specify timeout, retry, backoff, circuit-breaker, queueing, or degradation behavior.]
- **System Error**: [Fail Closed | Fail Securely | Fail Open with documented and approved justification]
- **Transaction Behavior**: [Specify atomicity, rollback, idempotency, duplicate handling, and partial-failure behavior.]
- **User-Facing Error**: [Specify safe, actionable messaging that does not expose stack traces, secrets, internal identifiers, or implementation details.]
- **Security Logging**: [Specify event type, severity, required fields, correlation identifiers, and prohibited fields.]
- **Alerting**: [Define alert condition, threshold, aggregation window, severity, destination, and escalation path.]
- **Recovery Objective**: [Specify recovery expectations, including RTO/RPO where applicable.]

## Verification and Test Strategy

- **Verification Method**: [Inspection | Analysis | Demonstration | Automated Test | Manual Test | Security Assessment]
- **Unit Tests**: [Identify testable logic, boundary cases, and required branch or mutation coverage.]
- **Integration Tests**: [Identify cross-component, cross-service, dependency, protocol, and trust-boundary validation.]
- **End-to-End Tests**: [Identify user-visible or API-visible workflows that verify the requirement in a production-like environment.]
- **Negative Tests**: [Identify malformed, unauthorized, expired, replayed, duplicate, out-of-order, or malicious scenarios.]
- **Security Tests**: [Specify applicable SAST rules, DAST targets, dependency checks, fuzzing input classes, abuse-case tests, and penetration-test procedures.]
- **Performance Tests**: [Specify latency, throughput, concurrency, resource, or degradation tests when applicable.]
- **Resilience Tests**: [Specify timeout, retry, failover, dependency outage, and recovery tests when applicable.]
- **Compliance Tests**: [Specify automated evidence collection, configuration validation, policy evaluation, log verification, and audit artifacts.]
- **Coverage Target**: [Example: at least 80% branch coverage for the implementing module, with 100% coverage of security-critical decision paths.]
- **Required Evidence**: [Test reports, logs, screenshots, configuration snapshots, scan results, approvals, or other artifacts required to demonstrate compliance.]

## Observability

- **Metrics**: [Counters, rates, latency distributions, error rates, rejection rates, saturation indicators, or business-security metrics.]
- **Logs**: [Required structured events and fields, including correlation or trace identifiers.]
- **Traces**: [Required spans or attributes across trust boundaries and service dependencies.]
- **Dashboards**: [Operational or security dashboards that MUST expose the requirement’s health and enforcement status.]
- **Detection Coverage**: [Rules or analytics used to detect attempted bypass, abuse, control failure, or anomalous behavior.]
- **Retention**: [Required retention period and access restrictions for telemetry and audit evidence.]

## Dependencies and Traceability

- **Parent Requirement**: [Higher-level business, product, security, or compliance requirement]
- **Upstream Dependencies**: [Requirements or capabilities that MUST be satisfied first]
- **Downstream Dependencies**: [Requirements, components, or controls that depend on this requirement]
- **External Dependencies**: [Third-party services, identity providers, HSMs, payment processors, model providers, or core platform APIs]
- **Related Requirements**: [Associated requirement IDs]
- **Conflicting Requirements**: [Known conflicts, required tradeoffs, or precedence decisions]
- **Implementation References**: [Source files, modules, pull requests, issues, configuration, infrastructure resources, or deployment manifests]
- **Test References**: [Test case IDs, test suites, CI jobs, security scans, or audit procedures]

## Implementation Constraints and Guidance

- **Technical Constraints**: [Required or prohibited technologies, protocols, libraries, deployment patterns, or platform capabilities]
- **Performance Constraints**: [Latency, throughput, concurrency, payload-size, or resource budgets]
- **Compatibility Constraints**: [Supported clients, protocol versions, browsers, API versions, or migration requirements]
- **Regulatory or Delivery Deadline**: [Applicable date and source]
- **Secure Defaults**: [Configuration and behavior that MUST be enabled by default]
- **Anti-Patterns**: [Explicitly prohibited approaches]
  - The implementation MUST NOT rely on client-side validation as the sole enforcement mechanism.
  - The implementation MUST NOT log credentials, secrets, access tokens, raw payment-card data, or unnecessary personal information.
  - The implementation MUST NOT silently bypass or disable a security control when a dependency fails.
- **AI-Assisted Development Guidance**: [Reference approved AI-development policies, repository instruction files such as `CLAUDE.md`, prompt libraries, data-handling restrictions, generated-code review requirements, and mandatory human approval gates.]
- **Implementation Notes**: [Non-normative guidance that helps implementers without weakening or expanding the requirement.]

## Exceptions and Risk Acceptance

- **Exception Allowed**: [Yes | No]
- **Exception Conditions**: [Conditions under which an exception may be requested]
- **Compensating Controls**: [Controls required to reduce the resulting risk]
- **Approver**: [Authorized risk owner]
- **Expiration Date**: [ISO 8601 date after which the exception MUST be reviewed or removed]
- **Tracking Reference**: [Risk record, exception ticket, or governance-system identifier]

## Approval

- **Product Approval**: [Name, role, date]
- **Engineering Approval**: [Name, role, date]
- **Security Approval**: [Name, role, date]
- **Privacy Approval**: [Name, role, date, or N/A]
- **Compliance Approval**: [Name, role, date, or N/A]
- **Final Approval Status**: [Pending | Approved | Rejected]
