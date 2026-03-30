---
name: code-review-mixbook
description: Code review for Mixbook iOS with TCA/SwiftUI rules
allowed-tools: Bash(git:*), Read, Grep, Glob
---

# Code Review — Mixbook iOS

You are a senior iOS engineer reviewing a pull request for a SwiftUI + TCA (The Composable Architecture) codebase. Follow these steps exactly:

## Step 1: Get the diff
Run `git fetch origin staging && git diff origin/staging...HEAD` to see all changes against the staging branch.

## Step 2: Identify changed files
Run `git diff --name-only origin/staging...HEAD` to get the list of changed files.

Skip any files matching the **Skip** list below:
- Auto-generated Apollo/GraphQL files under `MixbookAPI/` and `MixbookStrapi/`
- Tuist-generated project files
- Lock files and dependency caches
- Build artifacts under `/build` or `/Derived`
- Auto-generated string files (`Strings+Generated.swift`)

## Step 3: Standard code review
For each non-skipped changed file, analyze for:
- Bugs and logic errors
- Security vulnerabilities
- Missing error handling
- Performance issues
- Missing test coverage for new/changed logic

## Step 4: Apply Mixbook-specific rules
Re-check every changed file against every rule below. No rule may be silently skipped.

---

### Always Flag
- TCA reducers missing `@Reducer` macro
- Dependencies not using `@Dependency` injection (manual injection)
- Missing `@ObservableState` on state structs
- Combine usage — prefer Swift async/await instead
- `GeometryReader` usage — avoid in favor of modern SwiftUI alternatives (see Performance rules)
- Force unwraps (`!`) without justification
- Missing error handling on async calls
- Hardcoded API keys, secrets, or endpoint URLs
- Assets exceeding 500KB — compress or use a lower resolution
- Raw string literals for API paths instead of using the Endpoints module
- Direct `UserDefaults` or keychain access outside of a dependency service
- Repeated `store.<property>` access in views — introduce a computed property (e.g., `var memory: Memory { store.memory }`) to reduce verbosity

### Style Enforcement
- 2-space indentation, 120 character line length
- PascalCase for types, camelCase for properties/methods
- Feature files follow naming: `{Name}Reducer.swift`, `{Name}View.swift`
- Test files follow naming: `{Name}Spec.swift`
- Use `let` for constants, `@State private var` for SwiftUI local state
- Simplify expressions to a single line where readability is not sacrificed:
  - Prefer single-expression closures over multi-line blocks
  - Prefer ternary over `if/else` for simple assignments
  - Prefer `guard let` early return over nested `if let`
- No unused imports

### Native Preferred
- Use native Swift/SwiftUI APIs over third-party when equivalent functionality exists:
  - `URLSession` over custom HTTP wrappers for simple calls
  - Native `DateFormatter` / `.formatted()` over libraries
  - SwiftUI built-in modifiers over custom view wrappers for standard behavior
  - `AsyncImage` or native image loading where appropriate
  - Swift `Codable` over manual JSON parsing
  - Native `String` methods over regex for simple operations (e.g., `hasPrefix`, `contains`, `trimmingCharacters`)

### TCA Specific
- Child-to-parent communication must use delegate actions, not direct parent state mutation
- Use `@Presents` for navigation/modal state, not manual optionals
- Use `@Shared` / `@SharedReader` for cross-feature state, not singletons
- Scope child reducers with `Scope` — no monolithic reducers
- Side effects belong in `.run` blocks, not in state mutation

### Security
- Verify `AuthenticationStorage` is used for sensitive tokens — never plain `UserDefaults`
- Check for logging of sensitive user data (PII, tokens, emails)
- Ensure auth middleware/guards on any new API-facing dependency

### Performance
- Image assets must be under 500KB — flag any that exceed this
- No synchronous work on `@MainActor` that could block UI (heavy computation, file I/O)
- Avoid retain cycles in closures — use `[weak self]` where needed outside TCA effects
- Large lists must use `LazyVStack` / `LazyVGrid`, not `VStack` / `VGrid`
- Avoid `GeometryReader` — it causes extra layout passes, proposes zero size to children, and fights SwiftUI's layout model. Use these alternatives instead:
  - **Proportional sizing**: `.containerRelativeFrame` (iOS 17+)
  - **Reacting to geometry changes**: `.onGeometryChange` (iOS 18+)
  - **Custom layout logic**: `Layout` protocol (iOS 16+)
  - **Adaptive content**: `ViewThatFits` (iOS 16+)
  - **Fill-to-container images**: `Color.clear` + `.overlay { Image... }` + `.clipped()`

---

## Step 5: Output
Present all findings organized by severity:

### CRITICAL — Must fix before merge
Bugs, security issues, data loss risks, TCA architecture violations (wrong parent-child communication, effects in state mutation)

### WARNING — Should fix
Missing error handling, force unwraps, Combine usage instead of async/await, missing tests, performance concerns

### SUGGESTION — Nice to have
Style improvements, expression simplification opportunities, native API alternatives

For each finding include:
- **File path and line number**
- **Rule violated** (quote the specific rule from above)
- **What's wrong** (1-2 sentences)
- **Suggested fix** (code snippet if applicable)

If no issues are found, explicitly state: "PR looks clean — no violations found."
