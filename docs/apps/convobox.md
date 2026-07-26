# ConvoBox

**Local, backend-agnostic voice frontend for CLI coding agents.**

[:fontawesome-brands-github: github.com/LegionForge/convobox](https://github.com/LegionForge/convobox){ .md-button .md-button--primary }

## What it is

ConvoBox sits between you and whichever coding-agent CLI you're driving — Claude Code, Codex, OpenCode, and eventually others — and lets you work by voice instead of, or alongside, the keyboard. It's a developer tool, not a general-purpose voice assistant: without an already-running coding-agent CLI, ConvoBox has nothing to talk to.

The pipeline is entirely local by default: continuous mic capture, on-device voice-activity detection, local speech-to-text (faster-whisper), a deterministic safeword check outside the model path, an orchestrator that routes each utterance as a new command, a soft interjection, or a hard stop, and local text-to-speech (Kokoro by default, Apache-2.0; Piper available as an explicit opt-in extra since it's GPL-3.0). A thin adapter layer (`send_text`, `send_interject`, `send_hard_stop`, `is_busy`) maps that routing onto whichever backend CLI is actually running, preferring each tool's native structured/headless interface over scraping terminal output.

## Status

All three backend adapters — OpenCode (HTTP+SSE), Claude Code (stream-json subprocess), and Codex (app-server JSON-RPC) — have been driven through the full live voice loop, including real tool use, on Windows 11. Linux/macOS are implemented but not yet voice-validated end to end.

Past the initial pipeline (mic → VAD → STT → safeword → orchestrator → backend → TTS), a substantial interaction/safety layer has since landed: barge-in presets, a live conversation TUI, response tiering, voice-gated tool approval, acoustic echo cancellation, and a local web UI. None of this is a stable end-user product yet — it's a working, extensively tested prototype under active live-UAT iteration, not a packaged release.

## Interaction & safety features

- **Barge-in, as a two-axis preset system.** `conversational` / `patient` / `do-not-disturb` / `halt` / `take-over` presets independently control what happens to the *current turn* versus the user's *new words* when they talk over a response — not a single on/off interrupt flag.
- **Deterministic hard stops.** A configurable safeword (e.g. "stop stop stop") is checked on the raw transcript before anything else touches it, so an abort can't be second-guessed or reinterpreted by an LLM.
- **"Stop listening" / resume word.** A pause phrase puts ConvoBox into a resume-word-only state; ordinary speech is ignored until the resume word is heard again, and the safeword still works while paused (it just doesn't resume).
- **Response tiering.** Optionally, voice speaks only the first paragraph of a multi-paragraph reply; saying "continue" within a timeout speaks the rest, already in hand, with no backend round-trip.
- **Voice-gated tool approval.** A configurable approval phrase gates pending destructive tool calls — honored on Codex (a native per-call approval channel) and Claude Code (a PreToolUse hook + local IPC channel, since headless mode has no native equivalent). Approve, deny, or ask "explain"/"clarify" for a plain-language description of what's pending before deciding.
- **Acoustic echo cancellation** (WebRTC AEC3, an opt-in extra) so the assistant's own voice played through speakers doesn't trip its own barge-in detection — headphones sidestep the need for this entirely.
- **A live conversation TUI** (`--tui`): transcript pane, full response detail, and a status/barge-in indicator, alongside a separate Settings TUI for config editing (audio device picker with live level meter, voice picker with in-place audition, backend/permission-mode selection).

## Local web UI

An optional, local-only browser view of a live session (`--web` / `web.enabled`, off by default): transcripts, backend responses, tool calls, and pending approvals streamed to the browser over Server-Sent Events, plus optional SQLite-persisted history — a separate opt-in from the live view itself, since viewing a session and writing it to disk are different privacy decisions. No authentication; bound to loopback by design. Ships as a single dependency-free HTML/JS page, no separate frontend build. Browser-side tool approval isn't built yet — voice and the TUI remain the only channels for that.

## Backend adapters

Each coding-agent CLI gets its own adapter behind the same small interface:

| Backend | Transport | Notes |
|---|---|---|
| OpenCode | HTTP + SSE | Runs its own local server; ConvoBox connects to it. |
| Claude Code | stream-json subprocess | Headless mode has no native runtime approval channel — ConvoBox builds one via a PreToolUse hook. |
| Codex | app-server JSON-RPC | Has a native per-call approval channel ConvoBox answers directly. |

`backend.permission_mode` (`plan` / `approve` / `permissive`) is the single source of truth for how much a spawned agent may do, translated per-backend at spawn — defaulting to read-only (`plan`) since a voice channel has no per-action confirmation by default and misheard words are a real risk.

## Local speech engines

- **STT:** [faster-whisper](https://github.com/SYSTRAN/faster-whisper) (CTranslate2-based Whisper), downloaded on first use, not bundled.
- **TTS:** [Kokoro](https://github.com/thewh1teagle/kokoro-onnx) (Apache-2.0, default) or [Piper](https://github.com/rhasspy/piper) (GPL-3.0, opt-in extra only — kept out of the default install so a plain `uv sync`/`pip install .` never pulls in GPL-licensed code).

## Known limits

- Linux and macOS have the same adapters/pipeline as Windows but haven't been voice-validated end to end there yet.
- Browser-side approve/deny for the web UI isn't built — it's an open design question (how a browser decision should interact with a simultaneous voice answer), not just an unwritten endpoint.
- No remote-access or authentication story for the web UI by design — it's meant for the same machine only.
