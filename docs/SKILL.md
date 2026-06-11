###
name: bce-core
version: v2.3
description: >
  BCE core skill — Brownfield Context Extraction methodology and all Phase 8 BCE
  close-out procedures. Load for BCE extraction sessions, Phase 8 Part 2A (greenfield
  BCE adapter pipeline), Phase 8 Part 2B (enhancement BCE gap detection and impact log),
  or any work involving discovery/ artifacts. Contains: extraction paths, intake step,
  Stage 1/2/3 execution prompts, BCE gap detection prompt, ENH-NNN_BCE_IMPACT.md
  template, storage and maintenance rules. Also trigger when working with TOPOLOGY.md,
  MODULE_CONTRACTS.md, INTEGRATION_CONTRACTS.md, INVARIANT_CATALOGUE.md,
  RISK_REGISTER.md, INTAKE_SUMMARY.md, ANNOTATION_CHECKLIST.md, DOMAIN_MODEL.json,
  UI_SURFACE.md (dual-registered discovery artifact for UI projects),
  or any discovery/ file.
  PBVI-011: Module classification extended with five UI layer types (route, page,
  layout, component, store). Stage 2 reconciliation gains UI_SURFACE.md cross-reference
  when present: every screen in UI_SURFACE.md must map to at least one page module;
  missing page modules become P1 annotation items. UI_SURFACE.md is dual-registered —
  produced as a Planning Artifact in PBVI Phase 1 Decide, referenced as a Discovery
  Artifact by BCE Stage 2 and Stage 3.
OWNER: DataGrokr LLC
COPYRIGHT: "© 2025 DataGrokr LLC. All rights reserved."
LICENSE: >-
  Proprietary — for use by licensed DataGrokr clients only.
  Unauthorized reproduction, modification, or transfer is prohibited.
CONTACT: naveen@datagrokr.com
---

# BCE Core Skill — v2.3

## Changelog

| Version | Date | Change Summary |
|---|---|---|
| v2.3 | May 2026 | PBVI-011 (UI as a First-Class Citizen) — Stage 2 module classification extended with five UI layer types: `route`, `page`, `layout`, `component`, `store`. These apply when the project's APPLICATION_SURFACE contains UI (UI+API or UI_ONLY). Stage 2 reconciliation extended: when UI_SURFACE.md is present, every screen in UI_SURFACE.md must map to at least one page module; missing page modules become P1 annotation items. UI_SURFACE.md is dual-registered as both Planning Artifact (PBVI Phase 1) and Discovery Artifact (BCE Stage 2/3 reference) — see PROJECT_MANIFEST.md cross-reference rows. BCE-006 (brownfield UI layer extraction — producing UI_SURFACE.md from an existing codebase) is declared as a follow-on item and is out of scope for PBVI-011. Non-breaking — additive only. |
| v2.2 | April 2026 | BCE-006 — Session F Gate at Intake + Graph Construction Stage Step. Three targeted additions to make existing requirements explicit at the point of action. (1) Session F pre-assessment block added to Section 5 (intake): three-step trigger table with outcomes (FULL EXTRACTION / CATALOG STUB / SPARSE STUB) and disposition→output mapping; must be completed before INTAKE_SUMMARY.md is committed. (2) `Session F disposition` field added to INTAKE_SUMMARY.md template: must be set before F01 begins; valid values FULL EXTRACTION / CATALOG STUB / SPARSE STUB. (3) Graph construction elevated from trailing prose to a named mandatory Stage 3 close-out step: explicit trigger phrase, mandatory flag, cross-graph edge note for non-sparse DOMAIN_MODEL.json. Non-breaking — additive only; no artifact schema changes. |
| v2.1 | April 2026 | BCE-005 v1.1 — Domain Model Artifact. New Session F (F01/F02/F03 sub-sessions) producing discovery/DOMAIN_MODEL.json — a graph-structured entity model with structural and semantic layers. Session F trigger is a three-step pre-condition assessment: (1) is a maintained metadata catalog already present? — if yes, stub with catalog reference; (2) does a data layer exist structurally (ORM, migrations, DDL)?; (3) engineer declares canonical layer boundary, recorded in INTAKE_SUMMARY.md. Entity nodes gain four engineer-annotated semantic properties (business_definition, lifecycle_summary, synonyms, domain). New edge type SAME_AS (Entity → Entity) for cross-system alignment. Metric node (MT-NNN) added as conditional section — present only when a defined metrics layer exists (dbt, LookML, semantic layer tool), never stubbed. New DOMAIN_MODEL.json header flags: metricsLayer, catalogBacked. New Templates T6/T7. Section renumbering 11.3–11.7 → 11.4–11.8. SYSTEM_GRAPH.json gains four cross-graph edge types. Stage 3 Check 0 and Check 6 extended. ENH-NNN_BCE_IMPACT.md template gains Section 7. New coordination artifact: discovery/ID_REGISTRY.md. GrokLayer extension layer documented. Supersedes GROK-002. |
| v2.0.1 | April 2026 | Bug fix — Stage 1 prompt now detects path before reading any files. If docs/ARCHITECTURE.md is absent, stops immediately and directs engineer to Session A0. Prevents confusing file-not-found failures when "Run BCE Stage 1" is triggered on Path A or Path B projects. No change to Stage 1 output for valid Path C runs. |
| v2.0 | April 2026 | BCE-004 — Artifact Graph Construction. M-NNN ID schema for modules (assigned in Session A via Module Roster in A02); IP-NNN ID schema for integration points (assigned in Session A A03 reconciliation). Sessions A, B/C, D, E updated to emit and reference IDs. Stage 3 adds schema validation gate (Check 0) — all relationship fields must use IDs before Stage 3 proceeds. New Section 11.7: graph construction prompt producing discovery/SYSTEM_GRAPH.json. New Section 16: subgraph context query for BCE-aware PBVI Phase 1 planning. SCHEMA_VIOLATION added as a checklist item Type. Breaking change: existing BCE artifacts must be migrated (assign IDs) before Stage 3 can run under v2.0 or later. |
| v1.9 | April 2026 | BCE-CL-019 — Per-Source Extraction Model (I01–I05) removed from Section 5. Non-code source extraction on Path A and Path B is now owned by the BCE-S pipeline. Section 5 "Per-Source Extraction Model" replaced with cross-reference to bce_signal.md. |
| v1.8 | April 2026 | BCE-002 — Human Annotation Record (HAR) formally specified as a named artifact. New subsection in Section 8: definition, ID convention (HAR-NNN), directory rules (discovery/annotations/ for greenfield, enhancement package for enhancement-level), canonical template. Section 14.3 Annotation Amendment prompt extended with STEP 5 — produce HAR per item resolved; commit step renumbered to STEP 6 and updated to include HAR paths. Section 9 Resolution Log: RESOLVED-ANNOTATION evidence field updated to reference HAR-NNN; resolution type vocabulary clarified. |
| v1.7 | April 2026 | PBVI-006 — All Stage 1 references to PHASE4_RISK_DECISIONS.md updated to docs/PHASE4_GATE_RECORD.md: artifact assignment summary, Stage 1 read list, INTAKE_SUMMARY Known Pain Points source, ARTIFACT 5 RISK_REGISTER.md instructions (five references). |
| v1.0 | March 2026 | Initial release |
| v1.1 | April 2026 | BCE-CL-001: ANNOTATION_CHECKLIST.md structure redesigned — severity/type/source model replaces category-based model. BCE-CL-002: Stage 1, Stage 2, Stage 3 execution prompts embedded directly in skill. BCE-CL-003: Stage 1 tool corrected to CC. BCE-CL-004: Stage 1 and Stage 2 artifact assignments enumerated explicitly — INTEGRATION_CONTRACTS.md assigned to Stage 2. BCE-CL-005: Template reference section updated. |
| v1.3 | April 2026 | BCE-CL-012: "When to Update Artifacts" table updated — enhancement close-out row split into sprint mode and single-enhancement sprint rows reflecting PBVI sprint extension mandate; standalone enhancement close-out path removed. BCE-CL-013: Section 13 general maintenance note updated — artifacts updated once per sprint at close-out in sprint mode, not after each individual enhancement; CD project file update happens at sprint close-out after the single sprint close-out commit. BCE-CL-014: ENH-NNN_BCE_IMPACT.md template added as new Section 14 — authoritative source; PBVI skill and PBVI templates file cross-reference this section; template no longer duplicated in PBVI skill. |
| v1.2 | April 2026 | BCE-CL-006: Artifact model redesigned — seven artifacts in three roles (prerequisite, living extraction, attestation) replacing ambiguous "six canonical artifacts" count. BCE-CL-007: INTAKE_SUMMARY.md gate condition made path-specific — mandatory pre-gate for Paths A/B, first output of Stage 1 for Path C. BCE-CL-008: INTAKE_SUMMARY.md lifecycle clarified — point-in-time record, not updated after Stage 1. BCE-CL-009: "Why INTAKE_SUMMARY is Not a Cover Sheet" explanation added to Section 5. BCE-CL-010: Path C description updated to reflect INTAKE_SUMMARY as Stage 1 output. BCE-CL-011: When to Update Artifacts table updated — enhancement close-out row added, explicit INTAKE_SUMMARY exclusion. |

## 1. What This Skill Does

This skill makes Claude Desktop (CD) a co-pilot for BCE extraction, annotation, and
artifact assembly. It covers the full extraction lifecycle — intake, session execution,
component file review, contradiction resolution, artifact assembly, and final CD
cross-artifact review — for both PBVI-governed and non-PBVI systems. A non-PBVI engineer
loads only this skill. A PBVI engineer loads both this skill and the PBVI org skill.

---

## 2. BCE/SIL Naming Convention

**BCE** (Brownfield Context Extraction) is the process name — used in practitioner guides,
skill files, and engineering documentation. **SIL** (System Intelligence Layer) is the
output name — used in external and client-facing communication. These are not interchangeable.
Internally: BCE Adapter Pipeline. Externally: SIL Bootstrap Pipeline.

---

## 3. BCE Artifacts — Seven in Total, Three Roles

BCE extraction produces seven artifacts organised into three roles. Every artifact is
mandatory for every DataGrokr-managed system. Artifacts that cannot be completed due to
insufficient information must be produced with explicit NOT DETERMINABLE entries — not omitted.

### Role 1 — Prerequisite Artifact (produced once, not updated)

| Artifact | File | What it captures |
|---|---|---|
| Pre-Extraction Intake Summary | INTAKE_SUMMARY.md | What was known before extraction began — from docs, runbooks, Confluence, Jira, prior engineer conversations, or PBVI planning artifacts on Path C. Point-in-time record. Not updated after Stage 1. Enhancement close-outs do not touch it. |

### Role 2 — Living Extraction Artifacts (updated at enhancement close-out)

| # | Artifact | File | What it captures |
|---|---|---|---|
| 1 | System Topology Map | TOPOLOGY.md | Layer boundaries, module call map, external system boundary contracts |
| 2 | Module Contract Registry | MODULE_CONTRACTS.md | Per-module: inputs, outputs, error behaviour, fragilities, change impact |
| 3 | Integration Contract Registry | INTEGRATION_CONTRACTS.md | External system promises vs. application assumptions — and where they diverge |
| 4 | Implicit Invariant Catalogue | INVARIANT_CATALOGUE.md | Conditions that must always be true — whether or not currently enforced |
| 5 | Risk and Debt Register | RISK_REGISTER.md | Known fragilities, operational gaps, technical debt — with severity and action |

### Role 3 — Attestation Artifact (produced at Stage 3, actively maintained)

| Artifact | File | What it captures |
|---|---|---|
| BCE Backlog | ANNOTATION_CHECKLIST.md | BCE completeness attestation and backlog — what the system knows it doesn't know well. Produced at Stage 3 from the five living artifacts. New items added at every enhancement close-out. Most actively maintained of all seven artifacts. |

**CODEBASE_MAP.md** is a navigational prerequisite for extraction — not one of the seven.
It is generated mechanically at Session A0 using Template T0, before any topology prompts
run. It gives engineers and CC a confirmed file inventory so all subsequent extraction
prompts are anchored to verified paths. It does not require annotation and is not a
living document in the same sense as the five extraction artifacts.

**SYSTEM_GRAPH.json** is a machine-readable derived artifact produced at Stage 3 close-out
from the five living artifacts — not one of the seven. It represents the system as a
typed nodes-and-edges graph (modules, integration points, invariants, risks, and their
relationships). It is regenerated at every sprint close-out when BCE artifacts are updated.
It is never edited directly. See Section 11.8 for the graph construction prompt and
Section 16 for the subgraph context query used in PBVI Phase 1 planning.

**DOMAIN_MODEL.json** is a machine-readable derived artifact produced by Session F (three
sub-sessions: F01 structural, F02 vocabulary, F03 CD synthesis) — not one of the seven.
It represents the domain entity model as a typed nodes-and-edges graph (entities, attributes,
status vocabularies, status values, and relationships with full semantic annotation). It is
mandatory for any system with a detectable data layer; systems without a data layer produce
a sparse stub. It bridges to SYSTEM_GRAPH.json via a common ID namespace, enabling
cross-graph traversal from behavioral nodes to domain entity nodes. See Section 11.3 for
the full Session F specification.

**discovery/ID_REGISTRY.md** is a new coordination artifact — a sequential log of every
ID assigned across both graphs (SYSTEM_GRAPH.json and DOMAIN_MODEL.json). It guarantees
no ID collisions across separate extraction sessions. Produced by CC at Session A (for
M-NNN and IP-NNN) and extended at Session F03 (for E-NNN, A-NNN, SV-NNN, SVV-NNN,
REL-NNN). Updated at every sprint close-out when new IDs are assigned.

---

## 4. Extraction Paths

### Path A — Custodian-Led Extraction

Use when: the team includes engineers with direct working knowledge of the system. The intake
summary is rich, NOT DETERMINABLE entries are resolved during annotation rather than carried
forward, and the gap between structural extraction and operational understanding is small.

Key discipline on Path A: even when engineers know the system well, do not skip extraction
and write artifacts from memory. Memory and code diverge. Extraction surfaces the divergence.

### Path B — Stranger-Led Extraction

Use when: the team has no prior knowledge of the system. The intake summary will be thin,
CC extraction will produce many NOT DETERMINABLE entries, and annotation will be partial.
The resulting artifacts have explicit gaps and open questions. Gaps are filled progressively
as the team gains operational knowledge.

A context package with honest gaps is more valuable than no context package at all.

### Path C — PBVI Adapter Pipeline

Use when: the system was built under PBVI governance and all governing documents are
present in `docs/`. This is the fast path.

The adapter pipeline replaces the manual intake step with CC-driven docs extraction.
Stage 1 opens by producing INTAKE_SUMMARY.md from PBVI planning artifacts
(ARCHITECTURE.md, INVARIANTS.md, EXECUTION_PLAN.md, Claude.md, manifests) — these
are the I01–I05 source set. INTAKE_SUMMARY.md is the first Stage 1 output, not a
precondition to Stage 1. There is no coordination overhead, interview step, or waiting —
all intake material is already in docs/. The human gate between Stage 1 and Stage 2 is
where INTAKE_SUMMARY.md is reviewed, not separately before Stage 1 begins.

Stage 2 runs standard BCE extraction seeded with Stage 1 output. Stage 3 runs the
standard CD cross-artifact review.

All three stage execution prompts are embedded in this skill — see Section 10 (Stage 1),
Section 11 (Stage 2), Section 12 (Stage 3). They are standard methodology artifacts and
do not vary by project. Do not create per-project copies in docs/prompts/.

Note: all Stage 1 artifacts carry header marker `STAGE-1-DRAFT: DOCS-DERIVED`.

---

## 5. Pre-Extraction Intake Step

**For Paths A and B:** INTAKE_SUMMARY.md must be written, reviewed, and committed
to `discovery/` before Stage 1 begins. It is the pre-extraction gate — Stage 1 is
seeded by this document and cannot run without it. No CC extraction prompt may run
until INTAKE_SUMMARY.md is committed.

**For Path C:** INTAKE_SUMMARY.md is the first output of Stage 1, not a precondition
to it. CC produces it from the PBVI planning artifacts (ARCHITECTURE.md, INVARIANTS.md,
EXECUTION_PLAN.md, Claude.md) as the opening step of Stage 1. There is no coordination
overhead, interview step, or waiting — all intake material is already in docs/.
INTAKE_SUMMARY.md on Path C is reviewed as part of the Stage 1 human gate, not
separately before it.

### Why INTAKE_SUMMARY.md Is Not a Cover Sheet

INTAKE_SUMMARY.md is a functional extraction anchor, not a project formality.
Stage 1 uses it to seed extraction in three specific ways:

**1. Known systems and data models anchor T4 skeletons and A01/A03 topology.**
CC uses stated architecture, components, and integration points from the intake to
produce module contract skeletons and layer boundary records. Without this anchor,
Stage 1 produces generic structural records with no design context — the equivalent
of mapping a city from aerial photography without knowing what the buildings are for.

**2. Stated invariants and constraints seed INVARIANT_CATALOGUE.md candidates.**
On Path C, INVARIANTS.md is the intake source. On Paths A and B, custodian knowledge
about what must always be true — even if undocumented — is captured here and carried
into invariant candidate generation.

**3. STAGE-2-DIVERGENCE markers are meaningful only against a Stage 1 baseline.**
When Stage 2 code extraction disagrees with Stage 1 docs-derived content, the
divergence flag surfaces a gap between design intent and implementation reality.
Without a Stage 1 baseline — which itself requires a completed intake — there is
nothing to diverge from. Stage 2 becomes a raw extraction with no signal about
whether what was found was expected or surprising.

