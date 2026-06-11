---
name: bce-signal
version: v1.0
description: >
  BCE Signal skill — BCE-S pipeline for extracting intent, business constraints, and
  operational history from non-code sources. Load for BCE-S extraction sessions on any
  path (A, B, or C). Contains: SOURCE_INVENTORY.md production, Stage 1 adapter
  procedures (Documents, Teams, Git), Stage 2 automated validation prompt, Stage 3 CD
  assembly prompt, SIGNAL_GAPS.md structure, ratification model, and planning
  consumption rules. Also trigger when working with SOURCE_INVENTORY.md, SYSTEM_INTENT.md,
  BUSINESS_INVARIANTS.md, OPERATIONAL_RISK_HISTORY.md, SIGNAL_GAPS.md, or any
  discovery/signal/ file.
OWNER: DataGrokr LLC
COPYRIGHT: "© 2025 DataGrokr LLC. All rights reserved."
LICENSE: >-
  Proprietary — for use by licensed DataGrokr clients only.
  Unauthorized reproduction, modification, or transfer is prohibited.
CONTACT: naveen@datagrokr.com
---

# BCE Signal Skill — v1.0

## Changelog

| Version | Date | Change Summary |
|---|---|---|
| v1.0 | April 2026 | Initial release — BCE-S pipeline: SOURCE_INVENTORY.md, Stage 1 adapter procedures (Documents, Teams, Git), Stage 2 validation prompt, Stage 3 assembly prompt, SIGNAL_GAPS.md structure, ratification model. |

---

## 1. What This Skill Does

This skill makes CC and CD co-pilots for BCE-S (Signal) extraction. BCE-S is the
non-code pipeline — it extracts intent, business constraints, and operational history
from documents, collaboration tools, and work history sources. It runs independently
of BCE-C and produces a structured enrichment layer consumed in PBVI planning sessions.

BCE-S is optional. Planning always proceeds on BCE-C alone when BCE-S is not available.
When BCE-S artifacts are present, only RATIFIED claims are injected into planning context.

**Load this skill when:**
- Running BCE-S Stage 1 adapter extraction (any source class)
- Running BCE-S Stage 2 validation
- Running BCE-S Stage 3 assembly
- Working with any file in `discovery/signal/`
- Producing or reviewing SOURCE_INVENTORY.md

**Also load bce_core.md when:** BCE-C and BCE-S are running on the same project.

---

## 2. BCE-S Naming and Pipeline Position

BCE-S is one of two pipelines under the BCE umbrella:

| Pipeline | Sources | Primary artifact location | Skill |
|---|---|---|---|
| BCE-C | Source code | discovery/ | bce_core.md |
| BCE-S | Documents, collaboration tools, work history | discovery/signal/ | bce_signal.md (this file) |

Both pipelines feed the SIL (System Intelligence Layer) — the external name for the
combined BCE output. Neither pipeline replaces the other. BCE-C is mandatory. BCE-S
is enrichment.

BCE-S Stage 2 always runs after BCE-C completes — BCE-C discovery/ artifacts are the
mandatory validation baseline. There is no fallback mode. If BCE-C has not completed,
BCE-S Stage 2 does not run.

---

## 3. BCE-S Artifact Set

BCE-S produces artifacts in three roles, parallel to BCE-C.

### Role 1 — Navigational Prerequisite (produced before Stage 1, not updated after)

| Artifact | File | What it captures |
|---|---|---|
| Source Inventory | SOURCE_INVENTORY.md | Every non-code source identified, BCE-S source class, adapter, accessibility status, scope constraints. Drives Stage 1 adapter selection. Point-in-time record. |

### Role 2 — Phase-Aligned Extraction Artifacts (produced at Stage 3)

| Artifact | File | PBVI phase consumed | What it captures |
|---|---|---|---|
| System Intent Record | SYSTEM_INTENT.md | Phase 1 — Architecture | System purpose; design decisions and rejected alternatives; capability boundaries |
| Business Invariant Record | BUSINESS_INVARIANTS.md | Phase 2 — Invariants | Explicit business rules; compliance constraints; organisational commitments |
| Operational Risk History | OPERATIONAL_RISK_HISTORY.md | Phase 4 — Design Gate | Recurring failure patterns; contested decisions; known gaps and deferred problems |

### Role 3 — Attestation Artifact (produced at Stage 3, maintained on re-run)

| Artifact | File | What it captures |
|---|---|---|
| Signal Gap Backlog | SIGNAL_GAPS.md | All UNVERIFIABLE claims, cross-source contradictions unresolved at assembly, open questions from Stage 1. Parallel role to ANNOTATION_CHECKLIST.md in BCE-C. |

All BCE-S artifacts live in `discovery/signal/`. Component files live in
`discovery/signal/components/`. Both directories are uploaded to CD project files.
Repository is the source of truth.

---

## 4. Source Classes and Adapters

BCE-S sources are organised into three classes. Each has a defined adapter and
extraction characteristic.

