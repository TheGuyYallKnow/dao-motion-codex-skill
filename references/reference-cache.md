# Reference Cache Authoring

Use Reference Cache for a stable graph-scoped reference tree that several branches or activations reuse. Keep one-off and activation-local lookups as Find Child or Resolve Target Map.

## Node pair

- `target.reference-cache` owns one `cacheId` and a dense ordered `entries` array. With no incoming Flow it initializes at graph start; with incoming Flow it initializes on first execution.
- `target.reference-linking` selects one entry with the same `cacheId` plus `entryId`. Its `out` Flow carries the cached TargetRef payload to a compatible downstream `In`, `Parent`, or `Target` route.

## Canonical entry shape

Use only the live schema fields below, then require `graph.validate` before Stage:

```text
entryId             stable unique ID inside this cache
referenceName       unique user-facing name
parentSource
  kind              Normal | CachedEntry
  value             GraphTarget descriptor or parent entryId
childNameBinding    literal child name string, or another live-supported binding value
ignoreName          boolean
expectedClass       Roblox class name
displayOrder        finite ordering number
```

Normal graph-root parent:

```json
{
  "kind": "Normal",
  "value": { "kind": "GraphTarget" }
}
```

Cached parent entry:

```json
{
  "kind": "CachedEntry",
  "value": "parent-entry-id"
}
```

Do not use editor labels such as `displayName`, or legacy aliases such as `name`, `anyName`, or raw `childName`, in newly authored entries. Re-read the live catalog/template and stop if the installed schema rejects these canonical fields.

## Compaction workflow

1. Group fixed Find Child nodes by graph-scoped root and exact hierarchy.
2. Add one cache entry per unique path segment so descendants can reference cached parents.
3. Add Reference Linking only for entries consumed by live branches; intermediate parent-only entries do not need readers.
4. Replace each lookup branch with `upstream Flow -> Reference Linking.in -> downstream In`.
5. Preserve timing, Yield, missing-reference suppression, expected class, and downstream parameters.
6. Remove the old Find Child nodes and only their incident edges.
7. Validate, diff, Stage, organize, read back, validate again, then Apply when authorized.

Reference Cache compacts lookup structure; it may not reduce node count because each independently timed branch still needs one typed Reference Linking reader. Report node and edge counts honestly.
