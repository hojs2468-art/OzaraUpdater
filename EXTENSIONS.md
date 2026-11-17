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
| `events` | `roblox_opened`, `roblox_closed`, `game_joined`, `game_left` |
| `notify` | `ozara.notify`, `ozara.status` |
| `page` | `ozara.ui.*`: your own sidebar tab |
| `http` | `ozara.http.get` to the hosts in `http_hosts` |

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

### Other

```lua
ozara.log("anything", 123)          -- shows in the extension's Log on the Extensions page (print does the same)
local utils = require("utils")      -- loads utils.luau from your extension folder
ozara.extension.id / .name / .version
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

They can't read or write files, start programs, inject into or read memory of Roblox, see your Roblox login, or reach websites that aren't in their manifest. If an extension claims it needs any of that, it can't have it in Ozara.
