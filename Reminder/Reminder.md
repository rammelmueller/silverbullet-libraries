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
  color: oklch(from var(--modal-help-background-color) calc(l - 0.14) c h);
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
        html = html .. '<svg class="rem-empty-art" viewBox="497 17 466 363" xmlns="http://www.w3.org/2000/svg" fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round">'
        html = html .. '<text x="575" y="79" font-size="30" fill="currentColor" stroke="none" opacity="1">Z</text>'
        html = html .. '<text x="605" y="53" font-size="22" fill="currentColor" stroke="none" opacity="0.85">z</text>'
        html = html .. '<text x="628" y="31" font-size="15" fill="currentColor" stroke="none" opacity="0.7">z</text>'
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
        html = html .. '<path d="M899.536,356.999c1.088,2.46,2.266,4.958,3.787,7.182c.844,1.204,1.813,2.391,3.106,3.131c.559,.318,1.18,.525,1.816,.614c1.034,.15,2.792,.054,3.849,.071c3.516,-.048,7.101,-.395,10.433,-1.574c1.698,-.604,3.313,-1.45,4.802,-2.462c3.758,-2.577,6.842,-6.06,9.476,-9.75c2.902,-4.107,5.247,-8.652,7.056,-13.341c1.806,-4.702,3.104,-9.659,4.124,-14.587c1.728,-8.518,2.757,-17.352,3.076,-26.036c.159,-4.921,.031,-9.937,-.695,-14.812c-.556,-3.772,-1.497,-7.535,-2.793,-11.121c.409,-2.816,.572,-5.72,.533,-8.564c-.04,-1.539,-.075,-3.13,-.386,-4.641c-.157,-.672,-.4,-1.502,-1.13,-1.738c-.486,-.156,-1.005,-.013,-1.467,.159c-1.277,.515,-2.454,1.322,-3.604,2.07c-.814,-1.362,-1.8,-2.915,-2.703,-4.215c.56,-3.197,.636,-6.493,.335,-9.722c-.329,-3.614,-1.173,-7.288,-2.044,-10.808c-.387,-1.499,-.809,-3.051,-1.37,-4.495c-.281,-.705,-.597,-1.415,-1.043,-2.032c-.781,-1.072,-1.662,-1.285,-2.683,-.354c-.625,.575,-1.121,1.297,-1.603,1.993c-.827,1.246,-1.689,2.551,-2.353,3.892c-1.699,3.463,-2.471,7.318,-3.053,11.106c-6.293,-1.336,-12.443,-.004,-16.515,5.249c-2.363,-5.403,-5.281,-10.677,-8.431,-15.659c-1.377,-2.214,-3.05,-4.598,-4.473,-6.779c-7.334,-10.524,-15.461,-20.813,-24.583,-29.847c-18.406,-18.3,-40.579,-32.634,-63.384,-44.829c.449,-.822,.563,-1.779,.477,-2.702c-.109,-1.288,-.506,-2.543,-1.004,-3.731c-.774,-1.829,-1.82,-3.579,-2.898,-5.245c-2.414,-3.742,-5.548,-7.058,-9.318,-9.452c-2.816,-1.809,-5.953,-3.126,-9.116,-4.196c-4.36,-1.461,-8.945,-2.47,-13.488,-3.17c-3.529,-.528,-7.186,-.909,-10.756,-.894c-4.51,.014,-9.088,.666,-13.261,2.437c-4.541,1.93,-8.491,5.097,-11.737,8.782c-22.076,-.005,-44.272,3.825,-64.818,11.985c-6.949,-11.804,-15.983,-22.898,-28.071,-29.692c-4.11,-2.334,-8.557,-4.133,-13.068,-5.528c-3.371,-1.044,-6.915,-1.965,-10.383,-2.625c-5.385,-5.036,-11.497,-9.352,-18.123,-12.597c-.812,-.391,-1.717,-.852,-2.552,-1.182c-.322,-.124,-.657,-.239,-.998,-.295c-.243,-.04,-.495,-.046,-.737,.004c-.238,.048,-.463,.151,-.667,.281c-.231,.146,-.443,.326,-.644,.51c-.889,.849,-2.123,2.222,-2.961,3.127c-3.061,3.428,-5.962,7.322,-6.907,11.909c-4.233,1.281,-8.383,3.025,-12.247,5.176c-4.773,2.667,-9.225,5.963,-13.329,9.569c-5.41,4.783,-10.401,10.206,-13.882,16.574c-.536,.94,-1.573,3.106,-2.204,3.916c-.177,.227,-.38,.437,-.617,.603c-.217,.154,-.46,.27,-.709,.363c-.234,.086,-.504,.167,-.748,.23c-1.139,.285,-2.612,.6,-3.743,.906c-2.565,.68,-5.099,1.59,-7.484,2.754c-2.075,1.01,-4.076,2.246,-5.977,3.553c-4.408,3.076,-8.604,6.582,-12.354,10.436c-1.062,1.119,-2.121,2.309,-3.008,3.572c-.491,.709,-.961,1.456,-1.261,2.268c-.151,.417,-.248,.858,-.255,1.302c-.009,.462,.074,.923,.197,1.366c.968,3.123,2.608,6.058,4.796,8.491c4.902,5.321,11.86,9.029,18.046,12.669c.737,.466,1.563,.927,2.08,1.647c.359,.517,.522,1.142,.676,1.744l1.056,4.507c.881,3.551,1.996,7.243,3.223,10.688c.435,1.19,.937,2.62,1.477,3.752c.49,1.053,1.082,2.094,1.73,3.057c2.291,3.375,5.336,6.189,8.462,8.779c5.965,4.867,12.626,9.135,19.194,13.14c-4.059,15.623,-5.016,31.985,-4.359,48.067c.531,11.4,1.871,22.934,4.73,33.995c-1.895,5.064,-3.587,10.319,-4.455,15.664c-.535,3.386,-.781,6.896,-.093,10.278c-2.167,1.513,-4.331,3.216,-5.715,5.508c-.464,.773,-.799,1.653,-1.129,2.49c-.144,.393,-.306,.798,-.348,1.218c-.009,.126,-.013,.257,.018,.38c.023,.085,.078,.159,.148,.212c.157,.117,.354,.167,.542,.21c.372,.08,.766,.111,1.146,.136c1.12,.066,2.373,.042,3.497,.018c-.455,.772,-.865,1.586,-1.198,2.417c-.229,.581,-.436,1.186,-.537,1.804c-.054,.37,-.091,.777,.087,1.12c.167,.322,.533,.462,.872,.522c.505,.091,1.088,.073,1.601,.078c1.565,-.002,3.226,-.015,4.769,-.277c2.166,-.362,4.265,-1.322,5.84,-2.868c.002,.973,.16,2.131,1.045,2.705c.274,.179,.591,.284,.911,.345c.858,.156,2.007,.077,2.883,.092c1.088,-.019,2.233,-.02,3.302,-.242c.431,-.096,.866,-.241,1.217,-.517c2.896,.37,5.923,.759,8.842,.672c.748,-.037,1.516,-.076,2.243,-.268c.249,-.071,.497,-.17,.699,-.334c.152,-.123,.27,-.285,.343,-.466c1.055,.43,2.19,.666,3.314,.831c2.333,.337,4.859,.33,7.217,.326c.567,-.008,1.239,.001,1.798,-.071c.147,-.02,.298,-.047,.439,-.095c.172,-.057,.337,-.156,.434,-.314c3.234,.248,7.015,.364,10.265,.421c61.187,-.013,175.522,.165,235.189,-.015c5.273,-.093,11.094,-.237,16.32,-.784c4.215,-.438,8.463,-1.164,12.534,-2.356C889.674,363.18,894.983,360.582,899.536,356.999z" stroke-width="2.2" vector-effect="non-scaling-stroke"/>'
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
