# E-Rechnung Generator — Implementierungsplan

## Übersicht

Kostenloser, quelloffener E-Rechnungs-Generator für Europa. Generiert EN 16931 konforme E-Rechnungen (XRechnung CII, Factur-X, Peppol BIS UBL) und ZUGFeRD/Factur-X PDF/A-3 aus einem Webformular. Kein Account, keine serverseitige Datenspeicherung, Open Source (MIT).

**Repo:** `github.com/V3SP45/e`
**Hosting:** Vercel Free Tier
**Lizenz:** MIT
**Sprachen:** i18n — Englisch (Basis), Deutsch, Französisch, Niederländisch, Italienisch, Polnisch, + Community
**Positionierung:** Radikal Open Source. Gratis. Made in Europe. Proof of Concept für Größeres.

---

## Länder-Support

### Launch (Tier 1) — Volle Unterstützung

Alle EN 16931 konform, kein nationales Sonderformat nötig.

| Land | USt-Sätze | E-Rechnungs-Format | i18n Locale |
|---|---|---|---|
| 🇩🇪 Deutschland | 19%, 7%, 0% | XRechnung CII | de |
| 🇦🇹 Österreich | 20%, 13%, 10%, 0% | EN 16931 CII/UBL | de |
| 🇨🇭 Schweiz | 8.1%, 2.6%, 3.8%, 0% | EN 16931 CII | de, fr, it |
| 🇫🇷 Frankreich | 20%, 10%, 5.5%, 2.1%, 0% | Factur-X | fr |
| 🇳🇱 Niederlande | 21%, 9%, 0% | Peppol BIS UBL | nl |
| 🇧🇪 Belgien | 21%, 12%, 6%, 0% | Peppol BIS UBL | nl, fr |
| 🇱🇺 Luxemburg | 17%, 14%, 8%, 3%, 0% | Peppol BIS UBL | fr, de |
| 🇩🇰 Dänemark | 25%, 0% | Peppol BIS UBL | en* |
| 🇸🇪 Schweden | 25%, 12%, 6%, 0% | Peppol BIS UBL | en* |
| 🇫🇮 Finnland | 25.5%, 14%, 10%, 0% | Peppol BIS UBL | en* |
| 🇳🇴 Norwegen | 25%, 15%, 12%, 0% | Peppol BIS UBL | en* |
| 🇪🇪 Estland | 22%, 9%, 0% | Peppol BIS UBL | en* |
| 🇱🇻 Lettland | 21%, 12%, 5%, 0% | Peppol BIS UBL | en* |
| 🇱🇹 Litauen | 21%, 9%, 5%, 0% | Peppol BIS UBL | en* |
| 🇮🇪 Irland | 23%, 13.5%, 9%, 4.8%, 0% | Peppol BIS UBL | en |
| 🇸🇮 Slowenien | 22%, 9.5%, 5%, 0% | Peppol BIS UBL | en* |
| 🇭🇷 Kroatien | 25%, 13%, 5%, 0% | Peppol BIS UBL | en* |
| 🇸🇰 Slowakei | 23%, 10%, 5%, 0% | Peppol BIS UBL | en* |
| 🇨🇿 Tschechien | 21%, 12%, 0% | Peppol BIS UBL | en* |

*en = Englisch als UI-Fallback. Native Locales können von der Community beigesteuert werden.

### Später (Tier 2) — Nationales Sonderformat nötig

| Land | Format | Aufwand |
|---|---|---|
| 🇮🇹 Italien | FatturaPA (SDI) | Hoch — eigenes XML-Schema, eigener Übermittlungsweg |
| 🇵🇱 Polen | KSeF | Hoch — eigenes Schema, Regierungsplattform |
| 🇭🇺 Ungarn | NAV Real-Time | Hoch — Live-Reporting an Steuerbehörde |
| 🇬🇷 Griechenland | myDATA | Hoch — eigenes System |
| 🇪🇸 Spanien | Factura-e (B2G) | Mittel — B2G eigenes Format, B2B geht mit EN 16931 |
| 🇵🇹 Portugal | SAF-T Meldepflicht | Mittel — zusätzliche Reporting-Pflicht |
| 🇷🇴 Rumänien | RO_CIUS | Mittel — nationale CIUS |

---

## Architektur

