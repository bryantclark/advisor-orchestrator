# The model ladder

Single source of truth for routing. Three axes, 1–10, from the operator's seat (not list price or
public benchmarks). Edit here — the orchestration doctrine and `~/.claude/CLAUDE.md` both read this file.

- **intelligence** — hardest problem you'd hand it _unsupervised_ and trust the result.
- **taste** — UI/UX, code quality, API design, copy. Decides user-facing work (bar: taste ≥ 7).
- **value** — how cheap _to you_ in practice (generous plan limits count), higher = cheaper. Tie-breaker only.
- **reason@** — reasoning effort where it's best _value_ on delegated work. All current lanes are good enough at **low**.

| model         | family    | intel | taste | value | reason@ | access                                               |
| ------------- | --------- | ----- | ----- | ----- | ------- | ---------------------------------------------------- |
| **fable-5**   | Anthropic | 9     | 9     | 2     | low     | `model: fable`                                       |
| **gpt-5.6**   | OpenAI    | 7     | 5     | 8     | low     | Codex CLI — `codex exec` / `codex exec -s read-only` |
| **opus-4.8**  | Anthropic | 6     | 8     | 5     | low     | `model: opus`                                        |
| **sonnet-5**  | Anthropic | 4     | 7     | 5     | low     | `model: sonnet`                                      |
| **haiku-4.5** | Anthropic | 2     | 4     | 8     | low     | `model: haiku` — **avoid**                           |

- **intelligence:** fable > gpt-5.6 > opus > sonnet > haiku
- **taste:** fable > opus > sonnet > gpt-5.6 > haiku
- **value:** gpt-5.6 > haiku > opus ≈ sonnet > fable

## Reasoning effort

Effort tracks **direction**: delegating determined volume → **low** (you verify anyway; extra reasoning is
wasted tokens/latency); escalating for judgment → **high**. Bump a delegated task to high only when it's
correctness-critical (concurrency, money, migrations).

## Notes

- **gpt-5.6** — strong intelligence _and_ the cheapest lane (generous CLI limits): the default delegation
  target. It sits below the taste-7 bar, so never on user-facing surfaces — that's Opus/Fable. It also
  doubles as a cross-vendor advisor (run high, read-only).
- **Claude model pins silently fall back** to the session model if that tier isn't on your plan; the codex
  lane fails loudly instead. If routed results feel flat, check your plan.
