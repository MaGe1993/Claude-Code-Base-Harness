# Changelog

All notable changes to Claude-Code-Base-Harness are documented here.
The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.0.0] - 2026-06-14

First public release. Plain files — copy what you need into `~/.claude/` or a project's `.claude/`.

### Added
- 26 on-demand skills covering the full Claude Code surface: artifact authoring (`skills`, `rules`, `hooks`, `subagents`, `settings`, `mcp`, `outputstyle`, `new-project`), document authoring (`file-claude`, `file-readme`, `file-project`, `file-changelog`, `file-license`), formats and visuals (`toon`, `mermaid`, `echarts`, `frontend`), and analysis (`lean-review`, `audit-security`, `audit-legal`, `proofread`, `sota`), plus `explain`, `masterprompt`, `tasks`, and `loop`
- Three C-suite agents that orchestrate skills: `ag-cqo` (Chief Quality Officer — audits harness structure, source-code security, legal compliance, and finished deliverables, then renders an Approved/Rejected stage-gate), `ag-cto` (Chief Technology Officer — writes, verifies, and self-reviews code), and `ag-cso` (Chief Strategy Officer — web-grounded state-of-the-art assessment)
- Eight rules loaded on demand through the `CLAUDE.md` routing table: `md-style`, `json-style`, `yaml-style`, `toon-style`, `command-execution` (always on), `security`, `error-handling`, and `visual-verify`
- Eight lifecycle hooks, each a `.ps1`/`.sh` pair behind a shared `dispatch.mjs` launcher that runs the right script per platform with no setup: `check-credentials`, `check-memory`, `cleanup-sessions`, `dangerous-cmd-guard`, `format-on-write`, `post-compact`, `pre-compact`, and `session-cost-logger`
- `Executive` output style — bottom-line-first, decision-ready — and a `settings.json` that sets it as the default, wires every hook event, and enables session cleanup
- base-ui — a no-dependency glassmorphism CSS design system for AI-generated HTML, bundled with the `frontend` skill
- `CLAUDE.md` harness entry point: format routing table, on-demand skill, agent, and rule loading, rule-authority governance, and 30-day staleness checks that refresh artifacts through `sota`
- TOON skill and agent indexes (`skills.index.toon`, `agents.index.toon`) with a many-to-many skill-to-agent mapping
- `PROJECT.md` architecture spec, bilingual `README.md` / `README.de.md`, and an MIT `LICENSE`
