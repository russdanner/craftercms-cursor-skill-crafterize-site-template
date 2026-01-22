# CursorAI HTML Template to CrafterCMS Project Conversion Skill
This repository contains a **Cursor skill/rule** for converting (“**crafterizing**”) a static HTML website template into a fully functional, authorable **CrafterCMS** project.
It is designed to produce **pixel-perfect parity** with the original HTML while moving all author-editable content into Crafter content items and enabling **Experience Builder (XB)** in-context editing.

## Demo
https://www.youtube.com/watch?v=9pdY-nCkyls

<img width="720px" alt="image" src="https://github.com/user-attachments/assets/58a80cd0-d1a1-4625-b434-3c15fe73d5b6" />

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
- Generate Markdown documentation for the site and content model
  ```
  ## `/component/header`

  **Template**: `/templates/web/components/header.ftl`  
  **Content Path**: `/site/components/headers/`  
  **Description**: Site header with branding, contact info, social links, and navigation

  #### Key Fields

  | Name | Type | Description | Required | Notes |
  |------|------|-------------|----------|-------|
  | siteName_t | input | Site name/logo text | true | XB editable (iceId=branding), maxlength=100 |
  | logo_s | image-picker | Logo image (optional) | false | XB editable (iceId=branding) |
  | email_t | input | Contact email | false | XB editable (iceId=contact), maxlength=100 |
  | phone_t | input | Contact phone | false | XB editable (iceId=contact), maxlength=50 |
  | socialLinks_o | repeat | Social media links | false | XB editable (iceId=social), minOccurs=0, maxOccurs=* |
  | navItems_o | repeat | Navigation menu items | false | XB editable (iceId=navigation), supports nested dropdowns |

  #### Nested Fields (socialLinks_o)
  - `platform_t` (input): Platform name (twitter, facebook, etc.), required
  - `url_s` (input): Social media URL, required

  #### Nested Fields (navItems_o)
  - `label_t` (input): Menu item label, required
  - `url_s` (input): Menu item URL, required
  - `hasDropdown_b` (checkbox): Whether item has dropdown
  - `dropdownItems_o` (repeat): Nested dropdown items (supports multi-level)
  ```
  
---

## Installation
1. Open Cursor Preferences
2. Navigate to Rules and Commands
3. Add a User Rule
4. Paste the skill markdown into the rule input area 
(https://raw.githubusercontent.com/russdanner/craftercms-cursor-skill-crafterize-site-template/refs/heads/main/crafterize-html-template-skill.md)
5. Save the preferences

## Use

### Running the Crafterization skill
1. In Crafter Studio, create a new project in CrafterCMS using the Empty Blueprint

In CursorAI:

2. open the sandbox folder for the new project4

3. Add a folder called `html-template`

4. Add all of the html files and assets from the template to the `html-template` folder

5. Open chat and invoke the skill by entering "Crafterize this html template"

### Using Preview to enable Cursor to test and fix issues
You can ask Cursor to check templates and content for correctness and errors using preview with the following type of prompt:
```
Use a curl command like the following to check that the markup of the new site matches the original template exactly and that there are no errors:
curl --header "cookie: crafterPreview=[PREVIEW_TOKEN];" "http://localhost:8080/?crafterSite=[SITE_ID]"

Use curl to review every page you created. Fix issues and errors found in each response. Report back all of the corrections
```

Crafter Studio's Preview requires a Preview token in order to access it externally from Studio. You can follow these instructions to generate a preview token for your site:
https://craftercms.com/docs/current/reference/modules/studio.html#preview-token

### Open Browser preview for the site in Cursor
It's entirely optional, but you can open a Cursor browser for the site and directing Cursor to your preview server with the following URL:
http://localhost:8080/?crafterSite=[SITE_ID]&crafterPreview=[URL_ENCODED_PREVIEW_TOKEN]

## Other tips
1. Cursor will generally run for 10 to 20 minutes before stopping for you to check its work (assuming all commands it wants to execute are allowed.)  Large sites may not complete in this time. In this case, we've found it may be best to ask it to focus on certain page types or sections in each iteration.
```
Can you check the departments page to make sure it is complete and correct any missing or broken elements.
```
2. Give Cursor the ability to use Cursor to check its work. See above for help.
3. Sometimes XB tags interfere with the CSS styles. You can ask Cursor to correct the template or CSS as needed to ensure the design is preserved with a prompt similar to:
   ```
   Double check that your XB modifications do not break CSS styles, if they do, modify the CSS to correct the style.
   ```
4. Remember to ask cursor to generate documentation for the site. This often helps Cursor be consistent as it leverages content types.
