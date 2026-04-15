# gridcoin-stamp-action-tests

E2E test harness for [gridcoin-stamp-action](https://github.com/gridcat/gridcoin-stamp-action). Creates throwaway releases and runs the action against them.

## Setup

Store a fine-grained PAT with `Contents: Read and write` as repo secret **`PAT_TOKEN`**. Required because releases created by the default `GITHUB_TOKEN` don't fire `release: published` on downstream workflows.

## Usage

Actions → **Create test release** → Run workflow → pick scenario → Run. This creates a release tagged `e2e-<scenario>-<timestamp>` which auto-fires `stamp.yml`.

| Scenario | Inputs overridden |
|---|---|
| `basic` | — |
| `with-uploads` | — (fixtures always attached) |
| `no-source-archives` | `include-source-archives=false` |
| `manifest-only` | `include-source-archives=false`, `include-release-assets=false` |
| `wait-for-confirmation` | `wait-for-confirmation=true`, `poll-timeout=600` |
| `rerun-idempotency` | — (manual re-run required) |
| `tag-mutation` | — (manual force-push + re-run required) |

The `upload_fixtures` toggle on `create-release.yml` attaches `test-fixtures/*` to any scenario regardless of its default.

## Special scenarios

### `rerun-idempotency`

1. Run scenario `rerun-idempotency`, wait for `stamp.yml` to finish.
2. Actions → Stamp test release → that run → **Re-run all jobs**.
3. Second run's log should show `reusing uploaded bytes` / `already stamped` for every artifact. If you see `Producing` or `Submitting hash`, idempotency is broken.

### `tag-mutation`

1. Run scenario `tag-mutation`, note the generated tag.
2. Force-push the tag to a different commit:
   ```bash
   git fetch --tags
   echo "bump" >> README.md && git commit -am "bump"
   git tag -f <the-tag>
   git push --force origin <the-tag>
   ```
3. Re-run `stamp.yml` from the Actions UI.
4. Expected: run **fails** with `Error: Tag mutation detected: ...`. If it succeeds, the preflight is broken.

## `tag:` input scenario (no release event, no PAT)

Separate entry point: Actions → **Stamp by tag (no release event)** → Run workflow.

Exercises the `tag:` input path that goreleaser and same-workflow semantic-release users land on. Self-contained single job — creates the release with the default `GITHUB_TOKEN` (which deliberately does **not** fan out `release: published` to `stamp.yml`) and then calls the action in the same job with `tag: ${{ steps.tag.outputs.tag }}`.

**Expected:** the run stamps the release successfully without a PAT, and `stamp.yml` stays idle for the generated `e2e-tag-input-*` tag. If `stamp.yml` also fires, either GitHub changed its `GITHUB_TOKEN` fan-out rules or `stamp.yml` got wired to a non-release trigger — investigate before merging.

This is the only scenario that proves the `tag:` input works; the payload-based path is covered by every other scenario via `stamp.yml`.

## Cleanup

Actions → **Cleanup old e2e releases** → Run workflow. Deletes `e2e-*` releases and their tags older than `older_than_days` (default 7). Not scheduled — run manually.

## Known first-run gotchas

- `stamp.yml` pins `gridcat/gridcoin-stamp-action@v1`. If no `v1` tag exists on the action repo yet, switch to `@main` temporarily.
- If `GITHUB_TOKEN` default permission in repo settings is read-only, the action's upload step 403s. Set it to Read and write, or rely on the job-level `permissions: contents: write`.
- Fixtures are deterministic (no timestamps, no random bytes) so rerun-idempotency testing produces identical SHA-256s across runs.
bump
