# E-Rechnung Generator — Technischer Implementierungsplan

## Übersicht

Kostenloser, quelloffener E-Rechnung-Generator. Generiert XRechnung-XML (EN 16931) und ZUGFeRD-PDF (PDF/A-3) aus einem Webformular. Open Source (MIT), Web + Android + iOS.

**Repo:** `github.com/V3SP45/e` (public)
**Hosting:** Vercel Free Tier
**Lizenz:** MIT

---

## Architektur

```
┌──────────────────────────────────────────┐
│            React SPA (Vite)              │
│                                          │
│  ┌──────────┐   ┌────────────────────┐   │
│  │ Formular │──▶│ Zod-Validierung    │   │
│  │ (RHF)    │   │ (UStG §14 Regeln) │   │
│  └──────────┘   └─────────┬──────────┘   │
│                           │              │
│              ┌────────────┼────────────┐ │
│              ▼                         ▼ │
│  ┌───────────────────┐  ┌─────────────┐ │
│  │ XRechnung XML     │  │ ZUGFeRD PDF │ │
│  │ @e-invoice-eu/core│  │ POST /api/  │ │
│  │ (100% Browser)    │  │ zugferd     │ │
│  └────────┬──────────┘  └──────┬──────┘ │
│           │                    │         │
│      Download XML        Download PDF   │
└───────────┼────────────────────┼─────────┘
            │                    │
            │         ┌──────────▼─────────┐
            │         │ Vercel Serverless   │
            │         │ node-zugferd        │
            │         │ JSON → PDF/A-3      │
            │         │ (kein Datenspeicher)│
            │         └────────────────────┘
            │
       Kein Server nötig
       Daten bleiben im Browser
```

### Design-Entscheidungen

| Entscheidung | Begründung |
|---|---|
| **Vite + React statt Next.js** | Pure SPA, kein SSR nötig, Capacitor-kompatibel für App Stores |
| **Client-side XML** | Kein Server = keine Datenschutz-Bedenken, offline-fähig |
| **Serverless für PDF** | PDF/A-3-Generierung im Browser nicht möglich (kein JS-Library kann das), Vercel Free Tier reicht |
| **Kein Monorepo** | Overkill für ein Single-Page-Tool |
| **shadcn/ui** | Copy-paste Komponenten, kein Dependency-Lock-in, Tailwind-basiert |

---

## Tech-Stack

### Dependencies

| Paket | Version | Zweck |
|---|---|---|
| **react** | ^19 | UI-Framework |
| **react-dom** | ^19 | DOM-Rendering |
| **vite** | ^8 | Build-Tool, Dev-Server |
| **typescript** | ^5.9 | Type Safety |
| **tailwindcss** | ^4 | Styling |
| **react-hook-form** | ^7 | Formular-Management |
| **@hookform/resolvers** | ^5 | Zod-Integration für RHF |
| **zod** | ^4 | Schema-Validierung |
| **@e-invoice-eu/core** | ^3 | XRechnung/ZUGFeRD XML-Generierung (EN 16931) |
| **lucide-react** | ^1 | Icons |

### Dev Dependencies

| Paket | Zweck |
|---|---|
| **@vitejs/plugin-react** | React Fast Refresh |
| **eslint** + Plugins | Linting |
| **vitest** | Unit Tests |
| **@testing-library/react** | Komponenten-Tests |

### Serverless (api/)

| Paket | Zweck |
|---|---|
| **node-zugferd** | ZUGFeRD PDF/A-3 Generierung |
| **@vercel/node** | Vercel Serverless Function Runtime |

---

## Projektstruktur

