# How to push this update

I can't push to the repo myself (no authenticated git access from here), so
these are the steps for you. Three files are ready in the outputs folder:

- `index.html` — the updated app (this replaces the current one)
- `COMMIT_NOTES_feature_set_4.md` — the commit note, repo's existing style
- `README_updated.md` — the README with Feature Set 4 changes folded in

## Recommended: branch + PR (lets your colleague review before it goes live)

Since this is your colleague's deployed file and GitHub Pages publishes
`main` automatically, a branch + pull request is the safe path — it does
**not** change the live site until merged.

```bash
# from a fresh clone or your existing local checkout, on an up-to-date main
git checkout -b feature-set-4-ai

# copy the three files in from wherever you downloaded them:
#   index.html                      -> repo root (overwrite)
#   README_updated.md               -> repo root, save as README.md (overwrite)
#   COMMIT_NOTES_feature_set_4.md   -> repo root (new file)

git add index.html README.md COMMIT_NOTES_feature_set_4.md
git commit -F COMMIT_NOTES_feature_set_4.md   # uses the prepared message
git push -u origin feature-set-4-ai
```

Then open a PR from `feature-set-4-ai` into `main` on GitHub and let your
colleague review. Merging it triggers the GitHub Pages redeploy.

> Note: `git commit -F` uses the whole file as the message. If you'd rather
> keep the commit message short, open `COMMIT_NOTES_feature_set_4.md` and
> copy just the fenced `Commit message` block into `git commit -m "..."`.

## Alternative: straight to main (publishes immediately)

Only if you want it live now and don't need review first:

```bash
git checkout main && git pull
# copy the three files in as above
git add index.html README.md COMMIT_NOTES_feature_set_4.md
git commit -F COMMIT_NOTES_feature_set_4.md
git push origin main
```

## Before you push — worth doing

1. **Test it live once.** None of this was exercised in a real browser from
   my side. Serve the folder (`python3 -m http.server 8080`), open it in
   Chrome/Edge, load a WebLLM model, and run one pass: save an entry, open
   it, then try Auto title/tags, Suggest questions, Find related threads,
   Think it through, and the AI myth scan. Each should stream and finish.
2. **Hard-reload after deploy** (or clear site data) — the ES module and the
   model are cached, so a stale cache can mask the update.
3. **The `.zip` and `.patch` files** already in the repo (e.g.
   `loob-transcription-prototype-updated.zip`, the `ios_*.patch` files) are
   from earlier work and are untouched by this update. If they're meant to
   track the latest build you may want to refresh or remove them separately —
   I left them alone rather than guess.

## Not included

- I did not modify `.nojekyll`, the `.patch` files, or the `.zip`.
- The `web-llm` import is still unpinned (see the commit note and README).
  Pinning it is the recommended follow-up once you have the known-good
  version from whoever built the original — that's a one-line change to the
  import URL in `index.html`.
