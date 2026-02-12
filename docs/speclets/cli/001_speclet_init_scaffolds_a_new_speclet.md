---
id: "01JMBJK7QRVX3N4P5G6H8W9Y0Z"
name: "001_speclet_init_scaffolds_a_new_speclet"
type: "behavior"
status: "draft"
last_verified: "2026-02-12"
---

## Related Files

- `src/commands/init.rs`
- `src/ulid.rs`
- `src/template.rs`
- `tests/commands/init_test.rs` (Test)

## Functional Overview

`speclet init` creates a new speclet Markdown file from a template, auto-generating a ULID for the `id` field. It accepts a target directory and an optional name, then writes a ready-to-edit speclet to disk.

## Design Intent

The init command lowers the barrier to creating speclets. By auto-generating the ULID and populating boilerplate sections, developers can focus on writing scenarios rather than remembering the front-matter schema. The command should complete in under a second and require no network access.

## Key Members

- `target_dir: String` — the directory where the new speclet will be created (e.g., `docs/speclets/auth/`)
- `name: Option<String>` — optional speclet name; if omitted, the user is prompted or a placeholder is used
- `type: Option<String>` — optional type classification; defaults to none (field omitted from front-matter)

## Scenarios

### Basic invocation with a name

1. User runs `speclet init docs/speclets/auth --name "user_can_sign_up_with_email"`
2. CLI generates a new ULID
3. CLI determines the next sequential number by scanning existing files in `docs/speclets/auth/` (e.g., if `001_*.md` and `002_*.md` exist, the next number is `003`)
4. CLI writes `docs/speclets/auth/003_user_can_sign_up_with_email.md` with:
   - `id`: the generated ULID
   - `name`: `"003_user_can_sign_up_with_email"`
   - `status`: `"draft"`
   - `last_verified`: today's date
   - All recommended sections as empty placeholders
5. CLI prints the path of the created file to stdout

### Invocation without a name

1. User runs `speclet init docs/speclets/auth`
2. CLI generates a ULID and determines the next sequential number
3. CLI writes `docs/speclets/auth/003_untitled.md` with `name: "003_untitled"`
4. CLI prints the path to stdout
5. The user renames the file and updates the `name` field manually

### Invocation with a type

1. User runs `speclet init docs/speclets/auth --name "password_policy" --type "constraint"`
2. CLI includes `type: "constraint"` in the front-matter
3. All other behavior is identical to the basic invocation

### Target directory does not exist

1. User runs `speclet init docs/speclets/new_domain --name "some_behavior"`
2. CLI creates the directory `docs/speclets/new_domain/` recursively
3. Sequential number starts at `001`
4. File is created as `docs/speclets/new_domain/001_some_behavior.md`

### Target directory already contains non-sequential files

1. User has manually created files without sequential numbering in the target directory
2. CLI scans for the pattern `NNN_*.md` and finds no matches
3. Sequential number starts at `001`

## Generated Template

The output file contains the following content:

```markdown
---
id: "<generated ULID>"
name: "<NNN>_<provided or 'untitled'>"
status: "draft"
last_verified: "<today's date>"
---

## Related Files

-

## Functional Overview



## Scenarios

###

1.
```

The template includes only the required sections with minimal placeholders. Optional sections (Design Intent, Key Members, Failures / Exceptions) are omitted to avoid clutter — users add them when needed.

## Failures / Exceptions

- If `target_dir` points to a path where a file (not a directory) already exists, CLI exits with an error: `Error: '<path>' is a file, not a directory`
- If a file with the exact same name already exists, CLI exits with an error: `Error: '<path>' already exists`
- If the filesystem is read-only or permissions are insufficient, CLI exits with the OS-level error message
