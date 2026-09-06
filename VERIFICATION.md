# Verification

Captured 2026-09-06 against published `@next/codemod@canary` on this pnpm Next.js 14.2.18 repo.

## Setup

- Package manager: pnpm (`pnpm-lock.yaml`, `pnpm-workspace.yaml`)
- Legacy ESLint config: `.eslintrc.json`
- `package.json` script: `"lint": "next lint"`
- Git working tree was clean (`main`)

## Commands

```bash
pnpm install
pnpx @next/codemod@canary next-lint-to-eslint-cli . --force
```

## Output

```
Migrating from next lint to the ESLint CLI...
   Found legacy ESLint config: .eslintrc.json
   Running "npx @eslint/migrate-config /Users/admin/Workspace/next-codemod-pnpm-repro/.eslintrc.json" to convert legacy config...
   Found existing ESLint Flat config: eslint.config.mjs
   Updated eslint.config.mjs to use direct eslint-config-next imports
⚠    Config does not export an array or supported pattern. Manual migration required.
   Could not automatically update the existing flat config.
   Please manually ensure your ESLint config includes the Next.js configurations
   Updated script "lint": "next lint" → "eslint ."
Updated package.json scripts and dependencies

Migration complete! Your project now uses the ESLint CLI.
```

## Pass criteria

- Codemod was started with `pnpx` (pnpm).
- The migrate step logged `npx @eslint/migrate-config`, not `pnpm dlx` / `pnpm --silent dlx`.
