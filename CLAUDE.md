# KIMS Medical Oncology - Cost Estimation App

A single-page web app for generating chemotherapy/immunotherapy cost estimation sheets for patients at KIMS Hospitals, Bengaluru.

## Tech Stack
- **Single file**: `index.html` (HTML + CSS + JS, no build step)
- **Hosting**: GitHub Pages (static site) with custom domain `kimsmedoncestimate.in`
- **Print**: Browser print dialog → Save as PDF

## Project Structure
```
index.html          # Entire app — markup, styles, and logic
kims-macs-logo.jpg  # KIMS MACS logo (used in header + print sheet)
```

## Key Architecture
- All drugs stored in `drugDatabase` array with fields: `brand`, `generic`, `mrp`, `type`, `contentMg`, and optionally `form` (`vial`/`tablet`/`capsule`/`kit`) and `unitsPerPack`
- Drug types: `chemotherapy`, `immunotherapy`, `targeted`, `hormonal`, `supportive`
- Tab buttons filter the generic dropdown by `type`
- Hospital charges (consumables, consultation, ward, procedure) are selectable dropdowns
- Print sheet is generated dynamically in `generatePrint(mode)` — two modes: `'patient'` and `'doctor'`
- Patient print: 3-line cost breakdown (Drugs + Supportive, Procedure Charges, Hospital Charges) + notes + signatures
- Doctor print: detailed drug list with quantities + hospital charges breakdown
- App summary shows all line items individually
- Mobile responsive via `@media (max-width: 600px)` — stacked layout without changing desktop design

## Dose Calculator
- Enter Height, Weight, Age, Sex, Serum Creatinine → auto-calculates BSA (Mosteller) and GFR (Cockcroft-Gault, capped at 125)
- Select regimen → calculates per-drug doses and optimal vial/tablet indent
- **Formulas**: BSA = √(ht × wt / 3600), GFR = ((140-age) × wt) / (72 × Cr) × 0.85 if female, Carboplatin = AUC × (GFR + 25)
- **Dose types**: `mg_per_m2`, `auc` (Calvert), `mg_per_kg`, `flat_mg`, `oral_bd_14d` (capecitabine)
- `timesPerCycle` for multi-day or weekly drugs within a cycle (e.g., paclitaxel weekly ×3 in Q3W, ifosfamide D1-3)
- **Smart vial optimizer** (`optimizeVials`): never mixes brands — optimizes within each brand family separately (via `_optimizeFamily`), picks cheapest family result; within a family tries all combinations, finds cheapest, then within 5% of cheapest prefers fewer vials for practicality
- **Brand family**: extracted via `getBrandFamily()` — prefix before first digit in brand name (e.g. `"OXALTERO 50MG INJ"` → `"OXALTERO"`). Same brand, different vial sizes = same family and can be combined. Different brands = never mixed.
- **Dose rounding tolerance**: injectable chemotherapy vials (`type === 'chemotherapy'`, form not tablet/capsule) allow rounding down up to **10%** of the calculated dose if a cheaper vial combination covers ≥ 90% of the dose. When rounding occurs, the results table shows `⚠ Giving: Xmg (rounded from Ymg)` in amber. Immunotherapy, targeted, hormonal, supportive, and oral drugs use `tolerance = 0` (no rounding).
- **Drug forms**: vials, tablets, capsules, kits — `getUnitLabel()` helper returns appropriate labels (tabs/caps/kits/vials)
- Pack display: for drugs with `unitsPerPack`, shows number of packs/strips needed
- Typing in the patient details Regimen field auto-selects matching regimen in the calculator (all words must match, min 2 chars each)
- Results stay visible after adding drugs to estimate

## Regimens
Regimens stored in `regimenDatabase` array, **sorted alphabetically**. New regimens must be inserted in alphabetical order.

Current regimens: ABCP, AC, AC+Pembrolizumab (KEYNOTE-522), AP, CAPOX, Carboplatin+Paclitaxel (3-weekly), Carboplatin+Paclitaxel (weekly), Carboplatin+Paclitaxel+Bevacizumab (Ovarian), Carboplatin+Paclitaxel+Pembrolizumab (KEYNOTE-522), Carboplatin single agent (weekly), Cisplatin single agent (weekly), FLOT, FOLFIRI, IA, mFOLFOX6, Paclitaxel+Trastuzumab (APT trial weekly), Pemetrexed+Carboplatin, TC, TCH, TCHP, TPF

## Consultants
- Dr Suresh Babu MC
- Dr Prathyusha Eaga
- Dr Anup R Hegde
- Dr Swathi Prakash
- Dr Vivek B Maleyur
- Dr Chetan V

## Drug Categories
- **Chemotherapy**: Carboplatin, Cisplatin, Cyclophosphamide, Paclitaxel, Docetaxel, Doxorubicin, Epirubicin, Fluorouracil, Oxaliplatin, Irinotecan, Capecitabine, Nab-Paclitaxel, Pemetrexed, Ifosfamide, Thalidomide
- **Immunotherapy**: Pembrolizumab (Keytruda), Atezolizumab (Tecentriq)
- **Targeted Therapy**: Bevacizumab (Avastin/Bevatas), Rituximab (Mabtas), Trastuzumab (Vivitra), Pertuzumab (Pertuza), Ribociclib (Kryxana)
- **Hormonal Therapy**: Leuprolide (Leulide)
- **Supportive Care**: Antiemetics (Akynzeo, Nykron, Aprecap, Palnox, Fosaprepitant), GCSF/PEG-GCSF (Neukine, Peg Religrast), IVIG (Globucel, Immunorel), Mesna, Zoledronic Acid (Zoltero)

## Hospital Charges
- **Procedure Types**: Chemo Infusion Single/Multi Drug, Chemotherapy Minor/Medium/Major/Major Plus/Supra Major, Immunotherapy
- **Consumables**: Rs. 5,000 or Waive Off
- **Consultation**: Rs. 1,600 / Rs. 2,000 / Waive Off
- **Ward**: Day Care (Rs. 2,500) / Private (TBD)

## Print Sheet
- Two print modes: `generatePrint('patient')` and `generatePrint('doctor')`
- **Patient print**: KIMS MACS logo, patient details, 3-line cost breakdown (Drugs + Supportive, Procedure Charges, Hospital Charges with consumables/ward/consultation), total, +/- 10% variance disclaimer, acknowledgement with signature fields
- **Doctor print**: detailed drug list table (brand, generic, qty, unit price, total) + hospital charges breakdown
- Designed to fit on a single A4 page

## Conventions
- Drug brand names: UPPERCASE (e.g., "CAPEGARD 500MG TAB", "CISTERO 50MG")
- Generic names in regimens must exactly match `generic` field in `drugDatabase` (e.g., `"BEVACIZUMAB"` not `"BEVACIZUMAB (AVASTIN)"`)
- Generic names: UPPERCASE, common aliases in brackets (e.g., "DOXORUBICIN (ADRIAMYCIN)", "FLUOROURACIL (5-FU)")
- MRP prices in Indian Rupees with Indian number formatting (e.g., Rs. 1,23,506.00)
- All amounts use `toLocaleString('en-IN')` for formatting
- Quantity default: 0 (validation requires qty >= 1 before adding)
- Some drug prices are placeholders — verify with KIMS pharmacy (notably: Pemetrexed, Pertuzumab, Ifosfamide)
