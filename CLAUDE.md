# CLAUDE.md — E-Rechnung Generator

## Commands
- Dev: `pnpm dev`
- Build: `pnpm build`
- Lint: `pnpm lint`
- Test: `pnpm test`
- Test watch: `pnpm test:watch`
- Preview: `pnpm preview`

## Architecture
- React 19 + Vite 8 + TypeScript 5.9 (strict)
- Tailwind CSS 4 + shadcn/ui
- Forms: React Hook Form + Zod
- XRechnung: @e-invoice-eu/core (100% client-side)
- ZUGFeRD PDF: node-zugferd via Vercel Serverless (`api/zugferd.ts`)
- No database, no auth, no data storage

## Code Style
- TypeScript strict, no `any`
- Named exports only, no default exports
- Functional components only
- Zod schemas = single source of truth
- Files: kebab-case. Components: PascalCase
- Imports: `@/` alias → `src/`
- UI text in German, code in English

## Structure
- `src/lib/` — Pure logic (schema, calculations, XML generation)
- `src/components/` — React components
- `src/components/ui/` — shadcn/ui base
- `src/hooks/` — Custom hooks
- `api/` — Vercel Serverless Functions
- `specs/` — Module specifications

## Domain
- XRechnung = EN 16931 CII XML for Germany
- ZUGFeRD = PDF/A-3 with embedded XML
- UStG §14 = mandatory invoice fields (DE law)
- VAT: DE 19%/7%, AT 20%/10%/13%, CH 8.1%/2.6%/3.8%

## Boundaries
- NEVER store user data (no DB, no localStorage for invoices)
- NEVER use `any` type
- NEVER import `node:*` in client code
- ASK before adding dependencies
- ASK before changing Zod schemas
