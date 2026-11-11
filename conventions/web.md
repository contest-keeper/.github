# TypeScript and Svelte

Used by the Hub dashboard, the extension and the Forge window.

## Tooling

- Package manager: **pnpm**. Commit `pnpm-lock.yaml`; CI runs `pnpm install --frozen-lockfile`.
- Node: current LTS, written in `.nvmrc`.
- Svelte 5 with runes, TypeScript in strict mode (`"strict": true`, `"noUncheckedIndexedAccess": true`).
- Prettier formats, ESLint lints, `svelte-check --fail-on-warnings` type-checks. Configs in `templates/web/`.
- Every web project has the same scripts: `dev`, `build`, `format`, `lint`, `check`, `test`.

## Layout

- `src/core/` - pure logic with no browser, DOM or network access. Easy to test.
- `src/lib/` or `src/browser/` - adapters: HTTP, storage, browser APIs.
- `src/components/` - reusable Svelte components; `src/routes/` or `src/pages/` - screens.
- File names: `kebab-case.ts`; Svelte components: `PascalCase.svelte`; rune modules: `name.svelte.ts`.

## Code

- No `any`. Use `unknown` and narrow it. Validate data from the network at the boundary (e.g. with `zod` or the generated API types).
- `type` for data shapes, `interface` only when something implements it.
- Prefer `const`, small functions and early returns. No default exports except where a tool requires them.
- API types are generated from the Hub's OpenAPI spec, never written by hand.
- Async: always handle the error path; show the user something useful, log the details.

## Svelte

- One component per file, under ~200 lines; split when it grows.
- State with runes (`$state`, `$derived`, `$effect`); keep `$effect` rare and side-effect only.
- Props typed with `let { ... }: Props = $props()`.
- Styles scoped in the component. Shared colours and spacing come from CSS variables in one theme file.
- Accessible by default: labels on inputs, buttons for actions (not clickable `div`s), keyboard works, enough contrast.
- No user text built with string concatenation; keep UI strings in one place so they can be translated later.