**Quality signal:** A thin INTAKE_SUMMARY.md on Path B is expected and acceptable —
the honest acknowledgment that little was known before extraction began. A thin
INTAKE_SUMMARY.md on Path A or C is a process failure — it means documented or
derivable knowledge was not used. The quality of Stage 1 output is directly
proportional to the quality of the intake.

### Canonical INTAKE_SUMMARY.md Template (six sections)

```markdown
# INTAKE_SUMMARY.md — [System Name]
**Date:** [date]
**Engineer:** [name]
**Path:** [A — Custodian / B — Stranger / C — PBVI Adapter]
**Canonical layer boundary:** [Which layer or schema represents the canonical entity model for this system — e.g. "gold layer in dbt project", "Django ORM models", "public schema in Postgres". Required before Session F begins. Set to N/A if no data layer.]
**Session F disposition:** [FULL EXTRACTION / CATALOG STUB / SPARSE STUB — set from three-step pre-assessment in Section 5 below. Must be recorded before INTAKE_SUMMARY.md is committed. Session F01 may not begin until this field is set.]

## System Purpose
[What does this system do in business terms — one paragraph maximum]

## Known Architecture
[What is known about system structure before looking at code]

## Known Pain Points
[Operational problems, fragilities, or debt items already documented or known]

## Documents Reviewed
[List of every document read, with a one-line summary of what it contributed]

## Open Questions Before Extraction
[Questions that could not be answered from available documentation]

## Confidence Assessment
[HIGH / MEDIUM / LOW — how complete is the pre-code picture]
```

### Session F Pre-Assessment (complete before committing INTAKE_SUMMARY.md)

Session F (Domain Model Extraction) runs immediately after Session A on all paths. Its output is determined by the following three-step assessment, which must be completed before INTAKE_SUMMARY.md is committed. Record the outcome in the `Session F disposition` field.

**Step 1 — Catalog check.** Does a maintained metadata catalog already own an active business glossary, column-level descriptions, and lineage for this system? (AWS Glue Data Catalog, Unity Catalog, Collibra, Alation, Apache Atlas, or equivalent.)
- If **yes** → Session F produces a **catalog stub** (`sparse: true, catalogBacked: true`). Record: CATALOG STUB.

**Step 2 — Structural check.** Does a data layer exist that is structurally detectable in the codebase? (ORM model definitions, database migration scripts, DDL files, schema declarations.)
- If **yes** → Session F produces a **full extraction** via F01 → F02 → F03. Record: FULL EXTRACTION.
- If **no** → Session F produces a **sparse stub** (`sparse: true`). Record: SPARSE STUB.

**Step 3 — Canonical boundary declaration (FULL EXTRACTION only).** Declare which layer or schema represents the canonical entity model and record it in `canonical_layer_boundary` in INTAKE_SUMMARY.md. Session F entity promotion is bounded to this declared layer — anything outside it is excluded. Set `canonical_layer_boundary` to N/A for CATALOG STUB and SPARSE STUB.

| Disposition | canonical_layer_boundary | Session F output |
|---|---|---|
| FULL EXTRACTION | Required — declare layer | Full DOMAIN_MODEL.json |
| CATALOG STUB | N/A | Sparse DOMAIN_MODEL.json (`sparse: true, catalogBacked: true`) |
| SPARSE STUB | N/A | Sparse DOMAIN_MODEL.json (`sparse: true`) |

Both stubs are valid BCE outputs. DOMAIN_MODEL.json is committed in all cases.

---

### Authorship Rule

Substantive content must come from verifiable sources. When inputs are structured and
high-fidelity (PBVI artifacts, formal architecture docs), CC may synthesise them into
INTAKE_SUMMARY format — this is the normal Path C case. When inputs are unstructured or
absent (Path B), the engineer must supply content directly. The human review gate before
Stage 2 is the enforcement point — not a categorical prohibition on CC drafting.

### BCE-S — Non-Code Source Extraction

For Path B, non-code source extraction (documents, team communications, Git history) is
owned by the BCE-S pipeline. See **`bce_signal.md`** for full adapter procedures:
Stage 1 (Documents, Teams, and Git adapters), Stage 2 validation, Stage 3 CD assembly,
and SIGNAL_GAPS.md structure.

On Path B, BCE-S Stage 1 runs before BCE-C. BCE-S Stage 2 and Stage 3 run after
BCE-C completes. INTAKE_SUMMARY.md on Path B is synthesised by CD at BCE-S Stage 3
from validated Stage 1 component files — not engineer-authored.

On Path A, BCE-S runs after BCE-C completes using the full BCE-C artifact set as the
Stage 2 validation baseline. SOURCE_INVENTORY.md is produced from custodian interview
responses before BCE-S Stage 1 begins.

---

## 6. Extraction Sessions

Extraction is organised into sessions. Each session targets a defined scope. Sessions run
sequentially — do not begin a later session until the earlier session's artifact is committed
and reviewed.

| Session | Scope | Artifacts produced |
|---|---|---|
| A0 | Codebase map — always first | CODEBASE_MAP.md |
| A | System topology | TOPOLOGY.md (assembled from A01 + A02 + A03) |
| F | Domain model (conditional — data layer required) | DOMAIN_MODEL.json (or sparse stub) |
| B | Serving layer modules | MODULE_CONTRACTS.md (serving layer section) |
| C | Pipeline modules | MODULE_CONTRACTS.md (complete) |
| D | Implicit invariant walkthrough | INVARIANT_CATALOGUE.md |
| E | Integration contracts + risk register | INTEGRATION_CONTRACTS.md, RISK_REGISTER.md |

**Session F trigger condition (assessed at intake via three-step pre-condition):**
Session F is mandatory when the semantic layer is absent or implicit. Step 1: does a
maintained metadata catalog (AWS Glue, Unity Catalog, Collibra, Alation, Apache Atlas,
or equivalent) already own an active business glossary, column-level descriptions, and
lineage? If yes — Session F produces a catalog stub, not a full extraction. Step 2: does
a data layer exist structurally (ORM definitions, migration scripts, or DDL)? If yes —
full extraction. If none — sparse stub. Step 3: before F01 begins, the engineer declares
the canonical layer boundary and records it in INTAKE_SUMMARY.md as
`canonical_layer_boundary`. Both sparse stubs and catalog stubs are valid BCE outputs.

**Step A0 always runs first** using Template T0 — a directory traversal with one-line
purpose descriptions per file. Engineer reviews it against the intake summary to confirm
the file inventory is complete. Only after A0 is reviewed do the three topology prompts
(A01/A02/A03) run.

**Topology is three component files assembled into one artifact:**
- A01 (T1 — Layer Boundary Map)
- A02 (T2 — Module Call Map — requires source code, always deferred to Stage 2 in Path C)
- A03 (T3 — External System Boundary Map)

**Component file naming:** `[Session][NN]_[module_name].md`
Examples: `A01_layer_boundary_map.md`, `B01_main.md`, `B02_router.md`

In the PBVI adapter pipeline: Stage 1 skeletons use prefix S1. Stage 2 Sessions B and C
replace S1 skeletons.

---

## 7. Template Reference (T0–T7)

Full template text lives in the BCE methodology document. This section describes what
each template produces and why it is a separate prompt — not the executable template
content. Execution prompts for Path C (PBVI Adapter Pipeline) are embedded in Sections
10, 11, and 12 of this skill.

| Template | What it produces | Why separate |
|---|---|---|
| T0 | CODEBASE_MAP.md — complete file inventory with one-line purpose per file | Navigational prerequisite; must be confirmed before any topology work begins |
| T1 | Layer Boundary Map — what artifacts cross system boundaries and how | Layer boundaries require reading only boundary-crossing code; reading everything would introduce noise |
| T2 | Module Call Map — who calls whom, startup sequence, async boundaries. Session A first produces a Module Roster assigning M-NNN IDs; the call table emits typed edges: M-NNN --[CALLS]--> M-NNN. | Requires full read of the target layer; cross-module calls only (same-module excluded) |
| T3 | External System Boundary Map — one record per external system called. Session A assigns IP-NNN IDs to each external system record during A03 reconciliation. Called-by fields reference M-NNN. | External calls require tracing auth, error handling, and retry patterns — a distinct analytical pass |
| T4 | Module Contract — per-module record. References M-NNN assigned in Session A; integration call fields reference IP-NNN from A03. | One prompt per module ensures focused extraction; cross-module synthesis is a separate step |
| T5 | Invariant Candidate Walkthrough — IC-N format candidates. Owning module and enforcing module fields reference M-NNN. | Invariant extraction requires walking every data touchpoint; bundled with module extraction it produces goal statements, not constraints |
| T6 | F01 Structural Inventory — raw entity/table list, relationship inventory, constraint inventory, divergence flags. Output: discovery/F01_structural_inventory.md | Structural extraction reads schema sources in priority order; analytical separation from vocabulary extraction prevents conflating structure with semantics |
| T7 | F02 Vocabulary Extraction — per-field vocabulary sets, cardinality samples from seed data, null frequency, naming pattern flags. Output: discovery/F02_vocabulary_extraction.md | Vocabulary sources (seed data, fixtures, enum types) require a different read pattern than schema sources; combined with F01 it produces noise |

---

## 8. The Annotation Loop

### What Annotation Means

Annotation converts a structurally accurate CC extraction into an operationally accurate
record. It has three inputs: CC extraction output, CD structured review, and the engineer's
tacit knowledge. Without annotation, the output is a structural map. With annotation, it is
a system intelligence layer. They are not the same thing.

### ENGINEER NOTE Format

All engineer annotations are marked with:

```
[ENGINEER NOTE — YYYY-MM-DD]: [content]
```

This provenance marking is mandatory. It distinguishes code-derived content from
human-supplied knowledge and makes the date of annotation visible for maintenance.

### CD Review Structure

When a component file is submitted to CD for review, CD produces a structured review with
four sections:

1. **Overall Assessment** — one sentence on output quality
2. **Items Requiring Confirmation** — specific questions that cannot be resolved from
   extraction alone. Each includes: what is ambiguous, why it matters, what answer is needed.
3. **Items to Annotate From Tacit Knowledge** — fields where human operational knowledge is
   needed. Not extraction failures — the honest boundary of what code alone can supply.
4. **No Issues With** — explicit confirmation of which sections are accepted without change.

### Handling NOT DETERMINABLE FROM SOURCE Entries

| Path | Action |
|---|---|
| Path A — Custodian | Resolve during annotation. If engineer knows the answer, write as ENGINEER NOTE. If no one knows, log as open question in artifact header and as context gap in RISK_REGISTER.md. |
| Path B — Stranger | Most will remain unresolved in first pass. Track as open questions. Fill progressively as the team gains operational knowledge. |
| Both paths — Never do this | Delete NOT DETERMINABLE entries or invent answers. The honest gap is more valuable than a fabricated answer. |

### Contradiction Resolution Rule

When two component files produce contradictory findings about the same behaviour: CD
identifies the contradiction and states both positions explicitly. Engineer determines the
correct answer (code re-examination, team confirmation, or operational evidence). Both
component files are updated with the confirmed answer. Resolution documented with ENGINEER NOTE.

**Never assemble an artifact with a known unresolved contradiction. Resolve first.**

### Human Annotation Record (HAR)

The Human Annotation Record is the formal artifact produced each time an engineer
resolves an ANNOTATION_CHECKLIST.md item via human annotation. It is the source
document for every RESOLVED-ANNOTATION entry in the Resolution Log.

**Why it exists:** The ENGINEER NOTE embedded in a BCE artifact records *what* was
supplied. The HAR records *who supplied it, what they said verbatim, and which
artifact field changed*. Without it, RESOLVED-ANNOTATION entries in the Resolution
Log have no traceable source.

**ID convention:** HAR-NNN (sequential per project, starting at HAR-001).

**Directory rules:**
- Greenfield or high-volume extraction (Stage 1–3): `discovery/annotations/HAR-NNN_[short-title].md`
  Create `discovery/annotations/` on first HAR; register the directory in
  PROJECT_MANIFEST.md at that point.
- Enhancement-level: `enhancements/ENH-NNN/HAR-NNN_[short-title].md`
  Scoped inside the enhancement package — no separate directory needed.

**When to produce:** One HAR per ANNOTATION_CHECKLIST.md item resolved via the
Annotation Amendment prompt (Section 14.3). The prompt produces the HAR as part of
the per-item steps, before the human gate.

**Template:**

```markdown
# HAR-NNN — [short title derived from annotation item]
**Checklist item:** [ANNOTATION_CHECKLIST.md item ID, e.g. P1-001]
**Engineer:** [name]
**Date:** YYYY-MM-DD

## Raw Annotation

[Exact text as provided by the engineer — verbatim, not paraphrased]

## BCE Artifact Updated

**Artifact:** [TOPOLOGY | MODULE_CONTRACTS | INTEGRATION_CONTRACTS | INVARIANT_CATALOGUE | RISK_REGISTER]
**Section:** [exact section name within artifact]
**Field:** [exact field or row identifier]

## ENGINEER NOTE Applied

[ENGINEER NOTE — YYYY-MM-DD]: [structured summary as applied to the artifact — exact text]

## Resolution Log Entry

| [ITEM-ID] | RESOLVED-ANNOTATION | [engineer] | YYYY-MM-DD | HAR-NNN |
```

---

## 9. ANNOTATION_CHECKLIST.md

ANNOTATION_CHECKLIST.md is the BCE backlog — a living record of everything the system
knows it doesn't know well. It is a first-class BCE artifact, not a working document.
It is never empty on a real system. Honest extraction always produces a backlog.

### Item Structure

Every checklist item has five fields:

| Field | Values | Notes |
|---|---|---|
| ID | P1-NNN, P2-NNN, P3-NNN, CON-NNN | Stage 3 items: P1-S3-NNN. Enhancement items: P1-ENH001-NNN |
| Severity | P1 \| P2 \| P3 | P1 blocks enhancement planning. P2 resolves before first enhancement. P3 informational |
| Type | NOT_DETERMINABLE \| CONTRADICTION \| OPEN_QUESTION \| SCHEMA_VIOLATION | What kind of gap this is |
| Source | CODE_EXTRACTION \| HUMAN_ANNOTATION \| STAGE3_REVIEW \| ENHANCEMENT_CLOSEOUT | What produced this item |
| Surfaced by | CC \| CD \| ENGINEER:[name] | Who or what found it |

**Default severity by type — apply unless explicitly overridden with rationale:**
- CONTRADICTION → P1. Two artifacts giving conflicting ground truth is the most dangerous gap.
- NOT_DETERMINABLE → P2. Tracked gap, not immediately blocking unless it affects a known planning decision (then P1).
- OPEN_QUESTION → P3. Informational unless it affects an invariant or integration contract (then P2).
- SCHEMA_VIOLATION → P1. A relationship field uses a prose name instead of an ID reference (M-NNN or IP-NNN). Produced only by Stage 3 Check 0. Stage 3 does not complete until all SCHEMA_VIOLATION items are resolved.

Downgrading from the default requires a stated rationale. Upgrading never requires justification.

### Item Format

Each item uses narrative format — not table rows. Tables are for the resolution log only.

```markdown
## [ITEM-ID] · [short title] ([artifact affected])

**Severity:** P1 | P2 | P3
**Type:** NOT_DETERMINABLE | CONTRADICTION | OPEN_QUESTION
**Source:** CODE_EXTRACTION | HUMAN_ANNOTATION | STAGE3_REVIEW | ENHANCEMENT_CLOSEOUT
**Surfaced by:** CC | CD | ENGINEER:[name]
**Artifact:** [TOPOLOGY | MODULE_CONTRACTS | INTEGRATION_CONTRACTS |
               INVARIANT_CATALOGUE | RISK_REGISTER | CROSS_ARTIFACT]
**Section:** [exact section name within artifact]

**Observation:** [what was found — specific, not general]

**Risk for planning:** [what goes wrong if this item is not resolved before
the next enhancement begins]

**Recommended action:** [exactly what needs to happen to resolve this item]

**Engineer action required:** [what the engineer must confirm or supply]

**STATUS:** OPEN | SIGNED-OFF | RESOLVED | PARTIALLY_RESOLVED | ESCALATED
[If SIGNED-OFF: Engineer name, date]
[If PARTIALLY_RESOLVED: what remains open and why]
[If ESCALATED: new item ID created]
```

### Resolution Log

At the end of ANNOTATION_CHECKLIST.md, after all items:

```markdown
## Resolution Log

| Item ID | Resolution type | Resolved by | Date | Evidence |
|---|---|---|---|---|
| P1-001 | RESOLVED-CODE | [name] | YYYY-MM-DD | [what confirmed it] |
| P2-003 | RESOLVED-ANNOTATION | [name] | YYYY-MM-DD | HAR-NNN |
| CON-001 | CONFIRMED-CONTRADICTION / FAIL-ACCEPTED | [name] | YYYY-MM-DD | [rationale] |

Resolution type vocabulary:
RESOLVED-CODE — answer found in source code
RESOLVED-ANNOTATION — resolved via human annotation; evidence field contains HAR-NNN reference
CONFIRMED-CONTRADICTION / FAIL-ACCEPTED — contradiction confirmed, accepted as known gap
RESOLVED-INFORMATIONAL — item confirmed as not requiring action
```

