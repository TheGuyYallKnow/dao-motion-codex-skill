---
name: dao-motion
description: Author, inspect, repair, validate, safely compact, stage, and apply Dao Motion Lab graphs entirely inside Roblox Studio through the official Roblox Studio MCP. Use for Motion Lab nodes, triggers, conditions, bindings, edges, timelines, graph cleanup or optimization, repeated-node reduction, custom techniques, harnesses, visual groups, recipes, Harness Presets, Advanced Text motion, runtime generation, or separating visual button motion from LocalScript gameplay communication. Never run a preview or enter Play mode; the user owns visual and runtime testing. Never use external programs, CLIs, daemons, HTTP APIs, WebSockets, filesystem synchronization, or direct writes to DaoMotion persistence.
---

# Dao Motion

Operate Motion Lab only through the official Roblox Studio MCP and the plugin-owned Studio gateway. Treat the gateway as the sole authoring API. Never reconstruct or mutate private `DaoMotion` Instance persistence with arbitrary Luau.

## Verification boundary

- Never call `preview.start`, `preview.pause`, `preview.resume`, `preview.stop`, or `preview.reset`.
- Never call `start_stop_play`, enter Play mode, simulate UI input, or perform a runtime/play test.
- Never open a preview surface, including Motion Preview or Advanced Text Preview. If safe authoring requires one, stop and hand that step to the user.
- If `context.get` reports an active preview or Studio is already in Play mode, do not stop or switch it. Ask the user to return Studio to an idle Edit state before continuing.
- Verify only through read-only inspection, `graph.validate`, `graph.diff`, Stage/Organize read-back, Apply read-back, and `runtime.status`. These prove structure and persistence, not appearance or runtime behavior.
- Always report that visual and runtime behavior remain user-tested.

## Required tools

Use the available Roblox Studio MCP equivalents of:

- `list_roblox_studios`
- `set_active_studio`
- `get_studio_state`
- `search_game_tree`
- `inspect_instance`
- `execute_luau`
- `screen_capture`

If the gateway is unavailable, report that the Dao Motion Studio gateway is not installed or loaded. Do not fall back to an external transport or direct persistence edits.

## Load references selectively

- Read [gateway-contract.md](references/gateway-contract.md) before the first gateway call in a task, when handling an error, or when the protocol version changes.
- Read [authoring-workflows.md](references/authoring-workflows.md) before creating or materially changing a custom technique, harness, visual group, recipe, or Harness Preset.
- Use live `operation.describe`, `catalog.index`, `catalog.describe`, `guide.index`, `guide.get`, and `template.get` results as the source of truth. References explain the workflow; live gateway descriptions define current capabilities.

## Core workflow

1. Call `list_roblox_studios` before any modification.
2. If more than one Studio exists, identify the exact intended Studio and call `set_active_studio`. Never rely on a heuristic for writes.
3. Call `get_studio_state`. Require Studio to already be in Edit mode for graph authoring, Apply, and runtime generation. Never switch modes; ask the user to return to Edit mode when needed.
4. Invoke `ping`, then `capabilities`, through the Studio gateway.
5. Call `operation.describe` when an operation is unfamiliar, newly added, or changed in the current protocol session.
6. Call `context.get` to read selection, graph target, graph identity, revision, dirty state, preview state, and available bindings.
7. If Studio selection was changed through MCP and `context.get.target.path` does not equal the exact single `selection[1].path`, require the current graph to be unlocked and call `graph.target.refresh` with that exact selection path. Re-read `context.get` and require the target to match.
8. Before any graph read or authoring action, call `graph.lock` with `locked = true` and the exact `context.get.target.path`. Re-read `context.get`; require `graphLock.locked == true` and the same target path. Never author on an unlocked or mismatched target.
9. Inspect relevant instances with `search_game_tree` and `inspect_instance`.
   For repeated siblings, enumerate every candidate and compare the exact structure needed by the motion: event-surface class/path, visual root, required named children, and optional special descendants. Do not infer uniformity from matching names.
   Compare only behaviorally relevant structure. Ignore decorative descendants, unused properties, and source-unreferenced differences by default. Treat a difference as relevant only when it can change event routing, target resolution, property ownership, or an optional branch the requested motion actually uses.
