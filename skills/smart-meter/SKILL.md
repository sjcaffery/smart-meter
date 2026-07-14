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

Reveal the full assessment block ONLY when the operator asks, or the score is BELOW its bar. When BELOW, the verification gate below is mandatory before the response proceeds - a red score is an instruction to verify, not a caveat to attach and move past.

---

## Below-bar protocol (mandatory verification gate with behavioral forcing)

BELOW is not a label you attach and carry on past - it changes what you are allowed to do next. While the score is under its bar you **MUST** execute this sequence before proceeding:

### Behavioral directives (non-negotiable):
1. **MUST auto-expand the assessment block** - do not proceed with one-line meter only
2. **MUST trigger research/lookup** (see Research Trigger section below):
   - Search memory for relevant prior learning on this task type (e.g., `[[20-questions-strategy]]`, `[[code-review-patterns]]`, `[[estimation-methods]]`)
   - If memory exists: apply it immediately to reconsider your approach
   - If memory doesn't exist: use WebSearch or Agent to research best practices for this specific problem
   - Document findings in memory for cross-session reuse
3. **MUST reconsider your approach** - ask clarifying questions, shift strategy, or explicitly state why you are proceeding despite low confidence
4. **MUST NOT assert a conclusion, make a guess, or take an irreversible action** until verification is complete
5. **If irreversible (85%): CANNOT proceed without explicit user approval**

### Verification steps (when research is complete):
1. **Name the load-bearing assumption** - the single belief that, if wrong, collapses the answer.
2. **Verify it, do not restate it** - re-read the actual inputs, research an external source, widen the candidate set, or apply domain strategy from research. Confidence may rise ONLY from a check that could have come back the other way; repeating the same unchecked belief in more confident words is not verification and must not move the score.
3. **State the corrective move** - what the check changed, or that it held - then re-score. If the check moved you, the next output reflects it.

The commonest way a below-bar turn goes wrong is a single unverified inference silently promoted to fact (failure mode 5). The bar being red is the trigger to test that inference, not to hedge it and proceed.

**This is the point of the skill on big multi-step client builds.** One unchecked assumption early - a column that moved, an API whose shape changed, a requirement read too narrowly - propagates through every later step and is expensive to unwind. Below bar, stop and verify the assumption before building anything on top of it. Do not let momentum substitute for a check.

---

## Research trigger (when BELOW bar)

Low confidence signals a knowledge gap. Research closes it and compounds across sessions.

### Trigger (automatic when BELOW):
1. **Search memory first** — look for a related memory file on this task type:
   - Pattern-matching: if guessing games, search for `[[20-questions-strategy]]`; if code-reviewing, search for `[[code-review-patterns]]`; if estimating, `[[estimation-methods]]`
   - Memory naming convention: task_type-knowledge or task-strategy
   - Apply any memory directly to reconsider the current approach
2. **If memory missing: research the gap** — use WebSearch or an Agent to fetch domain strategy/best practices for this specific problem type
   - Example: low confidence on game-guessing → search "20 questions strategy" → find genre-branching heuristics → apply to next questions
   - Example: low confidence on code architecture → search "microservice patterns" or "API design" → bring findings into design reconsideration
3. **Document findings in memory** — store learnings under a persistent name so they compound:
   - Format: `task-type_knowledge.md` or `task-strategy_name.md` with the `[[name]]` registered in MEMORY.md
   - Findings accessible in future sessions/projects on the same task type
   - Over time, confidence on repeated task types should rise as memory builds

### Persistence across sessions:
- Low-confidence patterns trigger research → findings saved to memory → reused in later sessions
- Same session: confidence may rise after research; re-score and output the improved meter
- Later sessions: consult memory FIRST before doing new research — reuse validated learnings
- Pattern: task_type_method → research once → memory → apply every session thereafter

