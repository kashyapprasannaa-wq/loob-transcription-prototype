# LOOB

A browser-based, **local-first** ambient journaling portal. Everything runs on the
user's own device — voice is transcribed in the browser tab via Whisper (transformers.js),
and every entry is stored in this browser's IndexedDB. **No server, no upload.** The
privacy claim is structural and verifiable: open DevTools → Network while recording and
you'll see no audio leave the device.

Live: https://prithya-r.github.io/LOOB_transcription-API_prototype/

## What's in this build

The prototype is a single self-contained `index.html`. It now has three layers:

**1. Portal (entry gate).** Landing visitors give their exact date, time, and location
of birth before entering — the framework's "total authenticity from the start" rule.
Two placeholder art variants are toggleable (raw portal symbol vs. gatekeeper + Xolo);
real artwork drops into the `#portalArtSymbol` / `#portalArtGatekeeper` blocks. Birth
data is stored on-device only (localStorage) and skips the gate on return visits.

**2. Today (weekly/daily matrix).** A Sunday-start week strip; the seven fixed power
moves (Acknowledgment, Belief, Choice, Discipline, Expansion, Feel, Going for it) in
order Sun→Sat; lunar phase, sun sign, and numerology shown as a coordinate strip; and a
planned-vs-actual split schedule that exposes the friction between intention and action.

- **Numerology** = the Sunday-start **week number of the year, reduced to a single
  digit**. Week 1 is the week containing 1 January; the count resets at the year
  boundary. Example: 26 July 2026 → week 31 → 3 + 1 = **4** (foundation, discipline,
  constancy). Because it's week-based, the number changes weekly, not daily.
- **Lunar phase / sun sign** are lightweight on-device approximations — good as an
  interface cue, not ephemeris-grade.

**3. Diary (story bank).** The existing capture + library engine: record a voice note,
transcribe it on-device, then save it as an entry you can edit, tag, annotate with
notes, categorise by Life Era, and browse. Includes rule-based tag suggestions,
Myth vs Truth limiting-belief detection, and optional AI reflection (in-browser WebLLM,
or a locally-run Ollama model as fallback).

## Transcription notes

- Whisper small runs client-side; model weights (~250 MB) are fetched once from the
  Hugging Face CDN, then cached by the browser for offline reuse.
- Language handling is explicit: name the primary language, and optionally a second for
  code-switching. Recordings over 15s are chunked (15s windows, 3s overlap) so the
  30-second context limit never truncates a long recording.
- Raw audio is never stored with an entry — the one-time WAV download offered right
  after recording is the only window to keep the audio itself.

## Running locally

It's a static file. Serve it over HTTP (not `file://`, which blocks some browser APIs):

```bash
python3 -m http.server 8080
# then open http://localhost:8080
```

## Deploying

GitHub Pages serves `index.html` from the repo root. `.nojekyll` is included so Pages
doesn't run the file through Jekyll. After pushing, hard-refresh (Ctrl/Cmd+Shift+R) to
bypass the cached previous build.
