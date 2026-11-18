# Making Ozara extensions

Ozara extensions are small add-ons written in **Luau**, the same language Roblox uses for game scripts. An extension can add its own tab to Ozara, react to what's happening (Roblox opening, games joined), show notifications and fetch data from the web.

Every extension runs in a **locked sandbox**. It can't read or write files, run programs or touch Roblox's process. It can only use the `ozara` API below, and only the parts its user approved.

---

## Quick start

1. In Ozara, open **Extensions → Create example**. A working extension appears in `%APPDATA%\Ozara\Extensions\ozara.example.hello`, along with its own sidebar tab.
2. Open that folder and edit `main.luau`.
3. Press **Reload** next to the extension in Ozara to see your changes.

---

## Folder layout

```
my-extension/
├─ manifest.json     required
├─ main.luau         the script Ozara runs (name set by "main")
├─ utils.luau        optional extra modules, loaded with require("utils")
└─ README.md         optional
```

### manifest.json

```json
{
  "id": "yourname.coolthing",
  "name": "Cool Thing",
  "version": "1.0.0",
  "author": "YourName",
  "description": "One line about what it does.",
  "main": "main.luau",
  "permissions": ["events", "notify", "page", "http"],
  "http_hosts": ["api.example.com"]
}
```

| Field | Notes |
|---|---|
| `id` | 3–64 characters: letters, digits, `.` `-` `_`. Unique per extension. Reusing an id **updates** that extension. |
| `name` | Shown in Ozara and on its sidebar tab (first 10 characters). |
| `main` | A `.luau` file at the top of the extension folder. Defaults to `main.luau`. |
| `permissions` | Only ask for what you use. Users see this list before installing. |
| `http_hosts` | Websites `ozara.http.get` may reach. Subdomains count (`example.com` allows `api.example.com`). HTTPS only. |

### Permissions

| Permission | Unlocks |
|---|---|
| `events` | `roblox_opened`, `roblox_closed`, `game_joined`, `game_left`; `ozara.currentGame()`, `ozara.robloxRunning()` |
| `notify` | `ozara.notify`, `ozara.status` |
| `page` | `ozara.ui.*`: your own sidebar tab |
| `http` | `ozara.http.get` to the hosts in `http_hosts` |
| `storage` | `ozara.storage.*`: save your extension's data (1 MB) |
| `friends` | `ozara.friends.list()`, `ozara.friends.open(userId)` |
| `roblox_api` | `ozara.roblox.*`: public Roblox lookups (games, players, servers) |
| `launch` | `ozara.join(placeId, jobId?)`, `ozara.joinUser(userId)` |
| `settings` | `ozara.settings.*`: read and change the FPS cap and graphics settings |
| `theme` | `ozara.theme.set{…}`, `ozara.theme.reset()` |
| `open_links` | `ozara.open(url)` and `ozara.ui.link(label, url)` |
| `clipboard` | `ozara.copy(text)` |
| `favorites` | `ozara.favorites.*` |
| `playtime` | `ozara.playtime.sessions()`, `ozara.playtime.total(placeId?)` |
| `discord` | `ozara.discord.set(details, state)`, `ozara.discord.clear()` |
| `discover` | `ozara.discover.setRow(…)`: your own rows of games on Discover |
| `game_page` | `ozara.gamePage.setButtons(…)` and the `game_viewed` / `game_button` events |
| `profile_card` | `ozara.profileCard.setButtons(…)` and the `profile_viewed` / `profile_button` events |
| `home` | `ozara.home.*`: cards on the Home page |
| `settings_panel` | `ozara.settingsPanel.*`: your own section in Settings |
| `overlay` | `ozara.overlay.*`: items in the F8 in-game wheel |
| `hotkeys` | `ozara.hotkeys.set(…)` and the `hotkey` event |
| `tray` | `ozara.tray.set(…)` and the `tray` event |
| `badge` | `ozara.badge(…)`: a number or dot on your sidebar tab |
| `toasts` | `ozara.toast(text)`: pop-ups inside Ozara |
| `sounds` | `ozara.sound.play("file.wav")` |
| `mods` | `ozara.mods.apply()` / `remove()`: Roblox file replacements from your `mods/` folder |
| `notes` | `ozara.notes.*` |
| `alerts` | `ozara.alerts.add(text)` |
| `navigate` | `ozara.navigate(page)`, `ozara.openGame(placeId)` |
| `window` | `ozara.window.focus()`, `minimize()`, `setOnTop(on)` |
| `roblox_events` | the `roblox_crashed` event |
| `messaging` | `ozara.broadcast(…)` and the `message` event |
| `friends_manage` | `ozara.friends.add(…)`, `ozara.friends.remove(…)` |
| `cache` | `ozara.cache.*` |
| `profiles` | `ozara.profiles.list()`, `ozara.profiles.apply(name)` |
| `account` | `ozara.account()` |
| `servers_page` | `ozara.serversPage.setButtons(…)` and the `server_button` event |
| `search` | the `search` event, `ozara.search.setResults(…)` and the `search_result` event |
| `stats_page` | `ozara.stats.*`: cards and charts on the Stats page |
| `game_tags` | `ozara.gameTags.set(…)`: small tags on game tiles |
| `context_menu` | `ozara.contextMenu.set(…)` and the `context_menu` event |
| `commands` | `ozara.commands.set(…)` (Ctrl+K palette) and the `command` event |
| `launch_hooks` | the `before_launch` event and `ozara.launch.setProfiles(…)` |
| `dialogs` | `ozara.dialog.ask(…)` and the `dialog` event |
| `system_info` | `ozara.system.info()` |
| `layout` | `ozara.layout.set{…}`, `ozara.layout.reset()` |
| `style` | `ozara.style.set{…}`, `ozara.style.reset()`: restyle buttons, Play buttons, game tiles, panels, text and more |

