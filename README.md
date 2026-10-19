# PersonalClaude

My personal [Claude Code](https://claude.com/claude-code) setup: a global development ruleset and a
few slash commands.

## What is in here

| File | What it is |
|---|---|
| [`.claude/CLAUDE.md`](.claude/CLAUDE.md) | The global development rules: how to work, what to run before saying something is done, the review steps before opening a PR, and the code rules (no inline styles, strict TypeScript, and so on). |
| [`/review-dev-diff`](.claude/commands/review-dev-diff.md) | Review-only code review of the current branch against `origin/development`, as if it were an open PR. Findings are grouped by severity with `file:line` references. |
| [`/render-me <component>`](.claude/commands/render-me.md) | Temporarily swaps `src/App.tsx` for a sandbox that renders one component on its own, with mock props and providers, so you can style it in isolation. |
| [`/unrender-me`](.claude/commands/unrender-me.md) | Undoes `/render-me` and keeps every real edit you made to the component. |
| [`/extract-inline-styles [file]`](.claude/commands/extract-inline-styles.md) | Moves inline `style={{...}}` props out of a file into a typed `styles.ts`. |

## Use it

Claude Code reads user-level files from `~/.claude`, so copy what you want there:

```bash
git clone https://github.com/Bar-2020/PersonalClaude.git
cd PersonalClaude

cp .claude/CLAUDE.md ~/.claude/CLAUDE.md          # global rules, applied to every project
mkdir -p ~/.claude/commands
cp .claude/commands/*.md ~/.claude/commands/      # slash commands, available as /name
```

If you already have a `~/.claude/CLAUDE.md`, merge the two by hand instead of overwriting it.

The ruleset's PR step uses the `ponytail` and `ponytail-review` skills, which are not part of
Claude Code and are not in this repo. A copy lives in
[`Bar-2020/silent-whisker`](https://github.com/Bar-2020/silent-whisker/tree/main/.claude/skills).

## The commands are written for one codebase

The four commands were written against a specific React, TypeScript and pnpm project, and they
assume things about it: a `src/features/` layout with a fixed set of subfolders, a `development`
base branch, `pnpm` scripts, a dev server on port 4008, and a particular set of shared UI packages
and providers. In another project they will need adapting before they are useful.

`/review-dev-diff` also mentions a `/fix-structure` command that is not in this repo.

The ruleset in `.claude/CLAUDE.md` is generic and has no such assumptions.
