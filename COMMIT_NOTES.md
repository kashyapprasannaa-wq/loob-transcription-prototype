# GitHub update — mobile uses whisper-base for speed

## Commit message

```
Use whisper-base on mobile/WASM for usable CPU speed

whisper-small on mobile CPU (WASM) transcribes far too slowly. Mobile
and any no-WebGPU path now load whisper-base — markedly smaller and
faster on CPU, still good quality. Desktop WebGPU keeps whisper-small.
Model choice now lives in pickBackend() alongside device/dtype.
```

## How to apply

Overwrite index.html, commit, push. (Or: git apply mobile_base_model.patch)

## What changed

- pickBackend() now also returns a `model`: MODEL_SMALL for desktop
  WebGPU, MODEL_BASE for mobile/WASM and the desktop WASM fallback.
- Status text shows which model loaded ("Ready — Whisper base on WASM").

## Post-deploy check

- Hard-reload / clear site data on the device first (Pages caches).
- Status should read "Ready — Whisper base on WASM." on iOS.
- Transcription should be noticeably faster than before.
- Note: base downloads fresh (~1/3 the size of small) — first load
  re-fetches weights, so allow one download before it's cached.

## If base is still too slow

Swap MODEL_BASE to "onnx-community/whisper-tiny" — smallest/fastest,
lower accuracy. One-line change at the top of pickBackend().
