# Audit Review Checklist

Structured walk-through for the two-pass audit flow. Read this file when you
need a step-by-step guide for running Pass 1 or Pass 2 correctly.

## Contents

- Pass 1: pre-flight
- Pass 1: walk order
- Findings severity rubric
- Findings output format
- Pass 2: execution order
- Pass 2: verification gates
- Final summary format

---

## Pass 1: pre-flight

Before reading any code:

1. **Detect the stack** — language(s), framework(s), package manager(s),
   entry points. Run `ls *.{json,yaml,yml,toml,cfg,ini,csproj,fsproj,gemspec,
   gradle,pom.xml}` to identify package manifests.
2. **Check git safety** — run `git status`. If there are uncommitted changes,
   ask the user to commit before Pass 2.
3. **Detect tests** — check for `test/`, `tests/`, `__tests__/`, `spec/`,
   `*_test.go`, `*.test.js`, `*.spec.ts`, `*Test.php`, `test_*.py`. Report
   test framework and run command if found. If none, say so plainly.
4. **Establish baseline** — run the build command, the test suite (if any),
   and start the app once. Record the exact commands. If the app does not
   build or run today, report that first and stop.

---

## Pass 1: walk order

Follow this order, one section at a time. Read code before flagging it.

### Section 1: Security

- [ ] Hardcoded secrets, API keys, tokens, passwords in code or committed
      config (`.env` in git, connection strings).
- [ ] Secrets visible in git history (`git log -p`).
- [ ] SQL injection: string interpolation in query building.
- [ ] Command injection: shell commands with user input.
- [ ] XSS: `dangerouslySetInnerHTML`, `innerHTML`, `v-html`, unescaped
      `<?= $var ?>`.
- [ ] Path traversal: file operations with user-supplied paths.
- [ ] SSRF: user-supplied URLs in server-side requests.
- [ ] Insecure deserialization: `pickle`, `yaml.load`, `unserialize` on
      untrusted data.
- [ ] Missing auth on routes, resolvers, or bulk endpoints.
- [ ] IDOR: object access without ownership check.
- [ ] Mass assignment: accepting full model objects from the client.
- [ ] Sensitive data in logs, `localStorage`, URLs, or error responses.
- [ ] Permissive CORS, debug mode in production, weak password hashing,
      missing rate limiting.

Use [security-checklist.md](security-checklist.md) for detailed checks per
category.

### Section 2: Dependencies

- [ ] Run the real scanner for the detected package manager (see
      [dependency-scanners.md](dependency-scanners.md)).
- [ ] Report actual output — never recite CVEs from memory.
- [ ] Build a table: current vs latest version for each direct dependency.
- [ ] Separate patch/minor bumps from major upgrades (major upgrades get a
      one-line migration note).
- [ ] Check for unused/dead dependencies.
- [ ] Check license red flags for the project's license type.

### Section 3: Duplicated logic

- [ ] Find ≥5-line blocks appearing in ≥2 places where a bug fix would need
      to be applied twice.
- [ ] Verify it's knowledge duplication, not coincidental textual similarity.
- [ ] Check for wrong abstractions (accumulated per-caller branches).

Use [duplication-and-refactors.md](duplication-and-refactors.md) for detailed
guidance.

### Section 4: Obvious refactors

- [ ] Unused imports, variables, functions.
- [ ] Dead code / unreachable branches.
- [ ] Functions >50 lines or cyclomatic >10.
- [ ] Boolean flag parameters.
- [ ] Parameters >4 without a config object.
- [ ] Misleading or generic names (`data`, `result`, `temp`, `helper`,
      `manager`, `process_*`, `handle_*`).
- [ ] Mixed abstraction levels in one function.

### Section 5: Reusable pieces (only if obvious)

- [ ] UI or logic repeated ≥3 times with the same shape.
- [ ] Extraction candidate where you can name the shared knowledge.

### Section 6: Quick health checks