10. Call `catalog.index` to compare all available routes. Use its summary, `useWhen`, `avoidWhen`, categories, tags, and target requirements.
11. Call `catalog.describe` only for plausible candidates. Read the full what/how/why/when description, ports, parameters, binding rules, yield behavior, constraints, preview/runtime behavior, examples, and related alternatives.
12. Call `graph.get` before changing an existing graph.
13. Before authoring, show the user a compact route proposal whenever the source behavior, target hierarchy, or requested look permits materially different graphs. Include the shared structural group, every outlier and exact difference, the proposed query/static split, excluded behavior, and any hierarchy change required. Ask the user to choose; do not Stage or Apply a guessed design.
14. Produce the smallest graph or patch that satisfies the request. Preserve unrelated nodes, edges, bindings, harnesses, editor state, and metadata.
15. Call `graph.validate`. Resolve every error and reassess every warning.
16. Call `graph.diff` and summarize the material graph changes.
17. Call `graph.stage` by default after the user confirms any material design choice. Staging must open the draft in Motion Lab without persisting it. Treat requests to implement, fix, port, complete, or make the graph work as authorization to continue through Apply; stop at Stage only when the user explicitly asks for a draft, review, proposal, or staged-only result.
18. After Stage, call `graph.organize` by default for every non-empty newly authored or materially changed graph. Pass the current draft token, then call `graph.get` and `graph.validate` again before Apply. Skip only when the user explicitly wants the existing manual node layout preserved.
19. Call `graph.apply` when the user explicitly requests Apply or asks to implement, fix, port, complete, or make the graph work. Pass the current Studio session, graph revision, and draft token. After Apply, re-read `context.get` and require the expected graph ID/revision plus `runtime.applied == true`; never report completion from Stage alone.
20. Never run Preview or Play. Stop after structural validation, persistence read-back, and runtime-status inspection; the user performs all visual and runtime testing.
21. Before Apply when the candidate uses dynamic-target nodes, call `runtime.status`. If the installed runtime is old or cannot execute any applied node type, report the exact incompatibility and obtain explicit authorization for `runtime.generate`; never claim the graph visually or behaviorally works from validation alone.
22. Call `runtime.generate` only when explicitly requested or when the user asked for a complete Play-ready installation and the gateway reports it is required.

## Route selection

Prefer built-in nodes and bundled techniques before custom executable techniques. Choose the narrowest route:

1. One built-in node when it directly expresses the behavior.
2. A small built-in node chain when flow or conditions are required.
3. A bundled recipe or technique when it already owns the mechanic.
4. A harness when grouping, reusable proxy ports, or a callable lifecycle is needed.
5. A visual group when Animate Children or Frame In ordering is the actual problem.
6. A declarative recipe or Harness Preset for reusable graph structure.
7. A custom technique only when the existing catalog cannot express the behavior without unreasonable duplication or missing runtime logic.

Treat every available validated catalog entry as a reusable route, including saved global and graph-local custom techniques. Reuse a suitable existing custom technique before authoring a new one.

For one distinct known `GuiButton` visual, inspect Button Preset first. Before expanding the same preset across two or more siblings, perform the repeated-sibling structural audit and inspect the dynamic-target route. Never clone an isomorphic per-button subgraph unless the query route is impossible for a stated contract reason and the user explicitly accepts static duplication. Motion Lab does not expose or use `GuiButton.Activated`; never invent or recommend a `trigger.activated` node.

For repeated or changing UI trees, inspect `target.query`, `flow.for-each-target`, `target.find-child`, and the ordinary target triggers before duplicating static motion branches or declaring dynamic targeting unavailable. Prefer the explicit `Query Targets -> For Each Target -> ordinary trigger -> action/call` topology when the user wants a fully editable graph. An unactivated For Each Body configures each ordinary trigger's persistent listener, and For Each Current Target binds only the trigger. Every ordinary target trigger exposes the actual instance that fired as Event Target; bind all downstream actions, conditions, Find Child nodes, and Graph Function TargetRef inputs to that trigger output. For a Graph Function call, connect `trigger.out -> call.in` and `trigger.current` (shown as **Event Target**) `-> call.target` (shown as **Event Target**); ordinary Flow never replaces the separate TargetRef edge, and neither edge may terminate at the other's input. Never bind a downstream node directly to For Each Current Target after an event trigger. `trigger.target-event` / On Each Target Event is a backward-compatible fused shortcut, not the preferred replacement for an explicitly requested For Each plus ordinary-trigger graph. Motion Lab never exposes Activated. Use Button Preset for every known `GuiButton`.

