---
name: smart-meter
description: Use when the user asks for a confidence or certainty check, says "fire up / use the smart meter", wants a gut-check before an irreversible or production action (sending email, deleting records, mutating a live sheet or CRM, deploying), is relying on aged cross-session memory, or wants the model to flag when it is not sure enough to proceed. Also use when the token budget matters - substantial or multi-step work, bulk/mechanical tasks (wide searches, batch edits, drafting, reading many files), or when the user mentions saving tokens, the weekly limit, or working economically. Do NOT use for casual conversation or simple self-contained lookups.
---

# Smart Meter

*Combines two axes that answer different questions: how sure am I (confidence), and how expensive should this be (model economy). Kept as one skill because the same per-turn stakes classification drives both.*

## Purpose
Append a read-out to responses where being wrong, or being needlessly expensive, would cost something. Surface the gap between local correctness and global coherence, between confident and irreversible, and between the task's real difficulty and the model tier being spent on it - so the operator can put attention and budget where they actually matter.

---

## Default output (read this first)

By default, append ONE line, all reasoning hidden:

```
Confidence NN% | bar BB% (reversible | irreversible) - clears | BELOW | model: Haiku|Sonnet|Opus | effort: low|medium|high
```

- **NN%** = confidence: weighted score (see Scoring below), rounded to the nearest 5%. Directional, not a calibrated probability.
- **BB%** = the bar this action is judged against: 70 reversible, 85 irreversible (set by the blast-radius check below).
- **verdict** = whether the score clears its bar.
- **model / effort** = the recommended tier for the CURRENT task, driven by the SAME stakes classification as BB - Haiku/low for routine/mechanical/read-only/bulk, Sonnet/medium for normal implementation, Opus/high for high-stakes/irreversible/ambiguous or below-bar. Not a readout of what the user is running - a recommendation to compare against it (same desktop UI picker switches both) and change if they differ.

Reveal the full assessment block ONLY when the operator asks, or the score is BELOW its bar.

---

## Acceptable bars (is the score good enough to act on?)

Stakes set the bar, not the score. Classify the action's blast radius first.

- **Irreversible / high blast radius: 85%.** Below this, stop, auto-expand, recommend a check before proceeding.
- **Reversible / local / read-only: 70%.** Below this, auto-expand and flag; above, proceed.

### Blast-radius check
Irreversible bar applies if ANY of these are true:
- sends email or messages to real people
- deletes or hard-deletes records, especially external systems
- mutates a live production sheet, funnel, or CRM
- deploys to prod
- runs under an auto-trigger / cron / watcher, OR performs an unbounded search/loop feeding any of the above

That last bullet is the amplifier - a small bounded operation feeding an auto-running loop is not bounded. Score the pipeline the code reaches, not just the diff in front of you.

### Gating rule
Unbounded AND irreversible AND external-write -> require explicit confirmation or a hard cap, even at high confidence. A deliberately bounded destructive action (one label, one thread, internal addresses refused) is correct and must not be penalised.

---

## The full assessment block (shown only on request or below bar)

```
---
**Smart Meter Assessment**

| Dimension | Weight | Score | Notes |
|-----------|--------|-------|-------|
| Global coherence | 35% | 0-100% | Fits the broader system? Independent supporting evidence? |
| Cross-reference completeness | 30% | 0-100% | Checked prior decisions AND verified any memory-named file/flag/function still exists? |
| Information accuracy | 20% | 0-100% | Facts/logic correct? |
| Retrieval completeness | 15% | 0-100% | Relevant context visible and applied, or truncated/missing? |

**Overall: NN%** (weighted sum, capped by evidence tier)

**Blast radius:** [reversible / irreversible] - [signals fired] - bar = [70 / 85]

**Recommended tier:** model: [X] | effort: [Y] - [one line why]

**Cheapest check to falsify this:**
- [fastest action that would prove this wrong]

**Flags:**
- [any dimension below 70%]
- [any memory fact acted on but not re-verified]
- [unbounded + irreversible + external-write -> gating applies]
```

---

## Scoring guidance

### Global coherence (35%)
90-100%: reviewed relevant earlier decisions, fits cleanly. 70-89%: broadly consistent. Below 70%: stop, restate invariants. **Independence check:** agreeing passes/sources sharing >50% basis get downgraded - shared-source convergence is not independent confirmation.

### Cross-reference completeness (30%)
90-100%: explicitly checked against session + memory. 70-89%: checked most relevant context. Below 70%: verify against earlier decisions. **Memory-staleness check:** a remembered file/flag/function/id may have been renamed or deleted since - verify before trusting it.

### Information accuracy (20%)
90-100%: stable facts, no recency risk. 70-89%: mostly confident, some uncertainty. Below 70%: flag and recommend verification.

### Retrieval completeness (15%)
90-100%: all relevant context visible and applied. 70-89%: some context may be missing. Below 70%: context-window pressure - flag and request what's needed.

### Evidence-tier ceiling
Cap the overall score by the weakest load-bearing evidence: direct verification this session up to ~95%; recent memory/docs up to ~85%; single unverified source up to ~70%; inference only up to ~55%.

### Weighting
Weighted sum, not average: coherence 35 / cross-ref 30 / accuracy 20 / retrieval 15. Blast radius is a SEPARATE stakes axis, deliberately not in the average - "how sure" and "how bad if wrong" stay distinct.

---

## Model-economy routing

The blast-radius classification above IS the router - reuse it, don't recompute a separate one. The tiers ship as **pinned agent definitions** (`agents/grind.md`, `agents/worker.md`, `agents/reviewer.md`) - dispatch the named agent and its model is fixed by the definition, so the cheap tier is guaranteed once you delegate rather than depending on remembering a per-call `model` param (an omitted param silently inherits the main-loop model - usually the most expensive one).

