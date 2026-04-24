# Epistemic Trace — Specification v0.1

> An append-only reasoning log framework.
> Records the **reasoning chain** behind decisions — not just conclusions.
> Designed for two audiences: the author's own knowledge flow + an LLM agent's context source.

---

## 0. Design Philosophy

**Why record reasoning chains instead of conclusions**:
Conclusions can always be overturned. Reasoning chains retain value forever — even when a conclusion is invalidated, every analytical step in the chain remains a valid constraint elimination. For an LLM, reading a reasoning chain means it can assess "could the current hypothesis be overturned by the same class of evidence?" rather than simply accepting an ungrounded assertion.

**Why Append-Only**:
Modifying history destroys evidence. Reasoning that looks "wrong" in hindsight is precisely the highest-value information — it marks dead ends on the cognitive map and prevents future repetition.

---

## 1. Core Rules

| # | Rule | Rationale |
|---|------|-----------|
| R1 | **Append only, never modify** the body of historical entries | Event sourcing integrity |
| R2 | The only permitted retroactive operation: append `↗ Superseded by [date] entry` to the tag line of an old entry | Preserves the causal chain; readers can follow the link |
| R3 | **Later entries automatically supersede earlier ones**; the most recent entry takes precedence | Avoids maintaining metadata about "which version is current" |
| R4 | `#tag` provides high-dimensional addressing, linking across modules and time | Flat tags > hierarchical directories; supports multi-axis cross-referencing |

---

## 2. Document Structure

An Epistemic Trace document consists of two zones:

### 2.1 Working Hypotheses Zone

Located at the top of the document. Contains **currently valid** conditional inferences.

Format for each hypothesis:

```markdown
### H{N}: {One-sentence proposition}
> #tag1 #tag2

{Body: why this is believed, what evidence supports it}

**Falsification Condition**: {Under what experiment or evidence should this hypothesis be overturned}
```

**Required fields**:
- **Number** `H{N}`: Globally unique, only increments. Deprecated hypotheses are not renumbered — mark `[deprecated]` in the body and link to the replacement entry.
- **Proposition**: One sentence; may be an assertion or a constraint ("avoid X", "must Y").
- **Tags**: At least one `#tag`.
- **Falsification Condition**: **Mandatory**. A hypothesis without a falsification condition is a dogma.

**Optional fields**:
- **Important addendum [date]**: Non-overturning incremental correction to a hypothesis. Appended after the original text; original is not modified.
- **Tables/lists**: Supporting exposition.

**Hypothesis update protocol**:
- Incremental correction → append an `**Important addendum [YYYY-MM-DD]**:` paragraph at the end of the hypothesis.
- Overturning correction → write a new entry in the Timeline zone; mark the old hypothesis `[deprecated] ↗ superseded by [date] entry`; assign the new hypothesis a new number.

### 2.2 Timeline Zone

Entries are ordered chronologically (oldest first, newest last). Each entry is a **cognitive event**.

---

## 3. Timeline Entry Format

```markdown
### [YYYY-MM-DD HH:MM] {Title: action + object}
> #tag1 #tag2 #tag3
> ↗ Superseded by [YYYY-MM-DD] entry        ← optional; append only when a later entry overturns this one

{Body}

**Cognitive Update**:
{What changed in the mental model after this event. 1–3 sentences.}
```

### 3.1 Entry Types (non-mandatory categories, for reference only)

| Type | Typical title prefix | Notes |
|------|---------------------|-------|
| Design decision | `Architecture:`, `Implementation:`, `Selection:` | What was chosen and why |
| Experiment failure | `Failure:` | **Highest value**. Record dead ends and root causes |
| Experiment result | `Result:` | Data/observations + interpretation |
| Cognitive turning point | (no prefix; annotate with `⚑ Key Inflection Point`) | Overturns a prior core assumption |
| Abandoned approach | `Interim approach (abandoned):` | Approach discarded without full validation; include reason |

### 3.2 Body Structure (recommended, not mandatory)

A complete decision entry typically contains:

1. **Trigger**: The phenomenon or problem that prompted this thinking
2. **Analysis / Diagnosis**: The reasoning process; describe the causal chain explicitly
3. **Decision**: What was done, or which approach was chosen
4. **Outcome** (if available): Experimental results or validation conclusion
5. **Cognitive Update**: Incremental update to the mental model

Failure entries additionally contain:
- **Observation**: The observable anomalous behavior
- **Root cause analysis**: Numbered list of each fatal defect
- **Cognitive Update**: General lessons extracted from the failure

