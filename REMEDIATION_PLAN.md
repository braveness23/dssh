# Remediation Plan (Phased)

**Branch:** `remediation-plan`  
**Last updated:** 2026-04-05 03:46:39  

This plan upgrades `dssh` into a trustworthy, reputable FOSS utility by adding:
- repeatable local checks (`make lint`, `make security`, `pre-commit`)
- CI enforcement on pushes, PRs/MRs, and merges
- security scanning and dependency hygiene
- standard FOSS project hygiene (CoC, contributing, security policy, templates)

The plan is designed to be executed as **separate commits for separate concerns**, with a clear status ledger so we can resume gracefully if interrupted.

## Status ledger

> Rule: **Only mark items complete after they are merged into `main` of this fork** (or into upstream if/when PR’d).

- [ ] Phase 0 — Planning baseline committed
- [ ] Phase 1 — Local developer loop (Make + pre-commit)
- [ ] Phase 2 — FOSS hygiene (docs + templates)
- [ ] Phase 3 — CI: tests (Linux/macOS/Windows) required
- [ ] Phase 4 — CI: lint required
- [ ] Phase 5 — CI: security required
- [ ] Phase 6 — CodeQL required
- [ ] Phase 7 — Release workflow hardening
- [ ] Phase 8 — Test expansion (targeted)

Each phase below includes:
- **Deliverables** (files / workflows)
- **Acceptance criteria** (what “done” means)
- **Resume from here** steps

---

## Phase 0 — Plan + scope lock (docs only)

**Goal:** Establish the roadmap and enforce “separate commits per concern”.

**Deliverables**
- `REMEDIATION_PLAN.md` (this file)

**Acceptance criteria**
- File is merged into the working branch.

**Resume from here**
- Start Phase 1.

---

## Phase 1 — Local developer loop: Make targets + pre-commit

**Goal:** Contributors can run the same checks locally that CI runs.

**Deliverables**
- `.pre-commit-config.yaml`
- `Makefile` additions:
  - `fmt`: run `gofmt -w` on Go files
  - `fmt-check`: fail if `gofmt` would change files
  - `lint`: runs `go vet`, `staticcheck`, `golangci-lint`, and `fmt-check`
  - `security`: runs `gosec` and `govulncheck`
  - `test`: keep `go test ./...` (existing)
  - `test-race`: run `go test -race ./...` when supported
- `tools.go` (optional but recommended) to pin tool versions for `golangci-lint`, `staticcheck`, `gosec`, `govulncheck`.

**Acceptance criteria**
- `make lint` passes on Linux/macOS/Windows (where tools are supported)
- `make security` passes on Linux/macOS/Windows (or documented exceptions; CI must match)
- `pre-commit run --all-files` passes

**Resume from here**
1. Install hooks: `pre-commit install`
2. Run: `pre-commit run --all-files`
3. Run: `make lint` then `make security`

---

## Phase 2 — FOSS hygiene: policies + templates + automation metadata

**Goal:** Standard trusted FOSS repo scaffolding.

**Deliverables**
- `CODE_OF_CONDUCT.md` (Contributor Covenant)
- `CONTRIBUTING.md` (dev setup, how to run checks, how to file issues)
- `SECURITY.md` (responsible disclosure process)
- `.github/ISSUE_TEMPLATE/bug_report.yml`
- `.github/ISSUE_TEMPLATE/feature_request.yml`
- `.github/ISSUE_TEMPLATE/config.yml` (contact links)
- `.github/PULL_REQUEST_TEMPLATE.md`
- `.github/dependabot.yml` for:
  - `gomod` updates
  - GitHub Actions updates

**Acceptance criteria**
- All files present and coherent
- Contributing doc includes: `make test`, `make lint`, `make security`, `pre-commit`

**Resume from here**
- Verify templates render on GitHub and docs match the repo.

---

## Phase 3 — CI: tests (required) on push/PR/merge

**Goal:** Every change is tested across OSes.

**Deliverables**
- `.github/workflows/ci-test.yml`
  - triggers: `push`, `pull_request`
  - matrix: `ubuntu-latest`, `macos-latest`, `windows-latest`
  - steps:
    - checkout
    - setup Go (pinned to repo’s Go version)
    - `go test ./...`
    - run `go test -race ./...` **where supported** (typically Linux; document any exclusions)

**Acceptance criteria**
- Workflow runs on every push and PR
- Branch protection can require this check

**Resume from here**
- Inspect failing OS job logs; fix portability issues first.

---

## Phase 4 — CI: lint (required)

**Goal:** Enforce style and static analysis.

**Deliverables**
- `.github/workflows/ci-lint.yml`
  - runs `make lint`
- `.golangci.yml` (reasonable defaults; avoid noisy linters)

**Acceptance criteria**
- `ci-lint` required and passing
- gofmt is enforced (no formatting drift)

**Resume from here**
- Run `make fmt` locally, then `make lint`.

---

## Phase 5 — CI: security (required)

**Goal:** Catch common security issues in code and dependencies.

**Deliverables**
- `.github/workflows/ci-security.yml`
  - runs `make security`
  - uses caching where helpful

**Acceptance criteria**
- `ci-security` required and passing on PRs
- If a tool is unsupported on an OS runner, explicitly skip on that OS and document why (CI must reflect the same policy)

**Resume from here**
- Run `make security` locally and address findings.

---

## Phase 6 — CodeQL (required)

**Goal:** Add deep static analysis scanning.

**Deliverables**
- `.github/workflows/codeql.yml` (Go)

**Acceptance criteria**
- CodeQL runs on PRs (and scheduled)
- CodeQL check is required via branch protections

**Resume from here**
- Fix CodeQL alerts with minimal, targeted commits.

---

## Phase 7 — Release workflow hardening

**Goal:** Releases should only be cut from passing code, with least-privilege permissions.

**Deliverables**
- Update existing release workflow(s) to:
  - pin action versions
  - set minimal permissions
  - reuse the same `make test`/`make lint`/`make security` gates where appropriate

**Acceptance criteria**
- A release tag build fails if tests/lint/security fail

**Resume from here**
- Simulate by pushing a test tag in the fork.

---

## Phase 8 — Test expansion (targeted)

**Goal:** Increase confidence in security-sensitive and data-handling code.

**Deliverables (suggested)**
- `internal/ssh`:
  - tests for `escapeShellSingleQuote`
  - tests for `buildArgs` (password/key modes, directory behavior)
- `internal/db`:
  - CRUD tests using a temporary DB path

**Acceptance criteria**
- Added tests are stable across OS runners
- Critical logic has coverage and regression protection

**Resume from here**
- Prioritize `internal/ssh` tests first.

---

## Completion criteria ("trustworthy")

We consider this fork “trustworthy” when:
- `make test`, `make lint`, `make security` exist and are documented
- pre-commit hooks enforce formatting/lint locally
- CI runs on push + PR with Linux/macOS/Windows and required checks are enabled
- CodeQL + dependency updates (Dependabot) are enabled
- FOSS hygiene docs/templates exist

## Notes for upstream PR

When upstreaming, keep PRs small:
- one PR per phase (or per 1–2 phases)
- ensure CI runtimes are acceptable
- include rationale in PR descriptions
