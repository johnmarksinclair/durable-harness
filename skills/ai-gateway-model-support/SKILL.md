---
name: ai-gateway-model-support
description: Implement or debug AI Gateway model support in coding-agent harnesses such as Claude Code, Codex, OpenCode, Pi, and libfx. Use when new model IDs fail discovery, tool dispatch, reasoning settings, or durable recovery, or when building a model compatibility matrix.
---

# AI Gateway model support

Support new interactive models by discovering their metadata and adapting the
harness's actual protocol. A growing hardcoded model allowlist becomes stale.
A catalog entry or successful text response alone does not establish working
agent tools or durable recovery.

## Establish the integration being changed

Identify the harness, installed SDK/CLI versions, binary actually running in
the session, generated provider configuration, and model transport. Inspect the
pinned implementation before applying settings from a newer standalone CLI.
A laptop's configuration does not configure a durable-harness session's
generated environment.

Read [harness-specific guidance](references/harnesses.md) for the selected
integration. Read [observed cases](references/observed-cases.md) when diagnosing
a matching model or error; those dated results are examples, not a permanent
support allowlist.

## Discover eligible models and preserve metadata

Fetch the complete current catalog rather than constructing names by analogy:

```sh
curl -fsS https://ai-gateway.vercel.sh/v1/models
```

Use the exact returned `provider/model` ID. For an interactive text agent, look
for a language model with text input/output, and check `tool-use` metadata when
tools are required. Keep embeddings, media-generation and batch IDs out of
the interactive agent picker. Metadata describes eligibility; test required
capabilities through the chosen harness.

Preserve context/output limits, input modalities, reasoning capabilities,
prices, and existing SDK compatibility settings. Gateway quotes dollars per
token; Pi and OpenCode registries use dollars per million tokens. Do not infer
limits or reasoning behavior from names. For an alias absent from the catalog,
use documented metadata or an explicit definition rather than inventing limits.

Distinguish a missing bundled registry entry from an inference failure. Known
entries should remain usable without discovery. Refresh unknown IDs with a
timeout and caller cancellation; retain the previous usable catalog on failure
and surface the discovery cause. Preserve bundled compatibility metadata when
adding IDs. An explicitly offline refresh should make no network request.

## Match the harness's transport and runtime

| Harness | Transport in durable-harness | Implementation requirement |
| --- | --- | --- |
| Claude Code / CC | Anthropic Messages through the session's relay | Configure main/background models, native aliases and the session allowlist together. |
| Codex | Responses API through the session's relay | Expose journaled tools directly and preserve RPC/rollout call IDs. |
| OpenCode / OC | Built-in `vercel` provider over Gateway's native language-model API | Supply new model metadata to the embedded provider through its existing relay. |
| Pi | pi-ai Gateway provider over Anthropic Messages | Refresh an unknown ID while preserving bundled provider compatibility. |
| Pi durable | The same pi-ai Gateway provider | Explicitly refresh the collection before opening a Harness for new IDs. |
| fx / libfx | libfx's own agent/provider integration | Validate the actual engine and transport; workspace tools do not select the provider. |
| AI SDK adapter | The installed AI SDK Gateway provider | Inspect version-matched APIs and validate its own tool loop. |

The selected model's vendor does not select the wire format: Codex still speaks
Responses with a Claude model, and Pi's Gateway provider still speaks Messages
with an OpenAI model. Raw API success cannot certify a CLI or a harness with
different request transformations.

Keep authentication on its existing boundary: `AI_GATEWAY_API_KEY`, or Vercel
OIDC through a fresh credential callback. In relay-based integrations, the
child keeps its placeholder and the driver adds the real credential. Preserve
allowed paths, model allowlists, budgets and tool policies when adding discovery.
Use current per-agent Gateway docs for standalone CLI setup; do not transplant
their endpoint paths into a fixed-route relay.

Apply reasoning settings in the selected API's supported shape. Some routed
models require thinking and reject a default `off`; others reject unsupported
effort values. Preserve the user's requested model and routing. Validate any
configured fallback against the same required capabilities.

## Prove a real tool round trip

Use the exact pinned harness/SDK/binary and generated configuration. Give the
probe one harmless tool with a strict schema, such as a required integer
`value`. Ask the model to call it with `17`. Validate the argument, return a
fresh unpredictable string from the implementation, and require the final
model response to contain that string. The string must not appear in the user
prompt. Verify inference continues after the tool result.

Inspect structured errors, stderr and tool traces as well as terminal status.
A model can describe a disabled tool and still end with `completed`. Exercise
streaming tool-argument assembly when the integration streams.

For a durable harness, also check the journal boundary: save the pending call
before a never-replay side effect executes; the RPC call ID must identify the
same journal effect and native call. Then crash and resume around intent and
result persistence. Verify recorded results are recovered, safe tools follow
their replay policy, and never-replay tools are not executed twice. Provider
round trips and recovery are separate evidence.

Cover a known bundled model, a new ID, a model requiring reasoning, the affected
tool-transport family, and modalities or built-ins the app actually uses. Run
focused offline tests for discovery failure/cancellation and provider mapping.
Run live probes within the task's authorized spend, with bounded timeouts and
the application's retry policy. Keep credentials out of output and artifacts.

## Diagnose the failing layer

| Symptom | Investigate |
| --- | --- |
| Unknown model before any inference | Bundled catalog, refresh, ID normalization and registration. |
| `code-mode host is disabled` | Codex's generated tool configuration and direct-function exposure. |
| Never-replay barrier declines tools | Native persistence and RPC/native call-ID identity. |
| `Disabling thinking is not supported` | Routed provider requirements and the harness's default. |
| Failure only after a tool result | Message roles/order, reasoning blocks, tool IDs and provider templates. |
| 401, credit/budget failure or policy 403 | Credentials, spend controls or team/provider access; record the actual classification. |
| Timeout, rate limit or capacity error | Transient availability; keep results inconclusive unless reproduced as structural failures. |

Do not resolve a policy block by changing team rules or a model failure by
silently substituting a different model. Stay within the requested integration
and routing policy.

## Report support at the level actually tested

Record exact model ID, harness/dependency versions, date, protocol, routed
provider when known, reasoning settings, tool/result outcome and recovery
outcome. Keep catalog-listed, API-tested, provider-tested, harness-tested and
recovery-tested evidence separate. Mark unrun cases untested, policy blocks as
access restrictions, and timeouts inconclusive. Limit incompatibility claims to
the observed version, route and settings.

For a matrix, enumerate the complete catalog and retain those evidence levels
per model/harness pair. Keep before/after results separate. Document unsupported
modalities and unresolved cases instead of promising every Gateway model can
serve as an interactive coding agent.

Current authoritative sources:

- [Gateway model catalog](https://ai-gateway.vercel.sh/v1/models)
- [Gateway APIs](https://vercel.com/docs/ai-gateway/sdks-and-apis)
- [Coding-agent setup](https://vercel.com/docs/ai-gateway/coding-agents)
- [Authentication](https://vercel.com/docs/ai-gateway/authentication-and-byok)
