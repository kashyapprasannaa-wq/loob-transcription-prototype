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
- **Won't catch a language switch faster than ~15 seconds**, including mid-sentence code-switching —
  a hard limit of how Whisper decodes (it picks one language token per generation call), not
  something a setting fixes.
- **Repetition/hallucination on quiet or ambiguous audio.** Whisper can occasionally get stuck
  looping a phrase on silence or background noise. A repetition penalty and a generation-length cap
  are applied to limit the damage, but this is a mitigation, not a guarantee.
- **No speaker diarization.** Out of scope for this prototype — a separate stage in the fuller
  technical blueprint.
- Segment-level (not word-level) timestamp trimming at chunk boundaries, since the ONNX build of
  Whisper small used here isn't exported with the cross-attentions word-level alignment needs.

## Feedback

The in-page **debug log** (below the transcript, open by default) shows per-chunk language decisions
and scoring — useful to include if flagging something that looks wrong.