### 3.2.1 Confidence Tags (Epistemic Honesty)

Analysis and conclusions must distinguish **confirmed facts** from **inferential judgments**. Use the following tags:

| Tag | Meaning | When to use |
|-----|---------|-------------|
| `[confirmed]` | Supported by direct evidence | Reproducible experimental results; observable data |
| `[inferred]` | Reasonable inference based on evidence, but not directly verified | Root cause analysis; causal attribution |
| `[intuition]` | No hard evidence; based on experience or analogy | Early hypotheses; directional judgment |

**Why this matters**:
LLMs naturally tend toward deterministic language ("the root cause *is* X", "this *proves* Y"), even when the claim is merely inferred. When subsequent readers (human or LLM) encounter an inference written in certain language, they treat it as confirmed fact, leading to:
- New decisions stacked on false premises
- Ignoring alternative explanations
- Prematurely closing the exploration space

Confidence tags do not need to be added to every sentence. Only annotate **critical judgment points** (root cause attribution, rationale for a decision, cognitive update).

### 3.3 Key Inflection Point Annotation

When an entry overturns a prior core assumption or changes the project's direction, add below the tag line:

```markdown
> ⚑ Key Inflection Point
```

Readers (human or LLM) can scan for `⚑` markers to quickly locate major cognitive shifts.

### 3.4 Causal Uncertainty Annotation

When a problem is ultimately resolved by a different means (e.g., expanding the dataset, extending training), making it impossible to confirm whether earlier work contributed, append to the tag line of relevant entries:

```markdown
> ◇ Causal uncertainty: resolved by {actual solution}; contribution of this entry is indeterminate
```

**Semantics**: This is not a retraction (`↗`), not a turning point (`⚑`), and carries no action item. It is a pure epistemic honesty marker — "this work remains in the final design, but I don't know whether it helped."

**Meaning for LLMs**: Design decisions in `◇`-marked entries remain part of the current architecture, but must not be cited as evidence of "verified effectiveness."

---

## 4. Tag System

### 4.1 Format

- Format: `#kebab-case`, all lowercase, hyphen-separated
- Location: blockquote line immediately below the entry heading, space-separated
- An entry may carry any number of tags
- Tags do not need to be predefined — create on demand; let them emerge naturally

### 4.2 Uses of Tags

- **Cross-time linking**: The same `#tag` strings together the complete chain for a topic, from initial hypothesis through every revision or retraction.
- **Cross-module linking**: `#loss #topology` appearing together signals a cross-cutting insight between loss and topology.
- **LLM retrieval anchor**: An LLM can be instructed to "focus only on entries tagged #tag."

### 4.3 What Tags Do Not Do

