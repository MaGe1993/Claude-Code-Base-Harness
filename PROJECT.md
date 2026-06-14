---
name: Claude-Code-Base-Harness
description: LLM-facing architecture reference for the Claude Code configuration harness — decisions, structure, constraints, and non-goals for AI-assisted development.
updated: 2026-06-12
---

# Claude-Code-Base-Harness — Project Spec

## 1§ Goal

A curated, opinionated set of Claude Code configuration artefacts — rules, skills,
agents, hooks, output styles, and a base UI — that gives any project a correct Claude
Code setup without custom authoring from scratch. Users copy what they need into
`~/.claude/` or `.claude/`; nothing is installed or linked.

## 2§ Core features

- Format routing table in `CLAUDE.md` maps file types to style rules; Claude loads only the relevant rule for each file it touches.
- On-demand skill loading via `skills.index.toon`: CLAUDE.md directs Claude to read the index first, then load the full `SKILL.md` only when the skill is invoked.
- On-demand agent loading via `agents.index.toon`: same pattern as skills.
- Lifecycle hooks (each as `.ps1` + `.sh`): `check-memory` (SessionStart) detects Claude auto-memory files and prompts migration into `CLAUDE.md`; `pre-compact` (PreCompact) blocks compaction until the PROJECT.md Current work section is updated; `post-compact` (SessionStart, `compact`) re-injects PROJECT.md; `cleanup-sessions` (SessionStart, `clear`) prunes stale session artifacts.
- base-ui CSS design system for AI-generated HTML (bundled in the `frontend` skill's `assets/base-ui/`): glassmorphism, dark default, `prefers-color-scheme`, no build step.
- `settings.json` pre-sets `outputStyle: Executive` — bottom-line-first, signal-dense communication active by default.
- Skill coverage: harness authoring, documentation, diagramming, data conversion, analysis, project scaffolding, output style authoring.

## 3§ Non-goals

- No plugin or extension structure — Claude Code has no plugin API; plain files are the only stable interface.
- No shared build system, test suite, or CI pipeline for the harness itself.
- No enforcement of rules — all rules and output styles are advisory additions to the system prompt.
- No global toolchain assumption — skills that invoke CLIs verify existence with `(Get-Command <name>).Source` first.
- No auto-loading of rules — the routing table in `CLAUDE.md` is the sole trigger; rules never carry `paths` or `globs` frontmatter.

## 4§ Directory structure

```text
Claude-Code-Base-Harness/
  .claude/
    rules/            — on-demand format and behavior rules; no auto-load frontmatter
    skills/           — one subdirectory per skill: SKILL.md + assets/
      skills.index.toon — TOON registry; read before loading any SKILL.md
      frontend/assets/base-ui/ — CSS design system: tokens.css, components.css, demo.html
    agents/           — one subdirectory per agent: <name>/AGENT.md + optional assets/
      agents.index.toon — TOON registry; read before loading any agent
    output-styles/    — output style files; executive.md is the pre-built default
    hooks/            — lifecycle scripts (.ps1 + .sh): check-memory, pre-compact, post-compact, cleanup-sessions
    settings.json     — pre-sets outputStyle: Executive; registers all hooks; copy to ~/.claude/ or .claude/
  CLAUDE.md           — harness entry point: routing table, skill and agent loading instructions
  PROJECT.md          — this file
  README.md           — user-facing overview (English, primary)
  README.de.md        — German translation; may lag English
  CHANGELOG.md        — Keep a Changelog 1.1.0
  LICENSE             — MIT
```

## 5§ Architecture decisions

### 5.1§ Plain files, no plugin structure (locked)

Users copy files from the harness into their own `~/.claude/` or `.claude/`. No install
step, no symlink, no registry entry.

Rationale: Claude Code's plugin system imposes a manifest, marketplace mechanics,
and namespaced invocation, and plugin subagents ignore several frontmatter fields.
Plain file copy permits partial adoption (single skills or rules), local editing
without forking a plugin, and identical behavior across user and project scope.

### 5.2§ Rules are never auto-loaded

Rules carry no `paths`, `globs`, or `alwaysApply: true` frontmatter — except
`command-execution.md` which is explicitly always-on. The `CLAUDE.md` routing table
is the sole loader.

Rationale: The `paths` frontmatter field has documented bugs in Claude Code (ignores
relative paths). Routing table gives deterministic, auditable control; it also
prevents unused rules from consuming context window tokens in unrelated sessions.

### 5.3§ TOON index pattern for skills and agents

`CLAUDE.md` instructs Claude to read the index file first, then load the full `SKILL.md`
or agent file only when that skill or agent is actually needed.

Rationale: A TOON index costs ~40% of equivalent JSON and provides one auditable
registry per artifact type. Native skill discovery keeps only descriptions in
context, but the index pattern also covers what native discovery does not reach —
agents and manually loaded artifacts — and gives Claude a deterministic lookup
instruction instead of discovery heuristics.

### 5.4§ No inter-skill dependencies — each skill is self-contained

Every skill stores its output templates in its own `assets/` subdirectory. No skill
references another skill's files.

Rationale: Users copy individual skills, not the full harness. Any cross-skill file
reference breaks partial copies. Self-containment is the only structure compatible
with selective adoption.

### 5.5§ One template per artifact type

Each skill provides exactly one output template. Variants and sub-types are handled
via numbered subsections under `## 2§ Rules`, not separate template files.

Rationale: Multiple templates require the skill to decide which applies, adding
decision logic that belongs in the skill body. One template forces all variants into
a single consistent structure that Claude adapts via the subsection rules.

### 5.6§ Scope confirmation lives in CLAUDE.md, not per-skill

The pattern "state full target path before writing; mark inferred scope `[assumed]`"
is documented once in `CLAUDE.md §2`, not repeated in individual skills.

Rationale: Per-skill duplication creates drift — when the convention changes, every
skill must be updated. A single authoritative line in `CLAUDE.md` applies uniformly
to all skills.

### 5.7§ 30-day staleness check before skill invocation

Before invoking any skill, `CLAUDE.md §4` instructs Claude to check the `updated:`
frontmatter field. If more than 30 days old, web research is performed and the skill
is updated before use.

Rationale: External standards the skills depend on (Keep a Changelog, SPDX, ECharts API,
TOON spec) evolve. A skill applying silently stale knowledge produces subtly wrong
output that the user may not catch.

### 5.8§ Executive output style as harness default

`settings.json` pre-sets `"outputStyle": "Executive"`. This takes effect in every
new session for any user who copies `settings.json`.

Rationale: Executive mode (bottom-line first, maximum signal density, structured
task-completion summary) is the optimal baseline for technical users who work
alongside Claude's output rather than reading it as documentation.

### 5.9§ `disable-model-invocation` is never set

All skills are context-invocable — any model in any session can invoke them.

Rationale: Restricting invocation would prevent skills from working in sessions
where the user has not enabled specific tools. Context invocation is always available.

### 5.10§ `name:` is the first frontmatter field in all skill and rule files

Convention enforced across the harness; `description:` is always second; `updated:`
always last.

Rationale: File identity visible on line 1 makes directory scanning faster for both
humans and LLMs. Consistent ordering enables mechanical validation.

## 6§ Constraints

- Windows 11 / PowerShell 7+ is the primary development platform; hooks use `$env:VAR` syntax and `pwsh` shebang-equivalent.
- `CLAUDE.md` must remain under ~80 lines; it is loaded into every session and has a permanent token cost proportional to its length.
- TOON files must have no trailing newline (TOON spec v3.2 requirement).
- Rule files carry no `paths` or `globs` frontmatter — documented bug in Claude Code.
- Output style body must not exceed 50 lines — focused styles outperform sprawling ones; dilution reduces behavioral adherence.
- base-ui (`frontend` skill assets) requires internet access at render time (Google Fonts `@import` in `tokens.css`); offline environments must self-host Roboto and Roboto Mono.
