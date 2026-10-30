# AGENTS

Oracle Buffet is specified but not built. The imported files are the source of truth.

@REQUIREMENTS.md
@ARCHITECTURE.md
@SECURITY.md
@DESIGN.md

Read `REQUIREMENT_TEMPLATE.md` only when you write or review a leaf implementation issue.

## Guardrails

- Implement only behavior, components, and controls that the specification defines. If a needed fact is missing, record it as `TO BE DECIDED` and stop. Do not guess.
- Do not implement anything that an open item (`OQ-*`, `DQ-*`, `AQ-*`, `OD-*`) leaves undecided.
- If two specification files conflict, stop and report the conflict with the IDs on each side. Do not choose one.
- Do not add a dependency, component, external service, or data flow that `ARCHITECTURE.md` does not list. Propose it in a decision issue.
- A change to behavior, architecture, design, or a security rule updates the owning specification file in the same change. Keep existing IDs. Give new items the next free ID. Never reuse a retired ID.
- Each specification file states its domain in its first lines. Write each fact once, in its owning file; other files cite its ID. When a fact fits more than one file: security controls go to `SECURITY.md`, externally observable product behavior goes to `REQUIREMENTS.md`, and everything else goes to its owning file.
- Cite the affected specification IDs (`FR-*`, `NFR-*`, `DR-*`, `TB-*`, `SC-*`, and open items `OQ-*`, `DQ-*`, `AQ-*`, `OD-*`) in issues, pull requests, and tests.

## Issues

- Every leaf implementation issue MUST follow `REQUIREMENT_TEMPLATE.md`.
- Epic, workstream, and decision issues use concise purpose-specific bodies that link to affected specification IDs and issues.

## Working Style

- Make the smallest change that satisfies the issue. Do not refactor or add features outside its scope.
- A change is done only when the tests named in its issue pass and each cited `SC-*` verification passes.
- Report failures and skipped steps with the output. Do not claim a result you did not verify.

## Workflow

- Repository layout: TO BE DECIDED
- Package managers and lockfile policy: TO BE DECIDED (AQ-15)
- Build, run, lint, and test commands: TO BE DECIDED
- Test frameworks and coverage target: TO BE DECIDED
- Local development environment: TO BE DECIDED
- Database migration process: TO BE DECIDED
- Branch, commit, and pull request conventions: TO BE DECIDED
- Issue labels and states: TO BE DECIDED
- Required human review before merge: TO BE DECIDED
