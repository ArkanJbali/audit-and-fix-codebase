# Duplication Detection and Refactoring Guidance

Supporting reference for Pass 1 sections 3–5 (duplicated logic, obvious
refactors, reusable pieces). Read this file when you need to classify a
duplication or decide whether a refactor is safe.

## Contents

- Knowledge vs text duplication
- The Rule of 3
- Wrong abstraction (Sandi Metz corollary)
- Dead code detection
- Function complexity heuristics
- N+1 query detection
- Reusable component identification
- Refactoring safety

---

## Knowledge vs text duplication

**Duplicated knowledge** (the real DRY violation): the same rule, formula,
business constraint, or domain invariant expressed in multiple places, possibly
in different forms (code + config + docs).

**Duplicated text** (not a DRY violation): two functions that look alike but
encode different business rules. Example:

```python
# These look the same but apply different discount rules
def black_friday_discount(price):
    return price * 0.7  # 30% off

# vs

def employee_discount(price):
    return price * 0.85  # 15% off
```

Can you name the underlying *knowledge* the duplicated lines represent? If not,
leave the duplication.

### When to flag

- ≥5 consecutive non-trivial lines appearing in ≥2 functions with the same
  *meaning* (not just the same *text*).
- The same regex, SQL fragment, or URL literal repeated in ≥3 sites.
- The same magic number or string repeated ≥3 times outside a constants module.
- The same validation branch duplicated across siblings of one module.
- A bug fix would need to be applied in two or more places.

---

## The Rule of 3

Don't extract an abstraction on the first occurrence of duplication. Don't
extract on the second. Wait for the third — by then you have enough signal
about the *real* shape of the shared knowledge to abstract correctly.

| Occurrence | Action |
|------------|--------|
| 1st | Nothing. Write the code. |
| 2nd | Nothing. Live with the duplication. |
| 3rd | Consider extraction, but only if you can name the underlying knowledge. |

---

## Wrong abstraction (Sandi Metz corollary)

From Sandi Metz, "The Wrong Abstraction" (Jan 2016):
*"duplication is far cheaper than the wrong abstraction."*

### Smells of a wrong abstraction

- An abstraction has accumulated per-caller branches and special-case flags.
- The shared function has 3+ optional parameters that each caller uses
  differently.
- Inline comments explaining "this is here for caller X."
- The function name no longer describes what it does (too many modes).
- Adding a new caller requires editing the shared function instead of adding
  new code.

### Remedy

1. Re-inline the abstraction back into each caller.
2. Delete the per-caller dead branches from the abstraction.
3. Live with honest duplication for a while.
4. Re-abstract only when the real shared knowledge becomes obvious.

### Rule

Do not introduce an abstraction to eliminate three lines of duplication unless
you can name the underlying *knowledge* the lines represent. If you can not
name it, leave the duplication.

---

## Dead code detection

### What to look for

- **Unused imports**: run `rg "^import "` on each file and check references.
- **Unused functions/methods**: search for the function name across the
  codebase (excluding the definition itself).
- **Unused variables**: linter catch. If no linter, look for variables that
  are assigned but never read after assignment.
- **Unreachable branches**: code after a `return`, `raise`, or `exit` in the
  same block. Code in an `if False:` block.
- **Half-implementations**: a function body containing only `pass`,
  `raise NotImplementedError`, `// TODO`, or a `return null` stub that was
  left unfinished.
- **Dead configuration**: env vars, config keys, or feature flags that nothing
  reads.

### When NOT to flag

- A function in a library or public API that exists for external consumers
  (even if no internal caller exists).
- A hook/event/middleware registration point where callers are registered
  dynamically.
- A recently introduced function that is in active development.

---

## Function complexity heuristics

### When to flag

- **Line count > 50 lines** (target ≤20) — likely doing too many things.
- **Nesting depth > 5** — hard to follow; extract nested blocks into helpers.
- **Parameters > 4** — extract a config object.
- **Boolean flag parameters** — split into two named functions.
- **Mixed abstraction levels** — HTTP calls + business logic + string
  formatting in the same function.

### Common LLM patterns to flag

- Functions that start with input validation, then business logic, then
  persistence, then formatting, then logging — all in one body.
- A "process" or "handle" function that is 100+ lines with no extraction.
- A function with a name like `processData` or `handleRequest` that is the
  largest function in the file.

---

## N+1 query detection

### What to look for

A query inside a loop that hits the database once per iteration:

```python
# N+1 bad
users = User.find_all()
for user in users:  # 1 query
    orders = Order.find_by_user(user.id)  # N queries

# Good (eager loading)
users = User.find_all(include='orders')
```

### What to flag

- Database queries inside `for`/`forEach`/`map` loops.
- ORM lazy-loading inside a loop (check for missing `.includes()`,
  `.eager_load()`, `.select_related()`, `.prefetch_related()`).
- REST API calling an endpoint per item in a collection.

---

## Reusable component identification

### Signs a reusable component is warranted

- The same UI markup or component structure appears ≥3 times with identical
  structure and different data.
- The same API call pattern (request + error handling + retry + transform)
  appears ≥3 times.
- The same validation rule appears in ≥3 locations across different features.

### Signs a reusable component is premature

- Only 2 occurrences of similar code.
- The two occurrences differ in intent (not just data).
- Extracting now would require parameterizing a behavior that currently exists
  in only one place.

---

## Refactoring safety

### Rules

1. **Preserve observable behavior**: same inputs produce the same outputs,
   same exceptions, same side effects. If a refactor changes behavior, it is
   not a refactor — it is a bug fix or a feature change. Flag it separately.
2. **Extract, don't inline** unless the Sandi Metz rule applies. Extracting a
   function is reversible; inlining is not.
3. **One refactor per commit**. Do not mix refactoring with feature work or
   bug fixes in the same change.
4. **Show before/after** for each refactor in the audit report. The user must
   be able to verify that behavior did not change.
5. **When in doubt, don't refactor**. If you cannot confidently say the refactor
   preserves behavior, flag the opportunity and let the user decide.
