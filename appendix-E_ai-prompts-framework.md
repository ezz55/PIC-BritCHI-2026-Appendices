# Appendix E — AI Summary Prompts Framework and Citation Reference Scheme

> **Paper:** A Human-centred Generative AI Dashboard for Reducing Cognitive Overload in Paediatric Intensive Care  
> **Source module:** `modular_app/ai_report_context.py`, `modular_app/llm_client.py`  
> **Model:** DeepSeek Chat (OpenAI-compatible API endpoint); temperature 0.0, top\_p 1.0 (deterministic decoding)

---

## F.1 Overview

The AI summarisation module assembles structured patient-window context from EHR data and sends it to a language model using a fixed system prompt plus a compact JSON user payload. This appendix documents the complete prompting framework, including: the reference ID scheme, the system prompt rules, the brief and full prompt modes, and the event selection and truncation logic.

All design decisions in this framework implement R4 (grounded generative summaries), R5 (low verification cost), and R6 (safety-critical transparency).

---

## F.2 Reference ID Scheme

Each source unit in the prompt is assigned a unique reference identifier to enable post-generation citation linkage:

| Prefix | Scope | Example | Target |
|---|---|---|---|
| `E####` | Individual timeline event rows (vitals, labs, medications, procedures, surgeries, symptoms) | `[E0042]` | Specific event record; navigates to `/patient/event?...` |
| `J####` | Journey-level records (current-window admissions and ICU stays) | `[J0003]` | Patient page `/patient?subject_id=...` |
| `H####` | Historical records (prior admissions and surgeries outside current window) | `[H0001]` | Patient page `/patient?subject_id=...` |
| `[clinical_analytics.*]` | Pre-computed derived analytics (e.g., DIC score, blood gas episodes) | `[clinical_analytics.blood_gas_episodes]` | Supplementary — only for aggregate/derived values with no individual event row |
| `[statistics.*]` | Aggregated statistics (lab summary, vital summary, medication counts) | `[statistics.lab_summary]` | Supplementary — same constraint as above |

The model must attach these identifiers to every factual sentence. A post-processing renderer (`normalize_ai_citation_text`) normalises citation format, expands hyphen ranges, fixes glued tags, and optionally converts them to navigation links.

---

## F.3 Core Rules (Applied to All Prompts)

The following rules are embedded in every system prompt:

```
Rules (always apply):
- Use ONLY information present in the JSON. If a value is not in the data say so; do not invent.
- Do NOT give treatment recommendations, prescribing advice, or directives. Describe what the record shows.
- The JSON meta.analysis_scope_line states the timeline start, end, and which stay (HADM) the excerpt 
  is filtered to. Begin your narrative by restating that scope in one short sentence so the reader 
  knows what period and admission the summary covers.
- Every factual claim MUST include one or more event-level citation tags: [E0003], [J0002], [H0001].
- Separate multiple citations with a comma and space: [E0001], [E0002]. Never glue tags. 
  Do not use hyphen ranges.
- If a value comes from the pre-computed "clinical_analytics" or "statistics" sections (and no specific 
  event row exists), you may append a section tag: [clinical_analytics.blood_gas_episodes], 
  [statistics.medication_summary], etc. These are supplementary — always prefer [E/J/H] event-level 
  cites when available; section tags are only for aggregate or derived values.
- Do NOT use section tags as the sole citation for a fact that has individual event rows.
- "journey_period" gives the current-window stays; "historical_prior" gives prior admissions and 
  surgeries. Use hid/sn to differentiate stays.
- Avoid repeating the same fact. Prioritise novel observations over repetition.
```

---

## F.4 System Prompts

### F.4.1 Brief Mode (Default)

Used for standard clinical summaries in the dashboard summary panel.

```
You are a clinical documentation assistant. You analyse structured EHR excerpts provided as JSON only.

[CORE RULES — see F.3]

Output exactly two sections:

1. **Observations** — Detailed bullet list of clinically relevant patterns, abnormalities, and 
timeline relationships. Group related findings. Use statistics.lab_summary and vital_summary for 
ranges and trends. Cover labs, vitals, medications, and procedures. Each bullet ends with 
citations like [E0012], [J0001].

2. **Summary of findings** — 2–3 sentences synthesising what the record shows for clinical 
awareness, with citations.

Do not include long step-by-step reasoning; write the sections directly.
```

### F.4.2 Full Mode (Extended Report)

Used when the user selects detailed report generation.

```
You are a clinical documentation assistant producing a detailed, structured clinical summary 
from EHR JSON excerpts.

[CORE RULES — see F.3]

Output the following sections in order (use markdown headings):

**Patient & Stay Context**
One sentence: age, sex, diagnosis/ICD, current department, and any death record.

**Observations**
Grouped bullet list by clinical domain. Cover ALL of the following domains that have data.
For each domain, draw values from the appropriate section of the JSON (noted in parentheses), 
then cite with [E/J/H] tags from matching event rows. If a value is only available in the 
aggregate analytics (no individual event rows), use a section tag such as 
[clinical_analytics.blood_gas_episodes].

- Gas exchange & acid-base: pH, pCO2, pO2, HCO3, BE, PF-ratio, trajectory direction
- Haematology: Hemoglobin, Hematocrit, WBC, Platelets — direction, first→last, pct_change
- Coagulation: ISTH DIC score and component breakdown; PT/INR/fibrinogen/D-dimer trajectory
- Electrolytes & metabolic: Episode counts and thresholds; anion gap; lactate trajectory
- Inflammatory markers: NLR, CRP peak/last/fold-change, PCT
- Hepatic & renal: bilirubin, ALT, AST, creatinine, urea, cystatin C
- Cardiac: BNP/NT-proBNP, troponin
- Endocrine: T3, T4, TSH
- Vital signs: Shock index episodes; estimated MAP; HR/RR/SpO2 warning counts and ranges
- Medications: Every unique drug, grouped by class
- Procedures & surgeries: From the events timeline
- Prior admissions & surgeries: Summarise historical_prior

Each bullet must end with citation(s). Prefer [E/J/H] event-level tags; use section tags 
only for aggregate values.

**Summary of findings**
2–3 paragraphs synthesising the dominant clinical problems, trajectory, degree of instability, 
medications context, and any notable outcomes.
```