---

## API

### Events

```lua
ozara.on("start", function() end)          -- your script finished loading
ozara.on("tick", function() end)           -- about once a second
ozara.on("page_opened", function() end)    -- user clicked your tab

-- needs "events"
ozara.on("roblox_opened", function() end)
ozara.on("roblox_closed", function() end)
ozara.on("game_joined", function(placeId, jobId, gameName) end)
ozara.on("game_left", function(placeId, jobId) end)

-- from your page (needs "page")
ozara.on("ui", function(buttonId) end)
ozara.on("input", function(inputId, text) end)  -- user pressed Enter
```

All arguments are strings. `gameName` can be empty if Ozara hasn't looked the name up yet.

### Timers

```lua
ozara.every(30, function() end)   -- repeat every 30 seconds
ozara.after(5, function() end)    -- once, 5 seconds from now
```

Timers run on whole seconds (1 minimum).

### Notifications (needs `notify`)

```lua
ozara.notify("Title", "Body text")   -- Windows notification, prefixed with your extension's name
ozara.status("Saved!")               -- line at the bottom of Ozara
```

Notifications are limited to one every 3 seconds per extension.

### Your page (needs `page`)

Build the page, then call `show()`. Calling `clear()` and rebuilding is the normal way to update it.

```lua
ozara.ui.clear()
ozara.ui.heading("Title")
ozara.ui.text("Normal text")
ozara.ui.muted("Smaller grey text")
ozara.ui.separator()
ozara.ui.button("refresh", "Refresh")     -- fires ozara.on("ui", ...) with "refresh"
ozara.ui.input("search", "Type here…")    -- fires ozara.on("input", ...) on Enter
ozara.ui.toggle("music", "Play music", true) -- fires ozara.on("toggle", id, "true"/"false")
ozara.ui.progress(0.4, "40% done")        -- value from 0 to 1
ozara.ui.game(920587237, "Adopt Me!")     -- a game tile; click opens its page in Ozara
ozara.ui.user(1)                          -- a player with avatar; click opens their profile card
ozara.ui.link("Wiki", "https://…")        -- needs "open_links"
ozara.ui.row(function()                   -- put things side by side
    ozara.ui.button("a", "A")
    ozara.ui.button("b", "B")
end)
ozara.ui.image("logo.png", 200)           -- a PNG/JPG from your folder (width optional)
ozara.ui.image("https://example.com/a.png") -- web images need "http" and the host in http_hosts
ozara.ui.chart("bar", "Wins per day", { 3, 5, 2 }, { "Mon", "Tue", "Wed" })
ozara.ui.chart("line", "Players", { 120, 340, 280 })
ozara.ui.show()
```

A page can hold up to 300 items.

### Other places to draw

`ozara.home` (needs `home`), `ozara.settingsPanel` (needs `settings_panel`), `ozara.stats` (needs `stats_page`) and `ozara.overlay` (needs `overlay`) work exactly like `ozara.ui`: `clear()`, add things, `show()`. Clicks on them fire the same `ui`, `input` and `toggle` events.