| Class | Sources | Primary signal | First-wave adapters |
|---|---|---|---|
| Intent Sources | Design docs, PRDs, ADRs, requirement docs, product brochures | What the system was designed to do and why — including rejected alternatives | Documents adapter |
| Organisational Memory | Microsoft Teams exports, Confluence, wikis, runbooks | What the team knows, debates, and decides — including decisions made in conversation | Teams adapter |
| Work History | Git history, Jira, PR reviews | Churn, failure patterns, contested decisions, recurring problems | Git adapter |

**First-wave adapters:** Documents, Teams, Git. Additional adapters (Jira, Confluence,
SharePoint, PR reviews) are defined as DEFERRED in SOURCE_INVENTORY.md until built.

---

## 5. Ratification Model

Every claim in a BCE-S artifact carries a ratification state. Planning consumption
depends on this state.

| State | How assigned | Planning use |
|---|---|---|
| RATIFIED | Stage 2: claim consistent with BCE-C evidence | Eligible — injected into planning session context |
| RATIFIED (adjudicated) | Engineer adjudicated a DIVERGENT claim and confirmed correctness | Eligible — adjudication note present on claim |
| DIVERGENT | Stage 2: claim contradicts BCE-C evidence — specific contradiction identified | Not eligible until adjudicated |
| UNVERIFIABLE | Stage 2: no BCE-C evidence to confirm or contradict | Not eligible — retained as honest gap |
| SUPERSEDED | Engineer adjudicated DIVERGENT — non-code source is no longer correct | Not eligible — retained for history |

**Planning consumption rule:** RATIFIED → inject. All other states → skip.
This rule is mechanical. No engineer judgment required at planning time to apply it.

UNVERIFIABLE claims are never surfaced for human adjudication. They are excluded
from planning automatically and retained in SIGNAL_GAPS.md as honest gaps.
On stranger paths, a high proportion of UNVERIFIABLE claims is expected and acceptable.

---

## 6. SOURCE_INVENTORY.md — Production and Structure

SOURCE_INVENTORY.md is the navigational prerequisite for BCE-S Stage 1. It is the
adapter selection decision made explicit — which sources exist, which adapters will
run, and what scope constraints apply. Stage 1 reads it before running any adapter.

**When to produce:**
- Path A: after custodian intake sessions — sources surfaced by the source inventory
  question in the interview
- Path B: before BCE-S Stage 1 — inventory what is discoverable without custodian access
- Path C: at the BCE-C Stage 1 human gate — engineer inventories sources outside docs/

**Trigger phrases:**
- "Produce SOURCE_INVENTORY.md"
- "Create source inventory"
- "Build source inventory"

**Tool:** Engineer-authored (CD may assist with structure and formatting)

**Template:**

```markdown
# SOURCE_INVENTORY.md — [System Name]
**Date:** [date]
**Engineer:** [name]
**Path:** [A — Custodian / B — Stranger / C — PBVI Adapter]

## Source Registry

| Source | Description | BCE-S Source Class | Adapter | Status | Scope Constraints |
|---|---|---|---|---|---|
| [e.g. Product Requirements v2.pdf] | [one line] | Intent Source | Documents adapter | ACCESSIBLE | [date range or none] |
| [e.g. Teams — #engineering channel] | [one line] | Organisational Memory | Teams adapter | ACCESSIBLE | [date range] |
| [e.g. Git — main repository] | [one line] | Work History | Git adapter | ACCESSIBLE | [branch, date range] |
| [e.g. Confluence — product space] | [one line] | Organisational Memory | Confluence adapter | DEFERRED — adapter not yet built | — |
| [e.g. Jira] | [one line] | Work History | Jira adapter | INACCESSIBLE — no export available | — |

## Adapter Run Plan
[Which adapters will run in Stage 1, in what order, and what scope each covers]

## Deferred and Inaccessible Sources
[Sources identified but not running — reason and any plan to address]
```

**Status values:**

| Status | Meaning |
|---|---|
| ACCESSIBLE | Source available — adapter will run in Stage 1 |
| DEFERRED | Source exists — adapter not yet built |
| INACCESSIBLE | Source exists — cannot be accessed in this run |
| NOT PRESENT | Source class has no instance for this project |

**Commit:** `discovery/signal/SOURCE_INVENTORY.md`
**Gate:** Engineer reviews before Stage 1 begins.

---

## 7. Stage 1 — Per-Source Extraction

Stage 1 reads SOURCE_INVENTORY.md and runs each ACCESSIBLE adapter independently.
No cross-source synthesis in Stage 1. One component file per source. Sources marked
DEFERRED or INACCESSIBLE are noted but do not run.

**Per-claim confidence markers applied in Stage 1:**

| Marker | Meaning |
|---|---|
| HIGH | Claim is explicit and unambiguous in the source |
| MEDIUM | Claim is inferred from context — source implies but does not state directly |
| LOW | Claim is implied but not clearly stated |
| STALENESS_RISK | Claim from a source with no recent date or beyond project age threshold |
| DESIGN_INTENT_ONLY | Claim from an intent source — states what was planned, not what was implemented |

**Human gate — Stage 1 to Stage 2:** Engineer reviews all component files. Confirms
extraction scope was correct. Flags sources that were missing or should be excluded.
Does not resolve individual claims — that is Stage 2.

---

### 7.1 — Documents Adapter

