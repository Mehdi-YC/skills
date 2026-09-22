# Agent workflow

Four-beat workflow, each backed by a skill from this collection. Drop into a repo as `AGENTS.md` and fill in repo-specific callouts.

Architecture: default Django/Odoo project structure for those frameworks; feature-based + hexagonal (when needed) for everything else.

## Workflow

1. **Isolate — `/new-feature`.** Fresh Git worktree from `origin/main`. Never build on `main`.
2. **Build — `/code-structure`.** Service-layer architecture: actions/boundaries own "why/when", service layer owns reusable "how" with explicit inputs and structured returns.
3. **Prove — `/evidence-driven-testing`.** Repo checks + runtime evidence. Capture **before** state while reproducing (before fixing), **after** once the change works.
4. **Ship — `/before-and-after`, then `/greploop`.** PR with before/after proof (screenshot/video for visible changes; measured numbers/output pairs otherwise). Run `/greploop` (or `/greploop-apps` for large PRs) until **5/5 with zero unresolved comments**. Present the PR URL.

Ship-beat notes:
- `--markdown` uploads a pair and prints a PR-ready table; accepts existing PNGs for reuse.
- Containers/VMs: set `AGENT_BROWSER_ARGS="--no-sandbox"` when Chrome fails with "No usable sandbox".
- Default upload host (0x0.st) is public; pass `--upload-url` for sensitive content.

## Writing for humans

Run `/unslop` on anything a person reads before committing, posting, or sending: commit messages, PR titles/bodies, docs, code comments, replies. Strips AI tells, adds voice. Apply only to text you wrote or changed.

## Multi-agent rules

- Never commit to `main`.
- One worktree + one branch per task per agent. Never reuse another agent's worktree, branch, or uncommitted work.
- **Scope check** first: skim open PRs' changed files (`gh pr list`, `gh pr diff <n> --name-only`) and uncommitted work. On overlap, stop and ask.
- Never `--force` anywhere; only `--force-with-lease` on your own task branch.
- Resolve lockfile conflicts by regenerating, never hand-merging.
- Worktrees don't isolate shared resources (ports, databases, lockfiles). Confirm before trusting.
- If a conflict can't be resolved confidently, stop and report.

## Completing a task

1. Keep changes limited to the assigned task.
2. Run repo checks *(list exact commands here)*.
3. Assemble before/after evidence pairs.
4. Commit with clear message, rebase onto `origin/main`, rerun checks.
5. Push (`git push -u origin <branch>`; after rebase: `--force-with-lease`).
6. Open PR. Body: what changed, how tested (evidence-backed), before/after proof, risks/follow-ups. Run through `/unslop` before posting.
7. Run `/greploop` until **5/5 with zero unresolved comments**.
8. Present the PR URL.

Do not merge unless instructed. Keep worktree until PR is merged or closed.

## Repo-specific sections

Append: commands & checks, hard invariants (security/architecture), environment reference, local test infrastructure (stubs/fixtures), anything untestable locally.

## Skill sources

| Skill | Source |
|---|---|
| `new-feature`, `code-structure`, `evidence-driven-testing` | this repo |
| `before-and-after` | this repo, vendored from [vercel-labs/before-and-after](https://github.com/vercel-labs/before-and-after) (or `npx skills add vercel-labs/before-and-after`) |
| `greploop` | this repo, vendored from [greptileai/skills](https://github.com/greptileai/skills) |
| `greploop-apps` | this repo (local variant for huge PRs) |
| `unslop` | this repo, vendored from [cursor/plugins (pstack)](https://github.com/cursor/plugins/tree/main/pstack/skills/unslop); frontmatter edited for auto-invocation |
