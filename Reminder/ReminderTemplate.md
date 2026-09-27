---
tags: meta/template/page
command: Reminder
suggestedName: "Reminder/${os.date('%Y-%m-%d/%H-%M-%S')}"
confirmName: false
frontmatter: | 
  title: 
  reminderDate: ${date.today()}
  reminderTime: 00:00
  creationDate: ${date.today()}
  tags: 
  - reminder
---