- [ ] Missing error handling around network/IO.
- [ ] Missing timeouts on outbound HTTP requests.
- [ ] Swallowed exceptions (broad `catch`/`except` with no recovery).
- [ ] N+1 queries (query inside a loop).
- [ ] Unbounded queries / missing pagination on large lists.
- [ ] Resource leaks (unclosed files, connections, streams).

---

## Findings severity rubric

| Severity | Label | Criteria |
|----------|-------|----------|
| Critical | 🔴 Must fix before merge | Security vulnerability, data loss risk, correctness bug, swallowed exceptions, hardcoded "success" returns |
| High | 🟠 Should fix before next deploy | Design defect with significant maintenance cost, SOLID violation, moderate security gap |
| Medium | 🟡 Fix within iteration | Cleanup with moderate benefit, minor security hardening |
| Low | 🔵 Fix when convenient | Style, naming, minor inconsistency |
| Nit | ⚪ Optional | Single-letter names outside loops, minor style mismatch |

---

## Findings output format

Use this template for each section. Empty sections get a single line:
"no findings".

```text
### [Section Name]

| Severity | File:Line | Finding | Suggested Fix |
|----------|-----------|---------|---------------|
| Critical | src/app.ts:42 | Hardcoded DB password | Move to env var + rotate |
| High | src/api.ts:88 | N+1 query for orders | Add eager loading |
| Low | src/utils.ts:15 | `processData` name | Rename to `parseInvoiceRows` |
```

End Pass 1 with:
- **Top 5 items by risk** (ranked list with severity)
- **Question**: "Which groups/items do I fix?" Then stop and wait for approval.

---

## Pass 2: execution order

After the user approves specific items, fix in this order. After each group,
re-run the baseline commands and make a checkpoint commit.

### Group 1: Security fixes

- [ ] Move secrets to env vars. Provide `.env.example`.
- [ ] Remind user: ever-committed secrets must be **rotated**.
- [ ] Fix injection vulnerabilities (parameterized queries, escaping).
- [ ] Add missing auth checks.
- [ ] Fix IDOR / mass assignment.
- [ ] Remove sensitive data from logs, storage, URLs, errors.
- [ ] Fix CORS, disable debug mode, switch to strong hashing.
- [ ] Add rate limiting.

🔥 **Behavior may change** — call out exactly what behavior changes
(e.g., "requests without a valid session now get 401").

After Group 1:
```bash
# Build
<baseline build command>

# Test
<baseline test command>

# Start app
<baseline start command>

# Checkpoint commit
git add -A && git commit -m "audit: security fixes"
```

Stop and confirm with the user before moving to Group 2.

### Group 2: Dependency updates

- [ ] Bump patch/minor versions freely.
- [ ] Update lockfile.
- [ ] Do NOT apply major upgrades — leave them flagged with migration notes.

After Group 2: build, test, start app, checkpoint commit
(`git commit -m "audit: dependency updates"`).

Stop and confirm before Group 3.

### Group 3: Safe cleanups

- [ ] Remove duplicated knowledge (confirmed as safe).
- [ ] Remove dead code / unused imports.
- [ ] Extract reusable components (only approved ones).
- [ ] Rename misleading identifiers.
- [ ] Simplify complex functions (approved only).

⚠️ **Behavior must NOT change** — same inputs, same outputs, same errors.
Show **before/after** for each.

After Group 3: build, test, start app, checkpoint commit
(`git commit -m "audit: safe cleanups"`).

---

## Final summary format

```text
## Audit summary

### Security
- <N> issues fixed (list each with behavior changes flagged)
- <N> secrets flagged for rotation

### Dependencies
- <N> packages updated (old → new)
- <N> major upgrades flagged (not applied — needs decision)

### Cleanups
- <N> duplication fixes
- <N> refactors
- <N> dead code removals

### Needs your decision
- Major upgrades: <list>
- Big rewrites / architectural changes: <list>
- Secrets to rotate: <list>
- Unresolved ambiguities: <list>
```
