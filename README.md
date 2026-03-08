# agent-rock

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![GitHub stars](https://img.shields.io/github/stars/akdenizemirhan/agent-rock?style=social)](https://github.com/akdenizemirhan/agent-rock/stargazers)
[![GitHub issues](https://img.shields.io/github/issues/akdenizemirhan/agent-rock)](https://github.com/akdenizemirhan/agent-rock/issues)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blueviolet)](https://claude.ai)

**Deep security audit skill for Claude Code.**

agent-rock is an open-source Claude Code skill that performs thorough static security analysis of any codebase. It thinks like a penetration tester with full source code access, systematically scanning across 8 security categories and producing a professional Markdown report with severity-rated, evidence-backed findings.

## Features

- **Auto-detects tech stack** — Works with JS/TS, Python, Java, Go, Ruby, PHP, C#, Rust, C/C++
- **Framework-aware guidance** — Loads focused heuristics for Express, Django, Spring, Rails, and Laravel
- **Quick and deep modes** — Supports fast triage scans and more exhaustive deep audits
- **OWASP Top 10:2025** — Scans against the latest OWASP edition including new categories
- **8 security categories** — Comprehensive coverage from injection to cryptography
- **Evidence-backed findings** — Every finding includes file path, line number, verified evidence, and confidence
- **Dual output** — Generates both Markdown and JSON reports for humans and tooling
- **Consistent finding schema** — Normalizes IDs, confidence, CWE mapping, and ordering across both outputs
- **Professional report** — Structured output with executive summary, severity ratings, confidence, and remediation guidance
- **Zero dependencies** — Uses only Claude Code built-in tools, nothing to install
- **Works on any codebase** — No configuration required

## Installation

### Option 1: Clone into your project (project-scoped)

```bash
git clone https://github.com/emirhanakdeniz/agent-rock.git /tmp/agent-rock
mkdir -p .claude/skills
cp -r /tmp/agent-rock/.claude/skills/agent-rock .claude/skills/
rm -rf /tmp/agent-rock
```

### Option 2: Install as personal skill (available in all projects)

```bash
git clone https://github.com/emirhanakdeniz/agent-rock.git /tmp/agent-rock
mkdir -p ~/.claude/skills
cp -r /tmp/agent-rock/.claude/skills/agent-rock ~/.claude/skills/
rm -rf /tmp/agent-rock
```

## Usage

In Claude Code, invoke the skill:

```
/agent-rock
```

Use the convenience wrappers:

```
/rock-quick
/rock-deep
```

If you run the command without arguments, it scans the current working directory.

Typical usage from the repo root:

```
/rock-quick
/rock-deep
```

Scan a specific subdirectory only when you need to narrow the scope:

```
/rock-quick ./src
/rock-deep ./src
/agent-rock ./src deep
```

## Output

The skill generates `security-audit-report.md` and `security-audit-report.json` in the scanned directory root containing:

- **Executive Summary** — Overall risk assessment for non-technical stakeholders
- **Risk Score** — Finding counts by severity (Critical, High, Medium, Low, Info)
- **Confidence** — A confidence level for each finding based on evidence quality
- **Findings Table** — Quick overview of all issues found
- **Detailed Findings** — Each with severity, location, evidence, impact, and remediation
- **Priority Matrix** — Actionable remediation roadmap
- **JSON Companion Report** — Machine-readable findings for automation and CI tooling
- **Methodology** — How the audit was conducted

## Categories Scanned

| # | Category | What It Checks |
|---|----------|---------------|
| 1 | OWASP Top 10:2025 | All 10 categories including new Supply Chain and Exceptional Conditions |
| 2 | Authentication & Authorization | Credential handling, session management, access control |
| 3 | Data Exposure | PII leaks, sensitive data in logs/URLs, missing encryption |
| 4 | Dependencies | Known CVEs, outdated packages, supply chain risks |
| 5 | Configuration | Secrets in code, debug modes, missing security headers |
| 6 | API Security | Missing auth, rate limiting, mass assignment, IDOR |
| 7 | Input Validation | Injection sinks, XSS vectors, command injection, deserialization |
| 8 | Cryptography | Weak algorithms, hardcoded keys, insecure randomness |

## How It Works

1. **Discovery** — Detects languages, frameworks, and maps the application architecture
2. **Analysis** — Systematically scans across all 8 categories using pattern matching and code review
3. **Verification** — Reads surrounding code context to eliminate false positives
4. **Reporting** — Generates a structured report with evidence-backed findings

## Important Notes

- This is a **static analysis** tool. It supplements but does not replace dynamic testing or professional penetration testing.
- Findings should be **validated by a security professional** before taking action on production systems.
- The skill **never fabricates findings** — every reported issue references real code in your codebase.
- It **skips dependency directories** (node_modules, vendor, etc.) to focus on your application code.

## Known Limitations

- Runtime-only vulnerabilities may not be visible from source code and configuration alone.
- Business logic flaws often need domain context or dynamic testing to confirm.
- Some controls may be enforced outside the repository, such as edge proxies, WAFs, or managed platform settings.

## Contributing

Contributions are welcome! Areas where help is needed:

- Adding vulnerability patterns for additional languages/frameworks
- Improving false-positive reduction heuristics
- Adding CWE mappings for more vulnerability types
- Testing against real-world codebases and reporting accuracy
- Translations of the report template

## License

MIT
