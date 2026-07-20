# LOOB — Local-First Whisper Transcription Prototype

A single-file, browser-based transcription prototype built for AEGGNOLOGIE's LOOB project. Runs
Whisper small entirely client-side via [transformers.js](https://github.com/huggingface/transformers.js) —
no server, no upload. Audio never leaves the device it's recorded or opened on.

## Running it

No build step, no install. Just open `index.html` in a recent version of Chrome or Edge.

- **WebGPU** speeds things up significantly if your browser/device supports it; it falls back to
  WASM otherwise.
- First model load downloads ~250MB from Hugging Face's CDN and caches it in the browser — that's
  the only point this ever touches the network. Every recording after that runs fully offline.

**To try it without cloning:** open this repo's GitHub Pages URL (see below) directly, or download
`index.html` and double-click it.

## What it does

- Transcribes microphone recordings or uploaded audio files of any length (chunked internally —
  Whisper's own context window caps out at 30 seconds per call).
- Supports two modes:
  - **Single language** — pick one language, one pass per ~15s window.
  - **Hybrid / code-switching** — set a second "Also expect" language, and each window is
    transcribed once per candidate language, keeping whichever result actually matches its claimed
    language (by script or word patterns). Roughly 2x the compute, but avoids Whisper's tendency to
    silently mistranslate rather than transcribe when no language is pinned down explicitly.

## Known limitations (worth reading before testing)

- **No true blind auto-detection across all ~99 Whisper languages.** This was attempted several
  ways and confirmed unreliable in-browser with this model/library combination — every unconstrained
  pass Whisper offers can silently translate instead of transcribing, with no dependable way to
  detect that after the fact. "Also expect" requires naming the languages in play up front.
- **Won't catch a language switch faster than ~15 seconds**, including mid-sentence code-switching,
  or a third language beyond the two candidates picked — a hard limit of how Whisper decodes (it
  picks one language token per generation call), not something a setting fixes.
- **Genuine hallucination on quiet or ambiguous audio.** Whisper can occasionally get stuck looping
  a phrase on silence or background noise — a real model failure mode, not a chunking artifact. A
  repetition penalty and a generation-length cap limit how much damage one bad chunk can do, but
  this is a mitigation, not a guarantee.
- **No speaker diarization.** Out of scope for this prototype — a separate stage in the fuller
  technical blueprint.
- Segment-level (not word-level) timestamp trimming at chunk boundaries, since the ONNX build of
  Whisper small used here isn't exported with the cross-attentions word-level alignment needs.

## Recent fixes worth knowing about

If you tested an earlier build of this, a few things changed:

- **Boundary repetition** (the same phrase appearing twice around a chunk transition) is now
  handled by two layers: segment-timestamp trimming first, then a content-based check that compares
  the tail of the transcript so far against the head of each new chunk and drops any real
  word-for-word overlap it finds. Each layer alone missed cases the other catches, since the two
  chunks sharing an overlap region segment that audio independently and don't always agree on
  exactly where a boundary phrase belongs.
- **Silent auto-translation** (French/Tamil/etc. audio coming back in English without being asked)
  is fixed by no longer trying to auto-detect language at all — every generation call now explicitly
  names a language, which was the one combination confirmed reliable after several other approaches
  weren't.
- **Dropped endings** on longer recordings are fixed by padding the true end with silence, capped so
  it can never spill into creating an extra chunk (an earlier version of this padding could do that,
  which ironically caused the same dropped-ending symptom for a different reason).

## Feedback

The in-page **debug log** (below the transcript, open by default) shows, per chunk: how many
segments came back with usable timestamps, whether the dedup layer skipped any duplicate words at a
boundary, and (in hybrid mode) both candidate languages' scores. Including a copy of this log — it
has its own "Copy debug log" button — makes any repetition or language issue much faster to track
down than the transcript text alone.


