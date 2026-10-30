# VidWatch — Product Requirements Document

**Version:** 0.1 — Initial draft  
**Status:** In planning  
**Author:** —  
**Last updated:** June 2026

---

## Table of contents

1. [Why we are building this](#1-why-we-are-building-this)
2. [What we are building](#2-what-we-are-building)
3. [What we are not building](#3-what-we-are-not-building)
4. [Decisions and trade-offs](#4-decisions-and-trade-offs)
5. [Architecture overview](#5-architecture-overview)
6. [Component deep-dives](#6-component-deep-dives)
7. [Data model](#7-data-model)
8. [Video watch-time accuracy](#8-video-watch-time-accuracy)
9. [Categorization](#9-categorization)
10. [Build phases](#10-build-phases)
11. [Open questions](#11-open-questions)

---

## 1. Why we are building this

### The problem

Existing time-tracking tools (browser extensions like RescueTime, Toggl, etc.) measure time spent on a *website*, not on an individual *video*. Opening YouTube for three hours could mean watching one long documentary, or skimming forty Shorts and closing every one after five seconds. Those two sessions have completely different meanings, and current tools cannot tell them apart.

Additionally, every tool that stores watch history in a useful, queryable form requires a paid subscription. The free tiers either lack history export, lack per-video granularity, or lock the data in a proprietary format with no local access.

### What we actually want to know

- How much time did I genuinely watch today, this week, this month?
- What did I actually watch — not just what page was open?
- How much of that time went to Gaming vs. Software tutorials vs. Sports?
- How much time am I losing to YouTube Shorts?
- When I skipped a video after 10 seconds, did I count that?
- Is my consumption trending up or down over time?

### Why build instead of buy

- **Privacy**: watch history is sensitive. We want zero data leaving the Mac.
- **Ownership**: the data lives in a local SQLite file we fully control.
- **Accuracy**: we track *actual* seconds watched, not page-open duration.
- **Cross-browser**: we switch between Chrome, Zen (Firefox-based), and Safari. No existing extension covers all three under one data store.
- **Learning**: this project is an intentional vehicle for learning Go and macOS development.

---

## 2. What we are building

VidWatch is a local-first video watch-time tracker for macOS. It consists of three components that work together:

1. **A browser extension** — runs inside Chrome, Zen (Firefox), and Safari. Detects when a video plays, tracks which time ranges were actually watched (handling seeks and skips), and sends that data to a local HTTP server.

2. **A Go daemon** — a background process that runs at login. It receives events from the browser extension, stitches them into sessions, stores everything in a local SQLite database, and exposes a REST API for the UI.

3. **A Swift menu bar app** — a native macOS menu bar application (built with SwiftUI's `MenuBarExtra` API) that polls the daemon's REST API and displays today's watch summary in a native popover.

### Core capabilities at launch

- Track per-video watch time accurately (accounting for seeking and skipping).
- Distinguish YouTube Shorts from regular YouTube videos.
- Support YouTube and Netflix at launch, with other platforms addable without code changes.
- Display today's total watch time, per-video breakdown, and per-category breakdown in the menu bar.
- Store all data locally in SQLite with no cloud dependency.

---

## 3. What we are not building

These items are explicitly out of scope for the initial version, to keep the project shippable:

| Out of scope | Reason |
|---|---|
| Historical import from YouTube Takeout | Takeout data lacks actual watch duration; the `time` field is when a video was opened, not how long it was watched. Not worth the complexity for inaccurate data. |
| Go TUI (terminal UI) | The menu bar app covers the primary use case. The TUI can be added later as a thin client against the same daemon API. |
| Local web dashboard | Same reasoning — addable later with minimal daemon changes. |
| Cloud sync or backup | Deliberate. Privacy is a hard requirement. Users can back up the SQLite file manually. |
| Safari extension (v1) | Safari web extensions require an Apple Developer account and Xcode packaging. Deprioritised to Phase 3. |
| LLM-based auto-categorisation | Phase 2 feature. Phase 1 uses keyword rules only. |
| Mobile apps | Out of scope entirely. |

---

## 4. Decisions and trade-offs

These are the significant architectural decisions made during planning, including the alternatives that were considered and why they were rejected.

---

### Decision 1: Go daemon as the central data store, not the browser extension

**What we decided:** All storage, logic, and session management lives in a native Go process. The browser extension is a thin event emitter only — it holds no persistent state.

**Alternatives considered:**
- Storing data in `chrome.storage` inside the extension. Rejected because this data is browser-specific, hard to query, not accessible from a native UI, and resets or becomes inaccessible when switching browsers.
- Storing data in a cloud database. Rejected because the hard requirement is local-only, no external dependencies.

**Why this is right:** The daemon is the system of record. Every UI (menu bar app, future TUI, future web dashboard) is a read-only client of the same REST API. Adding or removing a UI surface requires zero changes to the data layer. This is the architecture that scales.

---

### Decision 2: Plain Manifest V3 extension, not WXT or another framework

**What we decided:** Write the browser extension by hand as a small Manifest V3 extension (four files: `manifest.json`, `manifest.firefox.json`, `content.js`, `background.js`). No build step, no TypeScript, no `node_modules`.

**Alternatives considered:**
- WXT (a cross-browser extension framework). Evaluated and rejected for v1 because it introduces a Node.js build pipeline, TypeScript compilation, and significant abstraction for a content script that is fewer than 100 lines. The Firefox and Chrome manifests differ in only two keys; managing that manually is trivial.

**Trade-off accepted:** Manual manifest management for two browsers is slightly more work than a framework, but the total complexity is much lower and the debugging experience is far simpler.

---

### Decision 3: Swift + SwiftUI for the menu bar app, not Go

**What we decided:** The menu bar UI is written in Swift using SwiftUI's `MenuBarExtra` API. It is a thin polling client — it calls `GET /today` on the daemon's REST API every 60 seconds and renders the result.

**Alternatives considered:**
- Go with `getlantern/systray`. This library only supports a flat text menu, not a rich native popover. Replicating a CodexBar-style UI in Go would require embedding a WebView and serving an HTML page from the daemon — a significant extra layer of complexity with worse performance and no native feel.
- Electron. Rejected immediately: too heavy, defeats the purpose of a lightweight menu bar app.

**Why Swift:** SwiftUI `MenuBarExtra` gives a native popover, correct dark/light mode behaviour, Notification Centre integration, and correct energy impact classification — all for free. The app is genuinely ~200 lines of Swift because it does nothing except display data it fetched from the Go daemon. The learning goal (Go and macOS) is still served: Go is where the logic lives, and Swift is a new skill acquired for the UI layer.

**The architectural insight:** The boundary between Go and Swift is a clean local HTTP API. Neither component knows anything about the other's implementation. This is the right seam.

---

### Decision 4: Event-driven tracking with a 10-second flush, not a fixed heartbeat

**What we decided:** The content script listens to the HTML5 `<video>` element's native DOM events (`play`, `pause`, `ended`, `seeked`) and fires a POST to the daemon immediately on each state change. While the video is playing, it also samples `currentTime` every two seconds in memory and flushes accumulated watched segments to the daemon every 10 seconds as a checkpoint.

**Alternatives considered:**
- 30-second heartbeat: rejected because it is completely blind to YouTube Shorts. A 15-second Short that the user watches fully would generate zero heartbeats and be recorded as zero watch time.
- 1-second heartbeat: technically viable (localhost HTTP is ~0.1ms, SQLite handles thousands of writes/second), but it generates unnecessary write volume (~86,400 rows per active video day) and obscures the signal with noise. Events carry more information than heartbeats.

**Why this matters for Shorts:** A Short fires `play` when it starts and `ended` (or a URL change) when the user swipes away. The daemon records the exact `currentTime` at both events. A 5-second swipe on an 18-second Short is faithfully recorded as 5 seconds watched — something a heartbeat model cannot capture.

---

### Decision 5: Track watched segments (ranges), not total duration

**What we decided:** The extension tracks *contiguous ranges* of `currentTime` and sends them to the daemon as `[start_sec, end_sec]` pairs. The daemon merges overlapping ranges and stores the unique sum.

**Example:**
```
User watches  0:00 → 5:00   →  segment [0, 300]
User seeks to 15:00
User watches 15:00 → 18:00  →  segment [900, 1080]
User rewinds  to 2:00
User watches  2:00 → 4:00   →  segment [120, 240]  ← overlaps first segment

Merged: [0, 300], [900, 1080]
Unique seconds watched: 300 + 180 = 480s (8 minutes)
```

**Alternative rejected:** Recording `ended_at - started_at` wall-clock time. This massively overcounts for any video where the user seeks forward. A user who opens a 1-hour video, skips to 58:00, and watches the last 2 minutes would be recorded as watching 60 minutes.

---

### Decision 6: One content script for all platforms

**What we decided:** A single content script handles all streaming platforms. The HTML5 `<video>` element and its DOM events (`play`, `pause`, `ended`, `seeked`, `timeupdate`) are part of the W3C spec and behave identically on YouTube, Netflix, Twitch, and any other site. Platform-specific code is limited to a small adapter object that extracts metadata (title, video ID, whether it is a Short).

**Why this is important:** Adding support for a new platform (e.g. Twitch, Disney+) requires adding one entry to the platform adapters object — roughly 5 lines of JavaScript. No new event listeners, no new daemon endpoints, no new database tables.

---

### Decision 7: SQLite via `modernc.org/sqlite` (pure Go, no CGo)

**What we decided:** Use `modernc.org/sqlite` as the SQLite driver in Go. This is a pure-Go port of SQLite compiled from C to Go using a transpiler. It requires no CGo, no system SQLite library, and produces a single static binary.

**Alternative considered:** `mattn/go-sqlite3`, the most popular Go SQLite driver, requires CGo and a C compiler at build time. This complicates cross-compilation and the build environment. For a personal tool with no concurrency requirements beyond a handful of browser tabs, `modernc` is the better choice.

---

## 5. Architecture overview

```
┌────────────────────────────────────────────────────────┐
│              Browser (Chrome / Zen / Safari)           │
│                                                        │
│   content.js          background.js                    │
│   ──────────          ────────────                     │
│   • Attaches to       • Service worker                 │
│     <video> events    • Queues events if               │
│   • Tracks ranges       daemon is offline              │
│   • Fires POST to     • Retries on reconnect           │
│     localhost:5432                                     │
└──────────────────────────┬─────────────────────────────┘
                           │  POST /event  (JSON)
                           │  (HTTP on localhost)
                           ▼
┌────────────────────────────────────────────────────────┐
│                    Go daemon                           │
│                                                        │
│   HTTP server (net/http)                               │
│   ┌────────────┐  ┌──────────────┐  ┌──────────────┐  │
│   │ /event     │  │ Session      │  │ Categorizer  │  │
│   │ ingestor   │→ │ stitcher     │→ │ (keyword     │  │
│   │            │  │              │  │  rules)      │  │
│   └────────────┘  └──────────────┘  └──────────────┘  │
│                           │                            │
│                    REST API endpoints                  │
│          GET /today  GET /summary  GET /videos         │
└──────────────┬───────────────────────────┬─────────────┘
               │  reads/writes             │  JSON responses
               ▼                           ▼
┌──────────────────────┐      ┌────────────────────────┐
│  SQLite database     │      │  Swift menu bar app    │
│                      │      │                        │
│  ~/.local/share/     │      │  SwiftUI MenuBarExtra  │
│  vidwatch/db.sqlite  │      │  polls every 60s       │
│                      │      │  native popover UI     │
│  videos              │      └────────────────────────┘
│  sessions            │
│  watched_segments    │
└──────────────────────┘
```

**Communication between components:**

| From | To | Protocol |
|---|---|---|
| Browser extension | Go daemon | HTTP POST to `localhost:5432` |
| Swift menu bar app | Go daemon | HTTP GET to `localhost:5432` |
| Go daemon | SQLite | Direct file access (same process) |

There is no inter-process communication beyond plain HTTP on localhost. Any UI surface that can make an HTTP request can talk to the daemon. This is intentional — it is what makes adding a TUI or web dashboard in the future a one-afternoon task.

---

## 6. Component deep-dives

### 6.1 Browser extension

**Files:**

```
vidwatch-extension/
  manifest.json          # Chrome MV3
  manifest.firefox.json  # Firefox MV3 (two key differences: background.type and permissions)
  content.js             # injected into every matched tab
  background.js          # service worker — event queue and retry logic
```

**How it works:**

The browser injects `content.js` into every tab whose URL matches the declared patterns (`*://www.youtube.com/*`, `*://www.netflix.com/*`, etc.). The script queries `document.querySelector('video')` and attaches listeners to the native DOM events:

- `play` — video started or resumed. Fire POST immediately.
- `pause` — video paused or buffering. Fire POST immediately.
- `ended` — video reached its natural end. Fire POST immediately.
- `seeked` — user jumped to a new position. Flush current segment, start a new one.
- `timeupdate` — fires ~4× per second while playing. Used to record the current range in memory. Does *not* trigger a POST on every tick; segments are flushed on the events above and on a 10-second timer.

**Platform metadata adapters:**

```javascript
const PLATFORMS = {
  'youtube.com': {
    getTitle:   () => document.querySelector('h1.ytd-video-primary-info-renderer')?.textContent?.trim(),
    getVideoId: () => new URLSearchParams(location.search).get('v') ?? location.pathname.split('/').pop(),
    isShorts:   () => location.pathname.startsWith('/shorts/'),
    platform:   'youtube',
  },
  'netflix.com': {
    getTitle:   () => document.querySelector('.video-title')?.textContent?.trim(),
    getVideoId: () => location.pathname.match(/watch\/(\d+)/)?.[1],
    isShorts:   () => false,
    platform:   'netflix',
  },
};
```

Adding a new platform means adding one entry to this object.

**Offline resilience:** If the daemon is not running (e.g. Mac just woke from sleep), `background.js` queues events in memory and retries with exponential backoff. Events are not persisted across browser restarts — the acceptable data loss window is small (one browser session).

**Cross-browser differences (Chrome vs Firefox manifest):**

| Field | Chrome MV3 | Firefox MV3 |
|---|---|---|
| `background` | `{ "service_worker": "background.js" }` | `{ "scripts": ["background.js"], "type": "module" }` |
| `host_permissions` | Required for localhost fetch | Not required — covered by `<all_urls>` |

---

### 6.2 Go daemon

**Responsibilities:**
- Receive video events from the browser extension via HTTP.
- Deduplicate events (same video open in two tabs simultaneously).
- Stitch segments into sessions.
- Merge overlapping watched ranges and compute unique watch time.
- Categorise videos using keyword rules.
- Expose a REST API for UI clients.
- Run as a `launchd` service so it starts at login and restarts on crash.

**Key endpoints:**

| Method | Path | Description |
|---|---|---|
| `POST` | `/event` | Ingest a video event from the extension |
| `GET` | `/today` | Today's summary: total time, per-category breakdown |
| `GET` | `/videos` | Paginated list of watched videos with watch time |
| `GET` | `/summary?range=7d` | Aggregated summary for a time range |
| `GET` | `/health` | Liveness check for the menu bar app |

**Session stitching logic:**

A *session* is a continuous viewing of one video. It begins on the first `play` event for a given `(videoId, platform)` pair and ends when:
- A `pause` or `ended` event is received, and no `play` event arrives within 30 seconds (grace period for buffering).
- A new video starts in the same tab (URL change detected).
- The daemon shuts down (flush all open sessions on `SIGTERM`).

**Segment merge algorithm (Go pseudocode):**

```go
func mergeSegments(segs []Segment) float64 {
    if len(segs) == 0 { return 0 }
    sort.Slice(segs, func(i, j int) bool { return segs[i].Start < segs[j].Start })
    merged := []Segment{segs[0]}
    for _, s := range segs[1:] {
        last := &merged[len(merged)-1]
        if s.Start <= last.End {
            if s.End > last.End { last.End = s.End } // extend
        } else {
            merged = append(merged, s)
        }
    }
    var total float64
    for _, s := range merged { total += s.End - s.Start }
    return total
}
```

**Tech stack:**
- Language: Go (latest stable)
- HTTP: `net/http` standard library (no framework)
- SQLite: `modernc.org/sqlite` (pure Go, no CGo)
- ORM / query builder: raw SQL with `database/sql`
- Config: TOML file at `~/.config/vidwatch/config.toml` parsed with `BurntSushi/toml`
- Service management: `launchd` plist at `~/Library/LaunchAgents/com.vidwatch.daemon.plist`

**Project structure:**

```
vidwatch/
  cmd/
    daemon/
      main.go          # entry point, wires everything together
  internal/
    server/
      server.go        # HTTP handler registration
      events.go        # POST /event handler
      summary.go       # GET /today, GET /summary handlers
    store/
      store.go         # SQLite connection and migrations
      videos.go        # video upsert queries
      sessions.go      # session read/write queries
      segments.go      # segment merge and storage
    categoriser/
      categoriser.go   # keyword rule engine
      rules.go         # default rule set
    model/
      types.go         # shared structs: Video, Session, Segment, Event
  config/
    config.go          # config struct and loader
  Makefile
  go.mod
```

---

### 6.3 Swift menu bar app

**Technology:** SwiftUI with the `MenuBarExtra` API (macOS 13+). This API renders a native menu bar item with a popover — no `NSStatusItem` boilerplate required.

**What it shows:**
- Total watch time today (e.g. "2h 14m")
- Per-category breakdown with a small progress bar each (Gaming · 1h 02m, Tech · 45m, Shorts · 27m)
- Last 5 videos watched with individual times
- A "since install" total

**How it works:**

The app is a single Swift file of roughly 200 lines. A `Timer` fires every 60 seconds, calls `GET http://localhost:5432/today`, decodes the JSON response into a `WatchSummary` struct, and publishes it through an `ObservableObject`. The SwiftUI view re-renders automatically.

```swift
@main
struct VidWatchApp: App {
    var body: some Scene {
        MenuBarExtra("VidWatch", systemImage: "play.circle") {
            SummaryView()
        }
        .menuBarExtraStyle(.window)
    }
}
```

**Resilience:** If the daemon is offline (health check fails), the app shows "daemon offline" and a "restart" button that runs `launchctl kickstart`.

**Distribution:** No App Store. Direct install as a `.app` bundle. Launched at login via `launchd` or Login Items.

---

## 7. Data model

### `videos`

One row per unique video. `id` is the platform's native identifier (YouTube `v=` query param, Netflix `/watch/:id` segment).

| Column | Type | Notes |
|---|---|---|
| `id` | TEXT PK | Platform-native video ID |
| `platform` | TEXT | `youtube`, `netflix`, etc. |
| `url` | TEXT | Full URL at time of first watch |
| `title` | TEXT | Page title at time of first watch |
| `channel` | TEXT | Channel or show name (nullable) |
| `duration_sec` | REAL | Total video duration (nullable — unknown until video loads) |
| `is_shorts` | BOOLEAN | True if YouTube Shorts |
| `category` | TEXT | Assigned category. Default: `uncategorized` |
| `first_seen_at` | TIMESTAMP | When the video was first opened |

### `sessions`

One row per continuous viewing session of one video.

| Column | Type | Notes |
|---|---|---|
| `id` | TEXT PK | UUID generated by daemon |
| `video_id` | TEXT FK → videos.id | |
| `started_at` | TIMESTAMP | When play first fired in this session |
| `ended_at` | TIMESTAMP | When session was closed (nullable while open) |
| `unique_watched_sec` | REAL | Merged segment total. Written on session close. |

### `watched_segments`

One row per contiguous watched range within a session. This is the raw truth from which `unique_watched_sec` is derived.

| Column | Type | Notes |
|---|---|---|
| `id` | INTEGER PK | Auto-increment |
| `session_id` | TEXT FK → sessions.id | |
| `start_sec` | REAL | `currentTime` when segment started |
| `end_sec` | REAL | `currentTime` when segment ended |

**Why store raw segments and also the computed total?**

`watched_segments` is the audit trail — if there is ever a bug in the merge algorithm, the raw data can be reprocessed. `sessions.unique_watched_sec` is the cached result used by all read queries so the merge does not run on every API call.

---

## 8. Video watch-time accuracy

### The seek/forward problem

Naively recording `session_end - session_start` overcounts watch time whenever the user seeks forward. The segment model solves this: only time ranges where `timeupdate` events were received count as watched.

### The Shorts problem

YouTube Shorts are often under 30 seconds. They are handled correctly by the event model because:

1. The `play` event fires the instant a Short starts. The daemon opens a session.
2. If the user swipes away, the URL changes and the content script fires a synthetic `ended` event with the final `currentTime`.
3. A 5-second watch of a 15-second Short is recorded as a session with one segment `[0, 5]` — 5 seconds unique watched.

Shorts are tagged `is_shorts = true` via the platform adapter's `isShorts()` function, enabling separate queries like "total Shorts time today."

### Acceptable inaccuracy

The system has one known inaccuracy: if the user's Mac loses power or the browser crashes mid-video, the in-flight segments queued in the background service worker are lost. The maximum data loss is 10 seconds (the flush interval). This is acceptable.

---

## 9. Categorisation

### Phase 1: keyword rules

Categories are defined in `~/.config/vidwatch/categories.toml`:

```toml
[categories]
gaming  = ["let's play", "gameplay", "walkthrough", "speedrun", "gta", "minecraft"]
tech    = ["tutorial", "golang", "programming", "vs code", "docker", "kubernetes"]
sports  = ["highlights", "match", "nba", "premier league", "formula 1", "f1"]
balacho = ["balacho"]
```

The categoriser checks the video title (lowercased) against each rule list in order. First match wins. If no rule matches, the video is tagged `uncategorized`.

Shorts are always additionally tagged `is_shorts = true` regardless of category.

### Phase 2: manual override

The daemon will expose `PATCH /videos/:id` to allow the menu bar app (or a future CLI) to set `category` directly. Manual overrides persist and are never overwritten by the keyword engine.

### Phase 3: LLM-assisted classification

Videos that remain `uncategorized` after keyword matching can optionally be sent to the Claude API (via `anthropic-go`) for classification. This is opt-in, requires an API key in the config file, and runs as a background goroutine with rate limiting. It will never run by default.

---

## 10. Build phases

### Phase 1 — Core data pipeline (target: 2 weekends)

- Go daemon with `POST /event` and `GET /today` endpoints.
- SQLite schema with migrations.
- Session stitching and segment merge algorithm.
- Chrome extension content script and background worker.
- Verify data lands in SQLite with `curl` and `sqlite3`.

**Exit criteria:** Open a YouTube video, watch part of it, seek, close the tab. Query SQLite directly and see an accurate `unique_watched_sec` value.

### Phase 2 — Swift menu bar app (target: 1 weekend)

- `MenuBarExtra` popover with today's summary.
- Per-category breakdown.
- Health check and "daemon offline" state.
- `launchd` plist for daemon auto-start.

**Exit criteria:** Menu bar icon appears at login. Clicking it shows today's watch time without opening a terminal.

### Phase 3 — Firefox / Zen support (target: 1 afternoon)

- Create `manifest.firefox.json`.
- Test in Zen browser.
- Verify events reach the same daemon and appear in the same SQLite database.

**Exit criteria:** Watch a video in Zen. It appears in the menu bar alongside Chrome sessions.

### Phase 4 — Categorisation (ongoing)

- Keyword rule engine in Go.
- `categories.toml` config file with sensible defaults.
- Category breakdown in the menu bar UI.

### Phase 5 — Safari (future)

- Requires Apple Developer account.
- Xcode project to package the extension.
- Submit to Safari Web Extensions API review.

---

## 11. Open questions

| Question | Status | Notes |
|---|---|---|
| How do we handle the same video open in two tabs simultaneously? | Unresolved | Likely: last-write-wins on `sessions`, deduplication by `(videoId, startedAt window)` |
| Should `category` be a foreign key to a `categories` table, or a plain TEXT column? | Decided: TEXT | Keeps schema simple. If categories become complex, migrate later. |
| What port should the daemon use? | Unresolved | `5432` conflicts with PostgreSQL convention. `51432` or a user-configurable port may be better. |
| How do we handle Netflix's dynamic DOM (title loads after page) | Unresolved | Likely: `MutationObserver` on the title element with a short timeout. |
| Minimum macOS version for `MenuBarExtra` API? | Confirmed: macOS 13 Ventura | Acceptable — Ventura is 3 years old at time of writing. |