### Cross-Artifact Consistency Check

ANNOTATION_CHECKLIST.md ends with a mandatory cross-artifact consistency check section.
This is produced at Stage 3 and updated at enhancement close-out.

```markdown
## Cross-Artifact Consistency Check
**Last run:** [date] **By:** [CC | CD | ENGINEER]

| Check | Status | Notes |
|---|---|---|
| All invariants in INVARIANT_CATALOGUE.md match INVARIANTS.md | PASS / FAIL | |
| All module names in MODULE_CONTRACTS.md match TOPOLOGY.md | PASS / FAIL | |
| All external systems in INTEGRATION_CONTRACTS.md match TOPOLOGY.md A03 | PASS / FAIL | |
| All risks in RISK_REGISTER.md reference correct source artifacts | PASS / FAIL | |
| INTAKE_SUMMARY.md open questions accounted for in artifacts | PASS / FAIL | |
```

### Stage Progression

**Stage 1 (CC):** Produces stub with items identified from docs-read. P1/P2/P3 sections
populated. All items have Source: CODE_EXTRACTION or HUMAN_ANNOTATION. Surfaced by: CC.

**Stage 2 (CC):** Resolves items resolvable from source code. Updates STATUS fields.
Adds new items surfaced by code extraction. Adds Stage 2 Completeness Summary — does not
update Stage 1 Completeness Summary in-place (SC-001 rule).

**Stage 3 (CD):** Finalises. Promotes resolved items. Escalates unresolved P1 items.
Produces cross-artifact consistency check. All P1 items must be SIGNED-OFF or RESOLVED
before Stage 3 completes.

**Enhancement close-out:** New items added with Source: ENHANCEMENT_CLOSEOUT and ID
format P[N]-ENH[NNN]-[NNN]. These become the BCE backlog for the next sprint.

### SC-001 — Completeness Summary Rule

Each stage adds its own frozen Completeness Summary section. Stage 2 never updates
Stage 1's summary in-place — it adds a new frozen Stage 2 Completeness Summary below.
Each stage's summary is immutable once that stage is complete.

### SC-002 — Contradiction Resolution Placement

Contradiction resolutions belong in the Resolution Log — never in the Completeness
Summary. The Completeness Summary records counts and status only.

---

## 10. Stage 1 — Docs-Derived Extraction (CC) — Path C Only

**Stage 1 exists on Path C (PBVI Adapter Pipeline) only.** On Path A (Custodian-Led) and
Path B (Stranger-Led), there is no Stage 1 — the custodian intake (Path A) or BCE-S Stage 1
(Path B) replaces it. If you are on Path A or Path B, start with Session A0 instead:
trigger phrase "Run BCE Session A0".

Stage 1 runs in **CC** against docs/ and verification/S9. It does not read source code.
It produces draft BCE artifacts from PBVI planning documents. All Stage 1 artifacts carry
the header marker `STAGE-1-DRAFT: DOCS-DERIVED`.

**Artifact assignments — Stage 1 produces drafts of:**
- INTAKE_SUMMARY.md — synthesised from ARCHITECTURE.md, INVARIANTS.md, Claude.md
- TOPOLOGY.md — A01 (Layer Boundary Map) and A03 (External System Boundary Map) from ARCHITECTURE.md. A02 (Module Call Map) deferred to Stage 2 — requires source code.
- MODULE_CONTRACTS.md — T4 skeletons per module from ARCHITECTURE.md and EXECUTION_PLAN.md. Stage 2 completes them.
- INVARIANT_CATALOGUE.md — invariants from INVARIANTS.md. Stage 2 confirms enforcement points.
- RISK_REGISTER.md — draft risks from docs/PHASE4_GATE_RECORD.md, ARCHITECTURE.md key risks, and known planning gaps.

**Stage 1 does NOT produce:**
- INTEGRATION_CONTRACTS.md — requires reading actual API call code and error handler logic. Deferred to Stage 2.
- ANNOTATION_CHECKLIST.md — produced at Stage 3 only.
- A02 (Module Call Map) — deferred to Stage 2.

**Human gate — mandatory before Stage 2 begins:**
Engineer reviews all Stage 1 draft artifacts. Confirms they are reasonable starting points.
Signs off. Stage 2 does not begin until this gate passes.

### Stage 1 Execution Prompt

**Tool:** CC
**Path:** C (PBVI Adapter Pipeline) only — do not run on Path A or Path B projects
**Trigger phrases:**
- "Run BCE Stage 1"
- "Start BCE Stage 1 extraction"
- "Run docs-derived extraction"

```
Before doing anything else, check whether this is a Path C (PBVI Adapter Pipeline) project:
- Check whether docs/ARCHITECTURE.md exists.
- If it does not exist: STOP immediately. Output the following and do nothing else:

  "Stage 1 is for Path C (PBVI Adapter Pipeline) only. It reads PBVI planning artifacts
  (ARCHITECTURE.md, INVARIANTS.md, EXECUTION_PLAN.md, Claude.md) to produce draft BCE
  artifacts automatically.

  On Path A (Custodian-Led) and Path B (Stranger-Led), there is no Stage 1 — the custodian
  intake (Path A) or BCE-S Stage 1 (Path B) replaces it.

  If you are on Path A or Path B: start with Session A0 instead.
  Trigger phrase: 'Run BCE Session A0'

  If you intended to run Path C: ensure docs/ARCHITECTURE.md, docs/INVARIANTS.md, and
  docs/EXECUTION_PLAN.md are committed to docs/ before triggering Stage 1."

- If docs/ARCHITECTURE.md exists: proceed with the prompt below.

You are running BCE Adapter Pipeline Stage 1 for a PBVI-governed system.

Read the following files in full before producing any output:
- docs/ARCHITECTURE.md
- docs/INVARIANTS.md
- docs/EXECUTION_PLAN.md
- docs/Claude.md
- PROJECT_MANIFEST.md
- verification/ (S9 VERIFICATION_CHECKLIST.md if present)
- docs/PHASE4_GATE_RECORD.md if present

All output artifacts must carry this header on line 1:
STAGE-1-DRAFT: DOCS-DERIVED — [date] — Produced by BCE Adapter Pipeline Stage 1

Do not read source code. Do not invent behaviour not described in docs/.
Mark any field that cannot be derived from the above documents as:
NOT DETERMINABLE FROM SOURCE

Produce the following artifacts in order. Save each to discovery/ before
proceeding to the next.

---

ARTIFACT 1 — discovery/INTAKE_SUMMARY.md

Using the canonical six-section template:
- System Purpose: from ARCHITECTURE.md problem framing
- Known Architecture: from ARCHITECTURE.md key design decisions
- Known Pain Points: from docs/PHASE4_GATE_RECORD.md accepted risks and ARCHITECTURE.md key risks
- Documents Reviewed: list all docs/ files read with one-line contribution summary
- Open Questions Before Extraction: from ARCHITECTURE.md open questions section
- Confidence Assessment: HIGH (PBVI artifacts are structured and high-fidelity)

---

ARTIFACT 2 — discovery/TOPOLOGY.md (partial — A01 and A03 only)

Section A01 — Layer Boundary Map:
From ARCHITECTURE.md component breakdown, produce one boundary record per layer crossing:
- Produced by: [module and function]
- Artifact: [type, format, schema]
- Consumed by: [module and function]
- Handoff mechanism: [direct call / file / database / message queue / manual]
- Failure mode: NOT DETERMINABLE FROM SOURCE (requires source code)

Section A02 — Module Call Map:
DEFERRED TO STAGE 2 — requires source code
Write: "A02 — DEFERRED: Module Call Map requires source code inspection. Produced in Stage 2."

Section A03 — External System Boundary Map:
From ARCHITECTURE.md fixed stack and integration descriptions, produce one record per
external system. Mark all error handling and auth credential fields as NOT DETERMINABLE
FROM SOURCE unless explicitly stated in docs/.

---

ARTIFACT 3 — discovery/MODULE_CONTRACTS.md (T4 skeletons)

For each module identified in ARCHITECTURE.md and EXECUTION_PLAN.md, produce a T4
skeleton with these fields:
- Module, Layer, Primary Responsibility: from ARCHITECTURE.md
- Inputs, Outputs: from EXECUTION_PLAN.md task descriptions where available, else NOT DETERMINABLE FROM SOURCE
- External Dependencies, Internal Dependencies: from ARCHITECTURE.md fixed stack
- Public Interface, Error Behaviour, Known Fragility, Change Impact: NOT DETERMINABLE FROM SOURCE

Label each skeleton: S1-[NN]_[module_name] — Stage 2 will complete these.

---

ARTIFACT 4 — discovery/INVARIANT_CATALOGUE.md

For each invariant in docs/INVARIANTS.md, produce one IC record:
- ID: IC-[N] (match INV-[N] numbering)
- Statement: exact text from INVARIANTS.md
- Category: Data Correctness / Operational / Security / Integration
- Scope: GLOBAL | TASK-SCOPED — from INVARIANTS.md Scope field
- Why it matters: from INVARIANTS.md "why this matters" field
- Currently enforced: NOT DETERMINABLE FROM SOURCE (requires source code)
- Source: from INVARIANTS.md enforcement points where stated

---

ARTIFACT 5 — discovery/RISK_REGISTER.md

From docs/PHASE4_GATE_RECORD.md (if present) and ARCHITECTURE.md key risks section:
For each ACCEPT decision in docs/PHASE4_GATE_RECORD.md Section D, produce one risk entry:
- Risk ID: R-[NNN]
- Description: from risk finding text
- Severity: from risk register severity (Critical/High/Medium/Low → map to P1/P2/P3)
- Source artifact: docs/PHASE4_GATE_RECORD.md
- Mitigation: from ACCEPT rationale
- Recommended action: NOT DETERMINABLE FROM SOURCE (requires operational assessment)

For each key risk in ARCHITECTURE.md not covered by docs/PHASE4_GATE_RECORD.md:
produce a stub entry with severity NOT DETERMINABLE FROM SOURCE.

---

After producing all five artifacts, produce a Stage 1 Completeness Summary:

## Stage 1 Completeness Summary — [date]
Produced by: BCE Adapter Pipeline Stage 1 (CC)

| Artifact | Status | NOT DETERMINABLE count | Notes |
|---|---|---|---|
| INTAKE_SUMMARY.md | COMPLETE | [n] | |
| TOPOLOGY.md | PARTIAL — A02 deferred | [n] | A02 requires Stage 2 |
| MODULE_CONTRACTS.md | SKELETONS ONLY | [n] | Stage 2 completes |
| INVARIANT_CATALOGUE.md | DOCS-DERIVED | [n] | Stage 2 confirms enforcement |
| RISK_REGISTER.md | PARTIAL | [n] | Stage 2 adds code-observed risks |
| INTEGRATION_CONTRACTS.md | NOT PRODUCED | — | Stage 2 only artifact |

Human gate required before Stage 2 begins.
Engineer sign-off: [LEAVE BLANK]
```

---

## 11. Stage 2 — Code Extraction

Stage 2 runs in **CC** against source code, seeded with Stage 1 output. It completes
all Stage 1 drafts and produces INTEGRATION_CONTRACTS.md.

**Artifact assignments — Stage 2 produces or completes:**
- TOPOLOGY.md — completes A02 (Module Call Map). Reconciles A01 and A03 against code.
- MODULE_CONTRACTS.md — completes T4 fields left NOT DETERMINABLE in Stage 1 skeletons.
- INTEGRATION_CONTRACTS.md — produced entirely in Stage 2. Reads API call code, response parsing, error handler logic, auth patterns.
- INVARIANT_CATALOGUE.md — confirms enforcement points against actual code. Adds invariants implied by code behaviour not in docs/.
- RISK_REGISTER.md — completes with code-observed fragilities and risks surfaced by extraction.

**Stage 2 does NOT produce:**
- ANNOTATION_CHECKLIST.md — produced at Stage 3 only.

**Divergence rule:** Where Stage 2 code extraction diverges from Stage 1 docs-derived
content — flag the divergence explicitly. Do not silently resolve by preferring one source.
Divergences surface gaps between what was designed and what was built.

### 11.1 Session A0 — Codebase Map

**Tool:** CC
**Trigger phrases:**
- "Run BCE Session A0"
- "Run codebase map"
- "Create codebase map"
- "Build codebase map"

**Prerequisite:** INTAKE_SUMMARY.md committed to discovery/

```
You are running BCE Stage 2 Session A0 — Codebase Map.

Read discovery/INTAKE_SUMMARY.md before starting. This gives you the intake
context to anchor the file inventory.

Perform a complete directory traversal of the source repository. For every
file: record its path and write one line describing its purpose.

Group output by directory. Include all source files, configuration files,
and infrastructure-as-code files. Exclude node_modules/, .git/, build/,
dist/, and generated artefact directories.

Save the output as discovery/components/A00_codebase_map.md using this format:

## A00 — Codebase Map
Produced by: BCE Stage 2 Session A0 (CC)
Date: YYYY-MM-DD

### [directory/]
- [filename] — [one-line purpose]
...

After saving: state that A00 is complete and that an engineer must review
it against INTAKE_SUMMARY.md before Sessions A–E begin. Do not proceed to
Session A without confirming engineer review.
```

---

### 11.2 Session A — System Topology

**Tool:** CC
**Trigger phrases:**
- "Run BCE Session A"
- "Run topology extraction"
- "Create topology"
- "Extract topology"
- "Build system topology"

**Prerequisite:** A00_codebase_map.md committed and engineer-reviewed

```
You are running BCE Stage 2 Session A — System Topology.

Read these artifacts before starting:
- discovery/INTAKE_SUMMARY.md
- discovery/TOPOLOGY.md (Stage 1 draft — contains A01 and A03)
- discovery/components/A00_codebase_map.md

Read all source files in the serving layer and pipeline layer completely.

PART 0 — Assign Module IDs (mandatory — complete before Part 1):

Enumerate every module in the codebase. A module is any cohesive unit with
its own primary responsibility — a class, a service entry point, a route
handler group, or a pipeline stage. Assign M-NNN IDs sequentially starting
at M-001. These IDs are permanent for the life of this project — do not
reassign them at later sessions.

Layer values (PBVI-011 extended):
- `serving` — API routes, handlers, controllers (server-side request entry points)
- `pipeline` — data transformation stages, batch jobs, workers
- `infra` — configuration, initialisation, infrastructure-as-code
- `route` — URL routing definition that maps paths to page components
  (applies when APPLICATION_SURFACE contains UI)
- `page` — top-level screen component — directly corresponds to a UI_SURFACE.md
  screen entry (applies when APPLICATION_SURFACE contains UI)
- `layout` — shared wrapper component — navigation shell, authenticated layout,
  etc. (applies when APPLICATION_SURFACE contains UI)
- `component` — reusable UI component — not a full screen, not a layout
  (applies when APPLICATION_SURFACE contains UI)
- `store` — client-side state management — Redux store, Zustand store, React
  context provider (applies when APPLICATION_SURFACE contains UI)

For API_ONLY or BACKGROUND_SERVICE projects, the UI layer types do not apply.
For UI+API projects, both server-side (serving/pipeline/infra) and client-side
(route/page/layout/component/store) modules are extracted.

Produce a Module Roster as the opening section of A02:

## Module Roster — [system name]
Generated: YYYY-MM-DD by BCE Stage 2 Session A (CC)
Note: these IDs are permanent. Do not reassign at later sessions.

| ID | Module Name | Source File | Layer |
|---|---|---|---|
| M-001 | [name] | [file path] | [serving / pipeline / infra / route / page / layout / component / store] |
| M-002 | ... | | |

This roster is the canonical ID-to-module mapping for all subsequent
sessions. Sessions B, C, D, and E must reference modules by M-NNN in
all relationship fields — never by prose name.

**UI_SURFACE.md cross-reference (PBVI-011 — UI projects only):**

If `docs/UI_SURFACE.md` exists for this project (APPLICATION_SURFACE contains UI),
load it during Module Roster production and reconcile:

- Every screen in UI_SURFACE.md Screen Inventory must map to at least one module
  with Layer = `page`. Missing page modules are P1 annotation items.
- Every Global Element in UI_SURFACE.md (Navigation, Authentication Shell, etc.)
  must map to at least one module with Layer = `layout` or `component`.
- Route modules (Layer = `route`) must reference page modules they map to.

UI_SURFACE.md is dual-registered in PROJECT_MANIFEST.md (Planning Artifacts and
Discovery Artifacts). The Planning Artifacts row is canonical. The Discovery
Artifacts row is the cross-reference handle for BCE Stage 2 and Stage 3.

Brownfield UI layer extraction — producing UI_SURFACE.md from an existing
codebase rather than from a requirements brief — is out of scope here and is
tracked as BCE-006 (follow-on item).

PART 1 — Produce A02 (Module Call Map):
Save as discovery/components/A02_module_call_map.md
(The Module Roster from Part 0 is the opening section — append the call
table, startup sequence, and async boundaries below it.)

Section 1 — Internal Call Table
For every cross-module call in the codebase, emit a typed edge:

| Edge | Call Site (file:line) | Sync/Async |
|---|---|---|
| M-NNN --[CALLS]--> M-NNN | | S / A |

Section 2 — Startup Sequence
For each step in the application startup sequence:
| Step | Module (M-NNN) | Action | Failure Mode (STARTUP-FATAL / NON-FATAL) |

Section 3 — Async Boundaries
For each asynchronous handoff point:
| Producer (M-NNN) | Consumer (M-NNN) | Mechanism | Failure behaviour |

PART 2 — Reconcile A01 and A03 against source code:
For each Layer Boundary entry in A01: confirm or correct the failure mode field.
For each External System entry in A03: assign IP-NNN (sequential from IP-001)
as the first field of each record; confirm or correct auth mechanism, error
handling, and request/response fields. Add a "Called by" field referencing
M-NNN from the Module Roster for each module that calls this external system.
These IP-NNN IDs are permanent for the life of this project.

Update discovery/TOPOLOGY.md with completed A02 and any A01/A03 corrections.
Mark corrections with:
[STAGE-2-UPDATE — YYYY-MM-DD]: [what changed and why]

Mark any field that cannot be determined from source code as:
NOT DETERMINABLE FROM SOURCE

For any field where Stage 2 extraction diverges from Stage 1 content:
[STAGE-2-DIVERGENCE — YYYY-MM-DD]: Stage 1 stated [X]. Code shows [Y].
Resolution required before Stage 3.

After saving TOPOLOGY.md: produce discovery/ID_REGISTRY.md with all M-NNN and
IP-NNN IDs assigned in this session. Use the format specified in Section 11.3
(ID_REGISTRY.md — Format and Maintenance). If ID_REGISTRY.md already exists
(migration from pre-v2.1), append new rows only — do not overwrite existing entries.

After saving: state that Session A is complete and that TOPOLOGY.md and
ID_REGISTRY.md must be committed and engineer-reviewed before Session F and
Sessions B and C begin.
```

