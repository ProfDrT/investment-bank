# investment-bank

## Executive Overview – Kernprozesse Investmentbank (Mittelstand)

- Ziel: Ein prägnantes, visuelles Cockpit aller Kernprozesse mit schnell erfassbaren Abläufen, Rollen, Zeitplänen und Kontrollpunkten.
- Fokus: M&A/Finanzierung (CF), IPO/ECM, DCM, Sales & Trading inkl. DS, Research, Risk/Compliance, Operations/IT.

## Gesamtbild (Front/Middle/Back Office)

```mermaid
flowchart TB
  subgraph FO["Front Office"]
    CF["Corporate Finance (M&A/Finanzierung)"]
    ECM["ECM (IPO/Kapitalmaßnahmen)"]
    DCM["DCM (Anleihen/Schuldschein)"]
    ST["Sales & Trading / DS"]
    RES["Research"]
  end
  subgraph MO["Middle Office"]
    RISK["Risk Management"]
    COMP["Compliance"]
  end
  subgraph BO["Back Office"]
    OPS["Operations"]
    IT["IT/Plattform"]
  end

  CF --> ECM
  ECM --> ST
  DCM --> ST
  RES --> ST
  ST --> OPS
  RISK -. "Pre-Trade/Limit" .-> CF
  RISK -. "Pre-Trade/Limit" .-> ECM
  RISK -. "Pre-Trade/Limit" .-> DCM
  COMP -. "KYC/Publikationen" .-> CF
  COMP -. "KYC/Publikationen" .-> ECM
  IT --- FO
  IT --- MO
  IT --- BO
```

## Navigation

- Corporate Finance – M&A/Finanzierung: `01_Corporate_Finance_MA_Finanzierung.md`
- ECM – IPO: `02_ECM_IPO.md`
- DCM – Anleiheemission: `03_DCM_Anleiheemission.md`
- ECM – Kapitalerhöhungen & Secondary: `04_ECM_Kapitalerhoehung_Secondary.md`
- Sales & Trading inkl. DS: `05_Sales_Trading_Designated_Sponsoring.md`
- Research: `06_Research.md`
- Risk & Compliance: `07_Risk_Compliance.md`
- Operations & IT: `08_Operations_IT.md`
