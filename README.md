# next-codemod-pnpm-repro

Reproduction for `next-lint-to-eslint-cli` hardcoding `npx` in pnpm projects.

## Reproduce

```bash
pnpm install
pnpx @next/codemod@canary next-lint-to-eslint-cli . --force
```
