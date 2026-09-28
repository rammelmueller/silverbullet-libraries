---
name: "Library/rammelmueller/KanbanBoard"
tags: meta/library
pageDecoration.prefix: "📋️ "
---

# Kanban Board

This widget creates a customisable Kanban board to visualise and manage your **pages**.
Cards are whole pages; the columns are driven by a frontmatter attribute (typically `status`), and dragging a card between columns rewrites that attribute directly in the page's frontmatter.

## DEMO WIDGET
${KanbanBoard(
  query[[from item = index.tag "page" where table.includes(item.tags, "task")]], 
  {
    {"Column", "status"},
    {"Columns", {
      {"open", "📥 To Do","purple"},
      {"waiting", "⏳ Waiting","orange"},
      {"in progress", "🏃 In Progress", "blue"}
    }},
    {"Tags", {"personal"}}
  }
)}

Pages tagged `task` and `personal` in their frontmatter appear on this board. Drag a card to another column to update the page's `status`.

## How it Works

The Kanban board works by querying pages and organising them into columns based on a frontmatter attribute, typically `status`.
You can define the columns and their corresponding status values in the widget's parameters.

> **warning** Important
>   * If you manually edit a page's frontmatter, you **must refresh the widget** for changes to appear.

### ✨ Features

- **Pages instead of tasks** — cards are whole pages; the column attribute and all card fields are read from frontmatter
- **Drag & drop** — move cards between columns; the status attribute is updated directly in the page's frontmatter, creating a frontmatter block if the page has none yet
- **Tag filter** — restrict the board to pages carrying all of the given frontmatter tags (`Tags` option)
- **Tag chips** — every tag found on the board's pages (except the `Tags`-option tags and tags carried by *all* pages) appears as a toggleable chip in the top bar; switch one off to hide all pages carrying it
- **Customisable columns** — define your own workflow stages with labels, emoji, and optional accent colours per column
- **Custom card fields** — choose which frontmatter attributes are shown on each card (`Fields`)
- **Short card names** — cards display only the file name (the part after the last `/` of the page name, e.g. `task/buy-milk` shows as `buy-milk`); links still open the full page
- **Snooze support** — pages whose `snoozeDate` frontmatter date lies in the future are hidden from the board; the ⏰ button toggles them into view
- **Per-card snooze** — every card carries a 💤 button that opens the system date picker; the picked date is written to the page's `snoozeDate` field (clearing the picker writes `none`)
- **Sorted columns** — cards are sorted within columns by a configurable frontmatter attribute (default `urgency`)
- **HideKeys** — display an attribute value without showing its label (handy for IDs or long text)
- **Mobile-friendly** — columns hold their minimum width and the board scrolls horizontally on narrow screens instead of squishing


### 🚫 Known Limitations

- Tags for the `Tags` filter must live in **frontmatter** (`tags: kanban, project`); hashtags in the page body are not considered
- Manual frontmatter edits to a page require a widget refresh to appear on the board
- Status values are matched case-insensitively; pages without a `status` start in the first column, while pages with a status that matches no configured column (e.g. `status: done` on a board without a done column) are not shown at all
- Status values are written as plain YAML scalars (`status: done`) and are only quoted when a plain scalar would be ambiguous

## Setup and Configuration

### Widget Parameters

*   **`query`**: (Required) A SilverBullet query returning pages, e.g. `query[[from index.tag "page"]]`.
*   **`options`**: (Required) A table to configure the board's behavior.
    *   **`Column`**: The frontmatter attribute to use for column assignment (e.g., `"status"`).
    *   **`Columns`**: An ordered list of columns, where each column is a `{status, title}` pair with an optional third element as accent colour (e.g., `{"todo", "📝 To Do", "purple"}`).
    *   **`Tags`**: (Optional) A list of tags. A page must carry **all** of them in its frontmatter to appear on this board.
    *   **`SortDefault`**: (Optional) The frontmatter attribute used to sort cards within columns. Defaults to `"urgency"`. Numeric values sort highest-first, everything else alphabetically; pages without the attribute sort last.
    *   **`Fields`**: (Optional) A list of frontmatter attributes to display on the card. E.g. `{"due", "priority"}`.
    *   **`HideKeys`**: (Optional) Hide certain attribute keys/labels from the card. This can be useful if you have a longer text or a title as attribute and want to display the whole thing.

### Snoozed pages

Pages with a `snoozeDate` frontmatter field are hidden from the board until that date is reached (they reappear on the morning of the snooze date). Set `snoozeDate: none` — or omit the field — for pages that should always be visible.

```yaml
---
status: open
tags: task, personal
snoozeDate: 2026-03-02
---
```

The ⏰ button in the board's top bar reveals snoozed pages temporarily; the toggle state is kept for the session and survives widget re-renders. Snoozed pages are excluded from the column counts while hidden, and when revealed they are rendered in gray regardless of their column's accent color.

