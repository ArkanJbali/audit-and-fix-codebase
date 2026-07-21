# Dependency Scanners Reference

How to run each package manager's vulnerability scanner. Read this file during
Pass 1 section 2 to pick the correct scanner for the detected stack.

## Contents

- npm / Node.js
- pip / Python
- dotnet / .NET
- cargo / Rust
- composer / PHP
- govulncheck / Go
- bundler / Ruby
- mix / Elixir
- Detecting the package manager
- Fallback: manual inspection

---

## npm / Node.js

```bash
# Scan for vulnerabilities
npm audit

# JSON output for programmatic use
npm audit --json

# Detailed report
npm audit --audit-level=high

# List outdated packages
npm outdated

# Show dependency tree
npm ls --depth=5
```

**Limitations**: `npm audit` only reports advisories in the npm Advisory
Database. Private packages and scoped registries may not resolve. Runs only
with a `package-lock.json` present.

**Gotchas**:
- `npm audit fix` applies semver-safe patches automatically — flag this to the
  user in Pass 2, don't run it silently.
- Peer dependency warnings are not vulnerabilities.
- Workspaces need `npm audit --workspaces` or audit from each workspace root.

---

## pip / Python

```bash
# Install the scanner
pip install pip-audit

# Scan the current environment
pip-audit

# Scan requirements files
pip-audit -r requirements.txt

# Scan from lockfile (Pipfile.lock, poetry.lock)
pip-audit -r requirements.txt --strict

# JSON output
pip-audit --format json
```

**Alternatives**:
```bash
# Safety CLI
safety check --full-report

# pip-audit with pip-compile
pip-audit -r requirements.txt --require-hashes
```

**Gotchas**:
- `pip-audit` needs the package installed or a requirements file. Scanning a
  virtual environment is the most reliable.
- Poetry/Poetry.lock is supported via pip-audit's built-in resolver.
- Conda environments need `conda list --export` piped to a file.

---

## dotnet / .NET

```bash
# Scan for vulnerabilities
dotnet list package --vulnerable

# List all packages with versions
dotnet list package --outdated

# Check a specific project
dotnet list <project>.csproj package --vulnerable

# Include transitive dependencies
dotnet list package --vulnerable --include-transitive
```

**Gotchas**:
- Requires .NET SDK 6.0.300+ / 7.0.100+.
- Scans only the projects restored; run `dotnet restore` first.
- NuGet audit is available from NuGet 6.8+; earlier versions need
  `dotnet list package --vulnerable`.

---

## cargo / Rust

```bash
# Install the scanner
cargo install cargo-audit

# Scan the current crate
cargo audit

# JSON output
cargo audit --json

# Check for outdated dependencies
cargo outdated
```

**Gotchas**:
- Uses the RustSec Advisory Database.
- Runs from the crate root; needs a `Cargo.lock` file.
- `cargo audit --ignore RUSTSEC-YYYY-NNNN` for documented accepted risks.

---

## composer / PHP

```bash
# Scan for vulnerabilities
composer audit

# Detailed output
composer audit --format=json

# List outdated packages
composer outdated --direct

# Check for known security issues in locked dependencies
composer audit --locked
```

**Gotchas**:
- Uses FriendsOfPHP/security-advisories database.
- Works from the directory containing `composer.lock`.
- Private repositories may not be covered.

---

## govulncheck / Go

```bash
# Install
go install golang.org/x/vuln/cmd/govulncheck@latest

# Scan the current module
govulncheck ./...

# Show full call graph
govulncheck -show=traces ./...
```

**Gotchas**:
- Uses the Go Vulnerability Database (vuln.go.dev).
- `govulncheck` traces reachability — it reports only vulnerabilities in
  functions your code actually calls.
- Requires Go 1.18+.

---

## bundler / Ruby

```bash
# Install the scanner
gem install bundler-audit

# Scan
bundler-audit

# Check with update
bundler-audit --update
```

**Gotchas**:
- Uses the Ruby Advisory Database (ruby-advisory-db).
- Scans the `Gemfile.lock` in the current directory.
- `bundler-audit check --ignore OSVDB-XXXXX` for accepted risks.

---

## mix / Elixir

```bash
# Add to mix.exs
{:mix_audit, "~> 2.0", only: :dev}

# Run
mix deps.audit

# List outdated
mix hex.outdated
```

**Gotchas**:
- Uses the Elixir Advisory Database via Hex.
- Run `mix deps.get` first.

---

## Detecting the package manager

Check the project root for lockfiles / manifest files:

| Lockfile / manifest | Package manager | Language |
|---------------------|-----------------|----------|
| `package-lock.json` / `yarn.lock` / `pnpm-lock.yaml` | npm / yarn / pnpm | Node.js |
| `requirements.txt` / `Pipfile` / `poetry.lock` | pip / pipenv / poetry | Python |
| `*.csproj` / `*.fsproj` | dotnet | .NET / C# / F# |
| `Cargo.lock` | cargo | Rust |
| `composer.lock` | composer | PHP |
| `go.mod` / `go.sum` | go modules | Go |
| `Gemfile.lock` | bundler | Ruby |
| `mix.lock` | mix | Elixir |
| `build.gradle` / `build.gradle.kts` | gradle | Java / Kotlin |
| `pom.xml` | maven | Java |
| `cabal.project.freeze` / `stack.yaml.lock` | cabal / stack | Haskell |

---

## Fallback: manual inspection

If the project uses a package manager not listed above, or the scanner is
unavailable:

1. Read the dependency manifest files and check against public advisory
   databases manually.
2. Suggest the user set up CI scanning (GitHub Dependabot, GitLab Dependency
   Scan, Snyk, or similar).
3. Flag the gap in the findings report and move on — do not fabricate scan
   results.