```lua
ozara.home.clear()
ozara.home.heading("Daily streak: 4")
ozara.home.button("claim", "Claim")
ozara.home.show()
```

The F8 wheel shows up to 8 items from all extensions together: buttons are clickable and fire `ozara.on("overlay_button", function(id) … end)`; headings and text appear as labels. Everything else is ignored in the wheel.

### Web requests (needs `http`)

```lua
local body, err = ozara.http.get("https://api.example.com/data")
if body then
    ozara.log("got", #body, "bytes")
else
    ozara.log("failed:", err)
end
```

- HTTPS only, to hosts in `http_hosts`. Redirects aren't followed.
- 10-second timeout; responses are cut off at 2 MB.
- It blocks your extension while it runs (not Ozara), so keep requests occasional, e.g. in `ozara.every(60, …)`.

### Storage (needs `storage`)

```lua
ozara.storage.set("wins", 5)                       -- strings, numbers, booleans and tables
ozara.storage.set("settings", { color = "#ff00aa", sound = true })
local wins = ozara.storage.get("wins", 0)          -- second value = default if missing
ozara.storage.delete("wins")
for _, key in ipairs(ozara.storage.keys()) do print(key) end
ozara.storage.clear()
```

Saved to `%APPDATA%\Ozara\ExtensionData\<id>.json`. It survives updates of your extension, up to 1 MB.

### Friends (needs `friends`)

```lua
for _, f in ipairs(ozara.friends.list()) do
    print(f.displayName, f.username, f.id)
end
ozara.friends.open(f.id)     -- shows Ozara's profile card for them
```

### Roblox lookups (needs `roblox_api`)

Each returns the result, or `nil, errorText`. They go through Ozara's rate-limited Roblox client, so you don't list any hosts.

```lua
local game = ozara.roblox.game(920587237)          -- name, description, creator, playing, visits, maxPlayers, favorited, genre
local user = ozara.roblox.user(1)                  -- username, displayName, description, created, verified, banned, friends, followers, following
local servers = ozara.roblox.servers(920587237)    -- { id, players, maxPlayers, ping }, …
local found = ozara.roblox.searchGames("obby")     -- { placeId, universeId, name, playing }, …
local people = ozara.roblox.searchUsers("builder") -- { id, username, displayName }, …
local top = ozara.roblox.popular()                 -- same shape as searchGames
```

### Launching (needs `launch`)

```lua
ozara.join(920587237)                 -- join a game
ozara.join(920587237, servers[1].id)  -- join a specific server
ozara.joinUser(1)                     -- follow a player into their game
```

Ozara applies the user's graphics settings first, exactly like its own Play buttons.

### Settings (needs `settings`)

```lua
local s = ozara.settings.get()
print(s.fps)                          -- current FPS cap
print(table.concat(s.fpsOptions, ", "))
for _, f in ipairs(s.flags) do print(f.id, f.label, f.on, f.risky) end
ozara.settings.setFps(144)            -- must be one of fpsOptions
ozara.settings.setFlag("no_shadows", true)
```

Settings marked `risky` (they can freeze Roblox on some PCs) can only be changed by the user.

### Theme (needs `theme`)

```lua
ozara.theme.set({
    accent = "#ff4fa3",            -- buttons, highlights
    accent2 = "#4fd1ff",           -- secondary highlights
    backgroundTop = "#1b0b3a",
    backgroundBottom = "#3a0b2a",
})
ozara.theme.reset()
```

Leave a colour out to keep Ozara's own. When your extension is turned off, Ozara's colours come back automatically.

### Links and clipboard

```lua
ozara.open("https://example.com")   -- needs "open_links"; https only, one every 3 seconds
ozara.copy("text")                  -- needs "clipboard"; write only
```

### Favourites (needs `favorites`)

```lua
for _, g in ipairs(ozara.favorites.list()) do print(g.name, g.placeId) end
ozara.favorites.add(920587237, "Adopt Me!")
ozara.favorites.remove(920587237)
```

### Playtime (needs `playtime`)

```lua
for _, s in ipairs(ozara.playtime.sessions()) do   -- newest last, up to 300
    print(s.name, s.placeId, s.seconds, s.start, s.finish)
end
print(ozara.playtime.total())            -- all games, in seconds
print(ozara.playtime.total(920587237))   -- one game
```

### Discord status (needs `discord`)

```lua
ozara.discord.set("Grinding wins", "Level 42")   -- top line, second line
ozara.discord.clear()                              -- give it back to Ozara
```