---

### 11.3 Session F — Domain Model Extraction

**Trigger:** Session F runs after Session A is committed and before Sessions B and C.
Session F is mandatory when the semantic layer is absent or implicit. Run the three-step
pre-condition assessment at intake before any Session F work begins.

**Step 1 — Is a maintained, queryable semantic layer already in tooling?**
A maintained semantic layer means: a metadata catalog (AWS Glue, Unity Catalog, Collibra,
Alation, Apache Atlas, or equivalent) with an active business glossary, column-level
descriptions, and lineage. If yes: Session F produces a catalog stub — record catalog
name and reference URL in DOMAIN_MODEL.json header, set `"sparse": true` and
`"catalogBacked": true`. BCE does not replicate what a catalog already owns.
If no maintained semantic layer exists in tooling: proceed to Step 2.

**Step 2 — Does a data layer exist structurally?**
Any of: ORM model definitions (Django models, Prisma schema, SQLAlchemy, ActiveRecord,
Hibernate, or equivalent); migration scripts; a database schema dump or DDL. If yes:
Session F runs full extraction. If none: produce a sparse stub (`"sparse": true`).

**Step 3 — Engineer declares the canonical layer boundary.**
Before F01 begins, the engineer declares which layer or schema represents the canonical
entity model for this system. This is recorded in `INTAKE_SUMMARY.md` under
`canonical_layer_boundary`. "Gold layer" is not a universal definition — some teams place
the canonical model in silver, some in gold, some in a dedicated conformed layer. Session F
extraction is bounded to the declared boundary. Tables outside it are excluded from
DOMAIN_MODEL.json and appear in TOPOLOGY.md and MODULE_CONTRACTS.md as pipeline internals.

Session F has three sub-sessions (F01, F02, F03). All three run in order.

---

#### F01 — Structural Extraction (CC) → Template T6

**Tool:** CC
**Trigger phrases:**
- "Run Session F01"
- "Run F01 structural extraction"
- "Run BCE structural inventory"

**Prerequisite:** Session A committed (A02 Module Roster establishes M-NNN context for
cross-graph edges in the later synthesis step).
**Output:** `discovery/F01_structural_inventory.md`
**Header marker:** `STAGE-F1-DRAFT: STRUCTURAL`

```
You are running BCE Session F01 — Structural Extraction.

Before reading any files, apply the three-step pre-condition assessment:

STEP 1 — Read discovery/INTAKE_SUMMARY.md. Check whether a maintained metadata
catalog is recorded (field: canonical_layer_boundary or any catalog reference).
A maintained semantic layer means: AWS Glue, Unity Catalog, Collibra, Alation,
Apache Atlas, or equivalent — with active business glossary, column-level
descriptions, and lineage. If the engineer has confirmed a maintained catalog is
present: produce a catalog stub (discovery/F01_structural_inventory.md) with
header CATALOG-BACKED: [catalog name]. Record the catalog reference. Do not
proceed to structural extraction. Session F03 will produce a stub DOMAIN_MODEL.json
referencing the catalog.

STEP 2 — If no maintained catalog: check whether a data layer exists structurally:
- ORM model definitions (Django models, Prisma schema, SQLAlchemy,
  ActiveRecord, Hibernate, or equivalent), OR
- Migration scripts in the repository, OR
- A database schema dump or DDL

If none: produce a sparse stub (discovery/F01_structural_inventory.md) with
header SPARSE: NO DATA LAYER DETECTED and a single entry explaining the absence.
Do not proceed further.

STEP 3 — Read discovery/INTAKE_SUMMARY.md field canonical_layer_boundary.
This is the declared scope of extraction. Only include tables, models, and
schemas within the declared canonical layer. Record the boundary declaration at
the top of the output under "Canonical layer boundary: [declaration]".

If STEP 2 passed and STEP 3 has a declaration: read available schema sources
in priority order, scoped to the canonical layer.
Stop after reading source types that are present — do not wait for missing types.

Source priority order:
1. ORM model definitions — entities, relationships, and types named explicitly
2. Migration scripts — complete structural history in creation order
3. Schema dump / DDL — current state; surfaces schema drift from migrations
4. GraphQL schema — entity shapes as understood by API consumers
5. OpenAPI / JSON Schema — external contract shapes; may diverge from storage
6. Existing ERD / data dictionary — apply STALENESS_RISK flag if older than 12 months

Produce discovery/F01_structural_inventory.md with header STAGE-F1-DRAFT: STRUCTURAL
and four tables:

**Entity Inventory**
| Entity/Table Name | Columns | Types | Notes |
|---|---|---|---|
(Every candidate entity or table. Do not promote or exclude at this stage — record all.)

**Relationship Inventory**
| Relationship | Declaration Type | Source Entity | Target Entity | Notes |
|---|---|---|---|---|
Declaration Type: FK | ORM_RELATION | JOIN_TABLE | INFERRED

**Constraint Inventory**
| Entity | Constraint Type | Fields | Notes |
|---|---|---|---|
Constraint Type: UNIQUE | CHECK | INDEX | NOT_NULL

**Divergence Flags**
| Flag ID | Source A | Source B | What Diverges | Notes |
|---|---|---|---|---|
Flag ID: STAGE-F1-DIVERGENCE-NNN
(Populate only when schema dump disagrees with migrations. Leave empty if
sources are consistent.)

After saving: state that F01 is complete and that discovery/F01_structural_inventory.md
must be committed and reviewed before F02 begins.
```

---

#### F02 — Vocabulary Extraction (CC) → Template T7

**Tool:** CC
**Trigger phrases:**
- "Run Session F02"
- "Run F02 vocabulary extraction"
- "Run BCE vocabulary extraction"

**Prerequisite:** F01 output committed. If F01 produced a SPARSE stub, F02 also
produces a sparse stub — do not run against a system with no data layer.
**Output:** `discovery/F02_vocabulary_extraction.md`

```
You are running BCE Session F02 — Vocabulary Extraction.

Read discovery/F01_structural_inventory.md first. If it carries SPARSE: NO DATA
LAYER DETECTED, produce a sparse stub: discovery/F02_vocabulary_extraction.md
with header SPARSE and a single entry noting no data layer. Do not proceed further.

Read vocabulary sources:
- Seed data and fixture files — actual values in status and enum fields
- API / GraphQL schemas — enum definitions, constrained string types
- Test fixtures — edge-case values, boundary conditions

Produce discovery/F02_vocabulary_extraction.md with four tables:

**Per-Field Vocabulary**
| Entity | Field | Observed Values | Source |
|---|---|---|---|
(One row per status or enum field. List every value observed across all sources.)

**Cardinality Samples**
| Relationship | Observed Cardinality | Sample Size | Notes |
|---|---|---|---|
(From seed data — actual relationship cardinality, not declared cardinality.)

**Null Frequency**
| Entity | Field | Nullable (declared) | Null in Seed Data | Notes |
|---|---|---|---|
(Flag fields that are nullable but never null in seed data — and fields that
are always null — both are annotation candidates.)

**Naming Pattern Flags**
| Concept | Name in Source A | Name in Source B | Notes |
|---|---|---|---|
(Where the same concept appears under different names across sources or tables.
These are disambiguation candidates for the annotation pass.)

After saving: state that F02 is complete and that discovery/F02_vocabulary_extraction.md
must be committed and reviewed before F03 begins.
```

---

#### F03 — Domain Model Synthesis (CD)

**Tool:** CD
**Trigger phrases:**
- "Run Session F03"
- "Run domain model synthesis"
- "Build domain model"
- "Generate DOMAIN_MODEL.json"

**Prerequisite:** F01 and F02 outputs committed.
**Output:** `discovery/DOMAIN_MODEL.json`

```
You are running BCE Session F03 — Domain Model Synthesis.

Read before producing any output:
- discovery/INTAKE_SUMMARY.md (canonical_layer_boundary field)
- discovery/F01_structural_inventory.md
- discovery/F02_vocabulary_extraction.md
- discovery/TOPOLOGY.md (A02 Module Roster — M-NNN IDs for cross-graph edges)
- discovery/ID_REGISTRY.md (existing IDs — check for conflicts before assigning new ones)

If F01 carries CATALOG-BACKED: produce a stub DOMAIN_MODEL.json with
"sparse": true, "catalogBacked": true, and record the catalog name and
reference URL from the F01 header. Do not proceed to synthesis.

If either F01 or F02 carries SPARSE: NO DATA LAYER DETECTED, produce a sparse
DOMAIN_MODEL.json with "sparse": true, "nodes": [], "edges": [], and a single
annotation: "Session F produced a sparse stub — no detectable data layer."
Do not proceed further.

For non-sparse systems: apply these judgment calls in order.

STEP 1 — Read canonical layer boundary
Read the canonical_layer_boundary field from INTAKE_SUMMARY.md. This is the
declared scope of extraction — only entities within this boundary are promoted
to DOMAIN_MODEL.json. Tables and schemas outside the boundary are pipeline
internals recorded in TOPOLOGY.md and MODULE_CONTRACTS.md, not here.

STEP 2 — Entity Promotion
Decide which entities/tables from the F01 inventory are domain entities within
the canonical layer boundary:
- Domain entities: named business objects (Customer, Shipment, Alert, Policy)
  that fall within the declared canonical layer
- Exclude: pure join tables, platform tables (migrations, jobs, sessions,
  audit logs), infrastructure tables, and tables outside the canonical boundary
- For each candidate you exclude: state the rationale as a node annotation
- Assign E-NNN IDs sequentially (check ID_REGISTRY.md first)
- For each promoted entity, set all four semantic properties to NOT_DETERMINABLE:
  business_definition, lifecycle_summary, synonyms, domain
  These require annotation — do not attempt to derive from schema

STEP 3 — Attribute Extraction
For each promoted entity, create one Attribute node per column:
- Assign A-NNN IDs sequentially (check ID_REGISTRY.md first)
- Propose null_semantic for each nullable field from F01/F02 context:
  NOT_APPLICABLE (field does not apply in this state),
  NOT_YET_RECORDED (value will be filled later),
  ABSENT (entity exists without this property),
  NOT_DETERMINABLE (cannot infer from sources — requires annotation)
- Flag is_identifier for primary and natural keys
- Flag disambiguation_note where F02 naming pattern flags apply

STEP 4 — Vocabulary Synthesis
For each status or enum field in F02:
- Create one StatusVocabulary node (SV-NNN) per field
- Create one StatusValue node (SVV-NNN) per value
- Propose is_terminal from seed data patterns (values with no outgoing
  transitions in the data) — NOT_DETERMINABLE where insufficient evidence
- Set transition_trigger to NOT_DETERMINABLE for all transitions at this
  stage — this field requires annotation
- Emit CAN_TRANSITION_TO edges where F02 seed data shows observable transitions

STEP 5 — Relationship Naming
For each relationship in the F01 inventory between promoted entities:
- Assign REL-NNN IDs sequentially (check ID_REGISTRY.md first)
- Assign a business-meaningful relationship name (not the FK column name)
- Propose cardinality from F01 declaration and F02 cardinality samples
- Set creating_event and dissolution_semantic to NOT_DETERMINABLE — these
  require annotation

STEP 6 — Divergence Resolution
Review all STAGE-F1-DIVERGENCE-NNN flags from F01:
- If resolvable from context (e.g., DDL represents current state after
  migrations): resolve and note the resolution in the relevant node annotation
- If not resolvable: produce one ANNOTATION_CHECKLIST.md item per unresolved
  flag (Type: OPEN_QUESTION, default severity P2) and mark the affected
  node with NOT_DETERMINABLE

STEP 7 — Cross-Graph Edge Annotation
For entities where ownership is inferable from TOPOLOGY.md A02 (module is
the authoritative writer), add a cross_graph_edges annotation to the Entity
node listing: { "from": "M-NNN", "to": "E-NNN", "type": "OWNS_ENTITY" }.
These will be incorporated into SYSTEM_GRAPH.json when the graph is rebuilt.

STEP 8 — Metrics Layer (conditional)
Check whether F01/F02 sources indicate a defined metrics layer: dbt metrics
definitions, LookML, semantic layer tool, or explicit metric documentation.
- If yes: set "metricsLayer": true in the header. Scaffold Metric nodes (MT-NNN)
  from extracted metric definitions. Set business_definition, grain, time_dimension,
  and is_additive to NOT_DETERMINABLE for each — these require annotation.
  Set calculation_logic_summary from extraction where derivable.
  Add DERIVED_FROM edges where source entities or attributes are identifiable.
- If no: set "metricsLayer": false. Do not include any Metric nodes or
  DERIVED_FROM/RELATED_METRIC edges. Do not stub.

Produce discovery/DOMAIN_MODEL.json with this structure:

{
  "system": "[system name from INTAKE_SUMMARY.md]",
  "generated": "YYYY-MM-DD",
  "bce_version": "v2.1",
  "sparse": false,
  "catalogBacked": false,
  "grokLayer": false,
  "metricsLayer": false,
  "canonical_layer_boundary": "[from INTAKE_SUMMARY.md]",
  "nodes": [
    {
      "id": "E-NNN", "type": "Entity", "name": "[entity name]",
      "business_definition": "NOT_DETERMINABLE",
      "lifecycle_summary": "NOT_DETERMINABLE",
      "synonyms": [],
      "domain": "NOT_DETERMINABLE"
    },
    {
      "id": "A-NNN", "type": "Attribute",
      "name": "[field name]",
      "canonical_name": "[preferred name or same as name if unambiguous]",
      "data_type": "[data type]",
      "business_meaning": "[one sentence or NOT_DETERMINABLE]",
      "null_semantic": "NOT_APPLICABLE|NOT_YET_RECORDED|ABSENT|NOT_DETERMINABLE",
      "disambiguation_note": "[if overloaded — else omit]",
      "is_identifier": false
    },
    { "id": "SV-NNN", "type": "StatusVocabulary", "name": "[vocabulary name]" },
    {
      "id": "SVV-NNN", "type": "StatusValue",
      "value": "[exact string as it appears in data]",
      "business_meaning": "[one sentence or NOT_DETERMINABLE]",
      "is_terminal": false,
      "transition_trigger": "NOT_DETERMINABLE"
    },
    {
      "id": "REL-NNN", "type": "Relationship",
      "name": "[business relationship name]",
      "cardinality": "ONE_TO_ONE|ONE_TO_MANY|MANY_TO_MANY",
      "optional": true,
      "creating_event": "NOT_DETERMINABLE",
      "dissolution_semantic": "NOT_DETERMINABLE"
    }
  ],
  "edges": [
    { "from": "E-NNN", "to": "A-NNN", "type": "HAS_ATTRIBUTE" },
    { "from": "A-NNN", "to": "SV-NNN", "type": "HAS_VOCABULARY" },
    { "from": "SV-NNN", "to": "SVV-NNN", "type": "CONTAINS_VALUE" },
    { "from": "SVV-NNN", "to": "SVV-NNN", "type": "CAN_TRANSITION_TO" },
    { "from": "REL-NNN", "to": "E-NNN", "type": "FROM_ENTITY" },
    { "from": "REL-NNN", "to": "E-NNN", "type": "TO_ENTITY" }
  ]
}

When metricsLayer: true, Metric node template:
{
  "id": "MT-NNN", "type": "Metric", "name": "[canonical metric name]",
  "business_definition": "NOT_DETERMINABLE",
  "calculation_logic_summary": "[derived from source or NOT_DETERMINABLE]",
  "grain": "NOT_DETERMINABLE",
  "time_dimension": "NOT_DETERMINABLE",
  "is_additive": "NOT_DETERMINABLE"
}
And edges: { "from": "MT-NNN", "to": "E-NNN or A-NNN", "type": "DERIVED_FROM" }

After producing DOMAIN_MODEL.json: update discovery/ID_REGISTRY.md with all new
E-NNN, A-NNN, SV-NNN, SVV-NNN, REL-NNN, and MT-NNN (if any) IDs assigned.

Produce a brief synthesis summary:
- Entity count (promoted vs. excluded from F01 inventory; note canonical boundary)
- Attribute count
- StatusVocabulary count and StatusValue count
- Relationship count
- Metric count (if metricsLayer: true)
- NOT_DETERMINABLE field count (annotation candidates)
- ANNOTATION_CHECKLIST.md items produced (if any)

State that F03 is complete and that the annotation pass should be run before
Sessions B and C begin — entity ownership context and semantic properties from
the domain model will improve module contract extraction precision.
```

