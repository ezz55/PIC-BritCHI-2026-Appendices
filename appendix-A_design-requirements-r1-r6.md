# Appendix A — Design Requirements R1–R6: Traceability and Rationale

> **Paper:** A Human-centred Generative AI Dashboard for Reducing Cognitive Overload in Paediatric Intensive Care  
> **References:** Hevner et al. (2004); Nielsen (1994); Endsley (1995); Sweller (2011); Davidson et al. (2022); Wac et al. (2023); Strechen et al. (2024); Rosenbacke et al. (2024)

---

## A.1 Overview

The six design requirements (R1–R6) are design hypotheses derived from convergent gaps in the background literature review. Each requirement maps a documented clinical need or evidence gap onto a specific design decision, instantiated as a testable system feature and evaluated empirically. The derivation follows Design Science Research (DSR) methodology (Hevner et al., 2004).

Requirements are ordered by scope:

- **R1–R3** concern the temporal visualisation layer — what data is shown and how it is organised.
- **R4–R5** concern the GenAI summary layer — what AI says and how it is attributed.
- **R6** concerns the safety and governance frame applied across both layers.

---

## A.2 Traceability Table

| Req. | Literature Gap | Design Decision | System Feature | Evaluation Outcome |
|---|---|---|---|---|
| **R1** | Fragmented multi-stay trajectories; cross-system navigation overhead | Unified per-patient journey object on a shared temporal axis | Multi-stay timeline (§2.4, §2.5) | NASA-TLX extraneous load; task completion time |
| **R2** | Age-banded reference lookup burden; paediatric-specific physiology | Inline age-aware classification; colour/marker severity encoding | Vital sign colour encoding; tooltip reference ranges (§2.5) | Clinician SA score; error rate in severity identification |
| **R3** | Overplotting at long time horizons; alarm fatigue | Scale-adaptive temporal aggregation; overview-zoom-details pattern | Progressive aggregation engine (§2.5) | NASA-TLX perceptual load; pattern detection accuracy |
| **R4** | No ICU dashboard combines trend visualisation with grounded GenAI narrative | Source-attributed prompting with unique reference IDs | Grounded summary module with inline citations (§2.6) | Hallucination rate; source attribution accuracy |
| **R5** | Verification overhead drives XAI non-use and misapplication | Single-click citation-to-source navigation | Citation modal; timeline drill-down from summary (§2.6) | Verification task time; trust calibration score |
| **R6** | Prototype-to-deployment gap; binary classification ceiling; lack of safety framing | Advisory-only framing enforced at prompt and UI level; offline research environment | System prompt constraints; UI advisory label; no management advice output (§2.6) | Clinician-perceived safety; appropriateness ratings |

---

## A.3 Requirement Interdependencies

The six requirements form a coherent design theory in which each layer depends on the one below it:

- **R1** provides the unified data substrate without which R2's age-aware classifications and R3's aggregation engine have nothing to operate on.
- **R3's** overplotting solution is a precondition for R4's summary being clinically useful: a summary grounded in overplotted, unresolved dense data would reflect noise rather than signal.
- **R5's** single-click verification is only meaningful if R4 has produced genuinely source-attributed output — verification of an unattributed summary is not possible regardless of interaction cost.
- **R6's** advisory framing is the governance envelope that makes the entire system deployable in a research clinical environment.

This layered dependency structure also defines the evaluation priority ordering: R1–R3 are evaluated first through task-based cognitive load and situation awareness measures; R4–R5 through source attribution accuracy and verification task timing; and R6 through clinician-rated appropriateness and safety perception items in the structured interview.

---

## A.4 Individual Requirement Rationale

### R1 — Integrated Patient Journey

Reconstructing a patient's illness trajectory across multiple admissions, ICU episodes, and data streams is a task PICU clinicians currently perform by moving between fragmented systems, accumulating cognitive overhead with every navigation step. The extraneous cognitive load imposed by this navigation is distinct from the intrinsic complexity of the clinical reasoning itself. To address this, the system constructs a per-patient journey object from the PIC database, aggregating all encounter records — admissions, vitals, labs, medications, procedures, and outcomes — onto a shared temporal axis encompassing multi-stay trajectories.

R1 extends dashboard requirements documented by Davidson et al. (2022) and Wac et al. (2023) — who identified access to longer patient history and horizontal scrolling as top clinical priorities — by addressing the multi-stay dimension unique to PICU. Paediatric patients with complex congenital or chronic conditions may accumulate 10–20 PICU admissions over several years; cross-admission pattern recognition is a core PICU clinical task that no existing ICU dashboard addresses (Strechen et al., 2024).

### R2 — Age-Aware Interpretation

Interpreting paediatric physiology requires constant cross-referencing against age-appropriate reference ranges, a burden that compounds significantly across a shift. Unlike adult ICU systems, which can apply fixed reference thresholds, a PICU tool must classify each vital event against ranges that vary continuously with age. The system eliminates this lookup step by classifying each vital event as normal, warning, or critical against curated age-banded reference ranges. Medications are mapped to risk-stratified clinical categories. Classifications drive colour and marker shape on the timeline; tooltips surface both the measured value and the applicable reference range.

This design decision is absent from all adult ICU dashboards in the Strechen et al. (2024) systematic review, none of which implemented age-adaptive thresholds.

### R3 — Progressive Aggregation

Long PICU stays generate event densities that can obscure the very patterns clinicians need to detect; alarm fatigue in ICU settings is in part a visualisation problem, not only a threshold-calibration problem. The system responds with scale-adaptive temporal aggregation: when the visible window is wide and event counts are high, events collapse into calendar-based buckets sized to the visible span, with click-through access to full detail. At finer resolutions, concurrent events within the same hour merge into group markers — medications excepted, to preserve precise timing. This implements Shneiderman's (1996) overview first, zoom and filter, details on demand pattern, preventing overplotting from masking salient clinical signals.

### R4 — Grounded Generative Summaries

A generative AI summary is only clinically useful if every factual statement it contains can be traced back to specific source data. Each source unit in the prompt is therefore assigned a unique reference identifier: `[E####]` for individual timeline event rows, `[J####]` for journey-level records, `[H####]` for historical context, and section-level labels such as `[clinical_analytics.*]` or `[statistics.*]` for aggregated computed metrics. The model must attach these identifiers to the sentences they support; a post-processing renderer parses and normalises them into interactive UI elements. Deterministic decoding (temperature 0, top\_p 1.0) is used throughout to support hallucination evaluation and to maximise reproducibility.

### R5 — Low Verification Cost

Traceability is only effective if following a citation is fast enough to fit within ward-round time pressure. Rosenbacke et al.'s (2024) XAI trust taxonomy identifies verification overhead as a primary mechanism of both XAI non-use and XAI misapplication. The citation framework from R4 provides the structural mechanism for R5: a single click on any inline citation in the summary opens a source record panel without leaving the summary view, keeping verification to a single interaction.

### R6 — Safety-Critical Transparency

The system presents AI output strictly as an inspectable summary and never as a treatment recommendation or management directive. This constraint is enforced at two levels: the system prompt explicitly prohibits management advice, and the UI frames the output as a summary for inspection with a persistent advisory label. The generative module operates on de-identified data in an offline research environment, with no live clinical deployment pathway at this stage.
