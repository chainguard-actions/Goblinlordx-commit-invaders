<!-- markdownlint-disable -->

# Hardening Report: Goblinlordx--commit-invaders/v1.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Goblinlordx--commit-invaders/v1.2.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable tag or branch refs instead of pinned 40-character SHA commit hashes. This exposes the workflow to supply-chain attacks if the referenced action is compromised or its tag is moved.

- ci.yml: actions/checkout@v6, pnpm/action-setup@v5, actions/setup-node@v6
- commit-invaders.yml: actions/checkout@v6, Goblinlordx/commit-invaders@main (branch ref — especially risky)
- generate-example.yml: actions/checkout@v6, pnpm/action-setup@v5, actions/setup-node@v6
- release.yml: actions/checkout@v6, pnpm/action-setup@v5, actions/setup-node@v6, softprops/action-gh-release@v2

Locations:

- `.github/workflows/ci.yml:13`
- `.github/workflows/ci.yml:14`
- `.github/workflows/ci.yml:16`
- `.github/workflows/commit-invaders.yml:14`
- `.github/workflows/commit-invaders.yml:16`
- `.github/workflows/generate-example.yml:13`
- `.github/workflows/generate-example.yml:14`
- `.github/workflows/generate-example.yml:16`
- `.github/workflows/release.yml:14`
- `.github/workflows/release.yml:15`
- `.github/workflows/release.yml:17`
- `.github/workflows/release.yml:31`

### missing-permissions (severity: medium)

ci.yml has no top-level permissions: key and its only job ('test') also has no job-level permissions: key. Without explicit permissions, the workflow inherits the default repository permissions, which may be broader than necessary (e.g. write access to contents). A minimal permissions block such as `permissions: read-all` or specific scopes should be added.

Locations:

- `.github/workflows/ci.yml:1`

### script-injection (severity: high)

Rule (a) violation: generate-example.yml line 24 directly interpolates a ${{ }} expression inside a run: shell command string: `run: GITHUB_TOKEN=${{ secrets.GITHUB_TOKEN }} npx tsx scripts/generate.ts Goblinlordx commit-invaders.svg`. Any ${{ ... }} expression interpolated directly into a run: block is processed by the GitHub Actions template engine before the shell sees it, making it a script-injection risk. The safe pattern is to pass the value via an env: block and reference it as an environment variable (e.g. `env: GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}` and then use `$GITHUB_TOKEN` in the run: script).

Locations:

- `.github/workflows/generate-example.yml:24`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across 4 workflow files:

1. unpinned-uses: Pinned all action references to full 40-char SHAs with tag comments preserved:
   - actions/checkout@v6 → df4cb1c069e1874edd31b4311f1884172cec0e10
   - pnpm/action-setup@v5 → fc06bc1257f339d1d5d8b3a19a8cae5388b55320
   - actions/setup-node@v6 → 249970729cb0ef3589644e2896645e5dc5ba9c38
   - Goblinlordx/commit-invaders@main → bffc050fb921f841f03b6b210b3b3a5bcec102c3
   - softprops/action-gh-release@v2 → 3bb12739c298aeb8a4eeaf626c5b8d85266b0e65

2. missing-permissions: Added top-level `permissions: contents: read` to ci.yml (the test job only needs to read the repo).

3. script-injection: In generate-example.yml, moved `${{ secrets.GITHUB_TOKEN }}` from the inline `run:` shell string into an `env:` block (`GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}`), so the shell script references it as a plain environment variable rather than having it interpolated by the template engine.

