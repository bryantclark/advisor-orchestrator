# Advisor Orchestrator

**Whatever model you're running, route work to the right tier: advise up for judgment, delegate down for volume.**

Claude Code lets every subagent run on a different model — and lets the session itself run on a different model than its subagents. Most model-routing setups hard-code a hierarchy: _the session is always the smartest model, and it always delegates down._ This plugin drops that assumption. Routing is **relative to the caller's own seat** on a ranked model ladder, so the same doctrine works whether your session is Fable, Opus, or Sonnet:

- **Advise up** — hand _judgment_ to a model that beats you on the axis a decision threatens (hard reasoning → higher intelligence; UI/copy/API → higher taste; correctness → a different vendor).
- **Delegate down** — hand _determined volume_ (boilerplate, wiring, CRUD, mechanical edits) to the cheapest lane that's still adequate.

The two are orthogonal. Delegation is about **cost for work the spec already fixes**; escalation is about **judgment for work that's still open**. A Fable session escalates almost never and delegates almost everything. A Sonnet session escalates _often_ — most non-trivial judgment beats it — and delegates only genuinely determined typing. Same rules, different seat.

## The model ladder

Every routing decision reads from one ranked table ([`skills/orchestration/references/models.md`](skills/orchestration/references/models.md)). Three axes, 1–10, from the operator's seat — not list price or public benchmarks:

| model            | family    | intelligence | taste | value (cheapness to you) | reason@ |
| ---------------- | --------- | ------------ | ----- | ------------------------ | ------- |
| **fable-5**      | Anthropic | 9            | 9     | 2                        | low     |
| **gpt-5.6-sol**  | OpenAI    | 7            | 5     | 8                        | medium  |
| **opus-5**       | Anthropic | 6            | 8     | 5                        | medium  |
| **sonnet-5**     | Anthropic | 4            | 7     | 5                        | low     |
| **gpt-5.6-luna** | OpenAI    | 4            | 4     | 10                       | high    |
| **haiku-4.5**    | Anthropic | 2            | 4     | 8                        | low     |

The table is the tuning surface — **edit it** when your plans, bills, or judgment of a model change, and the doctrine follows. (`intelligence` = how hard a problem you'd hand it unsupervised; `taste` = UI/UX, code quality, API design, copy; `value` = how cheap it is _to you_ in practice, generous plan limits included; `reason@` = the reasoning effort where it's best value on delegated work.)

The anomaly that drives the whole system: **GPT-5.6-sol is strong-intelligence _and_ one of the cheapest lanes you have** (generous CLI limits). That makes it the best _delegation_ target for bulk work — and it doubles as an _escalation_ target for cross-vendor correctness. Its sibling **luna** is cheaper still but dumber, a bottom-of-ladder lane for the most trivial rote work. Both sit below the taste-7 bar, so neither touches user-facing surfaces; that's what Opus and Fable are for.

**Reasoning is a second knob.** Effort tracks direction: delegated volume runs **low** (you verify anyway — extra reasoning is wasted tokens), escalation runs **high** (you're buying judgment).

## What ships

- **One skill — `orchestration`.** The whole doctrine: find your seat, the two directions, the per-seat playbook, the five-part spec contract, cost discipline, and verification. It also carries the advisor prompt template, so escalating to a Claude model needs no extra file — you just spawn a subagent with `model: fable`/`opus`.
- **One thin CLI agent.** The only lane that needs packaging, because it wraps real Bash:
  - [`agents/codex.md`](agents/codex.md) — **GPT-5.6** via the OpenAI Codex CLI, dual-mode: `IMPLEMENT` (`codex exec`, workspace-write) for the default cheap volume lane, and `ADVISE` (`codex exec -s read-only`) for a cross-vendor second opinion.

Claude-family targets (fable/opus/sonnet/haiku) need no agent file — the Agent tool's `model` parameter reaches them directly.

## Install

```
claude plugin marketplace add bryantclark/advisor-orchestrator
claude plugin install advisor-orchestrator
```

Then run your session on whatever tier fits the work — the doctrine adapts to it. For an architect-heavy workflow, start high:

```
/model fable
```

## Use it

Just ask for work; the orchestration skill routes it relative to your seat:

```
Add rate limiting to our public API. Design it, delegate the
implementation, and verify the evidence before you call it done.
```

A Fable session writes the spec, delegates the implementation to `codex` (consulting Opus first on the concurrency-sensitive design), reads the diff and re-runs verification, and only then reports done. A Sonnet session running the same request leans harder on escalation — consulting Opus on the limiter's design before committing — because more of that decision sits above its seat.

To make the doctrine always-on, add one line to your project's `CLAUDE.md`:

```
Route work through the orchestration skill: advise up for judgment you can't
make well from your own model tier, delegate determined volume down to the
cheapest adequate lane, and verify evidence before accepting any lane's report.
```

## Requirements

- **Claude Code** recent enough for subagent model routing (CLI, desktop, VS Code, web). Subagent model routing is **Claude Code only** — this does nothing on claude.ai chat.
- **Claude lanes (fable/opus/sonnet/haiku):** a subscription that includes the tiers you route to. A pinned Claude model that isn't on your plan **silently falls back** to the session model — the pattern degrades quietly rather than erroring, so check your plan if results feel unremarkable.
- **codex lane (GPT-5.6):** the [OpenAI Codex CLI](https://github.com/openai/codex) installed and authenticated (`npm i -g @openai/codex`, then `codex login`). Without it the agent reports `STATUS: unavailable` — never a silent Claude fallback.

Model resolution order in Claude Code: `CLAUDE_CODE_SUBAGENT_MODEL` env → per-invocation `model` parameter → agent frontmatter → session model.

## FAQ

**Is this Anthropic's "advisor tool"?** No — that's a server-side API feature. This is plain Claude Code subagents plus a skill: readable, editable, no beta flags.

**Why an external GPT-5.6 lane in a Claude plugin?** Vendor diversity, and cost. Models from one family share blind spots; an independent implementation from a different lineage catches what same-family review misses — and with Claude as the reviewer, every codex diff gets cross-vendor review for free. It's also among the cheapest lanes you have, which is why it carries the bulk volume.

**Why not just run everything on Fable?** You can — it's excellent, and the most expensive lane per token. Most of a session's tokens are implementation mechanics the cheap lanes handle at near-parity. Spend the premium where judgment lives.

**Lineage.** Generalized from [DannyMac180/fable-advisor](https://github.com/DannyMac180/fable-advisor), which fixed the session at Fable and always delegated down. This fork makes the routing relative to any seat and collapses the per-model agents into one skill plus the codex CLI lane.

## License

MIT