```
e/
├── src/
│   ├── components/
│   │   ├── ui/                    # shadcn/ui Basis-Komponenten
│   │   │   ├── button.tsx
│   │   │   ├── input.tsx
│   │   │   ├── label.tsx
│   │   │   ├── select.tsx
│   │   │   ├── card.tsx
│   │   │   └── separator.tsx
│   │   ├── InvoiceForm.tsx        # Hauptformular (alle Felder)
│   │   ├── SellerSection.tsx      # Verkäufer-Daten
│   │   ├── BuyerSection.tsx       # Käufer-Daten
│   │   ├── LineItemsSection.tsx   # Rechnungspositionen (dynamisch)
│   │   ├── PaymentSection.tsx     # Zahlungsbedingungen
│   │   ├── TotalsDisplay.tsx      # Netto/USt/Brutto Berechnung
│   │   ├── Preview.tsx            # Rechnungs-Vorschau (formatiert)
│   │   ├── DownloadButtons.tsx    # XML + PDF Download
│   │   ├── Header.tsx             # App-Header + Branding
│   │   └── Footer.tsx             # Impressum, GitHub-Link
│   ├── lib/
│   │   ├── schema.ts             # Zod-Schemas (Invoice, Seller, Buyer, LineItem)
│   │   ├── xrechnung.ts          # XRechnung XML-Generierung
│   │   ├── calculations.ts       # Netto/USt/Brutto Berechnung
│   │   ├── format.ts             # Währung, Datum, Nummern formatieren
│   │   ├── download.ts           # File-Download Utilities
│   │   └── constants.ts          # USt-Sätze, Einheiten, Ländercodes
│   ├── hooks/
│   │   ├── useInvoiceForm.ts     # Form-State + Berechnung
│   │   └── useXRechnung.ts       # XML-Generierung Hook
│   ├── App.tsx                    # Hauptlayout
│   ├── main.tsx                   # Entry Point
│   └── index.css                  # Tailwind Base + Custom Styles
├── api/
│   └── zugferd.ts                 # Vercel Serverless: JSON → ZUGFeRD PDF/A-3
├── public/
│   ├── manifest.json              # PWA-Manifest
│   ├── favicon.svg
│   └── og-image.png               # Open Graph Image für SEO
├── specs/
│   ├── formular.md                # Spec: Formularfelder + Validierung
│   ├── xrechnung.md               # Spec: XML-Generierung
│   └── zugferd-pdf.md             # Spec: PDF/A-3 Generierung
├── __tests__/
│   ├── schema.test.ts             # Validierungs-Tests
│   ├── calculations.test.ts       # Berechnungs-Tests
│   └── xrechnung.test.ts          # XML-Generierungs-Tests
├── PROJEKTPLAN.md                 # ← Dieses Dokument
├── AGENTS.md                      # OpenClaw System Prompt
├── CLAUDE.md                      # Claude Code Kontext
├── README.md                      # Open Source Doku
├── LICENSE                        # MIT
├── index.html
├── vite.config.ts
├── tailwind.config.ts
├── tsconfig.json
├── vercel.json                    # Serverless Function Config
└── package.json
```

---

## Formularfelder (UStG §14 Pflichtangaben)

### Rechnungs-Stammdaten

| Feld | ID | Pflicht | Typ | Validierung |
|---|---|---|---|---|
| Rechnungsnummer | BT-1 | Ja | Text | Nicht leer, max 50 Zeichen |
| Rechnungsdatum | BT-2 | Ja | Datum | Gültiges ISO-Datum |
| Rechnungsart | BT-3 | Ja | Select | 380 (Rechnung), 381 (Gutschrift), 384 (Korrektur) |
| Währung | BT-5 | Ja | Select | EUR (Default), CHF, USD |
| Leistungsdatum/-zeitraum | BT-72/73 | Ja | Datum | Start ≤ Ende |
| Fälligkeitsdatum | BT-9 | Nein | Datum | ≥ Rechnungsdatum |

### Verkäufer (Seller)

| Feld | ID | Pflicht | Validierung |
|---|---|---|---|
| Name | BT-27 | Ja | Min 2 Zeichen |
| Straße | BT-35 | Ja | Nicht leer |
| PLZ | BT-38 | Ja | DE: 5 Ziffern, AT: 4 Ziffern, CH: 4 Ziffern |
| Ort | BT-37 | Ja | Min 2 Zeichen |
| Land | BT-40 | Ja | ISO 3166-1 alpha-2 (DE, AT, CH) |
| USt-ID | BT-31 | Ja | DE: `DE[0-9]{9}`, AT: `ATU[0-9]{8}`, CH: `CHE-[0-9]{3}\.[0-9]{3}\.[0-9]{3}` |
| Steuernummer | BT-32 | Nein | Format variiert nach Land |
| E-Mail | BT-34 | Nein | E-Mail-Format |
| Telefon | — | Nein | Optional |
| Bankverbindung (IBAN) | BT-84 | Nein | IBAN-Format |
| BIC | BT-86 | Nein | 8 oder 11 Zeichen |

