# Global development rules

Personal defaults for every project. Copy this file to `~/.claude/CLAUDE.md` to apply it on a
machine. Cloud sessions only see repo files, so a repo that needs these rules mirrors the
relevant parts in its own `CLAUDE.md`.

A project's own `CLAUDE.md` adds to these. If the two conflict, raise it with the user instead of
picking a side silently.

Distilled from my own projects. Only rules that are not tied to one product are here.

## How to work

1. **Think before coding.** State assumptions. If there is more than one reading, show them. If
   something is unclear, stop and ask.
2. **Simplest thing that works (ponytail).** Does it need to exist? Reuse what is in the repo,
   then the standard library, then the native platform, then an installed dependency, then new
   code. No feature, abstraction or option nobody asked for.
3. **Surgical changes.** Every changed line traces to the request. Mention unrelated dead code,
   never delete it. Remove only what your own change orphaned. Match the existing style.
4. **Goal driven.** Turn the task into a check and loop until it passes. For a bug, reproduce it
   first, then fix it.
5. **The spec wins over the code.** If the spec disagrees with itself, or a decision is missing,
   stop and ask.

## Before saying anything is done

- Run the project's `npm run verify` (typecheck, lint, tests, build) and show the output. If the
  project has no `verify` script, run each of those checks and say so.
- Every convention gets a mechanical check (a lint rule, a test or a hook) whose error message
  names the fix. A rule that only lives in this file is forgotten in a long session.

## Before opening a PR

Open a PR only when the user asks for one. Then:

1. Run `/ponytail-review` (over-engineering) and `/code-review` (correctness) on the branch diff.
2. Merge both outputs into one list. Number every finding `[R1]`, `[R2]`, ... and list every
   point, nits included. For each give the file and line, what is wrong, a concrete fix, and
   which review raised it.
3. Show the list and wait for the reply ("fix R1 and R3, skip R2"). If nobody can reply, say so
   and stop. Resolve each finding as the reply says, then run verify.
4. Open the PR once no finding is left open. If both reviews find nothing, say so in one line
   and open it.

The `ponytail` skills are not built in. Copy `ponytail` and `ponytail-review` from
`Bar-2020/silent-whisker/.claude/skills/` into `~/.claude/skills/`. If they are missing, say so and
ask before skipping the step.

## Code rules

**Styles**
- **No inline styles.** No `style={{ ... }}` object literals in markup and no `element.style`.
  Styles live in a separate file: a `.css` file beside the component (CSS Modules, preferred) or
  a typed `styles.ts` object used as `style={styles.x}`. With CSS Modules, vary a style by
  switching a class or a `data-*` attribute. A project's own `CLAUDE.md` may tighten this to ban
  the `style` prop entirely, as silent-whisker does.
- **RTL-safe.** Use logical properties (`margin-inline-start`, `inset-inline-end`,
  `text-align: start`), never `left` or `right`.
- **Mobile first.** Nothing scrolls sideways or is cut off from 360 px up. Breakpoints and design
  tokens are written once, in the styles folder.

**Types and constants**
- **TypeScript strict.** No `any`. No `@ts-ignore` to get past a real type error.
- **Constants live in `src/constants/`.** No magic numbers in logic: a numeric literal other than
  0, 1 or -1 is a lint error.
- **Localized projects:** every user-facing string goes through `t('key')`. Nothing hardcoded.

**Data and backend**
- **Timestamps are stored in UTC.** Convert at display time only.
- **No hard deletes of user content.** Change a status instead.
- **The server enforces every rule.** Client checks are UX, never enforcement. A rule is done when
  the server refuses to break it, not when the button is disabled.

**Structure**
- **Feature-based folders.** `src/features/<name>/` owns its screens, components, hooks and data
  access. Features never import from each other.
- **Shared code moves to `src/shared/`** when a second feature needs it, not in anticipation.

**Commits**
- Conventional Commits. One logical change per commit, small PRs.
