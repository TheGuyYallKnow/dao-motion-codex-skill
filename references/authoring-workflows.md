# Dao Motion Advanced Authoring Workflows

## Contents

1. General decision rule
2. Custom techniques
3. Harnesses
4. Stagger Loop Harnesses
5. Visual groups
6. Declarative recipes
7. Harness Presets
8. Advanced Text
9. Validation and verification

## 1. General decision rule

Use built-in nodes, recipes, and bundled techniques before custom authoring. Fetch `catalog.index`, compare `useWhen` and `avoidWhen`, and read `catalog.describe` for plausible routes.

Never run Motion Preview, Advanced Text Preview, or Play mode. Use structural validation and read-back only; the user performs all visual and runtime testing.

Use advanced authoring only when it provides a real mechanic:

- Custom technique: reusable or graph-specific executable behavior unavailable in the catalog.
- Harness: editor organization and reusable proxy-port structure around two or more nodes.
- Stagger Loop Harness: explicit run/stop loop lifecycle with callable body branches.
- Visual group: deterministic ordering and inclusion of children for Frame In or Animate Children.
- Recipe: reusable declarative graph structure with exposed parameters.
- Harness Preset: reusable harness structure plus binding hints.

## 2. Custom techniques

### When to use

Use a custom technique when required initialization, activation, runtime polling, cleanup, or dependency behavior cannot be expressed safely with existing nodes. Do not use it simply to hide a small graph.

### Scope selection

- Choose one-time `local.` scope for behavior owned by one graph target.
- Choose global `custom.` scope only when multiple graphs intentionally share the same stable contract.
- Fork a bundled technique when its behavior is the correct base but a maintained custom variant is required.

### Steps

1. Fetch `guide.get("custom-technique")` and `template.get("custom-technique")`.
2. Call `context.get` and confirm the intended graph owner and Studio mode.
3. Recheck `catalog.index` to prove no built-in route already fits.
4. Define immutable namespaced ID, display name, description, category, type, supported target classes, target cardinality, and tags.
5. Declare only necessary dependencies: BatchService and/or VirtualizerGateway.
6. Declare bounded adjustables using supported kinds: Number, Boolean, Enum, Direction, Vector2, Object Binding, or Target Selector.
7. Implement the required callbacks from the live template:
   - `Initialize`
   - `Activation`
   - Runtime-only `Condition` and `Update`
   - Optional runtime `Deactivation`
8. Use only documented context helpers for owned helpers, tween/animation completion, references, and dependencies.
9. Call `technique.validate`. Review source errors, metadata errors, dependency errors, and trust requirements.
10. Stage the technique draft in the authoring UI when supported.
11. Save only after validation and explicit acceptance of the executable-code trust boundary.
12. Add the validated technique node to a graph, bind it, validate the graph, and leave execution testing to the user.

### Important constraints

- Nested executable children are forbidden.
- Dirty source cannot Apply.
- IDs become immutable after validation.
- Breaking metadata or parameter changes on referenced definitions require a new ID/version strategy.
- One-time techniques must remain owned by their graph target.
- Global techniques belong under the exact plugin-owned global root.
- Never copy a custom technique into emitted runtime manually; the Emitter owns packaging.

## 3. Harnesses

### When to use

Use a harness to make a meaningful subgraph movable, collapsible, and reusable through proxy ports. Do not group unrelated nodes merely because they are nearby.

### Steps

1. Fetch `guide.get("harness")` and `template.get("harness")`.
2. Select at least two graph nodes and verify none already belongs to another harness.
3. Choose a bounded descriptive name that states the mechanic.
4. Call `harness.create` with exact member IDs.
5. Inspect generated proxy inputs and outputs.
6. Expand the harness to repair internal edges; collapse it to verify the public interface.
7. Add or remove members only when the harness still represents one coherent mechanic.
8. Validate the entire graph after every membership or proxy-port change.
9. Duplicate or save as a Harness Preset only when intentional reuse exists.

### Important constraints

- One node belongs to at most one harness.
- Normal harnesses require at least two members.
- Harness IDs/names and total harness count are bounded by the live schema.
- Ungroup preserves member nodes; delete-with-members intentionally removes them.
- Never treat a Stagger Loop Harness as a normal harness.

