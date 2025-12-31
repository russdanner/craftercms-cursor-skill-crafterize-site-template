# CursorAI HTML Template to CrafterCMS Project Conversion Skill
This repository contains a **Cursor skill/rule** for converting (“**crafterizing**”) a static HTML website template into a fully functional, authorable **CrafterCMS** project.

It is designed to produce **pixel-perfect parity** with the original HTML while moving all author-editable content into Crafter content items and enabling **Experience Builder (XB)** in-context editing.

---

## What “Crafterize” Means

Crafterizing transforms a static template into a structured CrafterCMS site by:

- Breaking pages into reusable **components**
- Creating **content types** for:
  - pages
  - components
  - **nested components** (items inside collections/grids)
- Creating **content items** for every page + every component (including nested items)
- Converting HTML into **FreeMarker templates (FTL)** while preserving the DOM structure exactly
- Adding **Experience Builder (XB)** markup so authors can edit content inline
- Wiring pages to sections via `sections_o` references (ordered component lists)

---

## Installation
1. Open Cursor Preferences
2. Navigate to Rules and Commands
3. Add a User Rule
4. Paste the skill markdown into the rule input area
5. Save the preferences

## Use
1. In Crafter Studio, create a new project in CrafterCMS using the Empty Blueprint

In CursorAI:
2. open the sandbox folder for the new project
3. Add a folder called `html-template`
4. Add all of the html files and assets from the template to the `html-template` folder
5. Open chat and invoke the skill by entering "Crafterize this html template"
