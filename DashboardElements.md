---
name: "Library/rammelmueller/DashboardElements"
tags: meta/library
pageDecoration.prefix: "📊 "
files:
- TileTemplate.md
- DemoTileLinks.md
- DemoTileGoals.md
- DemoTileMarkdown.md
- DemoTileButton.md
- DemoTileCommandButton.md
---

# Dashboard Elements

Pinterest-style dashboard tiles: a masonry wall of same-width, variable-height tiles — markdown text tiles and one-click shortcut buttons — to visually organize links and related things next to each other, e.g. on index pages. Tiles are separate pages; the dashboard page itself holds nothing but the widget.

## DEMO WIDGET
The tiles below come from the demo tile pages shipped with this library (tagged `dashboard-demo`).

${Dashboard{{"Columns", 3}, {"Tiles", "dashboard-demo"}}}

## How it Works

Tiles are separate pages in your space, tagged `dashboard-tile` plus a membership tag that assigns them to a dashboard. The widget — the only thing on the dashboard page — collects the pages carrying both tags, reads each page's body as markdown content, and renders them into a masonry layout: tiles flow into same-width columns, each tile keeping its natural (variable) height, placed greedily into the currently shortest column — a compact, Pinterest-style arrangement that approximates row-major reading order.

Create new tiles with the **Tile** command (the page template shipped with this library pre-fills the frontmatter); editing a tile is just editing a normal page.

> **warning** Important
>   * Tile content is rendered when the widget renders. If you edit a tile page, refresh the widget to see the changes.

### ✨ Features

- **Tiles as pages** — every tile is a regular page: full markdown editing, live preview, wiki links; the dashboard page holds nothing but the widget
- **Tile command** — the shipped page template creates pre-filled tile pages
- **Masonry layout** — same-width columns with variable-height tiles, placed greedily into the shortest column in `order` sequence
- **Markdown text tiles** — optional title header plus the page body as content
- **Shortcut button tiles** — the whole tile is one action: navigate to a page or run a command
- **Configurable colors** — any CSS background color per tile, with an optional text color override
- **Responsive breakpoints** — the configured column count applies on wide screens and is automatically reduced on narrow ones; the layout re-flows on window resize
- **No-flash rendering** — tiles are hidden until the layout engine has placed them

### 🚫 Known Limitations

- Greedy shortest-column placement approximates row-major order — the exact grid position of a tile can differ from strict left-to-right/top-to-bottom placement when tile heights vary
- The layout engine is installed once per browser session; **engine updates require a page reload** to take effect
- Tile content is static at render time — there is no interactive filtering or editing of tiles
- The Tile command pre-fills a fixed membership tag (`dashboard-main`); edit it in the new page's frontmatter when a tile belongs to another dashboard
- Updating the library re-pulls the shipped tile pages, overwriting local changes to them

## Setup and Configuration

### The widget

```lua
${Dashboard{{"Columns", 3}, {"Tiles", "dashboard-main"}}}
```

*   **`{"Columns", N}`**: (Optional) Number of columns on wide screens. Defaults to `3`. Below ~900px the board falls back to 2 columns (capped at the configured count), below ~600px to a single column.
*   **`{"Tiles", "<tag>"}`**: The membership tag whose tile pages this dashboard collects.

### Tile pages

Every tile is a page tagged `dashboard-tile` (the universal marker) plus the dashboard's membership tag:

```markdown
---
tags:
- dashboard-tile
- dashboard-main
title: 🥅 Current near-term goals
order: 20
color: oklch(0.85 0.08 95)
---
- Talk for CNCF meetup -> [[talks/the-why-and-how-of-self-hosted-ai]]
```

Frontmatter fields:

| Field | Meaning |
|---|---|
| `tags` | must include `dashboard-tile` and the dashboard's membership tag |
| `title` | (Optional) header line; absent or empty means no header |
| `order` | (Optional) number; tiles sort ascending, unset tiles last (then by page name) |
| `color` | (Optional) any CSS background color |
| `textColor` | (Optional) text color override, useful on dark tile colors |
| `label` | button label (button tiles) |
| `link` | navigate to this page when the tile is clicked (button tiles) |
| `command` | run this command when the tile is clicked (button tiles) |

The page body below the frontmatter is the tile's markdown content. A tile with a `link` or `command` field is a button tile; a tile with a non-empty body is a text tile. Quote frontmatter values that contain `: ` (colon followed by a space) or start with `#`.

### The Tile command

The shipped `TileTemplate` page template registers the **Tile** command: it prompts for a page name (suggested under `tiles/`) and creates a page pre-filled with the tile frontmatter — edit the membership tag when the tile belongs to another dashboard, fill in the `title`, and write the body.

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
-- ------------- Escape helper for plain-text values -------------
local function escapeHtml(s)
    local out = tostring(s or "")
    out = out:gsub('"', '&quot;')
    out = out:gsub('<', '&lt;')
    out = out:gsub('>', '&gt;')
    return out
end

-- Global action handler, redefined on every library evaluation so library
-- updates apply to live sessions without a page reload. The listener
-- registered below is a thin, stable shim that dispatches to this global.
function dashboardHandleAction(detail)
    if detail and detail.target ~= nil then
        if detail.action == "page" then
            editor.navigate(tostring(detail.target))
        elseif detail.action == "command" then
            editor.invokeCommand(tostring(detail.target))
        end
    end