```
┌──────────────────────────────────────────────┐
│              React SPA (Vite)                │
│                                              │
│  ┌──────────┐   ┌─────────────────────────┐  │
│  │ Formular │──▶│ Zod-Validierung         │  │
│  │ (RHF)    │   │ (dynamisch per Land)    │  │
│  └──────────┘   └──────────┬──────────────┘  │
│                            │                 │
│               ┌────────────┼──────────┐      │
│               ▼                       ▼      │
│  ┌────────────────────┐  ┌────────────────┐  │
│  │ E-Rechnung XML     │  │ ZUGFeRD PDF    │  │
│  │ @e-invoice-eu/core │  │ POST /api/pdf  │  │
│  │ XRechnung / UBL /  │  │                │  │
│  │ Factur-X / Peppol  │  │                │  │
│  │ (100% Browser)     │  │                │  │
│  └────────┬───────────┘  └───────┬────────┘  │
│           │                      │           │
│      Download XML          Download PDF      │
└───────────┼──────────────────────┼───────────┘
            │                      │
            │           ┌──────────▼──────────┐
            │           │ Vercel Serverless    │
            │           │ PDF/A-3 Generierung  │
            │           │ (stateless)          │
            │           └─────────────────────┘
```

### Design-Entscheidungen

| Entscheidung | Begründung |
|---|---|
| **Vite + React statt Next.js** | Pure SPA, kein SSR nötig, Capacitor-kompatibel |
| **Client-side XML** | Kein Server = kein Datenschutz-Problem, offline-fähig |
| **Serverless nur für PDF** | PDF/A-3 im Browser nicht möglich, Vercel Free reicht |
| **shadcn/ui** | Copy-paste, kein Lock-in, Tailwind-basiert |
| **react-i18next** | Bewährter Standard, JSON-basierte Locale-Dateien, Community-beitragsfähig |
| **Länderdaten als JSON** | USt-Sätze, Validierung, Formate pro Land — erweiterbar ohne Code-Änderung |

---

## Tech-Stack

### Core Dependencies

| Paket | Version | Zweck |
|---|---|---|
| react | ^19 | UI-Framework |
| react-dom | ^19 | DOM-Rendering |
| vite | ^6 | Build-Tool |
| typescript | ^5.7 | Type Safety |
| tailwindcss | ^4 | Styling |
| react-hook-form | ^7 | Formular-Management |
| @hookform/resolvers | ^5 | Zod-Integration |
| zod | ^3 | Schema-Validierung |
| react-i18next | ^15 | Internationalisierung |
| i18next | ^24 | i18n Core |
| lucide-react | ^0.468 | Icons |

### E-Rechnung Libraries (Phase 0: evaluieren)

| Paket | Zweck | Risiko |
|---|---|---|
| **@e-invoice-eu/core** | XML-Generierung (CII, UBL, XRechnung, Peppol) | Niedrig |
| **PDF/A-3 Library** | ZUGFeRD/Factur-X PDF | **Hoch — muss evaluiert werden** |

> **Phase 0 Aktion:** PDF/A-3 Library evaluieren:
> 1. `node-zugferd` — wenn stabil
> 2. Eigene PDF-Lösung mit `pdf-lib` + XML-Embedding
> 3. Puppeteer als Fallback (teuer auf Serverless)

### Dev Dependencies

| Paket | Zweck |
|---|---|
| @vitejs/plugin-react | React Fast Refresh |
| eslint + Plugins | Linting |
| vitest | Unit Tests |
| @testing-library/react | Komponenten-Tests |

---

## Projektstruktur