- Not a classification system (an entry does not need to "belong to" a category)
- Not a hierarchy (no `#parent/child`)
- Not a version marker (the timeline's own ordering handles this)

---

## 5. Retroactive Annotation Protocol

When a new entry overturns or corrects an old one:

**On the old entry**, append to the end of the tag line:
```markdown
> ↗ Superseded by [YYYY-MM-DD HH:MM] entry
```

**In the new entry**, explicitly reference the old entry:
> "Complete redesign addressing three defects from the [YYYY-MM-DD] entry"

This creates a bidirectional link: old→new (retroactive annotation) + new→old (body reference).

---

## 6. LLM Collaboration Design

Epistemic Trace is a **LLM-generated, human-reviewed** document. The LLM is both the primary author and the primary consumer. The human's role is to trigger writes (via conversation, tasks, or events) and to review content — most of the body text does not need to be written by hand.

### 6.1 Platform-Agnostic Layered Architecture

Different LLM agent platforms (VS Code Copilot, Claude Code, Cursor, etc.) each offer two context injection mechanisms:

| General concept | Function | Per-platform implementation |
|----------------|----------|----------------------------|
| **Auto-context** — loaded by default each session | Project identity + Trace entry pointer | AGENTS.md / CLAUDE.md / .cursorrules / etc. |
| **On-demand context** — actively fetched by the agent | Operation protocol + Trace content | Any platform's `read_file` capability |

File structure:

```
docs/EpistemicTrace/
├── SPEC.md              # Framework specification (this file; for both humans and LLMs)
├── PROTOCOL.md          # LLM operation protocol (distilled from SPEC; execution rules only)
├── EpistemicTrace.md    # L0 Full Trace
└── L1_Projection.md     # L1 Compressed view
```

**Design principle**: Trace-related token cost is zero on simple tasks. Only tasks that require project cognitive context — design decisions, problem diagnosis, etc. — prompt the agent to read PROTOCOL.md and the Trace files.

### 6.2 Project Brief (place in each platform's auto-context file)

The following standard text should be placed in each platform's auto-context file:

```markdown
## Epistemic Trace

This project uses the Epistemic Trace reasoning log framework to record all design decisions and reasoning chains.

- Location: `docs/EpistemicTrace/`
- For tasks involving design decisions, approach selection, problem diagnosis, or technical exploration — i.e., tasks that require project cognitive context — first read `docs/EpistemicTrace/PROTOCOL.md`, then read L0 or L1.
- After completing work that produces a cognitive event, generate an entry per PROTOCOL.md and submit it for human review.
- Do not recommend approaches already recorded as failures in the Trace.
```

**What the Project Brief does NOT include**: Trace content, hypotheses, timeline, or protocol details — those are in files the agent fetches on demand.

### 6.3 LLM Operation Protocol

Complete read/write rules are in `PROTOCOL.md`. That file is distilled from §2–§5 of this SPEC; it contains no design philosophy, only the operational rules the agent needs to execute.

Agent workflow in a new session:

```
New session → auto-load Project Brief
  → user request
    → simple task → work directly, do not read Trace
    → task requiring project cognitive context → read PROTOCOL.md → read L0 or L1 → work → generate entry → human review
```

---

## 7. Known Limitations and Future Directions

> This section records open problems with the framework itself; it is not project-specific.

### 7.1 Token Budget: Two-Level Crystallization Protocol

**Tension**: The value of reasoning chains lies in their detail, but LLM context windows are finite.

**Solution: L0 Full + L1 Projection**

| Level | Content | Author | Trigger | Compression |
|-------|---------|--------|---------|-------------|
| **L0 Full** | Complete reasoning chain | LLM generates; human reviews | Real-time (when a conversation, task, or event occurs) | **Never compressed** |
| **L1 Projection** | Compressed view | LLM derives from L0; human reviews | When L0 exceeds context budget | Lossy; can always be regenerated from L0 |

**Core invariant**: L1 is a pure function of L0. L1 can be regenerated from L0 at any time (costs tokens; loses no information).

#### Crystallization Rules (L0 → L1)

| Content type | Compression strategy |
|-------------|---------------------|
| Hypotheses zone | **Preserve verbatim** (already high-density; not compressible) |
| Active work zone (entries from the last N days) | **Preserve verbatim** (current work requires full context) |
| Stabilized entry chains (attempt → failure → fix → success) | LLM compresses to a summary paragraph; preserves tags, retroactive links, and cognitive updates; compresses the analysis body |
| Tag index | Auto-generated; placed at the top of L1 (`#tag: [date1], [date2], ...`) |

Each compressed summary carries metadata:
```markdown
> [crystallized: YYYY-MM-DD, source: L0 entries YYYY-MM-DD~YYYY-MM-DD]
```

#### Active Window (N value)

Principle for choosing N: the maximum time span that current work might need to look back over. A starting value of N=14 days is recommended; adjust based on actual usage. Entries within the active window are copied verbatim from L0 to L1 without any compression.

#### Usage Strategy

| Scenario | What to feed |
|----------|-------------|
| L0 fits within context | Feed L0 directly |
| L0 exceeds context | Feed full L1 |
| Deep dive on a `#tag` | Feed L1 + retrieve relevant entries from L0 by tag |

#### Semantic Drift Protection

LLM summarization is lossy compression; semantic drift is a risk (the LLM discards information based on current salience, but a future cognitive turning point may need exactly those details). Mitigations:
1. L0 is never compressed — drift is isolated to L1; L0 can always be used to regenerate L1.
2. Each summary carries a `[crystallized]` timestamp and source range — readers know which point-in-time view this represents.
3. When an L1 summary conflicts with current understanding, **always go back to the L0 source text**; do not make judgments based on L1 alone.

### 7.2 Cross-Project Knowledge Transfer

Some hypotheses are domain knowledge (e.g., "anime images are piecewise-smooth manifolds"), not project-specific.

**To be designed**: A separation mechanism for domain-level vs. project-level knowledge. Possible form: a domain-level Epistemic Trace (e.g., `anime-sr.epistemic.md`) plus a project-level Trace.
