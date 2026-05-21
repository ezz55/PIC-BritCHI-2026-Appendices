# Appendix E — Supplementary Evaluation Methods

> **Paper:** Designing an AI-Assisted Paediatric ICU Timeline Dashboard — BritCHI 2026  
> **Method type:** Expert-system (analytical) — no human participants required  
> **References:** Card et al. (1983); John & Kieras (1992); Wharton et al. (1994); Shneiderman (1996); Rosenbacke et al. (2024); Strechen et al. (2024)

---

## E.1 Rationale for Supplementary Methods

The heuristic evaluation (§2.8) and design artifact analysis (§2.9) constitute the two primary evaluation strands. Three additional expert-based methods are included to further strengthen the evaluation's rigour and to address dimensions of the system that the primary methods do not directly cover: interaction efficiency, task pathway coverage, and learnability. All three methods are analytical, require no human participants, and are well-established in the HCI and clinical informatics literature as complements to heuristic inspection (Card et al., 1983; Wharton et al., 1994).

---

## E.2 Log and Trace Analysis

The intervention dashboard (`modular_app`) generates structured interaction logs as a by-product of normal operation. These logs record, for each session, the sequence of user actions (page navigations, filter applications, AI summary generation requests, citation link activations, timeline zoom and pan interactions), together with timestamps. Log and trace analysis treats these records as a structured data source for expert-level analysis of interaction patterns (Shneiderman, 1996).

The analysis focuses on three questions:

1. **Citation activation rate:** Do users follow inline AI citations to their source records (R5)? A near-zero citation activation rate would constitute evidence that the single-click verification pathway is either insufficient or that users are not reading citations.
2. **Feature utilisation coverage:** Are all primary interface features (multi-stay view, timeline zoom, AI summary, category filter) accessed during representative sessions? Systematically avoided features may indicate discoverability or relevance problems.
3. **Error and fallback events:** Are there logged error states, empty-state renders, or fallback UI conditions that indicate data-handling edge cases requiring design attention?

Log analysis does not permit causal inference about user behaviour — it is observational and confounded by task design — but it provides direct, objective evidence about interaction patterns that neither heuristic inspection nor artifact analysis can access (Strechen et al., 2024).

---

## E.3 GOMS/KLM Analysis (Keystroke-Level Model)

The GOMS/KLM framework provides a method for estimating the time and cognitive steps required to complete a defined task using a given interface, based on a formal decomposition of user actions into elementary operators (Card et al., 1983; John & Kieras, 1992). Applied comparatively to the intervention and baseline interfaces, KLM analysis produces quantitative time estimates for representative clinical tasks, enabling comparison of interaction efficiency without user testing.

For the present evaluation, KLM analysis is applied to the following representative task: **locating the most recent critical vital sign abnormality for a patient with multiple ICU admissions and identifying the clinical context surrounding it**. This task was chosen because it directly instantiates R1 (multi-stay integration), R2 (age-aware severity encoding), and R3 (progressive aggregation) simultaneously.

Operators applied include:

- **K** — Keystroke/click
- **P** — Pointing
- **H** — Homing between devices
- **M** — Mental operation (recalling, deciding, or verifying)
- **R** — System response time

The baseline EHR viewer requires additional M operators at several points where the intervention dashboard pre-empts mental operations through visual encoding. For example, the clinician must recall the normal range for a given vital sign in the baseline but not in the intervention, because the encoding is embedded in the marker. Each additional M operator contributes approximately 1.35 seconds to task time in the standard KLM model (Card et al., 1983). The comparative task-time estimates derived from KLM analysis constitute quantitative evidence of the cognitive efficiency gain attributable to the R1–R3 design decisions.

---

## E.4 Cognitive Walkthrough

The Cognitive Walkthrough method simulates the experience of a new or infrequent user encountering an interface for the first time, by stepping through task sequences and evaluating, at each step, whether the action required is discoverable, learnable, and consistent with the user's prior knowledge and expectations (Wharton et al., 1994). For each action in the task sequence, four questions are posed:

1. Will the user try to achieve the right effect at this step?
2. Will the user notice that the correct action is available?
3. Will the user associate the correct action with the intended effect?
4. If the action is performed, will the user receive sufficient and correct feedback to understand progress?

A failure on any of these questions at any task step constitutes a *walkthrough breakdown*, recorded with the heuristic or design principle it violates. Cognitive Walkthrough is particularly appropriate for PICU CDS tools, where users may encounter the system under time pressure with limited training; identifying barriers to first-use and low-training-load use is therefore a safety-relevant design concern (Rosenbacke et al., 2024).

The walkthrough is conducted for both interfaces across two task scenarios:

1. The vital-sign abnormality localisation task used in the KLM analysis.
2. A secondary task (intervention only) requiring the user to generate an AI summary, read an inline citation, and navigate to the source event record. This scenario tests the learnability of the AI citation-and-verification pathway (R4, R5) for a first-time user with no prior exposure to the grounded summary feature.