**Source class:** Intent Sources
**Ingestion:** Uploaded files (PDF, Word). CC reads directly.
**Tool:** CC
**Trigger phrases:**
- "Run BCE-S Documents adapter"
- "Extract from documents"
- "Process design documents"
- "Ingest documents"

**What to provide before running:**
- Uploaded document files in session context
- SOURCE_INVENTORY.md confirming Documents adapter is ACCESSIBLE
- List of document types present (PRD / design doc / ADR / requirement doc / brochure)

**Step A — Document inventory (equivalent to CODEBASE_MAP.md in BCE-C):**

Before extraction, CC produces a one-line inventory of every uploaded document:
document name, apparent type, primary subject, and date if present. Engineer reviews
before extraction begins. This is the A0 equivalent for BCE-S documents.

```
You are performing a BCE-S document inventory before Signal extraction begins.

For each uploaded document, produce one line in this format:
[Document name] — [Type: PRD | Design Doc | ADR | Requirement Doc | Brochure | Other]
— [Primary subject: one sentence] — [Date: YYYY-MM or NOT STATED]

Do not read document contents beyond what is needed to classify them.
Produce only the inventory table. No commentary.
```

Engineer reviews the inventory. Confirms all relevant documents are present.
Flags any documents that should be excluded. Commits or notes the inventory before
extraction begins.

**Step B — Per-document extraction:**

Run one extraction prompt per document. Save each as a component file:
`discovery/signal/components/DOC-NN_[short-title].md`

```
You are performing BCE-S Stage 1 extraction from an intent source document.

Document: [document name]
Type: [PRD | Design Doc | ADR | Requirement Doc | Brochure | Other]
Date: [date if known, or STALENESS_RISK if unknown]

Read the document completely. Extract claims in three categories:

CATEGORY 1 — System Purpose and Capability
Claims about what the system does in business terms, who it serves, what it is
explicitly not intended to do, and what capability boundaries were stated.
Tag each claim: DESIGN_INTENT_ONLY unless the document explicitly states the
claim reflects implemented reality.

CATEGORY 2 — Design Decisions and Constraints
Explicit decisions made about how the system is built, why those decisions were
made, and what alternatives were considered and rejected.
Rejected alternatives are high-value — capture them explicitly.
Tag each claim: HIGH (explicitly stated) | MEDIUM (inferred from context) | LOW (implied)

CATEGORY 3 — Explicit Rules, Constraints, and Obligations
Business rules, compliance requirements, stated invariants, organisational commitments.
These are candidates for BUSINESS_INVARIANTS.md.
Tag each claim: HIGH | MEDIUM | LOW
Flag any claim that appears to be a regulatory or legal obligation as COMPLIANCE.

CATEGORY 4 — Stated Risks, Known Gaps, Deferred Problems
Problems, limitations, or risks the document explicitly acknowledges.
These are candidates for OPERATIONAL_RISK_HISTORY.md.

For each claim in each category, record:
- Claim text: [one sentence, precise]
- Confidence: [HIGH | MEDIUM | LOW | STALENESS_RISK | DESIGN_INTENT_ONLY]
- Source location: [section or page reference]
- Target artifact: [SYSTEM_INTENT | BUSINESS_INVARIANTS | OPERATIONAL_RISK_HISTORY]

Do not synthesise across categories. Do not infer beyond what is stated.
Mark any area where the document is ambiguous or contradicts itself as
INTERNAL_CONTRADICTION — flag for Stage 3 cross-source review.
Produce only the structured component file. No commentary outside the structure.
```

**Component file header (mandatory):**
```
# DOC-NN — [Document name]
STAGE-1-DRAFT: BCE-S DOCUMENTS ADAPTER — [date]
Source class: Intent Source
Document type: [type]
Document date: [date or STALENESS_RISK]
```

---

### 7.2 — Teams Adapter

**Source class:** Organisational Memory
**Ingestion:** Exported conversation file (JSON or similar). CC reads the export.
**Tool:** CC
**Trigger phrases:**
- "Run BCE-S Teams adapter"
- "Extract from Teams"
- "Process Teams conversations"
- "Ingest Teams export"

**What to provide before running:**
- Teams export file in session context or accessible path
- SOURCE_INVENTORY.md confirming Teams adapter is ACCESSIBLE with channel and date range
- Scope: which channels and what date range to process

**Step A — Channel and thread inventory:**

Before full extraction, CC reads the export structure and produces an inventory:
channels present, date range covered, approximate message volume, and a preliminary
signal score per channel (HIGH / MEDIUM / LOW signal density based on thread structure
and topic indicators). Engineer reviews before full extraction begins.

```
You are performing a BCE-S Teams export inventory before Signal extraction begins.

Read the Teams export file structure.

For each channel or conversation thread, produce:
- Channel name
- Date range covered
- Approximate message count
- Preliminary signal assessment: HIGH (decision threads, design debates, incident
  discussions) | MEDIUM (status updates, mixed content) | LOW (social, announcements,
  noise)
- Notable thread topics (one line per high-signal thread)

Do not extract claims yet. This is a scope confirmation step.
Produce only the inventory. Engineer reviews before full extraction runs.
```

Engineer reviews. Confirms which channels to include. Specifies any channels to exclude.
This pre-filter prevents context volume from overwhelming the extraction session.

