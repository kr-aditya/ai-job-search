---
framework_version: 1.0.0
---

# Agent Guidelines: AI Job Search

This workspace is structured to manage job search activities, scraper tools, CVs, cover letters, and interview preparation.

## Thin-Pointer Design (Single Source of Truth)

To prevent duplication and configuration drift across different AI agent frameworks (Claude Code, Google Antigravity, Codex, Cursor, Gemini CLI, etc.), this workspace uses a unified thin-pointer design. All agent runtimes should load the canonical specifications and candidate profiles from the files and directories below:

1. **Personal Candidate Profile:**
   - The candidate profile, contact details, education, and target preferences are defined in [CLAUDE.md](CLAUDE.md) and the individual profile methodology files under [.claude/skills/job-application-assistant/](.claude/skills/job-application-assistant/) (specifically `01-*.md` etc.).
2. **Canonical Workflow Specifications:**
   - The step-by-step instructions and triggers for tasks (setup, scrape, rank, apply, upskill, interview) are defined in the [.claude/](.claude/) directory (specifically under `.claude/skills/` and `.claude/commands/`).
   - Do not duplicate these rules or specifications. Treat `.claude/` files as the single source of truth.
3. **Portal Search Skills:**
   - Job-portal search CLIs live under [.agents/skills/](.agents/skills/) in the portable Agent Skills format (with a `SKILL.md` per portal). Codex and Antigravity discover these automatically; the `/scrape` workflow in [.claude/skills/job-scraper/](.claude/skills/job-scraper/) orchestrates them.

## Dual Claude Code / OpenCode Setup

This repository runs on both Claude Code and OpenCode. `.claude/` remains the canonical source of truth for workflows, skills, commands, agents, and the candidate profile. The OpenCode layer mirrors commands and agents under `.opencode/` and loads the same instructions.

### Instruction files

- `AGENTS.md` (this file) is the main OpenCode instruction file.
- `CLAUDE.md` is loaded explicitly through `opencode.json` → `instructions`, because OpenCode otherwise uses only `AGENTS.md` when both files exist.
- Do not duplicate profile or workflow rules here. Edit `.claude/` and/or `CLAUDE.md`, then keep the `.opencode/` mirrors aligned.

### Commands

All Claude Code commands have OpenCode equivalents in `.opencode/commands/` with the same names: `/setup`, `/apply`, `/interview`, `/rank`, `/outcome`, `/reset`, `/add-template`, `/add-portal`, `/expand`, `/gmail-sync`, `/notion-sync`, `/html-report`. `/scrape` and `/upskill` are thin OpenCode commands that load the existing `scrape` and `upskill` skills; the full workflows remain in `.claude/skills/`.

### Agents

The Claude Code subagents in `.claude/agents/` are mirrored in `.opencode/agents/`: `application-reviewer`, `posting-fetcher`, `posting-scorer`. They are read-only by default (`edit: deny`); `application-reviewer` may run bash only for `tools/fetch_page.py` and `tools/robots_check.py`.

### Skills

OpenCode discovers project skills from both `.claude/skills/` and `.agents/skills/`; no duplication is needed. The portal CLIs under `.agents/skills/*/cli/` are unchanged. Unknown Claude frontmatter fields such as `allowed-tools` and `framework_version` are ignored by OpenCode; the skill logic is unaffected.

### Tool name mapping

Claude Code → OpenCode: Read → `read`, Write → `write`, Edit → `edit`, Glob → `glob`, Grep → `grep`, Bash → `bash`, WebFetch → `webfetch`, WebSearch → `websearch`, Agent/Task → `task`, AskUserQuestion → `question`. The migrated commands and agents use the OpenCode names.

### Runtime notes

- **Scratchpad:** when a workflow needs a scratch directory, set `SCRATCHPAD` to a temporary directory outside the repo before running shell snippets. PowerShell: `$env:SCRATCHPAD = Join-Path $env:TEMP 'opencode-job-search'` and replace POSIX `${SCRATCHPAD:?...}` references with `$env:SCRATCHPAD`; POSIX: `export SCRATCHPAD=$(mktemp -d)`.
- **Windows PowerShell 5.1:** translate POSIX snippets where needed (`;` instead of `&&`, `Remove-Item` instead of `rm`, `python` instead of `python3`). The commands already warn about `python3` on Windows.
- **AI tooling naming:** `CLAUDE.md` says to name **Claude Code** in CVs and cover letters. That is a Claude Code artifact. Under OpenCode, name the AI coding agent actually used (**OpenCode**) instead of making a false claim; the rest of the profile and verification rules are unchanged.

### MCP integrations

The repository does not configure external MCP servers. `/gmail-sync` requires a Gmail MCP server; `/notion-sync` requires a Notion remote MCP server (`https://mcp.notion.com/mcp`). Add them to the `mcp` section of `opencode.json`, restart the session, and authenticate with `opencode mcp auth <server>`. Without them those commands exit cleanly with setup instructions; no other workflow depends on them.

### Permissions

`opencode.json` mirrors the intent of `.claude/settings.json`: portal CLIs, salary lookup, and the repository's Python tools are pre-approved; edits to framework files and `.opencode/` prompt for confirmation; `.env` and SSH/AWS/GnuPG reads are denied; outbound POST/upload commands are denied. `.claude/settings.json` remains authoritative for Claude Code; `tools/security_guards.py` still validates it, not `opencode.json`.
