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
ozara.ui.show()
```

A page can hold up to 300 items.

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
| Time per event handler | 2 seconds (longer handlers are stopped with an error) |
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
