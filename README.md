# Skills

A collection of [agent skills](https://code.claude.com/docs/en/skills) for Claude Code. Each skill is a folder with a `SKILL.md` (frontmatter + instructions) that Claude loads on demand.

[![skills.sh](https://skills.sh/b/michaelshimeles/skills)](https://skills.sh/michaelshimeles/skills)

## Available skills

### [before-and-after](before-and-after/SKILL.md)

Before/after screenshots or metrics comparison. Drives the `@vercel/before-and-after` CLI.

> Vendored from [vercel-labs/before-and-after](https://github.com/vercel-labs/before-and-after) (PolyForm Shield 1.0.0). Install: `npm i -g @vercel/before-and-after agent-browser`.

### [code-structure](code-structure/SKILL.md)

Service layer architecture: actions orchestrate domain rules ("why/when"), service layer centralizes reusable mechanics ("how").

### [evidence-driven-testing](evidence-driven-testing/SKILL.md)

Records annotated screen capture while testing UI behavior, posts video + results to PR/issue. Supports screen recording (Linux/macOS/Windows) and headless paths (Playwright screenshots). Recorder: `scripts/evidence.py` (Python 3 + FFmpeg). Run `python3 scripts/evidence.py doctor` to check dependencies.

### [new-feature](new-feature/SKILL.md)

Isolated Git worktree per task from `origin/main`. Covers naming, scope check, dependency install, cleanup.

### [unslop](unslop/SKILL.md)

Removes AI tells from prose. 31 patterns to detect, four-step loop: scan, rewrite, add soul, self-audit.

> Vendored from [cursor/plugins (pstack)](https://github.com/cursor/plugins/tree/main/pstack/skills/unslop) (MIT). Frontmatter edited for auto-invocation.

## Workflow

[`AGENTS.md`](AGENTS.md) ties skills into a four-beat workflow: isolate → build → prove → ship. Drop into a repo alongside skills and fill in repo-specific callouts.

## Installation

```bash
npx skills add michaelshimeles/skills
```

Claude Code auto-invokes skills when a task matches. Invoke explicitly with `/code-structure`, `/evidence-driven-testing`, etc.

## Adding a skill

1. Create kebab-case folder.
2. Add `SKILL.md` with `name` and `description` frontmatter (trigger-focused: "Use when...").
3. Keep instructions concise; link to reference files if long.
