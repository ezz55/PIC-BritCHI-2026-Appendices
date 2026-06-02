# Appendix D — Comparative Design Artifact Analysis

> **Paper:** A Human-centred Generative AI Dashboard for Reducing Cognitive Overload in Paediatric Intensive Care  
> **References:** Engeström (1987); Hutchins (1995); Endsley (1995); Nardi (1996); Zhang & Norman (1997); Wright et al. (2000); Davidson et al. (2022); Wac et al. (2023); Rosenbacke et al. (2024)

---

## D.1 Overview and Rationale

Design artifact analysis treats an interface as a cultural and cognitive artefact whose properties can be analysed against established theoretical frameworks, independently of empirical user data (Nardi, 1996). This approach is well-established in HCI and CSCW research as a means of generating theoretically grounded qualitative comparison between systems, producing insights into the structural cognitive and collaborative properties of designs rather than the measured performance of specific users under specific conditions (Zhang & Norman, 1997; Wright et al., 2000).

Both interfaces — the intervention dashboard and the baseline EHR viewer — are treated as design artefacts and subjected to structured analysis through three complementary theoretical lenses: Activity Theory, Distributed Cognition, and Endsley's (1995) three-level Situation Awareness model. Each lens foregrounds a different dimension of the human-interface relationship.

---

## D.2 Lens 1 — Activity Theory (Engeström, 1987)

Activity Theory models goal-directed human activity as a triadic structure: a **subject** (the clinician) acts on an **object** (the patient's clinical state) using **mediating artefacts** (tools, signs, instruments). The dashboard is analysed as a mediating artefact in the activity system of PICU bedside assessment.

The Activity Theory analysis proceeds across five structural components of the activity system for each interface:

1. **Subject:** PICU clinician (nurse, physician, or advanced practice provider) engaged in patient assessment during a shift or ward round.
2. **Object:** The patient's current clinical state, trajectory across the admission, and any pending care decisions.
3. **Mediating artefacts:** The interface itself — its navigation structure, representational conventions, and information architecture.
4. **Rules and community:** Implicit clinical workflow norms (e.g., rapid bedside assessment, handover conventions, alarm fatigue tolerances) that shape how the artefact is used in practice.
5. **Division of labour:** Role differentiation between nurses, physicians, and respiratory therapists, and whether the interface supports or conflicts with role-specific information needs (Davidson et al., 2022; Wac et al., 2023).

### baseline-ehr — Low-Mediation Artefact

The Activity Theory analysis characterises the baseline-ehr as a *low-mediation artefact*: data is surfaced as raw tabular records across five navigation tabs, requiring the clinician to perform all temporal and severity integration as internal cognitive operations. The artefact places no interpretive structure between the raw data and the clinical reasoning task, meaning the bulk of the mediating work remains with the subject rather than the tool. Cross-admission trajectory construction — a core PICU clinical task — is entirely absent from the artefact's structure.

### modular\_app — High-Mediation Artefact

The modular\_app is characterised as a *high-mediation artefact*: the interface pre-processes, classifies, and organises EHR data into a structure that explicitly mirrors the object of clinical interest (the patient trajectory across admissions), embedding interpretive operations — age-banded severity classification, progressive temporal aggregation, source-attributed AI narrative — directly into the mediating layer. This shifts cognitive work from the subject's internal processing to the artefact's representational structure, which is the intended effect of the R1–R3 design decisions.

---

## D.3 Lens 2 — Distributed Cognition (Hutchins, 1995)

Distributed Cognition frames cognition as a process distributed across people, artefacts, and representational media rather than confined within an individual's mind (Hutchins, 1995). Applied to clinical interface analysis, this lens examines how each interface distributes cognitive load between the clinician and the system, and whether the distribution is appropriate for the time pressure and working memory constraints of PICU care (Zhang & Norman, 1997).

The analysis examines each interface across three dimensions:

### Representational State Propagation

How clinical information is transformed across representational formats as it moves from EHR source records through the interface to the clinician's working model.

- **baseline-ehr:** This transformation is minimal: tabular data is displayed as tabular data, and the clinician must re-encode it into temporal, severity-categorised, and narrative formats internally.
- **modular\_app:** Representational state is progressively transformed by the system — raw events become timeline markers, markers acquire age-banded severity classifications, temporal density is managed by the aggregation engine, and the AI module generates a narrative synthesis with source attribution. Each step reduces the internal re-encoding demand on the clinician.

### Cognitive Off-Loading

The degree to which each interface allows the clinician to off-load working-memory operations onto the interface.

- Age-banded severity encoding (R2) off-loads reference-range lookup.
- The progressive aggregation engine (R3) off-loads density management.
- Source-attributed AI summaries (R4, R5) off-load temporal narrative construction.
- The baseline viewer provides no analogous off-loading mechanisms.

### Coordination Structure

How each interface supports the distributed nature of PICU care:

- The modular\_app's cohort-level Explore view provides a situational-awareness overview relevant to charge nurses and attending physicians.
- The patient-level workspace provides the per-patient detail relevant to bedside nurses and residents.
- The baseline viewer provides no role-differentiated views (Davidson et al., 2022; Wac et al., 2023).

---

## D.4 Lens 3 — Situation Awareness (Endsley, 1995)

Endsley's (1995) three-level model — perception (Level 1), comprehension (Level 2), and projection (Level 3) — maps the structural elements of each interface onto the three SA levels.

| SA Level | Intervention Dashboard | Baseline EHR Viewer |
|---|---|---|
| **Level 1 — Perception** (detection and registration of relevant elements) | Multi-stay timeline with age-banded severity encoding; colour and marker shape immediately distinguish normal, warning, and critical events without recall; events-per-day density chart provides at-a-glance activity overview | Raw tabular records across five tabs; no visual encoding of severity; clinician must parse individual values and compare against recalled reference ranges to identify abnormal events |
| **Level 2 — Comprehension** (integrating perceived elements into a coherent picture) | Progressive aggregation manages density; pattern recognition supported by temporal clustering and category-filtered views; multi-admission view allows cross-stay trajectory comprehension within a single interface; AI summary synthesises pattern-level narrative with explicit source attribution | Temporal filtering provides basic window control; no aggregation or pattern synthesis; clinician must manually integrate records across tabs and reconstruct trajectory from raw data; no cross-stay integration |
| **Level 3 — Projection** (anticipating future states based on comprehended situation) | AI-generated clinical summary (R4) provides explicit projection narrative, grounded in source events and qualified with uncertainty framing; advisory label enforces the distinction between AI projection and clinical decision (R6) | No projection support; clinician must construct anticipated trajectory entirely from internal reasoning with no AI-mediated support |

This analysis demonstrates a structural asymmetry between the two interfaces across all three SA levels: the intervention dashboard provides substantive support at each level, while the baseline viewer provides support primarily at Level 1 and only in the attenuated form of raw data availability rather than curated signal presentation. This asymmetry is the direct consequence of the R1–R6 design decisions.

---

## D.5 Synthesis

The three-lens analysis converges on a consistent structural conclusion: the intervention dashboard is architecturally better equipped to support PICU clinical cognition across mediation, cognitive off-loading, and situational awareness dimensions. This convergent qualitative evidence complements the heuristic evaluation (Appendix B) and provides theoretically grounded analysis of how the two interfaces structurally differ in their capacity to support clinical work — without requiring participant access or ethics approval for user testing.
