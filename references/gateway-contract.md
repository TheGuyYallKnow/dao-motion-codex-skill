# Dao Motion Studio Gateway Contract

## Contents

1. Transport boundary
2. Invocation sequence
3. Response envelope
4. Necessary Data rule
5. Discovery operations
6. Graph operations
7. Custom-authoring operations
8. Preview and runtime operations
9. Identity and concurrency
10. Error handling

## 1. Transport boundary

Use only the official Roblox Studio MCP. Invoke the plugin-owned Studio gateway through Edit-mode `execute_luau`. Do not call a shell command, CLI, external executable, local server, HTTP endpoint, WebSocket, or filesystem mailbox.

The planned gateway is one exact `BindableFunction` exposed by the loaded Library Bridge plugin. Its expected identity is:

- Name: `__DaoMotionMcpGateway`
- Parent: `ServerStorage`
- Class: `BindableFunction`
- Attribute `DaoMotionOwned`: `true`
- Attribute `ProtocolVersion`: supported positive integer
- Attribute `SessionId`: non-empty token generated for the current plugin load

Refuse duplicates, wrong classes, missing ownership, missing session identity, and unsupported protocol versions.

## 2. Invocation sequence

Use a fixed dispatcher shape rather than generating operation-specific persistence code:

```lua
local HttpService = game:GetService("HttpService")
local ServerStorage = game:GetService("ServerStorage")

local gateway = ServerStorage:FindFirstChild("__DaoMotionMcpGateway")
assert(gateway and gateway:IsA("BindableFunction"), "DAO_MOTION_GATEWAY_UNAVAILABLE")
assert(gateway:GetAttribute("DaoMotionOwned") == true, "DAO_MOTION_GATEWAY_UNOWNED")

local request = {
	protocolVersion = 1,
	operation = "ping",
	requestId = "request-ping-1",
	arguments = {},
}

local responseJson = gateway:Invoke(HttpService:JSONEncode(request))
print("DAO_MOTION_RESPONSE:" .. responseJson)
```

Parse only the exact `DAO_MOTION_RESPONSE:` marker. Treat malformed, missing, duplicate, or oversized responses as protocol failures.

Every request requires a unique bounded `requestId`. After `ping`, every non-ping request also requires `expectedSessionId = response.sessionId`; reread `ping` instead of reusing a stale session token. The `capabilities.requestContract` record is the live machine-readable source for these envelope fields and includes directly invokable ping and session-bound examples.

## 3. Response envelope

Successful responses use:

```json
{
  "ok": true,
  "protocolVersion": 1,
  "operation": "graph.get",
  "sessionId": "plugin-load-token",
  "result": {},
  "warnings": []
}
```

Failed responses use:

```json
{
  "ok": false,
  "protocolVersion": 1,
  "operation": "graph.get",
  "sessionId": "plugin-load-token",
  "error": {
    "code": "GRAPH_UNAVAILABLE",
    "message": "The selected target does not own a Motion graph.",
    "details": {}
  }
}
```

Never infer success from printed text. Require `ok == true`.

## 4. Necessary Data rule

**Necessary Data** is the protocol term for data required to complete, confirm, or recover from the current operation. Normal operations return only Necessary Data and must not repeat the complete documentation catalog.

Use progressive discovery:

- `catalog.index` returns compact route-selection data for every available node, recipe, technique, and harness type.
- `catalog.describe` returns the full description for one selected catalog item.
- `operation.describe` returns the full what/how/why/when contract for one gateway operation.
- `guide.index` returns compact guide discovery data.
- `guide.get` returns the full guide for one authoring mechanic.
- `template.get` returns a current machine-ready skeleton only when authoring that mechanic.

Descriptions must remain available but should not inflate unrelated graph responses.

## 5. Discovery operations

### `ping`

Use first to verify the gateway, protocol, plugin load session, and Edit-mode reachability. Return only gateway identity, protocol version, plugin version, Motion schema version, and current Studio mode.

### `capabilities`

