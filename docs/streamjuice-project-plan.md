

**StreamJuice**

Unified Streaming Suite

Project Plan & Development Roadmap

Version 1.2  |  March 2026

**CONFIDENTIAL**

# **1\. Executive Summary**

StreamForge is a unified streaming command centre built as a Tauri desktop application. It replaces the fragmented ecosystem of streaming tools (StreamElements, Streamlabs, separate chat clients, alert editors, social media managers) with a single, cohesive platform.

The only external dependency is OBS Studio (base version) for encoding and broadcasting. Everything else — alerts, overlays, chat, scene design, stream management — lives inside StreamForge.

A companion mobile app is planned for a future release once the desktop application is fully functional. The mobile app will add remote control, social media broadcasting, and stream deck functionality. Its architecture is accounted for in this plan to ensure the desktop app is built ready for mobile integration.

**Target Platform (v1):** Desktop (Windows, macOS, Linux via Tauri)

**Future Platform:** Mobile companion app (iOS, Android) — post-v1 release

**Tech Stack:** Tauri (Rust backend), React/TypeScript frontend

**External Dependency:** OBS Studio (base) for encoding/broadcast only

**Out of Scope (v1):** Companion mobile app, self-hosted donations/tipping

# **2\. Product Vision**

Streaming today requires a patchwork of disconnected tools. A typical streamer uses OBS for broadcasting, Streamlabs or StreamElements for alerts, a separate chat aggregator, social media apps for go-live announcements, a stream deck app for hotkeys, and a bot service for chat moderation. Each tool has its own account, its own design system, and its own quirks.

StreamForge consolidates all of this into one app with one design language. Streamers design their entire broadcast identity — overlays, alerts, panels, transitions — in a visual editor. They manage chat from all platforms in one window. And they control every aspect of their stream from a single interface.

## **Core Principles**

* One app to rule them all — eliminate tab-switching and account juggling

* Design-first — every visual element of a stream is customisable without code

* Platform-agnostic — Twitch, YouTube, and Kick treated as first-class citizens

* OBS-native — outputs standard browser sources and overlays, no plugins required

* Mobile-ready architecture — built to support a companion mobile app in a future release

# **3\. Design Language & Colour Scheme**

StreamForge uses a dark-first design language built around a carefully chosen palette. The scheme is designed for long streaming sessions, high contrast readability, and a premium feel that stands apart from the typical streaming tool aesthetic.

| Name | Hex | Swatch | Usage |
| :---- | :---- | ----- | :---- |
| **Deep Space Black** | \#0A0D14 |   | Primary background. Dominant colour with subtle blue tint, richer than pure black. |
| **Charcoal Navy** | \#151A26 |   | Surface/card background. Sidebars, dropdowns, floating cards for subtle depth. |
| **Crisp Off-White** | \#E6EDF3 |   | Primary text. High contrast readability without the harshness of pure white. |
| **Electric Cyan** | \#00E5FF |   | Primary accent. Buttons, active links, important icons, progress bars. |
| **Hyper Magenta** | \#FF007F |   | Secondary accent. Notifications, warnings, errors, badges. Use sparingly. |
| **Voltage Yellow** | \#FFD600 |   | Highlight/tertiary. Text highlights, star ratings, decorative borders. |

The contrast between Electric Cyan and Hyper Magenta creates a dynamic visual tension that gives StreamForge its distinctive identity. Voltage Yellow is reserved for moments that need to draw the eye without competing with the primary action colour.

# **4\. Architecture Overview**

The system is built around the desktop application as the core hub, with architecture designed to support a mobile companion app in a future release.

## **Desktop App (Tauri)**

The core hub. Rust backend handles local server operations, file system access, overlay serving, and platform API communication. The React/TypeScript frontend provides the UI for all design, management, and monitoring features. The app runs a local HTTP server that serves overlay/alert HTML pages as browser sources for OBS.

## **OBS Integration**

StreamForge does not replace OBS — it feeds into it. All visual outputs (overlays, alerts, scenes) are served as browser source URLs that OBS renders. This keeps the broadcast pipeline reliable and familiar while StreamForge handles everything around it.

## **Communication Layer**

Internal communication between StreamForge components (desktop to browser source overlays, and later desktop to mobile) uses Socket.IO for its built-in reconnection handling, event namespacing, and room support. External platform connections use raw WebSocket clients where required: Twitch EventSub, OBS WebSocket protocol (v5), and Kick chat all communicate over native WebSocket. The Socket.IO server is included from v1 to serve overlays and to ensure the architecture is ready for mobile integration without refactoring.

