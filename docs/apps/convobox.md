# ConvoBox

**Local, backend-agnostic voice frontend for CLI coding agents.**

[:fontawesome-brands-github: github.com/LegionForge/convobox](https://github.com/LegionForge/convobox){ .md-button .md-button--primary }

## What it is

ConvoBox sits between you and a coding-agent CLI so you can work by voice instead of, or alongside, the keyboard. It is designed to point at multiple backends, including Codex, Claude Code, OpenCode, and future tools, through a thin adapter layer.

The project is local-first. Speech-to-text and text-to-speech are intended to run on hardware you control, with the option to split the audio client from heavier processing over a private network later.

## Core direction

- **Full-duplex voice.** Continuous listening, voice-activity detection, barge-in, and spoken responses instead of push-to-talk only.
- **Backend adapters.** Each coding agent implements a small surface: send text, interject, hard stop, and report busy state.
- **Deterministic hard stops.** Safeword detection is handled outside the model path so an abort cannot be ignored or reinterpreted by an LLM.
- **Voice-aware risk policy.** Destructive actions can require stricter confirmation when triggered by speech.

## Pipeline

```text
mic -> VAD -> local STT -> safeword check -> orchestrator -> backend adapter
                                                           -> local TTS
```

The orchestrator routes each utterance as a new command, a soft interject into an active task, or a hard stop. Backend adapters prefer native structured interfaces where possible and use terminal control only as a fallback.

## Current implementation

- **Pipeline pieces exist.** Audio capture/playback, VAD segmentation, local STT, safeword detection, TTS, orchestration, and an OpenCode adapter are implemented.
- **Hard stops cut audio too.** A hard stop now aborts backend work and stops in-progress TTS/playback.
- **Real audio validation has started.** TTS -> STT round trips work without a microphone, and real speaker playback including barge-in has been tested on macOS.
- **Structured adapters are preferred.** The adapter boundary favors native HTTP/SSE or headless APIs over scraping terminal output.
- **Licensing is being cleaned up.** ConvoBox is intended to stay MIT and free for everyone; the current Piper TTS dependency has a GPL concern, so Kokoro is the identified replacement path.

## Known gaps

- Live microphone capture is still unverified on the current development machine because it has no input device.
- Windows and Linux are not yet verified, though the dependency choices are intended to be cross-platform.
- The OpenCode adapter's initially assumed endpoint paths did not match a real `opencode serve` instance; the API shape is documented, but the adapter still needs that correction.
- Codex and Claude Code adapters are not stable yet.

## Status

Scaffolding stage, with real pipeline validation underway. The project is not a stable end-user app yet, but the core audio/orchestration pieces are implemented and covered by automated tests.