---

#### DOMAIN_MODEL.json — Full Schema Reference

##### ID Namespace (Session F — registered in ID_REGISTRY.md)

| Prefix | Type | Graph |
|---|---|---|
| E-NNN | Entity | DOMAIN_MODEL.json |
| A-NNN | Attribute | DOMAIN_MODEL.json |
| SV-NNN | StatusVocabulary | DOMAIN_MODEL.json |
| SVV-NNN | StatusValue | DOMAIN_MODEL.json |
| REL-NNN | Relationship | DOMAIN_MODEL.json |
| MT-NNN | Metric | DOMAIN_MODEL.json (conditional — present only when metricsLayer: true) |

IDs are assigned sequentially by CD during F03 synthesis, starting from the
next available number after checking ID_REGISTRY.md. No prefix overlaps with
SYSTEM_GRAPH.json IDs (M-NNN, IP-NNN, IC-N, R-NNN).

##### Node Types

| Type | ID Format | What it represents |
|---|---|---|
| Entity | E-NNN | A named business object — Customer, Shipment, Alert, Policy. Not every table — domain objects only, not join tables, platform tables, or audit tables. |
| Attribute | A-NNN | A property of an entity. Separate node (not inline on Entity) to enable rich annotation and direct cross-graph edge targeting from SYSTEM_GRAPH.json invariants. |
| StatusVocabulary | SV-NNN | A controlled set of values governing a status or enum field. |
| StatusValue | SVV-NNN | A single value within a vocabulary (PENDING, RESOLVED, IN_TRANSIT, etc.). |
| Relationship | REL-NNN | A named relationship between two entities. Reified as a node to carry cardinality, optionality, creating event, and dissolution semantics as node properties. |
| Metric | MT-NNN | A derived business measure — Revenue, Churn Rate, On-Time Delivery %. **Conditional section** — present only when a defined metrics layer exists (dbt metrics definitions, LookML, semantic layer tool, or explicit metric documentation). Never stubbed; omitted entirely when absent. |

##### Entity Node Properties

Each Entity node carries:

| Property | What it captures | Source |
|---|---|---|
| `name` | Canonical entity name | Extraction |
| `business_definition` | What this entity is in business terms — 2–3 sentences, sufficient for a planning AI to reason about it without additional context | Engineer annotation |
| `lifecycle_summary` | The major phases of this entity's existence in business terms — the business story from creation to end state, distinct from StatusVocabulary field-level transitions | Engineer annotation |
| `synonyms` | Other names this entity is known by across teams, systems, or vocabularies — what a business user might call it that differs from the canonical name. Critical for GrokLayer natural language query resolution. | Engineer annotation |
| `domain` | Business domain classification — Logistics, Finance, Operations, Identity, etc. Relevant for multi-domain systems and GrokLayer query routing | Engineer annotation |

All four semantic properties are NOT DETERMINABLE from schema alone. They are added to the engineer annotation pass.

##### Attribute Node Properties

| Property | What it captures |
|---|---|
| `name` | Field name as it appears in schema |
| `canonical_name` | Preferred name if schema name is abbreviated or ambiguous |
| `data_type` | Data type |
| `business_meaning` | What this field represents in business terms — one sentence |
| `null_semantic` | NOT_APPLICABLE / NOT_YET_RECORDED / ABSENT / NOT_DETERMINABLE |
| `disambiguation_note` | Present when name is overloaded or same concept appears under different names elsewhere |
| `is_identifier` | Boolean — is this the primary or natural key for the entity |

##### StatusValue Node Properties

| Property | What it captures |
|---|---|
| `value` | Exact string as it appears in data |
| `business_meaning` | What this state means operationally |
| `is_terminal` | Boolean — no valid transitions out of this state |
| `transition_trigger` | What business event causes entry into this state |

##### Relationship Node Properties

| Property | What it captures |
|---|---|
| `name` | Business name of the relationship |
| `cardinality` | ONE_TO_ONE / ONE_TO_MANY / MANY_TO_MANY |
| `optional` | Boolean — is the relationship required or optional |
| `creating_event` | Business event that establishes this relationship |
| `dissolution_semantic` | What it means when this relationship is dissolved or null |

##### Metric Node Properties

*(Conditional — present only when metricsLayer: true)*

Each Metric node carries:

| Property | What it captures | Source |
|---|---|---|
| `name` | Canonical metric name | Extraction |
| `business_definition` | What this metric measures and what business question it answers — 2–3 sentences | Engineer annotation |
| `calculation_logic_summary` | How the metric is derived — which entities and attributes feed it, what the formula or aggregation is, what filters apply. Summary only — full logic lives in the metrics layer tool. | Extraction + engineer annotation |
| `grain` | The level of aggregation at which this metric is defined — per Customer, per Product, per Region, etc. A metric without grain is undefined. | Engineer annotation |
| `time_dimension` | How time factors into this metric — point-in-time, period-over-period, trailing window, cumulative | Engineer annotation |
| `is_additive` | Boolean — whether this metric can be summed across dimensions without distortion. Non-additive metrics (ratios, percentages, distinct counts) require special handling in GrokLayer query resolution. | Engineer annotation |

##### Edge Types Within DOMAIN_MODEL.json

| Edge | From → To | What it means |
|---|---|---|
| `HAS_ATTRIBUTE` | Entity → Attribute | This entity has this property |
| `HAS_VOCABULARY` | Attribute → StatusVocabulary | This attribute's values are governed by this vocabulary |
| `CONTAINS_VALUE` | StatusVocabulary → StatusValue | This value belongs to this vocabulary |
| `CAN_TRANSITION_TO` | StatusValue → StatusValue | Valid state transition |
| `FROM_ENTITY` | Relationship → Entity | The source entity of this relationship |
| `TO_ENTITY` | Relationship → Entity | The target entity of this relationship |
| `SAME_AS` | Entity → Entity | These two entities in different systems represent the same business concept — for portfolio-level semantic alignment |
| `DERIVED_FROM` | Metric → Entity or Attribute | The source data this metric is computed from — enables blast radius reasoning for metric-affecting changes (conditional) |
| `RELATED_METRIC` | Metric → Metric | This metric is a component of, variant of, or decomposition of another metric (conditional) |

##### Cross-Graph Edge Types (live in SYSTEM_GRAPH.json, target E-NNN or A-NNN)

| Edge | From → To | What it means |
|---|---|---|
| `OWNS_ENTITY` | Module (M-NNN) → Entity (E-NNN) | This module is the authoritative writer for this entity |
| `READS_ENTITY` | Module (M-NNN) → Entity (E-NNN) | This module reads but does not own this entity |
| `ENFORCES_ENTITY_INVARIANT` | Invariant (IC-N) → Entity (E-NNN) | This invariant constrains a property of this entity |
| `ENFORCES_ATTRIBUTE_INVARIANT` | Invariant (IC-N) → Attribute (A-NNN) | This invariant constrains a specific attribute |

---

#### Engineer Annotation Pass (after F03)

Fields requiring human input — cannot be derived from schema sources or seed data:

| Field | Why human input is required |
|---|---|
| Entity `business_definition` | Domain knowledge — cannot be derived from schema |
| Entity `lifecycle_summary` | Operational knowledge of the entity's business story — not in schema |
| Entity `synonyms` | Cross-team vocabulary knowledge — not in schema |
| Entity `domain` | Domain classification judgment — not derivable from schema alone |
| `transition_trigger` | What business event causes a status change — not in schema or seed data |
| `null_semantic` | Whether null means not applicable, not yet recorded, or absent — requires operational knowledge |
| Entity ownership | Which module is the authoritative writer — requires cross-referencing behavioral extraction |
| `dissolution_semantic` | What it means when a FK is nulled or a record deleted |
| `disambiguation_note` resolutions | Where the same concept has multiple names — only the engineer knows the canonical choice |
| Metric `grain` and `time_dimension` | Business definition of aggregation scope — not in metrics tool config alone (conditional — when metricsLayer: true) |
| Metric `is_additive` | Additivity is a semantic property requiring business knowledge — not derivable from formula (conditional — when metricsLayer: true) |

Annotation follows the HAR pattern (Section 8). HARs for Session F stored in
`discovery/annotations/` alongside behavioral extraction HARs.

---

#### ID_REGISTRY.md — Format and Maintenance

**Purpose:** Single sequential log of every ID assigned across both graphs. Guarantees
no ID collisions across separate extraction sessions.

**Location:** `discovery/ID_REGISTRY.md`

**When produced:** CC creates ID_REGISTRY.md at Session A with M-NNN and IP-NNN entries.
CD extends it at Session F03 with E-NNN, A-NNN, SV-NNN, SVV-NNN, REL-NNN, and MT-NNN
(when metricsLayer: true) entries. Updated at every sprint close-out when new IDs are assigned.

**Format:**

```markdown
# ID Registry — [System Name]

| ID | Type | Name | Graph | Assigned |
|---|---|---|---|---|
| M-001 | Module | [module name] | SYSTEM_GRAPH.json | Session A |
| IP-001 | IntegrationPoint | [external system] | SYSTEM_GRAPH.json | Session A |
| IC-1 | Invariant | [invariant text] | SYSTEM_GRAPH.json | Session D |
| R-001 | RiskItem | [risk description] | SYSTEM_GRAPH.json | Session E |
| E-001 | Entity | [entity name] | DOMAIN_MODEL.json | Session F03 |
| A-001 | Attribute | [attribute name] | DOMAIN_MODEL.json | Session F03 |
| SV-001 | StatusVocabulary | [vocabulary name] | DOMAIN_MODEL.json | Session F03 |
| SVV-001 | StatusValue | [value] | DOMAIN_MODEL.json | Session F03 |
| REL-001 | Relationship | [relationship name] | DOMAIN_MODEL.json | Session F03 |
| MT-001 | Metric | [metric name] | DOMAIN_MODEL.json (conditional) | Session F03 |
```

**Migration note for existing projects (pre-v2.1):** Create discovery/ID_REGISTRY.md
by populating M-NNN and IP-NNN rows from TOPOLOGY.md A02 Module Roster and A03
External Systems. IC-N and R-NNN rows from INVARIANT_CATALOGUE.md and RISK_REGISTER.md.
Run Session F03 to populate domain model IDs.

---

#### GrokLayer Extension Layer

For GrokLayer systems (`"grokLayer": true` in DOMAIN_MODEL.json header), an additional
section is appended after the core graph — the **entity-to-tool mapping**. This section
is produced after GrokLayer tool definitions are drafted, not during Session F.

It records:
- Which tools resolve which entities (coverage map)
- Entities with no resolving tool (coverage gaps)
- Tools whose parameter names diverge from canonical attribute names in the model
- Status filter parameters mapped to exact SVV-NNN vocabulary values

The `grokLayer` flag determines whether this section is expected. Omit for
non-GrokLayer systems.

---

#### Conditional Sections Summary

DOMAIN_MODEL.json has two conditional sections that are omitted entirely when their
trigger condition is not met. They are **never stubbed** with NOT DETERMINABLE entries —
absence is the correct representation when the section does not apply.

| Section | Trigger | Header flag |
|---|---|---|
| GrokLayer Extension Layer | System is a GrokLayer deployment | `grokLayer: true` |
| Metrics | System has a defined metrics layer — dbt metrics definitions, LookML, semantic layer tool, or explicit metric documentation | `metricsLayer: true` |

For data lake and BI systems where the Metrics section is present, detailed extraction
guidance — source priority for metrics definitions, dbt metrics YAML parsing, LookML
interpretation — lives in PWS-002 (Greenfield Data Lake Companion Guide), not here.
BCE-005 defines the schema; PWS-002 defines the extraction practice for metrics-heavy
contexts.

---

#### Production Path 2 — GrokLayer Greenfield Dataset

When a GrokLayer dataset is built from scratch (no existing system to reverse-engineer),
DOMAIN_MODEL.json is authored as a forward design artifact before seed data generation —
same schema, different production path:

1. Engineer drafts entity list and relationship structure
2. CC scaffolds DOMAIN_MODEL.json skeleton from the draft
3. Engineer completes semantic annotation (business meanings, state transitions, null
   semantics — these cannot be scaffolded)
4. GrokLayer extension layer added after tool definitions are drafted
5. Seed data generated to express the completed model

For extraction from existing systems (reverse-engineering path): run Sessions F01, F02,
F03 against existing schema sources.

---

#### Maintenance at Enhancement Close-Out

DOMAIN_MODEL.json is updated at enhancement close-out using the targeted update pattern:

- CC re-extracts only affected nodes from source (triggered by Domain Model Impact
  section of ENH-NNN_BCE_IMPACT.md listing affected E-NNN/A-NNN/REL-NNN)
- CD reviews for consistency
- DOMAIN_MODEL.json updated in place for affected nodes only
- Full regeneration required only when a schema migration restructures more than 30%
  of entities (engineer judgment call, stated in the BCE impact log)
- ID_REGISTRY.md updated with any new IDs assigned

---

### 11.4 Sessions B and C — Module Contracts

**Tool:** CC
**Trigger phrases:**
- "Run BCE Sessions B and C"
- "Run module contract extraction"
- "Extract module contracts"
- "Create module contracts"
- "Build module contracts"

**Prerequisite:** TOPOLOGY.md committed with A02 complete

```
You are running BCE Stage 2 Sessions B and C — Module Contracts.

Read these artifacts before starting:
- discovery/TOPOLOGY.md (for module list and layer assignments)
- discovery/MODULE_CONTRACTS.md (Stage 1 skeletons — S1-NN entries)
- discovery/components/A00_codebase_map.md (for file paths)
- discovery/components/A02_module_call_map.md (for M-NNN assignments from the Module Roster and typed call edges)

For each module skeleton in discovery/MODULE_CONTRACTS.md:
Read the module's source file completely.
Produce one component file using the fields below.

Save serving-layer modules as discovery/components/B[NN]_[module_name].md
Save pipeline-layer modules as discovery/components/C[NN]_[module_name].md

Per-module component file format:

## [Session][NN] — [Module Name]
ID: M-NNN (from A02 Module Roster)
Layer: [serving / pipeline]
Source file: [path]

**Module** — [name]
**ID** — M-NNN
**Layer** — [serving / pipeline]
**Primary Responsibility** — [one sentence]

**Inputs**
[exact types, sources, validation — one entry per input]

**Outputs**
[what is produced or mutated — all side effects]

**Public Interface**
[every function callable by other modules — exact signatures]

**Error Behaviour**
[per function — raised vs caught vs swallowed; what the caller experiences on failure]

**Known Fragility**
[specific things a future engineer could break without realising]

**Change Impact**
[what else is at risk if this module changes]

**Callers** — M-NNN [, M-NNN ...] (from A02 typed CALLS edges where this module is the callee)
**Calls** — M-NNN [, M-NNN ...] (from A02 typed CALLS edges where this module is the caller)
**Integration Points Used** — IP-NNN [, IP-NNN ...] (from A03 Called-by fields)

Mark any field that cannot be determined from source code as:
NOT DETERMINABLE FROM SOURCE

For any field where Stage 2 extraction diverges from Stage 1 skeleton content:
[STAGE-2-DIVERGENCE — YYYY-MM-DD]: Stage 1 stated [X]. Code shows [Y].
Resolution required before Stage 3.

After all component files are saved: update discovery/MODULE_CONTRACTS.md
with completed contracts, replacing the S1-NN skeletons.

After saving: state that Sessions B and C are complete and that
MODULE_CONTRACTS.md must be committed before Session D begins.
```

---

### 11.5 Session D — Invariant Catalogue

**Tool:** CC
**Trigger phrases:**
- "Run BCE Session D"
- "Run invariant extraction"
- "Extract invariants"
- "Create invariant catalogue"
- "Build invariant catalogue"

**Prerequisite:** MODULE_CONTRACTS.md committed

```
You are running BCE Stage 2 Session D — Invariant Catalogue.

Read these artifacts before starting:
- discovery/INVARIANT_CATALOGUE.md (Stage 1 records — IC-N and IC-CANDIDATE entries)
- discovery/MODULE_CONTRACTS.md (for Known Fragility and enforcement context)
- discovery/components/A02_module_call_map.md (for M-NNN assignments from the Module Roster)
- docs/INVARIANTS.md (if present — source of truth for declared invariants)
- discovery/components/A00_codebase_map.md (for file paths)

PART 1 — Confirm existing IC records:
For each IC-N record in discovery/INVARIANT_CATALOGUE.md:
- Confirm or correct the "Currently enforced" field from source code
- Add enforcement point: exact file path and function name
- Confirm or correct the Source field

For each IC-CANDIDATE record from Stage 1:
- Confirm whether it is a genuine invariant (a constraint that must always hold)
  or a goal (a desirable outcome, not a hard constraint)
- If confirmed invariant: assign ID IC-[N] and complete all fields
- If not an invariant: document the reason and remove the IC-CANDIDATE entry

PART 2 — Surface implicit invariants from code:
Walk every data touchpoint in the codebase. For any constraint that the code
enforces implicitly — a check, a guard, an assumption baked into the logic —
that has no corresponding IC-N entry: produce a new IC-N record.

Each IC-N record format:
**IC-N — [Invariant statement]**
Scope: GLOBAL / TASK-SCOPED [specify which tasks]
Source: Code-observed / Docs-declared / Both
Currently enforced: YES / NO / PARTIAL
Enforcement point: [file:function]
Owning module: M-NNN (the module that declares or most directly owns this invariant)
Enforcing modules: M-NNN [, M-NNN ...] (all modules with enforcement points — from Module Contracts Known Fragility and Public Interface fields)
Rationale: [why this must always hold]

Update discovery/INVARIANT_CATALOGUE.md.

Mark any field that cannot be determined from source code as:
NOT DETERMINABLE FROM SOURCE

For any field where Stage 2 extraction diverges from Stage 1 content:
[STAGE-2-DIVERGENCE — YYYY-MM-DD]: Stage 1 stated [X]. Code shows [Y].
Resolution required before Stage 3.

After saving: state that Session D is complete and that
INVARIANT_CATALOGUE.md must be committed before Session E begins.
```