Shown under Ozara's own Discord app and logo. It updates within a few seconds.

### Current state (needs `events`)

```lua
local g = ozara.currentGame()     -- { placeId, jobId, name } or nil
print(ozara.robloxRunning())      -- true / false
```

### Discover rows (needs `discover`)

```lua
ozara.discover.setRow("mygroup", "My group's games", {
    { placeId = 920587237, name = "Adopt Me!" },
    606849621,                       -- a plain place id works too
})
ozara.discover.removeRow("mygroup")
```

Up to 5 rows per extension and 30 games per row. Rows show after Trending, labelled with your extension's name.

### Game pages and profile cards

```lua
-- needs "game_page"
ozara.gamePage.setButtons({ { id = "wiki", label = "Open wiki" } })
ozara.on("game_viewed", function(placeId, name) end)
ozara.on("game_button", function(buttonId, placeId) end)

-- needs "profile_card"
ozara.profileCard.setButtons({ { id = "note", label = "Add note" } })
ozara.on("profile_viewed", function(userId) end)
ozara.on("profile_button", function(buttonId, userId) end)
```

### Keyboard shortcuts (needs `hotkeys`)

```lua
ozara.hotkeys.set({
    { id = "rejoin", combo = "Ctrl+Shift+R" },
    { id = "panic", combo = "F9" },
})
ozara.on("hotkey", function(id) end)
```