**Step B — High-signal thread extraction:**

Run extraction against HIGH and MEDIUM signal channels only.

```
You are performing BCE-S Stage 1 extraction from a Teams export.

Channels in scope: [list from inventory review]
Date range: [from SOURCE_INVENTORY.md]

Read all messages in the scoped channels. Identify and extract signal in four categories:

CATEGORY 1 — Decision Moments
Threads where a question was raised, debated, and resolved. The resolution is the
claim. The debate is context. For each:
- Decision: [what was decided, one sentence]
- Context: [what was debated, one to two sentences]
- Confidence: HIGH if resolution is explicit | MEDIUM if inferred from thread close
- Target artifact: SYSTEM_INTENT (architectural decisions) | BUSINESS_INVARIANTS
  (rules and constraints) | OPERATIONAL_RISK_HISTORY (operational decisions)

CATEGORY 2 — Unresolved Questions
Threads that surfaced a problem or question with no clear resolution.
- Question: [what was asked or raised]
- Status: UNRESOLVED
- Target artifact: SIGNAL_GAPS.md

CATEGORY 3 — Operational Warnings
Messages flagging known problems, workarounds, fragilities, or "don't do X" knowledge.
- Warning: [what was flagged, one sentence]
- Confidence: HIGH | MEDIUM | LOW
- Target artifact: OPERATIONAL_RISK_HISTORY

CATEGORY 4 — Stated Constraints or Rules
Moments where someone articulated a rule the system must follow.
- Rule: [what was stated]
- Confidence: HIGH | MEDIUM | LOW
- Target artifact: BUSINESS_INVARIANTS

All claims from Teams carry DESIGN_INTENT_ONLY by default unless a message
explicitly confirms implementation ("we shipped this", "this is now live").

Low-signal threads (social conversation, announcements, status updates with no
substantive content) — discard without recording.

Do not synthesise across threads. Process each thread independently.
Produce only the structured component file. No commentary outside the structure.
```

**Component file:** `discovery/signal/components/TEAMS-NN_[channel-name].md`

**Component file header (mandatory):**
```
# TEAMS-NN — [Channel name]
STAGE-1-DRAFT: BCE-S TEAMS ADAPTER — [date]
Source class: Organisational Memory
Channel: [name]
Date range: [from] to [to]
Messages processed: [count]
```

---

### 7.3 — Git Adapter

**Source class:** Work History
**Ingestion:** Pre-processing script produces normalised summary files. CC reads summaries.
**Tool:** Pre-processing script (bash/Python) → CC
**Trigger phrases:**
- "Run BCE-S Git adapter"
- "Extract from Git"
- "Process git history"
- "Ingest git history"

**What to provide before running:**
- Git repository accessible (local clone or path)
- SOURCE_INVENTORY.md confirming Git adapter is ACCESSIBLE with branch and date range
- Scope: branch, date range, any paths to exclude (e.g. generated files, lock files)

**Step A — Run pre-processing script:**

The pre-processing script runs against the repository before CC is involved. It produces
four normalised summary files in `discovery/signal/components/git-raw/`:

```bash
# BCE-S Git Pre-Processing Script
# Run from repository root. Adjust DATE_FROM, DATE_TO, BRANCH, EXCLUDE_PATHS.

DATE_FROM="[YYYY-MM-DD]"
DATE_TO="[YYYY-MM-DD]"
BRANCH="[main|master|as specified in SOURCE_INVENTORY]"
EXCLUDE_PATHS=":(exclude)*.lock :(exclude)package-lock.json :(exclude)*.generated.*"
OUTPUT_DIR="discovery/signal/components/git-raw"

mkdir -p $OUTPUT_DIR

# 1. Churn report — commit frequency per file
git log $BRANCH --after=$DATE_FROM --before=$DATE_TO \
  --name-only --pretty=format: -- $EXCLUDE_PATHS \
  | grep -v '^$' | sort | uniq -c | sort -rn \
  > $OUTPUT_DIR/churn_report.txt

# 2. Author concentration — commits per author per module (top-level directory)
git log $BRANCH --after=$DATE_FROM --before=$DATE_TO \
  --format="%an" --name-only -- $EXCLUDE_PATHS \
  | awk '/^[^/]+\// {dir=$0; sub(/\/.*/, "", dir)} /^$/ {next} !/\// {print author, $0} {author=$0}' \
  > $OUTPUT_DIR/author_concentration.txt

# Simpler author concentration — commits per author
git shortlog -sn $BRANCH --after=$DATE_FROM --before=$DATE_TO \
  > $OUTPUT_DIR/author_summary.txt

# 3. Commit intent patterns — categorised by message prefix
git log $BRANCH --after=$DATE_FROM --before=$DATE_TO \
  --pretty=format:"%H|%ad|%an|%s" --date=short \
  > $OUTPUT_DIR/commit_log.txt

# 4. Recency map — last modified date per file
git log $BRANCH --pretty=format: --name-only -- $EXCLUDE_PATHS \
  | grep -v '^$' | sort -u \
  | while read f; do
      last=$(git log -1 --pretty=format:"%ad" --date=short -- "$f" 2>/dev/null)
      echo "$last $f"
    done | sort \
  > $OUTPUT_DIR/recency_map.txt

echo "BCE-S Git pre-processing complete. Files in $OUTPUT_DIR"
```

