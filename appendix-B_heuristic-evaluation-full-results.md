# Appendix B — Heuristic Evaluation: Full Violation Tables and Summary

> **Paper:** Designing an AI-Assisted Paediatric ICU Timeline Dashboard — BritCHI 2026  
> **Interfaces audited:** `modular_app` (intervention) vs `baseline-ehr` (control — faithful traditional EHR replica)  
> **Evaluator:** Single expert (declared limitation; see §2.8.4 in main paper)  
> **Frameworks:** Nielsen's 10 Heuristics · TURF · ISO 9241-110 · HIMSS CDS Rights · FDA UX Guidance  
> **Correction passes:** 5 (see Appendix C for correction history)

---

## B.0 Framing

The `baseline-ehr` is a research-faithful replica of a traditional Electronic Health Record system, reproducing the domain-separated tab structure, raw-table data presentation, and absence of embedded clinical decision support that characterise deployed EHRs such as Epic and Cerner. Characteristics that are standard in the traditional EHR paradigm are documented in **Table B.1 (Study Rationale)** and treated as the independent variable under study — they are not scored as heuristic violations. Only behaviours that would constitute implementation gaps even by traditional EHR standards are recorded in the violation tables.

The `modular_app` is the intervention. Its violations are concentrated in AI transparency and human-in-the-loop design dimensions — addressable in future design iterations and carrying lower clinical risk than the baseline's structural gaps.

---

## Table B.1 — Traditional EHR Baseline Characteristics (Study Rationale)

| Characteristic | Traditional EHR behaviour (control) | modular\_app response | R-Map |
|---|---|---|---|
| Domain-separated tabs | Vitals, Labs, Medications, Procedures on separate tabs — standard in all major EHRs | Single-pane timeline aggregates all event types with colour-coded lanes | R1 / R3 |
| No cross-domain aggregation | Clinician must mentally integrate across tabs; no single deterioration view | Timeline + AI summary provide cross-domain synthesis | R1 / R3 / R4 |
| No trend visualisation | Raw timestamped tables only; no sparklines, no temporal encoding | Timeline encodes change over time; danger zones highlight severity trajectory | R2 |
| No severity encoding | All values displayed uniformly regardless of clinical significance | `ml/thresholds.py` rules layer drives `SEVERITY_COLORS` encoding | R2 |
| No AI synthesis | No automated narrative; clinician synthesises from raw data manually | Architected AI grounding pipeline with interactive citations | R4 / R5 / R6 |
| No role-adaptive views | Same record presentation for all clinical roles | Shared limitation with modular\_app in current prototype | R1 |
| External training model | No embedded help, tooltips, or onboarding; institutional training assumed | Tutorial external; citations are interactive (partial mitigation) | R1 / R6 |
| Session-only filter state | No persistent preferences across sessions | Shared characteristic; session-level state in both systems | R3 |
| Read-only viewer | No data export from the viewer interface | modular\_app provides CSV export on timeline view | R3 / R5 |
| Per-domain search | Each tab has its own search box; no unified cross-domain query | Timeline event-type toggles + date-window provide unified filtering | R3 |

---

## Table B.2 — Nielsen's 10 Heuristics: Violations

### B.2a — baseline-ehr (genuine implementation gaps)

| ID | Heuristic | Violation | Severity (0–4) | Clinical Risk | R-Map |
|---|---|---|---|---|---|
| N01 | H1 Visibility of system status | No loading indicator on tab switch or time filter application; major EHRs show loading states on data fetch | 2 | Low | R1 |
| N09 | H5 Error prevention | Date filter accepts From > To range without warning or validation | 3 | Med | R1 |
| N11 | H6 Recognition over recall | Age-banded paediatric reference ranges absent; clinician must recall normal ranges per age group — a documented PICU-specific EHR gap | **4** | **High** | R2 |
| N13 | H7 Flexibility & efficiency | Tab keyboard nav confirmed (Left/Right/Home/End); 24h/48h presets confirmed. Residual: no keyboard shortcuts within data tables; custom date range does not persist across tab switches | 1 | Low | R3 |
| N17 | H9 Help users recognise/recover errors | No in-interface error message on empty result sets after filter application | 1 | Low | R1 |

**Sub-total: 5 violations · Mean severity: 2.2 · High-risk: 1 (N11)**

### B.2b — modular\_app (intervention)

| ID | Heuristic | Violation | Severity (0–4) | Clinical Risk | R-Map |
|---|---|---|---|---|---|
| N04 | H2 Match real world | "Explore" label uses analytics vocabulary; clinicians expect terminology such as "Cohort" or "Ward overview" | 2 | Low | R1 |
| N06 | H3 User control & freedom | Regenerate available with spinner and failure message. Residual: no cancel mid-generation; user must wait for completion or failure | 1 | Low | R5 |
| N16 | H8 Aesthetic & minimalist design | Events-per-day density chart is intentional per DESIGN.md. Residual: potential cognitive overload when both timeline layers simultaneously active for unfamiliar users | 1 | Low | R3 |
| N20 | H10 Help & documentation | Individual citations are interactive; no panel-level explanation of AI grounding methodology or confidence weighting inside the summary panel | 1 | Low | R6 |