```
e/
├── src/
│   ├── components/
│   │   ├── ui/                      # shadcn/ui Basis-Komponenten
│   │   ├── InvoiceForm.tsx          # Hauptformular (orchestriert Sections)
│   │   ├── InvoiceMetaSection.tsx   # Rechnungsnummer, Datum, Währung, Land
│   │   ├── SellerSection.tsx        # Verkäufer-Daten
│   │   ├── BuyerSection.tsx         # Käufer-Daten
│   │   ├── LineItemsSection.tsx     # Positionen (dynamisch)
│   │   ├── PaymentSection.tsx       # Zahlungsbedingungen
│   │   ├── TotalsDisplay.tsx        # Live-Berechnung
│   │   ├── Preview.tsx              # Rechnungs-Vorschau (HTML)
│   │   ├── DownloadButtons.tsx      # XML + PDF Download
│   │   ├── LanguageSwitcher.tsx     # Sprachauswahl
│   │   ├── Header.tsx
│   │   └── Footer.tsx               # + Impressum-Link
│   ├── lib/
│   │   ├── schema.ts               # Zod-Schemas (dynamisch per Land)
│   │   ├── xrechnung.ts            # Mapping → @e-invoice-eu/core → XML
│   │   ├── calculations.ts         # Netto/USt/Brutto mit korrekter Rundung
│   │   ├── format.ts               # Locale-aware Formatierung
│   │   ├── download.ts             # Blob-Download Helper
│   │   ├── countries.ts            # Länderdaten-Loader
│   │   └── storage.ts              # localStorage opt-in für Seller-Daten
│   ├── data/
│   │   └── countries/
│   │       ├── _types.ts            # TypeScript Interface für Country-Config
│   │       ├── de.ts                # Deutschland: USt, Validierung, Format
│   │       ├── at.ts                # Österreich
│   │       ├── ch.ts                # Schweiz
│   │       ├── fr.ts                # Frankreich
│   │       ├── nl.ts                # Niederlande
│   │       ├── be.ts                # Belgien
│   │       ├── lu.ts                # Luxemburg
│   │       ├── dk.ts                # Dänemark
│   │       ├── se.ts                # Schweden
│   │       ├── fi.ts                # Finnland
│   │       ├── no.ts                # Norwegen
│   │       ├── ee.ts                # Estland
│   │       ├── lv.ts                # Lettland
│   │       ├── lt.ts                # Litauen
│   │       ├── ie.ts                # Irland
│   │       ├── si.ts                # Slowenien
│   │       ├── hr.ts                # Kroatien
│   │       ├── sk.ts                # Slowakei
│   │       ├── cz.ts                # Tschechien
│   │       └── index.ts            # Registry: alle Länder exportieren
│   ├── i18n/
│   │   ├── config.ts               # i18next Setup
│   │   └── locales/
│   │       ├── en.json             # Englisch (Basis/Fallback)
│   │       ├── de.json             # Deutsch
│   │       ├── fr.json             # Französisch
│   │       └── nl.json             # Niederländisch
│   ├── hooks/
│   │   ├── useInvoiceForm.ts       # Form-State + Berechnung
│   │   ├── useXRechnung.ts         # XML-Generierung
│   │   ├── useCountry.ts           # Aktives Land + Config
│   │   └── useSellerStorage.ts     # localStorage opt-in
│   ├── App.tsx
│   ├── main.tsx
│   └── index.css
├── api/
│   └── pdf.ts                      # Vercel Serverless: JSON → PDF/A-3
├── public/
│   ├── favicon.svg
│   └── og-image.png
├── __tests__/
│   ├── schema.test.ts
│   ├── calculations.test.ts
│   ├── xrechnung.test.ts
│   └── countries.test.ts           # Länderdaten-Validierung
├── PROJEKTPLAN.md
├── AGENTS.md
├── CLAUDE.md
├── README.md
├── LICENSE
├── index.html
├── vite.config.ts
├── tsconfig.json
├── vercel.json
└── package.json
```

### Country-Config Interface

Jedes Land ist eine TypeScript-Datei mit einheitlicher Struktur:

```typescript
interface CountryConfig {
  code: string;                    // ISO 3166-1 alpha-2
  name: string;                    // Englischer Name
  currency: string;                // ISO 4217 (EUR, CHF, NOK, etc.)
  locale: string;                  // Primäres Locale
  invoiceFormat: 'xrechnung-cii' | 'factur-x' | 'peppol-bis-ubl';
  vatRates: VatRate[];             // Steuersätze
  vatIdFormat?: RegExp;            // USt-ID Regex
  postalCodeFormat?: RegExp;       // PLZ Regex
  requiredFields?: string[];       // Zusätzliche Pflichtfelder (z.B. Leitweg-ID für DE B2G)
}

interface VatRate {
  rate: number;                    // z.B. 19
  label: string;                   // i18n-Key oder String
  category: 'S' | 'Z' | 'E' | 'AE'; // EN 16931 Kategorie
  isDefault?: boolean;
}
```

---

## Datenmodell

### Rechnungs-Stammdaten

