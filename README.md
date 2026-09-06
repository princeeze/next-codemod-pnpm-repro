# next-codemod-pnpm-repro

Minimal reproduction for [`next-lint-to-eslint-cli`](https://github.com/vercel/next.js/blob/canary/packages/next-codemod/transforms/next-lint-to-eslint-cli.ts) hardcoding `npx` when running `@eslint/migrate-config`, even in pnpm projects.

## What this repo is

- Next.js **14.2.18** (legacy `.eslintrc.json`, not flat config)
- Package manager: **pnpm** (`pnpm-lock.yaml`)
- `package.json` script: `"lint": "next lint"`

## Codemod version tested

`@next/codemod@canary` → **16.4.0-canary.18** (verified 2026-09-06)

## Reproduce (step by step)

1. Clone this repo:

   ```bash
   git clone https://github.com/princeeze/next-codemod-pnpm-repro.git
   cd next-codemod-pnpm-repro
   ```

2. Install dependencies:

   ```bash
   pnpm install
   ```

3. Ensure a clean git working tree (the codemod requires it):

   ```bash
   git status
   ```

4. Run the published codemod with pnpm:

   ```bash
   pnpx @next/codemod@canary next-lint-to-eslint-cli . --force
   ```

5. Check the terminal output for the migrate step.

## Expected vs actual

| | Command |
|---|---|
| **Actual** | `npx @eslint/migrate-config .eslintrc.json` |
| **Expected** (pnpm project) | `pnpm --silent dlx @eslint/migrate-config .eslintrc.json` |

The codemod is invoked via `pnpx` (pnpm), but the legacy ESLint config migration step still logs and runs `npx`.

## Observed output

```
Migrating from next lint to the ESLint CLI...
   Found legacy ESLint config: .eslintrc.json
   Running "npx @eslint/migrate-config /path/to/next-codemod-pnpm-repro/.eslintrc.json" to convert legacy config...
   Found existing ESLint Flat config: eslint.config.mjs
   Updated eslint.config.mjs to use direct eslint-config-next imports
⚠    Config does not export an array or supported pattern. Manual migration required.
   Could not automatically update the existing flat config.
   Please manually ensure your ESLint config includes the Next.js configurations
   Updated script "lint": "next lint" → "eslint ."
Updated package.json scripts and dependencies

Migration complete! Your project now uses the ESLint CLI.
```

The important line is the `npx @eslint/migrate-config` log — that is the bug.

## Related (out of scope)

- [vercel/next.js#85587](https://github.com/vercel/next.js/issues/85587) — `npm view` fails when `devEngines.packageManager` is pnpm during upgrade
- [vercel/next.js#85679](https://github.com/vercel/next.js/issues/85679) — different `next-lint-to-eslint-cli` failure (`.eslintrc.cjs` output extension)
