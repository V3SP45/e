# Tech Stack & Architektur

## Stack

- **Language:** TypeScript 5.7 (strict)
- **Framework:** React 19 + Vite 6
- **Styling:** Tailwind CSS 4 + shadcn/ui
- **Forms:** React Hook Form 7 + Zod 3
- **i18n:** react-i18next + i18next (en Basis, de, fr, nl)
- **E-Invoice:** @e-invoice-eu/core (100% client-side XML-Generierung)
- **PDF:** Serverless Function auf Vercel (PDF/A-3)
- **Icons:** lucide-react
- **Testing:** Vitest + Testing Library
- **Hosting:** Vercel Free Tier

## Architektur

```
Client (Browser)                    Server (Vercel Serverless)
├── React SPA + i18n                └── POST /api/pdf
├── Laenderwahl → CountryConfig         ├── JSON validieren
├── Formular → Zod-Validierung          ├── XML generieren
├── @e-invoice-eu/core                  ├── PDF/A-3 erzeugen
│   ├── XRechnung CII (DE)             └── PDF zurueckgeben
│   ├── Factur-X (FR)
│   └── Peppol BIS UBL (NL, BE, ...)
└── Download .xml
```

### Design-Entscheidungen

| Entscheidung | Begruendung |
|---|---|
| Vite + React statt Next.js | Pure SPA, kein SSR noetig |
| Client-side XML | Kein Server = kein Datenschutz-Problem, offline-faehig |
| Serverless nur fuer PDF | PDF/A-3 im Browser nicht moeglich, Vercel Free reicht |
| shadcn/ui | Copy-paste, kein Lock-in, Tailwind-basiert |
| Laenderdaten als TypeScript | USt-Saetze, Validierung, Formate pro Land -- erweiterbar ohne Kernlogik-Aenderung |

### Country System

Jedes Land = eine TypeScript-Datei in `src/data/countries/{code}.ts`:

```typescript
interface CountryConfig {
  code: string;           // ISO 3166-1 alpha-2
  currency: string;       // ISO 4217
  locale: string;
  invoiceFormat: 'xrechnung-cii' | 'factur-x' | 'peppol-bis-ubl';
  vatRates: VatRate[];
  vatIdFormat?: RegExp;
  postalCodeFormat?: RegExp;
}
```

Neues Land = neue Config-Datei + Registrierung. Keine Aenderung an der Kernlogik.

## Projektstruktur

```
e/
├── src/
│   ├── components/        # React-Komponenten (Form-Sections, UI)
│   │   └── ui/            # shadcn/ui Basis
│   ├── lib/               # Pure Logik (Schema, Berechnung, XML, Download)
│   ├── data/countries/    # Laenderkonfigurationen (19 Dateien)
│   ├── i18n/locales/      # Uebersetzungen (en, de, fr, nl)
│   └── hooks/             # Custom Hooks (Form, Country, Storage)
├── api/
│   └── pdf.ts             # Vercel Serverless: JSON → PDF/A-3
├── __tests__/             # Unit Tests
└── public/                # Static Assets
```

## Deployment

Vercel -- automatisches Deployment via GitHub Push.

```bash
# Lokal testen mit Vercel CLI
vercel dev

# Production
vercel --prod
```

## Entwicklung

```bash
pnpm install              # Abhaengigkeiten installieren
pnpm dev                  # Dev-Server (Vite)
pnpm build                # Production Build
pnpm preview              # Production-Preview
pnpm lint                 # ESLint
pnpm test                 # Vitest
pnpm test:watch           # Vitest Watch-Mode
```

### Code-Konventionen

- TypeScript strict, kein `any`
- Named Exports, keine Default Exports
- Funktionale Komponenten
- Zod-Schemas = Single Source of Truth
- Dateien: kebab-case, Komponenten: PascalCase
- Imports: `@/` Alias → `src/`
- Alle UI-Texte via i18n `t()` -- nie hardcoded
- Laenderspezifische Logik via CountryConfig -- nie in Kerncode
