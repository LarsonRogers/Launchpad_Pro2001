# AI Agent Starter Template

Drop these files into any repo root to set up dual-agent support for
Claude Code and ChatGPT Codex (plus any other agent that reads markdown).

## Files

```
ARCHITECTURE.md         — Read-first structural contract. Universal rules for
                          separation of concerns, function discipline, state
                          management, and error handling. Also contains:
                          - Pre-edit protocol (mandatory checklist before any change)
                          - Scope control rules (one task = one change)
                          - Checkpoint/rollback strategy (git gates)
                          - Pattern registry (document existing patterns here)

CLAUDE.md               — Canonical instruction manual. Tech stack, language
                          rules, validation commands, file structure, code style,
                          git workflow, and task prompts. Fill in [BRACKETED]
                          placeholders with your project details.

AGENTS.md               — Codex entry point. Summarizes the hardest constraints
                          and points to ARCHITECTURE.md then CLAUDE.md.

.claude/settings.json   — Claude Code: auto-approve edits, allow common
                          commands, deny destructive ops and secrets.

.codex/config.toml      — Codex CLI: auto-approve on failure, workspace
                          write access, network for git, read CLAUDE.md
                          as instruction file.
```

## Setup

1. Copy all files (plus the `.claude/` and `.codex/` directories) into your repo root.

2. Replace all `[PROJECT_NAME]` and `[BRACKETED]` placeholders in
   `CLAUDE.md` and `AGENTS.md`.

3. Fill in `ARCHITECTURE.md`:
   - Add your project-specific directory structure and data flow.
   - Add any established patterns to the Pattern Registry.
   - Add key invariants that must not be broken.

4. Fill in `CLAUDE.md`:
   - Tech Stack table
   - Validation Commands
   - File Structure
   - Code Style section
   - Task Prompts

5. Add language-specific lint/test commands to the `allow` list in
   `.claude/settings.json` if needed (e.g., `"Bash(npm run lint)"`).

6. Commit and push.

7. Run `codex` or `claude` from the repo root. The agent reads the
   docs automatically and starts with the context it needs.

## Use Cases

**New project** — Drop the pack in, fill in the placeholders, start coding.
The agent bootstraps the Captain's Log on first session.

**Active project** — Every session resumes from the Captain's Log automatically.
Switch between Claude, Codex, Cursor, or any other agent without losing context.

**Inherited / unmaintained codebase** — Drop the pack into a repo you've cloned.
The agent detects no Captain's Log, runs a full four-phase assessment (read, map,
report, document), fills in the project-specific sections of `ARCHITECTURE.md`
and `CLAUDE.md` from what it finds, then creates the first log entry documenting
the codebase as inherited. You get an honest architectural overview and problem
area report before a single line is changed.

---

## How It Works

- **Claude Code** reads `CLAUDE.md` on session start, plus
  `.claude/settings.json` for permissions.

- **Codex CLI** reads `AGENTS.md` on session start (which directs it to
  `ARCHITECTURE.md` first), plus falls back to `CLAUDE.md` via
  `project_doc_fallback_filenames` in the config.

- **Other agents** (Cursor, Windsurf, Aider) can be pointed to
  `ARCHITECTURE.md` then `CLAUDE.md` manually. All instructions are agent-agnostic.

## Key Behaviors Enforced by This Template

**Pre-edit protocol** — The agent must read, map, and report before touching anything.
Prevents agents from diving in without understanding existing structure.

**Scope control** — One prompt = one logical change. Agent must declare scope
before editing and must not make opportunistic refactors. Prevents rewrite creep.

**Checkpoint gates** — Agent commits after every completed task, verifies clean
git state before starting, and rolls back on failure. Prevents accumulation of
broken state across tasks.

**Pattern registry** — Existing patterns are documented so the agent discovers
them instead of inventing alternatives. Prevents pattern fragmentation.

**Separation of concerns** — Logic, I/O, and data live in distinct layers.
No business logic inside handlers or callbacks. Prevents logic burial inside functions.

## Customization

### Adding language-specific commands to Claude Code

Add to the `allow` array in `.claude/settings.json`:
```json
"Bash(npm run lint)",
"Bash(npm run test *)",
"Bash(python -m pytest *)",
"Bash(ruff check *)",
"Bash(cargo build *)",
"Bash(make *)"
```

### Tightening Codex permissions

Change `approval_policy` in `.codex/config.toml`:
- `"on-failure"` — auto-run everything, ask only on failure (default)
- `"on-request"` — agent decides when to ask
- `"untrusted"` — only auto-run safe commands (ls, cat, etc.)
- `"never"` — never ask, fail silently on blocked ops
