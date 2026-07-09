# The model ladder

Single source of truth for routing. Three axes, 1–10, from the operator's seat (not list price or
public benchmarks). Edit here — the orchestration doctrine and `~/.claude/CLAUDE.md` both read this file.

- **intelligence** — hardest problem you'd hand it *unsupervised* and trust the result.
- **taste** — UI/UX, code quality, API design, copy. Decides user-facing work (bar: taste ≥ 7).
- **value** — how cheap *to you* in practice (generous plan limits count), higher = cheaper. Tie-breaker only.
- **reason@** — reasoning effort where it's best *value* on delegated work. Most lanes are good enough at **low**; Grok is the exception, meaningfully better at **high**.

| model | family | intel | taste | value | reason@ | access |
|---|---|---|---|---|---|---|
| **fable-5** | Anthropic | 9 | 9 | 2 | low | `model: fable` |
| **gpt-5.5** | OpenAI | 7 | 5 | 8 | low | Codex CLI — `codex exec` / `codex exec -s read-only` |
| **opus-4.8** | Anthropic | 6 | 8 | 5 | low | `model: opus` |
| **grok-4.5** | xAI | 6 | 6 | 9 | **high** | Grok CLI — `grok … -m grok-4.5 --effort high` |
| **sonnet-5** | Anthropic | 4 | 7 | 5 | low | `model: sonnet` |
| **haiku-4.5** | Anthropic | 2 | 4 | 8 | low | `model: haiku` — **avoid** |

- **intelligence:** fable > gpt-5.5 > opus ≈ grok > sonnet > haiku
- **taste:** fable > opus > sonnet > grok > gpt-5.5 > haiku
- **value:** grok ≈ gpt-5.5 > haiku > opus ≈ sonnet > fable

## Reasoning effort

Effort tracks **direction**: delegating determined volume → **low** (you verify anyway; extra reasoning is
wasted tokens/latency); escalating for judgment → **high**. Per-model `reason@` overrides direction: **Grok
4.5 runs high even for volume**. Bump a delegated task to high only when it's correctness-critical
(concurrency, money, migrations).

## Notes

- **gpt-5.5 + grok-4.5** — strong intelligence *and* the cheapest lanes (generous CLI limits): the default
  delegation targets. Both sit below the taste-7 bar, so never on user-facing surfaces — that's Opus/Fable.
  gpt-5.5 also doubles as a cross-vendor advisor (run high, read-only).
- **grok's taste (6) and reason@ (high)** are the operator's estimate, not benchmarks — recalibrate with use.
- **Claude model pins silently fall back** to the session model if that tier isn't on your plan; the codex
  and grok lanes fail loudly instead. If routed results feel flat, check your plan.
