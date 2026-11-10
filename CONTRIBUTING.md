# Contributing

## Making a change

1. Branch from `main`. Edit only the `<application>/<environment>/values.yaml` you intend to change.
2. Validate locally (see README). The PR pipeline repeats it and blocks merge on failure.
3. One PR may touch several application/environment files. The pipeline processes each affected deployment independently and never deploys unrelated ones.
4. Get a review from the application owner. PROD paths additionally need a platform reviewer (CODEOWNERS).
5. After merge the pipeline deploys DR first. A person with approver rights then promotes Primary for that exact commit.

## Rules for reviewers

- Reject any site-specific key (ID, locator, endpoint, hostname, certificate).
- Reject secrets of any kind.
- Treat these as high risk and ask for a rollout plan: `totalNumBuckets` (cannot change without recreating the region), region renames or removal, `redundantCopies` reductions, `ttl` shortening (deletes data sooner), `pdx` changes, storage class changes, `memberCount` reductions (needs the gated scaling workflow).
- Shortening storage is not possible in place; growth depends on StorageClass volume expansion.

## File deletions and renames

- Deleting or renaming a values file never triggers a teardown. The pipeline fails the run and asks for a manual decommission request.
- A renamed path is treated as a new application/environment and must start with a fresh DR deployment.

## Conventions

- 2-space YAML indent, no tabs, comments explain non-obvious values.
- Keep the `yaml-language-server` header on the first line.
- PROD: requests equal limits for memory; `ignoreUnreadFields: false`.
