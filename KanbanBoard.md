---
name: "Library/Mr-xRed/KanbanBoard"
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
      {"waiting", "⏳ In Progress","blue"},
      {"in progress", "👀 Needs Review","orange"},
      {"done", "✅ Done","green"}
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
- **Customisable columns** — define your own workflow stages with labels, emoji, and optional accent colours per column
- **Custom card fields** — choose which frontmatter attributes are shown on each card (`Fields`)
- **HideKeys** — display an attribute value without showing its label (handy for IDs or long text)
- **Mobile-friendly** — columns hold their minimum width and the board scrolls horizontally on narrow screens instead of squishing


### 🚫 Known Limitations

- Tags for the `Tags` filter must live in **frontmatter** (`tags: kanban, project`); hashtags in the page body are not considered
- Manual frontmatter edits to a page require a widget refresh to appear on the board
- Status values are matched case-insensitively; pages with an unknown or missing status appear in the first column
- Status values are written as plain YAML scalars (`status: done`) and are only quoted when a plain scalar would be ambiguous

## Setup and Configuration

### Widget Parameters

*   **`query`**: (Required) A SilverBullet query returning pages, e.g. `query[[from index.tag "page"]]`.
*   **`options`**: (Required) A table to configure the board's behavior.
    *   **`Column`**: The frontmatter attribute to use for column assignment (e.g., `"status"`).
    *   **`Columns`**: An ordered list of columns, where each column is a `{status, title}` pair with an optional third element as accent colour (e.g., `{"todo", "📝 To Do", "purple"}`).
    *   **`Tags`**: (Optional) A list of tags. A page must carry **all** of them in its frontmatter to appear on this board.
    *   **`Fields`**: (Optional) A list of frontmatter attributes to display on the card. E.g. `{"due", "priority"}`.
    *   **`HideKeys`**: (Optional) Hide certain attribute keys/labels from the card. This can be useful if you have a longer text or a title as attribute and want to display the whole thing.

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
  max-width: 500px;
  flex-shrink: 0;
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
    -- Normalizes a tag: strips a leading '#', trims and lowercases
    local function normalizeTag(t)
        local s = tostring(t):gsub("^#", "")
        return (s:gsub("^%s*(.-)%s*$", "%1"):lower())
    end

    local statusKey = "status"
    local columnOrder = {}
    local columnTitles = {}
    local columnColors = {} -- optional accent color per column, set via 3rd element in Columns config
    local fields = {} -- for custom fields
    local hideKeys = {} -- field keys whose label is hidden on cards, set via {"HideKeys", {"key1","key2"}}
    local requiredTags = {} -- a page must carry all of these frontmatter tags, set via {"Tags", {"tag1","tag2"}}

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
            local raw_status = p[statusKey] or columnOrder[1]
            local status = tostring(raw_status):gsub("^%s*(.-)%s*$", "%1")
                  status = status:lower()

            if pagesByStatus[status] then
                table.insert(pagesByStatus[status], p)
            else
                -- If it doesn't have a status or is unknown, move to first column
                table.insert(pagesByStatus[columnOrder[1]], p)
            end
        end
    end

    -- The root wrapper carries the board config as data attributes; a single
    -- session-persistent delegated event engine (installed once, see jsCode
    -- below) operates on any board instance purely from the DOM. This keeps
    -- drag & drop working even when SilverBullet re-renders or replaces this
    -- widget's DOM without re-invoking this Lua function.
    local html = '<div data-kanban-root="true" data-status-key="' .. statusKey .. '">'
    html = html .. '<div class="kanban-board">'

    for _, status in ipairs(columnOrder) do
        local title = columnTitles[status]
        local pages = pagesByStatus[status]

        -- Build optional color class and inline CSS variable for this column
        local colorAttr = ""
        if columnColors[status] then
            colorAttr = ' kanban-column-colored" style="--column-card-color: ' .. columnColors[status]
        end

        html = html .. '<div class="kanban-column' .. colorAttr .. '" data-status="' .. status .. '">'
        html = html .. '<div class="kanban-column-title">' .. title .. ' (<span class="kanban-col-count">' .. #pages .. '</span>)</div>'
        html = html .. '<div class="kanban-cards">'

        for _, p in ipairs(pages) do
            local pageName = tostring(p.name or "")
            pageName = pageName:gsub('"', '&quot;')
            pageName = pageName:gsub('<', '&lt;')
            pageName = pageName:gsub('>', '&gt;')

            html = html .. '<div class="kanban-card" draggable="true" data-page="' .. pageName .. '">'

            -- Clickable Title
            html = html .. '<a class="kanban-card-name" draggable="false" href="/' .. pageName .. '" data-ref="/' .. pageName .. '" title="' .. pageName .. '">' .. pageName .. '</a>'

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

        const recount = (column) => {
            const countEl = column.querySelector('.kanban-col-count');
            if (countEl) countEl.textContent = column.querySelectorAll('.kanban-card').length;
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
