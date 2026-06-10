---
description: Undo /render-me scaffolding while keeping the developer's real edits
---

You are tearing down the isolated-render scaffolding created by `/render-me`, while
preserving every real change the developer made afterward (styling, logic fixes).

Follow these steps exactly.

## 1. Read state

Read `.render-me/state.json`. If it does not exist, tell the developer there is
nothing to undo (no active `/render-me` session) and STOP.

Parse: `component`, `createdFiles`, `appTsxBackup`, `snapshotSha`.

## 2. Detect dev edits to scaffolding

Diff the current tree against the snapshot to see if the developer changed any
scaffolding (which is throwaway) vs. real files (which are kept):

```
git diff <snapshotSha> -- src/__render-me__ src/App.tsx
```

- Edits inside `src/__render-me__/` → throwaway. Discard by default (the files are
  deleted in step 4). Briefly mention if there were any.
- Edits to `src/App.tsx` → unexpected (it's the harness). If the dev changed
  `App.tsx` beyond the marked region, show those hunks and ask whether to keep them
  before restoring. Otherwise restore silently.

Do NOT diff the target component or other source files — those edits are the
developer's real work and are always kept.

## 3. Restore `App.tsx`

Restore `src/App.tsx` from `appTsxBackup` in the metadata. If the backup is missing
or the file was heavily modified, fall back to removing the
`RENDER-ME-START`/`RENDER-ME-END` marked region and reinstating the
`<MfeYieldsAndBalances ... />` render. Verify the restored file no longer references
`RenderMeSandbox`.

## 4. Delete scaffolding files

- Delete every path listed in `createdFiles`.
- Remove the `src/__render-me__/` directory if now empty.

## 5. Delete metadata

Delete the `.render-me/` directory (metadata + snapshot bookkeeping).

## 6. Verify and report

Run `pnpm build` to confirm the app still compiles. Then report:

- **Removed:** `src/__render-me__/`, `.render-me/`, the `App.tsx` swap.
- **Kept:** all edits to `<component>` and any other source files.

Confirm the component file and the developer's real changes were left untouched.
