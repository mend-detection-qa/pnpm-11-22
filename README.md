# workspace-schema-prune-v11

## Feature exercised

Two co-exercised pnpm 11.22 behaviors that together affect how
Mend UA discovers and resolves workspace dependencies:

1. **`pnpm-workspace.yaml` schema hardening** — the `settings:`
   block in `pnpm-workspace.yaml` now rejects or warns on keys
   that were previously silently accepted in pnpm ≤ 10. This
   probe uses `onlyBuiltDependencies: []` inside `settings:` to
   confirm pnpm 11 handles it gracefully and that Mend's YAML
   parser does not abort workspace discovery when it encounters
   an unexpected key in that block.

2. **`minimumReleaseAgeExcludePrune`** — a new `"pnpm"` section
   key in the root `package.json` that prevents the stale-package
   prune pass from removing named packages (here: `"zod"`) during
   `pnpm install`. This probe confirms that Mend's config-object
   parser does not choke on the new key and still surfaces the
   full resolved tree.

## Project layout

```
workspace-schema-prune-v11-20260815-173917/
├── package.json              # root; pnpm.minimumReleaseAgeExcludePrune
├── pnpm-lock.yaml            # lockfileVersion 9.0
├── pnpm-workspace.yaml       # packages glob + settings.onlyBuiltDependencies
├── .whitesource              # Bucket A versioning pins
├── README.md
├── expected-tree.json
└── packages/
    ├── server/
    │   └── package.json      # @probe/server — fastify + zod
    └── client/
        └── package.json      # @probe/client — hono + zod
```

## Expected dependency tree

- Root project `workspace-schema-prune-v11` has no production
  dependencies. Mend should detect it via the importers section.
- `@probe/server` direct dependencies:
  - `fastify@4.28.1` (registry)
  - `zod@3.23.8` (registry)
- `@probe/client` direct dependencies:
  - `hono@4.4.13` (registry)
  - `zod@3.23.8` (registry, same resolved version — shared entry)
- `fastify@4.28.1` transitive dependencies (partial):
  `@fastify/ajv-compiler`, `@fastify/error`,
  `@fastify/fast-json-stringify-compiler`, `abstract-logging`,
  `avvio`, `fast-content-type-parse`, `fast-json-stringify`,
  `find-my-way`, `light-my-request`, `pino`, `process-warning`,
  `rfdc`, `secure-json-parse`, `semver`, `tiny-lru`
- `hono@4.4.13` has no transitive dependencies.
- `zod@3.23.8` has no transitive dependencies.

## Mend failure modes targeted

| Failure | Cause |
|---|---|
| Workspace packages not detected | Mend YAML parser aborts on the `settings:` block in `pnpm-workspace.yaml` |
| All packages missing | `minimumReleaseAgeExcludePrune` key causes the `"pnpm"` config object to be misparsed, dropping peerDependencyRules and misreading importers |
| Tree identical regardless | Neither feature changes what is *installed*; Mend reads the lockfile. Both failure modes are parse-time, not resolution-time. |

## Mend config

**Bucket A** — `js-pnpm` has no dynamic version detection from the
manifest. This probe pins exact tool versions in `.whitesource`:

```json
{
  "scanSettings": {
    "configMode": "AUTO",
    "versioning": {
      "pnpm": "11.22.0",
      "node": "20.11.1"
    }
  }
}
```

`configMode` is `"AUTO"` because no `whitesource.config` ships with
this probe.

## Probe metadata

| Field | Value |
|---|---|
| pattern | `workspace-schema-prune-v11` |
| pm | pnpm |
| pm_version_tested | 11.22.0 |
| lockfile_version | 9.0 |
| categories | manifest_format, install_command |
| source | https://github.com/pnpm/pnpm/releases/tag/v11.22.0 |
| generated_at | 2026-08-15T17:39:17Z |
