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
  /* ink just a bit darker than the panel background, in both themes */
  color: oklch(from var(--modal-help-background-color) calc(l - 0.1) c h);
  font-style: italic;
  text-align: center;
}

.rem-empty-art {
  width: 150px;
  height: auto;
  margin-bottom: 8px;
  /* pencil-style line art: strokes use currentColor, inheriting the
     placeholder's light ink color */
}

.rem-empty-sub {
  font-size: 0.8em;
}

/* When the toggle reveals hidden (snoozed) notes, the placeholder retracts;
   with truly no notes at all it stays put regardless of the toggle */
.rem-wall.show-snoozed .rem-empty[data-has-hidden] {
  display: none;
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
    local visibleCount = 0
    for p in reminderQuery do
        noteCount = noteCount + 1
        local snoozed = isSnoozed(p)
        if not snoozed or showSnoozed then
            visibleCount = visibleCount + 1
        end
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
        if snoozed then
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

    if visibleCount == 0 then
        -- Empty state: a sleeping Snorlax with some subtle text. Shown when
        -- nothing is visible — including when all notes exist but are
        -- hidden as snoozed. data-has-hidden lets CSS retract the
        -- placeholder once the toggle reveals those notes (see below).
        local emptyAttrs = ""
        if noteCount > 0 then
            emptyAttrs = ' data-has-hidden="true"'
        end
        html = html .. '<div class="rem-empty"' .. emptyAttrs .. '>'
        -- Pencil-style sleeping Snorlax: line art only, inheriting the
        -- muted text color of the container
        html = html .. '<svg class="rem-empty-art" viewBox="45 -54 918 434" xmlns="http://www.w3.org/2000/svg" fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round">'
        html = html .. '<text x="514" y="36" font-size="55" fill="currentColor" stroke="none" opacity="1">Z</text>'
        html = html .. '<text x="570" y="4" font-size="40" fill="currentColor" stroke="none" opacity="0.85">z</text>'
        html = html .. '<text x="608" y="-26" font-size="27" fill="currentColor" stroke="none" opacity="0.7">z</text>'
        html = html .. '<path d="M442.606,302.737c2.14-0.958,4.151-2.231,5.902-3.792c1.764-1.565,3.273-3.421,4.508-5.427c1.971-3.205,3.306-6.791,4.215-10.432 c1.58-6.403,1.864-13.1,1.335-19.657c-0.529-6.321-1.788-12.637-3.981-18.597c-1.443-3.91-3.359-7.681-5.513-11.245 c-2.032-3.388-4.563-7.02-6.847-10.257c-4.82-6.823-10.12-13.641-15.805-19.766c-7.686-8.322-16.409-15.897-25.527-22.612 c-4.675-3.423-9.618-6.729-14.673-9.566c-3.444-1.908-7.071-3.724-10.814-4.968" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M414.4,268.434c-0.542-14.242-2.095-28.648-5.644-42.471c-2.225-8.7-5.217-17.287-9.095-25.392 c-2.794-5.845-6.131-11.511-9.843-16.819c-3.682-5.284-7.888-10.411-12.288-15.118c-2.99-3.152-6.152-6.326-9.501-9.089 c-4.287-3.576-9.075-6.587-14.012-9.179c-5.078-2.667-10.624-5.181-15.943-7.331c-10.446-4.189-21.381-7.537-32.436-9.655 c-12.339-2.359-25.146-3.24-37.699-3.342c-7.883-0.041-16.231,0.256-24.09,0.856c-7.673,0.599-15.588,1.566-23.116,3.163 c-11.541,2.394-22.826,6.367-33.615,11.085c-12.527,5.507-24.949,12.124-36.431,19.572c-10.325,6.717-20.157,14.439-28.538,23.495 c-22.086,23.636-30.761,54.197-32.32,85.913" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M140.3,366.549c4.666,.761,9.575,1.109,14.302,1.277c7.967,.329,183.719,.177,190.946,.118c7.168,-.133,14.441,-.766,21.401,-2.55" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M458.555,263.287c2.843,1.939,5.613,4.183,7.451,7.136c.808,1.295,1.331,2.755,1.729,4.222c.079,.333,.177,.685,.128,1.029c-.019,.118,-.074,.231,-.163,.312c-.158,.15,-.372,.221,-.583,.258c-.278,.05,-.573,.062,-.855,.071c-1.271,.024,-2.759,-.088,-4.035,-.165" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M459.809,286.489c2.088,-.102,4.653,-.193,6.71,-.439c.439,-.056,.903,-.118,1.332,-.224c.253,-.067,.523,-.136,.727,-.309c.143,-.128,.205,-.333,.245,-.515c.05,-.254,.028,-.517,-.021,-.77c-.073,-.368,-.199,-.728,-.333,-1.078c-.627,-1.572,-1.673,-2.945,-2.798,-4.194c-1.923,-2.095,-4.44,-3.557,-7.051,-4.628" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M453.638,292.459c1.753,1.262,3.589,2.481,5.57,3.357c1.039,.453,2.145,.801,3.284,.854c.286,.012,.58,.01,.859,-.063c.195,-.051,.383,-.151,.503,-.317c.19,-.268,.237,-.64,.236,-.961c-.021,-.716,-.254,-1.411,-.484,-2.083c-1.293,-3.744,-3.725,-6.981,-6.465,-9.795" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M446.531,300.512c.901,1.591,1.865,3.197,3.088,4.564c.493,.539,1.042,1.047,1.685,1.401c.541,.291,1.161,.489,1.777,.514c.338,.009,.688,-.066,.963,-.271c.303,-.222,.492,-.564,.615,-.912c.173,-.495,.243,-1.026,.289,-1.547c.071,-.884,.052,-1.806,.012,-2.691c-.135,-2.829,-.492,-5.738,-1.8,-8.289" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M402.302,262.7c7.605,2.792,14.783,6.838,21.032,12" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M445.312,333.186c-0.975,5.414-2.824,10.715-5.713,15.411c-4.242,6.9-10.599,12.512-18.019,15.776 c-5.91,2.615-12.452,3.434-18.864,3.585c-7.202,0.057-15.466,0.17-22.634-0.416c-5.139-0.43-10.327-1.201-15.25-2.77 c-7.972-2.485-15.161-7.199-21.018-13.11c-2.99-3.025-5.67-6.43-7.824-10.099c-3.623-6.161-5.917-13.063-7.276-20.059 c-1.684-8.763-2.18-17.95,0.004-26.671c1.203-4.846,3.255-9.476,5.921-13.692c2.652-4.189,5.921-8.01,9.701-11.222 c10.683-9.145,25.346-12.276,39.125-11.292" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M436.046,289.069c5.293,8.387,8.684,18.008,9.77,27.868" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M382.633,263.78c0.432,1.683,1.462,3.155,2.644,4.403c1.671,1.752,3.811,3.093,6.188,3.622c1.237,0.285,2.526,0.373,3.79,0.259 c1.334-0.115,2.652-0.509,3.79-1.223c0.93-0.574,1.823-1.287,2.341-2.271c0.692-1.291,0.818-2.794,0.89-4.233 c0.103-3.007-0.146-6.164-0.362-9.165c-0.181-2.135-0.398-4.468-0.751-6.577c-0.142-0.823-0.302-1.671-0.555-2.468 c-0.143-0.434-0.312-0.872-0.587-1.241c-0.147-0.194-0.331-0.365-0.555-0.465c-0.579-0.255-1.226-0.018-1.769,0.225 c-0.735,0.342-1.431,0.798-2.096,1.258c-1.249,0.876-2.482,1.883-3.634,2.882c-1.74,1.518-3.451,3.144-4.978,4.878 c-1.194,1.369-2.327,2.836-3.23,4.415c-0.568,1.008-1.036,2.096-1.219,3.244C382.409,262.137,382.433,262.979,382.633,263.78z" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M418.44,279.501c-.118,2.276,.17,4.576,.636,6.802c.445,1.975,1.053,4.01,2.386,5.58c.896,1.075,2.121,1.821,3.409,2.337c.881,.357,1.805,.688,2.728,.916c.647,.156,1.315,.278,1.982,.268c.464,-.005,.935,-.096,1.34,-.329c.745,-.43,1.308,-1.123,1.864,-1.766c1.571,-1.903,3.142,-4.025,4.539,-6.06c.996,-1.471,1.995,-3.022,2.794,-4.61c.972,-1.932,1.709,-4.041,1.752,-6.221c.015,-.749,-.059,-1.526,-.398,-2.204c-.709,-1.421,-2.381,-1.91,-3.836,-2.165c-3.695,-.595,-7.476,.138,-10.971,1.363c-1.596,.55,-3.27,1.247,-4.814,1.934c-.471,.217,-1.015,.449,-1.465,.705c-.491,.274,-.963,.615,-1.284,1.084C418.626,277.824,418.499,278.683,418.44,279.501z" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M432.765,318.906c-.575,1.614,-.87,3.332,-.983,5.038c-.099,1.666,.017,3.394,.713,4.932c.569,1.294,1.507,2.398,2.595,3.288c.945,.78,2.017,1.429,3.188,1.804c2.516,.824,5.325,.281,7.555,-1.077c1.51,-.913,2.789,-2.173,3.964,-3.477c1.415,-1.622,2.804,-3.351,3.979,-5.156c.373,-.602,.723,-1.236,.929,-1.916c.131,-.435,.199,-.899,.125,-1.351c-.093,-.595,-.449,-1.117,-.895,-1.509c-.638,-.562,-1.423,-.932,-2.211,-1.233c-2.744,-.986,-5.754,-1.323,-8.642,-1.619c-1.763,-.157,-3.621,-.27,-5.391,-.26c-.758,.01,-1.541,.031,-2.288,.165c-.585,.106,-1.176,.302,-1.627,.703C433.278,317.676,432.992,318.296,432.765,318.906z" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M345.471,323.554c0.236-2.51,0.922-4.996,2.148-7.207c2.032-3.646,5.35-6.585,9.239-8.112c3.683-1.458,7.763-1.612,11.65-1.044 c3.092,0.447,6.136,1.294,9.046,2.425c4.691,1.843,9.161,4.454,12.741,8.034c2.609,2.626,4.69,5.775,6.183,9.158 c3.134,7.118,4.332,16.854-0.16,23.636c-1.278,1.925-3.023,3.535-5.044,4.655c-1.807,1.007-3.814,1.64-5.841,2.025 c-2.408,0.452-4.883,0.564-7.327,0.433c-4.054-0.229-8.11-1.118-11.798-2.839c-3.125-1.444-5.965-3.483-8.52-5.776 c-4.045-3.655-7.692-7.908-10.057-12.853C345.899,332.209,345.049,327.834,345.471,323.554z" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M770.451,163.589c-1.6,-1.264,-3.784,-1.547,-5.754,-1.236c-1.051,.18,-2.183,.534,-2.844,1.426c-.344,.463,-.508,1.036,-.554,1.606c-.069,.828,.07,1.662,.3,2.455c.751,2.474,2.266,4.644,3.88,6.632c1.882,2.239,4.097,4.426,6.92,5.391c.528,.169,1.081,.294,1.638,.278c.34,-.011,.691,-.093,.962,-.308c.522,-.422,.613,-1.148,.631,-1.779c.011,-1.144,-.214,-2.298,-.445,-3.414c-.173,-.799,-.396,-1.687,-.611,-2.477c-.571,-2.04,-1.159,-4.147,-2.145,-6.031C771.923,165.182,771.297,164.267,770.451,163.589z" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M756.732,163.162c-1.012,-.443,-2.125,-.626,-3.217,-.738c-1.26,-.112,-2.552,-.11,-3.794,.154c-1.174,.251,-2.397,.811,-2.997,1.907c-.322,.582,-.445,1.253,-.477,1.912c-.052,1.225,.156,2.464,.374,3.665c.569,2.85,1.331,5.919,2.278,8.664c.251,.668,.54,1.339,.968,1.915c.223,.297,.485,.574,.807,.764c.347,.21,.774,.28,1.167,.173c.518,-.136,.949,-.486,1.321,-.859c.757,-.776,1.345,-1.713,1.863,-2.66c1.043,-1.93,1.747,-4.044,2.27,-6.17c.437,-1.83,.797,-3.729,.912,-5.609c.022,-.479,.025,-.969,-.068,-1.441c-.071,-.363,-.214,-.719,-.456,-1.002C757.429,163.535,757.086,163.324,756.732,163.162z" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M740.04,164.529c-.667,-1.3,-2.032,-2.07,-3.405,-2.426c-1.051,-.266,-2.224,-.437,-3.241,.035c-.54,.249,-.996,.655,-1.371,1.112c-.617,.758,-1.025,1.665,-1.322,2.59c-.48,1.507,-.671,3.108,-.791,4.681c-.14,1.966,-.167,3.995,-.108,5.964c.027,.699,.053,1.462,.138,2.156c.037,.278,.08,.566,.17,.833c.061,.177,.148,.355,.301,.469c.242,.178,.596,.142,.877,.101c.649,-.114,1.247,-.426,1.798,-.776c1.114,-.726,2.084,-1.668,2.981,-2.643c1.176,-1.299,2.243,-2.724,3.081,-4.265c.862,-1.595,1.446,-3.381,1.456,-5.206C740.602,166.254,740.456,165.333,740.04,164.529z" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M731.157,159.924c-.142,-.944,-.395,-1.886,-.815,-2.746c-.173,-.336,-.371,-.682,-.685,-.904c-.501,-.344,-1.156,-.15,-1.694,-.001c-1.947,.6,-3.802,1.541,-5.412,2.792c-1.511,1.19,-2.796,2.664,-3.946,4.199c-.622,.849,-1.242,1.747,-1.727,2.682c-.196,.391,-.382,.802,-.447,1.238c-.049,.334,.004,.719,.264,.957c.204,.184,.483,.251,.749,.283c.395,.044,.802,.015,1.196,-.027c.698,-.078,1.43,-.223,2.117,-.374c2.361,-.534,4.696,-1.298,6.867,-2.378c.676,-.341,1.368,-.709,1.992,-1.139c.383,-.268,.744,-.581,1.006,-.972c.281,-.414,.442,-.9,.531,-1.39C731.282,161.414,731.26,160.658,731.157,159.924z" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M730.241,163.991c.231,.345,.493,.675,.778,.977" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M740.517,168.343c1.918,.181,3.934,.239,5.861,.241" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M772.705,166.686c9.512-1.237,19.19-2.99,28.056-6.761c1.677-0.744,3.369-1.562,4.85-2.654c1.039-0.774,2.018-1.765,2.356-3.051 c0.213-0.791,0.176-1.628,0.054-2.431c-0.375-2.314-1.475-4.443-2.64-6.451c-1.526-2.58-3.288-5.07-5.396-7.209 c-2.042-2.084-4.408-3.86-6.969-5.257c-2.734-1.49-5.696-2.622-8.665-3.549c-3.915-1.215-8.03-2.092-12.082-2.715 c-6.444-0.958-13.143-1.439-19.537,0.081c-2.355,0.562-4.659,1.418-6.789,2.573c-3.286,1.78-6.221,4.187-8.756,6.925 c-5.015,5.427-8.728,12.462-8.805,19.976" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M729.103,150.571c-2.374,.275,-4.767,.749,-6.963,1.716c-1.118,.497,-2.192,1.141,-3.091,1.976c-.62,.572,-1.15,1.246,-1.579,1.972c-.136,.238,-.285,.485,-.322,.761l-.004,.063v.012l.258,.424c.237,.142,.515,.197,.784,.243c1.948,.253,4.348,.066,6.32,.024" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M579.518,220.333c-9.685-7.681-18.592-16.864-23.935-28.138c-2.022-4.255-3.497-8.786-4.434-13.401 c-1.186-5.821-1.522-11.873-0.829-17.778c0.337-2.851,0.919-5.733,1.769-8.476c0.687-2.206,1.572-4.38,2.776-6.357 c1.442-2.4,3.403-4.463,5.593-6.196c2.02-1.629,4.264-3.007,6.678-3.969c5.612-2.228,11.757-2.772,17.741-3.026" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M569.708,243.892c-6.569-4.006-13.228-8.273-19.194-13.14c-3.125-2.59-6.171-5.404-8.462-8.779 c-0.648-0.963-1.241-2.004-1.73-3.057c-0.539-1.129-1.047-2.57-1.477-3.752c-1.227-3.445-2.343-7.138-3.223-10.688l-1.056-4.507 c-0.154-0.602-0.317-1.227-0.676-1.744c-0.517-0.72-1.343-1.181-2.08-1.647c-6.198-3.641-13.133-7.349-18.046-12.668 c-1.558-1.726-2.832-3.715-3.786-5.834c-0.397-0.904-0.793-1.849-1.049-2.804c-0.11-0.43-0.176-0.877-0.154-1.321 c0.019-0.436,0.124-0.868,0.279-1.275c0.301-0.782,0.757-1.507,1.233-2.193c0.887-1.262,1.945-2.454,3.008-3.572 c3.749-3.853,7.945-7.36,12.354-10.436c1.9-1.306,3.901-2.543,5.977-3.553c2.385-1.164,4.92-2.074,7.484-2.754 c1.131-0.306,2.606-0.622,3.743-0.906c0.241-0.06,0.514-0.144,0.748-0.23c0.248-0.093,0.492-0.21,0.709-0.363 c0.236-0.166,0.44-0.376,0.617-0.603c0.668-0.87,1.665-2.972,2.204-3.916c2.579-4.716,6.002-8.946,9.796-12.736 c3.328-3.331,6.956-6.429,10.781-9.178c4.851-3.501,10.193-6.394,15.834-8.399c8.219-2.913,17.025-4.25,25.738-4.004 c8.054,0.264,16.033,2.072,23.65,4.638c5.422,1.843,10.646,4.391,15.343,7.675c9.841,6.848,17.382,16.525,23.428,26.773" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M586.585,112.83c.382,-1.93,1.161,-3.772,2.126,-5.48c1.455,-2.571,3.361,-4.895,5.359,-7.063c.605,-.623,1.757,-1.923,2.383,-2.494c.201,-.184,.414,-.365,.644,-.51c.21,-.133,.441,-.238,.686,-.284c.251,-.049,.511,-.038,.762,.007c.326,.058,.646,.169,.955,.288c1.11,.447,2.28,1.052,3.355,1.583c6.318,3.19,12.154,7.367,17.321,12.196" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M558.634,170.101c4.416-10.365,12.988-18.311,22.04-24.705" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M600.741,132.363c7.341-6.367,16.308-10.64,25.557-13.451" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M628.667,156.629c1.205-4.34,0.016-8.459-4.198-10.53c-1.539-0.755-3.247-1.131-4.95-1.266c-2.377-0.185-4.784,0.086-7.104,0.611 c-8.172,1.895-16.581,7.274-21.297,14.265c-1.519,2.295-2.724,4.894-2.971,7.662c-0.127,1.471,0.055,2.99,0.662,4.344 c0.571,1.294,1.528,2.402,2.687,3.206c2.181,1.512,4.896,2.031,7.51,2.063c2.419,0.023,4.846-0.375,7.174-1.015 c3.285-0.912,6.447-2.336,9.357-4.108c3.771-2.315,7.228-5.253,9.87-8.817C626.83,161.105,628.024,158.957,628.667,156.629z" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M594.008,176.184c0.912-2.535,1.93-5.136,3.166-7.53c0.885-1.703,1.911-3.369,3.214-4.787c0.983-1.076,2.151-1.985,3.424-2.694 c1.037-0.581,2.169-1.07,3.272-1.513c1.766-0.702,3.622-1.242,5.514-1.458c2.069-0.243,4.184-0.093,6.203,0.416 c2.626,0.656,5.092,1.883,7.343,3.37" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M572.846,357.233c-.66,-.445,-1.353,-1.031,-2.167,-1.135c-.458,-.061,-.923,.038,-1.348,.207c-.661,.265,-1.253,.684,-1.8,1.135c-.774,.647,-1.476,1.389,-2.117,2.167c-1.405,1.728,-2.576,3.687,-3.255,5.816c-.121,.407,-.225,.828,-.247,1.253c-.009,.217,.004,.441,.086,.644c.055,.137,.145,.259,.261,.35c.143,.113,.318,.181,.493,.226c.202,.051,.414,.075,.622,.09c.91,.047,2.084,.017,2.999,.004c1.194,-.038,2.411,-.128,3.578,-.389c.994,-.224,1.968,-.579,2.867,-1.06c.878,-.471,1.704,-1.063,2.401,-1.776c.468,-.483,.885,-1.029,1.15,-1.651c.196,-.457,.29,-.961,.235,-1.457c-.079,-.757,-.442,-1.457,-.902,-2.052C574.97,358.635,573.84,357.915,572.846,357.233z" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M576.519,361.18c1.267,-.31,2.576,-.537,3.884,-.518c.999,.015,2.015,.19,2.912,.645c.477,.242,.949,.544,1.26,.988c.323,.459,.43,1.026,.53,1.568c.122,.751,.192,1.532,.034,2.283c-.061,.272,-.165,.538,-.331,.763c-.144,.198,-.334,.362,-.545,.486c-.386,.228,-.828,.342,-1.266,.42c-.983,.165,-2.039,.174,-3.035,.19c-.739,-.019,-2.05,.054,-2.77,-.072c-.347,-.056,-.694,-.158,-.994,-.346c-.909,-.564,-1.074,-1.739,-1.076,-2.724" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M584.74,362.573c1.875,-.748,3.815,-1.477,5.804,-1.846c1.263,-.217,2.697,-.322,3.809,.433c.44,.3,.803,.704,1.113,1.133c.482,.677,.831,1.448,1.052,2.249c.169,.645,.291,1.329,.181,1.994c-.039,.209,-.114,.415,-.244,.585c-.121,.16,-.287,.283,-.466,.371c-.458,.221,-.977,.283,-1.477,.342c-3.29,.308,-6.76,-.184,-10.031,-.588" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M595.483,362.317c1.346,-1.061,2.848,-1.998,4.524,-2.423c1.049,-.268,2.162,-.297,3.218,-.048c2.48,.607,4.651,2.313,5.912,4.523c.302,.531,.562,1.098,.715,1.691c.084,.339,.141,.693,.098,1.042c-.03,.248,-.13,.502,-.353,.636c-.136,.085,-.293,.13,-.449,.163c-.396,.078,-.827,.085,-1.23,.097c-2.509,.017,-5.181,.045,-7.668,-.273c-1.234,-.164,-2.484,-.405,-3.641,-.876" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M736.516,136.933c-26.102-0.012-51.769,5.309-75.367,16.561c-14.02,6.598-27.219,15.277-39.577,24.595 c-9.013,6.81-18.148,14.145-26.1,22.168c-4.934,4.984-9.581,10.396-13.495,16.219c-3.522,5.222-6.512,10.869-8.808,16.736 c-3.053,7.789-4.989,16.055-6.23,24.318c-1.587,10.743-2.034,21.773-1.661,32.62c0.452,12.022,1.796,24.138,4.801,35.805" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M607.505,362.259c7.962-1.924,15.69-5.109,23.013-8.749c25.337-12.624,47.715-30.858,69.191-49.099" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M609.81,367.528c3.234,0.248,7.015,0.364,10.265,0.421c13.666-0.119,237.538,0.351,242.43-0.195 c3.857-0.164,7.951-0.401,11.771-0.925c6.136-0.821,12.264-2.361,17.821-5.137c6.861-3.397,12.722-8.683,17.139-14.911 c3.921-5.503,6.661-11.846,8.025-18.461c1.357-6.427,1.634-13.108,1.576-19.663c-0.1-8.302-0.736-16.837-1.906-25.059 c-3.256-22.502-9.325-36.812-22.4-55.328c-7.599-10.767-16.062-21.231-25.588-30.358c-17.989-17.335-39.364-31.069-61.326-42.81" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M570.116,356.097c-.126,-.962,-.326,-1.938,-.771,-2.807c-.308,-.597,-.743,-1.13,-1.252,-1.567c-.256,-.212,-.564,-.374,-.899,-.403c-.606,-.051,-1.173,.253,-1.664,.576c-2.167,1.513,-4.331,3.216,-5.715,5.508c-.464,.773,-.799,1.653,-1.129,2.49c-.144,.393,-.306,.798,-.348,1.218c-.009,.126,-.013,.257,.018,.38c.023,.085,.078,.159,.148,.212c.157,.117,.354,.167,.542,.21c.372,.08,.766,.111,1.146,.136c1.12,.066,2.373,.042,3.497,.018" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M606.044,254.121c-8.741,15.947-19.35,35.407-27.168,51.703c-3.258,6.85-6.674,14.3-9.271,21.406 c-1.747,4.811-3.288,9.814-4.058,14.881c-0.476,3.228-0.672,6.568-0.016,9.784" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M603.503,192.69c5.245,25.333,18.267,46.395,38.664,62.312c14.94,11.69,32.475,19.766,50.435,25.625c21.984,7.159,45.138,11.212,68.187,12.735" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M869.921,284.039c6.912-2.319,13.784-5.413,19.2-10.396c3.164-2.896,5.687-6.468,7.506-10.345 c5.421-11.65,6.07-22.409-1.042-33.52" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M851.438,308.967c-2.69-12.389-9.663-17.774-22.036-19.239c-3.451-0.389-6.963-0.391-10.413,0.011 c-3.959,0.461-7.899,1.512-11.4,3.449c-5.121,2.781-9.013,7.339-12.082,12.213c-2.587,4.102-4.72,8.542-6.281,13.134 c-1.311,3.895-2.234,7.973-2.437,12.085c-0.397,7.518,1.639,15.678,7.427,20.833c3.278,2.949,7.428,4.774,11.675,5.829 c3.807,0.937,7.78,1.344,11.698,1.298c5.753-0.089,11.583-1.217,16.721-3.874c4.023-2.082,7.534-5.116,10.302-8.692 c2.233-2.881,4.005-6.128,5.273-9.544c1.228-3.306,1.984-6.798,2.385-10.298C852.901,320.455,852.655,314.592,851.438,308.967z" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M780.302,261.27c-2.602,-1.643,-5.427,-3.247,-8.179,-4.625c-1.143,-.549,-2.333,-1.152,-3.561,-1.486c-.433,-.108,-.897,-.188,-1.336,-.071c-.771,.207,-1.036,1.067,-1.181,1.765c-.289,1.536,-.186,3.129,-.013,4.674c.481,3.874,1.693,7.662,3.166,11.267c.664,1.581,1.403,3.164,2.301,4.627c.606,.975,1.305,1.919,2.201,2.646c.546,.443,1.174,.79,1.846,1.001c1.041,.325,2.153,.369,3.235,.319c2.454,-.144,4.967,-.889,6.841,-2.533c1.128,-.982,2.028,-2.224,2.632,-3.591c.617,-1.397,.913,-2.96,.702,-4.481c-.091,-.661,-.281,-1.311,-.564,-1.915c-.346,-.744,-.829,-1.423,-1.359,-2.047C785.13,264.604,782.755,262.829,780.302,261.27z" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M804.34,238.698c-1.493,-1.267,-3.155,-2.663,-4.745,-3.8c-.745,-.527,-1.53,-1.06,-2.345,-1.472c-.468,-.23,-.961,-.451,-1.486,-.494c-.302,-.024,-.619,.042,-.858,.236c-.382,.31,-.552,.8,-.677,1.261c-.291,1.173,-.304,2.406,-.289,3.608c.151,4.771,1.016,9.681,1.891,14.37c.573,2.785,1.152,5.673,2.139,8.344c.334,.845,.725,1.706,1.381,2.352c.96,.953,2.373,.944,3.638,.984c1.52,.024,3.058,-.076,4.56,-.315c1.216,-.199,2.438,-.49,3.574,-.975c2.17,-.918,3.999,-2.575,5.194,-4.599c.478,-.806,.85,-1.687,1.012,-2.614c.217,-1.242,.078,-2.553,-.44,-3.706c-.535,-1.2,-1.354,-2.26,-2.196,-3.26C811.571,245.06,807.904,241.809,804.34,238.698z" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M857.099,245.827c0.012-1.096,0.037-2.251-0.146-3.334c-0.067-0.371-0.16-0.746-0.342-1.079c-0.15-0.275-0.381-0.511-0.672-0.633 c-0.334-0.143-0.708-0.149-1.064-0.106c-1.191,0.16-2.283,0.729-3.31,1.326c-1.985,1.19-3.905,2.6-5.726,4.027 c-1.821,1.455-3.61,3.027-5.138,4.791c-0.785,0.91-1.506,1.895-2.058,2.964c-0.628,1.209-1.054,2.534-1.205,3.891 c-0.217,1.971,0.119,4.037,1.105,5.771c1.672,2.924,5.09,4.196,8.241,4.796c1.759,0.306,3.812,0.584,5.313-0.606 c0.767-0.599,1.314-1.434,1.745-2.296c0.848-1.731,1.304-3.647,1.695-5.526C856.423,255.227,856.971,250.496,857.099,245.827z" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M795.594,251.17c-5.16,2.97-9.976,6.605-14.172,10.833" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M770.494,275.619c-3.649,5.564-6.891,11.564-9.649,17.618c-3.751,8.277-6.527,17.124-7.276,26.213 c-0.426,5.165-0.248,10.438,0.462,15.57c0.621,4.407,1.638,8.832,3.188,13.008c1.279,3.435,2.975,6.753,5.238,9.648 c1.995,2.56,4.456,4.779,7.269,6.406c3.294,1.932,7.046,2.972,10.819,3.434c2.329,0.299,4.81,0.402,7.159,0.455 c6.526,0.009,33.892,0.179,40.183-0.121c3.649-0.203,7.333-0.733,10.782-1.978c5.625-2.019,10.535-5.696,14.684-9.942 c5.182-5.343,9.323-11.662,12.709-18.269c3.419-6.771,6.133-14.011,7.268-21.535c1.422-9.281,0.17-18.783-2.19-27.807 c-1.524-5.795-3.539-11.534-6.059-16.972c-2.4-5.166-5.347-10.148-9.014-14.519" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M844.687,246.968c-10.787-6.455-22.242-6.019-33.915-2.431" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M925.003,246.967c.563,-3.68,1.311,-7.418,2.906,-10.805c.628,-1.334,1.467,-2.625,2.275,-3.857c.539,-.795,1.086,-1.625,1.785,-2.289c1.023,-.96,1.921,-.793,2.727,.314c.446,.617,.763,1.327,1.043,2.032c.56,1.443,.983,2.996,1.37,4.495c.87,3.521,1.714,7.193,2.044,10.808c.301,3.229,.225,6.525,-.335,9.722" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M943.066,286.428c-.783,.897,-1.794,1.626,-2.952,1.942c-1.632,.457,-3.419,.072,-4.839,-.815c-1.346,-.838,-2.361,-2.117,-3.121,-3.491c-1.11,-2.042,-1.814,-4.309,-2.157,-6.604c-.274,-1.894,-.253,-3.904,.564,-5.669c.971,-2.122,2.719,-3.764,4.445,-5.281c2.48,-2.119,5.164,-4.076,7.922,-5.816c.759,-.458,1.545,-.939,2.386,-1.23c.533,-.183,1.171,-.27,1.645,.102c.22,.171,.373,.413,.486,.665c.171,.384,.267,.801,.344,1.213c.322,1.965,.33,4.02,.319,6.007c-.035,2.138,-.187,4.362,-.478,6.48c-.374,2.697,-1.003,5.387,-1.989,7.929C944.996,283.483,944.215,285.098,943.066,286.428z" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M899.536,356.999c1.088,2.46,2.266,4.958,3.787,7.182c0.844,1.204,1.813,2.391,3.106,3.131c0.559,0.318,1.18,0.525,1.816,0.614 c1.034,0.15,2.792,0.054,3.849,0.071c3.516-0.048,7.101-0.395,10.433-1.574c1.698-0.604,3.313-1.45,4.802-2.462 c3.758-2.577,6.842-6.06,9.476-9.75c2.902-4.107,5.247-8.652,7.056-13.341c1.806-4.702,3.104-9.659,4.124-14.587 c1.728-8.518,2.757-17.352,3.076-26.036c0.159-4.921,0.031-9.937-0.695-14.812c-0.556-3.772-1.497-7.535-2.793-11.121" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M941.52,261.603c-1.463-2.447-3.073-4.878-4.845-7.112c-1.311-1.64-2.77-3.201-4.471-4.441c-2.719-2.008-6.048-3.045-9.378-3.436 c-5.626-0.641-10.804,1.043-14.338,5.602" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M110.15,262.7c-7.605,2.792-14.783,6.838-21.032,12" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M67.141,333.186c0.975,5.414,2.824,10.715,5.713,15.411c4.242,6.9,10.599,12.512,18.019,15.776 c5.91,2.615,12.452,3.434,18.864,3.585c7.202,0.057,15.466,0.17,22.634-0.416c5.139-0.43,10.327-1.201,15.25-2.77 c7.972-2.485,15.161-7.199,21.018-13.11c2.99-3.025,5.67-6.43,7.824-10.099c3.623-6.161,5.917-13.063,7.276-20.059 c1.684-8.763,2.18-17.95-0.004-26.671c-1.203-4.846-3.255-9.476-5.921-13.692c-2.652-4.189-5.921-8.01-9.701-11.222 c-10.683-9.145-25.346-12.276-39.125-11.292" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M76.407,289.069c-5.293,8.387-8.684,18.008-9.77,27.868" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M129.82,263.78c-.432,1.683,-1.462,3.155,-2.644,4.403c-1.671,1.752,-3.811,3.093,-6.188,3.622c-1.237,.285,-2.526,.373,-3.79,.259c-1.334,-.115,-2.652,-.509,-3.79,-1.223c-.93,-.574,-1.823,-1.287,-2.341,-2.271c-.692,-1.291,-.818,-2.794,-.89,-4.233c-.103,-3.007,.146,-6.164,.362,-9.165c.181,-2.135,.398,-4.468,.751,-6.577c.142,-.823,.302,-1.671,.555,-2.468c.143,-.434,.312,-.872,.587,-1.241c.147,-.194,.331,-.365,.555,-.465c.579,-.255,1.226,-.018,1.769,.225c.735,.342,1.431,.798,2.096,1.258c1.249,.876,2.482,1.883,3.634,2.882c1.74,1.518,3.451,3.144,4.978,4.878c1.194,1.369,2.327,2.836,3.23,4.415c.568,1.008,1.036,2.096,1.219,3.244C130.044,262.137,130.019,262.979,129.82,263.78z" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M94.013,279.501c.118,2.276,-.17,4.576,-.636,6.802c-.445,1.975,-1.053,4.01,-2.386,5.58c-.896,1.075,-2.121,1.821,-3.409,2.337c-.881,.357,-1.805,.688,-2.728,.916c-.647,.156,-1.315,.278,-1.982,.268c-.464,-.005,-.935,-.096,-1.341,-.329c-.745,-.43,-1.308,-1.123,-1.864,-1.766c-1.571,-1.903,-3.142,-4.025,-4.539,-6.06c-.996,-1.471,-1.995,-3.022,-2.794,-4.61c-.972,-1.932,-1.709,-4.041,-1.752,-6.221c-.015,-.749,.059,-1.526,.398,-2.204c.709,-1.421,2.381,-1.91,3.836,-2.165c3.695,-.595,7.476,.138,10.971,1.363c1.596,.55,3.27,1.247,4.814,1.934c.471,.217,1.015,.449,1.465,.705c.491,.274,.963,.615,1.284,1.084C93.827,277.824,93.954,278.683,94.013,279.501z" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M79.687,318.906c.575,1.614,.87,3.332,.983,5.038c.099,1.666,-.017,3.394,-.713,4.932c-.569,1.294,-1.507,2.398,-2.595,3.288c-.945,.78,-2.017,1.429,-3.188,1.804c-2.516,.824,-5.325,.281,-7.555,-1.077c-1.51,-.913,-2.789,-2.173,-3.964,-3.477c-1.415,-1.622,-2.804,-3.351,-3.979,-5.156c-.373,-.602,-.723,-1.236,-.929,-1.916c-.131,-.435,-.199,-.899,-.125,-1.351c.093,-.595,.449,-1.117,.895,-1.509c.638,-.562,1.423,-.932,2.211,-1.233c2.744,-.986,5.754,-1.323,8.642,-1.619c1.763,-.157,3.621,-.27,5.391,-.26c.758,.01,1.541,.031,2.288,.165c.585,.106,1.176,.302,1.627,.703C79.175,317.676,79.461,318.296,79.687,318.906z" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M166.982,323.554c-0.236-2.51-0.922-4.996-2.148-7.207c-2.032-3.646-5.35-6.585-9.239-8.112c-3.683-1.458-7.763-1.612-11.65-1.044 c-3.092,0.447-6.136,1.294-9.046,2.425c-4.691,1.843-9.162,4.454-12.741,8.034c-2.609,2.626-4.69,5.775-6.183,9.158 c-3.134,7.118-4.332,16.854,0.16,23.636c1.278,1.925,3.023,3.535,5.044,4.655c1.807,1.007,3.814,1.64,5.841,2.025 c2.408,0.452,4.883,0.564,7.327,0.433c4.054-0.229,8.11-1.118,11.798-2.839c3.125-1.444,5.964-3.483,8.52-5.776 c4.045-3.655,7.692-7.908,10.057-12.853C166.554,332.209,167.404,327.834,166.982,323.554z" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M169.255,153.797c-1.314,-8.784,-1.405,-17.926,.908,-26.552c.551,-2.002,1.27,-4.008,2.105,-5.91c.898,-2.116,2.532,-5.089,3.607,-7.147c1.265,-2.445,2.281,-5.079,2.545,-7.836c.165,-1.606,.11,-3.248,-.041,-4.854c-.259,-3.072,-1.533,-9.478,-2.056,-12.586c-.585,-3.552,-1.062,-7.21,-1.049,-10.811c.018,-4.736,.71,-9.503,1.942,-14.073c.774,-2.741,2.063,-5.874,4.983,-6.862c.981,-.338,2.034,-.427,3.066,-.4c1.59,.046,3.181,.36,4.72,.749c3.984,1.033,7.968,2.686,11.764,4.275c3.65,1.578,13.819,6.161,17.333,7.453c1.779,.661,3.643,1.281,5.497,1.685c1.987,.417,4.035,.74,6.069,.546c.752,-.063,1.573,-.187,2.319,-.321c3.044,-.576,7.739,-1.684,10.775,-2.197c5.422,-.966,11.007,-1.488,16.515,-1.479c7.587,.063,15.332,.776,22.805,2.089c1.512,.287,3.119,.648,4.597,1.076c.613,.177,2.356,.74,3.002,.838c.466,.081,.947,.105,1.417,.05c.499,-.056,.988,-.196,1.455,-.376c1.238,-.486,2.416,-1.224,3.553,-1.91l5.928,-3.749c2.724,-1.694,5.613,-3.232,8.607,-4.385c4.608,-1.768,9.5,-2.892,14.343,-3.798c3.405,-.61,6.909,-1.137,10.366,-1.309c1.76,-.071,3.56,-.082,5.292,.275c.803,.172,1.601,.464,2.244,.986c.725,.58,1.21,1.402,1.563,2.248c.387,.929,.649,1.932,.872,2.913c.405,1.81,.68,3.758,.893,5.603c.839,7.427,.657,15.018,-.046,22.45c-.2,2.519,-.799,6.94,-1.108,9.479c-.089,.73,-.172,1.592,-.208,2.328c-.071,1.358,.03,2.735,.355,4.057c.667,2.859,3.661,9.637,4.709,12.425c1.925,4.933,3.006,10.202,3.585,15.455c.621,5.498,.79,11.284,.795,16.822" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M259.846,92.283c5.123-3.385,10.682-6.191,16.554-8.015c4.973-1.558,10.196-2.381,15.404-2.508 c7.482-0.176,15.17,0.75,21.996,3.97c9.327,4.336,16.117,12.581,21.428,21.163c7.686,12.443,12.028,24.189,9.584,39.01" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M180.323,148.235c-0.003-8.617,0.304-17.451,1.603-25.977c1.17-7.55,3.192-15.161,7.34-21.656 c3.373-5.247,7.879-9.803,13.163-13.128c4.647-2.929,9.901-4.854,15.29-5.863c15.302-2.863,30.04,0.851,42.126,10.671" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M120.405,112.22c1.041,-3.725,2.925,-7.458,6.229,-9.646c.57,-.367,1.162,-.738,1.797,-.979c.279,-.099,.617,-.156,.864,.046c.273,.227,.398,.608,.51,.934c.558,1.87,.923,3.823,1.067,5.769c.222,3.012,-.053,6.074,-.611,9.037" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M110.518,110.549c-.094,-2.035,.023,-4.099,.469,-6.089c.378,-1.737,1.052,-3.411,1.859,-4.991c.346,-.654,.688,-1.348,1.217,-1.875c.229,-.219,.531,-.388,.854,-.281c.164,.052,.305,.159,.426,.277c.228,.225,.405,.498,.568,.772c.737,1.326,1.29,2.772,1.742,4.218c.887,2.944,1.182,6.052,1.212,9.116" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M100.488,114.091c-1.487,-2.537,-2.893,-5.202,-3.589,-8.077c-.433,-1.86,-.522,-3.845,-.015,-5.699c.219,-.748,.545,-1.502,1.117,-2.049c.274,-.261,.622,-.435,1.007,-.412c.324,.016,.632,.141,.916,.293c.537,.294,1.018,.694,1.474,1.098c3.409,3.097,5.703,7.242,7.362,11.493" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M92.281,121.703c-3.414,-3.696,-5.998,-8.323,-6.704,-13.348c-.038,-.359,-.06,-.726,-.02,-1.085c.025,-.209,.071,-.42,.174,-.605c.114,-.21,.313,-.357,.551,-.39c.193,-.029,.39,.007,.577,.059c.911,.289,1.772,.852,2.595,1.331c2.458,1.508,4.991,3.119,7.243,4.921c.892,.73,1.791,1.502,2.493,2.422" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M84.866,132.965c-2.422,-2.47,-4.635,-5.252,-5.958,-8.475c-.653,-1.571,-1.014,-3.258,-1.176,-4.948c-.043,-.473,-.099,-.963,-.021,-1.434c.028,-.154,.077,-.309,.173,-.435c.08,-.107,.194,-.184,.32,-.227c.16,-.056,.332,-.066,.5,-.063c.317,.009,.637,.07,.946,.136c1.625,.368,3.212,.958,4.733,1.633c2.377,1.066,4.649,2.43,6.706,4.029" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M98.692,226.982c-9.446-20.953-16.371-43.44-17.601-66.495c-0.319-7.096-0.439-14.393,1.399-21.308 c0.801-2.992,2.079-5.86,3.621-8.542c1.313-2.297,2.799-4.567,4.388-6.683c2.33-3.087,4.97-6.009,8.005-8.418 c2.888-2.298,6.253-4.115,9.919-4.742c4.533-0.81,9.19,0.229,13.386,1.975c6.669,2.736,12.443,7.276,17.756,12.071 c9.926,9.076,19.158,19.705,27.91,29.913" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M277.187,102.601c10.721,-.181,21.576,.972,32.014,3.43" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M203.675,105.896c11.909-2.061,24.178-2.973,36.251-2.219" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M283.442,112.958c2.49,2.849,4.841,5.951,6.218,9.505" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M278.684,121.723c1.348-3.029,2.929-5.998,4.757-8.765" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M227.901,122.716c2.081-4.109,4.847-7.935,8.394-10.901" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M227.901,122.716c20.409-2.539,41.296-1.776,61.759-0.253" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M171.661,152.532c-12.671,8.89-25.209,18.573-35.109,30.56c-4.211,5.162-7.892,10.881-10.226,17.145 c-1.195,3.19-2.032,6.545-2.575,9.906c-0.509,3.152-0.759,6.383-0.558,9.574c0.179,2.872,0.745,5.743,1.757,8.439 c0.953,2.559,2.309,4.972,3.9,7.186c3.448,4.8,7.994,8.835,13.222,11.602c3.509,1.87,7.365,3.074,11.301,3.618 c3.655,0.518,7.409,0.544,11.089,0.337c18.025-0.975,42.681-7.717,60.434-11.628c10.043-2.152,20.366-4.171,30.663-4.121 c3.895-0.002,8.092,0.267,11.978,0.64c12.684,1.152,49.056,7.129,62.017,8.718c5.148,0.614,10.508,1.176,15.692,1.151 c8.272,0.009,16.704-1.895,23.712-6.409c3.416-2.194,6.451-5.025,8.778-8.358c4.053-5.852,6.151-12.877,7.051-19.884 c0.603-4.925,0.601-9.979-0.288-14.869c-0.81-4.534-2.388-8.941-4.326-13.11c-2.791-5.938-6.161-11.689-10.27-16.815 c-6.556-8.227-15.01-14.813-24.181-19.897" stroke-width="1.7" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M899.536,356.999c1.088,2.46,2.266,4.958,3.787,7.182c.844,1.204,1.813,2.391,3.106,3.131c.559,.318,1.18,.525,1.816,.614c1.034,.15,2.792,.054,3.849,.071c3.516,-.048,7.101,-.395,10.433,-1.574c1.698,-.604,3.313,-1.45,4.802,-2.462c3.758,-2.577,6.842,-6.06,9.476,-9.75c2.902,-4.107,5.247,-8.652,7.056,-13.341c1.806,-4.702,3.104,-9.659,4.124,-14.587c1.728,-8.518,2.757,-17.352,3.076,-26.036c.159,-4.921,.031,-9.937,-.695,-14.812c-.556,-3.772,-1.497,-7.535,-2.793,-11.121c.409,-2.816,.572,-5.72,.533,-8.564c-.04,-1.539,-.075,-3.13,-.386,-4.641c-.157,-.672,-.4,-1.502,-1.13,-1.738c-.486,-.156,-1.005,-.013,-1.467,.159c-1.277,.515,-2.454,1.322,-3.604,2.07c-.814,-1.362,-1.8,-2.915,-2.703,-4.215c.56,-3.197,.636,-6.493,.335,-9.722c-.329,-3.614,-1.173,-7.288,-2.044,-10.808c-.387,-1.499,-.809,-3.051,-1.37,-4.495c-.281,-.705,-.597,-1.415,-1.043,-2.032c-.781,-1.072,-1.662,-1.285,-2.683,-.354c-.625,.575,-1.121,1.297,-1.603,1.993c-.827,1.246,-1.689,2.551,-2.353,3.892c-1.699,3.463,-2.471,7.318,-3.053,11.106c-6.293,-1.336,-12.443,-.004,-16.515,5.249c-2.363,-5.403,-5.281,-10.677,-8.431,-15.659c-1.377,-2.214,-3.05,-4.598,-4.473,-6.779c-7.334,-10.524,-15.461,-20.813,-24.583,-29.847c-18.406,-18.3,-40.579,-32.634,-63.384,-44.829c.449,-.822,.563,-1.779,.477,-2.702c-.109,-1.288,-.506,-2.543,-1.004,-3.731c-.774,-1.829,-1.82,-3.579,-2.898,-5.245c-2.414,-3.742,-5.548,-7.058,-9.318,-9.452c-2.816,-1.809,-5.953,-3.126,-9.116,-4.196c-4.36,-1.461,-8.945,-2.47,-13.488,-3.17c-3.529,-.528,-7.186,-.909,-10.756,-.894c-4.51,.014,-9.088,.666,-13.261,2.437c-4.541,1.93,-8.491,5.097,-11.737,8.782c-22.076,-.005,-44.272,3.825,-64.818,11.985c-6.949,-11.804,-15.983,-22.898,-28.071,-29.692c-4.11,-2.334,-8.557,-4.133,-13.068,-5.528c-3.371,-1.044,-6.915,-1.965,-10.383,-2.625c-5.385,-5.036,-11.497,-9.352,-18.123,-12.597c-.812,-.391,-1.717,-.852,-2.552,-1.182c-.322,-.124,-.657,-.239,-.998,-.295c-.243,-.04,-.495,-.046,-.737,.004c-.238,.048,-.463,.151,-.667,.281c-.231,.146,-.443,.326,-.644,.51c-.889,.849,-2.123,2.222,-2.961,3.127c-3.061,3.428,-5.962,7.322,-6.907,11.909c-4.233,1.281,-8.383,3.025,-12.247,5.176c-4.773,2.667,-9.225,5.963,-13.329,9.569c-5.41,4.783,-10.401,10.206,-13.882,16.574c-.536,.94,-1.573,3.106,-2.204,3.916c-.177,.227,-.38,.437,-.617,.603c-.217,.154,-.46,.27,-.709,.363c-.234,.086,-.504,.167,-.748,.23c-1.139,.285,-2.612,.6,-3.743,.906c-2.565,.68,-5.099,1.59,-7.484,2.754c-2.075,1.01,-4.076,2.246,-5.977,3.553c-4.408,3.076,-8.604,6.582,-12.354,10.436c-1.062,1.119,-2.121,2.309,-3.008,3.572c-.491,.709,-.961,1.456,-1.261,2.268c-.151,.417,-.248,.858,-.255,1.302c-.009,.462,.074,.923,.197,1.366c.968,3.123,2.608,6.058,4.796,8.491c4.902,5.321,11.86,9.029,18.046,12.669c.737,.466,1.563,.927,2.08,1.647c.359,.517,.522,1.142,.676,1.744l1.056,4.507c.881,3.551,1.996,7.243,3.223,10.688c.435,1.19,.937,2.62,1.477,3.752c.49,1.053,1.082,2.094,1.73,3.057c2.291,3.375,5.336,6.189,8.462,8.779c5.965,4.867,12.626,9.135,19.194,13.14c-4.059,15.623,-5.016,31.985,-4.359,48.067c.531,11.4,1.871,22.934,4.73,33.995c-1.895,5.064,-3.587,10.319,-4.455,15.664c-.535,3.386,-.781,6.896,-.093,10.278c-2.167,1.513,-4.331,3.216,-5.715,5.508c-.464,.773,-.799,1.653,-1.129,2.49c-.144,.393,-.306,.798,-.348,1.218c-.009,.126,-.013,.257,.018,.38c.023,.085,.078,.159,.148,.212c.157,.117,.354,.167,.542,.21c.372,.08,.766,.111,1.146,.136c1.12,.066,2.373,.042,3.497,.018c-.455,.772,-.865,1.586,-1.198,2.417c-.229,.581,-.436,1.186,-.537,1.804c-.054,.37,-.091,.777,.087,1.12c.167,.322,.533,.462,.872,.522c.505,.091,1.088,.073,1.601,.078c1.565,-.002,3.226,-.015,4.769,-.277c2.166,-.362,4.265,-1.322,5.84,-2.868c.002,.973,.16,2.131,1.045,2.705c.274,.179,.591,.284,.911,.345c.858,.156,2.007,.077,2.883,.092c1.088,-.019,2.233,-.02,3.302,-.242c.431,-.096,.866,-.241,1.217,-.517c2.896,.37,5.923,.759,8.842,.672c.748,-.037,1.516,-.076,2.243,-.268c.249,-.071,.497,-.17,.699,-.334c.152,-.123,.27,-.285,.343,-.466c1.055,.43,2.19,.666,3.314,.831c2.333,.337,4.859,.33,7.217,.326c.567,-.008,1.239,.001,1.798,-.071c.147,-.02,.298,-.047,.439,-.095c.172,-.057,.337,-.156,.434,-.314c3.234,.248,7.015,.364,10.265,.421c61.187,-.013,175.522,.165,235.189,-.015c5.273,-.093,11.094,-.237,16.32,-.784c4.215,-.438,8.463,-1.164,12.534,-2.356C889.674,363.18,894.983,360.582,899.536,356.999z" stroke-width="2.2" vector-effect="non-scaling-stroke"/>'
        html = html .. '<path d="M366.949,365.395c5.426,1.47,11.11,2.082,16.712,2.388c5.83,0.278,13.212,0.249,19.055,0.176 c6.278-0.15,12.679-0.933,18.493-3.426c7.579-3.237,14.077-8.923,18.39-15.936c2.889-4.696,4.738-9.997,5.713-15.411 c1.193-0.629,2.253-1.499,3.215-2.439c1.537-1.508,2.907-3.248,4.199-4.969c0.707-0.971,1.43-1.979,1.867-3.106 c0.253-0.664,0.387-1.424,0.124-2.106c-0.405-1.037-1.444-1.644-2.412-2.082c-2.047-0.876-4.297-1.232-6.489-1.548 c-0.524-4.822-1.626-9.618-3.21-14.201c1.37-0.613,2.698-1.36,3.925-2.225c0.901,1.591,1.865,3.197,3.088,4.564 c0.493,0.539,1.042,1.047,1.685,1.401c0.541,0.291,1.161,0.489,1.777,0.514c0.338,0.009,0.688-0.066,0.963-0.271 c0.303-0.222,0.492-0.564,0.615-0.912c0.173-0.495,0.243-1.026,0.289-1.547c0.071-0.884,0.052-1.806,0.012-2.691 c-0.135-2.829-0.492-5.738-1.8-8.289c0.122-0.202,0.361-0.615,0.477-0.821c1.753,1.262,3.589,2.481,5.57,3.357 c1.022,0.445,2.107,0.79,3.226,0.851c0.305,0.015,0.618,0.017,0.916-0.06c0.203-0.053,0.398-0.16,0.518-0.337 c0.203-0.308,0.237-0.732,0.215-1.092c-0.063-0.776-0.333-1.529-0.596-2.257c-0.853-2.33-2.145-4.487-3.678-6.432 c2.088-0.102,4.653-0.193,6.71-0.439c0.411-0.052,0.849-0.112,1.252-0.205c0.273-0.068,0.564-0.136,0.789-0.313 c0.116-0.092,0.177-0.233,0.222-0.37c0.189-0.608-0.049-1.266-0.25-1.842c-0.414-1.129-1.076-2.154-1.806-3.104 c-1.226-1.622-2.766-3.003-4.498-4.065c1.28,0.077,2.761,0.189,4.035,0.165c0.282-0.008,0.577-0.02,0.855-0.071 c0.211-0.037,0.424-0.109,0.583-0.258c0.092-0.084,0.147-0.202,0.165-0.324c0.043-0.342-0.052-0.686-0.13-1.016 c-0.399-1.467-0.921-2.927-1.729-4.222c-1.837-2.953-4.607-5.197-7.451-7.136c-0.561-6.526-1.897-13.065-4.247-19.189 c-1.594-4.153-3.734-8.14-6.093-11.907c-1.764-2.836-4.052-6.117-5.989-8.861c-4.821-6.826-10.119-13.638-15.805-19.766 c-7.686-8.322-16.409-15.897-25.527-22.612c-4.676-3.424-9.618-6.728-14.673-9.566c-3.445-1.912-7.07-3.721-10.814-4.968 c-3.007-3.069-6.23-6.081-9.652-8.681c-3.273-2.506-6.858-4.728-10.48-6.696c-0.005-5.543-0.173-11.321-0.795-16.822 c-0.58-5.253-1.66-10.522-3.585-15.455c-1.068-2.836-4.057-9.612-4.709-12.425c-0.325-1.322-0.426-2.698-0.354-4.057 c0.037-0.736,0.12-1.591,0.208-2.328c0.33-2.573,0.886-6.956,1.108-9.478c0.702-7.433,0.885-15.023,0.046-22.45 c-0.214-1.845-0.487-3.792-0.893-5.603c-0.224-0.98-0.485-1.984-0.872-2.913c-0.353-0.846-0.838-1.669-1.563-2.248 c-0.643-0.522-1.441-0.815-2.244-0.986c-1.732-0.357-3.532-0.346-5.292-0.275c-3.457,0.172-6.962,0.698-10.366,1.309 c-4.843,0.906-9.736,2.029-14.343,3.797c-2.993,1.153-5.885,2.691-8.607,4.385l-5.928,3.749c-1.143,0.686-2.313,1.423-3.553,1.91 c-0.467,0.181-0.956,0.32-1.455,0.376c-0.471,0.054-0.951,0.031-1.417-0.05c-0.667-0.105-2.319-0.641-3.002-0.838 c-1.479-0.427-3.083-0.789-4.597-1.076c-7.473-1.314-15.218-2.026-22.805-2.089c-5.507-0.008-11.094,0.513-16.515,1.479 c-3.038,0.511-7.737,1.622-10.775,2.197c-0.741,0.134-1.567,0.258-2.319,0.321c-2.193,0.205-4.402-0.173-6.537-0.653 c-1.978-0.468-3.971-1.167-5.866-1.898c-3.004-1.125-13.43-5.824-16.495-7.134c-3.8-1.59-7.778-3.241-11.764-4.275 c-1.603-0.405-3.259-0.729-4.917-0.753c-0.967-0.01-1.951,0.087-2.869,0.404c-2.92,0.989-4.208,4.12-4.983,6.862 c-1.231,4.57-1.924,9.337-1.942,14.073c-0.013,3.6,0.464,7.262,1.049,10.811c0.529,3.155,1.792,9.488,2.056,12.586 c0.151,1.606,0.206,3.247,0.041,4.854c-0.264,2.758-1.28,5.392-2.545,7.836c-1.085,2.084-2.704,5.023-3.607,7.147 c-0.834,1.902-1.554,3.907-2.104,5.91c-2.313,8.627-2.222,17.768-0.908,26.552l-1.78,0.956 c-7.062-8.254-14.921-17.255-22.608-24.873c-4.541-4.47-9.337-8.899-14.606-12.497c0.558-2.963,0.833-6.025,0.611-9.037 c-0.144-1.947-0.51-3.899-1.067-5.769c-0.113-0.326-0.237-0.707-0.51-0.934c-0.256-0.21-0.608-0.141-0.893-0.036 c-0.624,0.243-1.207,0.607-1.768,0.969c-3.304,2.189-5.188,5.921-6.229,9.646c-0.478-0.175-1.053-0.372-1.539-0.524 c-0.034-3.432-0.4-6.92-1.558-10.168c-0.403-1.095-0.845-2.21-1.429-3.222c-0.158-0.26-0.33-0.519-0.55-0.731 c-0.118-0.112-0.254-0.213-0.411-0.263c-0.338-0.111-0.651,0.078-0.885,0.311c-0.51,0.525-0.848,1.203-1.186,1.845 c-0.807,1.58-1.482,3.253-1.859,4.991c-0.446,1.99-0.563,4.055-0.469,6.089c-0.58,0.035-1.183,0.1-1.757,0.189 c-1.659-4.251-3.954-8.396-7.362-11.493c-0.457-0.405-0.937-0.805-1.474-1.098c-0.283-0.152-0.591-0.277-0.916-0.293 c-0.385-0.023-0.733,0.151-1.007,0.412c-0.572,0.547-0.898,1.301-1.117,2.049c-0.52,1.904-0.414,3.942,0.05,5.847 c0.711,2.818,2.091,5.435,3.554,7.929c-0.414,0.276-0.899,0.619-1.297,0.917c-0.702-0.92-1.601-1.692-2.493-2.422 c-2.252-1.802-4.786-3.414-7.243-4.921c-0.823-0.479-1.684-1.042-2.595-1.331c-0.186-0.052-0.384-0.088-0.577-0.059 c-0.245,0.035-0.447,0.188-0.56,0.407c-0.101,0.19-0.145,0.404-0.168,0.616c-0.036,0.35-0.015,0.708,0.023,1.058 c0.706,5.025,3.289,9.652,6.703,13.348c-0.303,0.363-0.9,1.106-1.191,1.479c-2.057-1.599-4.329-2.963-6.706-4.029 c-1.521-0.675-3.108-1.265-4.733-1.633c-0.31-0.066-0.629-0.127-0.946-0.136c-0.168-0.004-0.34,0.007-0.5,0.063 c-0.129,0.044-0.246,0.125-0.326,0.236c-0.091,0.124-0.138,0.275-0.166,0.426c-0.078,0.471-0.022,0.961,0.021,1.434 c0.162,1.69,0.522,3.377,1.176,4.948c1.323,3.223,3.536,6.005,5.958,8.475c-1.155,2.317-2.056,4.777-2.652,7.296 c-1.331,5.66-1.408,11.568-1.244,17.354c0.777,24.047,7.886,47.549,17.723,69.367c-5.462,15.082-8.064,31.155-8.865,47.139 l-0.709,0.579c-4.092-1.729-8.498-3.162-12.995-2.808c-0.827,0.065-1.66,0.187-2.459,0.414c-0.542,0.157-1.079,0.364-1.554,0.672 c-0.536,0.342-0.977,0.838-1.219,1.429c-0.329,0.792-0.341,1.674-0.287,2.517c0.134,2.004,0.833,3.932,1.729,5.715 c1.142,2.248,2.609,4.381,4.073,6.43c-5.293,8.387-8.684,18.008-9.77,27.868c-2.192,0.316-4.441,0.671-6.489,1.548 c-0.969,0.438-2.007,1.045-2.412,2.082c-0.263,0.682-0.129,1.441,0.124,2.106c0.436,1.126,1.16,2.135,1.867,3.106 c1.292,1.721,2.661,3.46,4.199,4.969c0.962,0.939,2.022,1.809,3.215,2.439c0.839,4.658,2.323,9.23,4.561,13.407 c2.812,5.248,6.828,9.847,11.613,13.386c4.343,3.231,9.369,5.545,14.661,6.693c5.577,1.267,11.562,1.342,17.266,1.337 c8.143-0.004,17.045-0.018,25.059-1.459c4.666,0.761,9.575,1.109,14.302,1.277c7.967,0.329,183.719,0.177,190.946,0.118 C352.716,367.811,359.989,367.179,366.949,365.395z" stroke-width="2.2" vector-effect="non-scaling-stroke"/>'
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
