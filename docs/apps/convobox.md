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

## Status

Scaffolding stage. Initial components exist for audio capture/playback, VAD, local STT, safeword detection, TTS, orchestration, and an OpenCode adapter. Codex and Claude Code adapters are not stable yet.
