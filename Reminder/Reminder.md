---
name: "Library/rammelmueller/Reminder"
tags: meta/library
pageDecoration.prefix: "⏰ "
---

# Reminder

Track reminders on pages via `reminderDate`/`reminderTime` frontmatter and list due reminders anywhere in your space.

## Usage

* Run the **Reminder** command — the shipped page template creates a new page under `Reminder/<timestamp>` with the reminder frontmatter pre-filled.
* Fill in `reminderDate` (format `YYYY-MM-DD`) and `reminderTime` (format `HH:MM`, defaults to `07:30`). Until both fields are set and parseable, the page is ignored.
* Embed `${get_active_reminders()}` on any page — it renders all **due** reminders (their date/time has passed), ordered by `reminderDate`, with a link to the reminder page.

> **note** Reminder pages carry the `reminder` tag. Any page tagged `reminder` with valid `reminderDate`/`reminderTime` fields counts — the template is just a convenience.
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
      title = "[[" .. name .. "|" .. title .. "]]",
      due = reminderDate .. " / " .. reminderTime,
    }
  ]]
end
```
