# Appendix H — Evaluation Study Design: Full Protocol

> **Paper:** Designing an AI-Assisted Paediatric ICU Timeline Dashboard — BritCHI 2026  
> **References:** Brooke (1996); Hart & Staveland (1988); Hart (2006); Lewis (2018); Richardson et al. (2017); Pollack et al. (2020); Zeng et al. (2020); Strechen et al. (2024); Endsley (1995); Kushniruk & Patel (2004)

---

## H.1 Study Design and Counterbalancing

A within-subject, counterbalanced crossover design was used to compare two interfaces for paediatric intensive care record review:

- **Control condition:** Table-based electronic health record viewer (`baseline-ehr`) — a research-faithful replica of a traditional EHR (domain-separated tabs, raw-table data presentation, no embedded CDS).
- **Intervention condition:** PIC dashboard (`modular_app`) — modular Dash analytics application with age-adaptive severity encoding, progressive timeline aggregation, and source-attributed AI summarisation.

This design was chosen to enable paired comparison of objective performance and subjective ratings within a small clinical sample, reducing between-participant variability attributable to prior EHR experience and clinical background (Richardson et al., 2017; Pollack et al., 2020).

**Counterbalancing:** Participants were assigned to one of two fixed sequences before recruitment:

- **Group A:** Baseline EHR first → PIC Dashboard second
- **Group B:** PIC Dashboard first → Baseline EHR second

The crossover platform was built as a distinct orchestration and data-capture layer, entirely separate from either interface under evaluation. It governed participant authentication, administered protocol steps in a deterministic order, launched each interface externally by URL, and stored all structured outcomes in a dedicated study datastore.

---

## H.2 Technical Architecture and Deployment

Three co-deployed components:

1. **Crossover orchestration platform:** React/Vite single-page frontend; Cloudflare Worker backend (Hono framework); Cloudflare D1 SQLite-compatible database. Participant assignment schedules, block structure, and task schemas were bundled as static JSON assets at deployment time.

2. **Intervention interface (`modular_app`):** Python Dash application with modular page routes and reactive callbacks. Accessed via study links embedded in the orchestration platform.

3. **Control interface (`baseline-ehr`):** Static HTML/CSS/JavaScript viewer with client-side JSON loading and no server-side runtime dependency. No authentication required.

---

## H.3 Participants, Counterbalancing, and Case Allocation

- **N = 6 participants** (P01–P06), each assigned to one of the two counterbalanced sequences.
- Sample size was constrained by the availability of qualified PICU clinical staff; the evaluation was framed as a formative feasibility study rather than a definitive powered trial (Richardson et al., 2017).
- Each participant completed **two blocks** (one per interface condition), each comprising **two patient cases** with **five scored information-retrieval tasks per case** → 10 scored items per block → **20 scored items per participant overall**.
- **Scored patient cohort:** 12 de-identified cases drawn from the PIC dataset (Zeng et al., 2020). Case allocation was balanced such that each scored patient appeared under both interface conditions across the full participant set.
- **Practice case:** A 13th case used exclusively for tutorial and practice purposes; excluded from all scored analyses.

---

## H.4 End-to-End Protocol Flow (Fixed Sequence)

| Step | Description |
|---|---|
| 1 | Authentication via study ID and PIN |
| 2 | Acknowledgement of the Participant Information Sheet (PIS), delivered as embedded in-app PDF |
| 3 | Electronic consent (7 mandatory statements, typed identity and signature capture, timestamped server-side) |
| 4 | Demographics questionnaire (clinical role, years PICU experience, years EHR experience, primary EHR system, self-rated visual fatigue) |
| 5 | Condition-specific tutorial with guided practice on the practice patient |
| 6 | Pre-start timing and response instructions |
| 7 | Scored task block |
| 8 | Post-block subjective instruments (SUS + NASA-TLX) |
| — | Steps 5–8 repeated for the second condition |
| 9 | End-of-study debrief free-text capture |
| 10 | Session completion |