---

### 11.6 Session E — Integration Contracts and Risk Register

**Tool:** CD
**Trigger phrases:**
- "Run BCE Session E"
- "Run integration contracts extraction"
- "Create integration contracts"
- "Extract integration contracts"
- "Build integration contracts"

**Prerequisite:** All Sessions A0–D committed; all five Stage 2 artifacts in discovery/

```
You are running BCE Stage 2 Session E — Integration Contracts and Risk Register.

Read all five Stage 2 artifacts before starting:
- discovery/INTAKE_SUMMARY.md
- discovery/TOPOLOGY.md (A03 — External System Boundary Map is your primary source)
- discovery/MODULE_CONTRACTS.md (Known Fragility and Error Behaviour fields)
- discovery/INVARIANT_CATALOGUE.md (invariants that touch external systems)
- discovery/RISK_REGISTER.md (Stage 1 draft)

PART 1 — Produce INTEGRATION_CONTRACTS.md:

For each external system recorded in TOPOLOGY.md A03, produce one
integration contract record using the IP-NNN assigned in Session A:

**IP-NNN — [External System Name]**

Called by: M-NNN [, M-NNN ...] (from A03 Called-by fields)

What the application promises to send:
[exact fields, types, serialisation format — derived from A03 and MODULE_CONTRACTS]

What the application assumes it will receive:
[exact fields consumed, fields ignored — derived from A03 and MODULE_CONTRACTS]

Auth mechanism:
[exact implementation recorded in A03; NOT DETERMINABLE FROM SOURCE if absent]

Error handling assumptions:
[what the application assumes when the external system fails — from A03 and MODULE_CONTRACTS Error Behaviour]

Known divergences:
[where the promise and assumption do not match — flag each as a STAGE-2-DIVERGENCE if new]

Gaps:
[operational fragilities in this integration not already in RISK_REGISTER.md]

Save as discovery/INTEGRATION_CONTRACTS.md

PART 2 — Complete RISK_REGISTER.md:

Review Stage 1 RISK_REGISTER.md against the assembled Stage 2 artifacts.
Add any risks not already captured:
- Known fragilities from MODULE_CONTRACTS.md Known Fragility fields
- Error handling gaps surfaced by Sessions A–D
- Integration risks from the INTEGRATION_CONTRACTS.md Gaps fields

For each risk entry, reference affected modules by M-NNN and threatened
invariants by IC-N — not by prose name.

Update discovery/RISK_REGISTER.md.

For any field that cannot be determined from assembled artifacts:
NOT DETERMINABLE FROM SOURCE

For any field where Session E synthesis diverges from Stage 1 content:
[STAGE-2-DIVERGENCE — YYYY-MM-DD]: Stage 1 stated [X]. Session E shows [Y].
Resolution required before Stage 3.

After saving: state that Session E is complete and that both
INTEGRATION_CONTRACTS.md and RISK_REGISTER.md must be committed before
the Stage 2 Completeness Summary is produced.
```

---

### 11.7 Stage 2 Completeness Summary

**Tool:** CC
**Trigger phrases:**
- "Produce BCE Stage 2 summary"
- "Produce Stage 2 completeness summary"

**Prerequisite:** All Sessions A0–E committed

```
You are producing the BCE Stage 2 Completeness Summary.

Read all five Stage 2 artifacts from discovery/:
- discovery/TOPOLOGY.md
- discovery/MODULE_CONTRACTS.md
- discovery/INTEGRATION_CONTRACTS.md
- discovery/INVARIANT_CATALOGUE.md
- discovery/RISK_REGISTER.md

Count all NOT DETERMINABLE FROM SOURCE entries and all
STAGE-2-DIVERGENCE flags in each artifact.

Add the following section to discovery/TOPOLOGY.md below any existing
Stage 1 Completeness Summary (do NOT replace it):

## Stage 2 Completeness Summary — [date]
Produced by: BCE Stage 2 Session A0–E (CC/CD)

| Artifact | Status | NOT DETERMINABLE count | Divergences from Stage 1 | Notes |
|---|---|---|---|---|
| TOPOLOGY.md | COMPLETE | [n] | [n] | |
| MODULE_CONTRACTS.md | COMPLETE | [n] | [n] | |
| INTEGRATION_CONTRACTS.md | COMPLETE | [n] | N/A — Stage 2 only | |
| INVARIANT_CATALOGUE.md | COMPLETE | [n] | [n] | |
| RISK_REGISTER.md | COMPLETE | [n] | [n] | |

Stage-2-Divergence items requiring engineer resolution before Stage 3:
[list each STAGE-2-DIVERGENCE by artifact and field]

Human gate required before Stage 3 begins.
Engineer sign-off: [LEAVE BLANK]

After saving: state that Stage 2 is complete. Remind the engineer that all
STAGE-2-DIVERGENCE items must be resolved and engineer sign-off recorded
before "Run BCE Stage 3" is triggered.
```

---

### 11.8 Graph Construction

**Tool:** CC
**Trigger phrases:**
- "Build system graph"
- "Construct system graph"
- "Generate SYSTEM_GRAPH.json"

**Prerequisite:** Stage 3 schema validation gate passed (all P1 SCHEMA_VIOLATION items resolved). Do not run until Stage 3 confirms schema validation PASS.

```
You are producing discovery/SYSTEM_GRAPH.json — the machine-readable knowledge
graph for this system.

Read all five living artifacts and the module call map before producing any output:
- discovery/TOPOLOGY.md (A02 for CALLS edges; A03 for IntegrationPoint nodes)
- discovery/MODULE_CONTRACTS.md (Module nodes; Callers/Calls/Integration Points Used fields)
- discovery/INTEGRATION_CONTRACTS.md (CAN_VIOLATE edges; CALLS_INTEGRATION edges)
- discovery/INVARIANT_CATALOGUE.md (Invariant nodes; OWNS and ENFORCES edges)
- discovery/RISK_REGISTER.md (RiskItem nodes; AFFECTS and THREATENS edges)
- discovery/components/A02_module_call_map.md (Module Roster for canonical M-NNN names)

Also check: if discovery/DOMAIN_MODEL.json exists and is not sparse, read it for
cross-graph edge derivation (OWNS_ENTITY, READS_ENTITY, ENFORCES_ENTITY_INVARIANT,
ENFORCES_ATTRIBUTE_INVARIANT). If DOMAIN_MODEL.json does not exist or is sparse,
omit cross-graph edges.

All relationships must use ID references (M-NNN, IP-NNN, IC-N, R-NNN, E-NNN, A-NNN).
Do not infer relationships not explicitly recorded in the artifacts.

Produce discovery/SYSTEM_GRAPH.json with this structure:

{
  "system": "[system name from INTAKE_SUMMARY.md]",
  "generated": "YYYY-MM-DD",
  "bce_version": "v2.1",
  "nodes": [
    { "id": "M-NNN", "type": "Module", "name": "[module name]", "layer": "[serving|pipeline|infra]" },
    { "id": "IP-NNN", "type": "IntegrationPoint", "name": "[external system name]" },
    { "id": "IC-N", "type": "Invariant", "statement": "[invariant text]", "scope": "[GLOBAL|TASK-SCOPED]" },
    { "id": "R-NNN", "type": "RiskItem", "description": "[risk description]", "severity": "[P1|P2|P3]" }
  ],
  "edges": [
    { "from": "M-NNN", "to": "M-NNN", "type": "CALLS" },
    { "from": "M-NNN", "to": "IC-N", "type": "OWNS" },
    { "from": "M-NNN", "to": "IC-N", "type": "ENFORCES" },
    { "from": "M-NNN", "to": "IP-NNN", "type": "CALLS_INTEGRATION" },
    { "from": "IP-NNN", "to": "IC-N", "type": "CAN_VIOLATE" },
    { "from": "R-NNN", "to": "M-NNN", "type": "AFFECTS" },
    { "from": "R-NNN", "to": "IC-N", "type": "THREATENS" },
    { "from": "M-NNN", "to": "E-NNN", "type": "OWNS_ENTITY" },
    { "from": "M-NNN", "to": "E-NNN", "type": "READS_ENTITY" },
    { "from": "IC-N", "to": "E-NNN", "type": "ENFORCES_ENTITY_INVARIANT" },
    { "from": "IC-N", "to": "A-NNN", "type": "ENFORCES_ATTRIBUTE_INVARIANT" }
  ]
}

Edge derivation rules:
- CALLS: from A02 Internal Call Table typed edges
- OWNS: from INVARIANT_CATALOGUE.md Owning module field
- ENFORCES: from INVARIANT_CATALOGUE.md Enforcing modules field (one edge per module)
- CALLS_INTEGRATION: from MODULE_CONTRACTS.md Integration Points Used field (one edge per IP-NNN)
- CAN_VIOLATE: from INTEGRATION_CONTRACTS.md Known divergences and Gaps fields — for each
  gap that threatens a named invariant, emit one CAN_VIOLATE edge
- AFFECTS: from RISK_REGISTER.md affected module references
- THREATENS: from RISK_REGISTER.md threatened invariant references
- OWNS_ENTITY: from DOMAIN_MODEL.json entity cross_graph_edges annotations (type OWNS_ENTITY),
  or from MODULE_CONTRACTS.md where entity ownership is stated; one edge per entity owned
- READS_ENTITY: from DOMAIN_MODEL.json entity cross_graph_edges annotations (type READS_ENTITY),
  or from MODULE_CONTRACTS.md where entity reads are stated; one edge per entity read
- ENFORCES_ENTITY_INVARIANT: from INVARIANT_CATALOGUE.md where invariant scope names an entity
  (E-NNN reference); one edge per entity-scoped invariant
- ENFORCES_ATTRIBUTE_INVARIANT: from INVARIANT_CATALOGUE.md where invariant constrains a
  specific attribute (A-NNN reference); one edge per attribute-scoped invariant

Cross-graph edges are only emitted when DOMAIN_MODEL.json exists and is not sparse.
If DOMAIN_MODEL.json is sparse or absent, omit all four cross-graph edge types.

After saving: state that SYSTEM_GRAPH.json is complete and committed to discovery/.
Report total node count by type and total edge count by type.
If cross-graph edges were emitted: also report count by cross-graph edge type.
```

---

## 12. Stage 3 — Cross-Artifact Review (CD)

Stage 3 runs in **CD** after all six artifacts are committed and Stage 2 engineer sign-off
is complete. CD is used for Stage 3 because it is the reasoning and challenge tool —
Stage 3 requires synthesis across artifacts and judgment about severity, not code reading.

Stage 3 produces ANNOTATION_CHECKLIST.md — the BCE backlog.

### Stage 3 Execution Prompt

**Tool:** CD
**Trigger phrases:**
- "Run BCE Stage 3"
- "Run cross-artifact review"
- "Produce ANNOTATION_CHECKLIST.md"

```
You are running BCE Adapter Pipeline Stage 3 — the cross-artifact CD review.

All six BCE artifacts are committed to discovery/. Read all of them before
producing any output:
- discovery/INTAKE_SUMMARY.md
- discovery/TOPOLOGY.md
- discovery/MODULE_CONTRACTS.md
- discovery/INTEGRATION_CONTRACTS.md
- discovery/INVARIANT_CATALOGUE.md
- discovery/RISK_REGISTER.md

Also read:
- docs/INVARIANTS.md (to cross-check INVARIANT_CATALOGUE.md)
- docs/ARCHITECTURE.md (to cross-check TOPOLOGY.md and MODULE_CONTRACTS.md)

Produce discovery/ANNOTATION_CHECKLIST.md using the canonical item format
(severity + type + source + surfaced-by fields per item).

Complete these checks in order. For each finding, produce one checklist item.

---

CHECK 0 — Schema Validation Gate (mandatory — must pass before any other check proceeds)

Verify that all relationship fields in the five artifacts use ID references
rather than prose names. Check each of the following:

- TOPOLOGY.md A02 Module Roster: M-NNN IDs assigned to all modules
- TOPOLOGY.md A02 Internal Call Table: all entries use M-NNN --[CALLS]--> M-NNN format
- TOPOLOGY.md A03: each external system record has an IP-NNN field
- MODULE_CONTRACTS.md: Callers, Calls, and Integration Points Used fields reference
  M-NNN and IP-NNN (not prose names)
- INVARIANT_CATALOGUE.md: Owning module and Enforcing modules fields reference M-NNN
- INTEGRATION_CONTRACTS.md: each record starts with IP-NNN; Called by fields reference M-NNN
- RISK_REGISTER.md: affected module fields reference M-NNN; threatened invariant fields reference IC-N

If discovery/DOMAIN_MODEL.json exists and is not sparse, also check:
- All Entity node IDs use E-NNN format
- All Attribute node IDs use A-NNN format
- All StatusVocabulary node IDs use SV-NNN format
- All StatusValue node IDs use SVV-NNN format
- All Relationship node IDs use REL-NNN format
- All cross-graph edges in SYSTEM_GRAPH.json (OWNS_ENTITY, READS_ENTITY,
  ENFORCES_ENTITY_INVARIANT, ENFORCES_ATTRIBUTE_INVARIANT) reference E-NNN or
  A-NNN IDs that exist in DOMAIN_MODEL.json

For each ID format violation or dangling cross-graph edge reference: produce one P1
checklist item with Type: SCHEMA_VIOLATION.

For each field using a prose name where an ID is expected: produce one P1 checklist
item with Type: SCHEMA_VIOLATION.

If any SCHEMA_VIOLATION items are produced: stop. Stage 3 does not proceed past this
check. State the violations found and instruct the engineer to resolve them (assign
the correct IDs) before re-running Stage 3.

If all checks pass: state "Schema validation PASS — all relationship fields use ID
references" and proceed to CHECK 1.

---

CHECK 1 — Cross-artifact contradiction detection

For each artifact pair:
- TOPOLOGY.md vs MODULE_CONTRACTS.md: module names, call relationships, layer assignments
- TOPOLOGY.md vs INTEGRATION_CONTRACTS.md: external system names, boundary descriptions
- MODULE_CONTRACTS.md vs INVARIANT_CATALOGUE.md: enforcement points, module sources
- INVARIANT_CATALOGUE.md vs docs/INVARIANTS.md: invariant statements, coverage
- RISK_REGISTER.md vs MODULE_CONTRACTS.md: Known Fragility fields vs risk entries

For each contradiction: produce a checklist item with Type: CONTRADICTION.
Default severity: P1.

---

CHECK 2 — NOT DETERMINABLE escalation

Review all NOT DETERMINABLE FROM SOURCE entries remaining across all artifacts.
For each:
- If it affects a known planning decision or invariant: severity P1
- If it is needed for a complete module contract: severity P2
- If informational only: severity P3

Produce one checklist item per unresolved NOT DETERMINABLE entry.

---

CHECK 3 — Stage-2-Divergence resolution check

For each STAGE-2-DIVERGENCE flag in any artifact:
- Has the engineer resolved it? If not: produce a P1 checklist item.
- If resolved: note in the resolution log.

---

CHECK 4 — Missing invariant candidates

Compare MODULE_CONTRACTS.md Known Fragility and Change Impact fields against
INVARIANT_CATALOGUE.md. For each fragility with no corresponding invariant:
produce a P2 checklist item recommending invariant candidate for engineer review.

---

CHECK 5 — Risk register severity review

Review each RISK_REGISTER.md entry. For any entry where severity appears
inconsistent with the fragility description or invariant impact: flag as
P3 checklist item with recommended severity adjustment.

---

CHECK 6 — Cross-artifact consistency check

Produce the mandatory consistency check table:

## Cross-Artifact Consistency Check
Last run: [date] By: CD

| Check | Status | Notes |
|---|---|---|
| All invariants in INVARIANT_CATALOGUE.md match INVARIANTS.md | PASS / FAIL | |
| All module names in MODULE_CONTRACTS.md match TOPOLOGY.md | PASS / FAIL | |
| All external systems in INTEGRATION_CONTRACTS.md match TOPOLOGY.md A03 | PASS / FAIL | |
| All risks in RISK_REGISTER.md reference correct source artifacts | PASS / FAIL | |
| INTAKE_SUMMARY.md open questions accounted for in artifacts | PASS / FAIL | |
| Entity names in DOMAIN_MODEL.json consistent with domain terminology in INVARIANT_CATALOGUE.md and MODULE_CONTRACTS.md (skip if DOMAIN_MODEL.json absent or sparse) | PASS / FAIL / N/A | |

---

After all checks complete, produce a Stage 3 Completeness Summary:

## Stage 3 Completeness Summary — [date]
Produced by: BCE Adapter Pipeline Stage 3 (CD)

P1 items: [count] — must be SIGNED-OFF or RESOLVED before Stage 3 completes
P2 items: [count] — tracked, resolve before first enhancement
P3 items: [count] — informational backlog
CON items: [count] — contradictions (default P1)
Total items: [count]

Consistency check: PASS / FAIL — [n] failures

Stage 3 is complete when all P1 items are SIGNED-OFF or RESOLVED by the engineer.

**Stage 3 Close-Out Step — Graph Construction (mandatory, all paths)**
After all P1 items are SIGNED-OFF or RESOLVED: run the graph construction prompt in CC (Section 11.8). Trigger phrase: "Build system graph". Output: `discovery/SYSTEM_GRAPH.json`. This step is not optional and must not be deferred. Stage 3 is not complete until SYSTEM_GRAPH.json is committed to discovery/. If DOMAIN_MODEL.json exists and is non-sparse, cross-graph edges (OWNS_ENTITY, READS_ENTITY, ENFORCES_ENTITY_INVARIANT, ENFORCES_ATTRIBUTE_INVARIANT) are included automatically.
```