## **Mobile-Ready Architecture**

Although the companion mobile app is not part of the v1 release, the desktop app is built with mobile integration in mind. The Socket.IO server, pairing system, and API abstraction layer are all included in the v1 architecture so that the mobile app can connect seamlessly when it is developed. This avoids costly refactoring later.

# **5\. Security & Authentication**

Security covers two domains: authenticating with external platforms, and securing internal communication between StreamForge components.

## **Platform Authentication**

All streaming platform connections use OAuth2 flows. Tokens are stored encrypted in the local SQLite database using the OS keychain where available (via Tauri’s native API). Refresh tokens are rotated automatically before expiry. Each platform (Twitch, YouTube, Kick) has its own token scope, and users can revoke individual platform connections at any time.

## **Local Server Security**

The embedded HTTP server that serves browser sources to OBS runs on localhost only by default, limiting access to the local machine. For future mobile app connections over the local network, a pairing system will be used: the desktop app generates a one-time pairing code displayed on screen, which the mobile app enters to establish a trusted connection. The Socket.IO connection will require an authenticated session token on every handshake. This pairing infrastructure is architected in v1 but only fully activated when the mobile app ships.

## **Data Protection**

Sensitive data (OAuth tokens, API keys, pairing secrets) is encrypted at rest. The app never sends credentials to any server other than the official platform OAuth endpoints. No telemetry or usage data is collected without explicit opt-in consent. Stream designs, chat history, and user preferences remain entirely local unless the user explicitly exports or backs them up.

# **6\. Data Persistence & Sync Strategy**

All application data lives locally in a SQLite database on the desktop.

## **Local Storage**

The SQLite database stores all user settings, stream designs (overlays, alerts, themes), chat bot configuration, and cached platform data. Media assets (images, audio, fonts, animations) are stored in a managed folder on disk with references in the database. The database is backed up automatically on a rolling basis to prevent data loss.

## **Export & Backup**

Users can manually export their full configuration (settings, designs, bot config, asset references) as a backup file. This same export format will be used for the theme/template sharing system. Asset files can optionally be bundled into the export for full portability.

## **Cloud Backup & Mobile Sync (Future)**

Cloud sync and mobile data sync are not in scope for v1. The architecture uses a serialisation format designed to support remote storage later. When the mobile companion app ships, it will operate as a thin client pulling state from the desktop over Socket.IO rather than maintaining its own database.

# **7\. OBS WebSocket Integration Scope**

StreamForge connects to OBS via the OBS WebSocket protocol (v5), which ships built-in with OBS Studio 28+. The integration covers both read and write operations.

## **Read Operations (Monitoring)**

* Stream status: live/offline, uptime, bitrate, dropped frames, encoding load

* Current scene and source list

* Recording status and file path

* Audio levels and mute states

## **Write Operations (Control)**

* Switch active scene

* Toggle source visibility (e.g. show/hide webcam, overlays)

* Start/stop recording

* Trigger studio mode transitions

* Set source properties (position, scale, crop) for advanced layout control

This means StreamForge functions as a full scene controller from the desktop. When the mobile companion app ships, these same controls will be accessible remotely via the stream deck mode. The stream health monitor in the analytics dashboard displays real-time encoding and connection stats pulled from OBS.

# **8\. Platform API Strategy**

Each streaming platform has different API capabilities, rate limits, and reliability characteristics. StreamForge must handle all three gracefully, including when things go wrong.

## **Rate Limiting**

All outbound API calls go through a per-platform rate limiter in the Rust backend. Each platform’s limits are tracked independently: Twitch Helix allows 800 requests per minute, YouTube Data API uses a daily quota system, and Kick’s API has its own constraints. The rate limiter queues non-urgent requests and prioritises real-time operations (chat, alerts) over background tasks (analytics, metadata sync).

## **Caching**

Platform data that doesn’t change frequently (channel info, emote sets, badge definitions, game/category lists) is cached locally in SQLite with configurable TTLs. This reduces API calls and ensures the app remains responsive even when a platform API is slow. Cache invalidation is event-driven where possible (e.g. Twitch EventSub notifications trigger cache refreshes for relevant data).

## **Retry & Fallback**

Transient failures (5xx errors, timeouts, rate limit hits) use exponential backoff with jitter. If a platform connection drops entirely, the app continues operating with cached data and queues outbound actions for retry. The UI clearly indicates which platforms are connected and which are degraded, so the streamer always knows the state of their broadcast.

# **9\. Offline & Disconnected Behaviour**