Treat the visible **Target** `TargetRef` input as the canonical single-target connection on flow-driven target consumers, except for `target.find-child.outputs.found`. Find Child **Found** is one visible success connection that carries both Flow and its found `TargetRef`; connect `Found -> In` once and let Motion Lab supply the hidden compatible Target/parent/function input. When a consumer has exactly one compatible Flow input and one TargetRef input, a user may drop **Found** on either **In** or **Target**; Motion Lab normalizes both gestures to the same single `Found -> In` payload wire. Never add a second visible `target.find-child.outputs.target` edge or a separate `Next` continuation. For every other producer, connect its TargetRef output to **Target** and do not author a hidden `node.target = NodeOutput` descriptor when the live catalog exposes **Target**. Use **Targets** only for a real `TargetSet`, and use inspector bindings only for fixed Studio instances. Require exactly one of inspector binding, Target, or Targets. Applied runtime metadata may compile either a visible Target wire or Find Child's hidden Found payload to a NodeOutput descriptor internally.

Treat structural outliers as a user decision, not an excuse to brute-force the graph. Report each exact path and mismatch, then ask whether to normalize the hierarchy, exclude the outlier, or keep one explicit exception branch. Never silently create missing instances, reinterpret the event surface, or duplicate the shared graph around an outlier.

After the user normalizes a hierarchy, re-inspect the live targets and recompute the smallest route. Do not implement a previously proposed custom node, parent traversal, or exception branch when the new structure makes it unnecessary. When every repeated visual root is itself the compatible event surface, query those roots directly, use For Each Current Target only to configure ordinary listeners, and bind shared child lookups from each trigger's Event Target.

Treat Query Targets' `expectedClass` as the class guarantee for its result. Do not add `condition.class` / Class Is after that query unless the graph intentionally mixes a broader class set and needs a runtime branch. Before staging any repeated event graph, verify each ordinary trigger has both Flow and Event Target connections downstream, and reject the draft if a caller/action is targeted directly from For Each Current Target instead of the trigger that fired.

When several activation-local `TargetRef` outputs need the exact same action parameters and timing, connect them to distinct dynamic inputs on `target.collect`, then connect its `TargetSet` output to one Many-cardinality action. Use `target.resolve-map` once when the same event target has several required named descendants; keep optional descendants or branches with different parameters, timing, or yield behavior separate. Use Multiple Bindings only for a fixed authored instance list, never as a substitute for Event Target or other activation-local references.

Ambient effect nodes support two explicit lifecycles. With no incoming Flow, **Action = Start** is a passive persistent effect and its targets must come from fixed inspector bindings or a static Query Targets set. With incoming Flow, connect one activation-local **Target** or **Targets** source and choose **Start** or **Stop**; Start and Stop cards match by effect kind, Tag, and runtime target. Use Ambient Controller only for tag-wide Suspend/Resume. Keep passive effects outside Graph Functions; flow-controlled effects and Ambient Controller may be used inside them.

During capability audits, separate internal runtime guarantees from graph-visible controls. Do not call generation guards, tween ownership, or cleanup unsupported merely because they are not nodes; inspect the live contract and report rapid re-trigger behavior as unverified until the user tests it. Propose a new activation or cancellation policy only when the requested behavior differs from the existing scheduler policy.

Distinguish an exact port from a reusable relative-value mechanic. Inspect authored target values first; concrete endpoints may fully cover a fixed UI. For reusable live-relative motion, use `motion.scale` with `fromMode = Current` plus `valueMode = AuthoredMultiplier`, or use a Number `motion.tween-property` with `fromMode = Current` plus `valueMode = AuthoredOffset` (for example, authored `Rotation = 20` and `To = 10` targets `30`). Report a relative-value limitation only when the required formula is outside these supported modes.

Use the built-in `flow.wait` node shown as **task.wait** in the Workflow category for a fixed flow delay. Do not create a custom technique or script merely to delay a visual branch.

Use **Print** (`debug.print`) and **Warn** (`debug.warn`) only to prepare user-run visual-flow diagnostics. They are targetless, emit one configured message when the user runs the graph, and continue immediately. For a silent graph, place Print directly after the suspected trigger first, then after the first branch or lookup only if needed; this distinguishes a dead trigger from a dead downstream path when the user tests it. Remove temporary diagnostics only after the user confirms the cause unless they ask to keep them. Debug nodes never replace gameplay or network communication.

Never choose a custom technique merely to reduce the visible node count.

## Post-cleanup audit

