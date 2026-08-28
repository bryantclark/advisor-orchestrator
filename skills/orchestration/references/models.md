# The model ladder

Single source of truth for routing. Three axes, 1–10, from the operator's seat (not list price or
public benchmarks). Edit here — the orchestration doctrine and `~/.claude/CLAUDE.md` both read this file.

- **intelligence** — hardest problem you'd hand it _unsupervised_ and trust the result.
- **taste** — UI/UX, code quality, API design, copy. Decides user-facing work (bar: taste ≥ 7).
- **value** — how cheap _to you_ in practice (generous plan limits count), higher = cheaper. Tie-breaker only.
- **reason@** — reasoning effort where it's best _value_ on delegated work. All current lanes are good enough at **low**.

| model            | family    | intel | taste | value | reason@ | access                                    |
| ---------------- | --------- | ----- | ----- | ----- | ------- | ----------------------------------------- |
| **fable-5**      | Anthropic | 9     | 9     | 2     | low     | `model: fable`                            |
| **gpt-5.6-sol**  | OpenAI    | 7     | 5     | 8     | low     | Codex CLI — `--model gpt-5.6-sol`         |
| **opus-4.8**     | Anthropic | 6     | 8     | 5     | low     | `model: opus`                             |
| **sonnet-5**     | Anthropic | 4     | 7     | 5     | low     | `model: sonnet`                           |
| **gpt-5.6-luna** | OpenAI    | 4     | 4     | 10    | low     | Codex CLI — `--model gpt-5.6-luna`        |
| **haiku-4.5**    | Anthropic | 2     | 4     | 8     | low     | `model: haiku` — **avoid**                |

- **intelligence:** fable > sol > opus > sonnet ≈ luna > haiku
- **taste:** fable > opus > sonnet > sol > luna ≈ haiku
- **value:** luna > sol ≈ haiku > opus ≈ sonnet > fable

## Reasoning effort

Effort tracks **direction**: delegating determined volume → **low** (you verify anyway; extra reasoning is
wasted tokens/latency); escalating for judgment → **high**. Bump a delegated task to high only when it's
correctness-critical (concurrency, money, migrations).

The codex lane sets effort per call with `-c model_reasoning_effort=low|medium|high` — independent of which
GPT model you pick. Pair a model with an effort: `sol` at low for cheap volume, `sol` at high for a
cross-vendor advisory verdict, `luna` at low for the easiest determined work.

## Notes

- **Two GPT-5.6 lanes, pick by task.** Both are OpenAI, both run through the Codex CLI, both selected with
  `--model`:
  - **sol** (intel 7, value 8) — strong intelligence _and_ cheap: the default GPT delegation target for real
    work, and the cross-vendor advisor when run high + read-only.
  - **luna** (intel 4, value 10) — incredibly cheap but dumber: the bottom-of-ladder volume lane, only for the
    easiest determined work (rote edits, mechanical wiring). Not for anything correctness-sensitive.
  - Both sit below the taste-7 bar, so neither touches user-facing surfaces — that's Opus/Fable.
- **Set the model explicitly.** Pass `--model gpt-5.6-sol` or `--model gpt-5.6-luna`. With no `--model`, codex
  inherits `~/.codex/config.toml`'s default, which is account-specific — fine as a fallback, but name the model
  when you want a specific lane. (A bare `--model gpt-5.6` is rejected on ChatGPT-app auth; use the `-sol` /
  `-luna` slugs.)
- **Claude model pins silently fall back** to the session model if that tier isn't on your plan; the codex
  lane fails loudly instead. If routed results feel flat, check your plan.
