---
name: "Library/rammelmueller/KanbanBoard"
tags: meta/library
pageDecoration.prefix: "✅ "
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

            html = html .. '<div class="' .. cardClass .. '" draggable="true" data-page="' .. pageNameEsc .. '" data-tags="' .. cardTagsEsc .. '"' ..
                (isSnoozed(p) and ' data-snoozed="true"' or '') .. '>'

            -- Clickable Title (link and tooltip keep the full page name)
            html = html .. '<a class="kanban-card-name" draggable="false" href="/' .. pageNameEsc .. '" data-ref="/' .. pageNameEsc .. '" title="' .. pageNameEsc .. '">' .. displayNameEsc .. '</a>'

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
