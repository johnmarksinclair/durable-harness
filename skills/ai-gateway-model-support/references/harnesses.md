# Harness implementation details

These instructions describe durable-harness's adapters. In another project,
map them to its generated configuration, registry and effect journal. Inspect
the installed package and binary before reusing version-specific keys.

## Claude Code / CC

The Agent SDK launches Claude Code inside the session's Vercel Sandbox.
Requests use Anthropic Messages regardless of the selected model's vendor.

Use the public `model` and optional `smallModel` settings. The adapter maps
ordinary Anthropic Gateway IDs to the native aliases the CLI recognizes;
other provider IDs pass through. Keep conversion inside the adapter rather
than rewriting every catalog ID with a slash or hyphen replacement.

The generated environment sets `ANTHROPIC_BASE_URL` to the relay and
`ANTHROPIC_AUTH_TOKEN` to a placeholder. Do not inject a direct-provider API
key that overrides this path. Main, small/background and alias model calls must
all fit the session's allowed IDs. Reusing the main model for background calls
avoids an unexpected default reaching a different model.

Validate the pinned SDK and bundled CLI, including tool schemas, reasoning
options, background calls and post-tool requests. A direct Messages test has
not exercised the CLI's transformations or Sandbox recovery.

Source paths: `packages/harness-claude-code/src/env.ts`, `harness.ts`,
`versions.ts`, and `packages/durable-harness/src/claude-code.ts`.

## Codex

The session generates `CODEX_HOME/config.toml` and launches `codex app-server`.
With the pinned 0.160.0 adapter, preserve this internal configuration in both
built-in tool modes:

```toml
[features]
code_mode_host = false

[features.code_mode]
enabled = true
direct_only_tool_namespaces = ["functions"]
```

Newer model families can otherwise select code-mode-only tools. Exposing
`functions` directly keeps dynamic calls as ordinary `function_call` items.
The saved `call_id`, the `item/tool/call` RPC's `callId`, and the journal key
`tool:<call_id>` must identify the same effect. Check the call's presence in
the rollout mirror before a never-replay tool executes.

Enabling only the nested code-mode host is insufficient for this adapter:
nested dynamic calls can have `exec-<uuid>` RPC IDs absent from the rollout's
parent `exec` call. This breaks the barrier and recovery lookup even when a
harmless tool round trip works. A future nested-host implementation would need
explicit durable parent/child identity and recovery support.

These are adapter-owned settings. Public `config.features` overrides are
reserved; update the adapter rather than advising apps to bypass that
restriction. Keep shell, image and other built-ins governed by their existing
flags. The public `effort` setting is Codex's reasoning option; set
`contextWindow` from the chosen model's metadata rather than assuming the
adapter's generic default fits every model.

The relay uses `POST /v1/responses` with WebSockets disabled. Adding `/codex`
to `gatewayUrl` does not activate the standalone compatibility endpoint:
the relay uses fixed paths on the upstream origin.

Source paths: `packages/harness-codex/src/config.ts`, `harness.ts`, `rollout.ts`,
and `packages/harness-process/src/gateway.ts`.

## Pi and Pi durable

The pi-ai Gateway provider here speaks Anthropic Messages for every vendor.
Keep its bundled compatibility metadata when extending the catalog.

`durable-harness/pi` accepts an exact Gateway ID as a string and refreshes on
an unknown ID. Its public reasoning option is `thinkingLevel`; pi-ai's lower
provider API calls this `reasoning`. Choose a setting the routed provider
supports. A custom pi-ai `Model` also needs an appropriate `streamFn`.

`durable-harness/pi-durable` keeps its synchronous factory API. Refresh and
check the result before opening a Harness for IDs newer than the bundle:

```ts
import { GATEWAY_PROVIDER, gatewayModels } from 'durable-harness/pi-durable';

const models = gatewayModels();
const refreshed = await models.refresh({ providers: [GATEWAY_PROVIDER] });
const error = refreshed.errors.get(GATEWAY_PROVIDER);
if (error) throw error;
// Pass models into Harness.open(...); wait for refresh before opening it.
```

The shared factory reads the current Gateway credential per request. Discovery
adds metadata for new interactive language IDs, retains the previous catalog
on failure and respects offline refresh. Provider success does not validate
Pi durable's storage, wakeup or recovery behavior.

Source paths: `packages/durable-harness/src/pi-gateway.ts`, `pi.ts`, and
`pi-durable.ts`.

## OpenCode / OC

Use the embedded server's built-in `vercel` provider. The adapter accepts
`provider/model`; internally OpenCode uses `vercel/provider/model`. Remove
only that leading OpenCode provider segment for a Gateway metadata lookup.

Read `GET /v1/models` through the existing authenticated loopback relay and
supply the selected entry under `providers.vercel.models[id]`. For the pinned
v2 integration, map `modelID`, `name`, `limit.context`, `limit.output`,
`capabilities.tools`, input/output modalities and per-million-token `cost`,
including cache prices. Read the installed schema when upgrading.

Keep `providers.vercel.settings.baseURL` and `apiKey` on the inference relay
and its placeholder. The host keeps `models: { fetch: false }` to avoid
separate models.dev fetching; that permits explicit live model registration.
Discovery failure falls back to bundled definitions, which can still lack
the requested new model.

Validate an actual embedded-engine tool round trip. Mapping tests and catalog
presence do not prove that the engine accepts the config or sends compatible
requests. Keep pinned OpenCode packages and Effect versions consistent;
updating a standalone OpenCode binary cannot update them.

Source paths: `packages/harness-opencode/src/models.ts`, `host.ts`,
`versions.ts`, and `packages/harness-process/src/gateway-models.ts`.

## fx / libfx and the AI SDK adapter

For public fx integration, use libfx's `createFxAgent`;
`durable-harness/fx` supplies workspace tools. Inspect the installed libfx
provider/model implementation and validate its actual requests and durability.
The separate `harness-world-fx` engine adapter is private. This repository has
no separate public `rfx` export.

For an AI SDK agent, inspect the installed SDK and Gateway provider APIs
before selecting a model/configuration shape. Exercise that adapter's own
tool loop, usage/error handling and durable steps. Messages or Responses
success does not prove a Gateway-native language-model request works.

Source paths: `packages/durable-harness/src/fx.ts`,
`packages/harness-fx/README.md`, and `apps/demo/lib/harnesses/ai-sdk.ts`.
