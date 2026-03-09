# Security Audit Report

**Project:** [PROJECT_NAME]
**Date:** [AUDIT_DATE]
**Auditor:** agent-rock v[VERSION] (Claude Code Security Audit Skill)
**Scope:** [TARGET_DIRECTORY]
**Audit Mode:** [quick | deep]
**Tech Stack:** [DETECTED_TECH_STACK]

---

## Executive Summary

[2-3 paragraph summary: what was audited, overall security posture assessment, key risk areas, and top priority recommendations. Written for a non-technical audience.]

### Risk Score

Confirmed findings only. Hardening observations are listed separately below and should not be counted as vulnerabilities.

| Severity | Count |
|----------|-------|
| Critical | [N] |
| High     | [N] |
| Medium   | [N] |
| Low      | [N] |
| Info     | [N] |
| **Total** | **[N]** |

### Overall Assessment: [CRITICAL / POOR / FAIR / GOOD / EXCELLENT]

---

## Findings Overview

| # | Title | Severity | Category | Location |
|---|-------|----------|----------|----------|
| 1 | [Finding title] | [Severity] | [Category] | `[file:line]` |
| 2 | [Finding title] | [Severity] | [Category] | `[file:line]` |

---

## Detailed Findings

### [F-001] [Finding Title]

| Field | Value |
|-------|-------|
| **Severity** | [Critical / High / Medium / Low / Info] |
| **Confidence** | [High / Medium / Low] |
| **Category** | [Category name] |
| **Location** | `[file_path:line_number]` |
| **Related Locations** | `[secondary_file:line]` or `None` |
| **CWE** | [CWE-ID: CWE Name] |

**Description:**

[Clear explanation of what the vulnerability is and why it matters.]

**Evidence:**

[Show only the verified code or config that proves the issue. If the finding is about a missing control, explain the affected entry point and what guard is absent there.]

```[language]
// file_path:line_number
[The actual vulnerable code snippet from the codebase]
```

**Impact:**

[What an attacker could achieve by exploiting this vulnerability.]

**Remediation:**

[Specific steps to fix the verified root cause. Keep this recommendation tightly scoped to the evidence above.]

```[language]
// Recommended fix
[Corrected code example]
```

---

<!-- Repeat the Detailed Findings block for each finding, ordered by severity (Critical first) -->

---

## Hardening Observations

These are evidence-backed defense-in-depth or configuration notes that did not meet the threshold for a confirmed exploit-ready finding.
Do not count them in the severity totals above.

| # | Title | Confidence | Category | Location |
|---|-------|------------|----------|----------|
| H-001 | [Observation title] | [High / Medium / Low] | [Category] | `[file:line]` |

For each observation, briefly document:

- why it was not promoted to a confirmed finding
- what code or config was reviewed
- the narrow hardening action to take

---

## Categories Without Findings

The following categories were audited with no issues found:

- [List categories that passed clean]

---

## Recommendations Priority Matrix

| Priority | Timeline | Action | Effort | Impact |
|----------|----------|--------|--------|--------|
| 1 - Immediate | Fix now | [Action item] | [Low / Medium / High] | [Impact description] |
| 2 - Short-term | Within 1 week | [Action item] | [Low / Medium / High] | [Impact description] |
| 3 - Medium-term | Within 1 month | [Action item] | [Low / Medium / High] | [Impact description] |
| 4 - Long-term | Within quarter | [Action item] | [Low / Medium / High] | [Impact description] |

---

## Compliance Reference

Where applicable, findings are mapped to relevant compliance frameworks:

| Finding | OWASP | CWE | PCI-DSS | GDPR | SOC 2 |
|---------|-------|-----|---------|------|-------|
| [F-001] | [A01] | [CWE-ID] | [Req #] | [Article #] | [CC #] |

*Note: This mapping is advisory. Consult your compliance team for authoritative assessments.*

---

## Methodology

This audit was performed as a static code analysis covering:

1. **Discovery**: Automated tech stack detection and attack surface mapping
2. **Analysis**: Systematic scanning across 8 security categories using pattern matching, code review, and dependency auditing
3. **Verification**: Each potential finding was verified by reading surrounding code context to eliminate false positives
4. **Classification**: Confirmed findings rated using severity criteria aligned with CVSS qualitative ratings

### Confidence Levels

- **High**: Direct evidence in code or configuration with clear exploit conditions
- **Medium**: Strong evidence with limited runtime assumptions
- **Low**: Evidence-backed concern with important runtime assumptions still inferred

### Output Classes

- **Confirmed Findings**: Verified vulnerabilities with enough evidence to justify severity, impact, and remediation.
- **Hardening Observations**: Real defense-in-depth or configuration concerns that did not meet the bar for a confirmed exploit-ready finding.

### Categories Audited

1. OWASP Top 10:2025
2. Authentication & Authorization
3. Data Exposure & Sensitive Data Handling
4. Dependency Vulnerabilities
5. Configuration Security
6. API Security
7. Input Validation & Output Encoding
8. Cryptographic Issues

---

## Disclaimer

This report was generated by an AI-powered static analysis tool (agent-rock). While thorough, it may contain false positives or miss vulnerabilities that require runtime analysis, dynamic testing, or deep business logic understanding. This audit supplements but does not replace a professional penetration test. Findings should be validated by a security professional before taking action on production systems.

### Known Limitations

- Runtime-only behaviors may not be visible from static code and config alone.
- Business logic flaws may require domain context or dynamic testing to confirm.
- Controls enforced outside the repository, such as edge proxies or managed platform settings, may not be fully observable.

---

*Generated by [agent-rock](https://github.com/KadirHarmanc/agent-rock) v[VERSION] — Open-source security audit skill for Claude Code*
