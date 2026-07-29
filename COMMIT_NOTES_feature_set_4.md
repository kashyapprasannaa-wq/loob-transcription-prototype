# GitHub update — Feature Set 4: richer on-device AI + WebLLM JSON fix + tidier detail view

## Commit message

```
Add conversational AI, cross-entry recall, AI labelling & guided questions

Feature Set 4 — all on-device, all routed through the existing
aiChatComplete() so the WebLLM-first / Ollama-fallback split is
inherited unchanged:

- Semantic recall: each entry is embedded on save with MiniLM
  (Xenova/all-MiniLM-L6-v2) via the transformers.js already imported —
  no new library. Vectors stored on the entry in IndexedDB (additive
  field, no DB_VERSION bump); older entries backfill lazily on open.
- "Think it through": a one-move-at-a-time conversational thread over
  the current entry, persisted per entry.
- "Threads across entries": retrieves the most semantically similar
  past entries and surfaces one real connection, with a
  connection_found:false escape hatch so a small model isn't forced to
  confabulate a link.
- AI title / category / tags: one structured call, category clamped to
  the Life Era vocabulary, tags merged with (never replacing) the
  rule-based suggestTags() baseline. Runs in the background on save
  when a model is loaded; also available as a button in the detail view.
- AI guided questions: register-seeded off the reflection voice, with a
  curated static fallback when no model is loaded.
- Myth vs Truth AI scan upgraded to return structured
  {sentence, category, why, truth}; verbatim guard unchanged; an
  AI-provided Truth now takes precedence over the templated one (the
  person's own answer still wins). Rule-based path untouched.

WebLLM JSON fix: this build's WebLLM hangs inside create() when
response_format (grammar/JSON mode) is requested — create() never
resolves and never errors (plain reflection streams fine on the same
model; only response_format calls stall). JSON features no longer use
response_format: they make the same plain streaming call the reflection
path uses and steer JSON via the prompt, then recover it with a
defensive extractJson() parser (handles code fences, preamble, trailing
prose, braces-in-strings, escaped quotes; 11/11 parser unit tests pass).

Serialization: every model call is now chained through a single queue,
because MLCEngine processes one request at a time and background
auto-title was colliding with user-initiated scans and hanging. A
45s first-token watchdog turns any genuine stall into a named error
instead of a dead spinner.

Latency: output token caps per feature (title 96, guided 160, myth
512), trimmed system prompts, and titling only sees the first ~150
words. All JSON calls stream, so progress is visible.

Detail view: the three added AI panels (Guided questions, Threads
across entries, Think it through) are collapsed by default via the
existing .more-detail disclosure pattern, so the entry view opens as a
compact stack instead of a long scroll. Built-in AI reflection and
Myth vs Truth panels unchanged. Auto title/tags is an inline control
by the title, not a panel.
```

## What changed

### New code (appended module block, ~640 lines)
- Semantic-recall layer: `getEmbedder()`, `embedText()`, `cosineSim()`,
  `ensureEmbedding()` (embed-on-save + lazy backfill), `relatedEntries()`.
- Feature functions: `aiAutofillEntry()`, `aiGuidedQuestions()`,
  `aiRepurpose()`, `buildConversationMessages()`, and the prompt
  constants (`AUTOFILL_/GUIDED_Q_/CONVERSATION_/REPURPOSE_SYSTEM_PROMPT`).
- UI wiring: collapsible Guided / Threads / Think-it-through panels
  (DOM-node construction, no innerHTML), the inline auto-title control,
  and an `openDetail()` wrapper that builds/refreshes them on open.
- `enrichEntryInBackground()` — non-blocking embed + optional AI autofill
  after save; never blocks the instant save, no-ops without a model.

### Edited in place
- `aiChatComplete()` split into a serializing wrapper (`_aiQueue`) plus
  `_aiChatCompleteInner()`; WebLLM JSON branch rewritten to prompt-based
  JSON + `extractJson()`; `maxTokens` plumbed to both engines
  (WebLLM `max_tokens`, Ollama `num_predict`); Ollama JSON path now
  streams too.
- Save handler: fires `enrichEntryInBackground(entry.id)` after `dbPut`.
- Myth scan: prompt swapped to the structured V2; handler stores
  `why`/`aiTruth`; the Truth block prefers an AI truth when present.
- CSS: `.more-detail summary.panel-title` sizing to match real panel
  titles; compact padding for collapsed `details.panel.more-detail`.

## Known issue carried forward (not fixed here)
- **The `web-llm` import is still unpinned** (`esm.run/@mlc-ai/web-llm`,
  no version). esm.run serves the newest published version, which can
  change behaviour between loads — this is the most likely origin of the
  create()/response_format hang above. The prompt-based JSON approach no
  longer depends on that fragile feature, so these features should
  survive version drift better, but pinning to a known-good version is
  the durable fix and still needs the version the original build was
  validated against. Recommend pinning in a follow-up.

## Post-deploy check
- Hard-reload / clear site data first (cached module + model).
- Confirm on Chrome/Edge with a WebLLM model loaded: open an entry, use
  Auto title/tags, Suggest questions, Find related threads, Think it
  through, and the AI myth scan — each should stream and complete, not
  hang. Console shows `[loob-ai]` stage logs; a stall would surface as a
  named 45s watchdog error rather than a frozen button.
- Plain AI reflection and the rule-based myth scan behave exactly as
  before.
- Not exercised in a live browser by me — verified by static analysis,
  full-module ES syntax check, and unit tests (retrieval/cosine,
  autofill clamp, extractJson battery, call serialization). Please run a
  real save → open → AI-features pass before wider sharing.
