# Repro: Vue SFC `lang="tsx"` fails in dev when plugin-react is present

A Vue SFC whose `<script setup lang="tsx">` contains TypeScript syntax
(`import type`, annotations, generics) fails to compile **in dev only**
when `@vitejs/plugin-react` (oxc backend) is registered alongside
`@vitejs/plugin-vue`. The error is thrown by the core `vite:oxc` plugin,
which appears to parse the virtual module `App.vue?vue&type=script&setup=true&lang.tsx`
with JSX enabled but TypeScript disabled.

## Run

```sh
bun install        # or npm/pnpm/yarn install
bunx vite dev      # or npx vite dev
```

Then open http://localhost:5173/ — or request the compiled script module directly:

```sh
curl "http://localhost:5173/App.vue?vue&type=script&setup=true&lang.tsx"
```

## Actual

```
Transform failed with 1 error:

[PARSE_ERROR] Expected `from` but found `{`
   ╭─[ App.vue?vue&type=script&setup=true&lang.tsx:2:13 ]
   │
 2 │ import type { Props } from './types'
   │             ┬
   │             ╰── `from` expected
───╯
```

plugin: `vite:oxc` · dev only (`vite build` succeeds) · `lang="ts"` SFCs and
real `.tsx` files are unaffected · removing the react plugin fixes it.

## Versions

vite 8.3.2 · @vitejs/plugin-react 6.1.1 · @vitejs/plugin-vue 5.2.4 · vue 3.5

Also reproduces on `rolldown-vite@7.3.1` + `@vitejs/plugin-react-oxc@0.4.3`.

## Workaround

See the linked issue for a small `enforce: 'pre'` / `apply: 'serve'` esbuild shim.