Mid-stream connectivity issues are inevitable. StreamForge is designed to degrade gracefully rather than fail hard.

## **Platform Connection Loss**

If a single platform’s connection drops (e.g. Twitch EventSub disconnects), alerts and chat for that platform pause while the others continue unaffected. The app attempts automatic reconnection in the background. A visual indicator in the UI and on the overlay (optional) shows which platforms are active. Queued alerts from the disconnected platform fire once the connection is restored, with a configurable option to discard stale alerts older than a set threshold.

## **OBS Connection Loss**

If the OBS WebSocket connection drops, StreamForge continues serving browser sources normally — overlays and alerts keep rendering because they’re served via HTTP, not WebSocket. Scene switching and OBS control features are disabled until reconnection. The stream health monitor shows a disconnected state.

# **10\. File & Asset Management**

Streamers accumulate a significant amount of media. StreamForge manages all assets in a structured local folder with database references.

## **Storage Structure**

All imported assets are copied into a managed directory within the StreamForge data folder, organised by type: images, audio, fonts, and animations. Original filenames are preserved alongside a generated unique ID. The database stores metadata (dimensions, duration, file size, tags) for quick searching and filtering in the asset library UI.

## **Supported Formats & Limits**

* Images: PNG, JPG, GIF, WebP, SVG — max 20MB per file

* Audio: MP3, WAV, OGG — max 10MB per file

* Fonts: TTF, OTF, WOFF2

* Animations: Lottie (JSON), APNG, animated GIF, WebM

# **11\. Testing Strategy**

Testing a streaming tool is challenging because many features depend on live platform events and real-time rendering. The strategy covers automated testing, manual testing, and simulated environments.

## **Unit & Integration Tests**

Standard unit tests cover the Rust backend (API rate limiting, token management, caching logic, database operations) and the TypeScript frontend (state management, design serialisation, chat message parsing). Integration tests verify the Socket.IO communication layer and the browser source rendering pipeline.

## **Platform Event Simulator**

A built-in development tool that simulates platform events (follows, subs, raids, chat messages, bits) without requiring a live stream. This feeds mock events into the same pipeline that real events use, allowing full end-to-end testing of the alert system, chat display, and overlay rendering. The simulator is also exposed in the production app as the alert preview/sandbox feature.

## **Visual Regression Testing**

Overlay and alert rendering is tested using screenshot comparison. A headless browser loads the browser source URLs, triggers events via the simulator, captures frames, and compares against baseline screenshots. This catches unintended visual changes in alert animations, overlay layouts, and chat display.

## **Manual Testing Protocol**

Before each release, a manual test run covers: multi-platform authentication flow, live alert triggering on each platform, chat message display and moderation actions, OBS WebSocket connection and scene switching. A test stream to private/unlisted channels on each platform is used to verify real-world behaviour.

# **12\. Licensing & Distribution**

Distribution model and licensing decisions affect architecture choices around updates, telemetry, and feature gating.

## **Distribution Model**

The initial plan is to release StreamForge as a free application, with the option to introduce a premium tier later for advanced features (e.g. cloud backup, marketplace access, priority support). The core streaming functionality — alerts, overlays, chat — remains free. This maximises adoption and positions StreamForge as a genuine alternative to the existing free tools.

## **Update Mechanism**

Tauri includes a built-in auto-updater. Updates are signed and distributed via a static update server (S3 or similar). Users are notified of updates on app launch with the option to install immediately or defer. Critical security patches can be flagged as mandatory.

## **Telemetry**

No telemetry is collected by default. An optional, opt-in analytics layer can be added later for crash reporting and anonymised usage patterns to inform development priorities. This is clearly communicated to users and requires explicit consent.

## **Theme Marketplace (Future)**

The theme/template export system is designed with a future marketplace in mind. Themes are self-contained packages (JSON config \+ bundled assets) that can be shared freely or sold. DRM is not planned — the marketplace would rely on convenience and curation rather than technical restrictions.

# **13\. Development Phases**

The project is structured into five phases for the v1 desktop release, each building on the previous. The companion mobile app is a separate phase that follows after v1 is functional. Phases are designed so that each delivers a usable increment of functionality.

## **Phase 1: Foundation & Core Infrastructure**

**Estimated Duration:** 4–6 weeks

Establish the Tauri app skeleton, project structure, local server, and platform authentication. This phase produces no user-facing streaming features but creates the foundation everything else depends on.

