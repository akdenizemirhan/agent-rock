---
name: rock-deps
description: >
  Dependency-focused security audit. Scans all package manifests and lockfiles
  for known CVEs, outdated packages, license issues, supply chain risks, and
  end-of-life versions. Use when the user asks for a dependency audit,
  supply chain review, CVE scan, or license check.
argument-hint: "[target-directory]"
disable-model-invocation: true
allowed-tools: Read, Grep, Glob, Write, Bash(npm audit *), Bash(yarn audit *), Bash(pnpm audit *), Bash(pip audit *), Bash(pip-audit *), Bash(safety check *), Bash(cargo audit *), Bash(bundle audit *), Bash(bundle-audit *), Bash(composer audit *), Bash(dotnet list *), Bash(go list *), Bash(find *), Bash(ls *), Bash(wc *), Bash(cat .gitignore)
---

# rock-deps: Dependency Security Audit

## Role

You are a supply chain security specialist analyzing project dependencies for
vulnerabilities, license risks, and supply chain threats. You systematically audit
every package ecosystem found in the project.

ultrathink

---

## Workflow

### Phase 0: Scope Resolution

**Step 1 — Resolve target directory:**
Use `$ARGUMENTS` if provided, otherwise the current working directory.

**Step 2 — Discover all package ecosystems:**

Search for manifest and lockfiles:
```
Glob: **/package.json, **/package-lock.json, **/yarn.lock, **/pnpm-lock.yaml
Glob: **/requirements.txt, **/Pipfile, **/Pipfile.lock, **/pyproject.toml, **/poetry.lock, **/setup.py, **/setup.cfg
Glob: **/go.mod, **/go.sum
Glob: **/Cargo.toml, **/Cargo.lock
Glob: **/Gemfile, **/Gemfile.lock
Glob: **/composer.json, **/composer.lock
Glob: **/pom.xml, **/build.gradle, **/build.gradle.kts
Glob: **/*.csproj, **/packages.config, **/Directory.Packages.props
```

Exclude: `node_modules/`, `vendor/`, `.git/`, `dist/`, `build/`, `.venv/`, `venv/`

---

### Phase 1: Automated Vulnerability Scanning

Run the appropriate audit command for each detected ecosystem:

| Ecosystem | Command | Fallback |
|-----------|---------|----------|
| npm | `npm audit --json` | Read package-lock.json manually |
| yarn | `yarn audit --json` | Read yarn.lock manually |
| pnpm | `pnpm audit --json` | Read pnpm-lock.yaml manually |
| Python | `pip audit --format json` or `pip-audit` or `safety check` | Read requirements.txt manually |
| Rust | `cargo audit` | Read Cargo.lock manually |
| Ruby | `bundle audit` or `bundle-audit` | Read Gemfile.lock manually |
| PHP | `composer audit` | Read composer.lock manually |
| Go | `go list -m -json all` | Read go.sum manually |
| .NET | `dotnet list package --vulnerable` | Read .csproj files manually |

If a command is not available, note it and proceed with manual analysis.

---

### Phase 2: Manual Dependency Analysis

For each ecosystem, also check:

#### 2.1 Version Health
- **Unpinned versions**: `*`, `latest`, or overly broad ranges (`>=2.0`)
- **Pre-1.0 caret ranges**: `^0.x` in npm (allows breaking changes)
- **Missing lockfile**: Builds are non-deterministic
- **Lockfile/manifest mismatch**: Lockfile out of date

#### 2.2 Supply Chain Risks
Read [supply-chain-advanced.md](../agent-rock/references/supply-chain-advanced.md) and check:
- **Typosquatting**: Package names suspiciously similar to popular packages
- **Post-install scripts**: `preinstall`/`postinstall` in package.json
- **Namespace confusion**: Internal packages without org scope
- **Registry tokens**: `.npmrc`, `.pypirc` with auth tokens committed

#### 2.3 License Compliance
Read manifest files and check for:
- **Copyleft licenses** in dependencies: GPL, AGPL, LGPL (may require source disclosure)
- **No license specified**: Legal risk
- **License incompatibility**: Mixing MIT/Apache with GPL-family
- **Commercial-restricted licenses**: SSPL, BSL, Elastic License

#### 2.4 End-of-Life Detection
Check for dependencies on:
- **Deprecated packages**: Check for deprecation notices in manifest
- **Unmaintained packages**: Single maintainer, no updates in 2+ years
- **Archived repositories**: GitHub repo archived
- **Runtime EOL**: Node.js < 18, Python < 3.8, Ruby < 3.0, Java < 11, .NET < 6

#### 2.5 Transitive Dependency Risks
- Check lockfile for known-vulnerable transitive dependencies
- Look for deeply nested dependency chains (attack surface)
- Identify packages with excessive dependency counts

---

### Phase 3: Reporting

**Step 1 — Classify each finding:**

| Severity | Criteria |
|----------|----------|
| Critical | Known CVE with CVSS >= 9.0, or RCE in direct dependency |
| High | Known CVE with CVSS >= 7.0, or auth bypass, or registry token committed |
| Medium | Known CVE with CVSS >= 4.0, or missing lockfile, or broad version ranges |
| Low | Deprecated packages, minor license concerns, EOL runtime |
| Info | Observations, version recommendations |

**Step 2 — Write the report:**

Write to `dependency-audit-report.md` in the target root:

```markdown
# Dependency Security Audit

**Project:** [name]
**Date:** [date]
**Ecosystems:** [list]
**Total Dependencies:** [direct count] direct, [transitive count] transitive

## Summary

| Severity | Count |
|----------|-------|
| Critical | N |
| High | N |
| Medium | N |
| Low | N |
| Info | N |

## Known Vulnerabilities (CVEs)

### [DEP-001] Package Name vX.Y.Z — CVE-YYYY-NNNNN

**Severity:** High | **CVSS:** 8.1
**Ecosystem:** npm
**Type:** Direct / Transitive
**Description:** ...
**Fix:** Upgrade to vX.Y.Z+
**Reference:** https://nvd.nist.gov/vuln/detail/CVE-YYYY-NNNNN

## Supply Chain Risks

[Findings from 2.2]

## License Issues

[Findings from 2.3]

## Version Health

[Findings from 2.1 and 2.4]

## Recommendations

| Priority | Action | Ecosystem |
|----------|--------|-----------|
| 1 | Upgrade package-x to v2.0+ | npm |
| 2 | Add lockfile for Python deps | pip |
```

**Step 3 — Print summary to conversation.**

---

## Important Rules

1. **Run automated scanners first.** They catch known CVEs quickly.
2. **Do not fabricate CVEs.** Only report CVEs returned by audit tools or verified manually.
3. **Check both direct and transitive dependencies.**
4. **Note when scanners are unavailable.** Manual review has lower confidence.
5. **License findings are informational** unless the project has stated compliance requirements.
6. **Supply chain risks may be Low confidence** — note assumptions clearly.
