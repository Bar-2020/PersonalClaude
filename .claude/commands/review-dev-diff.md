# Review Dev Diff

Code review the current branch's diff against the latest `origin/development`, as if reviewing an open PR.

## Steps

1. Sync with remote and install dependencies (run in parallel):
   - `git fetch origin development`
   - `pnpm i`

2. Confirm there are no uncommitted local changes that would muddy the diff. If `git status` is not clean, surface this to the user and ask whether to proceed.

3. Gather the review inputs (run in parallel):
   - `git log origin/development..HEAD --oneline` — commits on this branch
   - `git diff origin/development...HEAD --stat` — files changed summary
   - `git diff origin/development...HEAD` — the full diff to review
   - Current branch name via `git rev-parse --abbrev-ref HEAD`

4. Review the diff as a thorough PR reviewer would. Cover:
   - **Correctness** — logic errors, off-by-ones, unhandled edge cases, race conditions, error handling gaps.
   - **Security** — injection risks, unsafe input handling, secrets, authz/authn issues, OWASP-style concerns.
   - **Architecture & design** — adherence to the codebase's existing patterns, separation of concerns, models/types/interfaces placement, unnecessary abstractions or premature generalization.
   - **Readability & maintainability** — naming, dead code, commented-out code, comments that explain *what* instead of *why*, missing TODO refs in comments.
   - **Project conventions** — whatever the repo itself sets in its `CLAUDE.md`, lint config and existing code (import paths and aliases, design tokens, accessibility rules).
   - **Testing** — missing or weak test coverage for new behavior, brittle tests, mocked things that should be integration-tested.
   - **Performance** — needless re-renders, missing memoization where it matters, N+1 query patterns, large bundle additions.
   - **Diff hygiene** — unrelated changes bundled in, formatting-only churn obscuring real changes, leftover debug logs.

5. For each finding, report:
   - `file_path:line_number` reference so the user can jump to it
   - Severity: **blocker** / **major** / **minor** / **nit**
   - What's wrong and a concrete suggestion to fix it

6. Group findings by severity, blockers first. End with a short overall verdict (approve / request changes / comment) and a one-line summary of the change set.

7. Do **not** modify any code as part of this command — this is review-only.
