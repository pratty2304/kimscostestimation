# KIMS Medical Oncology - Cost Estimation App

A single-page web app for generating chemotherapy/immunotherapy cost estimation sheets for patients at KIMS Hospitals, Bengaluru.

## Tech Stack
- **Single file**: `index.html` (HTML + CSS + JS, no build step)
- **Hosting**: GitHub Pages (static site) with custom domain `kimsmedoncestimate.in`
- **Print**: Browser print dialog → Save as PDF

## Project Structure
```
index.html          # Entire app — markup, styles, and logic
kims-onco-sciences-logo.png  # KIMS Onco Sciences logo (used in overlay, header, and print sheet)
```

## Key Architecture
- Branch selection overlay appears first; choosing a branch sets `currentBranch` and `currentDrugDatabase`
- Branch config lives in `BRANCH_CONFIG`, including branch subtitle, drug database, and consultant list
- Drug databases are branch-specific: `mahadevapuraDrugDatabase` and `electronicCityDrugDatabase`
- Drug records use fields: `brand`, `generic`, `mrp`, `type`, `contentMg`, and optionally `form` (`vial`/`tablet`/`capsule`/`kit`) and `unitsPerPack`
- Drug types: `chemotherapy`, `immunotherapy`, `targeted`, `hormonal`, `supportive`
- Tab buttons filter the generic dropdown by `type`
- Brand dropdowns sort highest MRP first and auto-select the highest-MRP brand by default; users can manually choose a lower-cost brand later
- Hospital charges (consumables, consultation, ward, procedure) are selectable, and `numberOfDays` multiplies these charges
- Print sheet is generated dynamically in `generatePrint(mode)` — two modes: `'patient'` and `'doctor'`
- Patient print: 3-line cost breakdown (Drugs + Supportive, Procedure Charges, Hospital Charges) + notes + signatures
- Doctor print: detailed drug list with quantities + hospital charges breakdown
- App summary shows all line items individually
- Mobile responsive via `@media (max-width: 600px)` — stacked layout without changing desktop design

## Dose Calculator
- Enter Height, Weight, Age, Sex, Serum Creatinine → auto-calculates BSA (Mosteller, capped at 2.00 m² for dosing) and GFR (Cockcroft-Gault, capped at 125)
- Select regimen → calculates per-drug doses and optimal vial/tablet indent
- **Formulas**: BSA = √(ht × wt / 3600), GFR = ((140-age) × wt) / (72 × Cr) × 0.85 if female, Carboplatin = AUC × (GFR + 25)
- **Dose types**: `mg_per_m2`, `auc` (Calvert), `mg_per_kg`, `flat_mg`, `oral_bd_14d` (capecitabine), `oral_daily`
- `timesPerCycle` for multi-day or weekly drugs within a cycle (e.g., paclitaxel weekly ×3 in Q3W, ifosfamide D1-3)
- **Smart vial optimizer** (`optimizeVials`): never mixes brands — optimizes within each brand family separately (via `_optimizeFamily`) and picks the highest-MRP eligible family result by default
- Within a brand family, different vial sizes can be combined to cover the dose (e.g., oxaliplatin 150 mg can use `OXALTERO 100MG INJ` + `OXALTERO 50MG INJ`)
- **Brand family**: extracted via `getBrandFamily()` — prefix before first digit in brand name (e.g. `"OXALTERO 50MG INJ"` → `"OXALTERO"`). Same brand, different vial sizes = same family and can be combined. Different brands = never mixed.
- **Dose rounding tolerance**: injectable chemotherapy vials (`type === 'chemotherapy'`, form not tablet/capsule) allow rounding down up to **10%** of the calculated dose if a valid same-brand combination covers ≥ 90% of the dose. When rounding occurs, the results table shows `⚠ Giving: Xmg (rounded from Ymg)` in amber. Immunotherapy, targeted, hormonal, supportive, and oral drugs use `tolerance = 0` (no rounding).
- **Oral daily drugs**: choose the highest-MRP matching strength by default and multiply by days; do not mix strengths unless no exact strength exists
- **Drug forms**: vials, tablets, capsules, kits — `getUnitLabel()` helper returns appropriate labels (tabs/caps/kits/vials)
- Pack display: for drugs with `unitsPerPack`, shows number of packs/strips needed
- Typing in the patient details Regimen field auto-selects matching regimen in the calculator (all words must match, min 2 chars each)
- Results stay visible after adding drugs to estimate

## Regimens
Regimens stored in `regimenDatabase` array, **sorted alphabetically**. New regimens must be inserted in alphabetical order.

