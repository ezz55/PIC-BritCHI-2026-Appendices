# Appendix C — Heuristic Evaluation: Detailed Violation Log

> **Paper:** A Human-centred Generative AI Dashboard for Reducing Cognitive Overload in Paediatric Intensive Care  
> **Status:** Complete — corrected against live interface behaviour + source code verification (correction pass 5)  
> **Interfaces audited:** `modular_app` (intervention) vs `baseline-ehr` (control — faithful traditional EHR replica)

**Framing note:** The baseline-ehr is a research-faithful replica of a traditional EHR system. Characteristics that are standard practice in deployed EHRs (Epic, Cerner, etc.) are documented in the **Study Rationale** section of Appendix B as the *study's independent variable* — they are not design flaws. Only behaviours that would be considered implementation gaps even by traditional EHR standards are logged as heuristic violations here.

---

## Correction History

| Pass | Changes |
|---|---|
| Pass 1 | API key entry is evaluation-deployment artefact; regenerate + spinner + failure message confirmed; "Return to default view" confirmed |
| Pass 2 | N13 24h/48h presets confirmed; I04 citations interactive; F03 visual demarcation confirmed; F02 danger zones confirmed |
| Pass 3 | Zero-event window not a use-error — model falls back to demographics + milestones |
| Pass 4 | Repo-verified updates — N05/I05 downgraded, T06/T07 strengthened, CSV export + modal drill-down added |
| Pass 5 | baseline-ehr reframed as traditional EHR replica — paradigm characteristics moved to Study Rationale section; genuine implementation violations retained |

---

## Nielsen's 10 Usability Heuristics

### baseline-ehr (genuine implementation gaps by EHR standards)

| # | Heuristic | Violation Description | Severity (0–4) | Clinical Risk | R-Map |
|---|---|---|---|---|---|
| N01 | H1 Visibility of system status | No loading indicator when switching tabs or applying time filters | 2 | Low | R1 |
| N09 | H5 Error prevention | Time filter accepts From > To date range without warning or validation | 3 | Med | R1 |
| N11 | H6 Recognition over recall | Age-banded paediatric reference ranges not displayed inline; clinician must recall normal ranges from memory | 4 | High | R2 |
| N13 | H7 Flexibility & efficiency | No keyboard shortcuts within data tables; custom date range does not persist across tab switches | 1 | Low | R3 |
| N17 | H9 Help users recognise/recover errors | No in-interface error messages on empty result sets after filter application | 1 | Low | R1 |

**Sub-total: 5 violations · Mean severity: 2.2 · High-risk: 1**

### modular\_app

| # | Heuristic | Violation Description | Severity (0–4) | Clinical Risk | R-Map |
|---|---|---|---|---|---|
| N04 | H2 Match real world | "Explore" label uses analytics vocabulary; clinicians expect "Cohort" or "Ward overview" | 2 | Low | R1 |
| N06 | H3 User control & freedom | No cancel mid-generation; user must wait for completion or failure | 1 | Low | R5 |
| N16 | H8 Aesthetic & minimalist design | Potential cognitive load when both timeline layers simultaneously active for unfamiliar users | 1 | Low | R3 |
| N20 | H10 Help & documentation | No panel-level explanation of AI methodology or grounding approach | 1 | Low | R6 |

**Sub-total: 4 violations · Mean severity: 1.3 · High-risk: 0**

---

## TURF Framework (Task, User, Representation, Function)

### baseline-ehr (genuine implementation gaps)

| # | Dimension | Violation Description | Severity (0–4) | Clinical Risk | R-Map |
|---|---|---|---|---|---|
| T03 | User-role differentiation | No role-adaptive views — a genuine gap even within traditional EHR norms | 2 | Med | R1 |

**Sub-total: 1 violation · Mean severity: 2.0**

### modular\_app

| # | Dimension | Violation Description | Severity (0–4) | Clinical Risk | R-Map |
|---|---|---|---|---|---|
| T04 | User-role differentiation | No explicit role-switching; single view for all user types | 2 | Med | R1 |
| T08 | Functional completeness | No automatic threshold-based summary trigger; manual initiation only (deliberate human-in-the-loop design) | 1 | Low | R4/R5 |

**Sub-total: 2 violations · Mean severity: 1.5 · High-risk: 0**

---

## ISO 9241-110 Interaction Principles

### baseline-ehr (genuine implementation gaps)

| # | Principle | Violation Description | Severity (0–4) | Clinical Risk | R-Map |
|---|---|---|---|---|---|
| I03 | Self-descriptiveness | Column headers lack consistent units across data types | 2 | Med | R2 |
| I07 | Error robustness | Invalid date ranges silently accepted; no guard on empty result sets | 3 | Med | R1 |

**Sub-total: 2 violations · Mean severity: 2.5**

### modular\_app

| # | Principle | Violation Description | Severity (0–4) | Clinical Risk | R-Map |
|---|---|---|---|---|---|
| I02 | Suitability for task | No explicit "current status" summary panel on patient load | 1 | Low | R1 |
| I10 | Controllability | AI summary scope always covers full visible window with no user-adjustable scope | 2 | Med | R5 |
| I12 | Individualisation | No user preferences; session-level state only (by design for evaluation) | 1 | Low | R4 |
| I14 | Learning support | No panel-level explanation of AI methodology | 1 | Low | R6 |

**Sub-total: 4 violations · Mean severity: 1.3 · High-risk: 0**

---

## HIMSS Clinical Decision Support Rights (modular\_app only)

| # | Right | Violation Description | Severity (0–4) | Clinical Risk | R-Map |
|---|---|---|---|---|---|
| H01 | Right information | Narrow window may produce historically-weighted summary; weighting shift not communicated in UI | 2 | Med | R5/R6 |
| H02 | Right person | No role verification; all users receive the same summary | 2 | Med | R4 |
| H03 | Right format | Free-text narrative with interactive citations; no structured clinical format (SBAR/SOAP) | 2 | Med | R5 |
| H04 | Right channel | No push notification or alert escalation for critical AI findings | 2 | Med | R4/R5 |
| H05 | Right time | Manually triggered; no automatic generation on patient load or threshold breach | 2 | Med | R4/R5 |

**Sub-total: 5 violations · Mean severity: 2.0 · High-risk: 0**

---

## FDA UX Guidance for Medical Devices (modular\_app only)

| # | Guidance Area | Violation Description | Severity (0–4) | Clinical Risk | R-Map |
|---|---|---|---|---|---|
| F02 | Primary vs secondary features | "Generate summary" trigger not immediately visible without scrolling to the timeline panel | 2 | Med | R4/R5 |

**Sub-total: 1 violation · Mean severity: 2.0 · High-risk: 0**
