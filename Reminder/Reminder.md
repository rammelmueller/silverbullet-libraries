---
name: "Library/rammelmueller/Reminder"
tags: meta/library
pageDecoration.prefix: "⏰ "
files:
- ReminderTemplate.md
---

# Reminder

Track reminders on pages via `reminderDate`/`reminderTime` frontmatter and list due reminders anywhere in your space.

## Usage

* Run the **Reminder** command — the shipped page template creates a new page under `Reminder/<timestamp>` with the reminder frontmatter pre-filled.
* New reminders default to **today at 00:00**, so they are due immediately. Set a `reminderDate` in the future (format `YYYY-MM-DD`) and `reminderTime` (format `HH:MM`) to schedule them; reminders without a title are listed under their page name.
* Embed `${get_active_reminders()}` on any page — it renders all **due** reminders (their date/time has passed), ordered by `reminderDate`, with a link to the reminder page.
* Embed `${ReminderWall()}` for a sticky-note style view of the same due reminders — see the demo wall below. Each note carries a 💤 button that snoozes the reminder until a picked date (clearing the picker reactivates it); snoozed notes disappear from the wall and can be revealed, grayed out, via the ⏰ toggle. A custom query can be passed to show a different selection.

> **note** Reminder pages carry the `reminder` tag. Any page tagged `reminder` with valid `reminderDate`/`reminderTime` fields counts — the template is just a convenience; pages with missing or unparseable fields are ignored.
> Updating the library re-pulls the library page and the shipped template, overwriting local changes to them.

## Example Reminder Page

```
---
title: Think that I can't forget
reminderDate: 2026-10-12
reminderTime: 07:30
creationDate: 2026-09-27
tags: 
- reminder
---
Some thing that I definitely can't forget, on a given day.
```

## DEMO WIDGET
${ReminderWall()}

## CSS Styling

```space-style
/* ---------------------------------
   Reminder Wall (sticky notes)
---------------------------------- */

.rem-controls {
  display: flex;
  justify-content: flex-end;
  padding: 0 10px;
}

.rem-snooze-toggle-btn {
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

.rem-snooze-toggle-btn:hover,
.rem-snooze-toggle-btn.active {
  background: var(--ui-accent-color);
  color: var(--modal-selected-option-color);
}

.rem-snooze-toggle-btn:active {
  opacity: 0.7;
}

.rem-wall {
  display: flex;
  flex-wrap: wrap;
  gap: 14px;
  padding: 10px;
  align-items: flex-start;
  align-content: flex-start;
}

.rem-empty {
  padding: 20px;
  opacity: 0.6;
  font-style: italic;
}

.rem-note {
  position: relative;
  width: 200px;
  min-height: 90px;
  box-sizing: border-box;
  padding: 12px 12px 16px;
  border-radius: 2px;
  background: linear-gradient(160deg, oklch(0.92 0.11 95), oklch(0.83 0.13 75));
  color: oklch(0.25 0.05 60);
  box-shadow: 2px 4px 10px rgba(0 0 0 / 0.3);
  transform: rotate(-1.2deg);
  transition: transform 0.15s ease;
}

html[data-theme='dark'] .rem-note {
  background: linear-gradient(160deg, oklch(0.8 0.12 90), oklch(0.68 0.13 70));
}

.rem-note:nth-child(even) { transform: rotate(1.3deg); }
.rem-note:nth-child(3n) { transform: rotate(-0.5deg); }

.rem-note:hover {
  transform: rotate(0) scale(1.04);
  z-index: 1;
}

.rem-note-title {
  display: block;
  font-weight: bold;
  margin-bottom: 6px;
  text-decoration-line: none;
  color: inherit;
  overflow-wrap: break-word;
}

.rem-note-due {
  font-size: 0.85em;
  opacity: 0.75;
}

.rem-snooze-btn {
  position: absolute;
  bottom: 0;
  right: 4px;
  font-size: 13px;
  cursor: pointer;
  opacity: 0.4;
  padding: 3px 5px;
  z-index: 1;
  line-height: normal;
  transition: opacity 0.2s;
}

.rem-snooze-btn:hover {
  opacity: 1;
}

.rem-note[data-snoozed="true"] .rem-snooze-btn {
  opacity: 0.8;
}

/* Snoozed notes are hidden unless the wall carries the show-snoozed class */
.rem-wall:not(.show-snoozed) .rem-note[data-snoozed="true"] {
  display: none;
}

/* Revealed snoozed notes are grayed out */
.rem-wall .rem-note[data-snoozed="true"] {
  filter: grayscale(0.9);
  opacity: 0.75;
}

```