Current regimen set includes: EXTREME regimens, ABCP, ABVD, AC, AC+Pembrolizumab, AP, cabozantinib+nivolumab, CAPOX, capecitabine with RT, carboplatin/paclitaxel variants, cisplatin/carboplatin with RT, CROSS, docetaxel single agent, EC, FLOT, FLOT+Durvalumab, FOLFIRI variants, FOLFIRINOX variants, FOLFOXIRI, gemcitabine variants, IA, irinotecan variants, mFOLFOX6 variants, nab-paclitaxel variants, paclitaxel variants, Pola-R-CHP, pemetrexed/carboplatin variants, R-CHOP, TC, TCH, TCHP, TPF, and TIP variants.

## Consultants
Consultants are branch-specific via `BRANCH_CONFIG`.

Mahadevapura:
- Dr Suresh Babu MC
- Dr Prathyusha Eaga
- Dr Vivek B Maleyur
- Dr Chetan V

Electronic City:
- Dr Suresh Babu MC
- Dr Anup R Hegde
- Dr Swathi S Prakash

## Drug Categories
- **Chemotherapy**: Carboplatin, Cisplatin, Cyclophosphamide, Paclitaxel, Docetaxel, Doxorubicin, Epirubicin, Fluorouracil, Oxaliplatin, Irinotecan, Capecitabine, Nab-Paclitaxel, Pemetrexed, Ifosfamide, Thalidomide
- **Immunotherapy**: Pembrolizumab, Atezolizumab, Nivolumab, Durvalumab, Tislelizumab
- **Targeted Therapy**: Bevacizumab, Rituximab, Trastuzumab, Pertuzumab, Cetuximab, Ribociclib, Osimertinib, Carfilzomib, Inotuzumab, T-DM1, Polatuzumab, Bortezomib, Cabozantinib, Regorafenib
- **Hormonal Therapy**: Abiraterone, Triptorelin, Degarelix, Fulvestrant, Leuprolide, Letrozole, Progesterone
- **Supportive Care**: Antiemetics, GCSF/PEG-GCSF, IVIG, Mesna, zoledronic acid, iron preparations, palonosetron/fosaprepitant/aprepitant, romiplostim, plerixafor, rasburicase, and related supportive medicines

## Hospital Charges
- **Procedure Types**: Chemo Infusion Single/Multi Drug, Chemotherapy Minor/Medium/Major/Major Plus/Supra Major, Immunotherapy
- **Consumables**: Rs. 5,000 or Waive Off
- **Consultation**: Rs. 1,600 / Rs. 2,000 / Waive Off
- **Ward**: Day Care (Rs. 2,500) / Private (TBD)
- **Number of Days**: multiplies procedure, consumables, consultation, and ward charges

## Print Sheet
- Two print modes: `generatePrint('patient')` and `generatePrint('doctor')`
- **Patient print**: KIMS Onco Sciences logo, branch label, patient details, 3-line cost breakdown (Drugs + Supportive, Procedure Charges, Hospital Charges with consumables/ward/consultation), total, +/- 10% variance disclaimer, acknowledgement with signature fields
- **Doctor print**: detailed drug list table (brand, generic, qty, unit price, total) + hospital charges breakdown
- Print defaults to B&W mode; user can tick “Keep colours” when saving a colour PDF
- Designed to fit on a single A4 page

## Conventions
- Drug brand names: UPPERCASE (e.g., `"CAPEGARD 500MG TAB"`, `"CISTERO 50MG INJ"`)
- Do not include `ML` in brand display names when a strength in MG/MCG/GM is enough (e.g., use `"KEYTRUDA 100MG INJ"`, not `"KEYTRUDA 100MG/4ML INJ"`)
- Generic names in regimens must exactly match the `generic` field in the active branch database (e.g., `"BEVACIZUMAB"` not `"BEVACIZUMAB (AVASTIN)"`)
- Generic names: UPPERCASE, common aliases in brackets (e.g., "DOXORUBICIN (ADRIAMYCIN)", "FLUOROURACIL (5-FU)")
- MRP prices in Indian Rupees with Indian number formatting (e.g., Rs. 1,23,506.00)
- All amounts use `toLocaleString('en-IN')` for formatting
- Quantity default: 1 (validation requires qty >= 1 before adding)
- Mahadevapura prices were updated from `/Users/prathyusha/Desktop/Chemo drugs MRP.xlsx`; oral pack prices are stored per unit when the app calculates by tablet/capsule, with `unitsPerPack` used for pack display
