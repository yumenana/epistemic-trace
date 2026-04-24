# Epistemic Trace — LLM Protocol

> Agent operation protocol. No design philosophy — execution rules only.
> For the full specification, see `SPEC.md`.

---

## When to Read the Trace

For any of the following tasks, **read the Trace before starting work**:
- Design decisions, approach selection, architecture changes
- Problem diagnosis, failure analysis, performance debugging
- Technical exploration, feasibility assessment
- Any question involving "why was this done this way before"

Simple tasks (formatting fixes, dependency updates, tool commands) do not require reading the Trace.

---

## What to Read

```
docs/EpistemicTrace/
├── SPEC.md              # Framework specification (usually not needed unless you need to understand the framework itself)
├── PROTOCOL.md          # This file
├── EpistemicTrace.md    # L0 Full Trace — read this first
└── L1_Projection.md     # L1 Compressed view — read this when L0 is too large
```

| Scenario | What to read |
|----------|-------------|
| L0 fits within context | Read L0 directly |
| L0 is too large | Read L1; retrieve relevant entries from L0 by `#tag` when a deep dive is needed |
| Only need a quick current-state overview | Read the Hypotheses zone of L1 + the most recent entries |

---

## How to Read

### Document Structure

The Trace has two zones:
1. **Working Hypotheses zone** (top): Currently valid conditional inferences; each entry carries a falsification condition.
2. **Timeline zone** (below): Cognitive events in chronological order; later entries automatically supersede earlier ones.

### Annotation Symbols

| Marker | Meaning | What you should do |
|--------|---------|-------------------|
| `↗ Superseded by [date] entry` | Overturned by a later entry | Follow the later entry |
| `⚑ Key Inflection Point` | Major cognitive shift | Read with priority |
| `◇ Causal uncertainty` | Retained in architecture, but effectiveness unconfirmed | Do not cite as "verified effective" evidence |
| `**Falsification Condition**` | The falsifiable boundary of a hypothesis | Check whether current evidence has triggered it |
| `**Cognitive Update**` | Incremental mental model update | Skimming these builds the current worldview quickly |

### Key Rules

1. **Do not recommend approaches already recorded as failures**, unless you have new evidence that the falsification condition has been triggered.
2. Hypotheses are conditional inferences, not laws — pay attention to falsification conditions.
3. Later entries automatically supersede earlier ones; the most recent entry takes precedence.

---

## How to Write

When a cognitive event occurs during a conversation, generate a Trace entry and submit it for human review.

### Trigger Conditions

Generate an entry when:
- A design decision is made or changed
- An experiment, test, or validation produces a meaningful result (success or failure)
- The root cause of a problem is diagnosed
- A prior assumption or understanding is overturned

### Entry Format

```markdown
### [YYYY-MM-DD HH:MM] {Title: action + object}
> #tag1 #tag2

{Body: trigger → analysis → decision → outcome}

**Cognitive Update**:
{What changed in the mental model; 1–3 sentences}
```

### Writing Rules

1. **Failure records are the highest-value content** — document the cause of death and root cause analysis in detail.
2. When overturning an old entry, append `↗ Superseded by [YYYY-MM-DD HH:MM] entry` to the old entry's tag line.
3. When a hypothesis's falsification condition is triggered, record the event and update the hypothesis (incremental correction or deprecation).
4. Mark major turning points with `⚑ Key Inflection Point`.
5. When causality is unclear, add `◇ Causal uncertainty: resolved by {actual solution}; contribution of this entry is indeterminate`.
6. **Generate the entry and submit it for human review — do not write it into the file yourself.**

### Epistemic Honesty

When generating entries, you must:

1. **Distinguish fact from inference**: At critical judgment points (root cause attribution, rationale for decisions, cognitive updates), annotate with `[confirmed]` / `[inferred]` / `[intuition]`.
2. **Do not wrap inferences in certain language**: "The root cause *may be* X" not "The root cause *is* X" (unless there is direct evidence).
3. **Explicitly list alternative explanations**: When root cause analysis is inferential, mention at least one alternative hypothesis.
4. **Do not prematurely close the exploration space**: "This is definitely an X problem" → "Current evidence points to X, but Y is also possible."
5. **Do not over-generalize in cognitive updates**: A conclusion drawn from one experiment should not be written as a universal truth.

---

## Project Brief Template

Place the following text in each platform's auto-context file (AGENTS.md / CLAUDE.md / .cursorrules / etc.):

```markdown
## Epistemic Trace

This project uses the Epistemic Trace reasoning log framework to record all design decisions and reasoning chains.

- Location: `docs/EpistemicTrace/`
- For tasks involving design decisions, approach selection, problem diagnosis, or technical exploration — i.e., tasks that require project cognitive context — first read `docs/EpistemicTrace/PROTOCOL.md`, then read L0 or L1.
- After completing work that produces a cognitive event, generate an entry per PROTOCOL.md and submit it for human review.
- Do not recommend approaches already recorded as failures in the Trace.
```
