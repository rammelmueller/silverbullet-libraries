---
name: "Library/rammelmueller/DashboardElements"
tags: meta/library
pageDecoration.prefix: "📊 "
---

# Dashboard Elements

Pinterest-style dashboard tiles: a masonry wall of same-width, variable-height tiles — markdown text tiles and one-click shortcut buttons — to visually organize links and related things next to each other, e.g. on index pages.

## DEMO WIDGET
${Dashboard{
  {"Columns", 3},
  {"Tiles", {
    { {"Title", "📌 Quick Links"}, {"Content", [==[
Shortcuts to the places that matter:
* [[Library/rammelmueller/KanbanBoard|Kanban Board]]
* [[Library/rammelmueller/Reminder|Reminder Library]]
]==]} },
    { {"Title", "Markdown tiles"}, {"Content", [==[
Tiles render **any markdown** — [links](https://silverbullet.md), lists:

* one
* two

`code blocks` too.
]==]}, {"Color", "oklch(0.85 0.08 95)"} },
    { {"Label", "⏰ Reminder"}, {"Page", "Library/rammelmueller/Reminder"} },
    { {"Label", "📄 New page from template"}, {"Command", "Page: From Template"} },
    { {"Label", "🏠 Space index"}, {"Page", "index"} },
    { {"Title", "Colors"}, {"Content", [==[
Use the **Color** and **TextColor** pairs with any CSS color — the yellow tile above uses `oklch(0.85 0.08 95)`.
]==]} },
  }}
}}

## How it Works

The widget takes a list of tiles and renders them into a masonry layout: tiles flow into same-width columns, each tile keeping its natural (variable) height. Tiles are placed in configuration order into the currently shortest column — a compact, Pinterest-style arrangement that approximates row-major reading order.

> **warning** Important
>   * Tile content is rendered once when the widget renders. If you edit the tile configuration, **refresh the widget** to see the changes.

### ✨ Features

- **Masonry layout** — same-width columns with variable-height tiles, placed greedily into the shortest column in configuration order
- **Markdown text tiles** — optional title header plus free markdown content: wiki links, lists, code, everything SilverBullet renders
- **Shortcut button tiles** — the whole tile is one action: navigate to a page or run a command
- **Configurable colors** — any CSS background color per tile, with an optional text color override
- **Responsive breakpoints** — the configured column count applies on wide screens and is automatically reduced on narrow ones; the layout re-flows on window resize
- **No-flash rendering** — tiles are hidden until the layout engine has placed them

### 🚫 Known Limitations

- Greedy shortest-column placement approximates row-major order — the exact grid position of a tile can differ from strict left-to-right/top-to-bottom placement when tile heights vary
- The layout engine is installed once per browser session; **engine updates require a page reload** to take effect
- Tile content is static at render time — there is no interactive filtering or editing of tiles

## Setup and Configuration

### Widget Parameters

*   **`{"Columns", N}`**: (Optional) Number of columns on wide screens. Defaults to `3`. Below ~900px the board falls back to 2 columns (capped at the configured count), below ~600px to a single column.
*   **`{"Tiles", { ... }}`**: The tile list. Each tile is itself a pair list made of the following pairs:
    *   **`{"Title", "..."}`**: (Optional) Header line for text tiles.
    *   **`{"Content", [==[ ... ]==]}`**: Markdown content — makes the tile a text tile.
    *   **`{"Color", "<css color>"}`**: (Optional) Background color — any CSS color value; defaults to the neutral panel background.
    *   **`{"TextColor", "<css color>"}`**: (Optional) Text color override, useful on dark tile colors.
    *   **`{"Label", "..."}`**: Button label — makes the tile a button tile.
    *   **`{"Page", "..."}`**: Navigate to this page when the tile is clicked.
    *   **`{"Command", "..."}`**: Run this command when the tile is clicked.

A tile with a `Page` or `Command` pair is a button tile; a tile with a `Content` pair is a text tile.

> **note** Write markdown content in `[==[ ... ]==]` long strings: plain `[[ ]]` strings would be terminated by `[[WikiLinks]]` inside the content.

### Widget example

```lua
${Dashboard{
  {"Columns", 3},
  {"Tiles", {
    { {"Title", "Work"}, {"Content", [==[
Everything work-related:
* [[work/inbox|Inbox]]
* [[work/meetings|Meetings]]
]==]}, {"Color", "oklch(0.85 0.08 95)"} },
    { {"Label", "⏰ Open Reminders"}, {"Page", "Library/rammelmueller/Reminder"} },
    { {"Label", "🔍 Search space"}, {"Command", "Search: Space"}, {"Color", "oklch(0.8 0.12 250)"}, {"TextColor", "white"} },
  }}
}}
```

# Implementation

## CSS Styling

```space-style
#sb-main .cm-editor .sb-lua-directive-block:has(.dash-board) .button-bar { 
  top: -40px; 
  padding:0; 
  border-radius: 2em; 
  opacity:0.2; 
  transition: all 0.5s ease;
} 
#sb-main .cm-editor .sb-lua-directive-block:has(.dash-board) .button-bar:hover { 
  opacity:1;
}


/* ---------------------------------
   Dashboard Masonry Board
---------------------------------- */

.dash-board {
  display: flex;
  gap: 10px;
  align-items: flex-start;
  padding: 10px;
}

/* Hidden until the masonry engine has laid out the tiles */
.dash-board.dash-pending {
  visibility: hidden;
}

.dash-column {
  flex: 1 1 0;
  min-width: 0;
  display: flex;
  flex-direction: column;
  gap: 10px;
}


/* ---------------------------------
   Tiles
---------------------------------- */

.dash-tile {
  width: 100%;
  box-sizing: border-box;
  background: oklch(from var(--modal-help-background-color) l c h / 0.4);
  border-radius: 10px;
  padding: 12px;
  overflow-wrap: break-word;
}

.dash-tile > :first-child { margin-top: 0; }
.dash-tile > :last-child { margin-bottom: 0; }

.dash-tile-title {
  font-weight: bold;
  margin-bottom: 6px;
}

/* Button tiles: the whole tile is one action */
.dash-btn-tile {
  cursor: pointer;
  min-height: 60px;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  font-weight: bold;
  transition: filter 0.15s ease;
}

.dash-btn-tile:hover {
  filter: brightness(1.08);
}

.dash-btn-tile:active {
  filter: brightness(0.95);
}


/* ---------------------------------
   Empty State
---------------------------------- */

.dash-empty {
  width: 100%;
  min-height: 120px;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 24px 16px;
  box-sizing: border-box;
  border-radius: 18px;
  background: oklch(from var(--modal-help-background-color) l c h / 0.4);
  color: oklch(from var(--modal-help-background-color) calc(l - 0.14) c h);
  font-style: italic;
  text-align: center;
}

```

## Lua Implementation
```space-lua
-- ------------- Main Dashboard Function -------------
function Dashboard(options)
    local columns = 3
    local tileSpecs = {}

    for _, opt in ipairs(options or {}) do
        if opt[1] == "Columns" then
            columns = math.max(1, math.floor(tonumber(opt[2]) or 3))
        end
        if opt[1] == "Tiles" then tileSpecs = opt[2] or {} end
    end

    -- Style attribute from the Color and TextColor pairs
    local function styleString(tile)
        local style = ""
        if tile.Color ~= nil then
            style = style .. "background: " .. tostring(tile.Color) .. ";"
        end
        if tile.TextColor ~= nil then
            style = style .. "color: " .. tostring(tile.TextColor) .. ";"
        end
        return style
    end

    local board = dom.div {
        class = "dash-board dash-pending",
        ["data-columns"] = tostring(columns),
    }

    local tileCount = 0
    for _, spec in ipairs(tileSpecs) do
        -- Normalize the tile's pair list into a lookup table
        local tile = {}
        for _, pair in ipairs(spec) do
            tile[pair[1]] = pair[2]
        end

        local tileEl = nil
        if tile.Page ~= nil or tile.Command ~= nil then
            -- Button tile: the whole tile is one action
            local onclick
            if tile.Page ~= nil then
                local page = tostring(tile.Page)
                onclick = function() editor.navigate(page) end
            else
                local command = tostring(tile.Command)
                onclick = function() editor.invokeCommand(command) end
            end
            local elSpec = {
                class = "dash-tile dash-btn-tile",
                onclick = onclick,
                tostring(tile.Label or "?"),
            }
            local style = styleString(tile)
            if style ~= "" then elSpec.style = style end
            tileEl = dom.div(elSpec)
        elseif tile.Content ~= nil then
            -- Text tile: optional title plus markdown content
            local elSpec = { class = "dash-tile" }
            local style = styleString(tile)
            if style ~= "" then elSpec.style = style end
            if tile.Title ~= nil then
                elSpec[#elSpec + 1] = dom.div { class = "dash-tile-title", tostring(tile.Title) }
            end
            elSpec[#elSpec + 1] = tostring(tile.Content)
            tileEl = dom.div(elSpec)
        end
        -- Tiles with neither Content nor Page/Command are skipped

        if tileEl ~= nil then
            board.appendChild(tileEl)
            tileCount = tileCount + 1
        end
    end

    if tileCount == 0 then
        board.appendChild(dom.div { class = "dash-empty", "No tiles configured." })
    end

    local jsCode = [[
    (function() {
        // Masonry engine: installed once per session. Distributes tiles into
        // N same-width columns using greedy shortest-column placement (tile
        // order preserved, Pinterest-style), reacting to widget re-renders
        // via a MutationObserver and to window resizes (column breakpoints).
        if (window.dashEngineInstalled) return;
        window.dashEngineInstalled = true;

        const GAP = 10;

        const effectiveColumns = (board) => {
            const configured = parseInt(board.dataset.columns || "3", 10) || 3;
            const width = board.clientWidth || 9999;
            let cols = configured;
            if (width < 600) cols = 1;
            else if (width < 900) cols = Math.min(configured, 2);
            return cols;
        };

        const placeTiles = (board, tiles) => {
            board.querySelectorAll(':scope > .dash-column').forEach((c) => c.remove());
            if (tiles.length === 0) return;
            const cols = Math.max(1, Math.min(effectiveColumns(board), tiles.length));
            const colEls = [];
            for (let i = 0; i < cols; i++) {
                const col = document.createElement('div');
                col.className = 'dash-column';
                board.appendChild(col);
                colEls.push({ el: col, height: 0 });
            }
            // Greedy shortest-column placement; each tile appended gets the
            // final column width, so offsetHeight is measured correctly
            for (const tile of tiles) {
                let shortest = colEls[0];
                for (const c of colEls) {
                    if (c.height < shortest.height) shortest = c;
                }
                shortest.el.appendChild(tile);
                shortest.height += tile.offsetHeight + GAP;
            }
        };

        const initBoard = (board) => {
            board.dataset.dashInit = "1";
            const tiles = Array.from(board.children).filter(
                (el) => el.classList && el.classList.contains('dash-tile')
            );
            // Tag configuration order once; used to restore tile order on
            // re-layouts (resize/breakpoints)
            tiles.forEach((t, i) => { t.dataset.idx = i; });
            placeTiles(board, tiles);
            board.classList.remove('dash-pending');
        };

        const relayoutBoard = (board) => {
            const tiles = Array.from(board.querySelectorAll('.dash-tile')).sort(
                (a, b) => (parseInt(a.dataset.idx || "0", 10) - parseInt(b.dataset.idx || "0", 10))
            );
            placeTiles(board, tiles);
        };

        // Fast path: layout boards already in the DOM at install time
        document.querySelectorAll('.dash-board:not([data-dash-init])').forEach(initBoard);

        // SilverBullet re-renders widget DOM without re-invoking the Lua
        // function; the observer catches every fresh board
        const observer = new MutationObserver(() => {
            document.querySelectorAll('.dash-board:not([data-dash-init])').forEach(initBoard);
        });
        observer.observe(document.body, { childList: true, subtree: true });

        // Breakpoints: re-layout all boards on window resize (debounced)
        let resizeTimer = null;
        window.addEventListener('resize', () => {
            if (resizeTimer) clearTimeout(resizeTimer);
            resizeTimer = setTimeout(() => {
                document.querySelectorAll('.dash-board[data-dash-init]').forEach(relayoutBoard);
            }, 150);
        });
    })();
    ]]

    if not js.window.dashEngineInstalled then
        local scriptEl = js.window.document.createElement("script")
        scriptEl.innerHTML = jsCode
        js.window.document.body.appendChild(scriptEl)
    end

    return widget.new {
        display = "block",
        html = board
    }
end
```
