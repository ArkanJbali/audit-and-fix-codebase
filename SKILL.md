---
name: audit-and-fix-codebase
description: "Use when the user asks to audit, health-check, security-review, or clean up a whole project or codebase — 'audit my code', 'find and fix issues', 'is my app secure', 'check my dependencies', 'clean up this project' — or wants issues found and fixed with approval before changes. Works on any stack and in any AI coding tool (Claude Code, opencode, Codex, Cursor, Continue, or any model that reads SKILL.md). DO NOT USE for reviewing a single diff/PR (use clean-code-guard), test review (use test-guard), docs review (use docs-guard), or implementing a new feature."
---

# Audit & Fix My Codebase

Act as a **senior software engineer and security reviewer**. Audit the user's codebase and fix what's safe to fix. Keep it practical — don't over-engineer, don't add abstractions the user didn't ask for, don't rewrite working code just to make it "cleaner."

Work in **two passes**. Pass 1 reports and changes nothing. Pass 2 fixes, only after the user approves, in a safe order with a verification checkpoint after each group. Nothing changes without the user's say-so.

This is a portable instruction skill. It needs no MCP server, no network access, no API key, no bundled script. It works in any runtime that reads `SKILL.md` (Claude Code, opencode, Codex) plus the lightweight `agents/openai.yaml` metadata for tool discovery. Normalize whatever tool you're running in to the verbs below: "edit a file" = your edit/apply tool, "run a command" = your shell/bash tool, "stop for approval" = pause and ask the user a question before continuing.

## Tool-agnostic operating notes

- If your runtime has guard skills for adjacent jobs, call them on your own output before presenting: a clean-code guard on Pass 2 diffs, a test guard on any tests you add.
- If you're an AI coding agent that auto-runs commands, still **stop and ask** at every marked checkpoint — never run a fix, commit, or app start without the user's explicit approval at that step.
- If you're a chat-only model with no file/shell access, run Pass 1 as a findings report only and tell the user you can't apply fixes from their environment — hand them the exact commands and diffs to run.

## Pre-flight (before Pass 1)

1. **Detect and announce the stack**: language(s), framework(s), package manager(s), entry points. In a monorepo, list each sub-project and confirm scope (all, or which ones) before auditing.
2. **Git safety net**: check `git status`. If there are uncommitted changes, ask the user to commit (or offer a checkpoint commit) before Pass 2. If not a repo, recommend `git init` + snapshot commit. Never start fixing on an unsaved working tree.
3. **Test detection**: say whether tests exist and how to run them. If there are none, say so plainly, do not claim any change is "safe" or "functionally equivalent," point out the riskiest changes, and suggest where one quick test would pay off before touching them. "Still builds and runs" becomes the safety net.
4. **Establish the baseline**: run the build, the test suite (if any), and start the app once BEFORE changing anything. Record the exact commands — these are the verification gate for every Pass 2 checkpoint. If the app doesn't build/run *today*, report that first and stop.
   - When you start the app, track its PID and stop **that specific process** when done. Never kill by image/process name (`taskkill /IM node.exe`, `pkill node`, `killall python`) — that terminates the user's unrelated processes and dev servers.
   - If scanning requires installing dependencies (e.g. `npm install` to get a lockfile for `npm audit`), disclose every non-source file it created in the report.

## PASS 1 — Find and report (no code changes)

Read code before flagging it — findings come from the source in front of you, not from how projects "usually" look. Every finding needs: **severity** (Critical/High/Medium/Low), **what** it is, **where** (file:line), **why** it matters, **suggested fix**. One table per section. Skip empty sections with one line ("no findings"). Do not edit anything in Pass 1.

