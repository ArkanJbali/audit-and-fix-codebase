# Sources

External references the audit skill draws on for severity reasoning and
AI-specific failure modes. Read this file only when you need to verify or cite
a source behind a finding.

## AI code-generation failure modes

- GitClear, *Code Quality Report* (2024–2025) — measured growth in code
  duplication and function size / cyclomatic complexity in AI-assisted commits.
- Spracklen et al., "The Use of LLMs for Code Generation:
  Hallucinated Package Vulnerabilities" (USENIX Security '25) — measured rate
  of hallucinated package / API references across LLM models.
- C. Le et al., arXiv:2409.19182 — trust-boundary reasoning for generated code
  (validate at boundaries, trust contracts inside).
- L. Karpathy — commentary on broad catch-all error swallowing in LLM-generated
  code.
- M. Fowler, *Patterns for Reducing Friction* — on agents declaring success
  while tests fail and returning hardcoded fixture values.

## Dependency / supply-chain

- `npm audit` — npm / Node.js vulnerable dependency scanner.
- `pip-audit` — PyPA's Python dependency scanner.
- `dotnet list package --vulnerable` — .NET vulnerable dependency listing.
- `cargo audit` — Rust advisory scanner (RustSec Advisory Database).
- `composer audit` — PHP / Composer advisory scanner.
- `govulncheck` — Go vulnerability checker (Vulncheck / Go team).

## Coding-agent behavior

- Claude Code issue #6984 — agents disabling / weakening tests to make them pass.

This file is a bibliography, not a checklist. No source here is required reading
to run the audit; the SKILL.md is self-contained.