| Feature | Description | Priority |
| :---- | :---- | ----- |
| **Tauri App Shell** | Rust backend \+ React/TS frontend scaffolding with build pipeline | **Critical** |
| **Local HTTP Server** | Embedded server to serve browser source URLs to OBS | **Critical** |
| **Platform Auth** | OAuth2 flows for Twitch, YouTube, and Kick APIs with encrypted token storage | **Critical** |
| **Database Layer** | Local SQLite for settings, designs, preferences, and cached data | **Critical** |
| **Socket.IO Server** | Internal communication server for desktop-to-overlay messaging (mobile-ready) | **Critical** |
| **Project Structure** | Monorepo setup with shared types and desktop app package | **High** |
| **Config System** | User preferences, keybindings, and app settings management | **High** |

## **Phase 2: Alert System & Visual Editor**

**Estimated Duration:** 6–8 weeks

The flagship feature. A visual editor for creating and customising alerts, overlays, and stream scenes. This phase also includes the alert rendering engine that listens for platform events and displays the correct alert via the browser source.

| Feature | Description | Priority |
| :---- | :---- | ----- |
| **Visual Alert Editor** | Drag-and-drop editor for designing follow, sub, raid, bits/stars alerts with animation, sound, and text customisation | **Critical** |
| **Alert Rendering Engine** | Listens to platform events (via WebSocket/EventSub) and renders alerts in the browser source overlay | **Critical** |
| **Overlay Designer** | Visual editor for persistent overlays: webcam frames, stream info bars, social handles, branding elements | **Critical** |
| **Scene Builder** | Design starting soon, BRB, and ending screens with countdown timers and dynamic content | **High** |
| **Alert Queue Manager** | Queue/stack behaviour when multiple alerts fire, with configurable delays and priority rules | **High** |
| **Alert Preview/Sandbox** | Test all alerts without going live, including sequencing and edge cases (powered by the platform event simulator) | **High** |
| **Asset Library** | Import and manage images, audio, fonts, and animations for use across all designs | **Medium** |

## **Phase 3: Unified Chat & Chat Bot**

**Estimated Duration:** 4–6 weeks

A combined chat view that merges messages from Twitch, YouTube, and Kick into a single timeline. Includes a built-in chat bot to replace the need for external bot services.

| Feature | Description | Priority |
| :---- | :---- | ----- |
| **Unified Chat View** | Single timeline merging Twitch IRC, YouTube Live Chat, and Kick chat with platform badges/indicators | **Critical** |
| **Chat Moderation** | Timeout, ban, delete messages, and slow mode controls per-platform from one interface | **Critical** |
| **Built-in Chat Bot** | Custom commands (\!commands), timed messages, auto-moderation rules (link filtering, caps, spam) | **High** |
| **Chat Overlay** | Browser source overlay for OBS showing chat on stream with customisable appearance | **High** |
| **Chat Filters/Search** | Filter by platform, user, mod status, or keyword. Search chat history | **Medium** |
| **User Highlights** | Highlight messages from VIPs, subs, mods with configurable colours and badges | **Medium** |

## **Phase 4: Stream Management & Platform Sync**

**Estimated Duration:** 3–4 weeks

Manage your stream metadata across all platforms from a single interface. Update titles, categories, and tags simultaneously. Schedule streams and view basic analytics.

| Feature | Description | Priority |
| :---- | :---- | ----- |
| **Multi-Platform Sync** | Update stream title, category/game, and tags on Twitch, YouTube, and Kick simultaneously | **Critical** |
| **OBS WebSocket Control** | Full read/write OBS integration: scene switching, source toggling, recording control, stream health monitoring | **Critical** |
| **Stream Scheduling** | Schedule upcoming streams with reminders | **High** |
| **Analytics Dashboard** | Unified view of viewer counts, chat activity, follower/sub events across all platforms | **High** |
| **Clip Capture Trigger** | One-button clip creation on all active platforms | **Medium** |
| **Stream Health Monitor** | Real-time display of bitrate, dropped frames, encoding stats from OBS WebSocket | **Medium** |

## **Phase 5: Themes, Polish & Release**

**Estimated Duration:** 4–6 weeks

Refinement, theme system, and features that set StreamForge apart as a polished product. This phase prepares the app for public release.

| Feature | Description | Priority |
| :---- | :---- | ----- |
| **Theme/Template System** | Export and import complete stream design packages (overlays, alerts, panels, colours, fonts) as self-contained bundles | **High** |
| **Transition Stingers** | Custom animated transitions between scenes, designed in the visual editor | **High** |
| **Panel Designer** | Design Twitch/YouTube channel panels and info sections | **Medium** |
| **Keyboard Shortcuts** | Global hotkeys for alert triggers, scene switches, chat actions | **Medium** |
| **Onboarding Flow** | Guided setup for new users: connect platforms, create first overlay, configure OBS | **Medium** |
| **Auto-Update System** | Tauri built-in updater with signed releases for seamless version updates | **Medium** |