Engineer reviews the four output files for completeness before CC extraction begins.

**Step B — CC extraction from normalised summaries:**

```
You are performing BCE-S Stage 1 extraction from Git history summaries.

You have four normalised summary files:
1. churn_report.txt — commit frequency per file (highest to lowest)
2. author_summary.txt — commits per author across the repository
3. commit_log.txt — full commit log with hash, date, author, message
4. recency_map.txt — last modified date per file

Read all four files completely before producing any output.

Extract signal in four categories:

CATEGORY 1 — Module Fragility Signals
Files and directories with high commit frequency relative to the rest of the
repository. High churn indicates instability — the system is being touched
repeatedly, suggesting ongoing problems, unclear ownership, or architectural pressure.

For each high-churn area (top 20% by commit count):
- Module or path: [file or directory]
- Commit count in period: [N]
- Fragility signal: [one sentence — what the churn pattern suggests]
- Confidence: HIGH (objective count) — always HIGH for churn data
- Target artifact: OPERATIONAL_RISK_HISTORY

CATEGORY 2 — Bus Factor Risks
Authors with disproportionate commit concentration on specific modules.
Single-author concentration = knowledge silo = bus factor risk.

For each module where one author accounts for >60% of commits:
- Module: [path]
- Concentrated author: [name]
- Concentration: [N of M commits]
- Risk signal: bus factor — [one sentence]
- Confidence: HIGH
- Target artifact: OPERATIONAL_RISK_HISTORY

CATEGORY 3 — Commit Intent Patterns
From commit_log.txt, categorise commits by message type:
- FIX / BUGFIX / HOTFIX — reactive fixes (fragility signal)
- REVERT — reversals (instability signal — what was reverted and why)
- FEAT / FEATURE — new capability (growth signal)
- REFACTOR — structural change (evolution signal)
- CHORE / DEPS — maintenance (neutral)

For each module with a high ratio of FIX or REVERT commits:
- Module: [derived from files touched in those commits]
- Fix/Revert count: [N]
- Pattern signal: [one sentence]
- Confidence: MEDIUM (commit message quality varies)
- Target artifact: OPERATIONAL_RISK_HISTORY

For any commit message that references an explicit decision, constraint, or
architectural rule — even in passing — extract it:
- Commit reference: [hash, date]
- Decision or constraint stated: [verbatim from message]
- Target artifact: SYSTEM_INTENT or BUSINESS_INVARIANTS as appropriate
- Confidence: MEDIUM (commit messages are informal)

CATEGORY 4 — Recency Map Signals
From recency_map.txt, identify:
- Modules last modified more than 12 months ago — potential staleness
- Modules modified very recently — active development, higher change risk

For each module not touched in 12+ months:
- Module: [path]
- Last modified: [date]
- Signal: STALENESS_RISK — may not reflect current design intent
- Target artifact: OPERATIONAL_RISK_HISTORY

Do not invent intent. Commit messages are informal and may be misleading.
Confidence for intent-derived claims is always MEDIUM or lower.
Churn counts and dates are objective — always HIGH confidence.
Produce only the structured component file. No commentary outside the structure.
```

**Component file:** `discovery/signal/components/GIT-01_history_summary.md`

**Component file header (mandatory):**
```
# GIT-01 — Git History Summary
STAGE-1-DRAFT: BCE-S GIT ADAPTER — [date]
Source class: Work History
Repository: [name or path]
Branch: [branch]
Date range: [from] to [to]
Pre-processing script run: [date]
```

---

## 8. Stage 2 — Automated Validation Pass

Stage 2 validates Stage 1 component file claims against BCE-C discovery/ artifacts.
This is a CC job. It always runs after BCE-C completes — BCE-C discovery/ artifacts
are the mandatory validation baseline.

**Tool:** CC
**Trigger phrases:**
- "Run BCE-S Stage 2 validation"
- "Validate BCE-S claims"
- "Run signal validation"
- "Validate signal sources"

**What to provide before running:**
- All Stage 1 component files in session context (from discovery/signal/components/)
- All BCE-C discovery/ artifacts in session context
- SOURCE_INVENTORY.md

**Execution prompt:**

