# Extract Inline Styles

Extract all inline `style={{...}}` props from the current file into a `styles.ts` file.

## Steps

1. Read the file provided as `$ARGUMENTS`, or if none given, ask the user which file to extract styles from.

2. Find all inline `style={{ ... }}` occurrences in the file.

3. For each inline style found, create a descriptive camelCase key name based on the element or context (e.g. `downloadIconContainer`, `headerWrapper`).

4. Write/update `styles.ts` next to the source file using this exact pattern — import `CSSProperties` from `react`, name the export after the file (no `use` prefix, even for hooks), and close with `as const satisfies Record<string, CSSProperties>`:

```ts
import type { CSSProperties } from 'react';

export const <fileNameInCamelCase>Styles = {
  <key>: {
    // extracted properties
  },
} as const satisfies Record<string, CSSProperties>;
```

5. Replace each inline `style={{ ... }}` in the source file with `style={<exportName>.<key>}`.

6. Add the import for the styles export at the top of the source file (grouped with other local imports), using the same import style (alias or relative path) as the file's other local imports.

7. Verify no TypeScript diagnostics are introduced.
