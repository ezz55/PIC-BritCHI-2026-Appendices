# Appendix H — PIC Dataset Reference: Table Schemas and Data Flow

> **Paper:** A Human-centred Generative AI Dashboard for Reducing Cognitive Overload in Paediatric Intensive Care  
> **Dataset:** Paediatric Intensive Care (PIC) v1.1.0  
> **Reference:** Zeng et al. (2020) — original PIC dataset publication

---

## I.1 Dataset Overview

| Field | Detail |
|---|---|
| Name / version | PIC (Paediatric ICU database), v1.1.0 |
| Content | De-identified paediatric ICU-style hospital EHR data (MIMIC-like structure) from a Chinese children's hospital |
| Modalities | Admissions, ICU stays, vital signs, laboratory results, medications, microbiology, imaging reports, surgery, and reference dictionaries |
| Storage format | CSV files; 17 named tables loaded at runtime |

---

## I.2 Full-Cohort Table Scale

| Table | Rows | Columns |
|---|---|---|
| MERGED\_ADMISSIONS\_PATIENTS\_WITH\_AGE | 13,449 | 28 |
| CHARTEVENTS | 2,278,978 | 10 |
| D\_ICD\_DIAGNOSES | 25,379 | 5 |
| D\_ITEMS | 479 | 7 |
| D\_LABITEMS | 832 | 7 |
| DIAGNOSES\_ICD | 22,712 | 6 |
| EMR\_SYMPTOMS | 402,142 | 8 |
| ICUSTAYS | 13,941 | 11 |
| INPUTEVENTS | 26,884 | 8 |
| LABEVENTS | 10,094,117 | 9 |
| MICROBIOLOGYEVENTS | 183,869 | 14 |
| OR\_EXAM\_REPORTS | 183,809 | 8 |
| OUTPUTEVENTS | 39,891 | 9 |
| PATIENTS | 12,881 | 6 |
| PRESCRIPTIONS | 1,256,591 | 13 |
| SURGERY\_INFO | 7,488 | 12 |
| SURGERY\_VITAL\_SIGNS | 1,944,187 | 9 |

---

## I.3 Core Identifiers and Relationships

### Primary Entities

| Identifier | Meaning | Typical cardinality |
|---|---|---|
| **SUBJECT\_ID** | Stable patient identifier | One row per patient in `PATIENTS`; repeated in all clinical tables |
| **HADM\_ID** | Hospital admission | One admission; spine for merging most fact tables |
| **ICUSTAY\_ID** | ICU stay | One ICU episode; links `ICUSTAYS` to ICU-scoped events |
| **ROW\_ID** | Table-local row surrogate | **Not** a global key; unique only within each CSV export |

### Dictionary / Code Keys

| Key | Used in | Joins to |
|---|---|---|
| **ICD10\_CODE\_CN** | MERGED\_ADMISSIONS\*, DIAGNOSES\_ICD | D\_ICD\_DIAGNOSES |
| **ITEMID** (numeric) | CHARTEVENTS, INPUTEVENTS, OUTPUTEVENTS | D\_ITEMS (`LINKSTO` indicates target table) |
| **ITEMID** (numeric) | LABEVENTS | D\_LABITEMS |

### Surgery Composite Key

`SURGERY_INFO` and `SURGERY_VITAL_SIGNS` align on: `SUBJECT_ID`, `HADM_ID`, `VISIT_ID`, `OPER_ID`

Note: `SURGERY_VITAL_SIGNS.ITEMID` uses string monitor codes (e.g. `SV1`), not the numeric `D_ITEMS` chart ITEMID namespace.

---

## I.4 Entity-Relationship Diagram

```
PATIENTS ──< MERGED_ADMISSIONS_PATIENTS_WITH_AGE
PATIENTS ──< ICUSTAYS
MERGED_ADMISSIONS_PATIENTS_WITH_AGE ──< ICUSTAYS          (HADM_ID)
ICUSTAYS ──< CHARTEVENTS                                   (ICUSTAY_ID)
MERGED_ADMISSIONS_PATIENTS_WITH_AGE ──< CHARTEVENTS        (HADM_ID)
D_ITEMS ──< CHARTEVENTS                                    (ITEMID)
D_LABITEMS ──< LABEVENTS                                   (ITEMID)
D_ICD_DIAGNOSES ──< DIAGNOSES_ICD                          (ICD10_CODE_CN)
MERGED_ADMISSIONS_PATIENTS_WITH_AGE ──< DIAGNOSES_ICD      (HADM_ID)
SURGERY_INFO ──< SURGERY_VITAL_SIGNS                       (VISIT_ID, OPER_ID)
```

---

## I.5 Key Table Schemas

### PATIENTS

