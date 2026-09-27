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
* Embed `${ReminderWall()}` for a sticky-note style view of your reminders — see the demo wall below. Reminders whose date/time has not yet come count as snoozed: they are hidden by default and can be revealed, grayed out, via the ⏰ toggle. Each note carries a 💤 button that postpones the reminder to the picked date by updating its `reminderDate` (clearing the picker makes it due again today). Notes are colored by how overdue the reminder is: light yellow when it just popped up, continuously fading to dark red when it is more than a week overdue. A custom query can be passed to show a different selection.

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

/* Move SilverBullet's widget button bar out of the box (above it), so it
   does not cover the wall's snooze toggle in the top right corner;
   same approach as the Kanban board */
#sb-main .cm-editor .sb-lua-directive-block:has(.rem-wall) .button-bar {
  top: -40px;
  padding: 0;
  border-radius: 2em;
  opacity: 0.2;
  transition: all 0.5s ease;
}
#sb-main .cm-editor .sb-lua-directive-block:has(.rem-wall) .button-bar:hover {
  opacity: 1;
}

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

/* Empty state: kanban-column style background with a centered, subtle
   sleeping-Snorlax illustration */
.rem-empty {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 100%;
  min-height: 180px;
  padding: 24px 16px;
  box-sizing: border-box;
  border-radius: 18px;
  background: oklch(from var(--modal-help-background-color) l c h / 0.4);
  color: var(--text-muted);
  font-style: italic;
  text-align: center;
}

.rem-empty-art {
  width: 150px;
  height: auto;
  margin-bottom: 8px;
  opacity: 0.8;
}

.rem-empty-sub {
  font-size: 0.8em;
  opacity: 0.75;
}

