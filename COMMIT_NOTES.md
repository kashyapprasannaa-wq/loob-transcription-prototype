# GitHub update — force iOS/iPadOS to WASM

## Commit message

```
Force iOS/iPadOS to WASM (q4 WebGPU still OOM-crashed)

Quantized q4 WebGPU still overran Safari's WebGPU memory ceiling on
iOS/iPadOS and hard-crashed the tab on model load. Skip WebGPU
entirely on Apple mobile and run Whisper-small on CPU via WASM+q8,
which never touches the GPU budget. Slower but completes reliably.
```

## How to apply

Option A — replace the file:
Overwrite index.html in the repo root, commit, push.

Option B — apply the patch:
```
git apply ios_wasm_fix.patch
git add index.html
git commit -m "Force iOS/iPadOS to WASM (q4 WebGPU still OOM-crashed)"
git push
```

## What changed

One branch in pickBackend(): iOS/iPadOS now returns wasm+q8 instead of
webgpu+q4. Desktop/Android still use webgpu+fp32, unchanged.

## Post-deploy check

- Hard-reload / clear site data on the device first (Pages caches, and
  the crashed q4 version may be cached).
- Expect status: "Ready — running on WASM (CPU fallback)".
- Loading and transcription will be slower on mobile than desktop —
  that's expected on CPU. It should no longer crash.