Use after `ping` and whenever the plugin session changes. Return the request-envelope contract, supported operation IDs, feature flags, graph limits, response limits, schema versions, and whether preview, Apply, and runtime generation are currently possible.

### `operation.describe`

Use when an operation is unfamiliar, newly added, or changed in the active protocol session. Accept one exact operation ID and return what it does, how to call it, why it exists, when to use and avoid it, required and optional arguments, Necessary Data returned on success, errors, side effects, mode/authorization requirements, constraints, and a minimal example. Do not return every operation's full description from `capabilities`.

### `context.get`

Use before route selection or mutation. Return the exact Studio/place identity, current selection summary, graph target identity, graph ID, revision, draft/dirty state, preview state, runtime status, binding summaries, and current session token.

### `catalog.index`

Use to compare routes. Each item must include only:

- `id`
- `displayName`
- `kind` and category
- one-sentence `summary`
- one-sentence `useWhen`
- optional one-sentence `avoidWhen`
- tags
- target class/cardinality summary
- source: built-in, bundled, custom global, custom one-time, recipe, or harness
- `detailAvailable`

### `catalog.describe`

Use for likely candidates. The full record must explain:

- What the item does.
- How it behaves and how to configure it.
- Why it exists instead of nearby alternatives.
- When to use it.
- When not to use it.
- Required and optional targets.
- Input/output ports and each port's meaning.
- Every parameter, default, bounds, meaning, and interaction.
- Yield and completion semantics.
- Preview behavior and runtime behavior.
- Binding and layout restrictions.
- Validation failures and common mistakes.
- Minimal example and one realistic example.
- Related alternatives and selection guidance.
- Compatibility, source, version, and trust/dirty state where relevant.

### `guide.index`

Use before non-basic authoring. Index at least:

- `graph`
- `custom-technique`
- `harness`
- `stagger-loop-harness`
- `visual-group`
- `recipe`
- `harness-preset`
- `advanced-text`
- `preview`
- `runtime`

### `guide.get`

Return the complete current workflow for one guide, including what/how/why/when, prerequisites, ordered steps, validation, staging, persistence, risks, and examples.

### `template.get`

Return the current machine-ready skeleton for a graph, node, custom technique, harness, visual group configuration, recipe, or Harness Preset. Empty ID maps serialize as JSON arrays in Studio, so graph templates also return `collectionEncoding`: `nodes` and `editor.harnesses` accept either keyed ID maps or dense arrays of records containing `id`, and validation normalizes them to canonical ID maps. A template is data, not documentation; fetch the guide first.

## 6. Graph operations

### `graph.target.refresh`

When Studio selection was changed through MCP but Motion Lab still reports the previous target, load the exact single selected GuiObject through the existing selected-draft loader. Require an unlocked graph and the exact current selection path. Reject ambiguous selection, path changes, or a resulting target mismatch. Refresh never persists a graph.

### `graph.lock`

Lock or unlock only the exact active Motion Lab graph target. Require `locked` and the exact target path returned by the current `context.get`. Reject a changed or unavailable target. Locking is transient editor state: it does not change, Stage, Apply, or persist the graph. Lock the intended owner and verify `context.get.graphLock.locked == true` before `graph.get` or any authoring operation.

### `graph.get`

Read the canonical plain graph model and its revision. Preserve nodes, edges, bindings, harnesses, and editor state. Do not expose the private Instance persistence layout unless a diagnostic mode explicitly requests it.

### `graph.validate`

Validate a complete candidate or bounded patch without changing the active draft. Return normalized errors and warnings, resolved binding identities, schedule summary, graph caps, and whether Stage would be allowed.

### `graph.diff`

Compare a validated candidate with the current graph. Return material additions, removals, and changes for nodes, edges, bindings, harnesses, visual groups, custom references, and editor metadata. Ignore irrelevant serialization order.

### `graph.stage`

Load a validated candidate into Motion Lab as an unsaved visible draft. Open Motion Lab, preserve the previous draft for discard, and return a draft token, base revision, and material diff. Stage and Discard preserve the authored graph revision; a real manual edit still advances it and invalidates the staged token. Do not persist or generate runtime.

