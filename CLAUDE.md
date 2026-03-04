# KIMS Medical Oncology - Cost Estimation App

A single-page web app for generating chemotherapy/immunotherapy cost estimation sheets for patients at KIMS Hospitals, Bengaluru.

## Tech Stack
- **Single file**: `index.html` (HTML + CSS + JS, no build step)
- **Hosting**: GitHub Pages (static site)
- **Print**: Browser print dialog → Save as PDF

## Project Structure
```
index.html          # Entire app — markup, styles, and logic
kims-macs-logo.jpg  # KIMS MACS logo (used in header + print sheet)
```

## Key Architecture
- All drugs stored in `drugDatabase` array with fields: `brand`, `generic`, `mrp`, `type`
- Drug types: `chemotherapy`, `immunotherapy`, `targeted`, `hormonal`, `supportive`
- Tab buttons filter the generic dropdown by `type`
- Hospital charges (consumables, consultation, ward, procedure) are selectable dropdowns
- Print sheet is generated dynamically in `generatePrint()` and injected into `#estSheet`
- Print sheet clubs consumables + consultation + ward + procedure as "Hospital Charges"
- App summary shows all line items individually

## Consultants
- Dr Suresh Babu MC
- Dr Prathyusha Eaga
- Dr Vivek B Maleyur
- Dr Chetan V

## Drug Categories
- **Chemotherapy**: Carboplatin, Cisplatin, Cyclophosphamide, Paclitaxel, Docetaxel, Doxorubicin, Epirubicin, Fluorouracil, Oxaliplatin, Irinotecan, Capecitabine, Nab-Paclitaxel, Thalidomide
- **Immunotherapy**: Pembrolizumab (Keytruda), Atezolizumab (Tecentriq)
- **Targeted Therapy**: Bevacizumab (Avastin/Bevatas), Rituximab (Mabtas), Trastuzumab (Vivitra), Ribociclib (Kryxana)
- **Hormonal Therapy**: Leuprolide (Leulide)
- **Supportive Care**: Antiemetics (Akynzeo, Nykron, Aprecap, Palnox, Fosaprepitant), GCSF/PEG-GCSF (Neukine, Peg Religrast), IVIG (Globucel, Immunorel), Mesna, Zoledronic Acid (Zoltero)

## Hospital Charges
- **Procedure Types**: Chemo Infusion Single/Multi Drug, Chemotherapy Minor/Medium/Major/Major Plus/Supra Major, Immunotherapy
- **Consumables**: Rs. 5,000 or Waive Off
- **Consultation**: Rs. 1,600 / Rs. 2,000 / Waive Off
- **Ward**: Day Care (Rs. 2,500) / Private (TBD)

## Print Sheet
- Shows KIMS MACS logo, patient details, 2-line cost breakdown (Drugs + Hospital Charges), total
- Notes include +/- 10% variance disclaimer
- Patient/Attendant acknowledgement with signature fields
- Designed to fit on a single A4 page

## Conventions
- Drug brand names: UPPERCASE, no "INJ" suffix (e.g., "FOIRI 100MG" not "INJ FOIRI 100MG INJ")
- Generic names: UPPERCASE, common aliases in brackets (e.g., "DOXORUBICIN (ADRIAMYCIN)", "FLUOROURACIL (5-FU)")
- MRP prices in Indian Rupees with Indian number formatting (e.g., Rs. 1,23,506.00)
- All amounts use `toLocaleString('en-IN')` for formatting
