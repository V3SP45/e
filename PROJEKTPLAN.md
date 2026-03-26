# E-Rechnung Generator — Implementierungsplan

## Übersicht

Kostenloser, quelloffener E-Rechnung-Generator für den DACH-Raum. Generiert XRechnung-XML (EN 16931) und ZUGFeRD-PDF (PDF/A-3) aus einem Webformular. Kein Account, keine serverseitige Datenspeicherung, Open Source (MIT). Opt-in localStorage für Verkäufer-Stammdaten.

**Repo:** `github.com/V3SP45/e`
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
            │         │ PDF/A-3 Generierung │
            │         │ (kein Datenspeicher)│
            │         └────────────────────┘
```

### Design-Entscheidungen

| Entscheidung | Begründung |
|---|---|
| **Vite + React statt Next.js** | Pure SPA, kein SSR nötig, Capacitor-kompatibel |
| **Client-side XML** | Kein Server = kein Datenschutz-Problem, offline-fähig |
| **Serverless nur für PDF** | PDF/A-3 im Browser nicht möglich, Vercel Free Tier reicht |
| **shadcn/ui** | Copy-paste Komponenten, kein Lock-in, Tailwind-basiert |
| **Kein Monorepo** | Single-Page-Tool, Monorepo wäre Overhead |

---

## Tech-Stack

### Core Dependencies

| Paket | Version | Zweck |
|---|---|---|
| react | ^19 | UI-Framework |
| react-dom | ^19 | DOM-Rendering |
| vite | ^6 | Build-Tool, Dev-Server |
| typescript | ^5.7 | Type Safety |
| tailwindcss | ^4 | Styling |
| react-hook-form | ^7 | Formular-Management |
| @hookform/resolvers | ^5 | Zod-Integration für RHF |
| zod | ^3 | Schema-Validierung |
| lucide-react | ^0.468 | Icons |

### E-Rechnung Libraries (zu verifizieren)

| Paket | Zweck | Risiko |
|---|---|---|
| **@e-invoice-eu/core** | XRechnung/ZUGFeRD XML (EN 16931) | Niedrig — aktiv maintained, gute Docs |
| **node-zugferd** ODER **pdf-lib + manuelle Einbettung** | ZUGFeRD PDF/A-3 | **Hoch** — muss evaluiert werden |

> **AKTION vor Phase 2:** Library-Evaluation für PDF/A-3. Optionen prüfen:
> 1. `node-zugferd` — wenn stabil und maintained
> 2. `@e-invoice-eu/core` hat evtl. eigene PDF-Unterstützung
> 3. `pdf-lib` + manuelles XML-Embedding als Fallback
> 4. Puppeteer/Chromium-basierte Lösung als letzter Ausweg (teuer auf Serverless)

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
│   │   ├── ui/                    # shadcn/ui Basis-Komponenten
│   │   ├── InvoiceForm.tsx        # Hauptformular (orchestriert Sections)
│   │   ├── SellerSection.tsx      # Verkäufer-Daten
│   │   ├── BuyerSection.tsx       # Käufer-Daten
│   │   ├── InvoiceMetaSection.tsx # Rechnungsnummer, Datum, etc.
│   │   ├── LineItemsSection.tsx   # Rechnungspositionen (dynamisch)
│   │   ├── PaymentSection.tsx     # Zahlungsbedingungen
│   │   ├── TotalsDisplay.tsx      # Netto/USt/Brutto live-Berechnung
│   │   ├── Preview.tsx            # Rechnungs-Vorschau
│   │   ├── DownloadButtons.tsx    # XML + PDF Download
│   │   ├── Header.tsx             # App-Header
│   │   └── Footer.tsx             # GitHub-Link, Impressum
│   ├── lib/
│   │   ├── schema.ts             # Zod-Schemas (Invoice, Seller, Buyer, LineItem)
│   │   ├── xrechnung.ts          # Mapping Formular → @e-invoice-eu/core → XML
│   │   ├── calculations.ts       # Netto/USt/Brutto Logik
│   │   ├── format.ts             # Währung, Datum, Nummern
│   │   ├── download.ts           # Blob-Download Utilities
│   │   └── constants.ts          # USt-Sätze, UN/ECE Einheiten, Ländercodes
│   ├── hooks/
│   │   ├── useInvoiceForm.ts     # Form-State + live Berechnung
│   │   └── useXRechnung.ts       # XML-Generierung Hook
│   ├── App.tsx
│   ├── main.tsx
│   └── index.css                  # Tailwind Imports
├── api/
│   └── zugferd.ts                 # Vercel Serverless: JSON → ZUGFeRD PDF/A-3
├── public/
│   ├── favicon.svg
│   └── og-image.png
├── __tests__/
│   ├── schema.test.ts
│   ├── calculations.test.ts
│   └── xrechnung.test.ts
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

---

## Datenmodell (Formularfelder)

### Rechnungs-Stammdaten

| Feld | BT-ID | Pflicht | Validierung |
|---|---|---|---|
| Rechnungsnummer | BT-1 | Ja | Nicht leer, max 50 Zeichen |
| Rechnungsdatum | BT-2 | Ja | Gültiges ISO-Datum |
| Rechnungsart | BT-3 | Ja | 380 (Rechnung), 381 (Gutschrift), 384 (Korrektur) |
| Währung | BT-5 | Ja | EUR (Default), CHF, USD |
| Leistungszeitraum Start | BT-73 | Ja | Gültiges Datum |
| Leistungszeitraum Ende | BT-74 | Ja | ≥ Start |
| Fälligkeitsdatum | BT-9 | Nein | ≥ Rechnungsdatum |
| Buyer Reference | BT-10 | Ja | XRechnung-Pflicht (Leitweg-ID oder freier Text) |

### Verkäufer (Seller)

| Feld | BT-ID | Pflicht | Validierung |
|---|---|---|---|
| Name | BT-27 | Ja | Min 2 Zeichen |
| Straße | BT-35 | Ja | Nicht leer |
| PLZ | BT-38 | Ja | Länderspezifisch (DE: 5, AT/CH: 4 Ziffern) |
| Ort | BT-37 | Ja | Min 2 Zeichen |
| Land | BT-40 | Ja | ISO 3166-1 alpha-2 |
| USt-ID | BT-31 | Ja* | DE: `DE\d{9}`, AT: `ATU\d{8}`, CH: `CHE-\d{3}\.\d{3}\.\d{3}` |
| Steuernummer | BT-32 | Nein | Alternativ zu USt-ID |
| E-Mail | BT-34 | Nein | E-Mail-Format |
| IBAN | BT-84 | Nein | IBAN-Format |
| BIC | BT-86 | Nein | 8 oder 11 Zeichen |

*USt-ID oder Steuernummer — mindestens eins muss angegeben sein.

### Käufer (Buyer)

| Feld | BT-ID | Pflicht | Validierung |
|---|---|---|---|
| Name | BT-44 | Ja | Min 2 Zeichen |
| Straße | BT-50 | Ja | Nicht leer |
| PLZ | BT-53 | Ja | Länderspezifisch |
| Ort | BT-52 | Ja | Min 2 Zeichen |
| Land | BT-55 | Ja | ISO 3166-1 alpha-2 |
| USt-ID | BT-48 | Nein | Länderspezifisch |

### Rechnungspositionen (Line Items)

| Feld | BT-ID | Pflicht | Validierung |
|---|---|---|---|
| Beschreibung | BT-153 | Ja | Min 2 Zeichen |
| Menge | BT-129 | Ja | > 0, max 2 Dezimalstellen |
| Einheit | BT-130 | Ja | UN/ECE Rec 20 Code |
| Einzelpreis (netto) | BT-146 | Ja | ≥ 0, max 2 Dezimalstellen |
| USt-Kategorie | BT-151 | Ja | S / Z / E / AE |
| USt-Satz | BT-152 | Ja | Abhängig von Land + Kategorie |

Mindestens 1 Position. Dynamisch erweiterbar.

### Berechnungslogik

```
Position Netto  = Menge × Einzelpreis
Position USt    = Position Netto × (USt-Satz / 100)
Position Brutto = Position Netto + Position USt