| Stakes | Main loop (user's own click) | Automatic action (yours) |
|---|---|---|
| Reversible + high confidence; mechanical / read-only / bulk | leave it; flag only if plainly overkill | dispatch the `grind` agent (pinned `haiku`) |
| Normal implementation, moderate stakes | leave it | dispatch the `worker` agent (pinned `sonnet`) for parallel/heavy read work |
| Irreversible, production, ambiguous, or confidence below bar | flag: worth being on Opus | dispatch the `reviewer` agent (pinned `opus`) before proceeding |

**Division of labour:** the user drives their own main-loop model - they switch it in the desktop UI (same picker for effort), not you. Your job is what a mouse-click cannot do: dispatch the pinned `grind`/`worker`/`reviewer` agents so each task runs on the right tier, and bring in the `reviewer` where stakes demand it.

### Delegation rules
- Offload grind to the pinned cheap agents - wide searches, bulk transforms, reading many files to summarise, first drafts -> dispatch the `grind` (`haiku`) or `worker` (`sonnet`) agent. Moves those tokens off the main loop, and the tier is fixed by the agent definition.
- Reserve the strongest tier for the review pass - the final adversarial review before a push, and reasoning on any irreversible step, go to the `reviewer` agent (pinned `opus`) even if the grind ran cheap.
- The pin is a default, not a cage - if a task is misclassified (looks like grind but needs real reasoning, or vice versa), override the pinned model with a per-invocation `model` on that one dispatch rather than forcing a too-cheap agent through work it will fail or take 3x the turns on.
- Keep subagent output tight - a subagent returns the conclusion, not file dumps.
- **Effort has two separate uses - don't conflate them.** (1) The read-out's `effort:` tag is a recommendation TO THE USER for their own session, exactly parallel to the model tag - genuinely actionable (same one-click picker as model), and NOT made redundant by adaptive thinking, which self-tunes reasoning depth on whatever is running - main loop or subagent alike - automatically, regardless of which effort level is set. (2) Separately, don't fuss over YOUR OWN effort turn-by-turn as if it were a lever you personally hold - the one place you do explicitly set `effort` is at subagent dispatch (`low` for a deliberately cheap mechanical stage, `high`+ for the hardest verify stage). Neither use makes the other pointless.

### Gating rule (never economise into a mistake)
If the step is **irreversible** or the score is **below the bar**: do NOT let a cheap agent's first pass be the last word. Dispatch the `reviewer` agent (pinned `opus`) to check it. Never route a send/delete/deploy/live-data mutation to a cheap model and proceed unreviewed.

### Mid-build escalation (the hard ceiling, and the compromise) - symmetric, both directions
No skill or hook can switch the main-loop model itself, not even mid-task, and the read-out only bookends a turn (fires once at response-end) - so on a long multi-step build it will NOT catch a stakes change the moment it happens, in EITHER direction. Token overuse is this skill's own risk axis, not a lesser one than correctness - a long undelegatable stretch burning Opus is exactly the failure mode this exists to catch, so it gets the same active stop as an upward mismatch, not a passive note.

**Needs MORE power mid-build:** STOP before executing that step. In your own text: (1) name what changed and why this step is now Opus/high-effort territory, (2) recommend switching now, (3) offer the fallback of an `opus` review agent if the user would rather not context-switch.

**Needs LESS power mid-build:** (1) delegate first - if the stretch can run as a subagent, dispatch it to the `grind`/`worker` agent right then, no user click needed; (2) only when delegation genuinely isn't possible (needs the full running context you hold), STOP exactly as in the upward case and recommend dropping a tier now.

This is the honest ceiling, not a workaround settled for - the most automated version of "tell me when to switch" current tooling allows in either direction. Everything around the stop is automatic (subagent tiering, classification); the click is the one lever only the user holds.

---

## Key failure modes this skill is designed to catch
1. **Local vs global disconnect** - correct in isolation, contradicts an earlier decision.
2. **Confident, irreversible action** - high knowledge-confidence on a send/delete/deploy with no undo.
3. **Amplification/cascade** - a small operation feeding an auto-trigger or unbounded loop.
4. **Memory staleness** - acting on a remembered file/flag/function that no longer exists.
5. **Confident wrongness** - the falsify-check forces an adversarial counter-check, not reassurance.
6. **Token overuse** - grinding an expensive model through routine or bulk work that a cheap subagent could handle.
7. **Undersized reasoning** - staying on a cheap model/tier through a step that just turned high-stakes.

---

## Common mistakes
- **Building switch crutches.** Slash commands wrapping `/model` are redundant - the user already has a one-click picker in the desktop UI. This is about automatic delegation and honest flagging, not manual switching.
- **Suggesting `/fast` to save tokens.** `/fast` is Opus with faster output - same cost, not an economy lever.
- **A cheap agent's unreviewed first pass before an irreversible action.** The gating rule forbids it even at high confidence.
- **Treating downward tier mismatches as lower-priority than upward ones.** Token spend is this skill's own risk axis, not a lesser version of correctness risk - it earns the same active stop when delegation isn't possible.
- **A lean main loop swallowing huge subagent dumps.** Defeats the point; demand tight conclusions.

---

## When to apply this skill
Apply when: the action touches the irreversible bar; decisions depend on earlier context in a long session or aged memory; output will be acted on without review; substantial or multi-step work where model/effort tier matters.

Do NOT apply to casual conversation, simple factual lookups, or short self-contained tasks where confidence is obviously high or low and the task is trivially cheap.
