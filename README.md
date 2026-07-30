# LOOB — Local-first journaling portal

A voice-first journaling app that runs entirely in the browser. No server, no
accounts, no upload — everything (audio transcription, storage, and AI
reflection) happens on your own device.

Open `index.html` in a modern browser (Chrome/Edge recommended for WebGPU) to
run it. No build step, no dependencies to install.

## Features

- **Voice capture** — records audio and transcribes it on-device with Whisper
  (via [transformers.js](https://huggingface.co/docs/transformers.js)),
  including long recordings and multi-language / code-switching support.
- **Calendar & numerology matrix** — a daily view combining a numerology
  "power move," lunar phase, sun sign, a week strip, a full timed weekly
  schedule, and a month calendar with events.
- **Planned vs. actual** — log what you intended for the day next to what
  actually happened, to see the gap in real time.
- **Story bank (library)** — a persistent list of saved entries; click one to
  open its full detail (transcript, tags, life era, privacy) alongside the
  list, without leaving it.
- **AI reflection** — generates a short reflective response to an entry,
  running fully in-browser via WebLLM, or via a local Ollama server as a
  fallback. Never a third-party API.
- **Myth vs. Truth** — a rule-based (no AI, no network) scan for
  limiting-belief language, with an optional on-device AI scan for other
  languages.
- **Threads across entries** — on-device semantic recall that surfaces past
  entries related to the one you're viewing.
- **Notes**, **tags**, **export** (all entries as one JSON backup), and a
  **light/dark theme toggle**.

## Storage & privacy

Entries, notes, reflections, and calendar data are stored in this browser's
IndexedDB/localStorage for this origin only — nothing is uploaded, and
clearing site data removes it. Birth date/time/place (used to seed the
numerology matrix) is stored the same way and is never sent anywhere.

## Status

Single-file prototype (`index.html`) under active iteration.
