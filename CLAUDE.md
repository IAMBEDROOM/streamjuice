# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

StreamJuice is a unified streaming command centre built as a **Tauri desktop application** (Rust backend + React/TypeScript frontend). It consolidates alerts, overlays, chat, scene design, and stream management into a single app. The only external dependency is OBS Studio for encoding/broadcasting.

**Status:** Pre-development — project plans are in `docs/`, no source code yet. Currently in Phase 1 scaffolding.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Desktop backend | Rust (Tauri core, Actix-web/Axum local HTTP server) |
| Desktop frontend | React + TypeScript (strict mode) |
| Internal comms | Socket.IO (desktop ↔ browser source overlays, future mobile) |
| External comms | Raw WebSocket (Twitch EventSub, OBS WebSocket v5, Kick chat) |
| Database | SQLite (via rusqlite or sqlx, with migration system) |
| Alert/overlay rendering | HTML/CSS/JS served as browser sources to OBS |
| Platform APIs | Twitch EventSub/Helix, YouTube Data/Live API, Kick API |

## Planned Build Commands

```bash
# Frontend
npm run dev          # Development server
npm run build        # Production build
npm run lint         # ESLint
npm run format       # Prettier

# Rust backend
cargo clippy         # Lint
cargo fmt            # Format
cargo test           # Unit tests

# Tauri
npm run tauri dev    # Full dev mode (frontend + Rust backend)
npm run tauri build  # Production build
```

CI: GitHub Actions runs lint + build on push.

## Architecture

### Core Pattern
Tauri app where the Rust backend handles: local HTTP server (serves browser sources to OBS), platform API communication, file system access, SQLite database, OAuth token management (OS keychain), rate limiting, and caching. The React/TS frontend provides all UI. Socket.IO bridges desktop ↔ overlays (and future mobile).

### Browser Source Pipeline
StreamJuice serves overlay/alert HTML pages via a local HTTP server on localhost. OBS consumes these as browser source URLs:
- `localhost:PORT/alerts` — alert rendering
- `localhost:PORT/overlay/{id}` — persistent overlays
- `localhost:PORT/scene/{id}` — full-screen scenes (starting soon, BRB, ending)
- `localhost:PORT/chat-overlay` — chat display on stream

### Communication Flow
- **Internal:** Socket.IO with namespaces (`/overlays`, `/control`)
- **External:** Raw WebSocket clients to Twitch EventSub, OBS WebSocket v5, Kick chat
- Platform events are normalised into shared types before entering the alert/chat pipelines

### Data Flow for Alerts
Platform event → normalised format → alert queue (FIFO/priority) → resolve design template → replace dynamic variables ({username}, {amount}, etc.) → Socket.IO → browser source renders with entrance → hold → exit animation

### Planned Project Structure
```
src/              # React/TypeScript frontend
src/types/        # Shared TypeScript types/interfaces
src-tauri/        # Rust backend
```

## Design Language

Dark-first colour palette (use these exact values for UI work):

| Name | Hex | Usage |
|------|-----|-------|
| Deep Space Black | #0A0D14 | Primary background |
| Charcoal Navy | #151A26 | Surface/card backgrounds |
| Crisp Off-White | #E6EDF3 | Primary text |
| Electric Cyan | #00E5FF | Primary accent (buttons, active links) |
| Hyper Magenta | #FF007F | Secondary accent (notifications, warnings) — use sparingly |
| Voltage Yellow | #FFD600 | Tertiary highlights |

## Build Order Reference

Development follows 5 phases with 110 tasks detailed in `docs/streamjuice-build-order.md`:

1. **Phase 1: Foundation** (18 tasks) — Tauri scaffold, SQLite, local HTTP server, Socket.IO, OAuth2 for all platforms, rate limiter, cache layer
2. **Phase 2: Alert System & Visual Editor** (37 tasks) — Asset management, visual drag-and-drop editor (Canvas/Konva.js/Fabric.js), animation engine, alert queue, browser source pages
3. **Phase 3: Unified Chat & Bot** (22 tasks) — Multi-platform chat (Twitch IRC, YouTube polling, Kick WS), moderation, built-in bot engine, chat overlay
4. **Phase 4: Stream Management** (16 tasks) — Multi-platform metadata sync, OBS WebSocket control, scheduling, analytics
5. **Phase 5: Themes, Polish & Release** (17 tasks) — Theme export/import (.sftheme), stinger transitions, hotkeys, onboarding, auto-updater

Tasks have explicit dependencies (e.g., task 2.27 depends on 1.12–1.14). Always check dependencies before starting a task.

## Key Conventions

- **Platform-agnostic:** Twitch, YouTube, and Kick are first-class citizens. All platform interfaces must be normalised behind shared types.
- **OAuth tokens:** Stored in OS keychain (via Tauri plugin), fallback to encrypted SQLite. Never sent anywhere except official platform OAuth endpoints.
- **Offline resilience:** If a platform disconnects, the rest continue. Queued alerts replay on reconnect (with configurable stale threshold). OBS browser sources keep working even if OBS WebSocket drops.
- **No telemetry** by default. Any future analytics require explicit opt-in.
- **License:** GPL-3.0.