```
You are running BCE-S Stage 2 — Automated Validation Pass.

You have:
- BCE-S Stage 1 component files (discovery/signal/components/)
- BCE-C discovery/ artifacts (TOPOLOGY.md, MODULE_CONTRACTS.md,
  INTEGRATION_CONTRACTS.md, INVARIANT_CATALOGUE.md, RISK_REGISTER.md)

For each claim in each Stage 1 component file, perform a consistency check
against BCE-C artifacts. Assign one ratification state per claim.

RATIFICATION STATES:

RATIFIED — The claim is consistent with BCE-C artifact evidence. The BCE-C
artifact explicitly confirms or is consistent with the claim. Record which
BCE-C artifact and section provides the confirming evidence.

DIVERGENT — The claim contradicts a BCE-C artifact. The BCE-C artifact
explicitly or implicitly contradicts the claim. Record:
- Which claim (component file, claim text)
- Which BCE-C artifact and section contradicts it
- The specific contradiction — what the claim says vs. what BCE-C says
DIVERGENT claims require human adjudication before planning use.

UNVERIFIABLE — No BCE-C artifact contains evidence to confirm or contradict
the claim. This is not a failure — it is an honest gap. Record as UNVERIFIABLE
with a note on why no BCE-C evidence exists for this claim type.

RULES:
- DESIGN_INTENT_ONLY claims: if BCE-C confirms implementation, upgrade to RATIFIED.
  If BCE-C contradicts, mark DIVERGENT with the specific divergence.
  If BCE-C has no evidence, retain as UNVERIFIABLE with DESIGN_INTENT_ONLY marker.
- STALENESS_RISK claims: cannot be ratified from BCE-C alone — mark UNVERIFIABLE
  unless BCE-C directly addresses the claim content.
- HIGH confidence Git churn and date data: always RATIFIED (objective historical data,
  no BCE-C contradiction possible for historical facts).

Process component files in this order:
1. All Intent Source component files (DOC-NN)
2. All Organisational Memory component files (TEAMS-NN)
3. All Work History component files (GIT-NN)

For each component file, output an updated version with ratification states
appended to each claim. Do not modify claim text. Add after each claim:
RATIFICATION: [RATIFIED | DIVERGENT | UNVERIFIABLE]
EVIDENCE: [BCE-C artifact and section, or "No BCE-C evidence"]

After processing all component files, produce a DIVERGENT CLAIMS SUMMARY:
List every DIVERGENT claim with:
- Component file and claim reference
- The specific contradiction
- What human adjudication must determine

[HUMAN GATE]
Output the DIVERGENT CLAIMS SUMMARY and wait for engineer adjudication.
For each DIVERGENT claim: engineer states which source is correct and why.
Engineer records decision as an adjudication note on the claim.
Adjudicated claims become RATIFIED or SUPERSEDED based on the decision.

After all DIVERGENT claims are adjudicated, commit updated component files to
discovery/signal/components/ with header updated to:
STAGE-2-VALIDATED: [date]
```

**Commit after Stage 2:**
```bash
git add discovery/signal/components/
git commit -m "BCE-S Stage 2 validation complete — [N] claims RATIFIED, [N] DIVERGENT adjudicated, [N] UNVERIFIABLE"
```

**Human gate — Stage 2 to Stage 3:** Engineer confirms all DIVERGENT claims are
adjudicated. No DIVERGENT claims may carry into Stage 3 unresolved.

---

## 9. Stage 3 — CD Assembly

Stage 3 runs in CD after all Stage 2 component files are validated and committed.
CD performs cross-source contradiction detection, assembles the three phase-aligned
artifacts from RATIFIED claims only, synthesises INTAKE_SUMMARY.md on Path B, and
produces SIGNAL_GAPS.md.

**Tool:** CD
**Trigger phrases:**
- "Run BCE-S Stage 3 assembly"
- "Assemble signal artifacts"
- "Build signal artifacts"
- "Produce signal artifacts"

**What to load into CD project files before running:**
- All Stage 2 validated component files
- SOURCE_INVENTORY.md
- BCE-C discovery/ artifacts (for cross-reference)
- INTAKE_SUMMARY.md if already exists (Path A / C)
- Flag: Path A | Path B | Path C

**Execution prompt:**