**Sub-total: 4 violations · Mean severity: 1.3 · High-risk: 0**

> **Notable modular\_app strengths (no violations):**  
> — N12: Age-banded severity encoding directly addresses N11 gap in baseline  
> — N14: 48h preset + "Return to default view" one-click reset + CSV export  
> — N21: Timeline click → structured detail modal — recognition over recall

---

## Table B.3 — TURF Framework: Violations

### B.3a — baseline-ehr

| ID | Dimension | Violation | Severity | Clinical Risk | R-Map |
|---|---|---|---|---|---|
| T03 | User-role differentiation | No role-adaptive views — a genuine gap even within traditional EHR norms; PICU nurse vs consultant information needs differ significantly | 2 | Med | R1 |

**Sub-total: 1 violation · Mean severity: 2.0**

### B.3b — modular\_app

| ID | Dimension | Violation | Severity | Clinical Risk | R-Map |
|---|---|---|---|---|---|
| T04 | User-role differentiation | No explicit role-switching; single view for all user types — shared limitation with baseline-ehr in current prototype | 2 | Med | R1 |
| T08 | Functional completeness | AI summary + timeline + CSV export present. Residual: no automatic threshold-based trigger; initiation is manual only (deliberate human-in-the-loop design) | 1 | Low | R4 / R5 |

**Sub-total: 2 violations · Mean severity: 1.5 · High-risk: 0**

---

## Table B.4 — ISO 9241-110 Interaction Principles: Violations

### B.4a — baseline-ehr

| ID | Principle | Violation | Severity | Clinical Risk | R-Map |
|---|---|---|---|---|---|
| I03 | Self-descriptiveness | Column headers lack consistent units across data types — a technical implementation gap even by EHR standards | 2 | Med | R2 |
| I07 | Error robustness | Invalid date ranges (From > To) silently accepted; empty result sets after filter application produce no feedback | 3 | Med | R1 |

**Sub-total: 2 violations · Mean severity: 2.5**

### B.4b — modular\_app

| ID | Principle | Violation | Severity | Clinical Risk | R-Map |
|---|---|---|---|---|---|
| I02 | Suitability for task | No explicit "current status" summary panel displayed on patient load — clinician must scroll to locate the AI summary | 1 | Low | R1 |
| I10 | Controllability | Good window control (zoom/pan/reset). Residual: AI summary scope always covers full visible window; no user-adjustable scope | 2 | Med | R5 |
| I12 | Individualisation | No persistent user preferences; session-level state only (intentional for evaluation) | 1 | Low | R4 |
| I14 | Learning support | Individual citations interactive. No panel-level explanation of AI methodology or confidence weighting | 1 | Low | R6 |

**Sub-total: 4 violations · Mean severity: 1.3 · High-risk: 0**

---

## Table B.5 — HIMSS CDS Rights: modular\_app only

> HIMSS CDS Rights apply only to the intervention. The baseline-ehr makes no clinical decision support claim.

| ID | Right | Violation | Severity | Clinical Risk | R-Map |
|---|---|---|---|---|---|
| H01 | Right information | Narrow window produces historically-weighted summary; weighting shift not communicated in UI | 2 | Med | R5 / R6 |
| H02 | Right person | No role verification; all users receive the same summary regardless of clinical role | 2 | Med | R4 |
| H03 | Right format | Free-text narrative with interactive citations; no structured clinical format (SBAR/SOAP) | 2 | Med | R5 |
| H04 | Right channel | Summary in same panel as all data; no push notification or alert escalation for critical AI findings | 2 | Med | R4 / R5 |
| H05 | Right time | Manually triggered; no automatic generation on patient load or threshold breach (deliberate human-in-the-loop design) | 2 | Med | R4 / R5 |

**Sub-total: 5 violations · Mean severity: 2.0 · High-risk: 0**

---

## Table B.6 — FDA UX Guidance: modular\_app only

> FDA UX applies only to the intervention as the system making AI-assisted clinical recommendations.

| ID | Guidance Area | Violation | Severity | Clinical Risk | R-Map |
|---|---|---|---|---|---|
| F02 | Primary vs secondary features | Timeline danger zones surface criticality at patient load. Residual: "Generate summary" trigger not immediately visible without scrolling to timeline panel | 2 | Med | R4 / R5 |

**Sub-total: 1 violation · Mean severity: 2.0 · High-risk: 0**

---

## Table B.7 — Quantitative Summary

