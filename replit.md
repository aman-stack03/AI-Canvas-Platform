# Workspace

## Overview

pnpm workspace monorepo using TypeScript. Each package manages its own dependencies.

## Stack

- **Monorepo tool**: pnpm workspaces
- **Node.js version**: 24
- **Package manager**: pnpm
- **TypeScript version**: 5.9
- **API framework**: Express 5
- **Database**: PostgreSQL + Drizzle ORM
- **Validation**: Zod (`zod/v4`), `drizzle-zod`
- **API codegen**: Orval (from OpenAPI spec)
- **Build**: esbuild (CJS bundle)
- **Auth**: Replit Auth (OIDC with PKCE)
- **AI**: Replit AI Integrations (OpenAI — gpt-image-1 for image generation)

## Structure

```text
artifacts-monorepo/
├── artifacts/              # Deployable applications
│   ├── api-server/         # Express API server
│   └── imagegen/           # React + Vite SaaS frontend
├── lib/                    # Shared libraries
│   ├── api-spec/           # OpenAPI spec + Orval codegen config
│   ├── api-client-react/   # Generated React Query hooks
│   ├── api-zod/            # Generated Zod schemas from OpenAPI
│   ├── db/                 # Drizzle ORM schema + DB connection
│   ├── replit-auth-web/    # useAuth() hook for browser auth
│   └── integrations-openai-ai-server/  # OpenAI image generation
├── scripts/                # Utility scripts
├── pnpm-workspace.yaml
├── tsconfig.base.json
├── tsconfig.json
└── package.json
```

## Features

- **Authentication**: Replit Auth (OIDC) — signup/login via `/api/login`
- **Dashboard**: Protected page with prompt input + image gallery
- **Image Generation**: POST `/api/images/generate` (prompt → base64 PNG via gpt-image-1)
- **History**: GET `/api/images/history` (paginated user generation history)

## Database Schema

- `sessions` — Replit Auth sessions (required)
- `users` — User accounts (upserted on auth)
- `image_generations` — Generated images per user (prompt + base64 imageUrl + timestamps)

## API Routes

- `GET /api/healthz` — health check
- `GET /api/auth/user` — current auth state
- `GET /api/login` — start OIDC login
- `GET /api/callback` — OIDC callback
- `GET /api/logout` — logout
- `POST /api/images/generate` — generate AI image (auth required)
- `GET /api/images/history` — user's image history (auth required)

## TypeScript & Composite Projects

Every package extends `tsconfig.base.json` which sets `composite: true`. The root `tsconfig.json` lists all packages as project references.

- **Always typecheck from the root** — run `pnpm run typecheck`
- **`emitDeclarationOnly`** — only emit `.d.ts` files during typecheck
- **Project references** — when package A depends on package B, A's `tsconfig.json` must list B in its `references` array

## Root Scripts

- `pnpm run build` — runs `typecheck` first, then recursively runs `build`
- `pnpm run typecheck` — runs `tsc --build --emitDeclarationOnly`

## Environment Variables

- `DATABASE_URL`, `PGHOST`, etc. — PostgreSQL connection (auto-set by Replit)
- `AI_INTEGRATIONS_OPENAI_BASE_URL`, `AI_INTEGRATIONS_OPENAI_API_KEY` — AI integration (auto-set by Replit)
- `REPL_ID` — used by Replit Auth OIDC client
- `PORT` — set per service by Replit