# **14\. Future Roadmap (Post-v1)**

Features planned for after the v1 desktop release. These are noted here to inform architecture decisions but are not in scope for the development phases above.

## **Companion Mobile App**

**Estimated Duration:** 6–8 weeks (post-v1)

The first major post-v1 addition. The mobile companion app acts as a remote control for the desktop app and handles social media broadcasting. Built with React Native (to share TypeScript/component logic) or Flutter.

| Feature | Description | Priority |
| :---- | :---- | ----- |
| **Go-Live Social Posting** | One-tap posting to Twitter/X, Instagram, Discord (webhook), TikTok, Bluesky, Threads, and custom webhooks | **Critical** |
| **Remote Chat Monitor** | View and moderate unified chat from phone | **Critical** |
| **Remote Alert Trigger** | Manually fire alerts, play sounds, or trigger scenes from the mobile app | **High** |
| **Stream Deck Mode** | Customisable button grid for scenes, sounds, alerts, chat commands — a free stream deck alternative | **High** |
| **Stream Status Control** | Update title/game/tags, view analytics, and trigger clips from mobile | **High** |
| **Pre-Stream Checklist** | Configurable checklist to run through before going live | **Medium** |
| **Scheduled Post Templates** | Save and reuse go-live post templates with dynamic fields (game name, stream title, link) | **Medium** |
| **Mobile Pairing** | One-time code pairing system for secure desktop-to-mobile connection | **Critical** |

## **Additional Future Features**

* Self-hosted donations and tipping — custom donation page, alerts, and payment processing

* Theme marketplace — community sharing and potentially selling stream design packages

* Cloud backup and sync — sync designs and settings across devices via remote storage

* Plugin/extension system — allow third-party developers to extend StreamForge functionality

* Multi-cam/multi-source management — deeper OBS integration for source layout control

* VOD management — auto-export and organise VODs across platforms

* Loyalty points and channel currency — built-in loyalty system with redeemable rewards

* AI-powered features — auto-highlight detection, chat sentiment analysis, smart moderation

# **15\. Technical Stack Summary**

| Component | Technology |
| :---- | :---- |
| **Desktop Backend** | Rust (Tauri core, local HTTP server, file system, platform API bridge) |
| **Desktop Frontend** | React \+ TypeScript (UI, visual editors, chat, dashboards) |
| **Internal Comms** | Socket.IO (desktop ↔ browser source overlays, future mobile connection) |
| **External Comms** | Raw WebSocket clients (Twitch EventSub, OBS WebSocket v5, Kick chat) |
| **Local Database** | SQLite (settings, designs, cached data, chat history) |
| **Alert Rendering** | HTML/CSS/JS served as browser sources to OBS |
| **Platform APIs** | Twitch EventSub/Helix, YouTube Data/Live API, Kick API |
| **OBS Integration** | OBS WebSocket protocol (v5) for monitoring and full scene control |
| **Security** | OS keychain for secrets, encrypted SQLite, signed auto-updates |
| **Build/CI** | Tauri CLI, GitHub Actions, monorepo with Turborepo or Nx |

# **16\. Estimated Timeline**

Total estimated development time for the v1 desktop release across all five phases is approximately 21–30 weeks. The timeline assumes a solo or small-team development approach with Claude Code assistance for implementation.

| Phase | Duration | Cumulative |
| :---- | :---: | :---: |
| 1\. Foundation & Core Infrastructure | 4–6 weeks | 4–6 weeks |
| 2\. Alert System & Visual Editor | 6–8 weeks | 10–14 weeks |
| 3\. Unified Chat & Chat Bot | 4–6 weeks | 14–20 weeks |
| 4\. Stream Management & Platform Sync | 3–4 weeks | 17–24 weeks |
| 5\. Themes, Polish & Release | 4–6 weeks | 21–30 weeks |
| *Post-v1: Companion Mobile App* | *6–8 weeks* | *After v1* |

# **17\. Next Steps**

Once this project plan is reviewed and finalised, the next step is to break each phase down into a detailed build order of individual implementation tasks — each task scoped to be a single Claude Code prompt.

Immediate actions after plan approval:

* Finalise the working name (StreamForge is a placeholder)

* Set up the monorepo and Tauri project skeleton

* Begin Phase 1 build order breakdown