```
You are running BCE-S Stage 3 — CD Assembly.

You have:
- BCE-S Stage 2 validated component files — all claims carry ratification states
- SOURCE_INVENTORY.md
- BCE-C discovery/ artifacts
- Path: [A | B | C]

Run the following steps in order. Complete each step fully before proceeding.

---

STEP 1 — Cross-Source Contradiction Detection

Review all component files for contradictions between non-code sources — contradictions
that Stage 2 did not surface because they exist between two non-code sources rather
than between a non-code source and BCE-C.

For each contradiction found:
- Source A claim (component file, claim text)
- Source B claim (component file, claim text)
- Nature of contradiction
- Recommended resolution

[HUMAN GATE — Step 1]
Present all cross-source contradictions found. Engineer adjudicates each one before
Step 2 begins. If no contradictions found, state this explicitly and proceed.

---

STEP 2 — Assemble SYSTEM_INTENT.md

From RATIFIED claims only (including adjudicated RATIFIED), assemble SYSTEM_INTENT.md.
Organise into three sections:

Section 1 — System Purpose
What the system does in business terms. Who it serves. What problem it solves.
Source: primarily Intent Source (DOC-NN) claims in SYSTEM_INTENT category.

Section 2 — Design Decisions and Rationale
Decisions that shaped how the system was built. For each decision: what was decided,
why, and what alternatives were rejected.
Include rejected alternatives explicitly — they are high planning value.
Source: primarily Intent Source and Organisational Memory claims.

Section 3 — Capability Boundaries
What the system is intended to do and what it explicitly is not.
Source: Intent Source claims.

For each entry, record:
- Claim text
- Source: [component file reference]
- Ratification: RATIFIED | RATIFIED (adjudicated)
- Original confidence: [HIGH | MEDIUM | LOW | DESIGN_INTENT_ONLY]

Do not include UNVERIFIABLE or SUPERSEDED claims.

---

STEP 3 — Assemble BUSINESS_INVARIANTS.md

From RATIFIED claims in BUSINESS_INVARIANTS category, assemble BUSINESS_INVARIANTS.md.
Organise into three sections:

Section 1 — Business Rules
Explicit constraints from requirement documents and PRDs.
Each must be a verifiable condition — not a goal or target.

Section 2 — Compliance Constraints
Regulatory or contractual obligations. Flag each as COMPLIANCE.

Section 3 — Organisational Commitments
Things the team has explicitly committed to maintaining — from ADRs, design reviews,
or Teams decision moments.

For each entry, record:
- Rule or constraint text
- Source: [component file reference]
- Ratification: RATIFIED | RATIFIED (adjudicated)
- Type: BUSINESS_RULE | COMPLIANCE | ORGANISATIONAL_COMMITMENT
- BCE-C cross-reference: [INVARIANT_CATALOGUE.md entry if a corresponding invariant exists,
  or "No corresponding BCE-C invariant — candidate for INVARIANT_CATALOGUE.md"]

---

STEP 4 — Assemble OPERATIONAL_RISK_HISTORY.md

From RATIFIED claims in OPERATIONAL_RISK_HISTORY category, assemble
OPERATIONAL_RISK_HISTORY.md. Organise into three sections:

Section 1 — Failure Patterns
Recurring problems from Git churn, Teams incident discussions, and document-stated risks.
For corroborated patterns (same area flagged by multiple sources), note corroboration.

Section 2 — Contested Decisions
Design debates from Teams that were resolved but not unanimously. These are live risk
areas — the losing argument may resurface when the system is touched.

Section 3 — Known Gaps and Deferred Problems
Things the team documented as problems but did not fix. Explicitly deferred items.

For each entry, record:
- Risk or pattern description
- Source(s): [component file references]
- Ratification: RATIFIED | RATIFIED (adjudicated)
- BCE-C cross-reference: [RISK_REGISTER.md entry if corroborated, or "Not in BCE-C RISK_REGISTER"]
- Corroboration: CORROBORATED (multiple sources) | SINGLE_SOURCE

---

STEP 5 — Produce SIGNAL_GAPS.md

Review all component files for UNVERIFIABLE claims, unresolved open questions from
Stage 1 TEAMS-NN files, and any contradictions not yet resolvable.

For each gap item, produce a SIGNAL_GAPS.md entry:

```markdown
## SG-[P1|P2|P3]-NNN · [short title] ([target artifact])

**Severity:** P1 | P2 | P3
**Type:** UNVERIFIABLE | OPEN_QUESTION | CONTRADICTION
**Source:** STAGE1_EXTRACTION | STAGE2_VALIDATION | STAGE3_ASSEMBLY
**Artifact:** SYSTEM_INTENT | BUSINESS_INVARIANTS | OPERATIONAL_RISK_HISTORY

**Observation:** [what was found or not found — specific]

**Planning implication:** [what planning might miss if this gap is not resolved]

**Recommended action:** [what would be needed to resolve this gap]

**STATUS:** OPEN
```

Default severity by type:
- CONTRADICTION (cross-source, unresolved) → P1
- UNVERIFIABLE claim on a planning-relevant topic → P2
- UNVERIFIABLE claim on a peripheral topic → P3
- OPEN_QUESTION → P3 unless it affects an invariant candidate (then P2)

---

STEP 6 — Path B only: Synthesise INTAKE_SUMMARY.md

(Skip this step on Path A and Path C — INTAKE_SUMMARY.md already exists.)

From the validated Stage 1 component files, synthesise INTAKE_SUMMARY.md for BCE-C.
This is the handoff between BCE-S and BCE-C. Scope strictly to what BCE-C needs.

Template:
```markdown
# INTAKE_SUMMARY.md — [System Name]
**Date:** [date]
**Engineer:** [name]
**Path:** B — Stranger (derived from BCE-S Stage 1 component files)

## System Purpose
[From SYSTEM_INTENT claims — one paragraph maximum. What the system does in business
terms. If no clear claim exists, write: NOT DETERMINABLE FROM BCE-S SOURCES]

## Known Architecture
[From SYSTEM_INTENT design decisions — what is known about system structure before
looking at code. Keep to what BCE-S sources stated explicitly.]

## Known Pain Points
[From OPERATIONAL_RISK_HISTORY claims — operational problems, fragilities, debt items.
Include Git churn signals as objective evidence.]

## Documents Reviewed
[List all BCE-S source component files with one-line contribution summary]

## Open Questions Before Extraction
[From SIGNAL_GAPS.md items and Teams UNRESOLVED threads — questions that could not
be answered from available BCE-S sources]

## Confidence Assessment
[LOW — no custodian knowledge. BCE-S sources provide structural starting point only.
Substantive BCE-C annotation expected.]
```

INTAKE_SUMMARY.md is committed to discovery/ (not discovery/signal/).
It is the pre-extraction gate for BCE-C Stage 1 on Path B.

---

STEP 7 — Commit

Commit all assembled artifacts:

```bash
# BCE-S artifacts
git add discovery/signal/SYSTEM_INTENT.md
git add discovery/signal/BUSINESS_INVARIANTS.md
git add discovery/signal/OPERATIONAL_RISK_HISTORY.md
git add discovery/signal/SIGNAL_GAPS.md
git commit -m "BCE-S Stage 3 complete — [N] RATIFIED claims assembled; [N] gaps in SIGNAL_GAPS.md"

