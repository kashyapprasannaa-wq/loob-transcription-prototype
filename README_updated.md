# LOOB — Local Entries Prototype

A single-file, local-first browser prototype for LOOB. Runs Whisper small
entirely client-side via [transformers.js](https://huggingface.co/docs/transformers.js) —
no server, no upload, no network involvement beyond the one-time model
download. Everything below (transcripts, edits, notes, tags) is stored in
this browser's own IndexedDB, on this device, and nowhere else.

## Running it

No build step. Serve the folder over HTTP (opening `index.html` directly as
a `file://` URL breaks ES module imports and microphone access):

```bash
python3 -m http.server 8080
```

Then open `http://localhost:8080` in Chrome or Edge (WebGPU support gives a
real speed boost; it falls back to WASM otherwise). If this is already
hosted via GitHub Pages, just open that URL.

## What's in it

**Capture**
- Record from the mic (unbounded length — chunked into 15s windows under
  the hood) or upload an audio file
- Or write an entry directly as text, no transcription step
- Primary + optional secondary language, for code-switching recordings
  (each ~15s window is tried against both explicitly and the better match
  is kept — see the in-app footnote for the full reasoning and known
  limits of this approach)
- Per-entry Where / When / Life Era / Private-vs-shareable fields, set at
  save time
- A "Download recording (.wav)" button appears right after a mic
  recording — that's the only window to keep the audio, since **saved
  entries never store audio**

**Library**
- Every saved entry: title, transcript, metadata, tags, life era, notes
- Search across titles, transcripts, notes, tags, life era, and location
- Open any entry to edit the title/transcript (the original
  as-transcribed text is always kept alongside your edits, with a
  one-click "Restore original transcript")
- Add/remove notes on an entry, timestamped
- Rule-based tag suggestions (keyword matching across categories like
  work, family, health, relationships, travel, gratitude, loss & grief,
  achievement, conflict, milestone) — click a suggested tag to apply it,
  or add your own custom tags
- Two-press delete (no undo, matching the no-server storage model)
- Each AI reflection can grow into its own thread: reply by typing or by
  recording a short voice note (transcribed with the same on-device
  Whisper pipeline as the main capture flow), then optionally ask for
  another AI response that sees the whole conversation so far, not just
  the latest message
- Beyond reflection, each entry also has: **Think it through** (a
  one-move-at-a-time conversation, saved per entry), **Threads across
  entries** (semantic recall of related past entries via on-device
  MiniLM embeddings), **Guided questions**, and an **Auto title /
  category / tags** control — all on-device, all optional, collapsed by
  default in the entry view
- "Download entry" saves a single entry as a plain-text file (timestamp,
  life era, where, privacy, tags, transcript, notes, reflections and
  their threads); "Export all entries" in the Library saves every saved
  entry at once as one JSON file — both are local downloads to this
  device, nothing is uploaded

## Known limitations

- **Not exercised in a live browser session by me** — verified by static
  analysis (HTML structure, JS syntax, unit tests on the tag-rule engine
  and date formatting), but please test a full record → save → edit →
  tag → delete pass yourself before wider sharing.
- Transcription auto-translation avoidance, chunk-boundary dedup, and
  silence-padding fixes are all as before — see in-app footnote for full
  detail on Whisper's known failure modes and how this pipeline works
  around them.
- The rule-based tag engine is intentionally simple keyword matching, not
  inference — it will under-tag and occasionally mis-tag. That's expected
  at this stage, not a bug to chase; richer auto-tagging is scoped as a
  later, MCP-connected upgrade in the Feature Set 3 brief.
- The Private/Shareable toggle is a schema field and a UI signal right
  now, not an enforced permission — nothing in this prototype sends data
  anywhere regardless of its value. It exists so a future MCP server can
  read it as a real gate once Layer 2 exists.
- Entries live in this browser's IndexedDB for this origin: they don't
  sync across devices or browsers, and clearing this site's data (or
  using a private/incognito window) deletes them.
- `crypto.randomUUID` (used for entry/tag/note IDs) needs HTTPS or
  localhost — both GitHub Pages and `python3 -m http.server` qualify.
  There's a fallback ID generator regardless.
- The `web-llm` import is **unpinned** (`esm.run/@mlc-ai/web-llm`), so it
  loads whatever version esm.run currently serves. That version's
  JSON/grammar mode (`response_format`) hangs, which is why the AI
  features steer JSON via the prompt and parse it defensively instead.
  Pinning to a known-good version is the durable fix and is recommended
  as a follow-up — see `COMMIT_NOTES_feature_set_4.md`.
- The AI features (reflection, Think it through, Threads, guided
  questions, AI labelling, AI myth scan) need a WebLLM model loaded
  (Chrome/Edge with WebGPU) or an Ollama connection. Every model-free
  feature — capture, transcription, editing, notes, rule-based tags and
  the rule-based myth scan — works fully without either.

## Recent changes (Feature Set 4 — richer on-device AI)

All of the following run on-device and route through the same AI engine
as reflection (in-browser WebLLM, or Ollama as a fallback):

- **Think it through** — a back-and-forth over an entry that offers one
  thing at a time (a question, or something reflected back) to help you
  clarify what you're reaching toward. The exchange is saved per entry.
- **Threads across entries** — finds past entries that echo the current
  one *by meaning* (on-device MiniLM embeddings + similarity, not just
  shared words) and surfaces one thread you keep returning to. It will
  say so honestly rather than invent a link when nothing connects.
- **AI title, category & tags** — one call fills a plain-language title,
  a Life Era category, and theme tags. Runs quietly in the background on
  save when a model is loaded, and is also a button in the entry view.
  It never replaces the rule-based tags — it merges with them — and the
  category is constrained to the existing Life Era set.
- **Guided questions** — a few open questions to take an entry deeper,
  in your own register, with a small built-in set as a fallback when no
  model is loaded.
- **Myth vs Truth, AI scan** — now returns, for each flagged sentence, a
  short reason and a gentler truth in your own frame (not just the
  category). The always-on rule-based scan is unchanged.

Under the hood: entries are embedded on save for the semantic recall
above (older entries backfill when first opened); the added AI panels
are collapsed by default so the entry view opens compact; and all AI
calls are serialised and capped, with streaming progress, so nothing
hangs on a silent spinner. See `COMMIT_NOTES_feature_set_4.md` for the
full technical detail, including a note on the WebLLM JSON-mode
workaround and the still-unpinned `web-llm` import.

## Recent changes (previous update)

- Added saved entries backed by IndexedDB (fully on-device), a Library
  view with search, per-entry transcript editing with restore-to-original,
  and timestamped notes.
- Added a one-time raw-audio WAV download offered immediately after each
  recording — audio is never stored with entries.
- Added text entry as a third input path alongside record/upload.
- Added Where / When / Life Era fields per entry, editable after saving.
- Added a per-entry Private/Shareable flag, private by default.
- Added rule-based tag suggestions plus custom tags, with search now
  covering tags, life era, and location too.
- Added threaded follow-ups on AI reflections: type or record a voice
  reply to any reflection, then optionally get another AI response that
  takes the whole thread into account.
- Added local export/download: a plain-text download per entry, and a
  full-library JSON export from the Library view.

## Files

```
index.html   Everything — UI, styling, and the full app logic in one file
```
