# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## ⚠️ This is not the Next.js you were trained on

This repo pins `next@16.2.10` / `react@19.2.4` / `react-dom@19.2.4` — a version newer than your training data, with breaking API/convention changes. **Before writing or editing any Next.js code, read the relevant guide under `node_modules/next/dist/docs/01-app/`** (App Router docs; ignore the parallel `02-pages/` tree unless the task is Pages Router). Do not assume training-era Next.js conventions apply. Docs of note already confirmed to differ from older Next.js:

- **`middleware.ts` no longer exists — it's `proxy.ts`** now (`middleware()` export is gone; see `01-getting-started/16-proxy.md` and `03-api-reference/03-file-conventions/proxy.md`). The glossary entry for "Middleware" just redirects to "Proxy".
- **Turbopack is the default bundler** for both `next dev` and `next build`, not just dev.
- **Cache Components / `"use cache"` directive** (`cacheLife()`, `cacheTag()`, `updateTag()`) is the current caching/revalidation model — check `01-getting-started/08-caching.md` and `03-api-reference/04-functions/cacheLife.md` before assuming old ISR (`revalidate` export) patterns still apply as-is.
- There's a dedicated `01-app/02-guides/ai-agents.md` guide — check it if a task involves building agent/AI-backed routes.

When in doubt about any App Router file convention (`route.js`, `proxy.js`, metadata files, parallel/intercepted routes, etc.), grep `node_modules/next/dist/docs/01-app/` rather than relying on memory.

## Project

Arcade Vault — a platform for playing games online and competing for high scores/points. Per `README.md`, the intended workflow is **Spec Driven Design** based on `/spec` and `/spec-impl` conventions from the `Klerith/fernando-skills` skill pack (installed via `npx skills@latest add Klerith/fernando-skills`). Those skill/spec directories are not present in the repo yet — the codebase is currently the unmodified output of `create-next-app` (single home page, no custom routes, components, or data layer built out yet).

## Commands

```
npm run dev      # start dev server (Turbopack)
npm run build    # production build (Turbopack)
npm run start    # run production build
npm run lint     # ESLint (flat config, eslint.config.mjs)
```

There is no test runner configured yet.

## Stack & conventions

- **App Router only** — all routes live under `app/`. No `pages/` directory exists.
- **TypeScript strict mode** (`tsconfig.json`), path alias `@/*` → project root.
- **Tailwind CSS v4** via `@tailwindcss/postcss` — imported with `@import "tailwindcss"` in `app/globals.css`, theme tokens (colors, fonts) declared with `@theme inline`, not a `tailwind.config.js`.
- **ESLint 9 flat config** (`eslint.config.mjs`) extending `eslint-config-next` core-web-vitals + typescript presets.
- Fonts are loaded via `next/font/google` (Geist / Geist Mono) and exposed as CSS variables consumed by the Tailwind theme.
