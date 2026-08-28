# The model ladder

Single source of truth for routing. Three axes, 1–10, from the operator's seat (not list price or
public benchmarks). Edit here — the orchestration doctrine and `~/.claude/CLAUDE.md` both read this file.

- **intelligence** — hardest problem you'd hand it _unsupervised_ and trust the result.
- **taste** — UI/UX, code quality, API design, copy. Decides user-facing work (bar: taste ≥ 7).
- **value** — how cheap _to you_ in practice (generous plan limits count), higher = cheaper. Tie-breaker only.
- **reason@** — the reasoning effort that's best _value_ on delegated work, per model (not a global default).

| model            | family    | intel | taste | value | reason@ | access                                    |
| ---------------- | --------- | ----- | ----- | ----- | ------- | ----------------------------------------- |
| **fable-5**      | Anthropic | 9     | 9     | 2     | low     | `model: fable`                            |
| **gpt-5.6-sol**  | OpenAI    | 7     | 5     | 8     | medium  | Codex CLI — `--model gpt-5.6-sol`         |
| **opus-5**       | Anthropic | 6     | 8     | 5     | medium  | `model: opus`                             |
| **sonnet-5**     | Anthropic | 4     | 7     | 5     | low     | `model: sonnet` — **avoid**               |
| **gpt-5.6-luna** | OpenAI    | 4     | 4     | 10    | high    | Codex CLI — `--model gpt-5.6-luna`        |
| **haiku-4.5**    | Anthropic | 2     | 4     | 8     | low     | `model: haiku` — **avoid**                |

- **intelligence:** fable > sol > opus > sonnet ≈ luna > haiku
- **taste:** fable > opus > sonnet > sol > luna ≈ haiku
- **value:** luna > sol ≈ haiku > opus ≈ sonnet > fable

## Reasoning effort

Effort still tracks **direction** — escalating for judgment runs **high** — but the per-model `reason@` column
is the best-_value_ default for delegated volume: fable at **low**, sol and opus at **medium**, luna at
**high** (it needs the extra reasoning to compensate). Bump above a model's `reason@` only when the task is
correctness-critical (concurrency, money, migrations).

The codex lane sets effort per call with `-c model_reasoning_effort=low|medium|high` — independent of which
GPT model you pick. Default to the model's `reason@`: `sol` at medium, `luna` at high. Run `sol` high for a
cross-vendor advisory verdict.

## Notes

- **Two GPT-5.6 lanes, pick by task.** Both are OpenAI, both run through the Codex CLI, both selected with
  `--model`:
  - **sol** (intel 7, value 8, best at medium) — strong intelligence _and_ cheap: the default GPT delegation
    target for real work, and the cross-vendor advisor when run high + read-only.
  - **luna** (intel 4, value 10, best at high) — incredibly cheap but dumber: the bottom-of-ladder volume lane,
    only for the easiest determined work (rote edits, mechanical wiring). Not for anything correctness-sensitive.
  - Both sit below the taste-7 bar, so neither touches user-facing surfaces — that's Opus/Fable.
- **Set the model explicitly.** Pass `--model gpt-5.6-sol` or `--model gpt-5.6-luna`. With no `--model`, codex
  inherits `~/.codex/config.toml`'s default, which is account-specific — fine as a fallback, but name the model
  when you want a specific lane. (A bare `--model gpt-5.6` is rejected on ChatGPT-app auth; use the `-sol` /
  `-luna` slugs.)
- **Sonnet and Haiku are both _avoid_ right now** — Sonnet 5 hasn't been refreshed in a while and its
  price-vs-performance no longer competes with sol (cheaper _and_ smarter) or opus (better taste). Reach for it
  only for Claude-family consistency or when the codex lane is down.
- **opus-5 ratings are carried over from opus-4.8** — recalibrate intel/taste with use; a major version may
  have moved it up the ladder.
- **Claude model pins silently fall back** to the session model if that tier isn't on your plan; the codex
  lane fails loudly instead. If routed results feel flat, check your plan.