### Käufer (Buyer)

| Feld | ID | Pflicht | Validierung |
|---|---|---|---|
| Name | BT-44 | Ja | Min 2 Zeichen |
| Straße | BT-50 | Ja | Nicht leer |
| PLZ | BT-53 | Ja | Länder-spezifisch |
| Ort | BT-52 | Ja | Min 2 Zeichen |
| Land | BT-55 | Ja | ISO 3166-1 alpha-2 |
| USt-ID | BT-48 | Nein | Länder-Format |
| Leitweg-ID | BT-10 | Nein | `[0-9]{2,12}-[0-9A-Z]{1,30}-[0-9]{2}` |

### Rechnungspositionen (Line Items)

| Feld | ID | Pflicht | Validierung |
|---|---|---|---|
| Beschreibung | BT-153 | Ja | Min 2 Zeichen |
| Menge | BT-129 | Ja | > 0, max 2 Dezimalstellen |
| Einheit | BT-130 | Ja | UN/ECE Rec 20 (H87=Stück, HUR=Stunde, DAY=Tag, MON=Monat) |
| Einzelpreis (netto) | BT-146 | Ja | ≥ 0, max 2 Dezimalstellen |
| USt-Kategorie | BT-151 | Ja | S=Standard, Z=Null, E=Befreit, AE=Reverse Charge |
| USt-Satz | BT-152 | Ja | 0%, 7%, 19% (DE); 0%, 10%, 13%, 20% (AT); 0%, 2.6%, 3.8%, 8.1% (CH) |

**Dynamisch:** Mindestens 1 Position, beliebig viele hinzufügbar via „+ Position"-Button.

### Automatische Berechnungen

```
Position Netto    = Menge × Einzelpreis
Position USt      = Position Netto × USt-Satz
Position Brutto   = Position Netto + Position USt

Gesamt Netto      = Σ aller Position Netto
Gesamt USt        = Σ aller Position USt (gruppiert nach USt-Satz)
Gesamt Brutto     = Gesamt Netto + Gesamt USt
```

---

## XRechnung XML-Generierung

### Library: `@e-invoice-eu/core`

- 100% Client-Side (Browser)
- Unterstützt: EN 16931, XRechnung, ZUGFeRD/Factur-X, UBL, CII
- Input: JSON-Objekt mit Invoice-Daten
- Output: Valides XML (CII oder UBL Syntax)
- Lizenz: WTFPL (Do What The Fuck You Want To Public License)

### XRechnung-Profil

XRechnung ist eine **CIUS** (Core Invoice Usage Specification) auf Basis von EN 16931. Gegenüber dem Basis-Standard hat XRechnung zusätzliche Regeln:

- Leitweg-ID (BT-10) ist Pflicht bei Rechnungen an öffentliche Auftraggeber
- Buyer Reference (BT-10) ist immer Pflicht
- Payment Means (BG-16) ist Pflicht
- Spezifische Schematron-Regeln (BR-DE-*)

### Generierungsflow

```
Formular-Daten (Zod-validiert)
    ↓
Mapping auf @e-invoice-eu/core JSON-Format
    ↓
@e-invoice-eu/core.generate({ format: 'xrechnung-cii' })
    ↓
Valides XRechnung CII XML
    ↓
Blob → Download als .xml
```

---

## ZUGFeRD PDF/A-3 Generierung

### Serverless Function: `api/zugferd.ts`

**Endpoint:** `POST /api/zugferd`
**Input:** Invoice-JSON (identisch zum Formular-State)
**Output:** PDF/A-3 Binary (application/pdf)

### Flow