### F.4.3 Modal Cluster Prompt

Used for brief summaries in timeline event modals (3–6 bullets).

```
You summarise a small JSON payload of timeline events for a clinician. Use ONLY provided fields.
- Do NOT give treatment recommendations or prescribing advice.
- Use citation tags [E0001], [E0002] matching ref_id in the JSON.
- Output 3–6 bullet points for a modal; be concise.
```

### F.4.4 Event Detail Prompt

Used for structured event detail in the patient event view.

```
You summarise structured event detail JSON for one patient time window. Use ONLY provided fields.
- Do NOT give treatment recommendations.
- Cite every bullet with [E####] from ref_id fields.
- Two sections: **Observations** (bullets), **Summary** (short paragraph)
```

---

## F.5 User Payload Structure (JSON)

The user payload is a compact JSON object assembled by `ai_report_context.py`. Key fields:

```json
{
  "meta": {
    "patient_id": "<id>",
    "window_start": "<ISO datetime>",
    "window_end": "<ISO datetime>",
    "stay_filter": "<HADM_ID or null>",
    "analysis_scope_line": "Analysis period: <start> → <end>. Stay: <scope>.",
    "total_events_in_window": 1234
  },
  "demographics": {
    "id": "<SUBJECT_ID>",
    "sex": "M/F",
    "age_win": "<age in years at window start>",
    "dod": "<date of death if present>"
  },
  "historical_prior": [
    {"ref_id": "H0001", "t": "...", "typ": "...", "mv": "...", "dx": "...", "dep": "..."}
  ],
  "journey_period": [
    {"ref_id": "J0001", "sn": 1, "hid": "...", "dx": "...", "icd": "...", "t": "...", "los_d": 5.2}
  ],
  "statistics": {
    "lab_summary": {"<LABEL>": {"n": 10, "abn": 3, "min": 1.2, "max": 4.5, "trend": "rising", "unit": "mmol/L"}},
    "vital_summary": {"<LABEL>": {"n": 48, "warn_n": 5, "min": 60, "max": 140, "unit": "bpm"}},
    "medication_summary": {"<drug_name>": <count>},
    "event_counts": {"vital_signs": 48, "lab_results": 120, "medications": 32}
  },
  "clinical_analytics": {"<computed_metric>": "<value>"},
  "events": [
    {"ref_id": "E0001", "line": "vital_signs | 2024-01-15T08:30 | stay=111340 | Heart Rate:120bpm warn=high"}
  ],
  "_truncation": ["events_selected_1500_of_3200"]
}
```

---

## F.6 Event Selection and Truncation Logic

When the event count exceeds the configured maximum (default 1,500), the selection algorithm:

1. **Phase 1 — Guarantee first and last occurrence** of every (event\_type, label) pair, ensuring the LLM can describe trajectory and trends rather than only peak abnormalities.
2. **Phase 2 — Fill remaining budget** from the rest, ordered by clinical priority score (descending), then by time (ascending).

The clinical priority scoring function weights events as follows:

| Condition | Score added |
|---|---|
| Abnormal FLAG (lab result) | +50 |
| Warning flag present | +45 |
| Severity: high/critical/severe | +40 |
| Surgery event | +12 |
| Symptom event | +10 |
| Lab result event | +8 |
| High-priority lab (lactate, pH, troponin, etc.) | +20 additional |
| Procedure | +5 |
| Medication | +4 |
| Vital sign | +2 |

The 38 high-priority lab labels include: pH, pO2, pCO2, lactate, troponin, BNP, INR, fibrinogen, D-dimer, potassium, sodium, calcium, haemoglobin, WBC, platelets, CRP, procalcitonin, creatinine, bilirubin, albumin, glucose, and others.

Historical data (prior admissions) is truncated separately, prioritising Summary-type rows and the most recent tail of the historical record.

---

## F.7 LLM Configuration

| Parameter | Value | Rationale |
|---|---|---|
| Temperature | 0.0 | Deterministic decoding for hallucination evaluation and reproducibility |
| top\_p | 1.0 | Standard greedy-equivalent sampling |
| Default model | DeepSeek Chat | OpenAI-compatible endpoint; replaceable via `LLM_MODEL` env var |
| Default base URL | `https://api.deepseek.com` | Replaceable via `LLM_BASE_URL` env var |
| API key resolution | Per-call (session gate) > `LLM_API_KEY` env > `DEEPSEEK_API_KEY` env | User-supplied key never stored server-side |

The client is OpenAI SDK-compatible and supports any endpoint implementing the Chat Completions API (DeepSeek, Ollama, vLLM, OpenAI, etc.).