| Feld | BT-ID | Pflicht | Validierung |
|---|---|---|---|
| Rechnungsnummer | BT-1 | Ja | Nicht leer, max 50 Zeichen |
| Rechnungsdatum | BT-2 | Ja | Gültiges ISO-Datum |
| Rechnungsart | BT-3 | Ja | 380, 381, 384 |
| Währung | BT-5 | Ja | Aus CountryConfig, überschreibbar |
| Leistungszeitraum Start | BT-73 | Ja | Gültiges Datum |
| Leistungszeitraum Ende | BT-74 | Ja | ≥ Start |
| Fälligkeitsdatum | BT-9 | Nein | ≥ Rechnungsdatum |
| Buyer Reference | BT-10 | Ja | XRechnung-Pflicht, für andere Formate optional |
| Seller-Land | — | Ja | Bestimmt CountryConfig + Format |
| Buyer-Land | — | Ja | ISO 3166-1 alpha-2 |

### Verkäufer / Käufer / Positionen

*(Identisch zum bisherigen Plan — länderspezifische Validierung wird dynamisch aus CountryConfig geladen.)*

### Berechnungslogik

```
Position Netto  = Menge × Einzelpreis        → runden auf 2 Dezimalstellen
Position USt    = Position Netto × (Satz/100) → runden auf 2 Dezimalstellen
Position Brutto = Position Netto + Position USt

Gesamt Netto    = Σ Position Netto
Gesamt USt      = Σ Position USt (gruppiert nach USt-Satz)
Gesamt Brutto   = Gesamt Netto + Gesamt USt
```

---

## Phasenplan

### Phase 0: Setup & Evaluation

**Ziel:** Fundament steht, Risiken eliminiert.

- [ ] Library-Check: `@e-invoice-eu/core` — installieren, Beispiel-XML generieren (CII + UBL), gegen Validator prüfen
- [ ] Library-Check: PDF/A-3-Lösung evaluieren und Entscheidung dokumentieren
- [ ] Vite + React + TypeScript + Tailwind + shadcn/ui initialisieren
- [ ] i18next Setup + Basis-Locales (en, de)
- [ ] Projektstruktur anlegen
- [ ] Path-Alias `@/` konfigurieren
- [ ] ESLint + Vitest konfigurieren
- [ ] vercel.json, LICENSE (MIT), README.md Grundgerüst
- [ ] `pnpm dev`, `pnpm build`, `pnpm lint` laufen

**Done when:** Leere App mit i18n + Routing läuft, Library-Evaluation abgeschlossen.

### Phase 1: Datenmodell & Kernlogik

**Ziel:** Alle Nicht-UI-Logik steht und ist getestet.

- [ ] `data/countries/` — alle 19 Länderkonfigurationen + TypeScript Interface
- [ ] `lib/schema.ts` — Zod-Schemas, dynamisch validierend per CountryConfig
- [ ] `lib/calculations.ts` — Netto/USt/Brutto mit korrekter Rundung
- [ ] `lib/format.ts` — Locale-aware Währung/Datum
- [ ] `lib/xrechnung.ts` — Mapping Formular → @e-invoice-eu/core (CII, UBL, Factur-X je nach Land)
- [ ] `lib/download.ts` — Blob-Download
- [ ] `lib/storage.ts` — localStorage opt-in für Seller-Daten
- [ ] `__tests__/` — Schema, Berechnung, XML-Output, Länderdaten-Konsistenz

**Done when:** Alle Tests grün. Aus einem JSON-Objekt wird valides XML generiert für DE (XRechnung CII), FR (Factur-X), NL (Peppol BIS UBL).

### Phase 2: UI & Formular

**Ziel:** Funktionierendes Formular, Land wählen, XML runterladen.

- [ ] shadcn/ui Komponenten
- [ ] Header + Footer + LanguageSwitcher
- [ ] InvoiceMetaSection (mit Länderwahl → steuert Format + Validierung)
- [ ] SellerSection + BuyerSection (dynamische Validierung)
- [ ] LineItemsSection (dynamisch, USt-Sätze aus CountryConfig)
- [ ] PaymentSection
- [ ] TotalsDisplay (live)
- [ ] InvoiceForm (orchestriert alles)
- [ ] useInvoiceForm + useXRechnung + useCountry + useSellerStorage
- [ ] DownloadButtons (XML)
- [ ] Preview (funktionale HTML-Ansicht)
- [ ] Responsive (Mobile-First)
- [ ] i18n: de.json, fr.json, nl.json vollständig