| Column | Type | Role |
|---|---|---|
| SUBJECT\_ID | int | Patient key |
| GENDER | str | M/F |
| DOB | datetime | Date of birth |
| DOD | datetime (nullable) | Date of death |
| EXPIRE\_FLAG | int | 0/1 mortality flag |

### MERGED\_ADMISSIONS\_PATIENTS\_WITH\_AGE

Primary key: `HADM_ID`. Built by preprocessing pipeline from `ADMISSIONS.csv` + `PATIENTS.csv` + `D_ICD_DIAGNOSES.csv`.

Key columns: `HADM_ID`, `SUBJECT_ID`, `ADMITTIME`, `DISCHTIME`, `DEATHTIME`, `ICD10_CODE_CN`, `HOSPITAL_EXPIRE_FLAG`, `AGE_AT_ADMISSION`, `CATEGORY` (ICD chapter block).

### ICUSTAYS

Primary key: `ICUSTAY_ID`. Key columns: `SUBJECT_ID`, `HADM_ID`, `ICUSTAY_ID`, `FIRST_CAREUNIT`, `LAST_CAREUNIT`, `INTIME`, `OUTTIME`, `LOS` (days).

### CHARTEVENTS

Bedside charted events (vitals, scores, device readings). Key columns: `SUBJECT_ID`, `HADM_ID`, `ICUSTAY_ID`, `ITEMID`, `CHARTTIME`, `VALUE`, `VALUENUM`, `VALUEUOM`.

### LABEVENTS

Key columns: `SUBJECT_ID`, `HADM_ID`, `ITEMID`, `CHARTTIME`, `VALUE`, `VALUENUM`, `VALUEUOM`, `FLAG` (abnormal flag).

### PRESCRIPTIONS

Key columns: `SUBJECT_ID`, `HADM_ID`, `ICUSTAY_ID`, `STARTDATE`, `ENDDATE`, `DRUG_NAME`, `DRUG_NAME_EN`, `DOSE_VAL_RX`, `DOSE_UNIT_RX`, `DRUG_FORM`.

### SURGERY\_INFO

Composite key: `SUBJECT_ID`, `HADM_ID`, `VISIT_ID`, `OPER_ID`. Key columns: `ANES_START_TIME`, `ANES_END_TIME`, `SURGERY_BEGIN_TIME`, `SURGERY_END_TIME`, `SURGERY_NAME`, `ANES_METHOD`.

---

## I.6 Preprocessing Pipeline

High-level steps in `data-preprocessing.py`:

1. Load `D_ICD_DIAGNOSES.csv` and `ADMISSIONS.csv`.
2. Map `ICD10_CODE` to a `CATEGORY` using ICD-10 chapter/block rules (`categorize_disease`).
3. Merge dictionary fields onto admissions via `ICD10_CODE_CN` (deduplicated dictionary).
4. Write `ADMISSIONS_WITH_CATEGORY.csv`.
5. Merge `PATIENTS` on `SUBJECT_ID`; compute `AGE_AT_ADMISSION` from `ADMITTIME` and `DOB` (365.25-day year).
6. Write `MERGED_ADMISSIONS_PATIENTS_WITH_AGE.csv`.

---

## I.7 Data Flow: CSV → Dashboard

```
Source CSVs
    ↓
data-preprocessing.py
    ↓
MERGED_ADMISSIONS_PATIENTS_WITH_AGE.csv
    + Clinical and surgery CSVs (unchanged)
    ↓
modular_app/data_loader.load_data()
    → dict[str, DataFrame] keyed by table basename
    ↓
Dash modular_app routes and callbacks
```

At runtime, `app.py` calls `load_data()` once at startup; layouts and callbacks receive the shared `dataframes` dict for cohort, labs, meds, patient journey, surgery, and event-detail routes.

---

## I.8 Evaluation Subset

The evaluation study used 12 de-identified cases from the PIC dataset (plus 1 practice case). The subset bundle, across the 13 available cases, contained:

| Data type | Records |
|---|---|
| Vital sign records | 1,817 |
| Laboratory results | 10,054 |
| Medication records | 853 |
| Procedures | 152 |
| Surgical records | 11 |
| Symptom entries | 687 |
| Diagnosis records | 13 |
| ICU stay records | 13 |

Unit types represented: SICU, CICU, PICU, and NICU.

---

## I.9 Data Quality Notes

- `PATIENTS.DOD` can be empty while an admission row carries `DEATHTIME` — a known inconsistency requiring explicit handling when building mortality timelines.
- `SURGERY_VITAL_SIGNS.ITEMID` uses a string monitor code namespace (e.g. `SV1`) distinct from the numeric `D_ITEMS` `ITEMID` namespace; these must not be joined directly.
- `ICUSTAY_ID` may be NaN in some `CHARTEVENTS` exports; admission-level join via `HADM_ID` serves as fallback.
