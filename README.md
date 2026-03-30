# Mixbook iOS — Claude Code Review Command

A single-file Claude Code command that reviews Mixbook iOS pull requests against TCA/SwiftUI-specific rules.

Once installed, the `/code-review-mixbook` command is available in **any project** on your machine — not just Mixbook.

---

## Prerequisites

### 1. Install Claude Code
```bash
npm install -g @anthropic/claude-code
```

Log in with your Anthropic account:
```bash
claude login
```

### 2. Verify `git` is available
```bash
git --version
```

---

## Installation

Copy `code-review-mixbook.md` into Claude Code's user-level commands directory:

```bash
cp code-review-mixbook.md ~/.claude/commands/code-review-mixbook.md
```

That's it. No restart needed.

---

## Usage

1. Open a terminal inside the Mixbook iOS repo
2. Make sure you're on the branch you want to review
3. Start a Claude Code session:
   ```bash
   claude
   ```
4. Run the command:
   ```
   /code-review-mixbook
   ```

Claude will diff your branch against `origin/staging`, apply all Mixbook-specific rules, and output findings grouped by severity: **CRITICAL / WARNING / SUGGESTION**.

**Tip — fetch before reviewing so the diff is current:**
```bash
git fetch origin
```

---

## Updating the rules

1. Edit `code-review-mixbook.md` in this repo
2. Re-copy it to `~/.claude/commands/`:
   ```bash
   cp code-review-mixbook.md ~/.claude/commands/code-review-mixbook.md
   ```

---

## What's checked

| Category | Examples |
|---|---|
| **TCA architecture** | `@Reducer`, `@ObservableState`, delegate actions, `@Presents`, `Scope` |
| **Always flag** | Force unwraps, missing error handling, hardcoded secrets, raw API paths |
| **Style** | 2-space indent, 120 char line limit, naming conventions, no unused imports |
| **Native preferred** | `URLSession`, `Codable`, `AsyncImage`, native `String` methods |
| **Security** | `AuthenticationStorage` for tokens, no PII in logs |
| **Performance** | `LazyVStack`/`LazyVGrid` for large lists, no `GeometryReader`, 500KB image limit |

**Skipped automatically:** Apollo/GraphQL generated files, Tuist project files, lock files, `Strings+Generated.swift`
