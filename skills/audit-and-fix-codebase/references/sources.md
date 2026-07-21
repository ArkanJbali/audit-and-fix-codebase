# Sources

Central bibliography for `audit-and-fix-codebase`. Other reference files use
source names instead of inline URLs so guidance stays readable.

Read this file only when you need to verify or cite a source behind a finding.

## Contents

- Security references
- AI code-generation failure modes
- Dependency / supply-chain
- Coding-agent behavior

## Security References

- **OWASP Top 10 2021**: https://owasp.org/www-project-top-ten/
- **OWASP Cheat Sheet Series**: https://cheatsheetseries.owasp.org/
- **CWE Top 25 Most Dangerous Software Weaknesses**: https://cwe.mitre.org/top25/
- **OWASP ASVS (Application Security Verification Standard)**: https://owasp.org/www-project-application-security-verification-standard/
- **OWASP Dependency-Check**: https://owasp.org/www-project-dependency-check/
- **NIST SP 800-53 (security controls)**: https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final
- **Snyk vulnerability database**: https://security.snyk.io/
- **GitHub Advisory Database**: https://github.com/advisories
- **NPM Audit documentation**: https://docs.npmjs.com/cli/v11/commands/npm-audit
- **pip-audit (PyPA)**: https://github.com/pypa/pip-audit
- **dotnet list package --vulnerable**: https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-list-package
- **cargo-audit (RustSec)**: https://github.com/rustsec/rustsec/tree/main/cargo-audit
- **composer audit**: https://getcomposer.org/doc/03-cli.md#audit
- **govulncheck**: https://go.dev/security/vulncheck/
- **bundler-audit**: https://github.com/rubysec/bundler-audit

## AI Code-Generation Failure Modes

- **GitClear, *Code Quality Report* (2024–2025)** — measured growth in code
  duplication and function size / cyclomatic complexity in AI-assisted commits.
- **Spracklen et al., "The Use of LLMs for Code Generation:
  Hallucinated Package Vulnerabilities" (USENIX Security '25)** — measured rate
  of hallucinated package / API references across LLM models.
- **C. Le et al., arXiv:2409.19182** — trust-boundary reasoning for generated
  code (validate at boundaries, trust contracts inside).
- **L. Karpathy** — commentary on broad catch-all error swallowing in LLM-generated
  code.
- **M. Fowler, *Patterns for Reducing Friction* — on agents declaring success
  while tests fail and returning hardcoded fixture values.
- **Sandi Metz, "The Wrong Abstraction" (2016)** — duplication is cheaper than
  the wrong abstraction; guidance on when to re-inline.
- **McCabe, "A Complexity Measure" (NIST 235r)** — cyclomatic complexity as a
  structural quality metric.

## Dependency / Supply-Chain

- `npm audit` — npm / Node.js vulnerable dependency scanner (docs.npmjs.com).
- `pip-audit` — PyPA's Python dependency scanner (github.com/pypa/pip-audit).
- `dotnet list package --vulnerable` — .NET vulnerable dependency listing
  (learn.microsoft.com).
- `cargo audit` — Rust advisory scanner (RustSec Advisory Database).
- `composer audit` — PHP / Composer advisory scanner.
- `govulncheck` — Go vulnerability checker (Vulncheck / Go team).
- `bundler-audit` — Ruby / Bundler vulnerability scanner (rubysec).
- `mix_audit` — Elixir / Mix dependency auditor (hex.pm).

## Coding-Agent Behavior

- **Claude Code issue #6984** — agents disabling / weakening tests to make them pass.
- **M. Fowler, *Patterns for Reducing Friction in AI-Assisted Development* —
  agents declaring success despite failing tests.
- **Fowler, *Refactoring* (2nd ed.)** — definitional source for refactoring
  discipline (same inputs, same outputs, same errors; never bundle with
  bug fixes).

This file is a bibliography, not a checklist. No source here is required reading
to run the audit; the SKILL.md is self-contained.