Run a post-cleanup audit when the user asks to optimize, simplify, compact, or clean an existing graph. Also run it before Stage after authoring or materially changing a non-trivial graph. Skip the audit only for tiny edits where no repeated structure was introduced.

Treat cleanup as behavior-preserving graph work, not permission to redesign the motion.

### Baseline

1. Read the exact locked graph with `graph.get` and validate the unchanged graph.
2. Record node, edge, binding, Harness, and cap headroom counts.
3. Count node types, repeated exact parameter signatures, and repeated target paths.
4. Measure downstream reachability per trigger and identify branches that are duplicated rather than shared.
5. Separate flow edges, TargetSet/TargetRef data edges, and target descriptors so plumbing is not mistaken for behavior.
6. Inspect the live target hierarchy for every repeated sibling and structural outlier.

### Required normalization gate

Complete this gate before Stage for every non-trivial authored or cleaned graph:

1. Within each root or Graph Function scope, group `target.find-child` nodes by parent TargetRef source, activation source, Any Name, child name when applicable, expected class, Yield, and missing-flow contract. Reuse one lookup and fan out its **Found** payload when the complete signature matches. With Any Name enabled, require the runtime path to call `FindFirstChildOfClass(expectedClass)` directly.
2. When one activation needs several required descendants from the same dynamic parent, prefer one `target.resolve-map` over separate `target.find-child` chains. Keep separate lookups when any role is optional or has different missing-flow behavior.
3. Group Motion nodes by scope, activation source, node type, complete normalized parameters, Yield, completion flow, and cancellation ownership. When only explicit bindings differ and the live node cardinality is Many, combine the targets on one node with **Bind Selection** or **Add Selection**.
4. Never reuse a run-local TargetRef across root/Graph Function boundaries or independent activations.
5. Run `graph.validate` and treat `DUPLICATE_TARGET_LOOKUP` and `MERGEABLE_ACTION_TARGETS` warnings as required cleanup. Retain one only when equivalence is impossible, and report the exact semantic difference.
6. Recount nodes and edges after normalization. Do not Stage while an unexplained exact duplicate remains.

### Candidate selection

Use only node types, techniques, recipes, Harnesses, and mechanics advertised by the live catalog and capabilities. Never propose an unavailable node as an immediate cleanup route.

Look for these bounded candidates:

- Replace isomorphic per-sibling branches with `target.query`, `flow.for-each-target`, or `trigger.target-event` only when event surfaces, target classes, parameters, ordering, and missing-target behavior are equivalent.
- Replace repeated identical work over a discovered collection with one shared Body. Preserve Sequential, Parallel, Staggered, or Dependency behavior exactly.
- Reuse a suitable validated built-in, recipe, saved custom entry, or Harness Preset when it owns the complete mechanic and does not remove required editability.
- When the live catalog exposes a native multi-target, target-map, multi-property, state, or callable-subgraph route, compare its complete contract before using it.
- Recommend Harnesses or Warps only as readability improvements. State explicitly that they do not reduce real node count or runtime work.
- Remove only nodes and edges made redundant by the proven replacement. Preserve unrelated layout, metadata, definitions, and bindings.

Do not combine branches when any of these differ:

- trigger or event phase;
- event surface or visual root;
- target path, expected class, or ScreenGui;
- flow order, edge order, Yield, delay, or completion lifecycle;
- required versus optional lookup behavior;
- property, value mode, authored/current capture, easing, or duration;
- cancellation, re-entry, or restoration ownership;
- per-target values, conditions, or callable-scope ownership.

Never delete behavior because it appears unreachable from the screenshot. Never replace explicit per-target differences with one generic action. Never use a custom executable technique solely to hide or reduce nodes.

### Proposal and authorization

Before changing the graph, show one compact cleanup proposal containing:

- baseline node/edge counts and cap headroom;
- each exact repeated pattern;
- the live supported replacement;
- estimated node/edge reduction;
- preserved semantics;
- behavior that cannot be compacted with the current catalog;
- every structural outlier or user choice.

Ask the user to confirm any material rewrite. Do not Stage or Apply a guessed cleanup.

### Equivalence and verification

For an authorized cleanup:

1. Build the smallest candidate from the current graph revision.
2. Require the same trigger set, event phases, target classes, property writes, flow order, Yield behavior, and cancellation contract.
3. Call `graph.validate` and resolve every error and behavior-affecting warning.
4. Call `graph.diff`; reject unexpected removals or metadata changes.
5. Report before/after node and edge counts. Do not claim runtime speedup from graph-size reduction alone.
6. Stage, organize, read back, and validate according to the core workflow.
7. Apply only under the normal authorization rules.
8. Report remaining unsupported duplication separately from completed cleanup.

