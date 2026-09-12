# Claude-Agent

Guidance for Claude Code when working in this repository.

## Project overview

A home for Claude Code workflows — skills that encode a repeatable process so
it runs the same way every time, rather than being re-explained each session.

## Layout

```
.claude/skills/          workflow skills, one directory each
  research-agent/        SKILL.md + references/
output/                  generated reports, as paired .md and .docx
```

## Workflows

- **`/research-agent <topic>`** — gated research workflow. Clarifies scope,
  audience, depth and angle; presents a plan inline for approval; researches
  only once approved; saves a structured report to `output/` as Markdown plus
  a formatted .docx.

## Conventions

- Skills live at `.claude/skills/<name>/SKILL.md` with `name` and `description`
  frontmatter. The directory name is the slash command.
- Keep `SKILL.md` to the process. Long templates and reference material go in
  the skill's `references/` subdirectory.
- Generated output is committed, not gitignored — sessions run in ephemeral
  containers and uncommitted work is lost when they are reclaimed.

## Notes for Claude

- Keep this file current; it is the first thing read when starting work here.
- Commit anything worth keeping before the session ends.
