# Appendix F — Age-Adaptive Vital Sign Threshold and Medication Classification Framework

> **Paper:** Designing an AI-Assisted Paediatric ICU Timeline Dashboard — BritCHI 2026  
> **Source module:** `modular_app/ml/thresholds.py`, `modular_app/references/vitals.json`, `modular_app/references/medecations.json`  
> **Design requirement:** R2 — Age-Aware Interpretation

---

## G.1 Overview

Unlike adult ICU systems, which can apply fixed reference thresholds, a PICU tool must classify each vital event against ranges that vary continuously with age. The threshold framework implements age-adaptive vital sign classification and medication risk stratification, driving the colour and marker shape encoding visible in the timeline.

---

## G.2 Severity Encoding Scheme

All severity levels are encoded consistently across the interface:

### Colour Encoding

| Severity | Colour | Hex | Use |
|---|---|---|---|
| Normal | Green | `#4CAF50` | Within age-appropriate reference range |
| Warning | Amber | `#FF9800` | Up to 20% deviation from boundary |
| Critical | Red | `#F44336` | More than 20% deviation from boundary |

### Marker Shape (Vital Signs)

| Severity | Plotly Symbol | Visual cue |
|---|---|---|
| Normal | `circle` | Closed circle |
| Warning | `circle-open` | Open circle |
| Critical | `circle-x` | Circle with X |

### Marker Size

| Severity | Size (px) | Border Width |
|---|---|---|
| Normal | 14 | 1 |
| Warning | 18 | 2 |
| Critical | 22 | 3 |

### Event Icons (Tooltip/On-Chart Labels)

| Event type | Icon |
|---|---|
| Vital warning / critical | ⚠ |
| Surgery | ⚕ |
| High-risk medication | ⛔ |
| Abnormal lab flag | ↑ |
| Group warning | ⚠ |

---

## G.3 Vital Sign Label Mapping

The following dataset LABEL strings (case-insensitive) are mapped to the reference lookup keys used in `vitals.json`:

| Dataset label variants | Vital key |
|---|---|
| heart rate, heartrate, hr, pulse | `heart_rate` |
| respiratory rate, resp rate, rr | `respiratory_rate` |
| temperature, temp | `temperature_c` |
| spo2, sp02, oxygen saturation | `spo2` |
| systolic bp, systolic blood pressure, sbp, nibp systolic, blood pressure systolic | `systolic_bp` |
| diastolic bp, diastolic blood pressure, dbp, nibp diastolic, blood pressure diastolic | `diastolic_bp` |

---

## G.4 Severity Classification Algorithm

For each vital event, the classification algorithm:

1. Normalises the label string (lowercase, strip).
2. Maps to a vital key via `LABEL_TO_VITAL_KEY`.
3. Resolves the patient's age group from `vitals.json` (paediatric age bands).
4. Retrieves the appropriate min/max range for that vital in that age group. For heart rate, the awake range is used as the conservative normal band.
5. Classifies:
   - **Normal:** `min ≤ value ≤ max`
   - **Warning:** deviation from nearest boundary ≤ 20% of the range span `(max - min)`
   - **Critical:** deviation > 20% of the range span
6. Returns `(severity, range_description_string)` where the range description is embedded in hover text.

---

## G.5 Medication Classification

Medications are classified into four dimensions:

| Dimension | Values | Example |
|---|---|---|
| `severity_risk` | low · moderate · high | Adrenaline → high |
| `clinical_role` | supportive · prophylactic · chronic\_control · organ\_support · life\_saving | Adrenaline → life\_saving |
| `main_category` | 14 categories (see G.5.1) | Adrenaline → Cardiovascular / Vasoactive |
| `functional_class` | Drug-specific functional class | Catecholamine |

Lookup is first attempted by exact name match (lowercase, stripped), then by substring match. Unknown drugs fall back to `{severity_risk: "low", clinical_role: "supportive", main_category: "Unknown"}`.

### G.5.1 Medication Category Encoding

| Category | Colour | Symbol |
|---|---|---|
| Cardiovascular / Vasoactive | `#E53935` (red) | cross |
| CNS / Sedation / Analgesia | `#8E24AA` (purple) | diamond-tall |
| Anti-infective / Antibiotics | `#F9A825` (amber) | hexagram |
| Respiratory / Airway | `#1E88E5` (blue) | diamond-wide |
| Fluids and Electrolytes | `#43A047` (green) | triangle-up |
| Hematology / Coagulation | `#D81B60` (pink) | bowtie |
| Endocrine / Metabolic | `#FB8C00` (orange) | hourglass |
| Renal / Diuretic | `#00ACC1` (cyan) | triangle-down |
| Nutrition and Vitamins | `#7CB342` (light green) | circle |
| Gastrointestinal / Hepatobiliary | `#6D4C41` (brown) | square |
| Immunomodulating / Biological | `#3949AB` (indigo) | star |
| Neuromuscular Blocker | `#C62828` (dark red) | cross |
| Neurology / Organ Support | `#5E35B1` (deep purple) | diamond |
| Topical / Antiseptic | `#78909C` (grey) | circle |

Medication severity risk maps to a separate colour scale:

| Severity risk | Colour |
|---|---|
| low | `#4CAF50` (green) |
| moderate | `#FF9800` (amber) |
| high | `#F44336` (red) |

---

## G.6 Design Rationale (R2)

This threshold framework implements R2 (Age-Aware Interpretation) by eliminating the manual reference-range lookup step that constitutes a significant cognitive load burden in PICU settings. By embedding age-appropriate classification directly into the visualisation layer, the system allows clinicians to perceive severity at a glance rather than reconstructing it from recalled reference ranges. This design decision is absent from all adult ICU dashboards in the Strechen et al. (2024) systematic review and is the primary structural differentiator between the present system and generic ICU dashboard approaches.