# Path B only — INTAKE_SUMMARY.md handoff to BCE-C
git add discovery/INTAKE_SUMMARY.md
git commit -m "BCE-S Path B handoff — INTAKE_SUMMARY.md synthesised from Stage 1 component files"
```

[HUMAN GATE — Stage 3 complete]
Engineer reviews all assembled artifacts and SIGNAL_GAPS.md.
Signs off before BCE-C Stage 1 begins (Path B) or before artifacts are uploaded
to CD project files (Path A / C).
```

---

## 10. SIGNAL_GAPS.md — Structure and Severity Model

SIGNAL_GAPS.md is the BCE-S attestation artifact. It is the backlog of what BCE-S
knows it does not know well. Parallel role to ANNOTATION_CHECKLIST.md in BCE-C.

**Item ID convention:** SG-P[1|2|3]-NNN (e.g. SG-P1-001, SG-P2-003)

**Severity model:**

| Severity | Default triggers |
|---|---|
| P1 | Unresolved cross-source contradiction at Stage 3 assembly |
| P2 | UNVERIFIABLE claim on a topic that directly feeds Phase 1, 2, or 4 planning |
| P3 | UNVERIFIABLE claim on a peripheral topic; OPEN_QUESTION from Teams |

Downgrading from default requires stated rationale. Upgrading never requires justification.

**Planning use of SIGNAL_GAPS.md:**
SIGNAL_GAPS.md items do not block planning the way BCE-C ANNOTATION_CHECKLIST.md
P1 items do. They are informational context — the engineer knows what BCE-S could
not confirm. SG-P1 items (unresolved contradictions) are worth reviewing before
Phase 1 begins on an area the contradiction touches.

**Resolution:** SIGNAL_GAPS.md items are resolved when BCE-S is re-run with
additional source access, or when BCE-C extraction confirms or contradicts the
gap claim. Resolved items are marked RESOLVED with date and evidence. They are
never deleted.

---

## 11. Lifecycle and Maintenance

**Initial run:** BCE-S runs once at project onboarding. For Path B, BCE-S Stage 1
runs before BCE-C. For Path A and C, BCE-S runs after BCE-C completes.

**Ongoing updates:** Not required as routine discipline on PBVI-governed projects.
PBVI artifacts capture decisions and changes as they occur. Non-code sources on
PBVI projects are shadows of the PBVI record.

**Manual re-run triggers:**
- Major product or architectural direction change documented outside PBVI governance
- New compliance requirement added to organisational documentation
- Significant volume of new historical context becomes available
- BCE-C re-extraction reveals substantial divergence from BCE-S RATIFIED claims

**Re-run scope:** Targeted — re-run the affected source adapter, re-validate affected
claims only. Not a full pipeline re-run unless the source inventory itself has changed.

**Scheduled cadence:** Not required. BCE-S does not run on a timer.

---

## 12. Quick Reference

**BCE-S Stage 2 hard constraint:** Always runs after BCE-C completes. No exceptions.
No fallback mode. If BCE-C is not done, Stage 2 waits.

**Planning consumption rule:** RATIFIED → inject. All other states → skip.
Mechanical. No judgment required at planning time.

**UNVERIFIABLE is honest** — not a failure. Gap is tracked in SIGNAL_GAPS.md.
Not surfaced for human adjudication. Excluded from planning automatically.

**DIVERGENT requires human adjudication** — engineer decides which source is correct.
Decision recorded as adjudication note. Claim becomes RATIFIED or SUPERSEDED.

**Path B sequencing:** SOURCE_INVENTORY → BCE-S Stage 1 → BCE-C (all stages) →
BCE-S Stage 2 → BCE-S Stage 3 (which also produces INTAKE_SUMMARY.md for BCE-C).

**Stage 3 is always CD** — cross-artifact reasoning and assembly require the
Thinking AI. CC runs Stage 1 and Stage 2. CD runs Stage 3.

**Trigger phrases summary:**

| Action | Trigger phrases | Tool |
|---|---|---|
| Produce SOURCE_INVENTORY.md | "Produce SOURCE_INVENTORY.md" · "Create source inventory" · "Build source inventory" | Engineer + CD |
| Stage 1 — Documents adapter | "Run BCE-S Documents adapter" · "Extract from documents" · "Process design documents" · "Ingest documents" | CC |
| Stage 1 — Teams adapter | "Run BCE-S Teams adapter" · "Extract from Teams" · "Process Teams conversations" · "Ingest Teams export" | CC |
| Stage 1 — Git adapter | "Run BCE-S Git adapter" · "Extract from Git" · "Process git history" · "Ingest git history" | Script → CC |
| Stage 2 — Validation | "Run BCE-S Stage 2 validation" · "Validate BCE-S claims" · "Run signal validation" · "Validate signal sources" | CC |
| Stage 3 — Assembly | "Run BCE-S Stage 3 assembly" · "Assemble signal artifacts" · "Build signal artifacts" · "Produce signal artifacts" | CD |