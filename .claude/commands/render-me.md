---
description: Render a single component in isolation for styling/testing
argument-hint: <ComponentName or path>
---

You are setting up isolated rendering for a single component so the developer can
style and test it on its own. The component to isolate is: **$ARGUMENTS**

Follow these steps exactly.

## 0. Guard against nested renders

If `.render-me/state.json` already exists, STOP. Tell the developer that a component
is already isolated and they must run `/unrender-me` first before isolating another.
Do not proceed.

## 1. Locate the component

Resolve `$ARGUMENTS` to a single source file under `src/`:

- If it's a path, use it directly.
- If it's a component name, search `src/` (e.g. for `export const <Name>` /
  `function <Name>` / a file named `<Name>.tsx`).
- If multiple matches are plausible, list them and ask the developer which one.

Read the file and identify everything it needs to render. Look for:

- **Props** — its `Props` interface/type and which fields are required.
- **Hooks that need context:**
  - `useFormContext<T>()` (react-hook-form) → needs a `FormProvider`.
  - `useScreenFlowOverride()` → reads `useOutletContext()`, needs a `MemoryRouter`
    + `<Outlet context={...}>` (NOT a context provider).
  - `useTranslation()` / `useHostShell()` → already satisfied by the root providers
    in `src/main.tsx` (do not re-wrap these).
  - `useQuery` / `useMutation` → already satisfied by the root `QueryClientProvider`.
- Any local component-specific context providers it expects.

Note: `src/main.tsx` wraps `<App />` in `HostShellProvider` → `UmtbI18nProvider` →
`UmtbThemeProvider` → `QueryClientProvider`. You are ONLY editing `src/App.tsx`, so
all of these keep wrapping the isolated component automatically. Your sandbox only
adds the extra, component-specific wrappers identified above.

## 2. Snapshot + backup

- Read and remember the full current contents of `src/App.tsx` (this is the backup).
- Create a git snapshot of the current working tree WITHOUT touching the dev's
  working tree or their `git stash` list. Use:
  ```
  git stash create
  ```
  If that returns empty (clean tree), fall back to:
  ```
  git add -A && git write-tree && git reset
  ```
  Capture the resulting commit/tree SHA — call it `snapshotSha`.

## 3. Generate the sandbox

Create `src/__render-me__/Sandbox.tsx`. It must:

- Import the real component by its path alias (`@features/...`, etc.) — never copy or
  edit the real component.
- Wrap it with ONLY the extra providers it needs (from step 1):
  - **Form:** if it uses `useFormContext<T>`, set up
    `const methods = useForm<T>({ defaultValues: <hardcoded sensible mock> })` and
    wrap in `<FormProvider {...methods}>`. Infer defaults from the form type (e.g.
    `{ step1: { startDate: <iso>, endDate: <iso> } }`).
  - **Screenflow:** if it uses `useScreenFlowOverride`, wrap in `<MemoryRouter>` with
    a route whose element renders `<Outlet context={mock} />`, where `mock` is
    `{ provideNextRequest: () => {}, providePrevRequest: () => {}, reset: () => {}, nextResponses: [], overrideButtons: undefined }`.
    The component renders as the child route so its `useOutletContext` resolves.
  - **Props:** if it takes props, pass hardcoded mock props matching its `Props`
    interface (realistic values so it renders meaningfully).
- Default-export a component named `RenderMeSandbox`.

Keep the sandbox minimal and readable — the developer may tweak the mock values; it
is discarded on `/unrender-me`.

## 4. Swap `App.tsx`

Edit `src/App.tsx` so it renders `<RenderMeSandbox />` instead of
`<MfeYieldsAndBalances ... />`. Keep the dev-only language toggle block intact.
Wrap the swapped region in marker comments as a fallback restore locator:

```tsx
{/* RENDER-ME-START */}
<RenderMeSandbox />
{/* RENDER-ME-END */}
```

(Use `// RENDER-ME-START` style if the swap is outside JSX.)

## 5. Write metadata

Create `.render-me/state.json`:

```json
{
  "component": "<resolved path>",
  "createdFiles": ["src/__render-me__/Sandbox.tsx"],
  "appTsxBackup": "<full original App.tsx contents>",
  "snapshotSha": "<snapshotSha>",
  "createdAt": "<ISO timestamp>"
}
```

## 6. Verify and hand off

Run `pnpm build` to confirm it compiles (this repo uses `pnpm build` for TS checks).
Report any errors and fix sandbox issues if they're yours. When green, tell the
developer:

> Isolated **<Component>**. Run `pnpm dev` and open http://localhost:4008.
> When done, run `/unrender-me` to restore — your edits to the component are kept.
