# Air We Go — Claude Code Guide

Global air-quality platform: responsive web app, iOS/Android app, and REST API.
Status: **greenfield — no implementation exists yet.** These four files are
the entire source of truth; if any is missing, stop and tell the user rather
than guessing its content.

@REQUIREMENTS.md
@ARCHITECTURE.md
@SECURITY.md
@DESIGN.md

## Reading order

- **REQUIREMENTS.md** — what to build (functional/non-functional requirements,
  IDs like `WEB-001`, `SEC-AUTH-002`). Cite requirement IDs in code comments,
  commits, and PRs when a change implements or affects one.
- **ARCHITECTURE.md** — the chosen stack (Node.js, React, React Native, REST,
  Azure/Terraform) and what's still **TO BE DECIDED** (framework, data-store
  engines, queue tech, observability stack, CI/CD tooling, passkey library).
  Don't pick these unilaterally — flag the open decision to the user.
- **SECURITY.md** — enforceable security rules per stack layer. This is a
  floor, not a suggestion; nothing you write may fall below it.
- **DESIGN.md** — brand, color, typography, accessibility (WCAG 2.2 AA),
  air-quality visual encoding. Required reading before any UI work.

## Working style

- This repo currently has no source tree, package manifest, or CI config.
  Before assuming a build/test/lint command exists, check for it — don't
  invent one.
- When a requirement or architecture point is genuinely unresolved (see
  REQUIREMENTS.md §13 "Open Questions" and ARCHITECTURE.md "Open / deferred
  decisions"), treat it as blocking for that piece of work and surface it —
  do not silently choose a provider, library, or data model to fill the gap.
- Keep provider, map, and geocoding integrations behind the adapter
  interfaces ARCHITECTURE.md specifies (ING-003) — never hardcode a specific
  vendor into core logic.
- Business rules for AQI calculation, categorization, and aggregation must be
  versioned and test-covered (NFR-MAINT-002) — treat these as their own
  reviewable unit, not incidental logic.

## GitHub issues

Every new issue **must** follow `REQUIREMENT_TEMPLATE.md` — fill in Metadata,
Requirement, Scope, Security Context, Standards Alignment, Acceptance
Criteria (independently testable), Failure Behavior, Test Strategy, and
Dependencies. An issue that isn't a structured, testable requirement in this
shape should be reformatted before it's filed.

## Undecided workflow items (do not assume — ask if it matters for the task)

- Branch naming / commit message convention: **not yet defined**
- PR review / approval process, CODEOWNERS: **not yet defined**
- CI/CD pipeline and required checks: **not yet defined** (NFR-MAINT-005
  requires automation; tooling unchosen)
- Test/lint/build commands: **not yet defined** (no manifest exists)
- Release/versioning process: **not yet defined**
- Issue labels / triage workflow beyond the template requirement above: **not
  yet defined**
