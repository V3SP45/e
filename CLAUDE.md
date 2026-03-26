# CLAUDE.md — E-Rechnung Generator

## Commands
- Dev: `pnpm dev`
- Build: `pnpm build`
- Lint: `pnpm lint`
- Test: `pnpm test`
- Test watch: `pnpm test:watch`
- Preview: `pnpm preview`

## Architecture
- React 19 + Vite 6 + TypeScript 5.7 (strict)
- Tailwind CSS 4 + shadcn/ui
- Forms: React Hook Form + Zod
- i18n: react-i18next (en base, de, fr, nl)
- E-Invoice XML: @e-invoice-eu/core (100% client-side)
- PDF/A-3: Serverless function (`api/pdf.ts`)
- No database, no auth, no server-side data storage
- Seller data optionally in localStorage (opt-in)

## Code Style
- TypeScript strict, no `any`
- Named exports only, no default exports
- Functional components only
- Zod schemas = single source of truth
- Files: kebab-case. Components: PascalCase
- Imports: `@/` alias → `src/`
- All UI text via i18n `t()` — never hardcoded
- Country-specific logic via CountryConfig — never hardcoded in core

## Structure
- `src/lib/` — Pure logic (schema, calculations, XML generation)
- `src/data/countries/` — Per-country config (VAT rates, formats, validation)
- `src/i18n/locales/` — Translation JSON files
- `src/components/` — React components
- `src/components/ui/` — shadcn/ui base
- `src/hooks/` — Custom hooks
- `api/` — Vercel Serverless Functions

## Country System
Each country = one TypeScript file in `src/data/countries/{code}.ts`.
Interface: `CountryConfig` (code, currency, locale, invoiceFormat, vatRates, vatIdFormat, postalCodeFormat).
Adding a country requires no core logic changes.

## Supported Formats
- XRechnung CII — Germany
- Factur-X — France
- Peppol BIS UBL — Nordics, Benelux, Baltics, Ireland, etc.

## 19 Launch Countries
DE, AT, CH, FR, NL, BE, LU, DK, SE, FI, NO, EE, LV, LT, IE, SI, HR, SK, CZ

## Boundaries
- NEVER store user data server-side
- NEVER use `any` type
- NEVER import `node:*` in client code
- NEVER hardcode UI strings or country logic
- ASK before adding dependencies
- ASK before changing Zod schemas or API contracts
