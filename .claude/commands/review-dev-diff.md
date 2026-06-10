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

4. **Structure audit of changed paths** — for any feature touched in the diff:
   - Extract feature names: `git diff origin/development...HEAD --name-only | grep '^src/features/' | cut -d'/' -f3 | sort -u`
   - For each feature directory found, run `ls src/features/<name>/` and check:
     - All 8 required subdirs are present: `containers`, `main`, `titles`, `constants`, `enums`, `interfaces`, `tests`, `ui`
     - No extra directories beyond those 8
     - No loose files directly in the feature root
     - Key files exist: `containers/<PascalName>PageContainer.tsx`, `main/<PascalName>Screen.tsx`, `main/styles.ts`, `titles/<PascalName>Title.tsx`, `titles/styles.ts`, `interfaces/translations.interface.ts`
   - Flag any diff changes touching **locked** (template-controlled) files — i.e., paths NOT in this editable list: `src/features/**`, `src/api/**`, `src/data/**`, `src/models/**`, `src/infrastructure/routing/AppRoutesConfig.tsx`, `src/infrastructure/routing/routing-paths.ts`, `src/infrastructure/routing/sub-routes/**`, `VERSION.txt`, `public/**`
   - Include all structure findings in the grouped output under a **Structure** category
   - If violations found, suggest running `/fix-structure` to auto-remediate

5. Review the diff as a thorough PR reviewer would. Cover:
   - **Correctness** — logic errors, off-by-ones, unhandled edge cases, race conditions, error handling gaps.
   - **Security** — injection risks, unsafe input handling, secrets, authz/authn issues, OWASP-style concerns.
   - **Architecture & design** — adherence to existing patterns (e.g. QueryRenderer, dual router, TanStack Query factories), separation of concerns, models/types/interfaces placement, unnecessary abstractions or premature generalization.
   - **Readability & maintainability** — naming, dead code, commented-out code, comments that explain *what* instead of *why*, missing TODO refs in comments.
   - **Project conventions** — import paths (no `../../`, use aliases), no hex colors (design tokens), no `margin` CSS, union-only type aliases, curly braces on all control statements, accessibility rules.
   - **Testing** — missing or weak test coverage for new behavior, brittle tests, mocked things that should be integration-tested.
   - **Performance** — needless re-renders, missing memoization where it matters, N+1 query patterns, large bundle additions.
   - **Diff hygiene** — unrelated changes bundled in, formatting-only churn obscuring real changes, leftover debug logs.

6. For each finding, report:
   - `file_path:line_number` reference so the user can jump to it
   - Severity: **blocker** / **major** / **minor** / **nit**
   - What's wrong and a concrete suggestion to fix it

7. Group findings by severity, blockers first (include **Structure** violations here). End with a short overall verdict (approve / request changes / comment) and a one-line summary of the change set.

8. Do **not** modify any code as part of this command — this is review-only. If the user wants structure fixes applied, direct them to run `/fix-structure`.
