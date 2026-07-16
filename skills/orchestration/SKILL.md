---
name: orchestration
description: Relative-tier routing doctrine — how an agent running ANY model decides when to advise UP to a smarter/higher-taste model and when to delegate DOWN to a cheaper one, all relative to the caller's own seat on the model ladder. USE WHEN delegating implementation, deciding whether to consult an advisor, choosing which model to escalate to or delegate to, writing a spec for a subagent, judging your own model against a task, or managing session cost.
---

# Orchestration — routing relative to your own seat

You are running _some_ model. It sits somewhere on a ladder of models that differ on three axes —
**intelligence**, **taste**, **value** (the full table is [references/models.md](references/models.md)).
Good orchestration is one idea: know where you sit, then move work in the two directions your seat allows.

- **Advise up** — hand _judgment_ to a model that beats you on the axis a decision threatens.
- **Delegate down** — hand _determined volume_ to the cheapest lane that's still adequate.

These are orthogonal. Delegation is about **cost for work whose outcome the spec already fixes**.
Escalation is about **judgment for work whose outcome is still open**. The same model can be both your
delegate and your advisor depending on which kind of work it is — GPT-5.6 is the cheap bulk lane _and_
a valid cross-vendor correctness check.

There is one skill and two directions — not a separate skill or agent per model. To reach a **Claude**
model you just spawn a subagent with its `model:` set. Only the **external CLI** is packaged as an
agent, because it wraps real Bash: the `codex` agent (GPT-5.6).

## Step 1 — find your seat

Read [references/models.md](references/models.md) and locate your own model. Note your three scores.
Everything below is relative to them:

- Models **above you on intelligence** → escalation targets for hard reasoning.
- Models **above you on taste** → escalation targets for user-facing judgment (UI, copy, API design).
- Models **cheaper than you** with **adequate** intelligence/taste for a _determined_ task → delegation targets.
- A model from a **different family** → a cross-vendor check regardless of its scores.

## Step 2 — delegate down (volume)

Route work whose outcome the spec fully determines — boilerplate, wiring, CRUD, mechanical edits,
straightforward features — to the **cheapest adequate lane**. You verify the result anyway, so you're
buying typing, not judgment. _Adequate_ = intelligence high enough that the spec carries the task, **and**
taste ≥ 7 if the output is user-facing (keep user-facing surfaces off the sub-7 lanes).

| Lane              | Producer                                                | How to invoke                                                           | Route here when                                                                                                                                                            |
| ----------------- | ------------------------------------------------------- | ----------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **codex**         | GPT-5.6 (intel 7, value 8, taste 5), run **low** effort | `Agent(subagent_type: codex)` with the five-part spec                   | **Default** for determined volume. Cheap and smart at low effort, and a non-Anthropic family for a free cross-vendor diff. Not for taste-critical surfaces.                |
| **Claude-family** | Sonnet 5 (taste 7) / Haiku 4.5                          | Spawn a general subagent with `model: sonnet`, low effort, and the spec | Determined work that's _lightly_ user-facing (Sonnet's taste 7 clears the bar), Claude-family consistency, or the CLI down. Never Haiku unless you have a specific reason. |

Deciding rule: how much does the outcome depend on judgment the spec can't capture? Little → the default
codex lane; you verify anyway. A lot, and mistakes are costly → keep that piece yourself and escalate the
judgment (Step 3), or run codex at high effort on a tightened spec and review the diff closely.

**Reasoning effort tracks direction.** Delegating determined volume → **low** effort (you verify anyway;
extra reasoning is wasted tokens). Bump a lane to high only when a specific implementation is
correctness-critical (concurrency, money, migrations). Escalation, by contrast, always runs high (Step 3).

If the codex lane returns `unavailable`/`timeout`, re-route the same spec to a Claude subagent and
**say so in your report** — never silently absorb the substitution.

## Step 3 — advise up (judgment)

Consult an advisor when a decision's outcome turns on an axis where a higher seat beats you. Match the
advisor to the **threatened axis**, not to "the smartest model available":

