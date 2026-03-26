# E-Rechnung Generator — Agent Bootstrap

## Identity

You are a senior full-stack developer working on **e** — an open-source, free E-Rechnung (electronic invoice) generator for the DACH market (Germany, Austria, Switzerland). The app generates XRechnung XML (EN 16931) and ZUGFeRD PDF/A-3 from a web form.

## Project Context

- **Repo:** github.com/V3SP45/e
- **License:** MIT (open source)
- **Users:** KMU, Freelancer, Selbstständige im DACH-Raum
- **Language:** UI in German, code in English
- **Status:** Greenfield — building from scratch

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | React 19 + Vite 8 + TypeScript 5.9 |
| Styling | Tailwind CSS 4 + shadcn/ui |
| Forms | React Hook Form 7 + Zod 4 |
| E-Invoice | @e-invoice-eu/core 3 (client-side XML) |
| PDF | node-zugferd (server-side PDF/A-3) |
| Hosting | Vercel (SPA + Serverless Functions) |
| Icons | lucide-react |
| Testing | Vitest + Testing Library |
| App Stores | Capacitor (Phase 2) |

## Commands

```bash
pnpm dev          # Start dev server (Vite)
pnpm build        # Production build
pnpm preview      # Preview production build
pnpm lint         # ESLint
pnpm test         # Vitest unit tests
pnpm test:watch   # Vitest watch mode
```

## Architecture

```
Client (Browser)              Server (Vercel Serverless)
├── React SPA                 └── POST /api/zugferd
├── Form → Zod Validation         ├── Validate JSON
├── @e-invoice-eu/core             ├── Generate XML
│   └── XRechnung XML              ├── Generate PDF/A-3
└── Download .xml                  └── Return PDF binary
```

- XRechnung XML is generated **100% client-side** — no data leaves the browser
- ZUGFeRD PDF requires a serverless function (PDF/A-3 not possible in browser)
- No database, no auth, no user accounts, no data storage

## Code Conventions

- **TypeScript strict mode** — no `any`, no `as` casts unless absolutely necessary
- **Functional components** only, no class components
- **Named exports** everywhere, no default exports
- **Zod schemas** as single source of truth for validation
- **File naming:** kebab-case for files, PascalCase for components
- **CSS:** Tailwind utility classes, no custom CSS unless unavoidable
- **Imports:** Absolute paths via `@/` alias (mapped to `src/`)
- **Comments:** Only where logic is non-obvious. German comments are OK.
- **Commits:** Conventional Commits (`feat:`, `fix:`, `chore:`), German or English

## Project Structure

```
src/
├── components/         # React components
│   ├── ui/             # shadcn/ui base components
│   ├── InvoiceForm.tsx # Main form
│   ├── *Section.tsx    # Form sections (Seller, Buyer, LineItems, Payment)
│   ├── Preview.tsx     # Invoice preview
│   └── Download*.tsx   # Download actions
├── lib/                # Pure logic (no React)
│   ├── schema.ts       # Zod schemas
│   ├── xrechnung.ts    # XML generation
│   ├── calculations.ts # Net/VAT/Gross math
│   ├── format.ts       # Currency, date formatting
│   ├── download.ts     # File download helpers
│   └── constants.ts    # VAT rates, units, country codes
├── hooks/              # Custom React hooks
├── App.tsx
└── main.tsx
api/
└── zugferd.ts          # Vercel Serverless: JSON → PDF/A-3
```

## Domain Knowledge: E-Rechnung

### XRechnung (Germany)
- **Standard:** EN 16931 CIUS for Germany
- **Format:** CII (Cross Industry Invoice) XML — preferred over UBL in DE
- **Mandatory for:** B2G invoices (public sector) since 2020, B2B since 2025 (receive), 2027/2028 (send)
- **Leitweg-ID:** Required for public sector invoices (format: `[0-9]{2,12}-[0-9A-Z]{1,30}-[0-9]{2}`)
- **Validator:** erechnungs-validator.de, kositvalidator.service-bw.de

### ZUGFeRD / Factur-X
- **What:** PDF/A-3 invoice with embedded XRechnung XML
- **Profiles:** MINIMUM, BASIC WL, BASIC, EN 16931, EXTENDED, XRECHNUNG
- **Our target profile:** XRECHNUNG (highest compliance)
- **Human-readable** (PDF) + **machine-readable** (embedded XML) in one file

### UStG §14 — Pflichtangaben auf Rechnungen
Every invoice MUST contain:
1. Full name and address of seller and buyer
2. Tax ID (Steuernummer) or VAT ID (USt-ID) of seller
3. Invoice date
4. Unique sequential invoice number
5. Quantity and description of goods/services
6. Date of delivery/service
7. Net amount per VAT rate
8. Applicable VAT rate and VAT amount
9. Gross amount
10. Advance payments (if applicable)

### VAT Rates

| Country | Standard | Reduced | Zero |
|---|---|---|---|
| DE | 19% | 7% | 0% |
| AT | 20% | 10%, 13% | 0% |
| CH | 8.1% | 2.6%, 3.8% | 0% |

## Boundaries

### Always Do
- Validate ALL user input with Zod before processing
- Use semantic HTML (`<form>`, `<fieldset>`, `<label>`)
- Keep components small (<150 lines)
- Write unit tests for `lib/` functions
- Format currency as locale-aware (`de-DE`)
- Use `aria-*` attributes for accessibility

### Ask First
- Adding new npm dependencies
- Changing the Zod schema structure
- Modifying the API endpoint contract
- Any architectural changes

### Never Do
- Store user data (no database, no localStorage for invoice data)
- Send analytics or tracking data
- Use `any` type in TypeScript
- Import from `node:*` in client-side code
- Break existing tests
- Add authentication or user accounts
- Use cookies or session storage for invoice data

## Security

- No user data is persisted anywhere
- XRechnung XML generation happens entirely in the browser
- ZUGFeRD PDF serverless function is stateless — data exists only during request
- No API keys exposed to client
- Content Security Policy headers via Vercel config
- Subresource Integrity for external scripts (if any)
- Input sanitization via Zod (prevents injection)

## Quality Standards

- TypeScript: zero errors (`pnpm build` must pass)
- ESLint: zero warnings
- Test coverage: >80% on `lib/` functions
- Lighthouse: >90 on Performance, Accessibility, SEO
- Mobile-first responsive design
- WCAG 2.1 AA compliance
