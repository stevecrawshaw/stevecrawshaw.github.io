## Quantifying What You Don't Know: Using Empirica for Epistemic Tracking in AI-Assisted Research

**TLDR; When AI assists with research, how do you know what it actually knows versus what it's confidently guessing? I used Empirica — a framework for tracking AI epistemic state — during a research session on entheogenic phenomenology. The result: explicit uncertainty quantification that distinguished solid phenomenological patterns (0.80 confidence) from speculative causal explanations (0.30 confidence). Here's how it works and why it matters for any AI-assisted research.**

## The Problem: Confident AI, Hidden Uncertainty

AI assistants are remarkably good at synthesizing information and presenting coherent narratives. They're also remarkably bad at signalling when they're on solid ground versus thin ice.

When I asked Claude to explore the phenomenology of entheogenic entities — the spirits and beings people report encountering during psychedelic experiences — it produced detailed taxonomies, cross-cultural comparisons, and causal hypotheses. All delivered with the same confident tone.

But some of that content was well-documented ethnographic observation. Some was speculative hypothesis. And some was creative synthesis across incommensurable frameworks. **How would a reader know which was which?**

## Enter Empirica: Epistemic Self-Awareness for AI

[Empirica](https://github.com/Nubaeon/empirica#live-metacognitive-signal) is a framework that makes AI epistemic state explicit and trackable. Instead of hoping the AI will caveat appropriately, Empirica requires structured self-assessment across 13 epistemic vectors.

The core workflow is **CASCADE**:

```
PREFLIGHT → NOETIC → CHECK → PRAXIC → POSTFLIGHT
    ↓          ↓        ↓        ↓          ↓
 Baseline   Investigate  Gate    Act      Measure
```

For research tasks, this translates to:
1. **PREFLIGHT**: Assess what you actually know before starting
2. **NOETIC**: Investigate, logging findings and unknowns as you go
3. **CHECK**: Validate readiness — do you know enough to proceed?
4. **POSTFLIGHT**: Measure what changed

## The Session: Tracking Uncertainty in Real-Time

### Starting Point: Honest Baseline

At session start, I ran a PREFLIGHT assessment:

```bash
empirica session-create --ai-id claude-code --output json
empirica preflight-submit --session-id <ID> --vectors '{
  "know": 0.55,
  "uncertainty": 0.50,
  "context": 0.35,
  ...
}'
```

The vectors force honest self-assessment:
- **know: 0.55** — Moderate baseline knowledge of the topic
- **uncertainty: 0.50** — Significant uncertainty about scope and depth
- **context: 0.35** — User's specific interests unclear

This is the opposite of confident presentation. It's saying: "I know some things, but there are gaps."

### During Research: Logging Findings and Unknowns

As the session progressed, Empirica captured the epistemic landscape:

**Findings logged:**
```bash
empirica finding-log --finding "Cross-entheogen archetypal mapping:
  entity types by substance, geographic correlations" --impact 0.9
```

**Unknowns logged:**
```bash
empirica unknown-log --unknown "Causal mechanism for archetype consistency -
  competing hypotheses with no empirical adjudication method"
```

This creates a trail of what was discovered and what remains uncertain.

### The CHECK Gate: Am I Ready to Proceed?

Mid-session, I ran a CHECK to assess readiness:

```bash
empirica check --session-id <ID> --output json
```

Empirica's metacognition layer applied **bias corrections**:

| Vector | Self-Assessed | Corrected | Bias |
|--------|---------------|-----------|------|
| know | 0.60 | 0.55 | -0.05 (slight overestimate) |
| uncertainty | 0.55 | 0.65 | +0.10 (underestimate uncertainty) |

The readiness gate (`know >= 0.70 AND uncertainty <= 0.35`) **did not pass**. This is honest: the research was producing interesting patterns, but causal explanations remained speculative.

### Final Assessment: Differentiated Confidence

The POSTFLIGHT captured the final epistemic state with **confidence differentiated by claim type**:

| Claim Category | Confidence | Evidence Quality |
|----------------|------------|------------------|
| Phenomenological reports (what people experience) | **0.80** | Strong — thousands of documented reports |
| Cross-substance patterns | **0.70** | Moderate-strong — robust but selection bias |
| Geographic-entity correlation | **0.65** | Moderate — correlation clear, causation unclear |
| Neurochemical mapping | **0.40** | Weak-moderate — speculative |
| Causal explanations | **0.30** | Weak — multiple frameworks, no adjudication |

This is the payoff: **not all claims are equally certain, and now that's explicit**.

## Why This Matters for AI-Assisted Research

### 1. Prevents Confident Bullshit

Without epistemic tracking, AI tends toward uniform confident presentation. Empirica forces differentiation: "I'm 80% confident about X, 30% confident about Y."

### 2. Makes Unknowns Visible

The `unknown-log` command captures what **hasn't** been resolved:

```
- Causal mechanism for archetype consistency
- Ontological status of entities
- Entity persistence mechanism
```

These aren't failures — they're honest acknowledgment of the epistemic frontier.

### 3. Enables Calibration Over Time

Empirica tracks whether self-assessments are accurate. The bias corrections (+0.10 on uncertainty) come from historical calibration data. Over time, the AI learns where it systematically over- or under-estimates.

### 4. Creates Auditable Epistemic Trail

The session produces a traceable record:
- What was known at the start
- What was discovered (findings)
- What remained unknown
- How confidence changed

This is research provenance for AI-assisted work.

## Practical Application: When to Use This Approach

Empirica is most valuable when:

1. **Research involves synthesis** — combining sources with varying reliability
2. **Causal claims are tempting** — correlation is clear, mechanism isn't
3. **Multiple frameworks apply** — and they can't all be right
4. **Stakes matter** — the user will act on this information
5. **Uncertainty is decision-relevant** — knowing what you don't know changes the decision

It's less necessary for:
- Simple factual lookups
- Well-established procedures
- Cases where confidence is uniformly high or low

## The Takeaway

AI-assisted research is powerful but epistemically opaque. You can't see what the AI is confident about versus guessing at. Empirica makes that visible through:

- **Structured self-assessment** (13 vectors)
- **Bias correction** (calibrated against historical accuracy)
- **Explicit unknown tracking** (what hasn't been resolved)
- **Differentiated confidence** (not all claims are equal)

The research session produced interesting content about entheogenic entities. But more importantly, it produced an **honest map of certainty and uncertainty** — which is what you actually need to know whether to trust any of it.

---

## Getting Started with Empirica

```bash
# Create a session
empirica session-create --ai-id <your-ai-id> --output json

# Run preflight assessment
empirica preflight-submit --session-id <ID> --vectors '{"know": 0.5, ...}'

# Log findings and unknowns as you work
empirica finding-log --finding "..." --impact 0.7
empirica unknown-log --unknown "..."

# Check readiness
empirica check --session-id <ID>

# Complete with postflight
empirica postflight-submit --session-id <ID> --vectors '{"know": 0.7, ...}'
```

Full documentation: [Empirica on GitHub](https://github.com/Nubaeon/empirica#live-metacognitive-signal)

---

*The research session that generated this post explored entheogenic entity phenomenology — the spirits, beings, and archetypes reported across different psychedelic substances. The full session notes are available in the project repository. The epistemic assessment framework proved more interesting than the content itself: a tool for knowing what you know, and admitting what you don't.*