```
Client: POST /api/zugferd { invoice: {...} }
    ↓
Server: Validierung (Zod)
    ↓
Server: XRechnung XML generieren (@e-invoice-eu/core)
    ↓
Server: PDF generieren mit Rechnungsdaten
    ↓
Server: XML in PDF/A-3 einbetten (node-zugferd)
    ↓
Client: PDF als Blob empfangen → Download
```

### vercel.json

```json
{
  "functions": {
    "api/zugferd.ts": {
      "memory": 256,
      "maxDuration": 10
    }
  }
}
```

### Datenschutz

- Keine Daten werden gespeichert
- Kein Logging von Invoice-Inhalten
- Serverless Function ist stateless
- Daten existieren nur während der Request-Verarbeitung

---

## Phasenplan

### Phase 1: Grundgerüst (Tag 1, ~30 Min)

- [x] Vite + React + TypeScript initialisieren
- [ ] Tailwind konfigurieren
- [ ] shadcn/ui Basis-Komponenten
- [ ] CLAUDE.md schreiben
- [ ] Git-Repo + GitHub Remote (public, MIT)
- [ ] Specs schreiben (3 Dateien in specs/)

### Phase 2: Formular + XRechnung (Tag 1–2, Agenten-Arbeit)

**Parallel ausführbar:**
- **Agent A:** Zod-Schemas + Berechnungslogik (`lib/schema.ts`, `lib/calculations.ts`)
- **Agent B:** Formular-Komponenten (`components/InvoiceForm.tsx` + Sections)
- **Agent C:** XRechnung-Integration (`lib/xrechnung.ts`, `hooks/useXRechnung.ts`)

**Sequenziell danach:**
- Integration: Form → Schema → XML → Download
- Tests: Schema-Validierung, Berechnung, XML-Output

### Phase 3: ZUGFeRD PDF (Tag 2–3)

- Serverless Function (`api/zugferd.ts`)
- PDF-Generierung mit node-zugferd
- Download-Button in UI
- Test gegen KoSIT-Validator

### Phase 4: Polish + Deploy (Tag 3)

- Responsive Design (Mobile-First)
- PWA-Manifest + Service Worker
- SEO: Meta-Tags, OG-Image, robots.txt
- README.md (Badges, Screenshots, Quick Start)
- Vercel Production Deploy
- GitHub Repo public stellen

### Phase 5: App Stores (Woche 2+)

- Capacitor initialisieren (`npx cap init`)
- Android-Projekt generieren (`npx cap add android`)
- iOS-Projekt generieren (`npx cap add ios`)
- Google Play: $25 einmalig, Android Studio Build
- iOS App Store: $99/Jahr, Xcode Build (braucht Mac oder CI-Service)

---

## Verifizierung

| Was | Wie | Tool |
|---|---|---|
| XRechnung XML-Validität | Upload auf Validator | [erechnungs-validator.de](https://erechnungs-validator.de/) |
| ZUGFeRD PDF-Validität | Upload auf Validator | [kositvalidator.service-bw.de](https://kositvalidator.service-bw.de/) |
| UStG §14 Pflichtfelder | Formular ohne Pflichtfeld absenden → Fehler | Manuell + Unit Tests |
| Berechnung | Netto × USt = korrekt | Unit Tests (vitest) |
| Responsive | Mobile + Tablet + Desktop | Chrome DevTools |
| Performance | Lighthouse Score > 90 | Chrome Lighthouse |
| Build | `pnpm build` ohne Fehler | CI |
| Lint | `pnpm lint` ohne Fehler | CI |

---

## SEO-Strategie (für späteres Ranking)

| Element | Inhalt |
|---|---|
| **Title** | „E-Rechnung erstellen — Kostenloser XRechnung Generator" |
| **Description** | „XRechnung und ZUGFeRD kostenlos erstellen. EN 16931 konform, Open Source, kein Account nötig. Sofort E-Rechnungen generieren." |
| **H1** | „Kostenlos E-Rechnungen erstellen" |
| **Keywords** | e-rechnung erstellen, xrechnung generator, zugferd erstellen, e-rechnung kostenlos |
| **OG-Image** | Screenshot der App mit Formular |
| **Domain** | TBD (z.B. e-rechnung.tools oder sub von Hauptprodukt) |