Up to 5. Letters and numbers need **Ctrl or Alt**, so a shortcut can never be used to watch normal typing; F-keys (except F8, Ozara's own) work alone. Shortcuts work even while Roblox is focused.

### Tray menu (needs `tray`)

```lua
ozara.tray.set({ { id = "quick", label = "Quick join" } })
ozara.on("tray", function(id) end)
```

Items appear under **Extensions** in Ozara's tray menu, prefixed with your extension's name. Up to 8.

### Tab badge, toasts and sounds

```lua
ozara.badge(3)          -- needs "badge"; ozara.badge(nil) clears it, ozara.badge("•") for a dot
ozara.toast("Saved!")   -- needs "toasts"
ozara.sound.play("ding.wav")  -- needs "sounds"; .wav files from your folder
```

### Mod packs (needs `mods`)

Put Roblox file replacements in your extension under `mods/content/…`, mirroring Roblox's own `content` folder:

```
my-extension/
└─ mods/content/textures/Cursors/KeyboardMouse/ArrowCursor.png
```

```lua
ozara.mods.apply()    -- copies them into Roblox (originals are backed up)
ozara.mods.remove()   -- puts the originals back
```

Only files inside `content` are ever touched. They take effect the next time Roblox starts, and they're removed automatically when the extension is turned off or uninstalled. Package files allowed under `mods/`: `.png .jpg .ogg .mp3 .wav .ttf .otf .ktx .tex .dds`.

### Notes, alerts and moving around

```lua
for i, text in ipairs(ozara.notes.list()) do print(i, text) end  -- needs "notes"
ozara.notes.add("Buy the gamepass on Friday")
ozara.notes.remove(1)

ozara.alerts.add("Your favourite game updated!")   -- needs "alerts"

ozara.navigate("discover")     -- needs "navigate"; discover, home, friends, servers, favorites,
                               -- settings, stats, notes, alerts, updates, extensions, or "extension" (your own tab)
ozara.openGame(920587237)
```

### Ozara's window (needs `window`)

```lua
ozara.window.focus()
ozara.window.minimize()
ozara.window.setOnTop(true)
```

### Roblox events (needs `roblox_events`)

```lua
ozara.on("roblox_crashed", function(reason)
    -- Roblox closed while you were in a game, without Ozara closing it.
    -- reason is a plain-English guess, or "" if Ozara couldn't tell.
end)
```

### Talking to other extensions (needs `messaging`)

```lua
ozara.broadcast("scores", { player = "builder", points = 10 })
ozara.on("message", function(channel, data, fromExtensionId) end)
```

Only extensions that also have `messaging` hear it. Messages can be up to 64 KB.

### Managing friends (needs `friends_manage`)

```lua
ozara.friends.add(1, "Roblox")
ozara.friends.remove(1)
```

### Cache, profiles and account

```lua
local c = ozara.cache.size()      -- needs "cache": totalBytes, files, pngBytes, jpegBytes, otherBytes
ozara.cache.clear()               -- returns removedFiles, freedBytes
ozara.cache.setTextureCap(256)    -- 0 (original), 128, 256, 512 or 1024
print(ozara.cache.textureCap())

for _, name in ipairs(ozara.profiles.list()) do print(name) end   -- needs "profiles"
ozara.profiles.apply("Max FPS")

local me = ozara.account()        -- needs "account": userId, username, displayName (or nil)
```

### Servers page (needs `servers_page`)

```lua
ozara.serversPage.setButtons({ { id = "copy", label = "Copy ID" } })
ozara.on("server_button", function(id, placeId, jobId)
    if id == "copy" then ozara.copy(jobId) end      -- ozara.copy needs "clipboard"
end)
```

### Search (needs `search`)

Ozara tells you what the user searched for; reply with your own results, shown under "From <your extension>".

```lua
ozara.on("search", function(query)
    ozara.search.setResults(query, {
        { id = "doc", title = "Docs for " .. query, subtitle = "Opens your page" },
        { title = "Adopt Me!", subtitle = "A game", placeId = 920587237 },   -- opens the game
        { title = "Roblox", userId = 1 },                                     -- opens the profile card
    })
end)
ozara.on("search_result", function(id) print("clicked", id) end)   -- results without placeId / userId
```

Up to 20 results. Results only show while their query matches what's in the search box.

### Game tags (needs `game_tags`)

```lua
ozara.gameTags.set({ { placeId = 920587237, text = "FAV" } })   -- up to 200, 12 characters each
```

If two extensions tag the same game, the first one wins.

### Right-click menus (needs `context_menu`)

```lua
ozara.contextMenu.set({
    game   = { { id = "note", label = "Add a note" } },
    friend = { { id = "dm",   label = "Say hi" } },
})
ozara.on("context_menu", function(itemId, kind, targetId)
    -- kind is "game" (targetId = place ID) or "friend" (targetId = user ID)
end)
```

Game items appear when right-clicking any game tile. Friend items appear when right-clicking a friend on Home (the menu also keeps "Join their game" and "View profile").

### Command palette (needs `commands`)

Ctrl+K opens Ozara's command palette. Add your own entries:

```lua
ozara.commands.set({ { id = "clear", label = "Clear the cache" } })
ozara.on("command", function(id) … end)
```

### Launch hooks (needs `launch_hooks`)

```lua
ozara.on("before_launch", function(placeId, jobId, userId)
    -- fires right before Ozara opens Roblox. Joining a friend: placeId is "" and userId is set.
end)
-- Load a FastFlag profile automatically when joining these games:
ozara.launch.setProfiles({ { placeId = 920587237, profile = "Max FPS" } })
```

### Pop-up questions (needs `dialogs`)

```lua
ozara.dialog.ask("reset", "Reset your stats?", "This can't be undone.", { "Reset", "Cancel" })
ozara.on("dialog", function(id, button)
    if id == "reset" and button == "Reset" then … end   -- button is "" if the user pressed Esc
end)
```

Up to 3 buttons. One question at a time per extension; asking again replaces the old one.

### PC details (needs `system_info`)

```lua
local pc = ozara.system.info()
-- cpu, cores, gpu, os, ramTotalMb, ramFreeMb, screenWidth, screenHeight (any can be missing)
```

The first call can take a second (it's not counted against your 2-second limit); after that it's cached for a minute.

### Layout (needs `layout`)

```lua
ozara.layout.set({ uiScale = 1.1, sidebarLabels = false, tile = 110, roundness = 1.5 })
ozara.layout.reset()
```

Keys: `uiScale`, `sidebar`, `sidebarLabels`, `tile`, `banner`, `bannersPerRow`, `button`, `friendAvatar`, `roundness` (same ranges as Settings → Layout). Changes aren't saved: the user's own layout comes back when your extension is turned off.

### Styles (needs `style`)

`ozara.style.set` restyles Ozara. Every group is optional; leave something out and it keeps Ozara's normal look. Colours are `"#RRGGBB"` or `"#RRGGBBAA"` (the last two digits are see-through-ness). Each `set` replaces your whole previous style, and `ozara.style.reset()` (or turning the extension off) brings Ozara's look back.

```lua
ozara.style.set({
    button    = { fill = "#335FFF", text = "#FFFFFF", hover = "#4A70FF", border = "#FFFFFF",
                  height = 36, padding = 18, textSize = 13, roundness = 8 },
    secondary = { fill = "#393B3D", text = "#FFFFFF", hover = "#4A4C4E", border = "#00000000" },
    chip      = { on = "#FFFFFF", onText = "#111216", off = "#393B3D", offText = "#FFFFFF" },
    play      = { kind = "roblox", fill = "#335FFF", text = "#FFFFFF", hover = "#4A70FF", height = 40, roundness = 8 },
    tile      = { kind = "roblox", labelFill = "#191B1D", labelText = "#FFFFFF", roundness = 8 },
    panel     = { fill = "#191B1D", border = "#2C2E30", roundness = 10, padding = 18 },
    heading   = { color = "#FFFFFF", size = 18 },
    text      = { color = "#E5E5E5", size = 14 },
    muted     = { color = "#A0A0A0" },
    sidebar   = { fill = "#111216", active = "#393B3D", icon = "#FFFFFF" },
    toggle    = { on = "#335FFF", off = "#393B3D", knob = "#FFFFFF" },
    input     = { fill = "#111216", border = "#393B3D" },
    selection = { color = "#335FFF" },
    popup     = { fill = "#232527", border = "#393B3D" },
    status    = { good = "#3FC679", warn = "#F7C948", bad = "#F25555" },
})
```

| Group | What it changes |
|---|---|
| `button` | Main buttons. `height`, `padding`, `textSize` and `roundness` also apply to secondary buttons |
| `secondary` | Secondary (see-through) buttons |
| `chip` | Filter chips, like the ones on Search |
| `play` | Play buttons. `kind`: `"pill"` (normal), `"roblox"` (wide blue button with ▶) or `"icon"` (just ▶) |
| `tile` | Game tiles. `kind`: `"ozara"` (normal) or `"roblox"` (square picture, name underneath). `labelFill` is for the Ozara kind |
| `panel` | The dark cards everything sits in |
| `heading` | Section titles |
| `text` | Normal text and text boxes |
| `muted` | The grey hint text |
| `sidebar` | Sidebar background, the selected tab and icon colour |
| `toggle` | On/off switches |
| `input` | Text box background and border |
| `selection` | Selected text highlight |
| `popup` | Pop-ups, menus and windows |
| `status` | The green / yellow / red used for good, warning and error messages |

Sizes are clamped to sensible ranges, so a typo can't make Ozara unusable. Combine with `ozara.theme.set` (accent and background colours) and `ozara.layout.set` (sizes) for a complete look. See `examples/ozara-styles.ozext`, which includes a Roblox look.

### JSON (always available)

```lua
local text = ozara.json.encode({ a = 1, list = { 1, 2, 3 } })
local value, err = ozara.json.decode(text)
```

Handy with `ozara.http.get` for web APIs.

### Other

```lua
ozara.log("anything", 123)          -- shows in the extension's Log on the Extensions page (print does the same)
local utils = require("utils")      -- loads utils.luau from your extension folder
ozara.extension.id / .name / .version
ozara.ozaraVersion                  -- e.g. "1.4.0"
```

---

## Limits

| Limit | Value |
|---|---|
| Time per event handler | 2 seconds of Luau code (longer handlers are stopped with an error). Time spent waiting on web requests, Roblox lookups or the cache doesn't count. |
| Memory | 64 MB |
| Package size | 10 MB |
| Files kept from a package | `.luau` `.json` `.md` `.txt` `.png` |

Errors never crash Ozara. They show up in red on the Extensions page and in the extension's Log.

---

## Sharing your extension

Zip the **contents** of your extension folder (so `manifest.json` is at the top of the zip, or inside a single folder) and rename it from `.zip` to **`.ozext`**.

Users install it by dragging it onto Ozara or double-clicking it. Ozara shows the name, author and permissions, and asks before installing.

---

## Example: game counter

```lua
local count = 0

local function draw()
    ozara.ui.clear()
    ozara.ui.heading("Games today")
    ozara.ui.text(tostring(count))
    ozara.ui.button("reset", "Reset")
    ozara.ui.show()
end

ozara.on("start", draw)
ozara.on("game_joined", function(_, _, name)
    count += 1
    ozara.notify("Joined", name ~= "" and name or "a game")
    draw()
end)
ozara.on("ui", function(id)
    if id == "reset" then
        count = 0
        draw()
    end
end)
```

Manifest permissions for this one: `["events", "notify", "page"]`.

---

## What extensions can't do

They can't read or write files (only their own storage), start programs, inject into or read memory of Roblox, see your Roblox login, or reach websites that aren't in their manifest. If an extension claims it needs any of that, it can't have it in Ozara.
