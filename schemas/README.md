# Schema: `application-config.schema.json`

JSON Schema (draft 2020-12) for every `<application>/<environment>/values.yaml`.
Editors that support the `yaml-language-server` header at the top of each file give inline validation.

## What the schema enforces

- Required fields, types, ranges and naming patterns (application, region, index, storage, resources).
- `additionalProperties: false` everywhere, so unknown or misspelled keys are rejected.
- Allow-listed `gemfire.properties` keys only. Identity, security and WAN endpoint properties are rejected.
- Region type is `PARTITION_PERSISTENT_OVERFLOW` only. Eviction action is `OVERFLOW_TO_DISK` only, so persistent data is never destroyed by eviction.
- TTL / idle actions are `DESTROY` or `INVALIDATE` (local actions are invalid for partitioned regions).
- Index types are `FUNCTIONAL` (gfsh `range`) and `PRIMARY_KEY` (gfsh `key`). `HASH` is deprecated upstream and rejected.
- PDX registry must be persistent.
- Environment policy: **prod** requires `redundantCopies >= 1`,
  `pdx.ignoreUnreadFields: false`, `wan.enabled: true`, plus `jvm`, `diskStore`, locator resources and `locatorSize`.
  **dev/stg/qa** still require `redundantCopies >= 1`.

## What JSON Schema cannot express (enforced by `gemfire-k8s-helm/scripts/validate-config.sh`)

| Rule | Reason |
|---|---|
| `application.name` / `environment` equal the folder names | needs the file path |
| Region names unique; index names unique per region | key-uniqueness inside arrays |
| `redundantCopies <= memberCount - 1` | cross-field comparison |
| `evictionHeapPercentage < criticalHeapPercentage` | cross-field comparison |
| `diskUsageWarningPercentage < diskUsageCriticalPercentage` | cross-field comparison |
| `requests <= limits` for CPU and memory; prod `requests == limits` | quantity parsing |
| Heap (`heapPercentOfMemoryLimit` x memory limit) leaves headroom | arithmetic |
| Primary and DR render the same shared-config checksum | deploy-time check |

## Status

- Implemented: this schema and the five example files.
- Tested: structural validation of all five examples with a minimal in-house subset validator (see repo README). Not yet run with a full draft 2020-12 validator.
- Proposed / to verify in Phase 3 against Geode 2.0.x gfsh help: exact gfsh option names behind each key, `FUNCTIONAL` to `range` mapping, and behavior of expiration under WAN replication.