### Gating rule:
Do not proceed with BELOW score until research is complete and verification is done. The research may raise confidence above bar; if it does, re-score and proceed. If it remains BELOW after research + verification, halt irreversible actions and ask for user approval.

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
- **Only delegate grind big enough to clear the hand-off cost.** A fresh subagent pays a cold start - its system prompt + tools are re-cached from scratch on every dispatch - so on a *small* grind that fixed cost cancels the cheaper-tier saving and the delegation nets roughly zero. Offload when the grind is genuinely bulky (many files, a long multi-step draft); for a handful of files or a quick transform, do it inline on the current model. *(Measured: a 10-file summarise delegated to `grind`/haiku came out a dead heat with doing it inline - the second agent's startup ate the saving.)*
- **Price the turns, not just the per-token rate.** A cheaper model routinely takes 2-6x the turns on multi-step work, and every extra turn re-reads the growing context - so a 'cheaper' tier can burn *more* total tokens and wall-clock. Delegate the doing to the cheapest model that will still finish in a comparable number of turns; don't push reasoning-heavy or from-scratch build work down to `haiku` expecting a proportional saving. *(Measured: an Opus build finished in ~7 turns; the same build delegated to `worker`/sonnet took ~29 turns, so its 2.5x-cheaper rate netted only ~20% off the doing - before any review.)*
- **The cheapest win is usually the main loop itself, not delegation.** Keeping an expensive main model resident *and* spawning subagents pays twice. When a *whole* task is routine, the highest-leverage move is the read-out's `model:` recommendation - drop the main loop to Sonnet/Haiku - not delegating from Opus. Delegation earns its keep only when the main loop must stay high-tier for the *surrounding* reasoning while a large sub-chunk is genuinely mechanical.
- Reserve the strongest tier for review **only when the stakes call for it** - an irreversible / high-blast-radius step (the gating rule below) or a below-bar confidence score: send reasoning and the final check to the `reviewer` (`opus`) even if the grind ran cheap. **Do not bolt a review pass onto reversible, low-stakes work** - re-reading a whole clean artifact at the top tier changes nothing and is pure spend. Let the blast-radius classification decide: if the action clears the reversible bar, ship the cheap draft unreviewed. *(Measured: an `opus` review of a clean, reversible build added ~1.4x the build's own cost and found nothing to change.)*
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

## Production work gates (ABC/Fletchers/TEFL)

These gates apply to work touching **live sheets, APIs, crons, webhooks, or external systems** where incomplete state assumptions propagate into unrecoverable errors.

### Gate 1: State-dependent work detection
**Flag if:** The task reads/writes sheets, APIs, configs, crons, or webhooks.  
**Action:** Require "enumerate candidates" step before implementation.
- Example: "Which sheets? Which columns? Which APIs? Which scripts already touch this?" List them explicitly.
- Then verify ALL exist and are current with `Read`/`Grep` before writing code.

### Gate 2: Candidate space verification
**Flag if:** Narrowing down to a specific implementation (e.g., "auto-send on booking").  
**Action:** Enumerate ALL candidates (existing systems, constraints, edge cases) before proceeding.
- Example failures caught:
  - [AIB sheet internals](reference_aib_sheet_internals.md): May/June 45-wide vs Jan-Apr 44-wide. Assume = break.
  - [Booking Welcome = manual paid gate](reference_booking_welcome_paid_gate.md): Form submit ≠ payment. Assume = send to non-payer.
  - [Switchboard clobber](project_2026_07_04_switchboard_clobber_incident.md): Two projects with same name. Enumerate first.

### Gate 3: Memory staleness check
**Flag if:** Citing a memory fact about system state (e.g., "AIB has X columns", "Cron runs daily at 6am").  
**Action:** Re-verify the memory against current state before depending on it.
- Use `Read` on the actual sheet/config/code. Don't assume memory is up-to-date.
- Example: Sales Dashboard wiring changed month columns (June=R, July=T). A stale assumption breaks the formula.

### Gate 4: Integration point inventory
**Flag if:** The work touches >1 system (sheet + API + cron + webhook) or depends on overlap.  
**Action:** Before implementation, map which scripts/processes touch each system and whether they conflict.
- Example: Post-deposit Portal + Mike's Final Payments CRM both onboard/handover/Wise-payment. Read both designs first.
- Prevents: Duplicate work, overwriting fields, version mismatch on shared data.

### Gate 5: Assumption inventory upfront
**Flag if:** About to write code that depends on unverified system facts.  
**Action:** BEFORE implementation, list load-bearing assumptions explicitly and verify each.
- Format: "Assumption: X. Verify: Y. Status: [verified/UNVERIFIED]."
- Do not proceed with unverified assumptions.
- Example assumptions:
  - "Column R is always 'Status'" — verify by reading the sheet
  - "Payment confirms before Booking Welcome applies" — verify against workflow design
  - "Cron runs daily at 6am" — verify in Apps Script trigger settings

### Gate 6: Spot-check rule
**Flag if:** Any change affects >1 record/row/candidate/email.  
**Action:** Before deploying, spot-check 3-5 actual records/rows to catch off-by-one, header mismatch, layout shift.
- Example: Leads quota incident (2026-07) — widened search without testing. Spot-check: fetch 5 threads, confirm they match the query.
- Prevents: Silent failures where query is wrong but code looks right.

---

## Key failure modes this skill is designed to catch
1. **Local vs global disconnect** - correct in isolation, contradicts an earlier decision.
2. **Confident, irreversible action** - high knowledge-confidence on a send/delete/deploy with no undo.
3. **Amplification/cascade** - a small operation feeding an auto-trigger or unbounded loop.
4. **Memory staleness** - acting on a remembered file/flag/function that no longer exists.
5. **Confident wrongness** - the falsify-check forces an adversarial counter-check, not reassurance. Below bar, this hardens into the verification gate: name the load-bearing assumption and actually test it before proceeding.
6. **Token overuse** - grinding an expensive model through routine or bulk work that a cheap subagent could handle.
7. **Undersized reasoning** - staying on a cheap model/tier through a step that just turned high-stakes.
8. **Incomplete candidate space** (new for production work) - narrowing without enumerating all candidates first. See Gate 2.
9. **State assumption not verified** (new for production work) - proceeding on cached knowledge of system state. See Gates 3 & 5.
10. **Passive transparency, no behavior change** (new) - reporting BELOW but not forcing reconsideration, research, or verification. The meter must drive action, not just display.
11. **Research skipped when BELOW** (new) - no domain knowledge lookup when confidence is low. Low confidence = knowledge gap = should research before proceeding.
12. **Research not persisted** (new) - findings from low-confidence research vanish instead of building in memory for cross-session reuse.
13. **Proceeding past BELOW without approval** (new, irreversible) - asserting conclusions or taking irreversible actions while still BELOW bar. The behavioral MUST directives prevent this.

---

## Common mistakes
- **Building switch crutches.** Slash commands wrapping `/model` are redundant - the user already has a one-click picker in the desktop UI. This is about automatic delegation and honest flagging, not manual switching.
- **Suggesting `/fast` to save tokens.** `/fast` is Opus with faster output - same cost, not an economy lever.
- **A cheap agent's unreviewed first pass before an irreversible action.** The gating rule forbids it even at high confidence.
- **Treating downward tier mismatches as lower-priority than upward ones.** Token spend is this skill's own risk axis, not a lesser version of correctness risk - it earns the same active stop when delegation isn't possible.
- **A lean main loop swallowing huge subagent dumps.** Defeats the point; demand tight conclusions.
- **Delegating trivial grind.** A subagent for a handful of files or a quick transform costs more in cold-start than it saves - do small grind inline; delegate only genuinely bulky work.
- **Reviewing reversible work.** The gating rule reserves the `reviewer` pass for irreversible / below-bar steps. A review bolted onto a reversible change is budget spent for no risk reduced.
- **Reporting BELOW and carrying on.** A red score with no verification step is the skill failing silently. Below bar, the verification gate is not optional - assumption named, checked, corrected - before the response asserts or acts.
- **Narrowing without enumerating.** (Production work) Confident that a feature exists or works a certain way without checking all candidates first. Incomplete enumeration = incomplete candidate space. Always list them explicitly, then verify.
- **Skipping memory staleness checks.** (Production work) Citing aged memory about system state (sheet layouts, API shapes, cron schedules) without re-verifying against current state. Memory can rot in days if the system changed.
- **Proceeding on unverified assumptions.** (Production work) Starting implementation before naming and checking load-bearing assumptions. A single wrong assumption early propagates through every downstream step and is expensive to unwind.
- **Reporting BELOW without forcing behavior change.** Outputting "Confidence 42% | bar 70% | BELOW" and then proceeding as if the meter were just a label. The BELOW signal MUST trigger reconsideration, research, and verification before proceeding. Transparency without action is the meter failing silently.
- **Skipping research when BELOW.** Low confidence = knowledge gap. Not researching domain strategy or prior learning leaves the gap unfilled. Memory + WebSearch close it; proceeding without is betting on unvalidated reasoning.
- **Research findings not persisted.** Finding a 20-questions strategy that improves your game, then discarding it instead of saving to memory. Learnings must accumulate across sessions; one-off research is wasted.
- **Proceeding past BELOW without approval (irreversible).** Taking a send/delete/deploy action while confidence is still 45% and below the 85% irreversible bar. The CANNOT directives prevent this; they are not advisory.

---

## When to apply this skill
Apply when: the action touches the irreversible bar; decisions depend on earlier context in a long session or aged memory; output will be acted on without review; substantial or multi-step work where model/effort tier matters.

Do NOT apply to casual conversation, simple factual lookups, or short self-contained tasks where confidence is obviously high or low and the task is trivially cheap.
