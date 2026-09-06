# next-codemod-pnpm-repro

Minimal reproduction for [`next-lint-to-eslint-cli`](https://github.com/vercel/next.js/blob/canary/packages/next-codemod/transforms/next-lint-to-eslint-cli.ts) hardcoding `npx` when running `@eslint/migrate-config`, even in pnpm projects.

This repo is the result of the steps below. Next.js **14.x** scaffolds legacy `.eslintrc.json` (not flat `eslint.config.mjs`), which is required to hit the migrate-config code path.

## Codemod version tested

`@next/codemod@canary` → **16.4.0-canary.18** (verified 2026-09-06)

## Reproduce (step by step)

1. Create a Next.js 14 app with pnpm and ESLint enabled:

   ```bash
   mkdir next-codemod-pnpm-repro && cd next-codemod-pnpm-repro
   pnpm dlx create-next-app@14.2.18 . \
     --ts --eslint --app --no-tailwind --no-src-dir \
     --import-alias "@/*" --use-pnpm
   ```

2. Confirm the project matches the bug prerequisites:

   ```bash
   test -f pnpm-lock.yaml && echo "pnpm-lock.yaml OK"
   test -f .eslintrc.json && echo ".eslintrc.json OK"
   grep '"lint": "next lint"' package.json
   ```

   You should see a legacy `.eslintrc.json` (not `eslint.config.mjs`) and a `pnpm-lock.yaml`.

3. Initialize git and commit (the codemod requires a clean working tree):

   ```bash
   git init
   git add -A
   git commit -m "Initial Next.js 14.2.18 pnpm repro"
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
