# GitHub update — iOS/iPadOS WebGPU load fix

## Commit message

```
Fix iOS/iPadOS model-load reload loop (WebGPU OOM)

iOS/iPadOS Safari exposes navigator.gpu, so the loader took the
webgpu + fp32 path. Mobile Safari's WebGPU memory ceiling can't hold
fp32 Whisper-small: weights download to 100%, then GPU buffer
allocation OOM-kills the WebKit process, which reloads the tab — the
"100% then restart" loop.

- Add isAppleMobile() detection (incl. iPadOS-reports-as-Mac via
  MacIntel + maxTouchPoints).
- pickBackend(): desktop/Android keep webgpu+fp32; iOS/iPadOS use
  webgpu with q4 encoder/decoder to fit the mobile budget; no-WebGPU
  falls back to wasm+q8.
- Runtime fallback: if a webgpu load throws, retry on wasm+q8 instead
  of leaving the user stuck.
```

## How to apply

Option A — replace the file (simplest):
Drop the new `index.html` into the repo root, overwriting the old one,
then commit with the message above.

Option B — apply the patch:
```
git apply ios_webgpu_fix.patch
git add index.html
git commit -m "Fix iOS/iPadOS model-load reload loop (WebGPU OOM)"
git push
```

## What changed

Only the model-loading block was touched. The old
`navigator.gpu ? "webgpu" : "wasm"` selection was replaced by
`pickBackend()` + `loadWithBackend()` + a runtime WASM fallback.
No other pipeline, storage, or UI logic changed. Net +59 lines.

## Post-deploy check

- Test on the actual iPad/iPhone — expect status to read
  "Ready — running on WEBGPU (mobile, quantized)".
- If q4 hurts transcription quality on your test recordings, bump the
  two dtype values in pickBackend() from "q4" to "q8".
- If an older/base iPad still loops on q4, force iOS/iPadOS straight
  to WASM (return the wasm config in pickBackend's middle branch).