.rem-note {
  position: relative;
  width: 200px;
  min-height: 90px;
  box-sizing: border-box;
  padding: 12px 12px 16px;
  border-radius: 2px;
  /* Fallback only — the Lua code sets the per-note background inline,
     colored by overdue age: light yellow when the reminder just popped up,
     fading to dark red once it is more than a week overdue */
  background: linear-gradient(160deg, oklch(0.93 0.12 100), oklch(0.85 0.13 85));
  color: oklch(0.25 0.05 60);
  box-shadow: 2px 4px 10px rgba(0 0 0 / 0.3);
  transform: rotate(-1.2deg);
  transition: transform 0.15s ease;
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
-- Deliberately a global function: the session-persistent event listener
-- below resolves globals at call time, so a library update redefines this
-- implementation and even a listener registered by an older library
-- version immediately uses the current logic.
-- (The Kanban board ships its own independent copy.)
function reminderSetFrontmatter(pageName, key, value)
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

-- ------------- Due time helpers -------------
-- Returns a reminder page's due timestamp, or nil when its fields are
-- missing or unparseable
local function dueTimeOf(p)
    local rd = p.reminderDate
    local rt = p.reminderTime
    if rd == nil or rt == nil then return nil end
    rd = tostring(rd)
    rt = tostring(rt)
    local y = tonumber(rd:sub(1, 4))
    local mo = tonumber(rd:sub(6, 7))
    local d = tonumber(rd:sub(9, 10))
    local h = tonumber(rt:sub(1, 2))
    local mi = tonumber(rt:sub(4, 5))
    if y == nil or mo == nil or d == nil or h == nil or mi == nil then return nil end
    return os.time({ year = y, month = mo, day = d, hour = h, min = mi })
end

-- On the wall a reminder counts as "snoozed" while its due time has not
-- yet come; once due it shows up normally. (The snoozeDate field is
-- deliberately not used here — postponing a reminder means moving its
-- reminderDate, see the 💤 button.)
local function isSnoozed(p)
    local due = dueTimeOf(p)
    if due == nil then return false end
    return due >= os.time()
end

-- Sticky color by overdue age: light yellow when the reminder just popped
-- up, continuously fading to dark red once it is more than a week overdue
local function overdueColorStyle(p)
    local ageT = 0
    local due = dueTimeOf(p)
    if due ~= nil then
        local age = os.time() - due
        if age > 0 then
            ageT = age / (7 * 24 * 60 * 60) -- fraction of a week overdue
            if ageT > 1 then ageT = 1 end
        end
    end
    -- Interpolate the gradient stops (top-left, bottom-right) in OKLCH:
    -- light yellow (hue ~100) towards dark red (hue ~30)
    local l1 = 0.93 + (0.68 - 0.93) * ageT
    local c1 = 0.12 + (0.17 - 0.12) * ageT
    local h1 = 100 + (35 - 100) * ageT
    local l2 = 0.85 + (0.58 - 0.85) * ageT
    local c2 = 0.13 + (0.17 - 0.13) * ageT
    local h2 = 85 + (27 - 85) * ageT
    local stop1 = string.format("oklch(%.3f %.3f %.1f)", l1, c1, h1)
    local stop2 = string.format("oklch(%.3f %.3f %.1f)", l2, c2, h2)
    return ' style="background: linear-gradient(160deg, ' .. stop1 .. ', ' .. stop2 .. ')"'
end

-- Global snooze handler, redefined on every library evaluation. The
-- listener registered below is a thin, stable shim that dispatches to this
-- global, so library updates take effect in live sessions without a
-- page reload.
function reminderHandleSnooze(detail)
    if detail and detail.action == "snooze" then
        local value = tostring(detail.value or "")
        if value == "" or value:lower() == "none" then
            -- Clearing the picker makes the reminder due again today
            value = os.date("%Y-%m-%d")
        end
        -- Snoozing a reminder means postponing its reminderDate
        reminderSetFrontmatter(detail.page, "reminderDate", value)
    end
end

-- ------------- Event Listeners -------------
if not js.window.reminderWallListenersAdded then
    js.window.addEventListener("sb-reminder-snooze-update", function(e)
        reminderHandleSnooze(e.detail)
    end)
    js.window.reminderWallListenersAdded = true
end

-- ------------- Reminder Wall Widget -------------
function ReminderWall(reminderQuery)
    -- Default: all reminders with valid date/time fields, ordered by
    -- reminderDate. Due ones (date/time passed) show up normally; ones whose
    -- date/time has not yet come count as snoozed (hidden, ⏰ reveals them).
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

        -- Note attributes: write target, pending state and current reminder
        -- date (pre-fills the snooze picker)
        local noteAttrs = ' data-page="' .. pageNameEsc .. '"'
        if isSnoozed(p) then
            noteAttrs = noteAttrs .. ' data-snoozed="true"'
        end
        local rd = p.reminderDate
        if rd ~= nil and tostring(rd):find("^%d%d%d%d%-%d%d%-%d%d") then
            noteAttrs = noteAttrs .. ' data-snooze-date="' .. tostring(rd) .. '"'
        end

        html = html .. '<div class="rem-note"' .. noteAttrs .. overdueColorStyle(p) .. '>'
        html = html .. '<a class="rem-note-title" draggable="false" href="/' .. pageNameEsc .. '" data-ref="/' .. pageNameEsc .. '" title="' .. pageNameEsc .. '">' .. displayNameEsc .. '</a>'
        html = html .. '<div class="rem-note-due">' .. tostring(p.reminderDate) .. ' ' .. tostring(p.reminderTime) .. '</div>'
        html = html .. '<div class="rem-snooze-btn" title="Postpone this reminder until...">💤</div>'
        html = html .. '</div>'
    end

    if noteCount == 0 then
        -- Empty state: a sleeping Snorlax with some subtle text
        html = html .. '<div class="rem-empty">'
        html = html .. '<svg class="rem-empty-art" viewBox="0 0 220 150" xmlns="http://www.w3.org/2000/svg">'
        html = html .. '<text x="182" y="34" font-size="20" fill="currentColor" opacity="0.45">Z</text>'
        html = html .. '<text x="197" y="21" font-size="14" fill="currentColor" opacity="0.35">z</text>'
        html = html .. '<text x="207" y="11" font-size="10" fill="currentColor" opacity="0.25">z</text>'
        html = html .. '<path d="M72 40 L64 12 L94 28 Z" fill="#2e6b63"/>'
        html = html .. '<path d="M148 40 L156 12 L126 28 Z" fill="#2e6b63"/>'
        html = html .. '<ellipse cx="110" cy="85" rx="75" ry="62" fill="#2e6b63"/>'
        html = html .. '<ellipse cx="110" cy="78" rx="48" ry="34" fill="#f2e3c2"/>'
        html = html .. '<path d="M76 66 q11 10 22 0" stroke="#3f3a33" stroke-width="3" fill="none" stroke-linecap="round"/>'
        html = html .. '<path d="M122 66 q11 10 22 0" stroke="#3f3a33" stroke-width="3" fill="none" stroke-linecap="round"/>'
        html = html .. '<ellipse cx="110" cy="77" rx="5" ry="3.5" fill="#3f3a33"/>'
        html = html .. '<ellipse cx="110" cy="93" rx="7" ry="5" fill="#3f3a33"/>'
        html = html .. '<ellipse cx="110" cy="122" rx="44" ry="25" fill="#f2e3c2"/>'
        html = html .. '<circle cx="70" cy="116" r="12" fill="#2e6b63"/>'
        html = html .. '<circle cx="150" cy="116" r="12" fill="#2e6b63"/>'
        html = html .. '</svg>'
        html = html .. '<div class="rem-empty-text">No reminders due</div>'
        html = html .. '<div class="rem-empty-sub">Shh... Snorlax is sleeping.</div>'
        html = html .. '</div>'
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
        // (pre-filled with the reminder's current reminderDate) and dispatches
        // an event that the Lua side persists to the page's frontmatter.
        // Clearing the picker makes the reminder due again today.
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
