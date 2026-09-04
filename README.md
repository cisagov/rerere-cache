# Shared `rr-cache` — cisagov/rerere-cache

This repository's *contents* are a `git rerere` cache: one
subdirectory per conflict fingerprint, each containing `preimage`
(the normalized conflict) and `postimage` (the recorded resolution).
It is the write-through cache behind the cisagov skeleton
linearization campaign, and afterwards the thing that makes every
skeleton sync a rebase that resolves itself.

Also here:

- `trained/<repo>` — training stamps: the tip SHA each repo's merge
  history was last trained at. Delete one to force a retrain.
- `.gitignore` (`thisimage*`) — rerere's reuse path drops `thisimage`
  scratch files into entry dirs; they are never resolution data and
  must not be committed.

## How to use it

The linearization harness wires this up for you:
`scripts/setup-shared-cache.sh` clones this repo (or attaches
`origin` to an existing copy), and every `linearize.sh` run symlinks
the work clone's `.git/rr-cache` here, trains, and commits new
entries. `campaign.sh run` fast-forwards from `origin` first.

Manual equivalent, in any skeleton-descendant clone:

```bash
rm -rf .git/rr-cache        # only if empty; move aside otherwise
ln -s /path/to/this/dir .git/rr-cache
git config rerere.enabled true
git config rerere.autoUpdate true
git config merge.conflictstyle merge   # fingerprints depend on
                                       # conflict TEXT — one style
                                       # teamwide or entries won't
                                       # cross-apply
git config gc.auto 0                   # `git gc` runs `rerere gc`,
                                       # which would prune THIS cache
                                       # through the symlink
```

Now every merge/rebase/cherry-pick in that clone consults — and
contributes to — this shared cache.

## Rules

- **Push only via `scripts/cache-push.sh`.** Entries are literal file
  content (conflict pre/postimages) from every repo trained in; the
  script refuses to push when any `trained/` repo is private or
  undetermined, so private-repo content never leaks.
- **Pull before, push after** each linearization or skeleton sync
  (the harness does the pull). Entries are keyed by SHA-1
  fingerprint, so parallel work merges cleanly. A genuine conflict on
  `postimage` means two repos resolved the same hunk differently —
  pick the right one deliberately.
- **Never run `git rerere gc` here** — and keep `gc.auto=0` in linked
  clones, since plain `git gc` runs it implicitly. If pruning is ever
  needed, do it via a reviewed commit, not `gc`.
- **Do not hand-edit `postimage` files.** Fix a bad resolution in a
  work tree with `git checkout --merge <file>; git rerere forget
  <file>`, resolve correctly, and let rerere re-record.

## Provenance

Every commit message names the repo whose training or linearization
contributed the entries; `trained/` records the exact tip trained.
The campaign's per-stop human-readable logs live in the harness repo
under `campaign/logs/<repo>-resolutions.md`.
