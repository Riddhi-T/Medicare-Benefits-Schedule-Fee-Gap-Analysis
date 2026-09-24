# MBS Out-of-Pocket Gap Analysis
A Power BI dashboard analyzing patient out-of-pocket cost exposure across Australia's
Medicare Benefits Schedule (MBS), built from the official government XML data release.
---

## Headline Finding
Out-of-hospital (85% benefit rate) patients are protected by a **legislated maximum
gap of $104.50** per service, regardless of how expensive the procedure is. In-hospital
(75% benefit rate) patients face **no equivalent cap** — for the same $9,999.95
procedure, the gap reaches **$2,499.95, over 24x higher**.
This isn't evenly spread across service types either: Therapeutic Procedures carry
the highest average in-hospital gap ($242), while even the highest average
out-of-hospital gap (Cleft and Craniofacial Services, $54) sits nowhere near it —
evidence that the missing cap, not the category itself, drives the disparity.
---

## Data Source
- **Medicare Benefits Schedule (MBS)** XML release, dated **1 August 2026**, published
  by [MBS Online](https://www.mbsonline.gov.au) (Department of Health, Disability and
  Ageing).
- 6,046 items as published in this release.
- Figures reflect the **government Schedule Fee and benefit structure only** — not
  what providers actually charge, nor what patients pay after private health insurance.
### Category & Group name sourcing
The MBS XML only provides numeric codes with no text labels. Names were verified
against official primary sources:
| Codes | Source |
|---|---|
| Categories 1–8, Groups A1–T (91 codes) | [MBS Book, July 2026](https://www.mbsonline.gov.au/internet/mbsonline/publishing.nsf/Content/downloads), Table of Contents |
| Category 10, Groups U0–U9 (dental) | [Dental Benefits Rules 2014](https://www.legislation.gov.au/F2026L00110) (Federal Register of Legislation), Schedule 1 — dental services sit under separate legislation (*Dental Benefits Act 2008*) and are not covered by the MBS Book |
| Category 7 rename (2024) | Officially renamed from "Cleft Lip and Cleft Palate Services" to ["Cleft and Craniofacial Services"](https://mbsonline.gov.au/internet/mbsonline/publishing.nsf/Content/A36606F736551929CA258B2C0008F9E6/$File/PDF%20Version%20%E2%80%93%20Changes%20to%20Category%207%20%E2%80%93%20Cleft%20and%20Craniofacial%20Services.pdf) |

**Note:** Category 9 does not appear in the MBS Book or the source data — no official
explanation for the gap in the numbering was located during this project.
---

## ETL Pipeline
Built in Python (pandas + `xml.etree.ElementTree`), full script in [`build_mbs_dataset.py`](./build_mbs_dataset.py).
1. **Parse** the XML into a flat DataFrame (one row per item, one column per field)
2. **Type conversion** — fee/benefit fields to numeric, `ItemStartDate` to datetime
3. **Category & Group name mapping** — via the sourced lookups above
4. **Split** items by `FeeType`: 84 "derived fee" items (fee is a formula, not a flat
   amount — e.g. item 4, calculated per-patient) excluded from the numeric gap
   calculation; 5,962 "normal fee" items retained
5. **Reshape wide → long by `BenefitType`** — since ~40% of items have two valid
   benefit rates (e.g. 75% in-hospital *and* 85% out-of-hospital for the same
   service), each item becomes one row per applicable tier. Result: 8,359 rows.
6. **Calculate** `FeeGap` (ScheduleFee − BenefitAmount) and `GapPct`
7. **Export** to CSV for the Power BI data model
---

## Key Mechanism: the Safety Net Gap Cap
For out-of-hospital (85%) services, Medicare pays *more* than 85% of the fee once the
uncapped 15% gap would otherwise exceed $104.50 — the benefit rate effectively scales
up above 85% for expensive items, specifically to keep the patient's gap capped at
$104.50. Below roughly **$697** in Schedule Fee, the standard 15% applies with no
override; above that threshold, the gap flattens at the cap.
No equivalent mechanism exists for in-hospital (75%) services in this dataset — the
gap continues to scale linearly with the Schedule Fee with no ceiling observed, up to
$2,499.95 for the highest-value item in the dataset ($9,999.95).
---

## Limitations
- In-hospital vs. out-of-hospital setting is generally determined by **clinical
  necessity, not patient choice** — this finding should not be read as a
  recommendation to seek out-of-hospital care.
- This analysis reflects only the **government-set Schedule Fee gap**. It does not
  account for:
  - **Private health insurance**, which frequently covers some or all in-hospital costs
  - **Provider charges above the Schedule Fee**, which occur in both settings and are
    common in practice
- Real-world patient costs may differ substantially from the figures shown.
---

## Tools
Python (pandas, ElementTree) · Power BI (Power Query, DAX) · Data current as of the
1 August 2026 MBS release.



















