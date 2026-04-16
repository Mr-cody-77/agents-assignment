# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LiveKit Agents is a Python framework for building realtime, programmable voice/video AI agents. Agents can see, hear, and understand using WebRTC. The framework integrates STT (speech-to-text), LLM, and TTS (text-to-speech) providers.

## Development Setup

```bash
# Install dependencies (uses uv package manager)
uv sync --all-extras --dev

# Run tests
uv run pytest tests/

# Run specific test file
uv run pytest tests/test_agent_session.py

# Run tests with Docker (for integration tests with toxiproxy)
cd tests && make test
```

## Running Agents

```bash
# Terminal mode (local audio I/O for testing)
python <agent.py> console

# Development mode (hot reload, connects to LiveKit server)
python <agent.py> dev

# Production mode (optimized)
python <agent.py> start
```

Environment variables required in `.env`:
- `LIVEKIT_URL`, `LIVEKIT_API_KEY`, `LIVEKIT_API_SECRET`
- Provider keys as needed (e.g., `OPENAI_API_KEY`, `DEEPGRAM_API_KEY`)

## Architecture

**Core Components:**
- `Agent` - LLM-based application with instructions and tools
- `AgentSession` - Container managing user interactions (STT, LLM, TTS, VAD)
- `AgentServer` - Coordinates job scheduling and agent lifecycle
- `JobContext` - Session context with room connection

**Module Structure:**
- `livekit-agents/` - Core framework (voice, llm, stt, tts, vad, ipc, worker)
- `livekit-plugins/` - Provider integrations (OpenAI, Deepgram, Cartesia, ElevenLabs, Google, etc.)
- `examples/` - Reference implementations

**Voice Pipeline:**
```
User Speech → STT → LLM → TTS → Agent Response
              ↑              ↓
           VAD/Turn Detection
```

**Key Patterns:**
- Use `@function_tool` decorator for LLM-callable functions
- Agents use `on_enter()` for initialization and `generate_reply()` for responses
- Multi-agent handoffs via tool returns: `return next_agent, "transition message"`
- Metrics collected via `MetricsCollectedEvent`

## Linting & Type Checking

```bash
uv run ruff check .
uv run mypy .
```

## Testing Notes

- Tests use `pytest-asyncio` with async fixtures
- Integration tests require Docker (toxiproxy for network testing)
- Mock implementations available in `tests/fake_*.py`
- Evals test LLM workflows with judge models
