# GitHub update — fix iOS crash on longer recordings (memory)

## Commit message

```
Stream chunks lazily to fix iOS crash on longer recordings

Crash-and-reload at ~1-2min in single-language mode was memory, not
speed: chunkAudio() pre-sliced the whole recording into an overlapping
chunk array held for the entire run, stacked on the base recording and
the ONNX runtime's accumulating per-call tensors — enough to pass iOS
Safari's memory ceiling and get the tab killed mid-run.

Now compute chunk boundaries (index pairs) instead of materialising all
chunks, slice one chunk at a time inside the loop, null it and the
result after each merge, and yield to the event loop between chunks so
Safari can GC and repaint. Only one 15s chunk is ever alive on top of
the recording. Chunking is byte-identical to before (verified across
5s-10min), so transcripts are unchanged.
```

## What changed

- Added chunkBoundaries() — returns [start,end) pairs, no audio copies.
- transcribeFloat32(): uses boundaries for padding math and the loop;
  slices chunkData lazily per iteration; frees chunkData + result and
  awaits a 0ms timeout each iteration.
- Old chunkAudio() kept (still used for the <=15s single-chunk case).
- No output change: lazy boundaries proven equivalent to the old
  array slicing for every tested length.

## Post-deploy check

- Hard-reload / clear site data first.
- iOS: record ~2-3 min in single language -> should now complete
  without the tab reloading. Progress ("Processing chunk N of M")
  should tick steadily.
- Transcript quality should be identical to short recordings.

## If it STILL crashes on very long recordings (10min+)

The remaining large object is the base recording held twice (raw PCM
in lastRecording for the WAV download, plus the 16k Float32). Next
lever would be releasing lastRecording's raw PCM once the WAV is
downloaded/declined, or streaming transcription during recording
rather than record-then-process. Say the word and I'll wire that in.
