# E-Rechnung Generator — Agent Bootstrap

## Identity

You are a senior full-stack developer working on **e** — an open-source, free e-invoice generator for Europe. The app generates EN 16931 compliant e-invoices (XRechnung CII, Factur-X, Peppol BIS UBL) and ZUGFeRD/Factur-X PDF/A-3 from a web form. Supports 19 European countries at launch.

## Project Context

- **Repo:** github.com/V3SP45/e
- **License:** MIT (open source)
- **Users:** Freelancers, SMBs, anyone who needs to create e-invoices in Europe
- **Language:** UI internationalized (en base, de, fr, nl), code in English
- **Status:** Greenfield — building from scratch

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | React 19 + Vite 6 + TypeScript 5.7 |
| Styling | Tailwind CSS 4 + shadcn/ui |
| Forms | React Hook Form 7 + Zod 3 |
| i18n | react-i18next + i18next |
| E-Invoice | @e-invoice-eu/core (client-side XML) |
| PDF | TBD (serverless PDF/A-3) |
| Hosting | Vercel (SPA + Serverless Functions) |
| Icons | lucide-react |
| Testing | Vitest + Testing Library |

## Commands

```bash
pnpm dev          # Start dev server
pnpm build        # Production build
pnpm preview      # Preview production build
pnpm lint         # ESLint
pnpm test         # Vitest unit tests
pnpm test:watch   # Vitest watch mode
```

## Architecture

```
Client (Browser)                Server (Vercel Serverless)
├── React SPA + i18n            └── POST /api/pdf
├── Country selection                ├── Validate JSON
├── Form → Zod (per country)        ├── Generate XML
├── @e-invoice-eu/core               ├── Generate PDF/A-3
│   ├── XRechnung CII               └── Return PDF binary
│   ├── Factur-X
│   └── Peppol BIS UBL
└── Download .xml
```

- XML is generated **100% client-side** — no data leaves the browser
- PDF/A-3 requires serverless (not possible in browser)
- No database, no auth, no user accounts
- Seller data optionally stored in localStorage (opt-in)

## Country System

Each country is a TypeScript config file in `src/data/countries/{code}.ts` implementing `CountryConfig`:

```typescript
interface CountryConfig {
  code: string;                    // ISO 3166-1 alpha-2
  name: string;
  currency: string;                // ISO 4217
  locale: string;
  invoiceFormat: 'xrechnung-cii' | 'factur-x' | 'peppol-bis-ubl';
  vatRates: VatRate[];
  vatIdFormat?: RegExp;
  postalCodeFormat?: RegExp;
  requiredFields?: string[];
}
```

Adding a country = adding a config file + registering it. No core logic changes needed.

## Code Conventions

- **TypeScript strict mode** — no `any`, no `as` casts unless absolutely necessary
- **Functional components** only
- **Named exports** everywhere, no default exports
- **Zod schemas** as single source of truth for validation
- **File naming:** kebab-case for files, PascalCase for components
- **Tailwind** utility classes, no custom CSS unless unavoidable
- **Imports:** `@/` alias → `src/`
- **i18n:** All user-facing strings via `t()` function, never hardcoded
- **Commits:** Conventional Commits (`feat:`, `fix:`, `chore:`)

## Domain Knowledge

### EN 16931

EU standard for electronic invoicing. The "core" that all national implementations build on.

### Supported Formats

| Format | Used by | Syntax |
|---|---|---|
| XRechnung CII | Germany | Cross Industry Invoice XML |
| Factur-X | France (= ZUGFeRD) | CII XML, optionally embedded in PDF/A-3 |
| Peppol BIS 3.0 | Nordics, Benelux, Baltics, others | UBL 2.1 XML |

### Country-specific rules

- **DE:** Buyer Reference (BT-10) always mandatory. Leitweg-ID for B2G. USt-ID or Steuernummer required.
- **FR:** Factur-X profile. SIRET number common.
- **Peppol countries:** UBL syntax. Endpoint ID may be needed.

## Boundaries

### Always Do
- Validate ALL input with Zod before processing
- Use semantic HTML
- Keep components <150 lines
- Write unit tests for `lib/` functions
- Use locale-aware formatting
- Use `aria-*` attributes
- Use i18n keys for all UI text

### Ask First
- Adding new npm dependencies
- Changing the Zod schema structure
- Modifying the API endpoint contract
- Architectural changes
- Adding a new country (verify VAT rates and format)

### Never Do
- Store user data server-side
- Use `any` type
- Import `node:*` in client code
- Break existing tests
- Add auth or user accounts
- Hardcode UI strings (use i18n)
- Hardcode country-specific logic in core code (use CountryConfig)
