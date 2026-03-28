# E-Rechnung Generator

> Kostenloser, quelloffener E-Rechnungs-Generator fuer Europa -- EN 16931 konforme XRechnung, Factur-X und Peppol BIS direkt im Browser.

## Status

Aktiv -- Planungsphase abgeschlossen, Implementierung beginnt.

## Quick Start

```bash
# Voraussetzungen: Node.js 18+, pnpm
pnpm install
pnpm dev        # Entwicklungsserver starten
pnpm build      # Production Build
pnpm test       # Tests ausfuehren
pnpm lint       # Linting
```

## Features

- **19 europaeische Laender** zum Launch (DE, AT, CH, FR, NL, BE, LU, DK, SE, FI, NO, EE, LV, LT, IE, SI, HR, SK, CZ)
- **3 E-Rechnungsformate:** XRechnung CII, Factur-X, Peppol BIS UBL
- **ZUGFeRD/Factur-X PDF/A-3** mit eingebettetem XML
- **100% client-seitige XML-Generierung** -- keine Daten verlassen den Browser
- **Kein Account, keine Registrierung** -- einfach Formular ausfuellen und runterladen
- **Mehrsprachig:** Deutsch, Englisch, Franzoesisch, Niederlaendisch (+ Community-Beitraege)
- **Laenderspezifische Validierung:** USt-Saetze, USt-ID-Formate, PLZ-Formate pro Land
- **Seller-Daten speichern** (opt-in, localStorage -- bleibt im Browser)
- **Open Source (MIT)** -- radikal offen, gratis, Made in Europe

## Laender hinzufuegen

1. Erstelle `src/data/countries/{code}.ts` nach dem `CountryConfig` Interface
2. Registriere es in `src/data/countries/index.ts`
3. Optional: Locale-Datei unter `src/i18n/locales/{lang}.json`
4. Tests in `__tests__/countries.test.ts`
5. PR oeffnen

## Links

- GitHub: https://github.com/V3SP45/e
- Hosting: Vercel (Free Tier)
- Lizenz: MIT