| Framework | modular\_app violations | baseline-ehr violations | modular\_app mean sev. | baseline-ehr mean sev. | modular\_app High-risk | baseline-ehr High-risk |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| Nielsen's 10H | 4 | 5 | 1.3 | **2.2** | 0 | **1** |
| TURF | 2 | 1 | 1.5 | **2.0** | 0 | 0 |
| ISO 9241-110 | 4 | 2 | 1.3 | **2.5** | 0 | 0 |
| HIMSS CDS Rights | 5 | n/a | 2.0 | — | 0 | — |
| FDA UX Guidance | 1 | n/a | 2.0 | — | 0 | — |
| **Total** | **16** | **8** | **1.6** | **2.3** | **0** | **1** |

### Reading the counts correctly

The baseline-ehr shows fewer raw violations (8 vs 16). This is the intended outcome of correctly classifying traditional EHR paradigm characteristics as the study's independent variable rather than design failures. After reclassification:

- The baseline-ehr's **mean severity (2.3) exceeds the modular\_app (1.6)**.
- The baseline-ehr holds the **only Severity-4 High-risk item** (N11 — absent age-banded paediatric reference ranges), directly addressed by modular\_app's severity encoding.
- The modular\_app's 16 violations concentrate in **HIMSS/FDA AI-transparency dimensions** — addressable design items for the next iteration, not structural workflow failures.

---

## Table B.8 — Severity Distribution

| Severity | Label | modular\_app count | baseline-ehr count |
|:---:|---|:---:|:---:|
| 1 | Cosmetic — fix if time permits | 7 | 2 |
| 2 | Minor — low priority fix | 8 | 4 |
| 3 | Major — high priority fix | 0 | 2 |
| 4 | Catastrophic — must fix before release | 0 | **1** |
| **Total** | | **16** | **8** |

---

## Table B.9 — modular\_app Architectural Advantages (Source-Verified)

| Feature | Code evidence | Heuristic relevance | R-Map |
|---|---|---|---|
| Age-banded severity colour encoding | `ml/thresholds.py`, `clinical_analytics.py` | Directly addresses N11 (High-risk baseline gap); TURF T06 | R2 |
| Click-to-detail structured modals on all timeline events | `timeline_modal_helpers.py` | Recognition over recall (N21); detail-on-demand (ISO I04) | R2 / R3 |
| Architecturally explicit AI grounding pipeline | `ai_report_context.py` + `ai_evidence_ui.py` | HIMSS H01 (right information); FDA F01 | R4 / R6 |
| CSV export on timeline view | `callbacks/`, `DESIGN.md` | TURF T09; data portability for handover/audit | R3 / R5 |
| Production-grade session management | `session_gate.py` | ISO I06; not an evaluation artefact | R4 |
| Progressive disclosure as non-negotiable design principle | `DESIGN.md` | TURF T02; Nielsen N15/N16 | R3 |

---

## B.5 Narrative Summary

The heuristic evaluation compared the proposed modular PICU dashboard (`modular_app`) against a research-faithful replica of a traditional EHR (`baseline-ehr`) across five established frameworks. Because the baseline condition deliberately reproduces the paradigm characteristics of deployed EHRs, traditional-EHR behaviours — including domain-separated tabs, absence of embedded AI synthesis, and reliance on institutional training — were documented as study rationale rather than scored as heuristic failures (Table B.1). Violations were recorded only where behaviour constituted an implementation gap by the standards of the traditional EHR paradigm itself.

After correction and source-code verification (five passes), the baseline-ehr registered 8 violations at a mean severity of 2.3, including the sole Severity-4 Catastrophic item: the absence of age-banded paediatric reference ranges (N11), which requires clinicians to retrieve developmental normal values from memory during time-critical assessment. The modular\_app registered 16 violations at a mean severity of 1.6, with no High-risk items. The higher raw count in the intervention reflects the additional HIMSS CDS and FDA UX frameworks applied exclusively to the AI-bearing system, and the concentration of violations in AI-transparency and human-in-the-loop dimensions — all of which are addressable in subsequent design iterations.

The modular\_app demonstrated six source-verified architectural advantages directly relevant to the research questions (Table B.9). Notably, severity encoding provides the inline age-banded context absent in the baseline (addressing N11); structured click-to-detail modals support recognition over recall; and the explicit AI grounding pipeline partially addresses HIMSS H01 at the architectural level.

---

## B.6 Limitations

- **Single evaluator:** Expert heuristic evaluation was conducted by a single evaluator (the researcher). This is a declared limitation; expert agreement rates typically improve with multiple evaluators.
- **No empirical participants yet:** This evaluation is analytical. The planned crossover usability study (n=6; NASA-TLX; SUS; 12 PIC cases; Wilcoxon paired analysis) will provide empirical validation.
- **Baseline representation:** The baseline-ehr replicates typical EHR structure but cannot capture all variation in deployed systems. NHS site-specific EHR configurations may differ.
- **HIMSS/FDA applicability:** HIMSS CDS Rights and FDA UX Guidance were applied analytically, not through a formal regulatory submission process.