The protocol was resumable through server-side step tracking combined with local browser progress markers.

---

## H.5 Tutorial Design

### Control Tutorial (baseline-ehr) — 5 steps

1. Patient selection
2. Temporal filtering (presets and custom range)
3. Tab navigation (Vitals, Labs, Medications, Procedures, Overview)
4. Table search and sort
5. Date filter clearing

### Intervention Tutorial (modular_app) — 16 steps

1–3. Credential entry (dashboard password + researcher-supplied LLM API key)
4–6. Cohort-level Explore view
7–9. Timeline controls (pan, zoom, 48-hour window preset)
10–12. Event category filtering
13–14. Timeline click → modal detail view
15–16. Generate summary workflow and citation activation

Both tutorials used **patient 4835** as the practice case; this patient did not appear in any scored task block.

---

## H.6 Task Design and Response Capture

Tasks were structured as information-retrieval queries requiring participants to locate specific clinical data items from the assigned patient record. Each item was presented sequentially and supported four response widget types:

- Free text
- Numeric entry
- Datetime selection
- Yes/no binary choice

Widget type was pre-specified per task item in the bundled task schema.

For each scored item, the platform captured:

| Field | Description |
|---|---|
| Response content | Participant's answer |
| `rendered_at` | Timestamp when item was displayed |
| `submitted_at` | Timestamp when response was submitted |
| `hint_used` | Binary indicator |
| `skipped` | Explicit skip path if unable to locate |

Response time = interval between `rendered_at` and `submitted_at` (derived post hoc).

---

## H.7 Outcome Measures

### Objective Performance

Item-level correctness assessed through manual researcher adjudication after all sessions were complete. Secondary measures: response time, hint usage rate, skip rate, block-level durations.

### Usability

**System Usability Scale (SUS)** — 10-item questionnaire on a 5-point Likert scale administered immediately following each condition block (Brooke, 1996; Lewis, 2018). Composite score 0–100.

### Perceived Workload

**NASA Task Load Index (NASA-TLX)** — administered in its **ratings-only form (Raw TLX)** immediately after each block (Hart & Staveland, 1988; Hart, 2006). Six subscale dimensions rated 0–20:

1. Mental demand
2. Physical demand
3. Temporal demand
4. Performance (self-rated)
5. Effort
6. Frustration

The ratings-only implementation was adopted to reduce respondent burden. The pairwise weighted TLX procedure was **not** implemented and must not be reported as such (Hart, 2006).

### Debrief Themes

End-of-study debrief free-text responses were collected to elicit participant perspectives on: interface usability, encountered friction, and self-reported influence of the AI summary feature on task performance. Analysed thematically; responses relevant to R4–R6 (citation utility, advisory framing perception, verification behaviour) were coded separately.

---

## H.8 Declared Limitations

| Constraint | Declaration |
|---|---|
| **NASA-TLX implementation** | Raw TLX (ratings only); pairwise weighted procedure was not implemented |
| **AI credential harmonisation** | Participants entered both a dashboard password and a researcher-supplied LLM API key; key retained in browser tab for session duration only; not stored server-side |
| **Session duration** | Conflicting total-duration estimates across documents; harmonised as an amendment post-recruitment |
| **Baseline case completeness** | Pre-session QC explicitly verified that all 12 scored cases were accessible in both interface conditions |
| **Partially integrated event-detail route** | An event detail route exists in application code but was not active in the routing configuration used for participant sessions |

---

## H.9 Pre-Analysis Quality Control Checks

Before any analytical reporting:

1. Verify counterbalancing integrity against the preloaded schedule.
2. Confirm that each participant has 20 scored response records.
3. Quantify skip and hint rates by condition and confirm their treatment in correctness scoring.
4. Validate temporal monotonicity (`rendered_at ≤ submitted_at`) for all item records.
5. Lock operational definition of workload as raw, unweighted NASA-TLX ratings.
6. Record any sessions in which AI functionality was unavailable, with explicit handling declared in analytical reporting.
