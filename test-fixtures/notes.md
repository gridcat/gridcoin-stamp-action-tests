# Test fixture: notes.md

A second test fixture attached to `with-uploads` (and any other scenario
with `upload_fixtures: true`) e2e releases. Used alongside `hello.txt`
and `data.json` to verify that `gridcoin-stamp-action` correctly
iterates over multiple pre-existing release assets, downloads each,
computes their SHA-256 hashes, and stamps them on the Gridcoin
blockchain via stamp.gridcoin.club.

This file's content is deliberately deterministic (no timestamps, no
build metadata) so each run of an e2e scenario produces the same
SHA-256 for this asset. That lets the rerun-idempotency scenario verify
that the second stamping attempt recognizes the hash as already stamped
instead of submitting it again.
