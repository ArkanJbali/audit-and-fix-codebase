# Audit & Fix My Codebase

A practical, **two-pass** audit skill for any project, in any AI coding tool.

> First it reports — security issues, vulnerable dependencies, duplicated logic, and obvious cleanups — as a prioritized list, without changing anything. Then, once you approve, it fixes in a safe order: **security first, dependencies next, cleanups last**, checking the app still runs after each step.

- Stack-agnostic — works on JS/TS, Python, .NET, Go, Rust, PHP, Ruby, Java, etc.
- Tool-agnostic — runs in Claude Code, opencode, Codex, Cursor, Continue, or any agent that reads a `SKILL.md`.
- No backend, no scripts, no network calls — it's a portable instruction skill.
- Nothing changes without your say-so.

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | The skill body. Tools that read `SKILL.md` (Claude Code, opencode, Cursor, Continue) load this automatically. |
| `agents/openai.yaml` | Lightweight Codex/discovery metadata (display name, short description, default prompt). |

## Install

### Claude Code / opencode / any tool that loads skills from a directory

Copy (or symlink) this folder into your global skills directory:

- **Windows**: `%USERPROFILE%\.claude\skills\audit-and-fix-codebase\`
- **macOS / Linux**: `~/.claude/skills/audit-and-fix-codebase/`

That's it. The next time you say *"audit my codebase,"* the skill activates.

### Codex

Copy the folder into your Codex skills directory, then point Codex at it. `agents/openai.yaml` provides the `display_name`, `short_description`, and `default_prompt` Codex expects.

### Cursor / Continue / other SKILL.md readers

Drop the folder wherever your tool looks for skills. Only `SKILL.md` is required; `agents/openai.yaml` is ignored by these tools and harmless.

### skill.sh / portable install

The skill follows the [skill.sh](https://skill.sh) portable-skill layout: a `SKILL.md` with YAML frontmatter (`name`, `description`) and an optional `agents/openai.yaml` for Codex discovery. To install from a URL:

```bash
# example: clone into your skills dir
git clone <this-skill-repo-url> ~/.claude/skills/audit-and-fix-codebase
```

Because it's a pure-instruction skill (no bundled scripts, no MCP server), there's nothing to build, no dependencies to install, and no permissions to grant.

## How to use it

1. Open your project in your AI coding tool.
2. **Commit your work first** so you can roll back to a clean tree.
3. Paste the default prompt (or just say *"audit my codebase"*):
   > Audit my codebase and fix what's safe to fix. Report findings first; fix only after I approve.
4. Review **Pass 1** (the findings report) before approving.
5. No tests? It'll tell you, and lean on "still builds and runs" as the safety net. Built for results without over-engineering — nothing changes without your say-so.

## What it does

**Pass 1 — Find and report (no code changes)**

- Stack detection + test detection + baseline-build/run check before anything.
- Security: secrets, injection, missing auth, sensitive data exposure, unsafe CORS/config.
- Dependencies: version diff + real vulnerability scan for your package manager (`npm audit` / `pip-audit` / `cargo audit` / `composer audit` / `govulncheck` / `dotnet list package --vulnerable` …) — never recited from memory.
- Duplicated logic, obvious refactors, reusable pieces, quick health checks (missing error handling, N+1 queries, missing pagination, swallowed exceptions).
- Ends with "top 5 by risk" + **"which items do I fix?"** and stops.

**Pass 2 — Fix (after you approve)**

- Security first — calls out exactly what behavior changes for each fix.
- Dependencies next — patch/minor bumped, lockfile updated; major upgrades left flagged with a one-line migration note.
- Safe cleanups last — before/after for each, behavior must not change.
- Re-runs the baseline build/test/run after each group; checkpoint commit (`audit: <group>`) so any group can be rolled back alone.

## Companion skills

If your runtime has these guards, the skill will run them on its own output before presenting:

- **clean-code-guard** — on Pass 2 implementation diffs.
- **test-guard** — on any tests it adds.

This skill explicitly stays out of: single-PR/diff review (use clean-code-guard), test review (use test-guard), docs review (use a docs guard), and feature implementation.
