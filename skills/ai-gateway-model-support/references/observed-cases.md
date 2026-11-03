# Observed model compatibility cases

These bounded development observations are from November 2, 2026, using this
repository's pinned dependencies and Gateway routes. Recheck the live catalog,
installed versions, provider route and settings before applying them. These
examples explain failure modes; they do not certify every model or variant.

| Exact Gateway model ID | Observed result and implementation lesson |
| --- | --- |
| `openai/gpt-5.6-sol` | Original Codex configuration failed tool dispatch. Direct `functions` exposure passed with pinned 0.160.0 and installed 0.160.1. |
| `openai/gpt-6-sol` | Same Codex tool-mode requirement; direct-function round trips passed. |
| `openai/gpt-6.1-sol` | Same Codex requirement; an apparently completed turn could still have skipped the tool. |
| `openai/gpt-6-astra` | Same Codex tool-mode requirement; direct-function round trips passed. |
| `openai/gpt-5.4-mini` | Positive control passed Codex tool round trips before and after the direct-function change. |
| `openai/gpt-6.1-terra` | Missing from bundled Pi/OpenCode catalogs; the refreshed actual Pi provider passed a tool/result round trip. |
| `anthropic/claude-haiku-5.5` | Same catalog gap and successful Pi provider result after discovery. |
| `minimax/minimax-m3.1-flash` | Same catalog gap and successful Pi provider result after discovery. |
| `alibaba/qwen3.8-max` | Routed Fireworks backend rejected Pi's default disabled thinking; provider `reasoning: 'medium'` passed. Codex's post-tool request failed with `System message must be at the beginning`. Raw API round trips passed, so an ID/catalog fix alone does not address it. |
| `google/gemini-4-argon` | Raw Responses round trip passed. Messages made a valid tool call but post-tool inference timed out; compatibility remains inconclusive on that path. |
| `moonshotai/kimi-k3.1-pro` | Team/provider policy returned 403. This is an access restriction, not established model or harness incompatibility. |

For the five direct-function Codex runs on 0.160.0, every RPC call ID matched
a saved native `function_call`, present before executing the tool. This checks
the identity needed by the journal barrier. Vercel Sandbox crash recovery was
not exercised by those app-server probes.

Additional baseline tool/result probes passed for `openai/gpt-5.5`,
`anthropic/claude-opus-5.5-fast`, `google/gemini-3.8-flash`, `zai/glm-5.3`,
`deepseek/deepseek-v4.1-flash` and `spacexai/grok-4.7` through installed Codex
0.160.1 and pinned pi-ai 1.0.4. Those passes do not extend to every variant,
long contexts, images, built-in tools or recovery.

Claude Code evidence was raw Gateway Messages tool/result tests; its pinned
CLI and Sandbox wrapper were not run. OpenCode evidence was catalog inspection
and metadata/relay tests, with no complete embedded-engine turn. fx's model
path was not run. Keep those evidence levels distinct from full harness support.

Reusable fixes are direct Codex tools with stable journal identity, live
registry extension for new IDs, and provider-appropriate reasoning settings.
Treat message-order failures and access policy errors at their own boundaries.
