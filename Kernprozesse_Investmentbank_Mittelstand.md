# Kernprozesse einer Investmentbank (Mittelstand) – Visualisierung & Kurzleitfaden

- Ziel: Klarer, visueller End-to-End-Überblick der Kernprozesse.
- Fokus: Mittelstand, Full-Service-Investmentbank, Front/Middle/Back Office.
- Stil: Prägnant, MECE, praktikabel für Umsetzung und Diskussion.

---

## Navigation

- Executive Overview: [README.md](README.md)
- Corporate Finance – M&A/Finanzierung: [01_Corporate_Finance_MA_Finanzierung.md](01_Corporate_Finance_MA_Finanzierung.md)
- ECM – IPO: [02_ECM_IPO.md](02_ECM_IPO.md)
- DCM – Anleiheemission: [03_DCM_Anleiheemission.md](03_DCM_Anleiheemission.md)
- ECM – Kapitalerhöhungen & Secondary: [04_ECM_Kapitalerhoehung_Secondary.md](04_ECM_Kapitalerhoehung_Secondary.md)
- Sales & Trading inkl. DS: [05_Sales_Trading_Designated_Sponsoring.md](05_Sales_Trading_Designated_Sponsoring.md)
- Research: [06_Research.md](06_Research.md)
- Risk & Compliance: [07_Risk_Compliance.md](07_Risk_Compliance.md)
- Operations & IT: [08_Operations_IT.md](08_Operations_IT.md)

Vorlagen/Templates:
- Teaser: [templates/Teaser_Template.md](templates/Teaser_Template.md)
- CIM-Outline: [templates/CIM_Outline.md](templates/CIM_Outline.md)
- Allocation Policy: [templates/Allocation_Policy_Template.md](templates/Allocation_Policy_Template.md)
- CP-Checklist (Closing Conditions): [templates/CP_Checklist_Template.md](templates/CP_Checklist_Template.md)

---

## Organisationsüberblick (Front/Middle/Back Office)

```mermaid
flowchart TB
  %% Organisationssicht mit Kernbereichen
  subgraph FO[Front Office]
    CF["Corporate Finance (M&A/Finanzierung)"]
    ECM["Kapitalmärkte (ECM/DCM)"]
    ST[Sales & Trading]
    RES[Research]
  end
  subgraph MO[Middle Office]
    RISK[Risikomanagement]
    COMP[Compliance]
  end
  subgraph BO[Back Office]
    OPS["Operations (Clearing, Settlement, Reconciliation)"]
    IT["IT & Datenplattform"]
    FIN[Finance, Legal, HR]
  end

  CF --> ECM
  ECM --> ST
  ST --> OPS
  RES --> ST
  RISK -. Pre/Limit/Kredit .-> CF
  RISK -. Pre/Limit/Kredit .-> ECM
  COMP -. KYC/Publikationen/Chinese Wall .-> CF
  COMP -. KYC/Publikationen/Chinese Wall .-> ECM
  OPS --> FIN
  IT --- FO
  IT --- MO
  IT --- BO
```

---

## Gesamtfluss (Kunde – Bank – Investoren)

```mermaid
flowchart LR
  %% Lanes
  subgraph U["Kunde (Mittelstandsunternehmen)"]
    U0(["Kapitalbedarf / M&A-Initiative"])
  end
  subgraph IB["Investmentbank"]
    direction TB
    CF[["Corporate Finance: Mandat → DD → Bewertung → Verhandlung → Signing/Closing"]]
    ECM[["ECM/DCM: Strukturierung → Prospekt → Marketing → Bookbuilding → Pricing → Zuteilung"]]
    ST[["Sales & Trading: Placement → Market Making → Sekundärhandel"]]
    RES[["Research: Coverage → Modell → Empfehlung → Updates"]]
    RISK[["Risk & Compliance: KYC, Limits, Freigaben, Monitoring"]]
    OPS[["Operations & IT: Bestätigung → Matching → Clearing → Settlement → Reporting"]]
  end
  subgraph INV["Investoren"]
    INV0(["Institutionelle/Family Offices/Banken"])
  end

  U0 --> CF
  CF -- "M&A (Sell-Side/Buy-Side)" --> INV0
  CF -- "Kapitalaufnahme" --> ECM
  ECM --> INV0
  INV0 -- "Zeichnung/Orders" --> ECM
  ECM --> ST
  ST <--> INV0
  ST --> OPS
  CF --> RISK
  ECM --> RISK
  ST --> RISK
  RISK -. "Freigaben/Policies" .-> CF
  RISK -. "Freigaben/Policies" .-> ECM
  CF --> OPS
  ECM --> OPS
  OPS --> INV0
  RES --> ST
  RES --> INV0
```

---

## Prozessdetails (prägnant + visuell)

### 1) Corporate Finance – M&A (Fokus Mittelstand)

- Zweck: Wertmaximierender Verkauf/Zukauf; Nachfolge (MBO/MBI) strukturiert umsetzen.
- Output: Signiertes SPA, Preis/Konditionen, gesicherte Finanzierung, Closing.
- Schlüsselentscheidungen: Prozessdesign (Bieterverfahren vs. Bilateral), Valuation-Band, Deal-Struktur.

```mermaid
flowchart TB
  A[Pitch & Mandat] --> B[Vorbereitung: Teaser, IM, Datenraum]
  B --> C[Long/Short List & NDA]
  C --> D[Management-Präsentationen]
  D --> E[Indikative Angebote]
  E --> F[Due Diligence koordinieren]
  F --> G[Bewertung & Deal-Struktur]
  G --> H[SPA-Verhandlung]
  H --> I[Signing]
  I --> J["Closing (Zahlung/Übergang)"]
  %% Buy-Side analog: Origination → Screening → DD → Finanzierung → Verhandlung → Closing
```