## 4. Stagger Loop Harnesses

### When to use

Use only for a repeatable callable body that needs Run, Stop, Ran, Stopped, or Completed lifecycle semantics.

### Steps

1. Fetch `guide.get("stagger-loop-harness")` and its template.
2. Create the special harness and its required controller together.
3. Configure interval, loop count (`0` means infinite), and cycle delay.
4. Connect Run and optional Stop inputs.
5. Build the callable Body branch inside the harness.
6. Connect lifecycle outputs only when consumers need them.
7. Validate finite/infinite behavior and stopping behavior.
8. Validate bounded loop counts and Stop wiring structurally; leave finite and infinite execution tests to the user.

The controller cannot be removed separately, the harness cannot be ungrouped, and ordinary Harness Preset behavior may not support this special lifecycle.

## 5. Visual groups

### When to use

Use visual groups for Animate Children or Frame In macros when child ordering, inclusion, exclusion, or descendant grouping changes the animation mechanic.

### Steps

1. Fetch `guide.get("visual-group")` and inspect the active node with `visual-group.inspect`.
2. Choose Immediate GUI Children or Descendant GUI Objects intentionally.
3. Review resolved groups and deterministic order.
4. Exclude plugin-owned helpers and non-visual descendants.
5. Apply only requested include/exclude overrides.
6. Reorder only when visual timing requires it.
7. Preserve unrelated `DaoMotionOrder` and group metadata.
8. With explicit user authorization, call `visual-group.configure` for the exact inspected group. This writes undoable include/order metadata immediately; then revalidate and read back the graph.

Default ordering uses explicit Motion order, layout order, visual position, name, and stable tree order. Do not replace it with arbitrary alphabetical ordering.

## 6. Declarative recipes

### When to use

Use a recipe for reusable graph structure that does not require custom executable runtime code. Prefer recipes over custom techniques when nodes already express the mechanic.

### Steps

1. Fetch `guide.get("recipe")` and `template.get("recipe")`.
2. Select the smallest coherent node subgraph.
3. Define a namespaced recipe ID, display name, category, description, supported root classes, and version.
4. Decide which parameters should be exposed; do not expose every internal value.
5. Capture target placeholders and binding hints.
6. Validate node/edge/parameter/hint caps and root-class compatibility.
7. Stage the recipe and inspect its Motion Library card.
8. Save or Save As explicitly.
9. Instantiate it into a clean draft and verify automatic plus manual binding behavior.

Recipes are declarative configurations under the exact plugin-owned `DaoMotionModules` root. Duplicate IDs, malformed containers, unknown node types, and incompatible roots must remain disabled rather than guessed or executed.

## 7. Harness Presets

### When to use

Use a Harness Preset when the harness structure and its relative binding intent should be reusable on similar UI trees.

### Steps

1. Fetch `guide.get("harness-preset")` and its template.
2. Start from one valid harness with coherent members and proxy ports.
3. Capture binding hints relative to the source graph root.
4. Mark only resolvable in-root targets searchable.
5. Validate hint count, path depth, string bounds, ancestry, and cycles.
6. Save the preset with supported root classes and useful description.
7. Instantiate it on a different compatible target.
8. Review automatic binding outcomes: exact name/class/path, class-only, or unresolved.
9. Confirm or manually repair unresolved mappings before Apply.

Automatic binding must never reuse one destination instance for multiple placeholders or cross into another `ScreenGui`.

## 8. Advanced Text

Fetch the Advanced Text guide before inspecting a `motion.advanced-text` node. Do not open Text Preview. Because safe Advanced Text authoring requires that guarded preview session, stop before modifying the text document and hand that step to the user. Never write `textDocumentJson` directly or from a stale editor session.

## 9. Validation and verification

For every advanced artifact:

1. Validate the definition/artifact itself.
2. Validate the containing graph.
3. Inspect the material diff.
4. Stage before persistent save or Apply.
5. Organize, read back, and revalidate the staged graph.
6. Apply only with a current revision/token.
7. Inspect `runtime.status` without entering Play.
8. Report source scope, binding resolution, warnings, staged/applied status, and that visual/runtime behavior remains user-tested.