1. **Security** (do this first — it's the priority)
   - Hardcoded secrets, API keys, tokens, passwords in code or committed config (`.env` in git, connection strings); also check whether committed files with secrets appear in git history.
   - Injection: SQL/command built by string concatenation, XSS (unescaped output, `dangerouslySetInnerHTML`, `innerHTML`), unsafe `eval`/dynamic code execution, path traversal on file endpoints, SSRF on user-supplied URLs, insecure deserialization.
   - Missing or broken auth: unprotected routes or actions, missing object-level ownership checks (IDOR), mass assignment / over-posting on create/update endpoints.
   - Sensitive data in logs, localStorage/sessionStorage, URLs, or error responses (stack traces to clients).
   - Config: overly open CORS, debug mode in production config, weak password hashing (MD5/SHA1/plain), missing rate limiting on login/OTP/reset endpoints.
2. **Dependencies**
   - Run the real scanner for the package manager (`npm audit` / `pip-audit` / `dotnet list package --vulnerable` / `cargo audit` / `composer audit` / `govulncheck`, etc.) and report its actual output — never recite CVEs from memory.
   - Table of direct packages: current vs latest; patch/minor bumps vs **major upgrades listed separately** with a one-line migration note.
   - Unused/dead dependencies; license red flags (copyleft or commercial licenses in a commercial codebase) — flag, don't adjudicate.
3. **Duplicated logic** — the same rule/validation/transform/API call/format implemented in 2+ places where a bug fix would have to be applied twice. Only flag duplication that actually causes maintenance pain — ignore coincidental textual similarity; duplicated *knowledge* is the target.
4. **Obvious refactors** — dead code, unused imports/variables, functions clearly too long or doing too many things, misleading names. Only the obvious wins; no architectural proposals.
5. **Reusable pieces** (only if obvious) — UI/logic repeated enough that one shared component/hook/function clearly pays off. Skip if it's a stretch.
6. **Quick health checks** — missing error handling around network/IO, missing timeouts on outbound HTTP, swallowed exceptions, N+1 queries, unbounded queries / missing pagination on large lists, resource leaks. Anything else genuinely risky you happen to notice — keep it brief.

End Pass 1 with: the **top 5 items by risk**, and the question **"which groups/items do I fix?"** Then **stop and wait** for approval.

## PASS 2 — Fix (after explicit approval, approved items only)

Fix in this order, and after **each** group: re-run the baseline commands (build, tests, app starts), confirm the app still builds and runs, and make a checkpoint commit (`audit: <group>`) so any group can be rolled back alone. Stop and confirm with the user before moving to the next group.

1. **Security fixes first.** These may change behavior on purpose — call out exactly what behavior changes for each one (e.g., "requests without a valid session now get 401"). Secrets: move to env vars/secret store, provide a `.env.example`, and remind the user that an ever-committed secret must be **rotated** — removing it from code is not enough.
2. **Dependencies.** Bump patch/minor freely; update the lockfile; do NOT apply major upgrades — leave them flagged with the one-line migration note. Build and test after updating.
3. **Safe cleanups.** Duplication, refactors, reusable pieces — only the ones the user approved. These must NOT change behavior: same inputs, same outputs, same errors. Show **before/after** for each. Never weaken, skip, or delete a test to make anything pass.

If companion guard skills are available, run them on your own output before presenting: a clean-code guard on Pass 2 diffs, a test guard on any tests you added.

## Rules

- Don't touch business logic without asking.
- Prefer the smallest change that solves the problem.
- A fix needing a big rewrite or a breaking upgrade gets a recommendation, not an implementation.
- If a finding is ambiguous (can't tell if code is dead, can't tell if behavior is intended), ask — don't guess.
- Final summary: security issues fixed (with behavior changes), packages updated (old → new), what was cleaned up, and a **"needs your decision"** list (majors, big rewrites, unresolved ambiguities, secrets to rotate).

## When NOT to use this skill

- Reviewing a single diff or PR — use a clean-code guard instead.
- Reviewing tests — use a test guard instead.
- Reviewing documentation — use a docs guard instead.
- Implementing a requested feature or making a targeted bug fix — just do that; this skill is an audit, not a feature pipeline.
- Factual/conceptual questions, CI/tooling config, pure architecture discussion, or running/debugging tests — answer directly.
