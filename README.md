# Ozara

A fast Roblox launcher and companion app, written in Rust.
Ozara never injects into, hooks, or reads the memory of Roblox. It only edits Roblox's own settings files,
reads its log files, opens official `roblox://` links, and uses Roblox's public web API.

## Layout

- `ozara-core/` standard-library-only logic (settings, FPS cap, join links, log parsing, playtime, servers, versions). Fully unit tested.
- `ozara-app/` the window (egui/eframe) that uses the core.

## Build (Windows)

1. Install Rust from https://rustup.rs (accept the defaults; it also asks for the Visual Studio C++ Build Tools).
2. Open this folder in VS Code (install the "rust-analyzer" extension).
3. In a terminal:

```
cd ozara-core
cargo test
cd ../ozara-app
cargo run
```

`cargo run --release` makes the fast build. If anything fails to compile, copy the full error text to Claude.

## What works in this first version

Home (continue playing), Servers (public server list, sort, join), Settings (graphics presets, FPS cap, Apply, Apply and rejoin),
Stats (playtime from Roblox's logs), Updates (shows installed versions).

## Not done yet

Sign in with Roblox (OAuth) and the friends list, game search, alerts, server notes, the real updater,
the overlay, the custom font and wordmark, and the Windows icon.
