# audit-and-fix-codebase

[![skills.sh](https://skills.sh/b/arkanjbali/audit-and-fix-codebase)](https://skills.sh/arkanjbali/audit-and-fix-codebase)

A focused, **two-pass audit skill** for coding agents: it reports security issues, vulnerable dependencies, duplicated logic, and obvious cleanups as a prioritized list — without changing anything — then, once the user approves, fixes them in a safe order (security first, dependencies next, cleanups last), checking the app still builds and runs after each step.

Stack-agnostic (works on JS/TS, Python, .NET, Go, Rust, PHP, Ruby, Java, …) and tool-agnostic (Claude Code, opencode, Codex, Cursor, Continue, or any agent that reads `SKILL.md`). It is a portable instruction skill: no MCP server, no bundled scripts, no network calls, no credentials. Nothing changes without the user's say-so.

## Install

Browse the package first:

```bash
npx skills add ArkanJbali/audit-and-fix-codebase --list
```

Install the package:

```bash
npx skills add ArkanJbali/audit-and-fix-codebase
```

Or install the one skill:

```bash
npx skills add ArkanJbali/audit-and-fix-codebase --skill audit-and-fix-codebase
```

Install for a specific agent:

```bash
npx skills add ArkanJbali/audit-and-fix-codebase --skill audit-and-fix-codebase --agent codex
npx skills add ArkanJbali/audit-and-fix-codebase --skill audit-and-fix-codebase --agent claude-code
npx skills add ArkanJbali/audit-and-fix-codebase --skill '*' --agent cursor
```

Install globally:

```bash
npx skills add ArkanJbali/audit-and-fix-codebase --global
```

Works with Claude Code, Codex, Cursor, OpenCode, and other supported agents via the [Skills CLI](https://github.com/vercel-labs/skills).

## Updating

Skills install as a copy, so a new version here does not reach your agent until you update. Refresh them with the Skills CLI:

```bash
npx skills update                 # all installed skills (alias: upgrade)
npx skills update audit-and-fix-codebase
```

Add `--global` for global installs or `--project` for project installs. Re-running `npx skills add ArkanJbali/audit-and-fix-codebase` also re-fetches the latest.

## How to use it

Invoke the skill after asking your agent to audit a whole codebase, then approve the fix list:

```text
Use $audit-and-fix-codebase to audit my whole codebase for security, dependencies, and cleanups; report findings first, then fix after I approve.
```

Or simply type the agent trigger:

```text
audit my codebase and fix what's safe — report findings first, then fix after I approve.
```

Workflow:

1. Open your project in your AI coding tool.
2. **Commit your work first** so you can roll back to a clean tree (the skill also offers a checkpoint commit before Pass 2).
3. Let the skill run **Pass 1** (the findings report). Review it.
4. Answer **"which groups/items do you fix?"** The skill then runs **Pass 2** for approved items only, in order: security → dependencies → safe cleanups, with a build/test/app check and a checkpoint commit after each group.

No tests? It will say so plainly, lean on "still builds and runs" as the safety net, and never claim a change is "safe" or "functionally equivalent" without one.

## Which version to run

| Phase | What it does | Catches |
| --- | --- | --- |
| Pass 1 | Reads the whole codebase, runs the package manager's real vulnerability scanner (`npm audit` / `pip-audit` / `cargo audit` / `composer audit` / `govulncheck` / `dotnet list package --vulnerable`), and reports findings as a prioritized table — without editing anything | Hardcoded secrets, injection (SQL/XSS/eval/SSRF), missing/weak auth (IDOR, mass assignment), sensitive data exposure, unsafe CORS & debug config, vulnerable & unused/outdated/dead dependencies, duplicated logic, obvious refactors, quick health checks (missing error handling, N+1, missing pagination, swallowed exceptions, resource leaks) |
| Pass 2 | Fixes approved items in a safe order with a verification checkpoint after each group | Moves secrets to env vars (+ `.env.example`, + remind to **rotate** ever-committed secrets), bumps patch/minor dependencies (majors left flagged), removes duplication, dead code, and does safe refactors with before/after — **behavior must not change** |

Pair it with adjacent guard skills if your runtime has them: a `clean-code-guard` on Pass 2 implementation diffs and a `test-guard` on any tests the audit adds. The SKILL.md does this on its own output before presenting.

## The skill

### audit-and-fix-codebase

Acts as a senior software engineer and security reviewer and audits the user's whole codebase, then fixes only what is safe to fix. It is deliberately practical — no over-engineering, no abstractions the user did not ask for, no rewriting of working code just to make it "cleaner." Two passes: Pass 1 reports and changes nothing; Pass 2 fixes only after explicit approval, in a safe order, with a verification checkpoint after each group and a checkpoint commit (`audit: <group>`) so any group can be rolled back alone.

Why the AI layer matters: it does not recite CVEs from memory — it runs the real scanner for the package manager and reports the actual output. It reads code before flagging it. It asks before touching business logic. And for anything ambiguous or too risky (a breaking upgrade, a big rewrite, an ever-committed secret), it gives a recommendation, not an implementation.

**You'll feel it when:** an audit produces prioritized, source-backed findings the user actually approves, secrets get moved to env vars plus a rotation reminder, dependencies bump patch/minor without surprise majors, and safe cleanups come with before/after and leave behavior untouched.

## When NOT to use this skill

- Reviewing a single diff or PR — use a `clean-code-guard` instead.
- Reviewing tests — use a `test-guard` instead.
- Reviewing documentation — use a `docs-guard` instead.
- Implementing a requested feature or making a targeted bug fix — just do that; this skill is an audit, not a feature pipeline.
- Factual/conceptual questions, CI/tooling config, pure architecture discussion, or running/debugging tests — answer directly.

## How this differs

This is not a reactor-time review gate for a single diff (that is what `clean-code-guard`, `test-guard`, and `docs-guard` are for) and not a platform-specific catalog. `audit-and-fix-codebase` is broader and works on a *whole project*: it gives the agent a disciplined, approval-gated audit-and-fix flow that catches common AI failure modes (hallucinated CVEs, silent behavior changes, hardcoded "success" returns, swallowed errors) while staying out of business logic the user did not ask it to touch.

## Repository shape

The skill is a folder with a `SKILL.md` entrypoint, lightweight agent metadata, and progressive-disclosure references:

```text
skills/
└── audit-and-fix-codebase/
    ├── SKILL.md
    ├── agents/openai.yaml
    └── references/
        ├── security-checklist.md
        ├── dependency-scanners.md
        ├── duplication-and-refactors.md
        ├── review-checklist.md
        └── sources.md
```

The `SKILL.md` stays small so it loads cheaply; deeper guidance for each audit section lives in the `references/` directory and loads only when the task needs it.

The skill content is pure-instruction Markdown plus lightweight `agents/openai.yaml` metadata — it needs no MCP server, network access, API key, shell command, local executable, or bundled script. `agents/openai.yaml` is lightweight display metadata (`display_name`, `short_description`, `default_prompt`) for Codex discovery.

## Trust and validation

This package is intentionally inspectable:

- Skill content is Markdown plus lightweight `agents/openai.yaml` metadata.
- There are no executable scripts, network calls, MCP server dependencies, or credentials.
- External source URLs live in the skill's `references/sources.md`.

Maintainer checks before publishing:

```bash
npx skills add . --list --full-depth
```

This lists every skill the CLI discovers by scanning `skills/`, with its references, so you can confirm structure and discovery before publishing.

## Files

| File | Purpose |
| --- | --- |
| `skills/audit-and-fix-codebase/SKILL.md` | The skill body. Tools that read `SKILL.md` (Claude Code, opencode, Cursor, Continue) load this automatically. |
| `skills/audit-and-fix-codebase/agents/openai.yaml` | Lightweight Codex/discovery metadata (`display_name`, `short_description`, `default_prompt`). |
| `skills/audit-and-fix-codebase/references/security-checklist.md` | Detailed security audit checklist — secrets, injection, auth, exposure, config, severity rubric. |
| `skills/audit-and-fix-codebase/references/dependency-scanners.md` | Exact scanner commands for each package manager (npm, pip, dotnet, cargo, composer, govulncheck, bundler, mix). |
| `skills/audit-and-fix-codebase/references/duplication-and-refactors.md` | Knowledge vs text duplication guidance, dead code detection, complexity heuristics, refactoring safety rules. |
| `skills/audit-and-fix-codebase/references/review-checklist.md` | Structured walk-through for Pass 1 and Pass 2: output format, severity rubric, verification gates, final summary template. |
| `skills/audit-and-fix-codebase/references/sources.md` | Bibliography for source URLs the skill cites when reasoning about severity. |

## License

MIT — see [LICENSE](LICENSE).
