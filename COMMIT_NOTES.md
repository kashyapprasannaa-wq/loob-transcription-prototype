# GitHub update — honest iOS AI-reflection messaging

## Commit message

```
Make iOS AI-reflection messaging honest about Ollama

Ollama can't run on a phone — it's a desktop server the device would
have to reach over the network (LAN IP, OLLAMA_HOST=0.0.0.0, plus
HTTPS to avoid mixed-content blocking). Reworked the iOS messaging so
the on-device tiny model is presented as the real mobile path and
Ollama is framed as a desktop/advanced option, not a one-tap fallback.
No behaviour change — messaging only.
```

## What changed (all copy, no logic)

- iOS "both tiny models failed" message: no longer implies Ollama is a
  simple next step. States AI reflection isn't available on the phone
  itself, that model-free features still work fully on-device, and that
  Ollama means desktop or a separate networked machine.
- Supported-iOS explainer: same correction — on-device model is the
  path; Ollama/desktop only if it doesn't fit.
- Static setup prose: added a "On phones and tablets" note spelling out
  why Ollama isn't a phone option (LAN IP not localhost, 0.0.0.0, HTTPS
  mixed-content) and to prefer the on-device model on mobile.
- Badge text: "AI reflection needs desktop or Ollama" instead of
  "using Ollama".

## Post-deploy check

- Hard-reload / clear site data first.
- Desktop and the on-device iOS cascade behave exactly as before; only
  the wording differs when a device can't fit a tiny model.
