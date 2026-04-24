**English** | [简体中文](./README_zh.md)

<div align="center">

# Epistemic Trace

**An append-only reasoning log for LLM-assisted research.**

*Record the* why *behind decisions — not just the* what.

<br/>

[![License: MIT](https://img.shields.io/badge/license-MIT-blue)](./LICENSE)
![Status](https://img.shields.io/badge/status-v0.1%20specification-brightgreen)
![Format](https://img.shields.io/badge/format-Markdown-lightgrey?logo=markdown)
![LLM](https://img.shields.io/badge/LLM-Copilot%20·%20Claude%20·%20Cursor%20·%20Aider-blueviolet)

</div>

---

> Falsifiability conditions, confidence tags, and a failure-first philosophy.
> Designed for both human and LLM consumption — across sessions, across months.

---

## The Problem

You are six months into a research project, and your AI coding assistant has helped you write most of the code.

Today you ask it: *"What about adding a global residual connection here?"*

It happily writes the patch. You stare at the diff for ten seconds before remembering — you tried this in week three. Training broke. You spent two days finding out why. **None of that survived.**

You might object: isn't that what the README is for? What about comments? — but the README records outcomes, with maybe a line on *why this conclusion*; comments live next to code, and the next refactor wipes them out, leaving at most a note on *why we refactored*. Neither remembers *why* you went one way and not the other, what you tried first, what failed, and what you learned from it failing. The git log is just *what changed when*, not *why we didn't take the other path*.

And your AI assistant starts every new conversation from zero. The same constraints you explain again. The same dead ends you warn against again. The same hard-won intuitions you restate again. Sometimes you don't catch it in time, and it confidently rebuilds something you already discarded.

**Epistemic Trace is a structured reasoning log designed to be read by both you and the LLM.** It preserves the chain of thought across sessions, marks failures as first-class content, and forces every hypothesis to declare what would falsify it.

---

## What It Looks Like

A Trace has two zones: **Working Hypotheses** at the top, and a **Timeline** of cognitive events below.

```markdown
### H4: Avoid normalization in the main reconstruction stream
> #normalization #amplitude-phase
> ↗ Scope corrected by H10 [2026-04-19] — decoder amplitude-accumulation nodes need GroupNorm

Not "super-resolution doesn't need normalization" — specifically:

1. [inferred] Normalization forces zero-mean + unit-variance. In 4× anime SR, the
   absolute color value of flat regions IS the reconstruction target; mapping to
   zero-mean causes color drift.
2. [inferred] Hair tips are extreme asymmetric high-energy spikes. Pulling them
   back to a "normal" distribution causes gap-closing failure and Gibbs ringing.

**Falsification Condition**: If a DC-aware normalization is designed (normalizes
only AC, preserves DC anchor), this restriction can be relaxed.
```

```markdown
### [2026-04-09 10:10] Failure: spatial palette + grid_sample + PDE convection collapsed
> #training-failure #grid-sample #pde #palette #gradient-disconnect
> ⚑ Key Inflection Point

**Observation** [confirmed]: After training, hair-tip reconstruction barely improved.

**Cause-of-death analysis** [inferred]:
1. **grid_sample's physical impossibility at low resolution**: palette_field is
   generated at 1/4 resolution. A 2px hair tip becomes 0.5px there. Bilinear
   interpolation physically blends 50%+ background color.
2. **Gradient disconnect on remote offsets**: Zero-init offset starts sampling
   locally → local neighborhood is 100% background → gradient only steers offset
   within background → never "sees" the 50px-distant hair-root color.
...

**Cognitive Update**:
- [inferred] Any color-routing scheme that depends on grid_sample interpolation
  has a physical resolution limit on low-resolution feature maps.
- [inferred] Any operator depending on local gradients has a gradient-disconnect
  problem under (zero-init + remote-target) conditions.
```

Three things are happening here that you don't see in conventional documentation:

1. **`[confirmed] / [inferred] / [intuition]`** — every critical claim declares its evidential strength. The LLM cannot launder an inference into a fact.
2. **`Falsification Condition`** — every hypothesis is required to state what would overturn it. Hypotheses without falsifiability are dogmas.
3. **`↗ Superseded by`** — when a later entry corrects an earlier one, both are preserved. The original reasoning is never deleted; the chain of revision is itself the most valuable record.

---

## Why It Works (First Principles)

### 1. Event sourcing beats state mutation

Every robust system that needs to recover from corruption — bank ledgers, distributed databases, version control — is append-only. Mutating history destroys the evidence needed to diagnose where things went wrong.

Most engineering documentation does the opposite: it overwrites yesterday's understanding with today's. When today's understanding turns out to also be wrong, there is no trail back. Epistemic Trace treats your reasoning the way Git treats your code: every state is preserved; revisions are explicit additions, not silent overwrites.

### 2. Falsifiability is what separates a hypothesis from a belief

Karl Popper's criterion: a claim that cannot be falsified is not a scientific claim. The same applies to engineering judgments. *"PixelShuffle is unsuitable for anime super-resolution"* is a belief. *"PixelShuffle is unsuitable for anime super-resolution; this is overturned if a strong post-processing layer like a powerful DCN demonstrates it can compensate for the math limitations"* is a hypothesis.

Forcing every hypothesis to declare its falsification condition does two things:
- It exposes which of your beliefs are actually testable
- It tells future-you (and the LLM) exactly what evidence to watch for

### 3. Human memory is reconstructive; LLM memory is non-existent

Cognitive science is clear: human episodic memory is not a recording. It is reconstructed from cues at retrieval time, and the reconstruction is biased by current state. Six months from now, your memory of "why I picked architecture B" will be a confabulation in the shape of your current beliefs.

LLMs have it worse — they have no episodic memory at all. Each conversation begins from a blank slate plus whatever context you can fit in the window.

The right response is to externalize reasoning into a durable, structured artifact that both audiences can read. Epistemic Trace is designed from day one to be that artifact for both — its format is structured enough for an LLM to parse reliably, and prose-rich enough for a human to skim.

### 4. Failure is the densest form of information

In a search problem, every dead end you've eliminated narrows the remaining space. Engineering project documentation almost universally records only the path that worked. This is the lowest-entropy possible record.

Epistemic Trace inverts this: failure entries are designated as the highest-value content. A `Failure:` entry with a detailed cause-of-death analysis prevents the same dead end from being proposed again — by you, by your collaborators, or by your LLM.

### 5. LLMs amplify epistemic dishonesty if you let them

LLMs default to confident, deterministic prose. *"The root cause is X."* When that text is later read — by a human, or by another LLM in another session — the reader treats it as fact. New decisions get stacked on a guess that was rendered as certainty.

The `[confirmed] / [inferred] / [intuition]` tagging system is a structural defense. The protocol requires the LLM to mark its evidential ground at every critical judgment point. You can scan a Trace and see exactly which load-bearing claims are guesses.

---

## Where It Fits

Epistemic Trace is most useful when *all* of the following are true:

- The project runs over weeks or months, not hours
- You collaborate with an LLM agent (Copilot, Claude Code, Cursor, Aider, etc.)
- Decisions accumulate, get revised, and need to be revisited
- A wrong recommendation costs real time (training runs, experiments, deployments)

Concretely: deep learning research, scientific software, complex refactors, product strategy, drug discovery, legal arguments — anywhere reasoning needs to compound rather than evaporate.

It is **overkill** for: short tasks, exploratory scripts, projects without LLM collaboration, or environments where you have a tight team and the tribal knowledge fits in everyone's head.

### Comparison to existing systems

| System | What it captures | What's missing for LLM-assisted research |
|--------|------------------|------------------------------------------|
| ADR (Architecture Decision Records) | Decision + context + consequences | No confidence tags, no LLM protocol, no failure-first design |
| Lab notebook | Day-by-day exploration | Hand-written, no LLM-readable structure, no causal-uncertainty marker |
| Zettelkasten / wiki | Atomic, linkable notes | History is mutable — destroys evidence of revision |
| `CHANGELOG` + git log | What changed, when | Records what you did, not what you tried and discarded |

---

## Getting Started

### 1. Add the framework files to your project

```
your-project/
└── docs/
    └── EpistemicTrace/
        ├── SPEC.md              # the framework specification
        ├── PROTOCOL.md          # the LLM operation protocol
        └── EpistemicTrace.md    # your project's Trace (start empty)
```

Copy `SPEC.md` and `PROTOCOL.md` from this repo. Create an empty `EpistemicTrace.md` with the two-zone skeleton.

### 2. Tell your LLM agent about it

Add the Project Brief snippet (from `PROTOCOL.md`, last section) to your platform's auto-context file:

| Platform | File |
|----------|------|
| GitHub Copilot | `.github/copilot-instructions.md` |
| Claude Code | `CLAUDE.md` |
| Cursor | `.cursorrules` |
| Aider / others | `AGENTS.md` |

This is the only thing loaded into every session by default. It instructs the agent when and how to consult the Trace — keeping the token cost on simple tasks at zero.

### 3. Generate the first entry from a real decision

Don't pre-populate the Trace by writing a manual. Let it grow from real cognitive events. The next time you make an architecture choice, ask your LLM agent: *"Per PROTOCOL.md, generate a Trace entry for this decision."* Review and merge.

### 4. Generate hypotheses as patterns emerge

Once 5–10 entries share a `#tag`, a working hypothesis is usually visible. Ask the LLM to draft an `H{N}` entry summarizing the constraint, including a falsification condition. Review and merge.

---

## Status

**v0.1 — Specification.** The framework is stable in real research use (see `examples/` once published). Tooling is intentionally minimal: a Markdown file and a protocol document. No build step, no dependencies, no lock-in.

Planned: a real-world Trace example, language translations, a `#tag` retrieval CLI, and an L0 → L1 compression script.

---

## Contributing

Translations, real-world Trace examples, integration guides for new platforms, and tooling are all welcome. Open an issue to discuss before larger contributions.

Translations go under `i18n/{locale}/`. Open a PR.

---

## License

MIT. Use it, fork it, adapt it. If it changes how you work with an LLM, I'd love to hear about it.