### `graph.organize`

Run Motion Lab's deterministic **Organize All** layout on the exact current clean staged draft. Require its draft token. Reuse rendered node heights, organize every connected component and loose node, frame the result, refresh the staged fingerprint, and return whether positions changed. Keep the same draft token clean and Apply-ready without creating history, persistence, or runtime output. Use it after Stage and before final validation, Preview, or Apply; skip it only when the user explicitly wants the manual layout preserved.

### `graph.discard`

Discard only the current staged gateway draft and restore the prior draft/selection state. Refuse when the user manually changed the staged draft unless the request explicitly confirms that newer draft should be discarded.

### `graph.apply`

Persist only an exact current staged draft. Require Studio ID, place ID, plugin session ID, graph ID, base revision, and draft token. Revalidate, stop conflicting preview, use existing Change History rollback, persist, emit, and return the new revision plus runtime-installed status.

### `graph.remove`

Remove only the exact owned applied graph module from the locked active target. Require the exact target path plus explicit authorization, preserve reusable binding ownership metadata, and record the removal through Change History. Do not use it for ordinary draft cleanup or without first applying and verifying a replacement during a graph-owner correction.

## 7. Custom-authoring operations

Expose these only when `capabilities` reports support:

- `technique.draft.create`
- `technique.draft.update`
- `technique.validate`
- `technique.save`
- `technique.fork`
- `harness.create`
- `harness.update`
- `harness.ungroup`
- `harness.delete`
- `visual-group.inspect`
- `visual-group.configure`
- `preset.validate`
- `preset.stage`
- `preset.save`
- `preset.save-as`
- `preset.remove`

Use [authoring-workflows.md](authoring-workflows.md) before any of these operations.

`visual-group.inspect` is read-only. `visual-group.configure` writes the selected include/order attributes immediately through the existing Change History path, so require the current graph revision, exact node/group identity, and explicit user authorization. It is separate from graph Stage and Apply.

## 8. Preview and runtime operations

### `preview.start`

Validate the current draft and simulate one or a supported set of triggers. Return schedule identity, selected triggers, duration or live status, warnings, and Auto Restore state. Targetless triggers such as Universal Key Bind must preview without resolving a Studio object; targeted triggers return `INVALID_TARGET` when their binding is unavailable or incompatible.

### `preview.pause`, `preview.resume`, `preview.stop`, `preview.reset`

Operate only on the active preview token. Distinguish Stop from Reset: Stop ends execution; Reset restores captured authored state.

### `runtime.status`

Return whether a current singleton runtime is installed, its version, graph readiness, missing dependencies, and whether generation is needed.

### `runtime.generate`

Require explicit authorization. Revalidate stored graphs and ownership, run the existing undoable generator, and return runtime version, migrations, included dependencies, and warnings.

## 9. Identity and concurrency

Every mutation must carry or derive:

- Studio ID
- game ID and place ID
- plugin `SessionId`
- graph ID
- graph target identity
- base graph revision
- draft token when staged

Reject stale sessions, changed targets, changed selections when selection is part of the request, changed revisions, manually changed drafts, and Play/Test mode.

Represent instance paths as arrays of path segments plus expected class. `binding.resolve` requires a bounded logical `alias` in addition to the dense path segments. Reject ambiguity and cross-`ScreenGui` bindings. Prefer existing stable Motion binding IDs when returned by `context.get`.

## 10. Error handling

Do not retry a write against a different Studio, graph, or target. Resolve errors by code:

- Gateway/protocol unavailable: stop and report setup state.
- Wrong Studio mode: stop Play only when the user authorized authoring continuation.
- Ambiguous or missing binding: inspect Studio and request/derive an exact target.
- Validation failure: revise the candidate, then validate again.
- Stale revision/session/draft: reread context and graph before replanning.
- Dirty custom technique: fetch its guide and validate/save it; do not bypass trust.
- Preview conflict: stop or reset through the gateway as required.
- Ownership conflict: report it; never overwrite a lookalike container.
