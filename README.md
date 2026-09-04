# Shared `rr-cache`

This directory is a git repository whose *contents* are a `git rerere`
cache: one subdirectory per conflict fingerprint, each containing
`preimage` (the normalized conflict) and `postimage` (the recorded
resolution).

## How to use it

In any skeleton-descendant clone:

```bash
rm -rf .git/rr-cache        # only if empty; move aside otherwise
ln -s /path/to/this/dir .git/rr-cache
git config rerere.enabled true
git config rerere.autoUpdate true
```

Now every merge/rebase/cherry-pick in that clone consults — and
contributes to — this shared cache.

## Rules

- **Pull before, push after** each linearization or skeleton sync.
  Entries are keyed by SHA-1 fingerprint, so parallel work merges
  cleanly. A genuine conflict on `postimage` means two repos resolved
  the same hunk differently — pick the right one deliberately.
- **Never run `git rerere gc` here.** Default expiry is 60d; during
  the campaign we want everything kept. If pruning is ever needed, do
  it via a reviewed commit here, not `gc`.
- **Do not hand-edit `postimage` files.** Fix a bad resolution in a
  work tree with `git checkout --merge <file>; git rerere forget
  <file>`, resolve correctly, and let rerere re-record.

## Provenance

Seeded by `scripts/linearize.sh` runs; each commit message names the
repo whose linearization contributed the entries. See
`../campaign/logs/<repo>-resolutions.md` for the human-readable log of
what each stop looked like.