Every card carries a 💤 button: clicking it opens the system date picker (pre-filled with the page's current `snoozeDate`); the picked date is written to the page's `snoozeDate` field and the board refreshes. To un-snooze, reveal snoozed pages via the ⏰ button, open the picker on the gray card and clear the date — this writes `snoozeDate: none`.

### Tag chips

All frontmatter tags found on the board's pages — except the ones required via the `Tags` option and tags carried by *every* page (those carry no distinguishing information, e.g. a `task` tag the query already enforces) — appear as `#tag` chips in the board's top bar. Chips are toggles: every chip is on by default and its pages are visible; click a chip to hide every page carrying that tag (a page with several tags disappears as soon as any of its tags is toggled off). Chip-hidden pages are excluded from the column counts. Like the snooze toggle, the chip state is kept for the session, survives widget re-renders and is shared by all boards of the session.

### Widget example

```lua
${KanbanBoard(
  query[[from index.tag "page"]], 
  {
    {"Column", "status"},
    {"Columns", {
      {"todo", "📝 To Do","purple"},
      {"doing", "⏳ In Progress","red"},
      {"review", "👀 Needs Review","yellow"},
      {"done", "✅ Done","green"}
    }},
    {"Tags", {"project"}},
    {"SortDefault", "urgency"},
    {"Fields", {"priority", "due"}},
    {"HideKeys", {"taskID"}}
  }
)}
```

### Example page

```yaml
---
status: doing
tags: project
priority: 3
due: 2026-03-02
taskID: P-17
---
# My page
```

# Implementation

## CSS Styling

```space-style
#sb-main .cm-editor .sb-lua-directive-block:has(.kanban-board) .button-bar { 
  top: -40px; 
  padding:0; 
  border-radius: 2em; 
  opacity:0.2; 
  transition: all 0.5s ease;
} 
#sb-main .cm-editor .sb-lua-directive-block:has(.kanban-board) .button-bar:hover { 
  opacity:1;
}


/* ---------------------------------
   Kanban Board Layout
---------------------------------- */

.kanban-board {
  display: flex;
  gap: 10px;
  padding: 10px;
  overflow-x: auto;
  align-items: normal;
  justify-content: flex-start;
}

.kanban-column {
  flex: 1;
  min-width: 250px;
  background: oklch(from var(--modal-help-background-color) l c h / 0.4);
  border-radius: 18px;
  padding: 10px;
  display: flex;
  flex-direction: column;
}

.kanban-column-title {
  font-weight: bold;
  margin-bottom: 10px;
  text-align: center;
}

.kanban-cards {
  min-height: 100px;
  display: flex;
  flex-direction: column;
  gap: 10px;
  flex-grow: 1;
}


/* ---------------------------------
   Kanban Cards
---------------------------------- */

.kanban-card {
  background: var(--modal-background-color);
  border: 1px solid var(--modal-border-color);
  box-shadow: 0 0 5px rgba(0 0 0 / 0.8);
  border-radius: 10px;
  padding: 10px;
  cursor: grab;
  position: relative;
  width: 100%; 
  box-sizing: border-box; 
}

.kanban-card-name {
  overflow: hidden; 
  text-overflow: ellipsis;
  white-space: nowrap; 
  font-weight: bold;
  margin-bottom: 4px;
  display: block;
  text-decoration-line: none;
}

.kanban-card-fields {
  margin-top: 8px;
  display: flex;
  flex-direction: column;
  font-size: 0.85em;
  opacity: 0.8;
}

.kanban-card-field {
  display: flex;
  gap: 4px;
}

.kanban-field-key {
  font-weight: bold;
  white-space: nowrap;
}


/* ---------------------------------
   Column Empty State (Gengar placeholder)
---------------------------------- */

.kanban-col-empty {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  flex-grow: 1;
  width: 100%;
  min-height: 100px;
  padding: 16px 8px;
  box-sizing: border-box;
  /* ink just a bit darker than the column background, in both themes */
  color: oklch(from var(--modal-help-background-color) calc(l - 0.1) c h);
  font-style: italic;
  text-align: center;
}

/* The placeholder is hidden while the column has at least one visible
   card — this reacts instantly to the snooze and tag-chip toggles, no
   re-render needed */
.kanban-cards:has(.kanban-card:not(.kanban-tag-hidden):not([data-snoozed="true"])) > .kanban-col-empty,
.kanban-board.show-snoozed .kanban-cards:has(.kanban-card:not(.kanban-tag-hidden)) > .kanban-col-empty {
  display: none;
}

.kanban-col-empty-art {
  width: 110px;
  height: auto;
  margin-bottom: 8px;
  /* pencil-style line art: strokes use currentColor, inheriting the
     placeholder's light ink color */
}

.kanban-col-empty-sub {
  font-size: 0.75em;
}


/* ---------------------------------
   Snooze Toggle
---------------------------------- */

.kanban-controls {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  justify-content: flex-end;
  gap: 6px;
  padding: 0 10px;
}

.kanban-tag-chips {
  display: flex;
  gap: 6px;
  flex-wrap: wrap;
  margin-right: auto; /* keeps the snooze button on the right */
}

.kanban-tag-chip {
  padding: 2px 10px;
  border-radius: 2em;
  border: 1px solid var(--modal-border-color);
  background: var(--modal-background-color);
  color: var(--text-muted);
  font-size: 0.8em;
  cursor: pointer;
  transition: all 0.2s;
  line-height: normal;
}

.kanban-tag-chip.active {
  background: var(--ui-accent-color);
  color: var(--modal-selected-option-color);
}

.kanban-tag-chip:hover {
  opacity: 0.8;
}

.kanban-snooze-toggle-btn {
  padding: 4px 10px;
  border-radius: 8px;
  border: 1px solid var(--modal-border-color);
  background: var(--modal-background-color);
  color: var(--text-muted);
  font-size: 0.85em;
  cursor: pointer;
  transition: all 0.2s;
  line-height: normal;
  flex-shrink: 0;
}

.kanban-snooze-toggle-btn:hover,
.kanban-snooze-toggle-btn.active {
  background: var(--ui-accent-color);
  color: var(--modal-selected-option-color);
}

.kanban-snooze-toggle-btn:active {
  opacity: 0.7;
}

/* Snoozed cards are hidden unless the board carries the show-snoozed class */
.kanban-board:not(.show-snoozed) .kanban-card[data-snoozed="true"] {
  display: none;
}

/* Pages carrying a toggled-off tag chip are hidden */
.kanban-card.kanban-tag-hidden {
  display: none;
}

/* When revealed, snoozed cards render gray, overriding any column accent color */
html[data-theme='dark'] .kanban-board .kanban-card[data-snoozed="true"] {
  background: oklch(0.35 0 0);
  border-color: oklch(0.45 0 0);
  box-shadow: none;
}

html[data-theme='light'] .kanban-board .kanban-card[data-snoozed="true"] {
  background: oklch(0.92 0 0);
  border-color: oklch(0.78 0 0);
  box-shadow: none;
}


/* ---------------------------------
   Per-Card Snooze Button
---------------------------------- */

.kanban-snooze-btn {
  position: absolute;
  bottom: 0;
  right: 5px;
  font-size: 14px;
  cursor: pointer;
  opacity: 0.4;
  padding: 4px 6px;
  z-index: 1;
  line-height: normal;
  transition: opacity 0.2s;
}

.kanban-snooze-btn:hover {
  opacity: 1;
}

/* Currently snoozed cards keep the button visible for un-snoozing */
.kanban-card[data-snoozed="true"] .kanban-snooze-btn {
  opacity: 0.8;
}

/* ---------------------------------
   Column Card Colors (optional accent color per column)
---------------------------------- */

html[data-theme='dark'] .kanban-column-colored .kanban-card {
  background: oklch(from var(--column-card-color) 0.3 0.2 h / 0.35);
  border-color: oklch(from var(--column-card-color) 0.55 0.2 h / 0.65);
  box-shadow: 0 0 5px oklch(from var(--column-card-color) 0.3 0.2 h / 0.5);
}

html[data-theme='light'] .kanban-column-colored .kanban-card {
  background: oklch(from var(--column-card-color) 0.95 0.1 h / 0.55);
  border-color: oklch(from var(--column-card-color) 0.6 0.15 h / 0.75);
  box-shadow: 0 0 5px oklch(from var(--column-card-color) 0.7 0.15 h / 0.5);
}

```

## Lua Implementation
```space-lua
-- ------------- Helper: Escape Magic Characters for Lua Patterns -------------
local function escapeLuaPattern(s)
    return s:gsub("([%^%$%(%)%%%.%[%]%*%+%-%?])", "%%%1")
end

-- ------------- Update a single key in a page's frontmatter -------------
function updatePageFrontmatter(pageName, key, value)
    local content = space.readPage(pageName)
    if not content then return end

    local val = tostring(value or "")

    -- Write the value as a plain YAML scalar; quote it only when a plain
    -- scalar would be ambiguous (empty, leading/trailing whitespace,
    -- ": ", trailing ":", " #", leading "#" or a YAML indicator character)
    local needsQuotes = val == ""
        or val:find("^%s") or val:find("%s$")
        or val:find(": ") or val:find(":$")
        or val:find(" #") or val:find("^#")
        or val:find("^[!&*%[%]{}|>%%%@\"'%-]")
    if needsQuotes then
        val = '"' .. val:gsub('"', '\\"') .. '"'
    end

    local newLine = key .. ": " .. val
    local sep = content:find("\r\n") and "\r\n" or "\n"

    -- Detect a frontmatter block (--- ... ---) at the very top of the page
    local linesStart, closeLineStart, closeLineEnd = nil, nil, nil
    local firstLineEnd = content:find("\n") or (#content + 1)
    local firstLine = content:sub(1, firstLineEnd - 1):gsub("\r$", "")
    if firstLine == "---" then
        local pos = firstLineEnd + 1
        while pos <= #content do
            local lineEnd = content:find("\n", pos) or (#content + 1)
            local line = content:sub(pos, lineEnd - 1):gsub("\r$", "")
            if line == "---" then
                linesStart = firstLineEnd + 1
                closeLineStart = pos
                closeLineEnd = lineEnd + 1
                break
            end
            if lineEnd > #content then break end
            pos = lineEnd + 1
        end
    end

    local newContent
    if linesStart then
        -- Replace the key's line in place, or insert it before the closing fence
        local fmLines = content:sub(linesStart, closeLineStart - 1)
        local keyPattern = "^" .. escapeLuaPattern(key) .. "%s*:"
        local found = false
        local out = {}
        for line in fmLines:gmatch("([^\r\n]*)\r?\n") do
            if line:match(keyPattern) then
                table.insert(out, newLine)
                found = true
            else
                table.insert(out, line)
            end
        end
        if not found then
            table.insert(out, newLine)
        end
        newContent = "---" .. sep .. table.concat(out, sep) .. sep ..
            "---" .. sep .. content:sub(closeLineEnd)
    else
        -- No frontmatter yet: create a block at the top of the page
        local gap = content:match("^\r?\n") and "" or sep
        newContent = "---" .. sep .. newLine .. sep .. "---" .. sep .. gap .. content
    end

    space.writePage(pageName, newContent)

    js.window.setTimeout(function()
        codeWidget.refreshAll()  -- Refresh widget after the frontmatter update
    end, 200)
end

-- ------------- Event Listeners -------------
if not js.window.kanbanListenersAdded then
    js.window.addEventListener("sb-kanban-dnd-update", function(e)
        if e.detail and e.detail.action == "move" then
            updatePageFrontmatter(
                e.detail.page,
                e.detail.statusKey,
                e.detail.newStatus
            )
        end
    end)

    js.window.addEventListener("sb-kanban-snooze-update", function(e)
        if e.detail and e.detail.action == "snooze" then
            updatePageFrontmatter(
                e.detail.page,
                "snoozeDate",
                e.detail.value
            )
        end
    end)
    js.window.kanbanListenersAdded = true
end

-- ------------- Main Kanban Board Function -------------
function KanbanBoard(pageQuery, options)
    -- Normalizes a tag: strips a leading '#'
    local function normalizeTag(t)
        local s = tostring(t):gsub("^#", "")
        return s
    end

    -- Returns a page's frontmatter tags as a normalized list
    local function pageTags(p)
        local result = {}
        local raw = p.tags
        if type(raw) == "string" then raw = { raw } end
        if type(raw) == "table" then
            for _, t in ipairs(raw) do
                local tag = normalizeTag(t)
                if tag ~= "" then
                    table.insert(result, tag)
                end
            end
        end
        return result
    end

    -- A page is snoozed while its snoozeDate frontmatter field (YYYY-MM-DD)
    -- lies in the future. A missing, empty or "none" value — or anything that
    -- does not parse as a date — means the page is not snoozed.
    local function isSnoozed(p)
        local sd = p.snoozeDate
        if sd == nil then return false end
        sd = tostring(sd)
        if sd == "" or sd:lower() == "none" then return false end
        local y = tonumber(sd:sub(1, 4))
        local m = tonumber(sd:sub(6, 7))
        local d = tonumber(sd:sub(9, 10))
        if y == nil or m == nil or d == nil then return false end
        local snoozeTime = os.time({ year = y, month = m, day = d, hour = 0, min = 0 })
        return snoozeTime >= os.time()
    end

    local statusKey = "status"
    local columnOrder = {}
    local columnTitles = {}
    local columnColors = {} -- optional accent color per column, set via 3rd element in Columns config
    local fields = {} -- for custom fields
    local hideKeys = {} -- field keys whose label is hidden on cards, set via {"HideKeys", {"key1","key2"}}
    local requiredTags = {} -- a page must carry all of these frontmatter tags, set via {"Tags", {"tag1","tag2"}}
    local sortDefault = "urgency" -- attribute used to sort cards within columns, set via {"SortDefault", "fieldname"}

    for _, opt in ipairs(options) do
        if opt[1] == "Column" then statusKey = opt[2] end
        if opt[1] == "Columns" then
          local cols = opt[2]
          for _, colData in ipairs(cols) do
            local status = tostring(colData[1]):gsub("^%s*(.-)%s*$", "%1")
                status = status:lower()
            table.insert(columnOrder, status)
            columnTitles[status] = colData[2]
            if colData[3] and colData[3] ~= "" then -- optional accent color
                columnColors[status] = tostring(colData[3])
            end
          end
        end
        if opt[1] == "Fields" then fields = opt[2] or {} end
        if opt[1] == "SortDefault" then sortDefault = opt[2] or "urgency" end
        if opt[1] == "HideKeys" then -- build a lookup set of keys whose label to hide on cards
            for _, k in ipairs(opt[2] or {}) do
                hideKeys[tostring(k)] = true
            end
        end
        if opt[1] == "Tags" then -- build a lookup set of required frontmatter tags
            for _, t in ipairs(opt[2] or {}) do
                local tag = normalizeTag(t)
                if tag ~= "" then
                    table.insert(requiredTags, tag)
                end
            end
        end
    end

    if #columnOrder == 0 then return widget.new{ display="block", html="<p>Error: No columns defined for Kanban board.</p>" } end

    -- Lookup set of the tags required via the Tags option; those never
    -- show up as toggleable chips
    local requiredTagSet = {}
    for _, t in ipairs(requiredTags) do
        requiredTagSet[t] = true
    end

    -- A page is shown only if it carries ALL required tags in its frontmatter
    local function hasRequiredTags(p)
        if #requiredTags == 0 then return true end
        local own = {}
        local raw = p.tags
        if type(raw) == "string" then raw = { raw } end
        if type(raw) == "table" then
            for _, t in ipairs(raw) do
                own[normalizeTag(t)] = true
            end
        end
        for _, req in ipairs(requiredTags) do
            if not own[req] then return false end
        end
        return true
    end

    local pagesByStatus = {}
    for _, status in ipairs(columnOrder) do
        pagesByStatus[status] = {}
    end

    for p in pageQuery do
        if hasRequiredTags(p) then
            local raw_status = p[statusKey]
            -- A missing or empty status means the page has not been started
            -- yet: it starts in the first column.
            local status
            if raw_status == nil or raw_status == "" then
                status = columnOrder[1]
            else
                status = tostring(raw_status):gsub("^%s*(.-)%s*$", "%1")
                status = status:lower()
            end

            -- A status that matches no configured column (e.g. "done" on a
            -- board without a done column) does not show up on the board.
            if pagesByStatus[status] then
                table.insert(pagesByStatus[status], p)
            end
        end
    end

    -- Sort each column by the configured attribute: numeric values sort
    -- highest first (e.g. urgency), everything else alphabetically;
    -- pages without the attribute sort last.
    local function hasSortValue(v)
        return v ~= nil and v ~= ""
    end

    local function comparePages(a, b)
        local av = a[sortDefault]
        local bv = b[sortDefault]
        local aMissing = not hasSortValue(av)
        local bMissing = not hasSortValue(bv)
        if aMissing or bMissing then
            return (not aMissing) and bMissing
        end
        local an = tonumber(tostring(av))
        local bn = tonumber(tostring(bv))
        if an ~= nil and bn ~= nil then
            return an > bn
        end
        local as = tostring(av)
        local bs = tostring(bv)
        return as < bs
    end

    for _, status in ipairs(columnOrder) do
        table.sort(pagesByStatus[status], comparePages)
    end

    -- Collect the board's toggleable tag chips: every frontmatter tag across
    -- the board's pages (snoozed/hidden pages included, so the chip set stays
    -- stable), except the tags required via the Tags option and tags carried
    -- by every page — those carry no distinguishing information (e.g. a
    -- "task" tag that the query already enforces on all pages)
    local totalPages = 0
    local tagPageCount = {}
    for _, status in ipairs(columnOrder) do
        for _, pg in ipairs(pagesByStatus[status]) do
            totalPages = totalPages + 1
            for _, tag in ipairs(pageTags(pg)) do
                tagPageCount[tag] = (tagPageCount[tag] or 0) + 1
            end
        end
    end

    local chipTagList = {}
    for tag, count in pairs(tagPageCount) do
        if not requiredTagSet[tag] and count < totalPages then
            table.insert(chipTagList, tag)
        end
    end
    table.sort(chipTagList)

    -- Chip toggle state persists in the browser window as a comma-separated
    -- list of toggled-off tags, so it survives widget re-renders
    local tagsOff = {}
    local tagsOffRaw = js.window._kanbanTagsOff
    if type(tagsOffRaw) == "string" then
        for tag in tagsOffRaw:gmatch("[^,%s]+") do
            tagsOff[tag] = true
        end
    end

    -- A page is chip-hidden when any of its tags is toggled off
    local function isTagHidden(p)
        for _, tag in ipairs(pageTags(p)) do
            if tagsOff[tag] then
                return true
            end
        end
        return false
    end

    -- The root wrapper carries the board config as data attributes; a single
    -- session-persistent delegated event engine (installed once, see jsCode
    -- below) operates on any board instance purely from the DOM. This keeps
    -- drag & drop working even when SilverBullet re-renders or replaces this
    -- widget's DOM without re-invoking this Lua function.
    local showSnoozed = (js.window._kanbanShowSnoozed == true)

    -- Snooze toggle button; its state persists in the browser window so it
    -- survives widget re-renders (e.g. after a drag & drop frontmatter update)
    local snoozeBtnClass = "kanban-snooze-toggle-btn"
    local snoozeBtnTitle = "Show snoozed tasks"
    if showSnoozed then
        snoozeBtnClass = snoozeBtnClass .. " active"
        snoozeBtnTitle = "Hide snoozed tasks"
    end

    -- Toggleable tag chips for the left side of the controls bar
    local chipsHtml = ""
    if #chipTagList > 0 then
        chipsHtml = '<span class="kanban-tag-chips">'
        for _, tag in ipairs(chipTagList) do
            local tagEsc = tag:gsub('"', '&quot;')
            tagEsc = tagEsc:gsub('<', '&lt;')
            tagEsc = tagEsc:gsub('>', '&gt;')
            local chipClass = "kanban-tag-chip"
            local chipTitle = "Hide pages tagged #" .. tag
            if tagsOff[tag] then
                chipTitle = "Show pages tagged #" .. tag
            else
                chipClass = chipClass .. " active"
            end
            chipsHtml = chipsHtml .. '<button class="' .. chipClass .. '" data-tag="' .. tagEsc .. '" title="' .. chipTitle .. '">#' .. tagEsc .. '</button>'
        end
        chipsHtml = chipsHtml .. '</span>'
    end

    local html = '<div data-kanban-root="true" data-status-key="' .. statusKey .. '">'
    html = html .. '<div class="kanban-controls">' .. chipsHtml .. '<button class="' .. snoozeBtnClass .. '" title="' .. snoozeBtnTitle .. '">⏰</button></div>'
    html = html .. '<div class="kanban-board' .. (showSnoozed and ' show-snoozed' or '') .. '">'

    for _, status in ipairs(columnOrder) do
        local title = columnTitles[status]
        local pages = pagesByStatus[status]

        -- Column count reflects what is visible: pages with a toggled-off
        -- tag chip, and snoozed pages while snoozes are hidden, don't count
        local visibleCount = 0
        for _, pg in ipairs(pages) do
            if not isTagHidden(pg) and not (isSnoozed(pg) and not showSnoozed) then
                visibleCount = visibleCount + 1
            end
        end

        -- Build optional color class and inline CSS variable for this column
        local colorAttr = ""
        if columnColors[status] then
            colorAttr = ' kanban-column-colored" style="--column-card-color: ' .. columnColors[status]
        end

        html = html .. '<div class="kanban-column' .. colorAttr .. '" data-status="' .. status .. '">'
        html = html .. '<div class="kanban-column-title">' .. title .. ' (<span class="kanban-col-count">' .. visibleCount .. '</span>)</div>'
        html = html .. '<div class="kanban-cards">'

        for _, p in ipairs(pages) do
            local pageName = tostring(p.name or "")
            -- On the card show only the file name (part after the last '/'),
            -- e.g. task/buy-milk is displayed as buy-milk
            local displayName = pageName:match("([^/]+)$") or pageName
            local pageNameEsc = pageName:gsub('"', '&quot;')
            pageNameEsc = pageNameEsc:gsub('<', '&lt;')
            pageNameEsc = pageNameEsc:gsub('>', '&gt;')
            local displayNameEsc = displayName:gsub('"', '&quot;')
            displayNameEsc = displayNameEsc:gsub('<', '&lt;')
            displayNameEsc = displayNameEsc:gsub('>', '&gt;')

            -- Tags for the toggleable chips in the top bar; a page whose
            -- chip-hidden state is on starts hidden until a chip toggles back
            local cardTags = table.concat(pageTags(p), ",")
            local cardTagsEsc = cardTags:gsub('"', '&quot;')
            cardTagsEsc = cardTagsEsc:gsub('<', '&lt;')
            cardTagsEsc = cardTagsEsc:gsub('>', '&gt;')
            local cardClass = "kanban-card"
            if isTagHidden(p) then
                cardClass = cardClass .. " kanban-tag-hidden"
            end

            -- Expose a parseable current snoozeDate so the per-card date
            -- picker can pre-fill it
            local snoozeDateAttr = ""
            local sd = p.snoozeDate
            if sd ~= nil and tostring(sd):find("^%d%d%d%d%-%d%d%-%d%d") then
                snoozeDateAttr = ' data-snooze-date="' .. tostring(sd) .. '"'
            end

            html = html .. '<div class="' .. cardClass .. '" draggable="true" data-page="' .. pageNameEsc .. '" data-tags="' .. cardTagsEsc .. '"' ..
                snoozeDateAttr ..
                (isSnoozed(p) and ' data-snoozed="true"' or '') .. '>'

            -- Clickable Title (link and tooltip keep the full page name)
            html = html .. '<a class="kanban-card-name" draggable="false" href="/' .. pageNameEsc .. '" data-ref="/' .. pageNameEsc .. '" title="' .. pageNameEsc .. '">' .. displayNameEsc .. '</a>'

            -- Per-card snooze button (opens the native date picker)
            html = html .. '<div class="kanban-snooze-btn" title="Snooze this task until...">💤</div>'

            -- Custom Fields
            if #fields > 0 then
                html = html .. '<div class="kanban-card-fields">'
                for _, fieldKey in ipairs(fields) do
                    local fieldValue = p[fieldKey]
                    if fieldValue then
                        -- Render key label only when it is not in the hideKeys lookup
                        local keyHtml = ""
                        if not hideKeys[fieldKey] then
                            keyHtml = '<span class="kanban-field-key">' .. fieldKey .. ': </span>'
                        end

                        local valueHtml = ""
                        if fieldKey == "tags" then
                            -- Render the tags field as SilverBullet hashtag links
                            local tagList = {}
                            if type(fieldValue) == "table" then
                                tagList = fieldValue
                            else
                                tagList = {tostring(fieldValue)}
                            end
                            local tagHtmlParts = {}
                            for _, tagText in ipairs(tagList) do
                                local t = tostring(tagText)
                                t = t:gsub("^#", "") -- strip leading # if already present
                                local tagLink = '<a class="sb-hashtag" href="/tag%3A' .. t .. '" rel="tag" data-tag-name="' .. t .. '"><span class="sb-hashtag-text">#' .. t .. '</span></a>'
                                table.insert(tagHtmlParts, tagLink)
                            end
                            valueHtml = table.concat(tagHtmlParts, " ")
                        else
                            if type(fieldValue) == "table" then fieldValue = table.concat(fieldValue, ", ") end
                            local rawVal = tostring(fieldValue)
                            -- Render any #hashtag tokens inside field values
                            -- as SilverBullet hashtag links. Non-hashtag segments
                            -- are wrapped in a plain <span>.
                            if rawVal:find("#[^%s%[%]%(%)%{%}%<>\"',;:!?@#\\/]+") then
                                local hashParts = {}
                                local scanPos = 1
                                while scanPos <= #rawVal do
                                    local s, e, tag = rawVal:find("#([^%s%[%]%(%)%{%}%<>\"',;:!?@#\\/]+)", scanPos)
                                    if s then
                                        if s > scanPos then
                                            table.insert(hashParts, '<span class="kanban-field-value">' .. rawVal:sub(scanPos, s - 1) .. '</span>')
                                        end
                                        local tagLink = '<a class="sb-hashtag" href="/tag%3A' .. tag .. '" rel="tag" data-tag-name="' .. tag .. '"><span class="sb-hashtag-text">#' .. tag .. '</span></a>'
                                        table.insert(hashParts, tagLink)
                                        scanPos = e + 1
                                    else
                                        table.insert(hashParts, '<span class="kanban-field-value">' .. rawVal:sub(scanPos) .. '</span>')
                                        break
                                    end
                                end
                                valueHtml = table.concat(hashParts, "")
                            else
                                valueHtml = '<span class="kanban-field-value">' .. rawVal .. '</span>'
                            end
                        end

                        html = html .. '<div class="kanban-card-field">' ..
                            keyHtml ..
                            valueHtml ..
                            '</div>'
                    end
                end
                html = html .. '</div>'
            end

            html = html .. '</div>' -- close kanban-card
        end

        -- Empty-state placeholder, only in the first (To Do) column; hidden
        -- via CSS while the column has at least one visible card (see the
        -- CSS section), so it reacts instantly to the snooze and chip
        -- toggles. All strokes use currentColor, i.e. the same muted color
        -- as the subtitle below.
        if status == columnOrder[1] then
            html = html .. '<div class="kanban-col-empty">'
            -- Gengar line art: vectorized paths, strokes in currentColor
            html = html .. '<svg class="kanban-col-empty-art" viewBox="104 122 528 513" xmlns="http://www.w3.org/2000/svg" fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round">'
            html = html .. '<path d="M498.56,413.763c-1.4,15.64-3.372,31.959-6.422,47.365" stroke-width="1.3" vector-effect="non-scaling-stroke"/>'
            html = html .. '<path d="M469.689,423.138c-1.706,17.622-4.827,35.76-8.935,52.988" stroke-width="1.3" vector-effect="non-scaling-stroke"/>'
            html = html .. '<path d="M432.407,425.654c-1.125,17.527-3.378,35.396-6.981,52.579" stroke-width="1.3" vector-effect="non-scaling-stroke"/>'
            html = html .. '<path d="M390.942,420.265c-0.681,17.222-2.825,34.571-6.372,51.438" stroke-width="1.3" vector-effect="non-scaling-stroke"/>'
            html = html .. '<path d="M352.347,406.053c-0.522,15.186-2.849,30.436-6.309,45.22" stroke-width="1.3" vector-effect="non-scaling-stroke"/>'
            html = html .. '<path d="M542.157,443.76c3.218,3.043,7.029,5.574,10.77,7.934c8.157,5.013,17.312,10.499,24.601,16.648 c2.065,1.771,4.041,3.738,5.438,6.091c1.158,1.926,1.758,4.143,2.294,6.308c0.42,1.627,0.783,3.597,1.559,5.092 c0.358,0.706,0.793,1.403,1.42,1.9c0.343,0.269,0.769,0.459,1.211,0.438c0.518-0.021,0.982-0.305,1.357-0.645 c0.697-0.643,1.193-1.475,1.633-2.307c1.346-2.631,2.221-5.841,3.857-8.32c0.427-0.627,0.924-1.238,1.582-1.632 c0.636-0.362,1.341-0.628,2.073-0.695c0.617-0.059,1.244,0.038,1.823,0.254c0.813,0.289,1.649,0.889,2.375,1.353 c1.062,0.659,2.156,1.358,3.379,1.672c0.461,0.109,0.966,0.136,1.407-0.062c0.421-0.185,0.732-0.551,0.958-0.943 c0.409-0.721,0.625-1.54,0.82-2.34c0.443-1.864,0.703-4.138,0.974-6.045c0.029-0.229,0.076-0.472,0.131-0.697 c0.122-0.467,0.307-0.958,0.703-1.261c0.268-0.207,0.621-0.271,0.953-0.245c0.649,0.054,1.268,0.316,1.873,0.541 c0.832,0.311,2.02,0.902,2.889,1.039c0.154,0.018,0.315,0.017,0.461-0.037c0.163-0.058,0.287-0.19,0.37-0.339 c0.242-0.461,0.325-1.054,0.443-1.559c0.491-2.311,0.111-4.701-0.456-6.963c-0.687-2.712-1.723-6.414-2.135-9.1 c-1.726-11.293-3.412-26.315-6.17-37.261c-1.453-5.938-3.421-11.85-5.721-17.513c-3.963-9.764-9.03-19.267-14.636-28.188 c-4.714-7.451-9.918-14.816-15.934-21.277c-10.61-11.453-23.577-20.587-36.817-28.744" stroke-width="1.8" vector-effect="non-scaling-stroke"/>'
            html = html .. '<path d="M311.712,386.369c2.837,10.734,6.196,21.603,10.642,31.778c3.587,8.198,7.96,16.188,13.638,23.134 c2.772,3.391,5.912,6.57,9.283,9.367c3.161,2.641,6.684,5.049,10.174,7.238c6.591,4.136,13.586,7.85,20.797,10.784 c7.373,3.009,15.163,5.27,22.981,6.773c10.106,1.937,20.515,2.763,30.796,2.892c10.039,0.09,20.221-0.454,30.131-2.106 c5.526-0.944,11.078-2.289,16.255-4.467c3.753-1.57,7.339-3.613,10.564-6.095c3.47-2.651,6.565-5.831,9.42-9.126 c3.381-3.907,6.649-8.087,9.335-12.506c3.656-5.95,6.588-12.472,9.098-18.981c3.354-8.69,6.135-18.107,8.583-27.104 c-12.929,9.816-27.159,18.497-42.923,22.817c-5.885,1.648-12.117,2.691-18.185,3.42c-7.337,0.885-15.15,1.428-22.541,1.516 c-8.81,0.106-18.014-0.417-26.765-1.44c-7.611-0.876-15.387-2.192-22.786-4.191c-10.28-2.746-20.494-6.622-30.297-10.741 C344.047,402.621,327.056,394.231,311.712,386.369z" stroke-width="1.8" vector-effect="non-scaling-stroke"/>'
            html = html .. '<path d="M235.062,482.81c10.231,33.698,25.922,70.759,55.643,91.606c10.597,7.52,22.822,12.491,35.176,16.319 c11.07,3.45,22.6,6,34.123,7.314c10.384,1.191,21.003,1.401,31.429,0.685c21.753-1.498,43.707-6.6,62.633-17.727 c7.06-4.169,13.596-9.326,19.491-15.016c11.393-11.082,21.277-23.932,30.234-37.033c13.286-19.592,24.429-40.971,32.052-63.408 c6.206-18.178,10.255-37.362,12.067-56.481c1.464-15.56,1.486-31.484-0.316-47.016c-1.384-12.01-3.748-24.209-7.048-35.841 c-1.177-4.047-2.591-8.612-4.023-12.582c-1.512-4.473-5.564-14.348-7.16-18.582c-1.7-4.532-3.26-9.291-3.102-14.189 c0.047-1.839,0.311-3.685,0.713-5.478c0.616-2.741,1.561-5.464,2.571-8.084c1.38-3.995,16.522-36.843,18.553-41.468 c5.757-13.327,12.428-29.359,17.556-42.912c2.393-6.424,4.873-13.294,6.718-19.889c0.369-1.372,0.746-2.83,0.947-4.234 c0.095-0.683,0.157-1.384,0.101-2.074c-0.063-0.879-0.412-1.858-1.261-2.254c-0.469-0.221-1.004-0.248-1.515-0.212 c-1.028,0.084-2.032,0.398-3.001,0.737c-5.884,2.229-11.703,5.778-17.188,8.866c-21.38,12.58-61.768,37.408-83.221,48.577 c-2.695,1.357-5.529,2.823-8.387,3.787c-1.014,0.32-2.078,0.619-3.151,0.539c-0.566-0.044-1.126-0.25-1.553-0.631 c-0.388-0.339-0.672-0.781-0.91-1.234c-1.591-3.413-4.671-12.712-6.038-16.354c-0.62-1.594-1.214-3.333-2.125-4.784 c-0.199-0.305-0.43-0.599-0.674-0.869c-0.054-0.058-0.113-0.115-0.179-0.16l-0.01-0.006l-0.01-0.006l-0.01-0.006l-0.01-0.006 l-0.01-0.006l-0.01-0.005l-0.01-0.005l-0.01-0.005l-0.011-0.005l-0.011-0.004l-0.011-0.004l-0.011-0.004l-0.011-0.004 l-0.011-0.003l-0.011-0.003l-0.011-0.003l-0.011-0.003l-0.012-0.002l-0.012-0.002l-0.012-0.002l-0.012-0.001l-0.012-0.001 l-0.012-0.001l-0.012,0l-0.013,0l-0.013,0l-0.013,0.001l-0.013,0.001l-0.013,0.001l-0.013,0.002l-0.014,0.002l-0.014,0.003 l-0.014,0.003l-0.014,0.003l-0.014,0.004l-0.014,0.004l-0.015,0.004l-0.015,0.005l-0.015,0.005l-0.015,0.006l-0.015,0.006 l-0.016,0.007l-0.016,0.007l-0.016,0.008l-0.016,0.008c-0.481,0.263-0.884,0.809-1.216,1.24c-0.69,0.936-1.329,2.007-1.918,3.011 c-1.961,3.404-3.956,7.377-5.715,10.899" stroke-width="1.8" vector-effect="non-scaling-stroke"/>'
            html = html .. '<path d="M412.525,207.237c0.997-4.978,2.178-10.059,3.992-14.804c0.64-1.673,1.406-3.357,2.238-4.944c0.62-1.137,2.571-4.965,3.239-5.987 c0.313-0.472,0.648-0.993,1.145-1.288c0.146-0.082,0.312-0.134,0.48-0.129c0.17,0.003,0.335,0.062,0.48,0.148 c0.26,0.155,0.47,0.381,0.659,0.614c0.414,0.522,0.743,1.122,1.051,1.712c0.569,1.099,1.075,2.308,1.526,3.463 c3.454,9.22,8.334,24.66,11.476,34.113" stroke-width="1.8" vector-effect="non-scaling-stroke"/>'
            html = html .. '<path d="M368.372,200.573c-1.072-12.362-1.219-25.07-0.681-37.466c0.169-3.496,0.398-7.288,0.796-10.762 c0.197-1.613,0.424-3.274,0.846-4.845c0.254-0.902,0.569-1.826,1.179-2.554c0.423-0.486,0.962-0.897,1.581-1.092 c0.455-0.141,0.951-0.125,1.402,0.026c0.82,0.276,1.485,0.875,2.063,1.501c0.666,0.731,1.239,1.571,1.777,2.399 c0.482,0.742,1.066,1.731,1.518,2.496c5.459,9.84,23.176,39.106,29.197,49.2l10.804,18.935" stroke-width="1.8" vector-effect="non-scaling-stroke"/>'
            html = html .. '<path d="M342.73,204.714c-0.582-3.094-1.352-6.406-2.187-9.439c-0.656-2.469-2.303-7.571-3.014-10.059 c-0.335-1.243-0.668-2.524-0.703-3.816c0-0.48,0.028-1.002,0.307-1.41c0.156-0.225,0.409-0.361,0.679-0.391 c0.582-0.062,1.141,0.197,1.651,0.449c2.633,1.43,6.781,4.922,9.231,6.724c10.965,8.218,28.116,19.377,39.666,27.079" stroke-width="1.8" vector-effect="non-scaling-stroke"/>'
            html = html .. '<path d="M308.516,199.257c0.147-2.567,0.322-5.298,0.775-7.825c0.178-0.944,0.389-1.902,0.771-2.787c0.17-0.386,0.382-0.761,0.669-1.074 c0.309-0.33,0.71-0.618,1.123-0.803c0.68-0.302,1.449-0.13,2.115,0.124c0.677,0.257,1.334,0.614,1.962,0.974 c0.501,0.243,11.368,7.246,12.11,7.706l44.413,27.297" stroke-width="1.8" vector-effect="non-scaling-stroke"/>'
            html = html .. '<path d="M228.046,301.291c-1.598-9.156-1.802-18.578-0.9-27.82c0.277-2.642,0.616-5.518,1.114-8.124c0.116-0.551,0.256-1.113,0.457-1.638 c0.193-0.505,0.452-0.994,0.81-1.402c0.33-0.38,0.746-0.684,1.2-0.899c1.458-0.671,3.186-0.653,4.761-0.592 c5.551,0.366,11.381,1.753,16.83,2.875" stroke-width="1.8" vector-effect="non-scaling-stroke"/>'
            html = html .. '<path d="M266.496,344.952c-2.384,19.483-8.551,38.758-19.31,55.263c-10.767,16.437-25.914,29.75-42.684,39.832 c-11.044,6.789-27.031,13.475-38.817,19.063c-3.563,1.659-8.088,4.281-11.554,6.159c-0.859,0.435-1.752,0.908-2.694,1.138 c-0.24,0.053-0.492,0.09-0.736,0.051l-0.018-0.003l-0.018-0.004l-0.018-0.004l-0.018-0.004l-0.017-0.004l-0.017-0.005 l-0.017-0.005l-0.017-0.005c-0.258-0.08-0.425-0.298-0.511-0.546c-0.146-0.427-0.146-0.893-0.146-1.34 c0.143-3.016,0.475-6.586,0.544-9.601c0.042-1.78,0.081-3.662-0.106-5.433c-0.076-0.573-0.142-1.182-0.412-1.701l-0.011-0.02 c-0.087-0.151-0.221-0.288-0.397-0.324c-0.125-0.028-0.255-0.011-0.377,0.02c-0.266,0.07-0.517,0.201-0.761,0.327 c-2.954,1.662-6.148,3.172-9.265,4.5c-2.031,0.841-4.139,1.699-6.272,2.237c-0.654,0.149-1.335,0.306-2.009,0.232 c-0.229-0.03-0.46-0.099-0.644-0.244c-0.163-0.126-0.276-0.306-0.344-0.498c-0.108-0.309-0.129-0.642-0.125-0.967 c0.014-0.63,0.135-1.264,0.275-1.877c1.119-4.421,3.404-9.35,4.802-13.691c0.22-0.699,0.451-1.432,0.57-2.156 c0.028-0.186,0.051-0.378,0.038-0.565c-0.009-0.123-0.035-0.253-0.118-0.349c-0.066-0.076-0.163-0.117-0.261-0.135 c-0.228-0.04-0.463-0.01-0.69,0.024c-0.656,0.11-1.318,0.333-1.949,0.543c-4.331,1.515-9.643,4.03-14.068,5.194 c-0.594,0.133-1.328,0.311-1.927,0.201l-0.02-0.005l-0.019-0.005l-0.019-0.005l-0.018-0.005l-0.018-0.006l-0.018-0.006 l-0.017-0.006l-0.017-0.007l-0.016-0.007l-0.016-0.007l-0.015-0.007l-0.015-0.008l-0.014-0.008l-0.014-0.008l-0.013-0.009 c-0.015-0.01-0.031-0.022-0.044-0.033l-0.006-0.005l-0.006-0.005l-0.006-0.005l-0.005-0.005l-0.005-0.005l-0.005-0.005 l-0.005-0.006l-0.005-0.006l-0.005-0.006l-0.005-0.006l-0.005-0.006l-0.005-0.006l-0.004-0.006l-0.004-0.006l-0.004-0.006 l-0.004-0.006l-0.004-0.006l-0.004-0.006l-0.004-0.006c-0.071-0.124-0.076-0.273-0.061-0.411c0.107-0.742,0.765-1.872,1.072-2.572 c1.022-2.23,2.235-5.836,3.168-8.118c0.854-2.136,1.863-4.319,2.917-6.363c3.156-6.116,8.646-15.264,11.945-21.338 c2.43-4.351,7.545-14.862,9.834-19.429c5.261-10.414,11.73-20.346,19.684-28.909c21.395-22.885,50.98-41.853,75.871-60.783 c3.883-3.091,7.846-6.318,10.707-10.418c1.323-1.908,2.404-4.028,2.911-6.305c0.434-1.891,0.419-3.87,0.062-5.772 c-0.388-2.115-1.12-4.183-1.915-6.177c-0.769-1.934-2.258-5.166-3.155-7.1c-10.192-21.346-27.58-57.411-36.534-78.861 c-4.238-10.074-8.981-21.574-12.522-31.893c-0.82-2.451-1.68-5.079-2.252-7.602c-0.169-0.82-0.348-1.674-0.271-2.515 c0.027-0.247,0.085-0.497,0.212-0.713c0.111-0.19,0.285-0.344,0.49-0.428l0.021-0.009l0.022-0.008l0.022-0.008l0.022-0.008 l0.023-0.007c0.475-0.142,0.983-0.068,1.459,0.027c1.575,0.361,3.068,1.095,4.516,1.797 c24.852,13.367,87.222,52.864,112.199,67.992c6,3.579,13.519,8.032,19.712,11.252c5.924,3.158,12.823,5.904,19.063,8.388" stroke-width="1.8" vector-effect="non-scaling-stroke"/>'
            html = html .. '<path d="M456.147,387.502c2.652,3.255,5.502,6.482,8.674,9.238c2.313,1.999,4.866,3.805,7.72,4.939c2.181,0.881,4.537,1.282,6.876,1.425 c2.109,0.134,4.25,0.053,6.332-0.32c5.339-0.931,10.205-3.866,13.879-7.803c4.4-4.727,7.203-10.73,9.132-16.838 c4.494-14.255,1.515-29.488-3.666-43.141L456.147,387.502z" stroke-width="1.8" vector-effect="non-scaling-stroke"/>'
            html = html .. '<path d="M354.571,328.935c-9.498,15.664-12.317,33.198-3.064,49.696c2.281,4.049,5.235,7.742,8.729,10.808 c4.36,3.829,9.565,6.732,15.152,8.321c4.585,1.31,9.449,1.613,14.166,0.929c9.817-1.425,18.845-6.439,26.139-13.04 L354.571,328.935z" stroke-width="1.8" vector-effect="non-scaling-stroke"/>'
            html = html .. '<path d="M309.011,584.583c-0.993,7.4-2.341,14.916-5.071,21.891c-1.252,3.144-2.838,6.216-5.01,8.828 c-1.623,1.952-3.601,3.628-5.805,4.889c-3.938,2.285-8.656,2.37-13.088,2.459c-10.101-0.045-51.119,0.153-60.388-0.091 c-1.595-0.06-3.329-0.141-4.909-0.331c-1.326-0.177-2.775-0.377-3.953-1.043c-0.584-0.338-1.066-0.866-1.181-1.549 c-0.087-0.482-0.007-0.979,0.136-1.442c0.253-0.8,0.685-1.536,1.144-2.233c2.459-3.585,6.722-7.643,9.694-10.856 c0.82-0.895,1.679-1.83,2.392-2.81c0.504-0.693,0.955-1.441,1.233-2.255c0.352-1.007,0.402-2.094,0.313-3.15 c-0.126-1.5-0.494-3.032-0.86-4.491c-2.731-9.903-7.119-23.64-9.422-33.57c-4.639-18.673-6.506-38.064-6.641-57.277l0.165-18.032 c0.012-14.207-2.188-28.601-4.846-42.534" stroke-width="1.8" vector-effect="non-scaling-stroke"/>'
            html = html .. '<path d="M440.268,587.885c8.282,6.994,16.91,14.308,25.958,20.279c3.007,2.01,6.61,3.842,9.815,5.536 c3.202,1.747,6.978,4.037,10.276,5.559c2.121,0.985,4.357,1.801,6.644,2.302c3.867,0.861,7.911,1.023,11.862,1.089 c4.197-0.048,58.627,0.148,60.956-0.079c1.271-0.07,2.576-0.176,3.826-0.423c1.381-0.271,2.779-0.747,3.835-1.712 c0.589-0.531,1.056-1.194,1.362-1.925c0.346-0.822,0.493-1.721,0.481-2.611c-0.011-2.024-0.909-3.917-2.059-5.541 c-0.803-1.144-1.741-2.229-2.691-3.253c-2.729-2.93-8.038-7.755-10.908-10.595c-1.786-1.811-3.739-3.651-4.71-6.054 c-0.636-1.576-0.814-3.3-0.881-4.986c-0.129-4.068,0.351-11.546,0.356-15.623c0.045-4.034-0.098-8.209-0.568-12.216 c-1.084-9.314-3.777-18.452-7.709-26.957c-2.514-5.359-5.644-10.52-8.918-15.445c-4.27-6.404-9.334-13.157-14.061-19.238" stroke-width="1.8" vector-effect="non-scaling-stroke"/>'
            html = html .. '<path d="M537.64,316.86c13.24,8.157,26.206,17.29,36.817,28.744c6.015,6.461,11.22,13.826,15.934,21.277 c5.606,8.922,10.672,18.422,14.636,28.188c2.301,5.662,4.269,11.575,5.722,17.513c2.754,10.906,4.468,26.05,6.17,37.261 c0.431,2.776,1.453,6.396,2.135,9.1c0.567,2.262,0.947,4.652,0.456,6.963c-0.118,0.506-0.2,1.098-0.443,1.559 c-0.084,0.15-0.211,0.284-0.377,0.341c-0.151,0.054-0.316,0.053-0.474,0.033c-0.86-0.14-2.05-0.731-2.871-1.037 c-0.623-0.23-1.259-0.503-1.929-0.544c-0.31-0.016-0.634,0.051-0.885,0.24c-0.39,0.291-0.578,0.767-0.702,1.222 c-0.063,0.24-0.111,0.497-0.144,0.744c-0.278,1.914-0.53,4.176-0.974,6.045c-0.199,0.817-0.42,1.654-0.847,2.385 c-0.229,0.385-0.546,0.741-0.969,0.914c-0.451,0.188-0.959,0.149-1.423,0.033c-1.2-0.323-2.279-1.009-3.325-1.659 c-0.731-0.464-1.56-1.063-2.375-1.353c-0.579-0.216-1.206-0.313-1.823-0.254c-0.743,0.068-1.457,0.34-2.1,0.711 c-0.645,0.395-1.135,0.997-1.556,1.616c-1.58,2.393-2.468,5.509-3.729,8.069c-0.475,0.932-1.009,1.873-1.79,2.584 c-0.382,0.339-0.858,0.618-1.382,0.62c-0.456,0.005-0.886-0.21-1.23-0.498c-0.587-0.495-1.003-1.163-1.346-1.841 c-0.775-1.492-1.147-3.466-1.559-5.092c-0.536-2.166-1.137-4.382-2.294-6.308c-1.396-2.353-3.372-4.32-5.437-6.091 c-7.276-6.137-16.47-11.654-24.601-16.648c-3.737-2.358-7.551-4.891-10.77-7.934c-4.307,17.997-10.787,35.663-19.025,52.231 c5.097,6.569,10.456,13.71,14.998,20.659c3.118,4.783,6.075,9.797,8.432,15.001c3.25,7.269,5.61,14.989,6.838,22.86 c0.952,5.99,1.107,12.206,0.933,18.266c-0.066,3.518-0.536,10.767-0.206,14.169c0.114,1.194,0.335,2.395,0.786,3.51 c0.439,1.101,1.106,2.102,1.857,3.013c4.038,4.659,10.226,9.685,14.346,14.277c1.455,1.646,2.888,3.408,3.664,5.488 c0.469,1.279,0.634,2.685,0.389,4.029c-0.16,0.875-0.51,1.721-1.05,2.43c-0.552,0.731-1.297,1.305-2.12,1.699 c-0.966,0.462-2.027,0.706-3.08,0.875c-1.928,0.3-3.96,0.349-5.909,0.387c-9.163-0.131-56.364,0.354-64.443-0.277 c-2.594-0.218-5.216-0.616-7.711-1.369c-5.564-1.676-10.714-5.16-15.822-7.852c-1.572-0.826-4.231-2.187-5.732-3.092 c-1.965-1.152-3.988-2.477-5.861-3.777c-7.927-5.555-15.787-12.14-23.173-18.405c-14.871,6.193-30.881,9.455-46.905,10.71 c-14.975,1.197-30.195,0.454-44.971-2.274c-13.435-2.518-26.791-6.403-39.381-11.737c-0.993,7.4-2.341,14.916-5.071,21.891 c-1.252,3.144-2.838,6.216-5.01,8.828c-1.623,1.952-3.601,3.628-5.805,4.889c-3.938,2.285-8.656,2.37-13.088,2.459 c-10.101-0.045-51.119,0.153-60.388-0.091c-1.595-0.06-3.329-0.141-4.909-0.331c-1.326-0.177-2.775-0.377-3.953-1.043 c-0.584-0.338-1.066-0.866-1.181-1.549c-0.087-0.482-0.007-0.979,0.136-1.442c0.253-0.8,0.685-1.536,1.144-2.233 c2.459-3.585,6.722-7.643,9.694-10.856c0.82-0.895,1.679-1.83,2.392-2.81c0.504-0.693,0.955-1.441,1.233-2.255 c0.352-1.007,0.402-2.094,0.313-3.15c-0.126-1.5-0.494-3.032-0.86-4.491c-2.731-9.903-7.119-23.64-9.422-33.57 c-4.639-18.673-6.506-38.064-6.641-57.277l0.165-18.032c0.012-14.207-2.188-28.601-4.846-42.534 c-10.427,6.23-26.16,12.879-37.233,18.126c-3.524,1.638-8.116,4.296-11.554,6.159c-0.847,0.43-1.717,0.887-2.641,1.126 c-0.257,0.06-0.526,0.104-0.789,0.063l-0.018-0.003l-0.018-0.004l-0.018-0.004l-0.018-0.004l-0.017-0.004l-0.017-0.005 l-0.017-0.005l-0.017-0.005c-0.258-0.08-0.425-0.298-0.511-0.546c-0.146-0.427-0.146-0.893-0.146-1.34 c0.143-3.016,0.475-6.586,0.544-9.601c0.042-1.78,0.081-3.662-0.106-5.433c-0.076-0.573-0.142-1.182-0.412-1.701l-0.011-0.02 c-0.087-0.151-0.221-0.288-0.397-0.324c-0.125-0.028-0.255-0.011-0.377,0.02c-0.266,0.07-0.517,0.201-0.761,0.327 c-2.954,1.662-6.148,3.172-9.265,4.5c-2.031,0.841-4.139,1.699-6.272,2.237c-0.654,0.149-1.335,0.306-2.009,0.232 c-0.229-0.03-0.46-0.099-0.644-0.244c-0.163-0.126-0.276-0.306-0.344-0.498c-0.108-0.309-0.129-0.642-0.125-0.967 c0.014-0.63,0.135-1.264,0.275-1.877c1.119-4.421,3.404-9.35,4.802-13.691c0.22-0.699,0.451-1.432,0.57-2.156 c0.028-0.186,0.051-0.378,0.038-0.565c-0.009-0.123-0.035-0.253-0.118-0.349c-0.066-0.076-0.163-0.117-0.261-0.135 c-0.228-0.04-0.463-0.01-0.69,0.024c-0.656,0.11-1.318,0.333-1.949,0.543c-4.331,1.515-9.643,4.03-14.068,5.194 c-0.594,0.133-1.328,0.311-1.927,0.201l-0.02-0.005l-0.019-0.005l-0.019-0.005l-0.018-0.005l-0.018-0.006l-0.018-0.006 l-0.017-0.006l-0.017-0.007l-0.016-0.007l-0.016-0.007l-0.015-0.007l-0.015-0.008l-0.014-0.008l-0.014-0.008l-0.013-0.009 c-0.015-0.01-0.031-0.022-0.044-0.033l-0.006-0.005l-0.006-0.005l-0.006-0.005l-0.005-0.005l-0.005-0.005l-0.005-0.005 l-0.005-0.006l-0.005-0.006l-0.005-0.006l-0.005-0.006l-0.005-0.006l-0.005-0.006l-0.004-0.006l-0.004-0.006l-0.004-0.006 l-0.004-0.006l-0.004-0.006l-0.004-0.006l-0.004-0.006c-0.071-0.124-0.076-0.273-0.061-0.411c0.107-0.742,0.765-1.872,1.072-2.572 c1.022-2.23,2.235-5.836,3.168-8.118c0.854-2.136,1.863-4.319,2.917-6.363c3.156-6.116,8.646-15.264,11.945-21.338 c2.43-4.351,7.545-14.862,9.834-19.429c2.615-5.101,5.522-10.258,8.722-15.012c3.456-5.151,7.434-10.135,11.672-14.663 c6.484-6.926,13.88-13.442,21.187-19.496c12.528-10.377,28.317-21.735,41.575-31.318c-1.598-9.156-1.802-18.578-0.9-27.82 c0.277-2.642,0.616-5.518,1.114-8.124c0.116-0.551,0.256-1.113,0.457-1.638c0.193-0.505,0.452-0.994,0.81-1.402 c0.33-0.38,0.746-0.684,1.2-0.899c1.458-0.671,3.186-0.653,4.761-0.592c5.551,0.366,11.381,1.753,16.83,2.875 c-0.671-1.701-1.564-3.688-2.329-5.359c-16.619-35.226-37.799-77.539-50.424-114.063c-0.561-1.717-1.16-3.632-1.601-5.381 c-0.191-0.777-0.384-1.6-0.472-2.396c-0.049-0.508-0.081-1.04,0.087-1.53c0.101-0.292,0.306-0.54,0.596-0.658l0.021-0.009 l0.022-0.008l0.022-0.008l0.022-0.008l0.023-0.007c0.377-0.116,0.782-0.086,1.167-0.027c1.18,0.204,2.309,0.695,3.399,1.178 c1.511,0.688,3.162,1.568,4.621,2.366c22.498,12.517,78.364,47.593,101.043,61.477c0.147-2.567,0.322-5.298,0.775-7.825 c0.178-0.944,0.389-1.902,0.771-2.787c0.185-0.42,0.419-0.826,0.744-1.153c0.341-0.333,0.801-0.658,1.258-0.8 c0.536-0.166,1.111-0.066,1.633,0.105c1.165,0.396,2.239,1.064,3.283,1.705c3.907,2.594,21.436,13.51,25.75,16.213 c-0.669-3.561-1.563-7.293-2.562-10.773c-0.55-1.97-2.212-7.13-2.741-9.096c-0.307-1.197-0.617-2.434-0.6-3.677 c0.018-0.565,0.138-1.29,0.74-1.51c0.496-0.171,1.029,0.005,1.492,0.2c0.898,0.398,1.736,0.982,2.542,1.539 c2.18,1.544,5.623,4.304,7.776,5.889c5.187,3.89,13.573,9.583,18.995,13.288c-1.009-11.572-1.187-23.507-0.779-35.116 c0.25-5.418,0.45-11.041,1.383-16.388c0.264-1.288,0.547-2.63,1.261-3.754c0.354-0.54,0.852-0.986,1.424-1.285 c0.511-0.268,1.118-0.349,1.676-0.196c0.755,0.201,1.389,0.706,1.935,1.247c1.003,1.015,1.783,2.248,2.537,3.452 c9.675,17.04,24.996,41.64,34.716,58.704c1.31-6.482,2.861-13.082,5.855-19.016c0.705-1.347,2.649-5.102,3.413-6.394 c0.229-0.377,0.474-0.762,0.76-1.099c0.29-0.337,0.684-0.701,1.163-0.634c0.264,0.037,0.495,0.194,0.688,0.37 c0.709,0.682,1.197,1.69,1.639,2.562c0.497,1.008,0.964,2.148,1.37,3.198c2.198,5.801,5.399,15.702,7.391,21.651 c1.761-3.527,3.753-7.491,5.715-10.899c0.589-1.005,1.228-2.075,1.918-3.011c0.332-0.432,0.734-0.977,1.216-1.24l0.016-0.008 l0.016-0.008l0.016-0.007l0.016-0.007l0.015-0.006l0.015-0.006l0.015-0.005l0.015-0.005l0.015-0.004l0.014-0.004l0.014-0.004 l0.014-0.003l0.014-0.003l0.014-0.003l0.014-0.002l0.013-0.002l0.013-0.001l0.013-0.001l0.013-0.001l0.013,0l0.013,0l0.012,0 l0.012,0.001l0.012,0.001l0.012,0.001l0.012,0.002l0.012,0.002l0.012,0.002l0.011,0.003l0.011,0.003l0.011,0.003l0.011,0.003 l0.011,0.004l0.011,0.004l0.011,0.004l0.011,0.004l0.011,0.005l0.01,0.005l0.01,0.005l0.01,0.005l0.01,0.006l0.01,0.006 l0.01,0.006l0.01,0.006l0.01,0.006c0.066,0.044,0.125,0.101,0.179,0.16c0.244,0.271,0.475,0.564,0.674,0.869 c0.911,1.449,1.506,3.191,2.125,4.784c1.389,3.705,4.431,12.898,6.038,16.354c0.238,0.453,0.522,0.894,0.91,1.234 c0.426,0.381,0.986,0.587,1.553,0.631c1.073,0.081,2.137-0.218,3.151-0.539c2.856-0.962,5.695-2.431,8.387-3.787 c21.278-11.068,62.101-36.151,83.221-48.577c5.493-3.092,11.298-6.634,17.188-8.866c0.992-0.345,2.02-0.669,3.073-0.741 c0.487-0.028,0.996,0.006,1.443,0.217c0.849,0.397,1.197,1.375,1.261,2.254c0.056,0.689-0.006,1.39-0.101,2.074 c-0.201,1.404-0.578,2.864-0.947,4.234c-1.844,6.592-4.326,13.469-6.718,19.889c-5.123,13.541-11.805,29.604-17.556,42.912 c-0.937,2.123-17.167,37.713-17.481,38.797c-0.997,2.427-2.007,5.017-2.776,7.52c-0.775,2.526-1.373,5.144-1.539,7.786 c-0.14,2.134,0.02,4.296,0.423,6.395c0.449,2.369,1.203,4.741,2.011,7.012C531.463,300.707,535.188,309.417,537.64,316.86z" stroke-width="2.5" vector-effect="non-scaling-stroke"/>'
            html = html .. '</svg>'
            html = html .. '<div class="kanban-col-empty-text">Nothing going on</div>'
            html = html .. '<div class="kanban-col-empty-sub">Just Gengar passing through.</div>'
            html = html .. '</div>'
        end

        html = html .. '</div></div>' -- close kanban-cards + kanban-column
    end

    html = html .. '</div>' -- close kanban-board
    html = html .. '</div>' -- close root wrapper

    local jsCode = [[
    (function() {
        // Drag & drop engine: installed once per session, operating via event
        // delegation on document. It resolves which board an event belongs to
        // at event time via closest('[data-kanban-root]'), so it keeps working
        // when SilverBullet re-renders or replaces this widget's DOM without
        // re-invoking the Lua function.
        if (window.kanbanEngineInstalled) return;
        window.kanbanEngineInstalled = true;

        const getRoot = (el) => (el && el.closest) ? el.closest('[data-kanban-root]') : null;

        let draggedCard = null;
        let sourceColumn = null;
        let isDragging = false;
        let dragTimer = null;
        let touchStartX, touchStartY;

        // Column counts reflect what is visible: snoozed cards are hidden
        // (via CSS) unless the board carries the show-snoozed class, and
        // cards with a toggled-off tag chip carry kanban-tag-hidden
        const recount = (column) => {
            const countEl = column.querySelector('.kanban-col-count');
            if (!countEl) return;
            const board = column.closest('.kanban-board');
            const showSnoozed = board && board.classList.contains('show-snoozed');
            let selector = '.kanban-card:not(.kanban-tag-hidden)';
            if (!showSnoozed) {
                selector += ':not([data-snoozed="true"])';
            }
            countEl.textContent = column.querySelectorAll(selector).length;
        };

        const dispatchMove = (root, card, column) => {
            const statusKey = root.dataset.statusKey;
            const newStatus = column.dataset.status;
            const page = card.dataset.page;

            // Dropped in the same column: nothing to persist
            if (sourceColumn === column) return;

            if (sourceColumn) recount(sourceColumn);
            recount(column);

            if (page && newStatus) {
                window.dispatchEvent(new CustomEvent("sb-kanban-dnd-update", {
                    detail: { action: "move", page: page, statusKey: statusKey, newStatus: newStatus }
                }));
            }
        };

        document.addEventListener("dragstart", (e) => {
            if (!e.target.classList || !e.target.classList.contains('kanban-card')) return;
            draggedCard = e.target;
            sourceColumn = e.target.closest('.kanban-column');
            setTimeout(() => { e.target.style.opacity = '0.5'; }, 0);
        });

        document.addEventListener("dragend", () => {
            if (draggedCard) {
                draggedCard.style.opacity = '';
                draggedCard = null;
                sourceColumn = null;
            }
        });

        document.addEventListener("dragover", (e) => {
            if (getRoot(e.target)) e.preventDefault();
        });

        document.addEventListener("drop", (e) => {
            const root = getRoot(e.target);
            if (!root || !draggedCard) return;
            e.preventDefault();
            const column = e.target.closest('.kanban-column');
            if (!column) return;
            column.querySelector('.kanban-cards').appendChild(draggedCard);
            dispatchMove(root, draggedCard, column);
        });

        document.addEventListener("touchstart", (e) => {
            const card = e.target.closest ? e.target.closest('.kanban-card') : null;
            if (card && e.touches.length === 1) {
                const touch = e.touches[0];
                touchStartX = touch.clientX;
                touchStartY = touch.clientY;
                dragTimer = setTimeout(() => {
                    isDragging = true;
                    draggedCard = card;
                    sourceColumn = card.closest('.kanban-column');
                    draggedCard.style.opacity = '0.5';
                }, 500);
            }
        }, { passive: true });

        document.addEventListener("touchmove", (e) => {
            if (dragTimer) {
                const touch = e.touches[0];
                const deltaX = Math.abs(touch.clientX - touchStartX);
                const deltaY = Math.abs(touch.clientY - touchStartY);
                if (deltaX > 10 || deltaY > 10) {
                    clearTimeout(dragTimer);
                    dragTimer = null;
                }
            }
            if (!isDragging || !draggedCard) return;
            e.preventDefault();
            const touch = e.touches[0];
            const elementOver = document.elementFromPoint(touch.clientX, touch.clientY);
            const columnOver = elementOver ? elementOver.closest('.kanban-column') : null;
            if (columnOver) {
                const cardsContainer = columnOver.querySelector('.kanban-cards');
                cardsContainer.appendChild(draggedCard);
            }
        }, { passive: false });

        document.addEventListener("touchend", () => {
            if (dragTimer) {
                clearTimeout(dragTimer);
                dragTimer = null;
            }
            if (!isDragging || !draggedCard) return;
            isDragging = false;
            draggedCard.style.opacity = '';
            const root = getRoot(draggedCard);
            const column = draggedCard.closest('.kanban-column');
            if (root && column) dispatchMove(root, draggedCard, column);
            draggedCard = null;
            sourceColumn = null;
        });

        // Snooze toggle: reveal/hide snoozed cards (hidden via CSS) and
        // persist the state so it survives widget re-renders. The state is
        // read by the Lua side on the next render.
        document.addEventListener("click", (e) => {
            const toggleBtn = e.target.closest ? e.target.closest('.kanban-snooze-toggle-btn') : null;
            if (!toggleBtn) return;
            e.preventDefault();
            const root = getRoot(toggleBtn);
            if (!root) return;
            const board = root.querySelector('.kanban-board');
            if (!board) return;
            board.classList.toggle('show-snoozed');
            const showSnoozed = board.classList.contains('show-snoozed');
            window._kanbanShowSnoozed = showSnoozed;
            toggleBtn.classList.toggle('active', showSnoozed);
            toggleBtn.title = showSnoozed ? "Hide snoozed tasks" : "Show snoozed tasks";
            root.querySelectorAll('.kanban-column').forEach(recount);
        });

        // Tag chip toggle: hide/show all pages carrying the chip's tag.
        // The set of toggled-off tags persists as a comma-separated string in
        // the browser window; the Lua side reads it on the next render.
        document.addEventListener("click", (e) => {
            const chip = e.target.closest ? e.target.closest('.kanban-tag-chip') : null;
            if (!chip) return;
            e.preventDefault();
            const root = getRoot(chip);
            if (!root) return;
            const tag = chip.dataset.tag;
            if (!tag) return;

            const off = new Set(
                (window._kanbanTagsOff || "").split(",").filter((t) => t.length > 0),
            );
            const nowActive = chip.classList.toggle("active");
            if (nowActive) {
                off.delete(tag);
                chip.title = "Hide pages tagged #" + tag;
            } else {
                off.add(tag);
                chip.title = "Show pages tagged #" + tag;
            }
            window._kanbanTagsOff = Array.from(off).join(",");

            root.querySelectorAll(".kanban-card").forEach((card) => {
                const tags = (card.dataset.tags || "").split(",").filter((t) => t.length > 0);
                card.classList.toggle("kanban-tag-hidden", tags.some((t) => off.has(t)));
            });
            root.querySelectorAll(".kanban-column").forEach(recount);
        });

        // Per-card snooze button: opens the browser's native date picker
        // (pre-filled with the card's current snoozeDate) and dispatches an
        // event that the Lua side persists to the page's frontmatter.
        // Clearing the picker writes "none" (not snoozed).
        document.addEventListener("click", (e) => {
            const snoozeBtn = e.target.closest ? e.target.closest('.kanban-snooze-btn') : null;
            if (!snoozeBtn) return;
            e.preventDefault();
            e.stopPropagation();
            const card = snoozeBtn.closest('.kanban-card');
            if (!card) return;
            const page = card.dataset.page;
            if (!page) return;

            // Remove any leftover picker input from a previous interaction
            document.querySelectorAll('.kanban-snooze-picker').forEach((el) => el.remove());

            const input = document.createElement('input');
            input.type = 'date';
            input.className = 'kanban-snooze-picker';
            input.value = card.dataset.snoozeDate || '';
            input.style.position = 'fixed';
            input.style.left = (e.clientX || 0) + 'px';
            input.style.top = (e.clientY || 0) + 'px';
            input.style.width = '1px';
            input.style.height = '1px';
            input.style.opacity = '0';
            input.style.padding = '0';
            input.style.border = 'none';
            input.style.zIndex = '9999';
            document.body.appendChild(input);

            input.addEventListener('change', () => {
                const value = input.value || 'none';
                input.remove();
                window.dispatchEvent(new CustomEvent("sb-kanban-snooze-update", {
                    detail: { action: "snooze", page: page, value: value }
                }));
            });
            // Dismissed without picking anything: just clean up
            input.addEventListener('cancel', () => input.remove());
            input.addEventListener('blur', () => setTimeout(() => input.remove(), 100));

            try {
                input.showPicker();
            } catch (err) {
                // showPicker() unsupported or blocked: fall back to showing
                // the input itself so it can be interacted with directly
                input.style.opacity = '1';
                input.style.width = 'auto';
                input.style.height = 'auto';
                input.style.padding = '4px 6px';
                input.focus();
            }
        });
    })();
    ]]

    if not js.window.kanbanEngineInstalled then
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

## Discussions to this library
- [Silverbullet Community](https://community.silverbullet.md/t/kanban-integration-with-tasks/925/12?u=mr.red)