## Visual and gameplay boundary

Keep Motion Lab responsible for presentation: visual triggers, conditions, flow, property motion, transitions, and restoration. Motion triggers start visual graph branches; they are not gameplay communication.

Keep gameplay actions and communication out of Motion graphs and custom techniques. Never fire a `RemoteEvent` or `BindableEvent`, call gameplay APIs, or mutate gameplay state from Motion behavior.

When a button must communicate with another game script:

1. Build its visual interaction with Button Preset and the smallest supporting Motion graph.
2. Inspect the existing game-script communication path through Roblox Studio MCP and reuse its event or API when available.
3. Create or edit a separate `LocalScript` through the official Roblox Studio MCP. Prefer an existing button integration script when it already owns the action; otherwise create the smallest appropriately placed `LocalScript`.
4. Connect `GuiButton.Activated` in that `LocalScript` for the communication itself. Motion Lab must not handle Activated; its Button Preset or dynamic visual events remain presentation-only.
5. Use the correct inspected integration: `RemoteEvent:FireServer(...)` for client-to-server requests, `BindableEvent:Fire(...)` for same-context signaling, or an existing ModuleScript API. Never invent `FireEvent` or an event path.

Keep visual verification and gameplay verification separate, and leave both execution steps to the user. Report what the Motion graph and integration script structurally contain without claiming observed behavior.

## Mutation rules

- Keep Stage separate from Apply.
- Treat `graph.organize` as staged-draft cleanup, not Apply. Require the current clean draft token and keep the returned organized draft token for later Apply.
- Keep Apply separate from Generate Runtime.
- Lock the exact intended graph owner through `graph.lock` before `graph.get`, catalog authoring, binding resolution, Stage, or Apply. If selection must change, unlock intentionally, select the new owner, verify it through `context.get`, then lock again before continuing.
- Never apply a stale draft token or revision.
- Never invent node type IDs, port IDs, parameter IDs, guide IDs, or template fields.
- Never silently resolve an ambiguous instance path.
- Never bind across the graph target's `ScreenGui`.
- Never animate Position or Size when a parent layout owns that property.
- Never modify an unrelated graph, custom definition, preset, or runtime package.
- Never port a script into a material graph from assumed intent. List the observed effects and ask about ambiguous, skipped, unreachable, or structurally exceptional behavior before Stage.
- Never suppress gateway errors by retrying a different target.
- If a preview is active, ask the user to stop it before operations that require authored state. Never stop or reset it yourself.
- Treat `visual-group.configure` as an immediate undoable metadata write. Call it only with explicit user authorization after `visual-group.inspect`; it is not part of graph Stage or Apply.

## Custom authoring

Before custom technique, harness, group, recipe, or preset work:

1. Call `guide.index`.
2. Fetch the exact guide with `guide.get`.
3. Fetch the current machine-ready skeleton with `template.get`.
4. Follow [authoring-workflows.md](references/authoring-workflows.md).
5. Validate before staging or saving.

Custom technique source is executable. Review the live guide, requested scope, dependencies, callbacks, and trust state. Prefer one-time scope for graph-specific behavior and global scope only for intentional reuse.

## Verification

- Treat successful gateway validation as structural evidence, not visual evidence.
- Never run Preview, enter Play, simulate input, or inspect runtime behavior as a test.
- Use graph read-back and `runtime.status` only to verify structure, persistence, version readiness, and generation requirements.
- Report staged, applied, and runtime-generated state distinctly, and explicitly mark appearance and runtime behavior as not tested by the agent.

## Response discipline

Return concise user-facing summaries. Include:

- Target and graph identity.
- Chosen route and why it was selected.
- Material nodes, edges, bindings, harnesses, groups, techniques, or presets changed.
- For cleanup work, before/after node and edge counts plus any deliberately retained duplication.
- Whether the final staged graph was organized or its manual layout was intentionally preserved.
- Validation warnings that affect behavior.
- Whether the result is staged, applied, or runtime-generated, plus that visual/runtime testing remains with the user.

Do not dump full gateway JSON unless the user asks.
Normal gateway JSON follows the **Necessary Data** rule: return only what the current operation needs; fetch full descriptions explicitly.