### 2) Kapitalaufnahme – IPO (ECM)

- Zweck: Eigenkapital für Wachstum, Sichtbarkeit, Investorenbasis verbreitern.
- Output: Zulassung, Erlös, Free Float, stabile Erstnotiz.
- Schlüsselentscheidungen: Timing, Preisspanne, Volumen, Allokation, Stabilisierungsrahmen.

```mermaid
flowchart LR
  A[Vorbereitung: Börsenreife, Umstrukturierung] --> B["Prospekt/Registrierung (BaFin/Börse)"]
  B --> C["Pre‑Marketing/Analystenpräsentation"]
  C --> D["Roadshow & Bookbuilding"]
  D --> E["Pricing & Zuteilung"]
  E --> F["Erstnotiz/Handelsstart"]
  F --> G["Stabilisierung (Greenshoe)"]
  G --> H["Research-Coverage & Sekundärmarkt"]
```

### 3) Kapitalaufnahme – Anleiheemission (DCM)

- Zweck: Fremdkapital zu passenden Laufzeiten/Konditionen mobilisieren.
- Output: Emittierte Schuldverschreibungen, erfolgreicher Platzierungsgrad, Listing (optional).
- Schlüsselentscheidungen: Volumen, Laufzeit, Kupon/Preis, Covenants, Rating.

```mermaid
flowchart LR
  A["Strukturierung & Terms"] --> B["Dokumentation/Prospekt + ggf. Rating"]
  B --> C["Investorenansprache/Fixed Income Marketing"]
  C --> D[Bookbuilding & Preisfindung]
  D --> E[Zuteilung]
  E --> F["Settlement & ggf. Börsenlisting"]
```

### 4) Sales & Trading inkl. Designated Sponsoring

- Zweck: Liquidität, faire Preisfindung, Investorenversorgung; Platzierungsunterstützung.
- Output: Ausgeführte Orders, enge Spreads, verlässlicher Sekundärmarkt.
- Schlüsselentscheidungen: Ausführungsstrategie, Risikolimits, Sponsoring-Parameter.

```mermaid
flowchart LR
  S[Sales: Kundenauftrag/Idea Push] --> O[Order Capture]
  O --> T[Trading: Ausführung]
  T --> M[Börse/MTF]
  T --> MM{Designated Sponsor?}
  MM -- Ja --> Q[Quotes: Bid/Ask stellen]
  Q --> M
  M --> C[Post-Trade: Confirmation]
  C --> L[Clearing & Settlement]
  L --> R["Reconciliation/Reporting"]
```

### 5) Research (Small-/Mid-Cap Fokus)

- Zweck: Fundierte, unabhängige Analysen; Informationsvorsprung für Kunden; Deal‑Sourcing.
- Output: Initiations- und Updates, Modelle, Empfehlungen, Events.
- Schlüsselentscheidungen: Coverage-Universum, Annahmen, Trigger, Bewertung.

```mermaid
flowchart TB
  U[Universe/Branchen-Scan] --> I[Initiating Coverage]
  I --> M[Finanzmodell & Bewertung]
  M --> R[Report & Empfehlung]
  R --> D[Distribution an Kunden]
  D --> U1["Feedback/Investorengespräche"]
  U1 --> Up["Updates/Ad-hoc"]
```

### 6) Risk & Compliance (Middle Office)

- Zweck: Schutz der Bank; Regelkonformität; robuste Limit- und KYC-Prozesse.
- Output: Freigaben, Überwachung, Eskalationen, Berichte.
- Schlüsselentscheidungen: Limits, Kreditlinien, Konfliktmanagement, Eskalationspfade.

```mermaid
flowchart LR
  K[KYC/Onboarding] --> P[Pre-Trade Checks: Limits, Conflicts]
  P --> A{Freigabe?}
  A -- Nein --> ESC[Eskalation/Anpassungen]
  A -- Ja --> EX["Ausführung (Front Office)"]
  EX --> MON[Post-Trade Monitoring/Surveillance]
  MON --> REP[Regulatorische/Management-Reports]
```

### 7) Operations & IT (Back Office)

- Zweck: Fehlerfreie Abwicklung, Datenqualität, Stabilität, BAIT-konforme IT.
- Output: Bestätigte/abgerechnete Trades, saubere Bücher, Reports.
- Schlüsselentscheidungen: Automatisierungsgrad, Systemarchitektur, Kontrollen, Notfallkonzepte.

```mermaid
flowchart TB
  TC["Trade Capture (FO/OMS)"] --> CNF["Bestätigung (Confirmations)"]
  CNF --> MT[Matching & Affirmation]
  MT --> CLR[Clearing]
  CLR --> SET["Settlement (DVP)"]
  SET --> REC[Reconciliation]
  REC --> CA[Corporate Actions]
  CA --> REP[Interne & regulatorische Reports]
  REP --> Q["Qualitätssicherung/Abstimmungen"]
```

---

## Hinweise für Umsetzung/Steuerung

- Governance: Klare Verantwortlichkeiten je Prozess; verbindliche RACI-Matrix.
- Schnittstellen: Definierte Übergaben zwischen FO/MO/BO; minimale manuelle Brüche.
- Daten & IT: Einheitliche Datenmodelle; revisionssichere Speicherung; API‑First.
- KPI/Controlling: Durchlaufzeiten, Platzierungsgrad, Fehlerraten, P&L/Value at Risk, Compliance‑Findings.
- Regulierung: KYC/AML, Marktmissbrauch, Prospektrecht, BAIT; regelmäßige Audits/Tests.

> Vereinfachte, aber praxistaugliche Darstellung. Konkrete Schritte je Jurisdiktion und Produkt feingranular anzupassen.
