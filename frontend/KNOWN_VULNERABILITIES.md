# Known Pre-existing Dependency Vulnerabilities

These 9 vulnerabilities (8 high, 1 critical) were identified by `npm audit` and
are **pre-existing** — they existed before the a11y PR and are not introduced by
it. Fixing them requires breaking-version upgrades to transitive dependencies
that are out of scope for this PR.

## Summary

| Severity | Count | Affected package(s) |
|----------|-------|---------------------|
| Critical | 1     | `happy-dom`         |
| High     | 8     | `axios`, `brace-expansion` / `postcss` |

## Details

### `happy-dom` — Critical

`happy-dom <=20.8.8` is a **dev-only** test dependency (used by Vitest's jsdom
environment). It is never bundled into or served from the production build.

- GHSA-37j7-fg3j-429f — VM Context Escape → RCE
- GHSA-w4gp-fjgq-3q4g — fetch credentials use page-origin cookies
- GHSA-6q6h-j7hj-3r64 — ECMAScriptModuleCompiler unsanitized export names

Fix requires `npm audit fix --force`, which would install
`happy-dom@20.11.1` — a breaking change to the test harness.

### `axios` / `@stellar/stellar-sdk` — High (multiple CVEs)

`@stellar/stellar-sdk@15.x` depends on `axios@1.x` which has numerous
prototype-pollution and request-smuggling advisories. The fix path is
`@stellar/stellar-sdk@16.1.0`, a breaking semver upgrade that may require
API changes to the Stellar integration layer. Upgrading is tracked separately.

### `brace-expansion` / `minimatch` / `glob` / `test-exclude` / `@vitest/coverage-v8` — High

- GHSA-3jxr-9vmj-r5cp — DoS via exponential brace expansion
- GHSA-mh99-v99m-4gvg — OOM via unbounded expansion length

These are transitive dev dependencies of `@vitest/coverage-v8`. Fix requires
`@vitest/coverage-v8@4.1.10`, a breaking change.

### `postcss` — High

- GHSA-r28c-9q8g-f849 — Path traversal in source map auto-loading

This is the only vulnerability fixable without breaking changes (`npm audit fix`).
It is also limited to the build toolchain and does not affect the runtime bundle.
It has been left in place to keep this PR narrowly scoped, but can be addressed
in a follow-up `npm audit fix` commit.

## Risk assessment

All critical and most high findings are in **dev-only** dependencies
(`happy-dom`, coverage tooling). They do not affect the production bundle
shipped to users. The `axios` vulnerabilities are runtime-reachable through
`@stellar/stellar-sdk` but are gated behind the user's own Stellar network
calls; exploitation requires a MITM or malicious server response, not
arbitrary code from an attacker. The `postcss` path-traversal is build-time only.

## Remediation path

| Action | Dependency | Scope |
|--------|-----------|-------|
| Follow-up PR: `npm audit fix` | `postcss` | build-time only |
| Tracked issue: upgrade `@stellar/stellar-sdk` to `^16.1.0` | `axios` chain | runtime |
| Tracked issue: upgrade `@vitest/coverage-v8` to `^4.1.10` | `brace-expansion` chain | dev only |
| Tracked issue: upgrade `happy-dom` to `^20.11.1` | `happy-dom` | dev only |