Gesamt Netto    = Σ Position Netto
Gesamt USt      = Σ Position USt (gruppiert nach USt-Satz für XML)
Gesamt Brutto   = Gesamt Netto + Gesamt USt
```

Rundung: Auf 2 Dezimalstellen pro Position, dann summieren (nicht umgekehrt).

---

## XRechnung-Generierung

### Library: @e-invoice-eu/core

- 100% Client-Side
- Input: JSON-Objekt → Output: CII XML
- Unterstützt XRechnung CIUS

### XRechnung-spezifische Regeln (BR-DE-*)

- BT-10 (Buyer Reference) ist **immer** Pflicht
- BG-16 (Payment Means) ist Pflicht → mindestens Zahlungsart angeben
- Leitweg-ID für öffentliche Auftraggeber
- Seller muss USt-ID ODER Steuernummer haben

### Generierungsflow

```
Formular (Zod-validiert)
  → Mapping auf @e-invoice-eu/core Format
  → core.generate({ format: 'xrechnung-cii' })
  → XML String
  → Blob Download als .xml
```

---

## ZUGFeRD PDF/A-3

### Serverless Endpoint: POST /api/zugferd

**Input:** Invoice-JSON (gleiche Struktur wie Formular-State)
**Output:** `application/pdf` Binary

### Flow

```
Client POST → Server validiert (Zod) → XML generieren → PDF erzeugen
→ XML in PDF/A-3 einbetten → PDF Binary zurück → Client Download
```

### Datenschutz

- Keine Daten werden gespeichert oder geloggt
- Serverless Function ist stateless
- Daten existieren nur während der Request-Verarbeitung

---

## Phasenplan

### Phase 0: Vorbereitung & Evaluation

**Ziel:** Fundament legen, Risiken eliminieren.

**Tasks:**
- [ ] Library-Check: `@e-invoice-eu/core` installieren, Beispiel-XML generieren, gegen Validator prüfen
- [ ] Library-Check: PDF/A-3-Lösung evaluieren (node-zugferd vs. Alternativen)
- [ ] Vite + React + TypeScript + Tailwind + shadcn/ui initialisieren
- [ ] Projektstruktur anlegen (leere Dateien/Ordner)
- [ ] Path-Alias `@/` konfigurieren
- [ ] ESLint + Vitest konfigurieren
- [ ] `pnpm dev` und `pnpm build` laufen fehlerfrei
- [ ] vercel.json anlegen
- [ ] LICENSE (MIT) + README.md Grundgerüst

**Done when:** `pnpm dev` zeigt leere React-App, `pnpm build` + `pnpm lint` laufen durch, Library-Evaluation abgeschlossen und Ergebnis dokumentiert.

### Phase 1: Datenmodell & Kernlogik

**Ziel:** Die gesamte Nicht-UI-Logik steht und ist getestet.

**Tasks:**
- [ ] `lib/constants.ts` — USt-Sätze, Einheiten (UN/ECE Rec 20), Ländercodes, Rechnungsarten
- [ ] `lib/schema.ts` — Zod-Schemas für alle Entitäten (Seller, Buyer, LineItem, Invoice)
- [ ] `lib/calculations.ts` — Netto/USt/Brutto-Berechnung mit korrekter Rundung
- [ ] `lib/format.ts` — Währung/Datum-Formatierung (de-DE Locale)
- [ ] `lib/xrechnung.ts` — Mapping Formular-Daten → @e-invoice-eu/core → XML
- [ ] `lib/download.ts` — Blob-Download Helper
- [ ] `__tests__/schema.test.ts` — Validierungs-Tests (gültig/ungültig je Feld)
- [ ] `__tests__/calculations.test.ts` — Berechnungs-Tests (Rundung, Grenzfälle)
- [ ] `__tests__/xrechnung.test.ts` — XML-Output gegen bekannte Struktur prüfen

**Done when:** Alle Tests grün. Aus einem JSON-Objekt wird valides XRechnung-XML generiert.

### Phase 2: UI & Formular

**Ziel:** Vollständiges, funktionierendes Formular mit Live-Berechnung und XML-Download.

**Tasks:**
- [ ] shadcn/ui Komponenten installieren (Button, Input, Label, Select, Card, Separator, Textarea)
- [ ] `Header.tsx` + `Footer.tsx`
- [ ] `InvoiceMetaSection.tsx` — Rechnungsnummer, Datum, Währung, etc.
- [ ] `SellerSection.tsx` — mit länderspezifischer PLZ/USt-ID-Validierung
- [ ] `BuyerSection.tsx`
- [ ] `LineItemsSection.tsx` — dynamisch Positionen hinzufügen/entfernen
- [ ] `PaymentSection.tsx` — IBAN, BIC, Zahlungsart
- [ ] `TotalsDisplay.tsx` — live-berechnete Summen
- [ ] `InvoiceForm.tsx` — orchestriert alle Sections, RHF + Zod
- [ ] `useInvoiceForm.ts` — Form-State, Berechnung, Submit-Handler
- [ ] `useXRechnung.ts` — XML-Generierung aus validiertem Form-State
- [ ] `DownloadButtons.tsx` — XRechnung XML Download
- [ ] `Preview.tsx` — formatierte Rechnungsvorschau
- [ ] `App.tsx` — Layout zusammenbauen
- [ ] Responsive Design (Mobile-First)

**Done when:** Formular ausfüllen → XML downloaden → XML ist valide bei erechnungs-validator.de.

### Phase 3: ZUGFeRD PDF

**Ziel:** PDF/A-3 mit eingebettetem XML generieren und downloaden.

**Tasks:**
- [ ] `api/zugferd.ts` — Serverless Function implementieren
- [ ] PDF-Template (Rechnungslayout als PDF)
- [ ] XML-Einbettung in PDF/A-3
- [ ] Download-Button in UI erweitern
- [ ] Error Handling (Serverless Timeout, Validierungsfehler)
- [ ] Lokal testen mit `vercel dev`

**Done when:** PDF Download funktioniert, PDF enthält eingebettetes XML, besteht KoSIT-Validator.

### Phase 4: Polish & Deploy

**Ziel:** Produktionsreif.

**Tasks:**
- [ ] SEO: Meta-Tags, OG-Image, Title/Description
- [ ] PWA: manifest.json, Service Worker (offline XML-Generierung)
- [ ] Accessibility: aria-Labels, Keyboard-Navigation, Focus-Management
- [ ] Error States: Nutzerfreundliche Fehlermeldungen
- [ ] Loading States: Spinner für PDF-Generierung
- [ ] README.md: Screenshots, Quick Start, Badges
- [ ] Vercel Production Deploy
- [ ] Smoke Test: Ende-zu-Ende Durchlauf auf Production

**Done when:** Lighthouse >90 (Performance, A11y, SEO), README vollständig, Production-URL funktioniert.

---

## Entscheidungen

| Frage | Entscheidung |
|---|---|
| Domain | Vercel-Subdomain, eigene Domain optional später |
| Impressum | Pflicht (TMG). Platzhalter im Footer, wird vom Maintainer ausgefüllt |
| ZUGFeRD-Scope | Vollständig — XRechnung XML + ZUGFeRD PDF/A-3 |
| UI-Sprache | Nur Deutsch. DACH-Tool. |
| localStorage | Ja, opt-in für Verkäufer-Daten. Ehrlich kommuniziert: "Deine Daten bleiben in deinem Browser." |
| Vorschau | Funktionale HTML-Ansicht. Zeigt Daten korrekt, kein PDF-Layout. |
| Positionierung | Radikal Open Source. Gratis-Tool, kein Marketing. Proof of Concept für Größeres. |

---

## Verifizierung

| Was | Wie |
|---|---|
| XRechnung XML-Validität | [erechnungs-validator.de](https://erechnungs-validator.de/) |
| ZUGFeRD PDF-Validität | [kositvalidator.service-bw.de](https://kositvalidator.service-bw.de/) |
| UStG §14 Pflichtfelder | Unit Tests + manueller Test |
| Berechnung korrekt | Unit Tests (Rundung, Grenzfälle) |
| Responsive | Chrome DevTools (Mobile/Tablet/Desktop) |
| Performance | Lighthouse >90 |
| Build | `pnpm build` fehlerfrei |
| Lint | `pnpm lint` fehlerfrei |
| Tests | `pnpm test` alle grün |

---

## SEO

| Element | Inhalt |
|---|---|
| Title | „E-Rechnung erstellen — Kostenloser XRechnung & ZUGFeRD Generator" |
| Description | „XRechnung und ZUGFeRD kostenlos erstellen. EN 16931 konform, Open Source, kein Account nötig." |
| H1 | „Kostenlos E-Rechnungen erstellen" |
| OG-Image | Screenshot der App |
| Domain | TBD |
