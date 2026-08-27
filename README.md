# Jejak Hijau KPJU

Campus sustainability pilot for KPJ Healthcare University (KPJU) — SPARK Challenge 2026.

## What this is, and what it isn't

This is a single-file dashboard that tracks two things no one currently measures anywhere in KPJ's reporting: electricity intensity (kWh per student) and paper intensity (sheets per student) at KPJU specifically. It is not a copy of KPJ Group's ESG reporting, and it doesn't try to be — the Group already publishes Group-level figures. The gap this fills is that KPJU, despite being named in the Group's own Sustainability Report as the university that "builds capability from within," has no unit-level sustainability data of its own. This project is that data, starting from zero.

It is currently **pilot-ready, not piloted**. Every number visible in the dashboard right now is placeholder data, clearly labeled as such. Nothing here should be read as a live KPJU figure until Finance, the Exam Unit, and the Registrar have actually supplied data.

## Why single-file, no backend

Same reasoning as every other Aufthority tool in this portfolio: a SPARK prototype needs to be demoable and deployable with zero setup friction. There's no user data being collected here that would justify a database — the dashboard's whole job is to visualize whatever spreadsheet it's given. A backend would add deployment risk for no functional gain at this stage. If this ever moves past pilot into an actual recurring institutional tool, that calculation changes — worth revisiting then, not now.

## How the numbers are supposed to work

The dashboard reports two metrics per semester, both normalized per student:

- **Electricity → Scope 2 emissions.** kWh/student is measured campus-wide (electricity is metered at the building/campus level, not per cohort, so the denominator has to be the full student population, not a single course). It's converted to tCO₂e/student using Malaysia's grid emission factor, and reported using the GHG Protocol's dual method — location-based and market-based — because that's the standard's own recommended disclosure, not something invented for this project.
- **Paper → a waste-volume metric, deliberately not converted to emissions.** Sheets/student comes from the Exam Unit. It's reported as a plain volume, not translated into a carbon figure, because that conversion would require supply-chain data (paper sourcing, manufacturing emissions) that KPJU doesn't have access to. The instinct to force every metric into a CO₂ number was a real temptation while building this — resisted deliberately, because a precise-looking number built on data we don't have is worse than an honest volume metric.

**Scope 1 and Scope 3 are shown in the dashboard's GHG Protocol panel but explicitly marked as scoped-for-later, not measured.** Scope 1 (generators, refrigerants, any owned vehicles) needs facilities records that haven't been gathered yet, and is likely small relative to Scope 2 — not worth diluting v1 with a rough estimate. Scope 3 beyond paper (procurement, commuting, etc.) is a much larger undertaking that's out of scope for a pilot.

**DPH 48 is not the reported cohort.** It shows up in the cohort comparison table, but tagged "pilot only" — its purpose is to validate that the Exam Unit can actually supply clean per-cohort paper data before the full School of Pharmacy figure is requested. If you see DPH 48-only numbers being cited as *the* KPJU figure anywhere outside this validation step, that's a scoping error — the real reported metric is always the full-population number.

## Data flow: two ways in, same destination

The dashboard accepts data two ways — a manual file upload (CSV or XLSX) and a "load from link" field — and both feed the same parsing function, so they behave identically once loaded.

The link-loading option exists because the plan is to eventually point it at a OneDrive-hosted sheet instead of manually re-uploading a file every time. This has not been tested against a real OneDrive link yet, and there's a real chance it won't work as-is: OneDrive's normal sharing links are built for browser viewing, not for a script on another domain to fetch raw bytes from (a CORS restriction). If the OneDrive link fails, the known-reliable fallback is a Google Sheets "Publish to web → CSV" link, which is explicitly designed for this kind of anonymous cross-origin fetch. This should be tested early, not assumed to work, once a real link exists.

Expected columns, regardless of source: `semester, students, kwh, cost_rm, sheets`. Extra columns are ignored, not rejected — the dummy dataset includes two status-flag columns that the dashboard simply skips over.

## Where the targets live

Semester reduction targets (currently 42 kWh/student, 30 sheets/student) are hardcoded in the dashboard's JavaScript (`targets` array), not read from the uploaded sheet. The dummy dataset's "Targets" tab mirrors these values for reference, but changing the spreadsheet alone won't move the target bars — both places need to be updated together if targets change. This is a known seam, not a bug: targets change rarely enough that wiring them through the upload didn't seem worth the added complexity yet.

## File inventory

- `jejak_hijau_kpju.html` — the dashboard. Single file, no build step.
- `Jejak_Hijau_KPJU_Dummy_Data.xlsx` — placeholder dataset, deliberately modeled as over-target and worsening on both metrics, for testing and for planning an improvement-strategy narrative before real data exists.
- `Jejak_Hijau_KPJU_SPARK_Narrative.docx` — the submission narrative, structured around SPARK's judging criteria.
- `KPJU_Sustainability_Gap_Datasheet.xlsx` — consolidated KPJ Group figures with an explicit gap map showing where KPJU has no equivalent data, plus the data-collection checklist (who to ask, for what).

## What's actually been verified vs. assumed

Verified: the dummy workbook parses correctly through the same SheetJS logic the dashboard runs in-browser — columns map, status flags compute, the sheet order (Semester Data must be first) works as expected.

Assumed, not yet verified: that a real OneDrive link will load without CORS issues; that the cited KPJ Group figures (ESG scores, GET enrollment count, etc.) are still accurate — they were pulled from search results during planning, not a fresh read of the source PDF, and should be checked against the actual Sustainability Report before anything here goes external.

## Open items before this moves past pilot

- Finance, Exam Unit, and Registrar conversations haven't started — no live data exists yet.
- OneDrive link untested; Google Sheets fallback identified but also untested.
- Source figures for the Group-level comparisons need a direct verification pass.
- Working title "Jejak Hijau KPJU" is locked; visual polish beyond the current Grain-system styling hasn't been prioritized, since substance mattered more than polish at this stage.
