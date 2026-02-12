# speclet

**Atomic, living specification cards for AI-agent-friendly development.**

speclet is a minimal specification format and toolkit for Spec-Driven Development (SDD). Each speclet is a single Markdown file describing exactly one behavior, with machine-readable front-matter for lifecycle tracking and agent navigation.

## Philosophy

Most SDD tools (GitHub Spec Kit, Amazon Kiro, BMAD) treat specifications as large, monolithic documents that feed into linear pipelines. This works for greenfield features but becomes a sledgehammer for everyday development — small fixes, incremental changes, and brownfield evolution.

speclet takes the opposite approach:

- **One file, one behavior.** Keep the unit of specification atomic. An AI agent should never need to parse an entire feature to understand a single behavior.
- **Context-window aware.** LLM context is a finite resource. A speclet is sized to fit comfortably alongside its test file and implementation in a single agent session.
- **Living documents, not session artifacts.** Each speclet carries its own lifecycle status and verification date. Specifications survive beyond the task that created them.
- **Process-agnostic data layer.** speclet defines the *format* of specifications, not the *workflow*. Use it with TDD, BDD, or any development process. Pair it with your own workflow commands or CI pipelines.
- **Engine and tool independent.** Plain Markdown with YAML front-matter. No IDE lock-in, no proprietary CLI required.

## Quick Start

### As a Git Submodule

```bash
git submodule add git@github.com:yoshiakist/speclet.git speclet
```

### Writing Your First Speclet

Create a Markdown file under your project's specification directory, organized by domain:

```
docs/domains/
  auth/
    001_user_can_sign_up_with_email.md
    002_user_can_reset_password.md
  cart/
    001_items_can_be_added_to_cart.md
    002_cart_total_reflects_quantity_changes.md
```

## Speclet Card Format

Every speclet follows this structure:

```markdown
---
name: "001_user_can_sign_up_with_email"
status: "draft"
last_verified: "2026-02-12"
---

## Related Files

- `src/auth/signup_controller.gd`
- `src/auth/email_validator.gd`
- `test/unit/auth/test_signup.gd` (Test)

## Functional Overview

Users can create an account by providing a valid email address and password.

## Design Intent

Email signup is the primary onboarding path. We validate email format
client-side and uniqueness server-side to provide fast feedback.

## Key Members

- `email: String` — the user's email address, validated against RFC 5322
- `password_hash: String` — bcrypt hash, never stored in plaintext

## Scenarios

### Successful signup

1. User submits a valid email and a password of 8+ characters
2. System creates the account and emits `account_created` signal
3. User is redirected to the welcome screen

### Duplicate email

1. User submits an email that already exists
2. System displays an error: "This email is already registered"
3. No account is created

### Invalid email format

1. User submits a malformed email (e.g., "foo@")
2. System displays a validation error before submission

## Failures / Exceptions

- If the database is unreachable, display a generic error and log the incident
```

### Front-Matter Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Matches the filename (without `.md`). Human-readable behavior title. |
| `status` | enum | `draft` · `in-development` · `stable` · `deprecated` |
| `last_verified` | date | `YYYY-MM-DD` — when this speclet was last confirmed to match the implementation. |

### Status Lifecycle

```
draft ──→ in-development ──→ stable ──→ deprecated
  ↑            │                │
  └────────────┘                │
  (requirements changed)        │
                                ↓
                           (superseded or removed)
```

- **draft**: Behavior is proposed but not yet implemented.
- **in-development**: Implementation or tests are in progress.
- **stable**: Implementation matches the speclet. Tests pass. Verified by `last_verified` date.
- **deprecated**: Behavior has been removed or superseded. Kept for historical reference.

### Recommended Sections

| Section | Required | Purpose |
|---------|----------|---------|
| Related Files | Yes | Links speclet to source and test files |
| Functional Overview | Yes | One-paragraph summary of the behavior |
| Design Intent | No | Explains *why*, not *what* |
| Key Members | No | Important state and parameters in natural language |
| Scenarios | Yes | Step-by-step behavior descriptions |
| Failures / Exceptions | No | Edge cases and error handling |

### Writing Guidelines

- Write scenarios in **natural language**, not code. Avoid copy-pasting implementation details.
- **Do** use exact names for signals, classes, and enums — these are the contract between spec and code.
- Keep each speclet **focused on a single behavior**. If you find yourself writing "also, ..." — split it into a separate speclet.
- Speclets can reference each other via relative paths in a "Referenced Specifications" section.

## Comparison with Other Tools

| | speclet | GitHub Spec Kit | Amazon Kiro | ADR | Gherkin |
|---|---|---|---|---|---|
| Granularity | One behavior | One feature | One feature | One decision | One feature (multi-scenario) |
| Lifecycle status | Yes (4 states) | No | No | Yes (3-4 states) | No |
| Verification date | Yes | No | No | No | No |
| Test integration | By convention | Optional | Optional | No | Executable |
| Process coupling | None | Linear pipeline | Linear pipeline | None | Test runner |
| Files per spec | 1 | 3-4 | 3 | 1 | 1 |
| IDE dependency | None | None | Kiro IDE | None | None |

## Roadmap

### v0.1 — Core CLI

- [ ] `speclet index` — Scan a specification directory and generate a machine-readable index (`INDEX.md` per domain + `index.json` at root)
- [ ] `speclet init` — Scaffold a new speclet from a template
- [ ] `speclet status` — Report speclet counts by status and flag stale `last_verified` dates

### v0.2 — Bidirectional Traceability

- [ ] `speclet tag` — Assign a ULID to a speclet and inject `SDD-REF: <ulid>` markers into related source files
- [ ] `speclet trace` — Resolve a ULID from either a speclet or a source file to its counterpart
- [ ] `speclet orphans` — Detect speclets with no matching source markers, or source markers with no matching speclet

### v0.3 — Drift Detection

- [ ] `speclet drift` — Compare `last_verified` dates against git history of related files; flag speclets where source has changed since last verification
- [ ] `speclet ci` — Exit with non-zero status if drift is detected (for CI integration)
- [ ] GitHub Actions workflow template

### v0.4 — Agent Integration

- [ ] Skill / prompt templates for Claude Code, Cursor, and other AI coding assistants
- [ ] `speclet search <query>` — Full-text + tag-based search across all speclets, optimized for agent consumption
- [ ] Agent-friendly output formats (JSON, structured Markdown)

### Future Considerations

- Mermaid diagram generation from cross-references between speclets
- Dependency graph visualization
- Multi-language support for speclet content (i18n metadata)
- Plugin system for custom front-matter fields and validation rules

## Contributing

speclet is in its early stages. Contributions, feedback, and real-world usage reports are welcome. Please open an issue or pull request on GitHub.

## License

MIT