**Engineer review after Stage 3:**
Read every P1 item. For each: sign off (understood and accepted) or resolve (fix the
artifact and mark resolved). P1 items that cannot be resolved immediately become
sprint planning blockers. Record sign-offs directly on each item in ANNOTATION_CHECKLIST.md.
To apply resolutions, use the Annotation Amendment prompt — Section 14.3.

---

## 13. Storage and Maintenance

### discovery/ Is the Source of Truth

All artifacts live in `discovery/` in the repository. CD project files are a convenience
layer. When an artifact is updated in the repository, the corresponding CD project file
must be updated — stale project files produce stale context.

In sprint mode (the standard PBVI path), artifacts are updated once per sprint at
close-out — not after each individual enhancement. The Sprint Lead runs the BCE refresh
at sprint close-out using all ENH-NNN_BCE_IMPACT.md logs for the sprint. The CD project
file update happens at sprint close-out after the single sprint close-out commit.

### Upload Pattern

Upload each artifact to CD project files after it is committed to `discovery/`. For
component files: upload after annotation and before assembly, then replace with the
assembled artifact after assembly is complete.

### When to Update Artifacts

| Change made to the system | Artifacts to update |
|---|---|
| Module inputs, outputs, or error behaviour changes | MODULE_CONTRACTS.md |
| External dependency added or removed | TOPOLOGY.md, INTEGRATION_CONTRACTS.md |
| New module introduced or existing module deleted | CODEBASE_MAP.md first, then MODULE_CONTRACTS.md, TOPOLOGY.md |
| File added, moved, or removed in repository | CODEBASE_MAP.md |
| Known invariant violated or enforcement changes | INVARIANT_CATALOGUE.md |
| Risk register item resolved | RISK_REGISTER.md |
| NOT DETERMINABLE entry resolved through operation | Relevant artifact |
| Enhancement close-out (sprint mode — standard PBVI path) | BCE artifacts in `discovery/` are frozen during the sprint. The building engineer produces ENH-NNN_BCE_IMPACT.md at Phase 8 Part 2B close-out — `discovery/` is NOT updated at this point. Sprint Lead runs the BCE refresh at sprint close-out using all ENH-NNN_BCE_IMPACT.md logs for the sprint. Where two impact logs touch the same artifact section, Sprint Lead resolves the conflict. Single sprint close-out commit to `discovery/`. CD project files updated at sprint close-out. See PBVI sprint extension Section F for full procedure and Prompt 5 for the CC execution prompt. Never INTAKE_SUMMARY.md — it is a point-in-time record and must not be updated retroactively. |
| Enhancement close-out (single-enhancement sprint) | Same path as sprint mode. Building engineer produces ENH-NNN_BCE_IMPACT.md. Sprint Lead = building engineer in the solo case — same person runs Phase 8 Part 2B then the sprint BCE refresh immediately after. Single sprint close-out commit. CD project files updated. Never INTAKE_SUMMARY.md. |

### Context Health Signals

A context package is degrading when:
- ENGINEER NOTE annotations reference behaviour that has since changed
- NOT DETERMINABLE entries the team now knows have not been resolved
- Module contracts reference function names, fields, or paths that no longer exist
- RISK_REGISTER.md contains resolved items not marked resolved

A quarterly context health review is recommended for all actively maintained systems.

---

## 14. BCE Close-Out Procedures

This section is the authoritative source for both BCE close-out steps. The PBVI skill
cross-references both subsections. Load the BCE skill as a project file when running
Phase 8 Part 2B.

### 14.1 — BCE Gap Detection Pass

**Tool:** CC
**Trigger phrases:**
- "Run BCE gap detection for ENH-NNN"
- "Check BCE gaps for ENH-NNN"
- "Run gap detection"

Run this prompt in CC (Step 1 of Phase 8 Part 2B). Re-run until all tables are empty.
Do not proceed to Step 2 until gap detection is CLEAN.

```
Read all Verification Records in sessions/ for ENH-[NNN].
Read all six BCE artifacts in discovery/.
Read enhancements/SPRINT-NNN/ENH-[NNN]-[slug]/ENH-[NNN]_EXECUTION_PLAN.md.

For each task in the enhancement execution plan:
1. Check the BCE Impact section in the corresponding Verification Record entry.
2. Cross-reference against discovery/ artifacts — identify any module, external
   system, invariant, or risk register entry that this task touched but has no
   corresponding BCE Impact recorded.

Produce a gap detection report grouped by BCE artifact:

### TOPOLOGY.md
| Task ID | Task Name | Surface Touched | Gap Description |
|---|---|---|---|

### MODULE_CONTRACTS.md
| Task ID | Task Name | Module Touched | Gap Description |
|---|---|---|---|

### INTEGRATION_CONTRACTS.md
| Task ID | Task Name | External System Touched | Gap Description |
|---|---|---|---|

### INVARIANT_CATALOGUE.md
| Task ID | Task Name | Invariant Touched | Gap Description |
|---|---|---|---|

### RISK_REGISTER.md
| Task ID | Task Name | Risk Item Touched | Gap Description |
|---|---|---|---|

### ANNOTATION_CHECKLIST.md
| Task ID | Task Name | Checklist Item Touched | Gap Description |
|---|---|---|---|

Empty table = no gaps for that artifact.
Populated table = gaps the engineer must resolve before BCE close-out proceeds.

Do not produce ENH-NNN_BCE_IMPACT.md until the engineer confirms all gaps
are resolved.
```

### 14.2 — ENH-NNN_BCE_IMPACT.md Template

**When to generate:** Phase 8 Part 2B close-out, in CC after gap detection is CLEAN.
**Inputs CC needs:** All Verification Records for the enhancement (sessions/), all six
BCE artifacts in discovery/, the enhancement execution plan. If discovery/DOMAIN_MODEL.json
exists, also read it for Section 7 (Domain Model Impact).
**Leave blank:** All engineer sign-off fields — engineer completes after CC produces the draft.

### CC Execution Prompt

**Tool:** CC
**Trigger phrases:**
- "Produce BCE impact log for ENH-NNN"
- "Generate ENH-NNN_BCE_IMPACT.md"
- "Run BCE close-out for ENH-NNN"

```
All BCE gap detection tables are empty. Proceed to produce
enhancements/SPRINT-NNN/ENH-[NNN]-[slug]/ENH-[NNN]_BCE_IMPACT.md.

Read:
- All Verification Records in sessions/ for this enhancement
- All six BCE artifacts in discovery/
- discovery/DOMAIN_MODEL.json (if present — for Section 7)
- The gap detection report (confirmed CLEAN)

Produce ENH-[NNN]_BCE_IMPACT.md using the template structure below.

Pre-populate all AFFECTED/NOT AFFECTED status fields and all Before/After
state fields from the Verification Records. Leave all engineer sign-off
fields blank. Leave confidence ratings blank where the Verification Records
do not contain enough information — flag these for engineer judgment.
For BCE artifact field definitions, refer to the BCE skill.
```

### Template

```markdown
# ENH-[NNN]_BCE_IMPACT.md

## Header
Enhancement: ENH-[NNN] — [slug]
Engineer: [LEAVE BLANK — engineer fills]
Phase 8 sign-off date: [LEAVE BLANK]
BCE close-out date: [LEAVE BLANK]
Sprint: [LEAVE BLANK — SPRINT-NNN]

## BCE Gap Detection
Status: CLEAN
[Paste confirmed clean gap detection tables here]

## 1. TOPOLOGY.md Impact
Status: AFFECTED | NOT AFFECTED

[If AFFECTED — for each changed layer:]

### Layer Boundary Map
Status: AFFECTED | NOT AFFECTED
| Boundary | Change Type | Before State | After State | Operational Notes |
|---|---|---|---|---|
Change Type: ADDED | MODIFIED | REMOVED

### Module Call Map
Status: AFFECTED | NOT AFFECTED
| Caller | Callee | Change Type | Before State | After State | Operational Notes |
|---|---|---|---|---|

### External System Boundary Map
Status: AFFECTED | NOT AFFECTED
| External System | Change Type | Before State | After State | Operational Notes |
|---|---|---|---|---|

Confidence: HIGH | MEDIUM | LOW
Notes: [if MEDIUM or LOW]

## 2. MODULE_CONTRACTS.md Impact
Status: AFFECTED | NOT AFFECTED

[If AFFECTED — one block per touched module:]

### [Module Name — exact name as in MODULE_CONTRACTS.md]
Change Type: ADDED | MODIFIED | REMOVED
| Field | Before State | After State | Operational Notes |
|---|---|---|---|
| Inputs | | | |
| Outputs | | | |
| Error Behaviour | | | |
| Dependencies | | | |
| Invariants Enforced | | | |
| Startup Behaviour | | | |
| Async Boundaries | | | |
Confidence: HIGH | MEDIUM | LOW
Notes: [if MEDIUM or LOW]

## 3. INTEGRATION_CONTRACTS.md Impact
Status: AFFECTED | NOT AFFECTED

[If AFFECTED — one block per affected external system:]

### [External System Name — exact name as in INTEGRATION_CONTRACTS.md]
Change Type: ADDED | MODIFIED | REMOVED
| Field | Before State | After State | Operational Notes |
|---|---|---|---|
| What application sends | | | |
| What application expects to receive | | | |
| Auth mechanism | | | |
| Error handling assumptions | | | |
| Known divergences | | | |
Confidence: HIGH | MEDIUM | LOW
Notes: [if MEDIUM or LOW]

## 4. INVARIANT_CATALOGUE.md Impact
Status: AFFECTED | NOT AFFECTED

[If AFFECTED:]

### Existing Invariants — Changes Only
| Invariant ID | Change Type | Before State | After State |
|---|---|---|---|
Operational Notes: [per invariant if needed]
Confidence: HIGH | MEDIUM | LOW per invariant

### New Invariants Discovered During Build
| Proposed ID | Statement | Category | Why It Matters |
|---|---|---|---|
| Enforcement Points | Currently Enforced | Source Module |
|---|---|---|
Note: New invariants are candidates pending senior review before
addition to INVARIANT_CATALOGUE.md.

## 5. RISK_REGISTER.md Impact
Status: AFFECTED | NOT AFFECTED

[If AFFECTED:]

### Risks Resolved By This Enhancement
| Risk ID | Resolution Description | Evidence |
|---|---|---|
Evidence: VERIFIED_IN_PHASE8 | BUILD_OBSERVATION | VERIFICATION_RECORD

### Risks Introduced By This Enhancement
| Proposed ID | Description | Severity | Affected Module/Artifact |
|---|---|---|---|
| Mitigation | Recommended Action |
|---|---|

### Existing Risks — Severity Changed
| Risk ID | Previous Severity | New Severity | Reason |
|---|---|---|---|

### Existing Risks — Confirmed Unmitigated
| Risk ID | Confirmation Note |
|---|---|
Confidence: HIGH | MEDIUM | LOW
Notes: [if MEDIUM or LOW]

## 6. ANNOTATION_CHECKLIST.md Impact
Status: AFFECTED | NOT AFFECTED

[If AFFECTED:]

### Existing Items — Resolved During This Enhancement
| Item ID | Resolution | Evidence Source |
|---|---|---|
Evidence Source: BUILD_OBSERVATION | VERIFICATION_RECORD | PHASE8_CHECKLIST

### Existing Items — Escalated During This Enhancement
| Item ID | Original Severity | New Severity | Reason for Escalation |
|---|---|---|---|

### New Items Surfaced During This Enhancement
| Proposed ID | Type | Description | Severity |
|---|---|---|---|
Type: NOT_DETERMINABLE | CONTRADICTION | OPEN_QUESTION

Confidence: HIGH | MEDIUM | LOW

## 7. DOMAIN_MODEL.json Impact
Status: AFFECTED | NOT AFFECTED | NOT APPLICABLE (Session F did not run for this system)

[If AFFECTED:]

**Entities affected:** [E-NNN list or NONE]
**Attributes affected:** [A-NNN list or NONE]
**Vocabulary changes:** [SV-NNN / SVV-NNN list or NONE]
**Relationship changes:** [REL-NNN list or NONE]
**New entities introduced:** [list or NONE]
**Cross-graph edges affected:** [M-NNN/IC-N → E-NNN/A-NNN list or NONE]

| Node ID | Change Type | Before State | After State | Operational Notes |
|---|---|---|---|---|
Change Type: ADDED | MODIFIED | REMOVED

Confidence: HIGH | MEDIUM | LOW
Notes: [if MEDIUM or LOW]

## Confidence Rating Guide

HIGH: Derived entirely from Verification Records and planning artifacts. No additional
engineer input needed beyond confirmation of accuracy.

MEDIUM: Best-effort derivation — CC is missing information only the building engineer
holds. An operational note is mandatory for every MEDIUM section. The note must
answer: what is CC uncertain about, and what is the correct state?

LOW: Structural placeholder — CC could not derive this from available inputs. The
engineer must supply the content from memory. A LOW section with no operational note
is incomplete, not conservative.

An impact log submitted for sign-off with MEDIUM or LOW sections and no operational
notes must be returned for completion before it is accepted.

## Engineer Sign-Off
[LEAVE ALL BLANK — engineer completes]

Enhancement: ENH-NNN — [slug]
Engineer:
Phase 8 sign-off date:
BCE close-out date:
Sprint: SPRINT-NNN

### BCE Gap Detection Gate
[ ] All gap detection tables empty — no unrecorded BCE impacts remaining
[ ] All Verification Record BCE Impact sections complete and accurate

### Impact Coverage Confirmation
[ ] TOPOLOGY.md — reviewed, status recorded
[ ] MODULE_CONTRACTS.md — reviewed, status recorded
[ ] INTEGRATION_CONTRACTS.md — reviewed, status recorded
[ ] INVARIANT_CATALOGUE.md — reviewed, status recorded
[ ] RISK_REGISTER.md — reviewed, status recorded
[ ] ANNOTATION_CHECKLIST.md — reviewed, status recorded
[ ] DOMAIN_MODEL.json — reviewed, status recorded (or marked NOT APPLICABLE)

### Tacit Knowledge Capture
[ ] For every MEDIUM confidence section: operational note present, answers what CC
    could not derive and what the correct state is
[ ] For every LOW confidence section: engineer-supplied content present, not a
    structural placeholder
[ ] For HIGH confidence sections with no operational notes: confirmed this reflects
    a genuinely clean derivation from Verification Records, not a skipped review

### Certification
I certify that this impact log accurately reflects what this enhancement
changed in the system, to the best of my knowledge at the time of
Phase 8 sign-off. Gaps in knowledge are recorded with MEDIUM or LOW
confidence ratings and operational notes explaining the uncertainty.

Signature:
Date:
```

### Engineer Review — Two Parts, Both Required

**Part A — Structural verification (what CC produced):**
Read every AFFECTED section. Confirm Before/After states are accurate against your
memory of the build. Correct any state that CC mis-derived. This part can be fast
if the Verification Records were complete.

**Part B — Tacit knowledge capture (what only you know):**
For each AFFECTED section, ask: "What do I know about this change that CC could not
have inferred from the Verification Records alone?" Work through these prompts:
- What did I consider and reject during this session that affected how this module ended up?
- What fragility did I notice that was not formally tested?
- What would the next engineer get wrong about this change if they only read the Verification Record?
- For each MEDIUM or LOW confidence section: what is CC uncertain about, and what is the correct answer?

Record findings as operational notes on the relevant AFFECTED section. If there are
no findings after working through these prompts, write "No additional tacit knowledge —
HIGH confidence state is complete." Do not leave operational notes blank on MEDIUM or
LOW confidence sections.

Sign off after both parts are complete.

---

### 14.3 — Annotation Amendment

**When to run:** After ANNOTATION_CHECKLIST.md is produced (greenfield — after Stage 3)
or updated (sprint close-out — after BCE refresh). Run once the engineer has reviewed
items and has operational knowledge to supply.

P1 items: run immediately after close-out. Use trigger phrase "Resolve P1 annotation items".
P2/P3 items: engineer-driven at any point. Provide specific item IDs and raw annotations.

**Tool:** CC
**Trigger phrases:**
- "Resolve P1 annotation items"
- "Resolve annotation items [ITEM_IDs]"
- "Apply engineer annotation for [ITEM_IDs]"

**Engineer provides before running:**
For each item to resolve: item ID and raw annotation in plain language.

