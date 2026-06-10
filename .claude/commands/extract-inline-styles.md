# Extract Inline Styles

Extract all inline `style={{...}}` props from the current file into a `styles.ts` file.

## Steps

1. Read the file provided as `$ARGUMENTS`, or if none given, ask the user which file to extract styles from.

2. Find all inline `style={{ ... }}` occurrences in the file.

3. Check whether the file already lives in a dedicated folder named after itself in kebab-case (e.g. `useAccountTransactionHistory.tsx` inside `use-account-transaction-history/`). If it does not, create that kebab-case folder as a sibling of the file, move the file into it, and update any imports in the codebase that referenced the old path. The `styles.ts` is then created inside that same new folder alongside the moved file.

4. For each inline style found, create a descriptive camelCase key name based on the element or context (e.g. `downloadIconContainer`, `headerWrapper`).

5. Write/update `styles.ts` using this exact pattern — import `NestedCssProperties` from `@umtb/shared-ui-cmp`, name the export after the file (no `use` prefix, even for hooks), and close with `as const satisfies Record<string, NestedCssProperties>`:

```ts
import { NestedCssProperties } from '@umtb/shared-ui-cmp';

export const <fileNameInCamelCase>Styles = {
  <key>: {
    // extracted properties
  },
} as const satisfies Record<string, NestedCssProperties>;
```

6. Replace each inline `style={{ ... }}` in the source file with `style={<exportName>.<key>}`.

7. Add the import for the styles export at the top of the source file (grouped with other local imports). Use the `@features/...` alias path if the file uses it, otherwise use a relative path.

8. Verify no TypeScript diagnostics are introduced.
