# The model ladder

The registry every routing decision reads from. Three axes, all 1–10, all from the
operator's seat — **not** list price or public benchmarks. Edit this table when your
plans, bills, or judgment of a model change; the doctrine reads from it.

- **intelligence** — how hard a problem you can hand the model *unsupervised* and trust the result.
- **taste** — UI/UX, code quality, API design, and copy. The axis that decides user-facing work.
- **value** — how cheap it is *to you* in practice (generous plan limits count), higher = cheaper. This is a tie-breaker, never the deciding axis for anything that ships.
- **reason@** — the reasoning-effort setting where the model earns its keep on *delegated* (determined) work. Most models are best *value* at **low** effort — they're good enough without burning the extra tokens/latency. Grok 4.5 is the exception: it's meaningfully better with more thinking, so low-effort grok is a false economy. (Escalation overrides this — see "Reasoning effort" below.)

| model | family | intelligence | taste | value | reason@ | access |
|---|---|---|---|---|---|---|
| **fable-5** | Anthropic | 9 | 9 | 2 | low | Agent/Workflow `model: fable` |
| **gpt-5.5** | OpenAI | 7 | 5 | 8 | low | Codex CLI — `codex exec` / `codex exec -s read-only` |
| **opus-4.8** | Anthropic | 6 | 8 | 5 | low | Agent/Workflow `model: opus` |
| **grok-4.5** | xAI | 6 | 6 | 9 | **high** | Grok CLI — `grok --prompt-file … -m grok-4.5 --effort high` |
| **sonnet-5** | Anthropic | 4 | 7 | 5 | low | Agent/Workflow `model: sonnet` |
| **haiku-4.5** | Anthropic | 2 | 4 | 8 | low | Agent/Workflow `model: haiku` — **avoid** |

Sorted views:

- **intelligence:** fable (9) > gpt-5.5 (7) > opus (6) ≈ grok (6) > sonnet (4) > haiku (2)
- **taste:** fable (9) > opus (8) > sonnet (7) > grok (6) > gpt-5.5 (5) > haiku (4)
- **value:** grok (9) > gpt-5.5 (8) ≈ haiku (8) > opus (5) ≈ sonnet (5) > fable (2)

## Reasoning effort

Effort is a second knob on every lane, and it interacts with **direction**, not just the model:

- **Delegating determined volume → low effort.** You verify the result anyway, so extra reasoning buys
  little and costs tokens and latency. Low is the value play. (Bump to high only when a specific
  implementation is correctness-critical — concurrency, money, migrations.)
- **Escalating for judgment → high effort.** Advice, hard reasoning, stuck-twice: you're buying quality,
  not volume. Run the advisor at high.
- **Per-model sweet spot overrides direction.** `reason@` in the table wins when it disagrees: **Grok 4.5
  runs high even as a delegation lane** — it's where it's actually better — while the others stay at low
  for volume and only climb when you're escalating.

## Why the numbers land where they do

- **fable-5** — top of both judgment axes, and the most expensive lane per token. Its whole
  economic point is to emit judgment, never volume.
- **gpt-5.5** — the anomaly that drives the whole system: high intelligence *and* one of the two
  cheapest lanes (generous Codex limits). Low taste keeps it off user-facing surfaces. It is
  simultaneously a top **delegation** target (cheap bulk work at low effort) and a valid
  **escalation** target (cross-vendor correctness check, run high). Reachable only through the Codex CLI.
- **opus-4.8** — second-highest taste, strong intelligence, mid cost. The default *advisor*
  when you don't need fable's absolute ceiling, and a fine architect seat.
- **grok-4.5** — "Opus-class" reasoning (Artificial Analysis Intelligence Index 54), ~4× more
  token-efficient than Opus 4.8 on the same tasks, and the **cheapest lane in the system** via the
  Grok CLI. The reasoning outlier: it's meaningfully better at **high** effort, so unlike the other
  lanes it's worth running high even for delegated volume. Taste (6) sits above gpt-5.5 but below
  Sonnet — decent, but under the taste-7 bar, so it's a cross-vendor implementation lane, not a taste
  seat. The taste number is the operator's estimate, not a benchmark; recalibrate it with use.
- **sonnet-5** — modest intelligence but genuinely good taste (7). Sits low on the ladder: much
  above it, little below. As a session it escalates often and delegates only determined volume.
- **haiku-4.5** — ~90% of Sonnet on agentic coding at a third of the price, but the operator's
  standing rule is **never use Haiku**. Listed for completeness; not a routing target.

## Access mechanics (how each lane is actually reached)

- **Claude family (fable/opus/sonnet/haiku)** — the Agent tool's `model` parameter or a
  workflow agent's `model`. Resolution order: `CLAUDE_CODE_SUBAGENT_MODEL` env → per-invocation
  `model` → agent frontmatter → session model. A pinned Claude model that isn't on your plan
  **silently falls back** to the session model — the pattern degrades quietly, so check your plan
  if results feel unremarkable.
- **gpt-5.5** — only through the Codex CLI. A thin Claude wrapper agent (`model: sonnet`) writes a
  self-contained prompt and runs `codex exec` (write) or `codex exec -s read-only` (advice), then
  returns the result. Fails loudly if `codex` is missing — never a silent Claude fallback.
- **grok-4.5** — only through the Grok CLI (`grok --prompt-file … -m grok-4.5`), same wrapper
  pattern. Fails loudly if `grok` is missing.

## Sources

Rankings are the operator's own (value reflects actual bills). Placement of grok-4.5 and
haiku-4.5 is informed by:

- [Introducing Grok 4.5 · Cursor](https://cursor.com/blog/grok-4-5)
- [Grok 4.5 — Intelligence, Performance & Price · Artificial Analysis](https://artificialanalysis.ai/models/grok-4-5)
- [SpaceXAI releases Grok 4.5, an "Opus-class model" · TechCrunch](https://techcrunch.com/2026/07/08/spacexai-releases-grok-4-5-which-elon-describes-as-an-opus-class-model/)
- [Introducing Claude Haiku 4.5 · Anthropic](https://www.anthropic.com/news/claude-haiku-4-5)