```
You are resolving ANNOTATION_CHECKLIST.md items by applying engineer-supplied
annotations directly to BCE artifacts.

Engineer provides for each item:
- ITEM_ID: [e.g. P1-001, P2-003]
- RAW_ANNOTATION: [engineer's operational knowledge or correction in plain language]

Scope rule: only process items with the severity of the item IDs provided.
Do not expand scope to other severity levels.

If any ITEM_ID is not found in ANNOTATION_CHECKLIST.md, stop and report before
proceeding.

For each item in the order provided:

STEP 1 — Read the item from ANNOTATION_CHECKLIST.md.
Identify: the BCE artifact it refers to, the specific field or section to amend,
and the item type (NOT_DETERMINABLE | CONTRADICTION | OPEN_QUESTION).

STEP 2 — Read the relevant section of the BCE artifact.
Confirm the exact location where the ENGINEER NOTE will be applied.

STEP 3 — Format and apply the amendment.

Format:
[ENGINEER NOTE — YYYY-MM-DD]: [structured summary of RAW_ANNOTATION — one to three
sentences. Preserve the engineer's meaning precisely. Do not paraphrase in a way
that loses specificity.]

Placement rules by type:
- NOT_DETERMINABLE: replace "NOT DETERMINABLE FROM SOURCE" with the resolved value,
  then append the ENGINEER NOTE on the next line.
- CONTRADICTION: add the ENGINEER NOTE to both conflicting entries, stating the
  confirmed answer and which source was correct.
- OPEN_QUESTION: append the ENGINEER NOTE immediately after the question text.

STEP 4 — Update ANNOTATION_CHECKLIST.md item:
- Status: RESOLVED
- Resolved: [YYYY-MM-DD]
- Resolution: [one-line summary of what was supplied]
- Source: HUMAN_ANNOTATION

STEP 5 — Produce Human Annotation Record.
Assign the next available HAR-NNN ID for this project.
Save to:
- discovery/annotations/HAR-NNN_[short-title].md (greenfield — Stage 1–3 context)
- enhancements/ENH-NNN/HAR-NNN_[short-title].md (enhancement close-out context)
If discovery/annotations/ does not exist, create it and register the directory
in PROJECT_MANIFEST.md before saving the HAR.
Populate all fields from the work done in Steps 1–4: checklist item reference,
engineer name, raw annotation verbatim, artifact and field updated, ENGINEER NOTE
as applied, and the Resolution Log row.
Use the template from Section 8 (Human Annotation Record).

[HUMAN GATE] — Output the following before proceeding to the next item:
- Artifact amended and field location
- ENGINEER NOTE as applied
- Checklist item status: RESOLVED
- HAR saved at: [path]
Engineer reviews and confirms before the next item begins.

After all items are resolved — STEP 6 — Commit:
git add discovery/ enhancements/
git commit -m "BCE annotation: [list all ITEM_IDs resolved] — engineer amendments applied; HAR-NNN through HAR-NNN produced"
```

---

## 15. Quick Reference

**NOT DETERMINABLE is honest** — invented answers are liabilities. Gaps are tracked, not
deleted. The gap tells the next engineer exactly where to investigate.

**Never assemble with an unresolved contradiction** — resolve first, then assemble.

**Annotation = operational knowledge externalised.** CC extracts structure. Engineers supply
operational knowledge. CD challenges both.

**Stage 2 adds its own Completeness Summary** — it never updates Stage 1's summary in-place
(SC-001). Contradiction resolutions go in the Resolution Log, not the Completeness Summary
(SC-002).

**Path B fills progressively** — a context package with honest gaps beats no context package.

**PBVI adapter pipeline — tool assignment:**
- Stage 1: CC reads docs/, produces draft artifacts. Not CD.
- Stage 2: CC reads source code, completes artifacts, produces INTEGRATION_CONTRACTS.md.
- Stage 3: CD runs cross-artifact review, produces ANNOTATION_CHECKLIST.md.

**ANNOTATION_CHECKLIST.md is the BCE backlog** — severity drives priority (P1/P2/P3),
type describes the gap kind (CONTRADICTION/NOT_DETERMINABLE/OPEN_QUESTION).
Default: CONTRADICTION → P1. Override requires stated rationale.

**Stage prompts are embedded in this skill** — Sections 10, 11, 12. Do not create
per-project prompt files in docs/prompts/.

**Annotation Amendment prompt — Section 14.3.** Run after Stage 3 (greenfield) or sprint
BCE refresh to resolve checklist items. P1: "Resolve P1 annotation items". P2/P3:
"Resolve annotation items [ITEM_IDs]". Human gate after each item — engineer reviews
before the next begins.

**SYSTEM_GRAPH.json** — produced at Stage 3 close-out (Section 11.8). Regenerated at
every sprint close-out. Use the subgraph query (Section 16) to extract planning context
for a specific module rather than loading all five full artifact files into CD.

**DOMAIN_MODEL.json** — produced by Session F (Section 11.3). Conditional: only for
systems with a detectable data layer (ORM definitions, migrations, DDL). Session F runs
after Session A and before Sessions B and C. Bridges to SYSTEM_GRAPH.json via shared ID
namespace (E-NNN, A-NNN in DOMAIN_MODEL.json; cross-graph edges in SYSTEM_GRAPH.json).

**Session F trigger condition** — ORM definitions, migration scripts, or DDL present.
If absent: Session F produces a sparse stub. Both are valid BCE outputs.

**ID_REGISTRY.md** — produced at Session A (M-NNN, IP-NNN), extended at Session F03
(E-NNN, A-NNN, SV-NNN, SVV-NNN, REL-NNN). Single coordination point guaranteeing no
ID collisions across separate extraction sessions.

---

## 16. BCE Subgraph Context Query

Use this when planning an enhancement on a system with BCE artifacts. Instead of loading
all five artifact files into CD, run this CC prompt first to produce a compact subgraph
for the module in scope. Paste the output into the PBVI Phase 1 Interrogate prompt
alongside the requirements brief.

**Tool:** CC
**Trigger phrases:**
- "Query BCE subgraph for M-NNN"
- "Get planning context for M-NNN"
- "Extract subgraph for M-NNN"

**Prerequisite:** discovery/SYSTEM_GRAPH.json exists (Stage 3 close-out complete)

```
You are producing a BCE planning subgraph for module [M-NNN].

Read discovery/SYSTEM_GRAPH.json.

Starting from node M-NNN, collect:
1. All nodes directly connected to M-NNN by any edge (1 hop)
2. All nodes connected to those 1-hop nodes (2 hops) — stop at 2 hops unless
   the engineer specifies a different depth
3. All Invariant nodes reachable from any collected Module node via OWNS or
   ENFORCES edges — regardless of hop count
4. All IntegrationPoint nodes reachable from any collected Module node via
   CALLS_INTEGRATION edges — regardless of hop count
5. All edges between collected nodes

Output a planning context fragment in this format:

## BCE Planning Subgraph — [M-NNN: module name]
Generated: YYYY-MM-DD | Depth: 2 hops | System: [system name]

### Modules in scope
| ID | Name | Layer | Relationship to entry point |
|---|---|---|---|
| M-NNN | [entry module] | | Entry point |
| M-NNN | [name] | | Calls entry / Called by entry / 2-hop |

### Governing invariants
| IC-N | Statement | Scope | Owning module | Enforcing modules |
|---|---|---|---|---|

### Integration points called
| IP-NNN | External System | Called by | Known gaps |
|---|---|---|---|

### Risks affecting these modules
| R-NNN | Description | Severity | Affected modules | Threatened invariants |
|---|---|---|---|---|

Paste this output into the PBVI Phase 1 Interrogate prompt as the
[BCE SUBGRAPH CONTEXT] section.
```

---

## 17. BCE Schema Migration — v1.x to v2.0

Use this when upgrading an existing extraction from BCE Skill v1.x (prose names in
relationship fields) to v2.0 (ID schema). Automates the migration steps in
BREAKING_CHANGES.md. Run in CC against the repository holding the existing artifacts.

**Tool:** CC
**Trigger phrases:**
- "Run BCE schema migration"
- "Migrate BCE artifacts to v2.0"
- "Assign BCE IDs to existing artifacts"

**Prerequisite:** All five living artifacts committed to discovery/. Source files
accessible (needed only to confirm module identity where MODULE_CONTRACTS.md is
ambiguous — not for re-extraction).

**Human gate:** The prompt pauses after Phase 1 (ID mapping) and does not update
any artifact until the engineer confirms the mapping is correct.

```
You are migrating BCE artifacts from v1.x (prose names in relationship fields)
to v2.0 (M-NNN and IP-NNN ID schema).

Read all of the following before producing any output:
- discovery/MODULE_CONTRACTS.md
- discovery/TOPOLOGY.md
- discovery/INTEGRATION_CONTRACTS.md
- discovery/INVARIANT_CATALOGUE.md
- discovery/RISK_REGISTER.md
- discovery/components/A02_module_call_map.md (if present)
- discovery/components/ (any component files — for module identity confirmation)

---

PHASE 1 — Discover and assign IDs

Step 1 — Enumerate modules:
From MODULE_CONTRACTS.md, list every module in the order it appears.
Assign M-001, M-002, M-003, ... sequentially.

Step 2 — Enumerate integration points:
From TOPOLOGY.md A03 (External System Boundary Map) and INTEGRATION_CONTRACTS.md,
list every distinct external system. Assign IP-001, IP-002, IP-003, ... sequentially.
If the same system appears under slightly different names in the two artifacts,
treat them as one record — use the INTEGRATION_CONTRACTS.md name as canonical.

Step 3 — Enumerate invariants (confirm existing IDs):
From INVARIANT_CATALOGUE.md, list every IC-N record. These IDs already exist —
do not reassign. Confirm the list is complete.

Step 4 — Enumerate risks (confirm existing IDs):
From RISK_REGISTER.md, list every R-NNN record. These IDs already exist —
do not reassign. Confirm the list is complete.

Produce the following mapping report — do not update any file yet:

## BCE Schema Migration — ID Mapping
Generated: YYYY-MM-DD

### Module IDs (new — assigned for migration)
| M-NNN | Module Name | Source as found in MODULE_CONTRACTS.md |
|---|---|---|

### Integration Point IDs (new — assigned for migration)
| IP-NNN | External System Name | Source as found in artifacts |
|---|---|---|

### Invariant IDs (existing — confirmed)
| IC-N | Statement (first 60 characters) |
|---|---|

### Risk IDs (existing — confirmed)
| R-NNN | Description (first 60 characters) |
|---|---|

[HUMAN GATE]
Output the following before proceeding:

"Phase 1 complete — ID mapping produced above.

Before I update any artifact, please confirm:
1. Every module in MODULE_CONTRACTS.md is listed with the correct ID
2. Every external system is listed with the correct IP-NNN
3. No modules or external systems are missing

If any are wrong (wrong name, missing entry, duplicate), correct them now and
tell me the corrected mapping. Once you confirm, I will apply the IDs to all
five artifacts. Type 'Confirmed — proceed with migration' to continue."

Do not proceed to Phase 2 until the engineer confirms.

---

PHASE 2 — Update TOPOLOGY.md

Step 1 — Update A02 (Module Call Map):

If discovery/components/A02_module_call_map.md exists:
- Add a Module Roster section at the very top of the file (before any existing content):
  ## Module Roster — [system name]
  Migrated: YYYY-MM-DD by BCE schema migration to v2.0
  | ID | Module Name | Source File | Layer |
  (populate from MODULE_CONTRACTS.md entries using confirmed M-NNN assignments)
- In the Internal Call Table: replace each prose Caller and Callee name with its
  M-NNN from the confirmed mapping. Change table columns to:
  | Edge | Call Site (file:line) | Sync/Async |
  | M-NNN --[CALLS]--> M-NNN | | S / A |
  If Call Site is unknown in the existing table, write NOT RECORDED.
- In Startup Sequence: replace prose module names with M-NNN.
- In Async Boundaries: replace prose producer/consumer names with M-NNN.
- Save the updated file.

If discovery/components/A02_module_call_map.md does not exist:
- Note this in the migration report. The Module Roster will be embedded in
  TOPOLOGY.md directly until Session A is run to produce a full A02.
- Proceed without creating A02.

Step 2 — Update A03 entries in TOPOLOGY.md:
For each External System entry in the A03 section:
- Add IP-NNN (from confirmed mapping) as the first field of the record
- If a "Called by" field does not exist: add it with NOT RECORDED (migration — to be
  confirmed by engineer from INTEGRATION_CONTRACTS.md Called by data)
- If a "Called by" field exists with prose module names: replace with M-NNN

Step 3 — If A02 module call map does not exist: add a Module Roster section to TOPOLOGY.md:
Add after the A03 section:
## Module Roster (Migration Placeholder)
Migrated: YYYY-MM-DD — full A02 module call map not yet produced.
Run Session A to produce a complete A02 with typed call edges.
| ID | Module Name | Source File (from MODULE_CONTRACTS.md) | Layer |
(populate from confirmed M-NNN mapping)

Save TOPOLOGY.md.

---

PHASE 3 — Update MODULE_CONTRACTS.md

For each module contract in MODULE_CONTRACTS.md:
1. Add an ID field immediately after the Module Name header:
   **ID** — M-NNN (from BCE schema migration — YYYY-MM-DD)

2. Add three fields at the end of the contract (after Change Impact):
   **Callers** — [populate from A02 CALLS edges where this module is the callee;
   if A02 does not exist or has no record: NOT RECORDED — confirm from team knowledge]
   **Calls** — [populate from A02 CALLS edges where this module is the caller;
   if A02 does not exist or has no record: NOT RECORDED — confirm from team knowledge]
   **Integration Points Used** — [populate from INTEGRATION_CONTRACTS.md Called-by fields
   for this module; if no record: NOT RECORDED]

3. If the contract already has Internal Dependencies or External Dependencies fields
   with prose names: do not delete them. Add a migration note below each:
   [MIGRATION NOTE — YYYY-MM-DD]: Callers/Calls fields above use M-NNN. Prose names
   in this field are superseded and may be removed after engineer review.

Save MODULE_CONTRACTS.md.

---

PHASE 4 — Update INVARIANT_CATALOGUE.md

For each IC-N record in INVARIANT_CATALOGUE.md:
1. Identify the owning module: the module most directly responsible for this invariant.
   Use one of these sources in priority order:
   (a) The "Enforcement point" field — the module containing that function is the owner
   (b) MODULE_CONTRACTS.md Known Fragility or Change Impact fields that reference this invariant
   (c) NOT DETERMINABLE — if neither source is conclusive

2. Identify enforcing modules: all modules listed in the Enforcement point field and any
   module whose Known Fragility or Change Impact section directly references this invariant.

3. Add two fields after the Enforcement point field:
   Owning module: M-NNN (or NOT DETERMINABLE — see migration note)
   Enforcing modules: M-NNN [, M-NNN ...] (or NOT DETERMINABLE)

4. If NOT DETERMINABLE: also add:
   [MIGRATION NOTE — YYYY-MM-DD]: Owning/enforcing module could not be determined from
   existing artifact fields. Engineer should supply M-NNN from operational knowledge.
   Add as ENGINEER NOTE once confirmed.

Save INVARIANT_CATALOGUE.md.

---

PHASE 5 — Update INTEGRATION_CONTRACTS.md

For each contract record:
1. Prepend IP-NNN to the record header:
   Change: **[External System Name]**
   To:     **IP-NNN — [External System Name]**

2. Add a Called by field as the first field after the header:
   Called by: [search MODULE_CONTRACTS.md Integration Points Used fields for this IP-NNN;
   list all M-NNN found; if none found: NOT RECORDED]

Save INTEGRATION_CONTRACTS.md.

---

PHASE 6 — Update RISK_REGISTER.md

For each risk entry:
1. Find any field named "Affected module", "Module affected", "Source module", or
   similar that contains a prose module name. Replace the prose name with M-NNN.
   If the prose name does not match any entry in the confirmed module mapping exactly:
   flag it as AMBIGUOUS and leave the prose name in place with a migration note.

2. Find any field that references an invariant by statement text or prose name.
   Replace with IC-N from the confirmed invariant list.
   If ambiguous: flag and leave in place with a migration note.

Save RISK_REGISTER.md.

---

PHASE 7 — Migration Report

Produce a migration summary:

## BCE Schema Migration Report — YYYY-MM-DD

### IDs Assigned
- Modules: [count] (M-001 through M-NNN)
- Integration Points: [count] (IP-001 through IP-NNN)
- Invariants: [count] confirmed existing IDs
- Risks: [count] confirmed existing IDs

### Artifacts Updated
| Artifact | Changes applied | NOT RECORDED entries | AMBIGUOUS entries |
|---|---|---|---|
| TOPOLOGY.md | | | |
| MODULE_CONTRACTS.md | | | |
| INVARIANT_CATALOGUE.md | | | |
| INTEGRATION_CONTRACTS.md | | | |
| RISK_REGISTER.md | | | |

### Items Requiring Engineer Follow-Up
[List every NOT RECORDED and AMBIGUOUS entry, with the artifact, field, and
what the engineer needs to supply to resolve it.]

### Next Steps
1. Review the items requiring follow-up above. Supply missing information where known.
2. Commit the migrated artifacts.
3. Run Stage 3 — Check 0 (schema validation gate) to confirm migration is complete.
4. If Check 0 passes: run "Build system graph" to produce discovery/SYSTEM_GRAPH.json.
```