end

-- ------------- Event Listeners -------------
if not js.window.dashboardActionListenerAdded then
    js.window.addEventListener("sb-dash-action", function(e)
        dashboardHandleAction(e.detail)
    end)
    js.window.dashboardActionListenerAdded = true
end

-- ------------- Main Dashboard Function -------------
function Dashboard(options)
    local columns = 3
    local tilesTag = nil

    for _, opt in ipairs(options or {}) do
        if opt[1] == "Columns" then
            columns = math.max(1, math.floor(tonumber(opt[2]) or 3))
        end
        if opt[1] == "Tiles" then tilesTag = tostring(opt[2]) end
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

    -- Body of a page: the text below the frontmatter block, or the whole
    -- text when the page has no frontmatter
    local function pageBody(text)
        if text == nil then return nil end
        local firstLineEnd = text:find("\n") or (#text + 1)
        local firstLine = text:sub(1, firstLineEnd - 1):gsub("\r$", "")
        if firstLine ~= "---" then return text end
        local pos = firstLineEnd + 1
        while pos <= #text do
            local lineEnd = text:find("\n", pos) or (#text + 1)
            local line = text:sub(pos, lineEnd - 1):gsub("\r$", "")
            if line == "---" then
                return text:sub(lineEnd + 1)
            end
            if lineEnd > #text then break end
            pos = lineEnd + 1
        end
        return text
    end

    -- Everything is rendered as a plain HTML string (the same battle-tested
    -- approach as the Kanban board); markdown content is rendered natively
    -- through SilverBullet's markdown pipeline.
    local html = '<div class="dash-board dash-pending" data-columns="' .. tostring(columns) .. '">'

    local tileCount = 0
    if tilesTag ~= nil then
        -- Tiles are pages tagged dashboard-tile plus the dashboard's
        -- membership tag; the tag is captured in a local and referenced
        -- inside the query, the same pattern the Std library's
        -- widgets.subPages uses.
        local tilePages = query[[
            from p = index.tag "page"
            where table.includes(p.tags, "dashboard-tile") and table.includes(p.tags, tilesTag)
        ]]

        local tiles = {}
        for p in tilePages do
            local order = tonumber(p.order)
            if order == nil then order = math.huge end
            table.insert(tiles, {
                Name = p.name,
                Title = p.title,
                Color = p.color,
                TextColor = p.textColor,
                Label = p.label,
                Link = p.link,
                Command = p.command,
                Order = order,
                Body = pageBody(space.readPage(p.name)),
            })
        end

        -- order ascending, unset tiles last, ties by page name
        table.sort(tiles, function(a, b)
            if a.Order ~= b.Order then return a.Order < b.Order end
            return tostring(a.Name) < tostring(b.Name)
        end)

        for _, tile in ipairs(tiles) do
            local tileHtml = nil
            if tile.Link ~= nil or tile.Command ~= nil then
                -- Button tile: rendered with data attributes; the click is
                -- handled by the delegated engine (see jsCode), which
                -- dispatches to the Lua event listener registered above
                local action, target
                if tile.Link ~= nil then
                    action = "page"
                    target = tostring(tile.Link)
                else
                    action = "command"
                    target = tostring(tile.Command)
                end
                tileHtml = '<div class="dash-tile dash-btn-tile"' ..
                    ' data-action="' .. action .. '"' ..
                    ' data-target="' .. escapeHtml(target) .. '"' ..
                    ' style="' .. styleString(tile) .. '">' ..
                    escapeHtml(tile.Label or tile.Name or "?") ..
                    '</div>'
                tileCount = tileCount + 1
            elseif tile.Body ~= nil and tile.Body:match("%S") ~= nil then
                -- Text tile: optional title plus the page body as markdown,
                -- rendered natively by SilverBullet's markdown pipeline
                local inner = ""
                if tile.Title ~= nil and tile.Title ~= "" then
                    inner = '<div class="dash-tile-title">' .. escapeHtml(tile.Title) .. '</div>'
                end
                inner = inner .. (markdown.markdownToHtml(tile.Body) or "")
                tileHtml = '<div class="dash-tile" style="' .. styleString(tile) .. '">' .. inner .. '</div>'
                tileCount = tileCount + 1
            end
            -- Tiles with neither a body nor Link/Command are skipped

            if tileHtml ~= nil then
                html = html .. tileHtml
            end
        end
    end

    if tileCount == 0 then
        html = html .. '<div class="dash-empty">No tiles configured.</div>'
    end

    html = html .. '</div>' -- close board

    local jsCode = [[
    (function() {
        // Masonry engine: installed once per session. Distributes tiles into
        // N same-width columns using greedy shortest-column placement (tile
        // order preserved, Pinterest-style), reacting to widget re-renders
        // via a MutationObserver and to window resizes (column breakpoints).
        // Also dispatches shortcut-button clicks to the Lua side.
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

        // Shortcut buttons: dispatch clicks to the Lua event listener
        document.addEventListener("click", (e) => {
            const btn = e.target.closest ? e.target.closest('.dash-btn-tile') : null;
            if (!btn) return;
            e.preventDefault();
            const action = btn.dataset.action;
            const target = btn.dataset.target;
            if (action && target) {
                window.dispatchEvent(new CustomEvent("sb-dash-action", {
                    detail: { action: action, target: target }
                }));
            }
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
        html = html
    }
end
```