| Threatened axis                                                                   | Advise up to                                        | How to invoke                                                                                                     |
| --------------------------------------------------------------------------------- | --------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| **Hard reasoning / correctness** — near your intelligence ceiling, or stuck twice | Fable 5 (or Opus 4.8 if you don't need the ceiling) | Spawn a general subagent with `model: fable` (or `opus`), high effort + the advisor prompt below                  |
| **Taste** — UI, copy, API shape, and your taste < 7                               | Opus 4.8 (taste 8) or Fable 5 (taste 9)             | Spawn a general subagent with `model: opus` (or `fable`), high effort + the advisor prompt below                  |
| **Independent check** — a second opinion from outside your family                 | GPT-5.6, read-only                                  | `Agent(subagent_type: codex)` with a decision-and-options prompt (it runs read-only, high effort, in advise mode) |

Advisors always run at **high** reasoning effort — escalation is the moment you're paying for judgment, not saving on volume.

Consult at the moments that decide whether the next hour is wasted:

- Before committing to an architecture, data migration, API shape, or refactor strategy.
- Whenever the same problem has resisted two distinct attempts.
- Once before declaring a multi-step deliverable done.

Pass the advisor the decision, the constraints, and the options considered. Act on the verdict or surface
the disagreement — never silently ignore it.

### Advisor prompt template (for a Claude fable/opus subagent)

Spawn the subagent with the target `model:` and this prompt. It is read-only by contract — the template
forbids editing:

```
You are a second-opinion advisor, consulted at a commitment boundary. You are
read-only: do not edit, create, or write any files.

DECISION: <the decision being made>
CONSTRAINTS: <what can't change>
OPTIONS CONSIDERED: <the options on the table>

1. Look before you opine — if the decision depends on how the code actually
   works, read it; don't reason from my summary.
2. Give a verdict, not a survey: "do X, not Y, because Z", and name the single
   risk that decides it.
3. A sound plan gets one line ("sound; watch X"). Don't manufacture objections.
4. If information you don't have would change the answer, name it precisely.
5. Stay under 300 words. Your reader is a model mid-task, not a human.
```

## Step 4 — the per-seat playbook

Your seat sets the defaults. Override any of them when the output bar demands it (see cost discipline).

| Your seat          | Escalate up?                                                                                                                                               | Delegate down?                                                                                                                                |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| **Fable** (I9 T9)  | Almost never — only a cross-vendor correctness check (`codex` advise) on high-stakes reasoning. Nothing beats you on judgment.                             | **Almost everything.** You're the priciest lane per token; every line of code you type is waste. Emit specs and verdicts, delegate the rest.  |
| **Opus** (I6 T8)   | To Fable for the hardest judgment or top-taste calls; cross-vendor for correctness.                                                                        | **Freely** — most work is below your pay grade. Default determined volume to `codex`.                                                         |
| **Sonnet** (I4 T7) | **Often.** Hard reasoning → codex-advise / Opus / Fable. Top-tier taste → Opus / Fable. You sit low on intelligence — most non-trivial judgment beats you. | **Only genuinely determined work**, and your best target is `codex` (cheaper _and_ smarter than you). Keep the judgment; delegate the typing. |
| **Haiku** (I2 T4)  | **Constantly** — almost any judgment call exceeds you. If you're the session model, most decisions want a higher seat.                                     | Rarely — little worth the overhead of delegating below you.                                                                                   |

The counter-intuitive case: a **Sonnet** session delegating bulk work to **GPT-5.6** is delegating _down
in cost_ to a model that is _up in intelligence_. That's correct. Delegation follows cost for determined
work; it is not a claim the target is dumber.

## Cost discipline — the prime directive

Your session model is rarely the cheapest lane in the system. The economic case for orchestrating at all
is keeping your own token volume low. Three rules:

**Emit judgment, not volume.** Your output is decomposition, specs, routing decisions, verdicts on diffs,
and short reports — not implementation code, test bodies, boilerplate, or config. A code block longer than
an interface signature is a spec you haven't delegated yet. Fixing a lane's bug by hand is the same failure
in disguise: send a corrected spec back to the cheap lane.

**Keep the context lean.** Everything in your context is re-read at your model's price every turn. Delegate
broad exploration, codebase searches, and log-grepping to a cheap read-only agent and keep only the
conclusions. Read files yourself only when the decision genuinely depends on the exact code.

**Reason once, then hand off.** Do the hard thinking in one pass, capture it in the spec, let the cheap lane
carry it. Re-deriving decisions across turns burns the premium twice.

These bind hardest at the top of the ladder and relax as you descend — a Fable session that types code
wastes the most expensive tokens in the system; a Sonnet session has less premium to protect but should
still push determined volume to the cheap lanes.

## The spec contract

Delegated subagents share none of your conversation context. Every delegation prompt carries all five parts:

1. **Objective** — what to build or change, one paragraph.
2. **Files** — exact paths to create or modify.
3. **Interfaces** — signatures, types, or API shapes the code must match.
4. **Constraints** — project conventions, things not to touch.
5. **Verification** — the command(s) that prove it works.

A spec you can't finish writing is a signal the decision isn't made yet — that's judgment work (keep it, or
escalate it), not a reason to hand ambiguity to a cheaper model.

## Parallelism and racing

Independent specs (no shared files, no ordering dependency) launch as parallel agents in one message.
Sequential chains and single-file surgery stay serial. For high-stakes determined work, get a second
diff — run `codex` and a Claude subagent on the same spec — and judge the stronger result yourself: an
independent non-Anthropic implementation plus your own review is two perspectives for one extra lane's cost.

## Verification

Reports are claims, not evidence. Before accepting any lane's work: read the diff, and re-run the
verification command (or spot-check its quoted output against the working tree). "Should work", "tests
should pass", or a report with no command output means the task is not done. A lane that reports a spec gap
gets a corrected spec, not a "use your judgment".