## Lua Implementation
```space-lua
function get_active_reminders()
  return query[[
    from index.tag "page"
    where table.includes(tags, "reminder") 
    and reminderDate ~= nil
    and reminderTime ~= nil
    and tonumber(reminderDate:sub(1,4)) ~= nil
    and tonumber(reminderDate:sub(6,7)) ~= nil
    and tonumber(reminderDate:sub(9,10)) ~= nil
    and tonumber(reminderTime:sub(1,2)) ~= nil
    and tonumber(reminderTime:sub(4,5)) ~= nil
    and
          os.time({
            year = tonumber(reminderDate:sub(1,4)), 
            month = tonumber(reminderDate:sub(6,7)), 
            day = tonumber(reminderDate:sub(9,10)), 
            hour = tonumber(reminderTime:sub(1,2)), 
            min = tonumber(reminderTime:sub(4,5))
          }) < os.time()
    order by reminderDate
    select {
      title = "[[" .. name .. "|" .. (title or name) .. "]]",
      due = reminderDate .. " / " .. reminderTime,
    }
  ]]
end

-- ------------- Helper: Escape Magic Characters for Lua Patterns -------------
local function escapeLuaPattern(s)
    return s:gsub("([%^%$%(%)%%%.%[%]%*%+%-%?])", "%%%1")
end

-- ------------- Write a single key into a page's frontmatter -------------
-- (private to this library; the Kanban board ships its own independent copy)
local function setFrontmatterValue(pageName, key, value)
    local content = space.readPage(pageName)
    if not content then return end

    local val = tostring(value or "")

    -- Write the value as a plain YAML scalar; quote it only when a plain
    -- scalar would be ambiguous
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
        codeWidget.refreshAll()
    end, 200)
end

-- ------------- Snooze check -------------
-- A reminder page is snoozed while its snoozeDate (YYYY-MM-DD) lies in the
-- future; a missing, empty, "none" or unparsable value means it is active.
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

-- ------------- Event Listeners -------------
if not js.window.reminderWallListenersAdded then
    js.window.addEventListener("sb-reminder-snooze-update", function(e)
        if e.detail and e.detail.action == "snooze" then
            setFrontmatterValue(e.detail.page, "snoozeDate", e.detail.value)
        end
    end)
    js.window.reminderWallListenersAdded = true
end

-- ------------- Reminder Wall Widget -------------
function ReminderWall(reminderQuery)
    -- Default: all due reminders, ordered by reminderDate
    if reminderQuery == nil then
        reminderQuery = query[[
            from index.tag "page"
            where table.includes(tags, "reminder") 
            and reminderDate ~= nil
            and reminderTime ~= nil
            and tonumber(reminderDate:sub(1,4)) ~= nil
            and tonumber(reminderDate:sub(6,7)) ~= nil
            and tonumber(reminderDate:sub(9,10)) ~= nil
            and tonumber(reminderTime:sub(1,2)) ~= nil
            and tonumber(reminderTime:sub(4,5)) ~= nil
            and
                  os.time({
                    year = tonumber(reminderDate:sub(1,4)), 
                    month = tonumber(reminderDate:sub(6,7)), 
                    day = tonumber(reminderDate:sub(9,10)), 
                    hour = tonumber(reminderTime:sub(1,2)), 
                    min = tonumber(reminderTime:sub(4,5))
                  }) < os.time()
            order by reminderDate
        ]]
    end

    -- Snooze toggle state persists in the browser window so it survives
    -- widget re-renders
    local showSnoozed = (js.window._reminderWallShowSnoozed == true)

    local snoozeBtnClass = "rem-snooze-toggle-btn"
    local snoozeBtnTitle = "Show snoozed reminders"
    if showSnoozed then
        snoozeBtnClass = snoozeBtnClass .. " active"
        snoozeBtnTitle = "Hide snoozed reminders"
    end

    local html = '<div data-reminder-root="true">'
    html = html .. '<div class="rem-controls"><button class="' .. snoozeBtnClass .. '" title="' .. snoozeBtnTitle .. '">⏰</button></div>'
    html = html .. '<div class="rem-wall' .. (showSnoozed and ' show-snoozed' or '') .. '">'

    local noteCount = 0
    for p in reminderQuery do
        noteCount = noteCount + 1
        local pageName = tostring(p.name or "")
        local displayName = tostring(p.title or pageName)
        local pageNameEsc = pageName:gsub('"', '&quot;')
        pageNameEsc = pageNameEsc:gsub('<', '&lt;')
        pageNameEsc = pageNameEsc:gsub('>', '&gt;')
        local displayNameEsc = displayName:gsub('"', '&quot;')
        displayNameEsc = displayNameEsc:gsub('<', '&lt;')
        displayNameEsc = displayNameEsc:gsub('>', '&gt;')

        -- Note attributes: write target, snooze state and current snooze date
        local noteAttrs = ' data-page="' .. pageNameEsc .. '"'
        if isSnoozed(p) then
            noteAttrs = noteAttrs .. ' data-snoozed="true"'
        end
        local sd = p.snoozeDate
        if sd ~= nil and tostring(sd):find("^%d%d%d%d%-%d%d%-%d%d") then
            noteAttrs = noteAttrs .. ' data-snooze-date="' .. tostring(sd) .. '"'
        end

        html = html .. '<div class="rem-note"' .. noteAttrs .. '>'
        html = html .. '<a class="rem-note-title" draggable="false" href="/' .. pageNameEsc .. '" data-ref="/' .. pageNameEsc .. '" title="' .. pageNameEsc .. '">' .. displayNameEsc .. '</a>'
        html = html .. '<div class="rem-note-due">' .. tostring(p.reminderDate) .. ' ' .. tostring(p.reminderTime) .. '</div>'
        html = html .. '<div class="rem-snooze-btn" title="Snooze this reminder until...">💤</div>'
        html = html .. '</div>'
    end

    if noteCount == 0 then
        html = html .. '<div class="rem-empty">No reminders due.</div>'
    end

    html = html .. '</div>' -- close rem-wall
    html = html .. '</div>' -- close root wrapper

    local jsCode = [[
    (function() {
        // Reminder wall engine: installed once per session, operating via event
        // delegation on document, resolving the wall at event time via
        // closest('[data-reminder-root]'). Kept independent from the Kanban
        // board's engine (own classes, own event names).
        if (window.reminderWallEngineAdded) return;
        window.reminderWallEngineAdded = true;

        const getRoot = (el) => (el && el.closest) ? el.closest('[data-reminder-root]') : null;

        // Snooze toggle: reveal/hide snoozed notes (hidden via CSS) and
        // persist the state for the next render
        document.addEventListener("click", (e) => {
            const toggleBtn = e.target.closest ? e.target.closest('.rem-snooze-toggle-btn') : null;
            if (!toggleBtn) return;
            e.preventDefault();
            const root = getRoot(toggleBtn);
            if (!root) return;
            const wall = root.querySelector('.rem-wall');
            if (!wall) return;
            wall.classList.toggle('show-snoozed');
            const showSnoozed = wall.classList.contains('show-snoozed');
            window._reminderWallShowSnoozed = showSnoozed;
            toggleBtn.classList.toggle('active', showSnoozed);
            toggleBtn.title = showSnoozed ? "Hide snoozed reminders" : "Show snoozed reminders";
        });

        // Per-note snooze button: opens the browser's native date picker
        // (pre-filled with the reminder's current snoozeDate) and dispatches
        // an event that the Lua side persists to the page's frontmatter.
        // Clearing the picker writes "none" (reminder active again).
        document.addEventListener("click", (e) => {
            const snoozeBtn = e.target.closest ? e.target.closest('.rem-snooze-btn') : null;
            if (!snoozeBtn) return;
            e.preventDefault();
            e.stopPropagation();
            const note = snoozeBtn.closest('.rem-note');
            if (!note) return;
            const page = note.dataset.page;
            if (!page) return;

            // Remove any leftover picker input from a previous interaction
            document.querySelectorAll('.rem-snooze-picker').forEach((el) => el.remove());

            const input = document.createElement('input');
            input.type = 'date';
            input.className = 'rem-snooze-picker';
            input.value = note.dataset.snoozeDate || '';
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
                window.dispatchEvent(new CustomEvent("sb-reminder-snooze-update", {
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

    if not js.window.reminderWallEngineAdded then
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