**Done when:** Formular → Land wählen → Ausfüllen → XML Download → Valide bei erechnungs-validator.de (DE) und Peppol Validator (NL).

### Phase 3: ZUGFeRD / Factur-X PDF

**Ziel:** PDF/A-3 mit eingebettetem XML.

- [ ] `api/pdf.ts` — Serverless Function
- [ ] PDF-Layout (Rechnungsdaten als lesbares PDF)
- [ ] XML-Einbettung als PDF/A-3 Attachment
- [ ] Download-Button in UI
- [ ] Error Handling
- [ ] Lokal testen mit `vercel dev`

**Done when:** PDF Download funktioniert, eingebettetes XML besteht KoSIT-Validator.

### Phase 4: Polish & Deploy

**Ziel:** Produktionsreif.

- [ ] SEO: Meta-Tags, OG-Image
- [ ] PWA: manifest.json (offline XML-Generierung)
- [ ] Accessibility: aria-Labels, Keyboard-Nav, Focus-Management
- [ ] Error/Loading States
- [ ] README.md: Screenshots, Länderliste, Contributing Guide (wie man ein Land hinzufügt)
- [ ] Impressum-Platzhalter
- [ ] Vercel Production Deploy
- [ ] Smoke Test auf Production

**Done when:** Lighthouse >90, README fertig, Production live.

### Später: Tier 2 Länder

- [ ] Italien (FatturaPA) — eigenes XML-Schema, SDI-Anbindung
- [ ] Polen (KSeF) — eigenes Schema
- [ ] Weitere nationale Formate nach Community-Bedarf

---

## Entscheidungen

| Frage | Entscheidung |
|---|---|
| Domain | Vercel-Subdomain, eigene Domain optional später |
| Impressum | Platzhalter im Footer, Maintainer füllt aus |
| ZUGFeRD-Scope | Vollständig — XML + PDF/A-3 |
| UI-Sprache | i18n: Englisch (Basis), Deutsch, Französisch, Niederländisch. Weitere community-driven. |
| localStorage | Opt-in für Seller-Daten. "Deine Daten bleiben in deinem Browser." |
| Vorschau | Funktionale HTML-Ansicht |
| Positionierung | Radikal Open Source. Gratis. Made in Europe, for Europe. |
| Tier 2 Länder (IT, PL) | EN 16931 Cross-Border sofort, nationale Formate später |

---

## Verifizierung

| Was | Wie |
|---|---|
| XRechnung XML | [erechnungs-validator.de](https://erechnungs-validator.de/) |
| Peppol BIS UBL | [Peppol Validator](https://peppol.helger.com/public/locale-en_US/menuitem-validation-bis3) |
| Factur-X | [Factur-X Validator](https://services.fnfe-mpe.org/DEMAT/index.php) |
| ZUGFeRD PDF | [kositvalidator.service-bw.de](https://kositvalidator.service-bw.de/) |
| Länderdaten | Unit Tests: USt-Sätze, Regex-Formate, Pflichtfelder |
| Berechnung | Unit Tests (Rundung, Grenzfälle, Cross-Country) |
| i18n | Alle Keys in allen Locale-Dateien vorhanden |
| Build | `pnpm build` fehlerfrei |
| Lint | `pnpm lint` fehlerfrei |
| Tests | `pnpm test` alle grün |
| Responsive | Chrome DevTools |
| Performance | Lighthouse >90 |

---

## SEO

| Element | Inhalt |
|---|---|
| Title | "Free E-Invoice Generator — XRechnung, Factur-X, Peppol BIS" |
| Description | "Create EN 16931 compliant e-invoices for free. XRechnung, Factur-X, Peppol BIS. 19 European countries. Open Source, no account needed." |
| H1 | Per Locale: DE "Kostenlos E-Rechnungen erstellen", EN "Create E-Invoices for Free" |
| OG-Image | App Screenshot mit Länderflaggen |

---

## Contributing (für README.md)

### Ein Land hinzufügen

1. Erstelle `src/data/countries/{code}.ts` nach dem `CountryConfig` Interface
2. Registriere es in `src/data/countries/index.ts`
3. Optional: Füge ein Locale-File hinzu unter `src/i18n/locales/{lang}.json`
4. Schreibe Tests in `__tests__/countries.test.ts`
5. PR öffnen

Das ist absichtlich so simpel gehalten, damit die Community Länder beitragen kann, ohne die Kernlogik verstehen zu müssen.
