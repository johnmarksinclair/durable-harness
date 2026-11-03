# Models through AI Gateway

Use the exact `provider/model` ID returned by
[`GET /v1/models`](https://ai-gateway.vercel.sh/v1/models). Catalog presence
describes availability and capabilities; a working agent also needs to call a
tool and consume its result through its harness's protocol.

Claude Code and Codex accept model IDs through their generated provider
configuration. Codex exposes the `functions` namespace directly, including for
newer models that otherwise select code-mode-only tools. Direct tool calls keep
the same call ID in app-server RPCs and the saved rollout, which the harness's
journal and never-replay barrier require. The nested code-mode host stays
disabled. Shell, image, plugin and sub-agent tools retain their separate flags.

`durable-harness/pi` first uses pi-ai's bundled Gateway registry. For an ID
that registry does not know, it refreshes from Gateway before resolving the
model. Existing bundled entries retain pi-ai's compatibility settings. A
failed discovery request is reported as the cause of a missing-model error;
known bundled models do not require discovery.

`durable-harness/pi-durable` keeps the synchronous `gatewayModels()` API.
Refresh its returned collection before opening a Harness when using newer
IDs, and check the refresh result:

```ts
const models = gatewayModels();
const refreshed = await models.refresh({ providers: [GATEWAY_PROVIDER] });
const error = refreshed.errors.get(GATEWAY_PROVIDER);
if (error) throw error;
```

The refresh adds interactive language models, including their context limits,
output limits, input modalities and prices. It preserves bundled compatibility
settings and keeps the previous catalog if discovery fails. Offline refresh
(`allowNetwork: false`) makes no request. Image-generation, embedding and batch
IDs are not added as interactive chat models.

OpenCode reads the selected model's Gateway metadata through the drive's
authenticated loopback relay and supplies it as a model definition to its
built-in `vercel` provider. It continues using the relay for inference and
does not fetch a separate models.dev catalog. If discovery is unavailable,
it falls back to OpenCode's bundled definitions.

Provider settings also matter. Some reasoning models reject requests that
disable thinking; for Pi, choose a supported `thinkingLevel` instead of its
default `'off'` when required by the routed provider. A provider-policy 403 is
an access restriction, and a tool-result request failing with a provider's
message-template error needs investigation at that protocol/provider boundary.
Neither error is fixed by adding an ID to a catalog.
