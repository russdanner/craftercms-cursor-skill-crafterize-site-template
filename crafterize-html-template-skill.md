# Skill: Convert HTML Template to CrafterCMS Project (Pages + Components + XB Editing)

This process is referred to as **crafterizing** a template.

## Purpose

Transform a static HTML website template into a functional CrafterCMS project by:

* Breaking pages into reusable components
* Creating content types for pages and components (including nested components)
* Moving static content into content items
* Converting HTML into FreeMarker templates (FTL) with exact structure preservation
* Adding Experience Builder markup so authors can edit content in-context
* Creating content items for every page and component

## Inputs

* A folder (typically `html-template`) containing a static site template:
  * `*.html` files (all pages)
  * `assets/` or `css/`, `js/`, `img/` directories
  * Fonts, images, and other static assets
* Optional: design notes (navigation rules, page variants, component list, content ownership)

## Outputs

A CrafterCMS project with:

* Site config + blueprint-style structure
* Content types in `/config/studio/content-types/` (NOT `/site/content-types/`)
* Templates in `/templates/web/` (pages, components, fragments)
* **Page content items in `/site/website/` (ONE FOR EACH HTML PAGE)**
* **Each page's `sections_o` field populated with component references**
* Shared components in `/site/components/`
* **Nested component items with actual data from HTML**
* **All component collections populated with item references**
* Static assets moved into `/static-assets/`
* Experience Builder (XB) markup across editable regions
* Create a page for each template type

---

## Constraints & Principles

1. **Content-first**: Anything authors should change must be in content (XML) and rendered via FTL
2. **Components over copy-paste**: Repeated blocks become components
3. **Keep HTML structure**: Preserve CSS classes and DOM shape exactly - pixel perfect parity
4. **XB everywhere it matters**: Headings, rich text, images, links, lists, CTA labels/URLs, nav items
5. **Avoid "mega content types"**: Create a small set of page models + modular components
6. **CRITICAL: Create page content items for EVERY HTML page** - don't just create templates
7. **CRITICAL: Populate `sections_o` fields** - pages must reference their components
8. **CRITICAL: Match HTML structure exactly** - preserve all div wrappers, CSS classes, and nesting
9. **CRITICAL: Content types go in `/config/studio/content-types/`** - NOT `/site/content-types/`
10. **CRITICAL: Create content types for ALL components** - including nested components in collections
11. **CRITICAL: Create content items for ALL nested components** - with actual data from HTML
12. **CRITICAL: Populate all component collections** - with item references in correct order

---

## Phase 0 — Quick Inventory (Template Triage)

### Step 0.1: Identify page set

* List all `.html` pages and group them by layout similarity
* Detect page "types" based on:
  * Unique hero/banner patterns
  * Sidebar vs full-width
  * Article/blog style vs marketing landing
  * Forms/contact
  * Search/list pages (if any)

**Deliverable**: List of all HTML pages and their page types

### Step 0.2: Identify repeated regions

Across pages, mark these as likely components (adapt to your template):

* Header / Nav
* Footer
* Hero / Banner / Jumbotron
* Feature cards / icon rows / benefit sections
* Testimonials / Reviews
* Pricing tables / Product grids
* FAQ accordion / Q&A sections
* Image galleries / Photo grids
* CTA strips / Call-to-action sections
* Breadcrumbs
* Sidebars / Aside content
* Content blocks / Rich text sections
* Forms (contact, newsletter, etc.)
* Any other repeated HTML blocks

**Deliverable**: Component map listing all main components

### Step 0.3: Identify nested components

**CRITICAL:** Identify components that appear WITHIN other components (e.g., product items in a product grid, feature items in a feature section, testimonial items in a testimonial slider). These need their own content types.

Common nested component patterns:
* Product items in product grids
* Feature items in feature sections
* Testimonial items in testimonial sliders
* Blog post items in blog sections
* Team member items in team sections
* Card items in card grids
* FAQ items in FAQ sections
* Any repeatable items within a parent component

**Deliverable**: List of all nested component types

### Step 0.4: Identify data-like content

Extract content candidates (adapt to your template):

* Site name, logo, brand elements
* Social media links
* Navigation menu items
* Footer link columns
* Global CTA (button label+url)
* Reusable disclaimers / Legal text
* Repeated card/item sets
* Contact information
* Any other content that appears in multiple places

**Deliverable**: Component Map and Page Type Map including nested components

---

## Phase 1 — Crafter Project Skeleton & Asset Migration

### Step 1.1: Create Crafter project structure

Ensure directories exist (typical):

* `/templates/web/` (page templates)
* `/templates/web/pages/` (page templates)
* `/templates/web/components/` (component templates)
* `/templates/web/fragments/` (head, scripts, etc.)
* `/scripts/` (optional helpers)
* `/static-assets/app/` (css/js/fonts)
* `/static-assets/images/` (images)
* `/site/website/` (pages - **ONE FOLDER PER PAGE** the actual page XML content goes in index.xml inside the folder)
* `/site/components/` (shared component items)
* `/config/studio/` (XB configuration if needed)
* `/config/studio/content-types/` (definitions) - CRITICAL: Use this path, NOT /site/content-types/**

### Step 1.2: Move static assets

* Copy CSS/JS/fonts into relevant subfolders of `/static-assets/app/...`
* Copy images into `/static-assets/images/...`
* Copy favicon to `/static-assets/`
* Update paths in templates to use Crafter-friendly URLs, e.g.:
  * `/static-assets/app/css/main.css`
  * `/static-assets/app/js/main.js`
  * `/static-assets/images/branding/logo.svg`

**CRITICAL**: Maintain the same relative path structure within `/static-assets/` as in the original template. If no structure exist, group images logically into sub folders based on use.


**Deliverable:** `/static-assets/images/branding/logo.svg` (or equivalent) + assets in place

---

## Phase 2 — Content Modeling (Pages + Components)

### Step 2.1: Define page content types

**CRITICAL: Content types MUST be created under `/config/studio/content-types/`**

Create 2–5 page types that cover each unique html file in the template:

**EXAMPLE:**
* **Generic Page**
  * title
  * slug/url
  * SEO fields (Seo Keywords, SEO Description)
  * `disabled_b` defaulted to false
  * `sections_o` (repeating component references)
  * `header_o` (optional override)
  * `footer_o` (optional override)
* **Home Page**
  * same as generic + hero variant fields if needed
* **Article/Blog Page** (if template has it)
  * title, date, author, body (rich text), categories
* **Landing Page** (optional)
  * campaign fields, special hero layout, sections

**Strong default pattern:** pages render a list of sections, each section is a component content item.

**CRITICAL:** For each page type:
1. Create `/config/studio/content-types/page/[page-type]/config.xml`
2. Create `/config/studio/content-types/page/[page-type]/form-definition.xml`
3. Ensure `sections_o` field is defined as a node-selector with appropriate contentTypes

#### config.xml Format

**CRITICAL**: Use attributes on the root element, NOT nested elements.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<content-type name="/page/page_generic" is-wcm-type="true">
  <label>Generic Page</label>
  <description>Standard page template</description>
	<file-extension>xml</file-extension>
	<content-as-folder>true</content-as-folder>
	<previewable>true</previewable>
	<quickCreate>false</quickCreate>
	<quickCreatePath></quickCreatePath>
	<noThumbnail>true</noThumbnail>
	<image-thumbnail>image.jpg</image-thumbnail>
	<paths>
		<excludes>
			<pattern>^/site/components.*</pattern>
		</excludes>
	</paths>  
</content-type>
```

**Common Mistake**: Do NOT use `<content-type><id>...</id></content-type>` format.
* Page type should have content-as-folder set to true. Component types should set content-as-folder to false
* Page types should set paths/excludes/pattern to ^/site/components.*
* Component types should set paths/excludes/pattern to ^/site/website.*

#### form-definition.xml Format

**Required Root Elements** (in order):
1. `<title>` - Display title
2. `<description>` - Description
3. `<objectType>` - Object type (usually "page" or "component")
4. `<content-type>` - Content type path (matches config.xml name)
5. `<imageThumbnail>` - Optional thumbnail path
6. `<quickCreate>` - Boolean (true/false)
7. `<quickCreatePath>` - Path for quick creation
8. `<properties>` - Required section with:
   - `display-template` - Path to FTL template (use kebab-case, e.g., `/templates/web/pages/generic.ftl`)
   - `no-template-required` - Boolean
   - `merge-strategy` - Merge strategy value
9. `<sections><section>` - Wrap all fields in sections
10. `<datasources>` - Required for image-picker and node-selector fields (see below)

section `defaultOpen` elements should always be set to true.

Example form defintion:
```xml
<form>
	<title>Home</title>
	<description></description>
	<objectType>page</objectType>
	<content-type>/page/home</content-type>
	<imageThumbnail>page-home.png</imageThumbnail>
	<quickCreate>false</quickCreate>
	<quickCreatePath></quickCreatePath>
	<properties>
		<property>
			<name>display-template</name>
			<label>Display Template</label>
			<value>/templates/web/pages/home.ftl</value>
			<type>template</type>
		</property>
		<property>
			<name>no-template-required</name>
			<label>No Template Required</label>
			<value></value>
			<type>boolean</type>
		</property>
		<property>
			<name>merge-strategy</name>
			<label>Merge Strategy</label>
			<value>inherit-levels</value>
			<type>string</type>
		</property>
	</properties>
	<sections>
		<section>
			<title>Page Properties</title>
			<description></description>
			<defaultOpen>true</defaultOpen>
			<fields>
				<field>
					<type>file-name</type>
					<id>file-name</id>
					<iceId></iceId>
					<title>Page URL</title>
					<description></description>
					<defaultValue></defaultValue>
					<help></help>
					<properties>
						<property>
							<name>size</name>
							<value>50</value>
							<type>int</type>
						</property>
						<property>
							<name>maxlength</name>
							<value>50</value>
							<type>int</type>
						</property>
						<property>
							<name>readonly</name>
							<value>true</value>
							<type>boolean</type>
						</property>
					</properties>
					<constraints>
					</constraints>
				</field>
				<field>
					<type>input</type>
					<id>internal-name</id>
					<iceId></iceId>
					<title>Internal Name</title>
					<description></description>
					<defaultValue></defaultValue>
					<help></help>
					<properties>
						<property>
							<name>size</name>
							<value>50</value>
							<type>int</type>
						</property>
						<property>
							<name>maxlength</name>
							<value>50</value>
							<type>int</type>
						</property>
					</properties>
					<constraints>
						<constraint>
							<name>required</name>
							<value><![CDATA[true]]></value>
							<type>boolean</type>
						</constraint>
					</constraints>
				</field>
				<field>
					<type>input</type>
					<id>navLabel</id>
					<iceId></iceId>
					<title>Nav Label</title>
					<description></description>
					<defaultValue></defaultValue>
					<help></help>
					<properties>
						<property>
							<name>size</name>
							<value>50</value>
							<type>int</type>
						</property>
						<property>
							<name>maxlength</name>
							<value>50</value>
							<type>int</type>
						</property>
						<property>
							<name>readonly</name>
							<value></value>
							<type>boolean</type>
						</property>
						<property>
							<name>tokenize</name>
							<value>false</value>
							<type>boolean</type>
						</property>
						<property>
							<name>escapeContent</name>
							<value>false</value>
							<type>boolean</type>
						</property>
					</properties>
					<constraints>
						<constraint>
							<name>required</name>
							<value><![CDATA[]]></value>
							<type>boolean</type>
						</constraint>
						<constraint>
							<name>pattern</name>
							<value><![CDATA[]]></value>
							<type>string</type>
						</constraint>
					</constraints>
				</field>
				<field>
					<type>checkbox</type>
					<id>disabled</id>
					<iceId></iceId>
					<title>Disable Page</title>
					<description></description>
					<defaultValue></defaultValue>
					<help></help>
					<properties>
						<property>
							<name>readonly</name>
							<value></value>
							<type>boolean</type>
						</property>
					</properties>
					<constraints>
						<constraint>
							<name>required</name>
							<value><![CDATA[]]></value>
							<type>boolean</type>
						</constraint>
					</constraints>
				</field>
				<field>
					<type>input</type>
					<id>title_t</id>
					<iceId>core</iceId>
					<title>Title</title>
					<description></description>
					<defaultValue></defaultValue>
					<help></help>
					<properties>
						<property>
							<name>size</name>
							<value>50</value>
							<type>int</type>
						</property>
						<property>
							<name>maxlength</name>
							<value>50</value>
							<type>int</type>
						</property>
						<property>
							<name>readonly</name>
							<value></value>
							<type>boolean</type>
						</property>
						<property>
							<name>tokenize</name>
							<value>false</value>
							<type>boolean</type>
						</property>
					</properties>
					<constraints>
						<constraint>
							<name>required</name>
							<value><![CDATA[true]]></value>
							<type>boolean</type>
						</constraint>
						<constraint>
							<name>pattern</name>
							<value><![CDATA[]]></value>
							<type>string</type>
						</constraint>
					</constraints>
				</field>
				<field>
					<type>node-selector</type>
					<id>header_o</id>
					<iceId>core</iceId>
					<title>Header</title>
					<description>Default header is inherited from Section Defaults. Specify a new header to overwrite it.</description>
					<defaultValue></defaultValue>
					<help></help>
					<properties>
						<property>
							<name>minSize</name>
							<value>0</value>
							<type>int</type>
						</property>
						<property>
							<name>maxSize</name>
							<value>1</value>
							<type>int</type>
						</property>
						<property>
							<name>itemManager</name>
							<value>components-header</value>
							<type>datasource:item</type>
						</property>
						<property>
							<name>readonly</name>
							<value></value>
							<type>boolean</type>
						</property>
						<property>
							<name>disableFlattening</name>
							<value></value>
							<type>boolean</type>
						</property>
						<property>
							<name>useSingleValueFilename</name>
							<value></value>
							<type>boolean</type>
						</property>
						<property>
							<name>contentTypes</name>
							<value>/component/header</value>
							<type>contentTypes</type>
						</property>
						<property>
							<name>tags</name>
							<value></value>
							<type>string</type>
						</property>
					</properties>
					<constraints>
						<constraint>
							<name>allowDuplicates</name>
							<value><![CDATA[]]></value>
							<type>boolean</type>
						</constraint>
					</constraints>
				</field>
				<field>
					<type>node-selector</type>
					<id>left_rail_o</id>
					<iceId></iceId>
					<title>Left Rail</title>
					<description>Default left-rail is inherited from Section Defaults. Specify a new left-rail to overwrite it.</description>
					<defaultValue></defaultValue>
					<help></help>
					<properties>
						<property>
							<name>minSize</name>
							<value>0</value>
							<type>int</type>
						</property>
						<property>
							<name>maxSize</name>
							<value>1</value>
							<type>int</type>
						</property>
						<property>
							<name>itemManager</name>
							<value>components-left-rail</value>
							<type>datasource:item</type>
						</property>
						<property>
							<name>readonly</name>
							<value></value>
							<type>boolean</type>
						</property>
						<property>
							<name>disableFlattening</name>
							<value></value>
							<type>boolean</type>
						</property>
						<property>
							<name>useSingleValueFilename</name>
							<value></value>
							<type>boolean</type>
						</property>
						<property>
							<name>contentTypes</name>
							<value>/component/left-rail</value>
							<type>contentTypes</type>
						</property>
						<property>
							<name>tags</name>
							<value></value>
							<type>string</type>
						</property>
					</properties>
					<constraints>
						<constraint>
							<name>allowDuplicates</name>
							<value><![CDATA[]]></value>
							<type>boolean</type>
						</constraint>
					</constraints>
				</field>
			</fields>
		</section>
		<section>
			<title>Hero Section</title>
			<description></description>
			<defaultOpen>true</defaultOpen>
			<fields>
				<field>
					<type>rte</type>
					<id>hero_title_html</id>
					<iceId>hero</iceId>
					<title>Hero Title</title>
					<description></description>
					<defaultValue></defaultValue>
					<help></help>
					<properties>
						<property>
							<name>height</name>
							<value>410</value>
							<type>int</type>
						</property>
						<property>
							<name>forceRootBlockPTag</name>
							<value>true</value>
							<type>boolean</type>
						</property>
						<property>
							<name>forcePTags</name>
							<value>true</value>
							<type>boolean</type>
						</property>
						<property>
							<name>forceBRTags</name>
							<value>false</value>
							<type>boolean</type>
						</property>
						<property>
							<name>supportedChannels</name>
							<value></value>
							<type>supportedChannels</type>
						</property>
						<property>
							<name>rteConfiguration</name>
							<value>generic</value>
							<type>string</type>
						</property>
						<property>
							<name>imageManager</name>
							<value>uploadImages,existingImages</value>
							<type>datasource:image</type>
						</property>
						<property>
							<name>videoManager</name>
							<value></value>
							<type>datasource:video</type>
						</property>
					</properties>
					<constraints>
						<constraint>
							<name>required</name>
							<value><![CDATA[true]]></value>
							<type>boolean</type>
						</constraint>
					</constraints>
				</field>
				<field>
					<type>rte</type>
					<id>hero_text_html</id>
					<iceId>hero</iceId>
					<title>Hero Text</title>
					<description></description>
					<defaultValue></defaultValue>
					<help></help>
					<properties>
						<property>
							<name>height</name>
							<value>200</value>
							<type>int</type>
						</property>
						<property>
							<name>forceRootBlockPTag</name>
							<value>true</value>
							<type>boolean</type>
						</property>
						<property>
							<name>forcePTags</name>
							<value>true</value>
							<type>boolean</type>
						</property>
						<property>
							<name>forceBRTags</name>
							<value>false</value>
							<type>boolean</type>
						</property>
						<property>
							<name>supportedChannels</name>
							<value></value>
							<type>supportedChannels</type>
						</property>
						<property>
							<name>rteConfiguration</name>
							<value>generic</value>
							<type>string</type>
						</property>
						<property>
							<name>imageManager</name>
							<value>existingImages,uploadImages</value>
							<type>datasource:image</type>
						</property>
						<property>
							<name>videoManager</name>
							<value></value>
							<type>datasource:video</type>
						</property>
					</properties>
					<constraints>
						<constraint>
							<name>required</name>
							<value><![CDATA[true]]></value>
							<type>boolean</type>
						</constraint>
					</constraints>
				</field>
				<field>
					<type>image-picker</type>
					<id>hero_image_s</id>
					<iceId>hero</iceId>
					<title>Hero Image</title>
					<description></description>
					<defaultValue></defaultValue>
					<help></help>
					<properties>
						<property>
							<name>width</name>
							<value>{ &quot;exact&quot;:&quot;&quot;, &quot;min&quot;:&quot;&quot;, &quot;max&quot;:&quot;&quot; }</value>
							<type>range</type>
						</property>
						<property>
							<name>height</name>
							<value>{ &quot;exact&quot;:&quot;&quot;, &quot;min&quot;:&quot;&quot;, &quot;max&quot;:&quot;&quot; }</value>
							<type>range</type>
						</property>
						<property>
							<name>thumbnailWidth</name>
							<value></value>
							<type>int</type>
						</property>
						<property>
							<name>thumbnailHeight</name>
							<value></value>
							<type>int</type>
						</property>
						<property>
							<name>imageManager</name>
							<value>existingImages,uploadImages</value>
							<type>datasource:image</type>
						</property>
						<property>
							<name>readonly</name>
							<value></value>
							<type>boolean</type>
						</property>
					</properties>
					<constraints>
						<constraint>
							<name>required</name>
							<value><![CDATA[]]></value>
							<type>boolean</type>
						</constraint>
					</constraints>
				</field>
				<field>
					<type>repeat</type>
					<id>articleSections_o</id>
					<iceId>article</iceId>
					<title>Article Sections</title>
					<description></description>
					<minOccurs>1</minOccurs>
					<maxOccurs>*</maxOccurs>
					<properties>
						<property>
							<name>minOccurs</name>
							<value>1</value>
							<type>string</type>
						</property>
						<property>
							<name>maxOccurs</name>
							<value>*</value>
							<type>string</type>
						</property>
					</properties>
					<fields>
						<field>
							<type>rte</type>
							<id>section_html</id>
							<iceId>article</iceId>
							<title>Section</title>
							<description></description>
							<defaultValue></defaultValue>
							<help></help>
							<properties>
								<property>
									<name>height</name>
									<value></value>
									<type>int</type>
								</property>
								<property>
									<name>forceRootBlockPTag</name>
									<value>true</value>
									<type>boolean</type>
								</property>
								<property>
									<name>forcePTags</name>
									<value>true</value>
									<type>boolean</type>
								</property>
								<property>
									<name>forceBRTags</name>
									<value>false</value>
									<type>boolean</type>
								</property>
								<property>
									<name>supportedChannels</name>
									<value></value>
									<type>supportedChannels</type>
								</property>
								<property>
									<name>rteConfiguration</name>
									<value>generic</value>
									<type>string</type>
								</property>
								<property>
									<name>imageManager</name>
									<value>upload_images,existing_images</value>
									<type>datasource:image</type>
								</property>
								<property>
									<name>videoManager</name>
									<value></value>
									<type>datasource:video</type>
								</property>
							</properties>
							<constraints>
								<constraint>
									<name>required</name>
									<value><![CDATA[true]]></value>
									<type>boolean</type>
								</constraint>
							</constraints>
						</field>
					</fields>
				</field>
                <field>
					<type>repeat</type>
					<id>sections_o</id>
					<iceId>article</iceId>
					<title>Sections</title>
					<description></description>
					<minOccurs>1</minOccurs>
					<maxOccurs>*</maxOccurs>
					<properties>
						<property>
							<name>minOccurs</name>
							<value>1</value>
							<type>string</type>
						</property>
						<property>
							<name>maxOccurs</name>
							<value>*</value>
							<type>string</type>
						</property>
					</properties>
					<fields>
						<field>
							<type>rte</type>
							<id>section_html</id>
							<iceId>article</iceId>
							<title>Section</title>
							<description></description>
							<defaultValue></defaultValue>
							<help></help>
							<properties>
								<property>
									<name>height</name>
									<value></value>
									<type>int</type>
								</property>
								<property>
									<name>forceRootBlockPTag</name>
									<value>true</value>
									<type>boolean</type>
								</property>
								<property>
									<name>forcePTags</name>
									<value>true</value>
									<type>boolean</type>
								</property>
								<property>
									<name>forceBRTags</name>
									<value>false</value>
									<type>boolean</type>
								</property>
								<property>
									<name>supportedChannels</name>
									<value></value>
									<type>supportedChannels</type>
								</property>
								<property>
									<name>rteConfiguration</name>
									<value>generic</value>
									<type>string</type>
								</property>
								<property>
									<name>imageManager</name>
									<value>upload_images,existing_images</value>
									<type>datasource:image</type>
								</property>
								<property>
									<name>videoManager</name>
									<value></value>
									<type>datasource:video</type>
								</property>
							</properties>
							<constraints>
								<constraint>
									<name>required</name>
									<value><![CDATA[true]]></value>
									<type>boolean</type>
								</constraint>
							</constraints>
						</field>
					</fields>
				</field>                
			</fields>
		</section>
		<section>
			<title>Features</title>
			<description></description>
			<defaultOpen>true</defaultOpen>
			<fields>
				<field>
					<type>input</type>
					<id>features_title_t</id>
					<iceId>features</iceId>
					<title>Features Title</title>
					<description></description>
					<defaultValue></defaultValue>
					<help></help>
					<properties>
						<property>
							<name>size</name>
							<value>50</value>
							<type>int</type>
						</property>
						<property>
							<name>maxlength</name>
							<value>50</value>
							<type>int</type>
						</property>
						<property>
							<name>readonly</name>
							<value></value>
							<type>boolean</type>
						</property>
						<property>
							<name>tokenize</name>
							<value>false</value>
							<type>boolean</type>
						</property>
					</properties>
					<constraints>
						<constraint>
							<name>required</name>
							<value><![CDATA[true]]></value>
							<type>boolean</type>
						</constraint>
						<constraint>
							<name>pattern</name>
							<value><![CDATA[]]></value>
							<type>string</type>
						</constraint>
					</constraints>
				</field>
				<field>
					<type>node-selector</type>
					<id>features_o</id>
					<iceId>features</iceId>
					<title>Features</title>
					<description></description>
					<defaultValue></defaultValue>
					<help></help>
					<properties>
						<property>
							<name>minSize</name>
							<value></value>
							<type>int</type>
						</property>
						<property>
							<name>maxSize</name>
							<value></value>
							<type>int</type>
						</property>
						<property>
							<name>itemManager</name>
							<value>featuresComponents</value>
							<type>datasource:item</type>
						</property>
						<property>
							<name>readonly</name>
							<value></value>
							<type>boolean</type>
						</property>
						<property>
							<name>disableFlattening</name>
							<value></value>
							<type>boolean</type>
						</property>
						<property>
							<name>useSingleValueFilename</name>
							<value></value>
							<type>boolean</type>
						</property>
						<property>
							<name>contentTypes</name>
							<value>/component/feature</value>
							<type>contentTypes</type>
						</property>
						<property>
							<name>tags</name>
							<value></value>
							<type>string</type>
						</property>
					</properties>
					<constraints>
						<constraint>
							<name>allowDuplicates</name>
							<value><![CDATA[]]></value>
							<type>boolean</type>
						</constraint>
					</constraints>
				</field>
			</fields>
		</section>
	</sections>
	<datasources>
		<datasource>
			<type>img-repository-upload</type>
			<id>existingImages</id>
			<title>Existing Images</title>
			<interface>image</interface>
			<properties>
				<property>
					<name>repoPath</name>
					<value>/static-assets/images/</value>
					<type>undefined</type>
				</property>
			</properties>
		</datasource>
		<datasource>
			<type>img-desktop-upload</type>
			<id>uploadImages</id>
			<title>Upload Images</title>
			<interface>image</interface>
			<properties>
				<property>
					<name>repoPath</name>
					<value>/static-assets/item/images/{yyyy}/{mm}/{dd}/</value>
					<type>undefined</type>
				</property>
			</properties>
		</datasource>
		<datasource>
			<type>shared-content</type>
			<id>components-header</id>
			<title>Components Header</title>
			<interface>item</interface>
			<properties>
				<property>
					<name>repoPath</name>
					<value>/site/components/headers/</value>
					<type>undefined</type>
				</property>
				<property>
					<name>browsePath</name>
					<value></value>
					<type>undefined</type>
				</property>
				<property>
					<name>type</name>
					<value></value>
					<type>undefined</type>
				</property>
			</properties>
		</datasource>
		<datasource>
			<type>shared-content</type>
			<id>components-left-rail</id>
			<title>Components Left Rail</title>
			<interface>item</interface>
			<properties>
				<property>
					<name>repoPath</name>
					<value>/site/components/left-rails/</value>
					<type>undefined</type>
				</property>
				<property>
					<name>browsePath</name>
					<value></value>
					<type>undefined</type>
				</property>
				<property>
					<name>type</name>
					<value></value>
					<type>undefined</type>
				</property>
				<property>
					<name>enableCreateNew</name>
					<value>true</value>
					<type>boolean</type>
				</property>
				<property>
					<name>enableBrowseExisting</name>
					<value>true</value>
					<type>boolean</type>
				</property>
				<property>
					<name>enableSearchExisting</name>
					<value>true</value>
					<type>boolean</type>
				</property>
			</properties>
		</datasource>
		<datasource>
			<type>components</type>
			<id>featuresComponents</id>
			<title>Features Components</title>
			<interface>item</interface>
			<properties>
				<property>
					<name>allowShared</name>
					<value>true</value>
					<type>boolean</type>
				</property>
				<property>
					<name>allowEmbedded</name>
					<value>true</value>
					<type>boolean</type>
				</property>
				<property>
					<name>enableBrowse</name>
					<value>true</value>
					<type>boolean</type>
				</property>
				<property>
					<name>enableSearch</name>
					<value>true</value>
					<type>boolean</type>
				</property>
				<property>
					<name>baseRepositoryPath</name>
					<value>/site/components</value>
					<type>string</type>
				</property>
				<property>
					<name>baseBrowsePath</name>
					<value>/site/components</value>
					<type>string</type>
				</property>
				<property>
					<name>contentTypes</name>
					<value>/component/feature</value>
					<type>contentTypes</type>
				</property>
				<property>
					<name>tags</name>
					<value></value>
					<type>string</type>
				</property>
			</properties>
		</datasource>
	</datasources>
</form>
```

**Field Type Guidelines**:
* Rich text fields: Use `rte` (NOT `rich-text`)
* Repeatable groups: Use `repeatable-group`
* Text fields: Use `input` or `text`
* Boolean fields: Use `checkbox`
* Image fields: Use `image-picker` (requires datasource)
* URL fields: Use `input` with validation
* Component references: Use `node-selector` (requires datasource and contentTypes)

**CRITICAL: All form definitions MUST include datasources section:**

```xml
<datasources>
  <datasource>
    <type>img-repository-upload</type>
    <id>existingImages</id>
    <title>Existing Images</title>
    <interface>image</interface>
    <properties>
      <property>
        <name>repoPath</name>
        <value>/static-assets/images/</value>
      </property>
    </properties>
  </datasource>
  <datasource>
    <type>img-desktop-upload</type>
    <id>uploadImages</id>
    <title>Upload Images</title>
    <interface>image</interface>
    <properties>
      <property>
        <name>repoPath</name>
        <value>/static-assets/item/images/{yyyy}/{mm}/{dd}/</value>
      </property>
    </properties>
  </datasource>
  <datasource>
    <type>components</type>
    <id>[component-collection-id]</id>
    <title>[Component Collection Title]</title>
    <interface>item</interface>
    <properties>
      <property>
        <name>allowShared</name>
        <value>true</value>
        <type>boolean</type>
      </property>
      <property>
        <name>allowEmbedded</name>
        <value>true</value>
        <type>boolean</type>
      </property>
      <property>
        <name>baseRepositoryPath</name>
        <value>/site/components</value>
        <type>string</type>
      </property>
      <property>
        <name>contentTypes</name>
        <value>/component/[item-type]</value>
        <type>contentTypes</type>
      </property>
    </properties>
  </datasource>
</datasources>
```

**CRITICAL: All image-picker fields MUST have datasources:**
```xml
<properties>
  <property>
    <name>imageManager</name>
    <value>existingImages,uploadImages</value>
    <type>datasource:image</type>
  </property>
</properties>
```

**CRITICAL: All node-selector fields MUST have datasources:**
```xml
<properties>
  <property>
    <name>itemManager</name>
    <value>[datasource-id]</value>
    <type>datasource:item</type>
  </property>
  <property>
    <name>contentTypes</name>
    <value>/component/[component-type]</value>
    <type>contentTypes</type>
  </property>
</properties>
```

### Step 2.2: Define component content types

**CRITICAL: Create content types for ALL components, including nested components**

Create component types aligned to your Component Map. Common component patterns include:

* `header` (brand, navigation items)
* `footer` (brand, description, social links, footer link columns, copyright)
* `hero` or `banner` (headline, subhead, background image, buttons)
* `richText` or `content-block` (html body)
* `card-grid` or `item-grid` (title + repeatable items)
* `card` or `item` (image, title, description, optional price/URL)
* `feature` (icon/image, title, description)
* `feature-grid` or `benefits-section` (title, description, image, features list)
* `ctaStrip` or `call-to-action` (text + button)
* `imageTextSplit` or `two-column` (image + text + alignment)
* `faq` or `accordion` (repeatable Q/A)
* `testimonial` (quote, author, position, image)
* `testimonial-slider` or `testimonials` (title + testimonials list)
* `blog-post` or `article-preview` (title, image, author, date, URL)
* `blog-section` or `articles-grid` (title + blog posts list)
* `team-member` or `person` (image, name, position, bio)
* `team-section` or `people-grid` (title + team members list)
* `contact-info` (icon, text)
* `contact-form` (contact info items + form)
* `product` (image, title, description, price, URL)
* `product-section` or `products-grid` (title + products list)
* Any other components identified in Phase 0

**CRITICAL:** For EACH component type (including nested components):
1. Create `/config/studio/content-types/component/[component-name]/config.xml`
2. Create `/config/studio/content-types/component/[component-name]/form-definition.xml`
3. Add all required fields with proper types
4. **Add datasources for image-picker and node-selector fields**
5. **Use kebab-case in display-template paths** (e.g., `/templates/web/components/product-section.ftl`)

**Component data design rules**

* Use repeatable groups for lists (cards, links, FAQs)
* For links: store both label and URL (and optionally target)
* For images: store image field + alt text (Crafter image-picker field)
* For rich text: store HTML body (Crafter `rte` field - NOT `rich-text`)
* **CRITICAL:** If a component is referenced in ANY page form definition's `contentTypes` list, it MUST have a complete form definition
* **CRITICAL:** If a component contains a collection of items (e.g., products in a product grid), create a separate content type for the item type (e.g., `product` for items in `product-grid`)

**Display Template Paths**
- **Always use kebab-case** (hyphens) in display-template paths
- **Match actual file names**: If template is `product-section.ftl`, path must be `/templates/web/components/product-section.ftl`
- **Never use underscores** in display-template paths, even if content type name uses underscores

**Deliverable:** content-type definitions ready for Studio, including all nested component types

---

## Phase 2.5 Create Documentation

### Output Location
/docs/

### Required Files (ALL MUST EXIST)
README.md – overall architecture
pages.md – page content types
components.md – section components
items.md – nested item components
global.md – level descriptor

### Documentation Rules
Completeness Contract
- If something exists → document it.
- If it is documented → it must exist.
- Documentation must match the actual implementation (no “idealized” fields).
- Use exact field IDs, content-type paths, and template paths.

### Field Documentation Requirements 
For every field in every content type (pages, components, nested items, level descriptor), documentation MUST include the following properties:

Name
The exact field ID as it appears in the form-definition (e.g., title_t, sections_o, hero_image_s).

Description
A human-readable explanation of what the field is used for.

Required
true / false (based on constraints in the form-definition; if ambiguous, treat as false and note ambiguity).

Constraints
Any relevant constraints and UI properties (as applicable):

minSize, maxSize (node-selector)
maxlength, pattern, readonly, tokenize, escapeContent

image size ranges (width, height) if present

RTE specifics (rteConfiguration, height, forced tags) if present

allowDuplicates / disableFlattening / useSingleValueFilename if present

For selectors and collections, notes must include:
- whether it’s a list (item-list=true / multi-select)
- allowed content types (contentTypes)
- datasource ID / itemManager ID
- expected repository path conventions (e.g., /site/components/...)

Selector Rule: If a field is a node-selector (single or list), the Notes section MUST explicitly list the allowed component content types and whether multiple items are permitted.

Example:
```md
| Name | Type | Description | Required | Constraints | Notes |
|------|------|-------------|----------|------------|-------|
| title_t | input | Page title shown in H1/H2. | true | maxlength=50 | XB editable. |
| sections_o | node-selector (list) | Ordered list of section components. | true | minSize=1; maxSize=999 | Allows: /component/hero, /component/feature_grid, /component/cta_strip. itemManager=components-sections. |
```

---

## Phase 3 — Convert HTML to FTL Templates

### Step 3.1: Page templates

For each page type:

* Copy the corresponding HTML file into an FTL template. **Maintain pixel perfect parity with the original markup unless unavoidable.**
* Replace hardcoded values with FTL expressions from the page content model
* Replace repeated blocks with component includes
* Render collections of components with the following macro:
```ftl
	<@crafter.renderComponentCollection $field="sections_o"/>
```

Example page template:
```ftl
<#import "/templates/system/common/crafter.ftl" as crafter />

<!--
	Editorial by HTML5 UP
	html5up.net | @ajlkn
	Free for personal and commercial use under the CCA 3.0 license (html5up.net/license)
-->
<!doctype html>
<html lang="en">
<head>
    <#include "/templates/web/fragments/head.ftl">
    <@crafter.head/>
</head>
<body class="is-preload">
<@crafter.body_top/>

<!-- Wrapper -->
<div id="wrapper">

	<!-- Main -->
	<div id="main">
		<div class="inner">

			<!-- Header -->
                    <@crafter.renderComponentCollection $field="header_o"/>
			<!-- /Header -->

			<!-- Banner -->
			<section id="banner">
				<div class="content">
                                    <@crafter.header $field="hero_title_html">
                                        ${contentModel.hero_title_html}
                                    </@crafter.header>
                                    <@crafter.div $field="hero_text_html">
                                        ${contentModel.hero_text_html}
                                    </@crafter.div>
				</div>
				<span class="image object">
          <@crafter.img $field="hero_image_s" src=(contentModel.hero_image_s!"") alt=""/>
        </span>
			</section>
			<!-- /Banner -->

			<!-- Section: Features -->
			<section>
				<header class="major">
                                    <@crafter.h2 $field="features_title_t">
                                        ${contentModel.features_title_t}
                                    </@crafter.h2>
				</header>
                            <@crafter.renderComponentCollection
                            $field="features_o"
                            $containerAttributes={ "class": "features" }
                            $itemAttributes={ "class": "feature-container" }
                            />
			</section>
			<!-- /Section: Features -->

			<!-- Section: Articles -->
			<section>
				<header class="major">
					<h2>Featured Articles</h2>
				</header>
				<div class="posts">
                                    <#list articles as article>
                                        <@crafter.article $model=article>
						<a href="${article.url}" class="image">
                                                    <#--
						    Note for docs:
						    Works: src=article.image???then(article.image, "/static-assets/images/placeholder.png")
						    Error: src="${article.image???then(article.image, "/static-assets/images/placeholder.png")}" 🤷
						    however...
						    Works: href="${article.url}"
						    -->
                                                    <@crafter.img
                                                    $model=article
                                                    $field="image_s"
                                                    src=article.image???then(article.image, "/static-assets/images/placeholder.png")
                                                    alt=""
                                                    />
						</a>
						<h3>
                                                    <@crafter.a $model=article $field="subject_t" href="${article.url}">
                                                        ${article.title}
                                                    </@crafter.a>
						</h3>
                                            <@crafter.p $model=article $field="summary_t">
                                                ${article.summary}
                                            </@crafter.p>
						<ul class="actions">
							<li>
								<a href="${article.url}" class="button">More</a>
							</li>
						</ul>
                                        </@crafter.article>
                                    </#list>
				</div>
			</section>
			<!-- /Section: Articles -->

		</div>
	</div>
	<!-- /Main -->

	<!-- Left Rail -->
    <@crafter.renderComponentCollection $field="left_rail_o" />
	<!-- /Left Rail -->

</div>
<!-- /Wrapper -->

<#include "/templates/web/fragments/scripts.ftl">
<@crafter.body_bottom/>

</body>
</html>

```
Example component template:
```ftl
<#import "/templates/system/common/crafter.ftl" as crafter />

<!-- Feature Component -->
<@crafter.article class="feature">
    <@crafter.span class="icon ${contentModel.icon_s}" $field="icon_s"/>
	<div class="content">
            <@crafter.h3 $field="title_t">
                ${contentModel.title_t}
            </@crafter.h3>
            <@crafter.div $field="body_html">
                ${contentModel.body_html}
            </@crafter.div>
	</div>
</@crafter.article>
```
Pattern (name templates based on page types identified):
* `templates/web/pages/home.ftl` (or `landing.ftl`, `index.ftl`, etc.)
* `templates/web/pages/generic.ftl` (or `standard.ftl`, `default.ftl`, etc.)
* `templates/web/pages/article.ftl` (if template has articles/blog)
* `templates/web/pages/[page-type].ftl` (create for each unique page type)

**CRITICAL: Preserve exact HTML structure** - all div wrappers, CSS classes, and nesting must match original

### Step 3.2: Component templates

For each component type:

* Extract the HTML snippet into `templates/web/components/<component>.ftl`
* Bind fields from the component content item model
* **CRITICAL: Preserve exact HTML structure** - all div wrappers, CSS classes, and nesting must match original

Pattern (name components based on their purpose, use kebab-case):
* `templates/web/components/hero.ftl` (or `banner.ftl`, `header-section.ftl`, etc.)
* `templates/web/components/card-grid.ftl` (or `item-grid.ftl`, `content-grid.ftl`, etc.)
* `templates/web/components/[component-name].ftl` (create for each component type)

**CRITICAL:** Create templates for ALL component types, including nested components (product.ftl, feature.ftl, testimonial.ftl, etc.)

**HTML Structure Preservation Process**:
1. Copy HTML from original template
2. Convert to FTL syntax (add FreeMarker directives)
3. Compare line-by-line with original HTML
4. Verify all classes, attributes, and nesting match exactly
5. Pay special attention to:
   - CSS class names (exact spelling)
   - HTML element types (`<div>` vs `<p>` vs `<span>`)
   - Attribute order (doesn't affect rendering but should match)
   - Whitespace and indentation (affects CSS in some cases)

### Step 3.3: FreeMarker Syntax Rules

#### Property Access
- **Hyphenated Properties**: Use bracket notation for properties with hyphens
  - ✅ `${item['display-template']}`
  - ❌ `${item.displayTemplate}` (will fail)

#### Boolean Comparisons
- **Direct Comparison**: Boolean fields are already booleans, compare directly
  - ✅ `<#if item.active_b>`
  - ❌ `<#if item.active_b == 'true'>` (type mismatch error)

#### Null Safety
- Always use null-safe operators (`!` or `??`) for optional fields
  - `${contentModel.title_t!''}`
  - `<#if contentModel.description_html??>`

#### Conditional Classes
- **NEVER use `?then()` inside string interpolation** - it causes type errors. Use `#if` directives instead:
  - WRONG: `class="btn ${button_index == 0?then('me-2', '')}"`
  - CORRECT: `class="btn <#if button_index == 0>me-2</#if>"`

**CRITICAL RULE:** When rendering component collections in grids (Bootstrap, Tailwind, CSS Grid, Flexbox, or any CSS framework), you MUST wrap each component in the proper column/wrapper divs that match the original HTML structure.

```ftl
<@crafter.renderComponentCollection
	$field="items_o"
	$containerAttributes={ "class": "grid-column-class" }
/>
```


## Phase 4 — Add Experience Builder (XB) Markup for In-Context Editing

### Goal

Authors should click/edit all visible copy, image and video content:

Examples:
* Headings, paragraphs, button labels
* Links
* Images (and alt text)
* Card titles/descriptions
* FAQ entries
* Global nav/footer links

### Step 4.1: Apply XB markers systematically

For each editable field, wrap output with the appropriate XB markup for:

* Text fields: `<@crafter.h1 $field="title_t">${contentModel.title_t!''}</@crafter.h1>`
* Rich text fields: `<@crafter.div $field="body_html">${contentModel.body_html!''}</@crafter.div>`
* Images: `<@crafter.img $field="image_s" src="${contentModel.image_s!''}" alt="..."/>`
* Links: `<@crafter.a $field="url_s" href="${contentModel.url_s!''}">${contentModel.label_t!''}</@crafter.a>`
* Repeatable groups: Use `$index` parameter for list items

**Rule of thumb**

* If the field is author-editable, it needs an XB hook at the point it renders

### Step 4.2: XB Macro Usage

Use `@crafter` macros for all editable content:
* `@crafter.div` - For div elements
* `@crafter.span` - For span elements
* `@crafter.p` - For paragraph elements
* `@crafter.h1`, `@crafter.h2`, etc. - For headings
* `@crafter.a` - For links
* `@crafter.img` - For images
* `@crafter.renderRepeatGroup` - For repeatable groups

### Step 4.3: Field Path Syntax

**Single Fields**:
```ftl
<@crafter.span $field="title_t">${contentModel.title_t!''}</@crafter.span>
```

**Repeatable Groups**:
```ftl
<@crafter.renderRepeatGroup $field="items_o"; item, index>
  <@crafter.span $field="items_o.label_t" $index=index>${item.label_t!''}</@crafter.span>
</@crafter.renderRepeatGroup>
```

**Nested Fields in Repeatable Groups**:
```ftl
<@crafter.renderRepeatGroup $field="columns_o"; column, column_index>
  <#list column.links_o.item as link>
    <@crafter.a $field="columns_o.links_o.label_t" $index="${column_index}.${link_index}" href="${link.url_s!''}">${link.label_t!''}</@crafter.a>
  </#list>
</@crafter.renderRepeatGroup>
```

### Step 4.4: XB Markup Rules

#### Rule 1: One Field Per Element
- **Each editable field should have ONE XB markup wrapper**
- **Example**:
  - ✅ `<@crafter.h3 $field="title_t">${contentModel.title_t}</@crafter.h3>`

#### Rule 2: Wrap the Correct Element
- **Wrap the actual HTML element that contains the content**
- **For links**: Wrap the `<a>` tag, not the content inside
- **For images**: Wrap the `<img>` tag
- **For text**: Wrap the text-containing element (span, p, h1-h6, etc.)

#### Rule 3: Preserve Original HTML Structure
- **If original has a link around text, keep the link structure**
- **If original has inline SVG, keep the SVG structure**
- **Do NOT replace HTML elements with XB markup if it changes structure**

### Step 4.5: Ensure components are XB-friendly

* Component templates should provide clear editable boundaries:
  * whole component selection
  * per-field in component
  * The outermost tag of the component is a @crafter tag example <@crafter.div>...</@crafter.div>


### Step 4.6: Global regions

Header/footer/nav should be driven from the Site Config content item and be XB-editable as well

**Deliverable:** authors can edit both page and component content visually

---

## Phase 5 — Create Content Items & Wire Everything Up

### Step 5.1: Create content items for pages

**CRITICAL: For EVERY HTML page, create a corresponding content item under `/site/website/`**

For each HTML page:

* Create a folder: `/site/website/<page-name>/`
* Create `index.xml` in that folder
* **CRITICAL: All content XML files MUST have a single root element:**
  - Pages: Use `<page>` as root element
  - Components: Use `<component>` as root element
* Set `content-type` to the appropriate page type
* Set `display-template` to the page template path (use kebab-case, match actual file name)
* **Required Elements**:
  - `<internal-name>` - Human-readable name (derived from file path or meaningful name)
  - `<content-type>` - Path to content type (e.g., `/page/page_generic`)
  - `<display-template>` - Path to FTL template (must match actual file name with kebab-case)
  - `<objectId>` unqiue GUID
  - All field values as child elements
* Populate:
  * Title, SEO fields
  * Header/footer component references
  * **CRITICAL: Populate `sections_o` field with component references**

Example structure (adapt to your template's pages):
```
/site/website/
  index.xml (home page)
  about/
    index.xml
  services/
    index.xml
  contact/
    index.xml
  [other-pages]/
    index.xml
```

**Note:** Create folders and content items for ALL HTML pages found in the template, regardless of their names or structure.

**CRITICAL:** Update existing pages (like the default `index.xml`) to use new content types and templates.

**URL Format**:
- **Pages**: Use simplified paths without `/index.xml` or `/index.html`
- **Examples**: 
  - ✅ `/cart` 
  - ✅ `/about`
  - ❌ `/cart/index.html`
  - ❌ `/cart/index.xml`

### Step 5.2: Create component content items

For each repeated region:

* Create component items under `/site/components/<component-type>/...`
* **CRITICAL: All component XML files MUST have `<component>` as root element**
* **Required Elements**:
  - `<internal-name>` - Human-readable name
  - `<content-type>` - Path to content type
  - `<display-template>` - Path to FTL template (kebab-case, match actual file name)
  - `<objectId>` unqiue GUID
  - All field values as child elements
* Populate fields from the original HTML text
* Reference in pages' `sections_o` fields

Example structure (adapt to your template's components):
```
/site/components/
  headers/
    main-header.xml
  footers/
    main-footer.xml
  heroes/ (or banners/)
    [page-name]-hero.xml
  [item-type]/
    [item-name].xml
  [section-type]/
    [section-name].xml
```

**Note:** Organize components by type in subdirectories. Use descriptive, generic names that reflect the component's purpose, not template-specific terminology.

### Step 5.3: Create nested component content items

**CRITICAL: Create content items for ALL nested components (items in collections) with actual data from HTML**

For each nested component type (products, features, testimonials, blog posts, team members, etc.):

* Create content items under `/site/components/[item-type]/...`
* **CRITICAL: All nested component XML files MUST have `<component>` as root element**
* **Required Elements**:
  - `<internal-name>` - Human-readable name
  - `<content-type>` - Path to content type
  - `<display-template>` - Path to FTL template (kebab-case, match actual file name)
  - `<objectId>` unqiue GUID
  - All field values as child elements
* Populate fields with actual data from the original HTML
* Reference these items in parent component collections

Example:
```
/site/components/
  products/
    nordic-chair.xml
    kruzo-aero-chair.xml
    ergonomic-chair.xml
  features/
    fast-shipping.xml
    easy-shopping.xml
    support-24-7.xml
  testimonials/
    maria-jones.xml
  blog-posts/
    first-post.xml
    second-post.xml
  team-members/
    john-doe.xml
    jane-smith.xml
```

### Step 5.4: Populate page sections

**CRITICAL: Every page's `sections_o` field must contain component references in the correct order**

For each page:
1. Identify which sections it should have (from original HTML)
2. Create or reference component items for those sections
3. Add them to the page's `sections_o` field in the correct order (match original HTML structure)

Example:
```xml
<sections_o item-list="true">
	<item>
		<key>/site/components/[component-type]/[component-name].xml</key>
		<value>[Component Display Name]</value>
		<include>/site/components/[component-type]/[component-name].xml</include>
		<disableFlattening>false</disableFlattening>
	</item>
	<!-- Add all sections in the order they appear in the original HTML -->
</sections_o>
```

**Note:** Reference components in the exact order they appear in the original HTML page. Match the original page structure.

### Step 5.5: Populate component collections

**CRITICAL: Every component collection field must contain item references in the correct order**

For each component that contains a collection (product-grid, feature-grid, testimonial-slider, etc.):

1. Identify which items it should have (from original HTML)
2. Create or reference nested component items for those items
3. Add them to the component's collection field in the correct order (match original HTML structure)

Example:
```xml
<products_o item-list="true">
	<item>
		<key>/site/components/products/nordic-chair.xml</key>
		<value>Nordic Chair</value>
		<include>/site/components/products/nordic-chair.xml</include>
		<disableFlattening>false</disableFlattening>
	</item>
	<!-- Add all items in the order they appear in the original HTML -->
</products_o>
```

### Step 5.6: Validate URLs and routing

* Ensure each page content item maps to the correct URL
* Ensure internal links are updated to Crafter URLs (not `.html`)
* **CRITICAL: Verify all display-template paths match actual FTL files (kebab-case)**

---

## Phase 6 — QA Checklist (Must Pass)

### Visual parity

* Layout matches original template in browser (CSS/JS intact)
* No broken asset paths
* **Grid items display correctly** - verify CSS framework column/wrapper classes are applied correctly (Bootstrap, Tailwind, CSS Grid, custom, etc.)
* **All div wrappers preserved** - check HTML structure matches original exactly
* **FTL markup and css match the original html templates**

### Content correctness

* All previously hardcoded text that should be editable lives in content
* No page-specific copy hiding inside templates
* a page object exists for each unique template type
* a component object exists for each section of every page
* a content type / form defintion exists for each time of content object
* **All pages have content in `sections_o` fields**
* **All component collections have item references**
* **All nested components have content items with actual data**

### Experience Builder editing

* Click-to-edit works for:
  * titles, headings, body text
  * images + alt text
  * buttons + URLs
  * list items (cards/FAQs/nav links)
* Components can be moved/duplicated if your model supports it (sections list)
* **Nested components (items in collections) are editable in XB**

### Reuse & maintainability

* Header/footer/nav are global, not duplicated per page
* Repeated blocks are components, not copy/paste
* **All components referenced in form definitions have complete form definitions**
* **All form definitions have datasources for image-picker and node-selector fields**

### Template correctness

* **No FreeMarker syntax errors** - test all templates
* **Grid structure preserved** - each grid item wrapped in proper column/wrapper divs matching the original HTML (regardless of CSS framework)
* **Component rendering works** - use items directly from node selectors, not loaded SiteItems
* **All templates match original HTML structure exactly** (line-by-line comparison)
* **All hyphenated properties use bracket notation** (`item['field-name']`)
* **All boolean comparisons are direct** (`<#if field_b>` not `== 'true'`)

### Content type location

* **All content types are in `/config/studio/content-types/`** - NOT `/site/content-types/`
* **All content items are in `/site/`** - pages in `/site/website/`, components in `/site/components/`

### XML Structure

* **All XML files have proper root element** (`<page>` or `<component>`)
* **All XML files have `<internal-name>` element**
* **All XML files have `<objectId>` element with a unique GUID as a value**
* **All XML files have `<lastModifiedDate_dt>` element with today's date time: Example 2025-13-18T18:08:23.638Z**
* **All XML files have `<createdDate_dt>` element with today's date time: Example 2025-13-18T18:08:23.638Z**
* **All display-template paths match actual FTL files** (kebab-case)
* **All URLs use simplified format** (no `/index.xml` or `/index.html`)

### XB Markup

* **No duplicate XB markup on same field**
* **All repeatable groups have correct `$field` and `$index` attributes**
* **All nested repeatable groups use correct field path syntax**
* **All links wrap `<a>` tags, not just content**
* **All images wrap `<img>` tags with `@crafter.img`**

---

## Common Pitfalls to Avoid

### XML Structure
* ❌ Missing root elements in content XML files (`<page>` or `<component>`)
* ❌ Missing `<internal-name>` in content items
* ❌ Wrong `config.xml` format (nested `<id>` instead of attributes)

### FreeMarker Syntax
* ❌ Using dot notation for hyphenated properties (`item.displayTemplate`)
* ❌ Comparing booleans to strings (`active_b == 'true'`)
* ❌ Not saving/restoring `contentModel` when rendering nested components
* ❌ Using `?then()` in string interpolation (causes type errors)
* ❌ Trying to load SiteItems before rendering (use items directly from node selectors)

### HTML Structure
* ❌ Changing HTML element types (`<div>` to `<p>` or vice versa)
* ❌ Changing CSS class names or order
* ❌ Removing or adding wrapper elements that affect CSS
* ❌ Changing link structures (removing `<a>` tags, etc.)
* ❌ Not preserving exact HTML structure (div wrappers, CSS classes, nesting)

### XB Markup
* ❌ Duplicate XB markup on same field (e.g., both `<h3>` and `<span>`)
* ❌ Wrong field paths in repeatable groups
* ❌ Missing `$index` attributes in repeatable groups
* ❌ Wrapping wrong elements (content instead of `<a>` tag)

### Field Types
* ❌ Using `rich-text` instead of `rte`
* ❌ Wrong display-template paths (underscores vs hyphens)
* ❌ Display-template paths that don't match actual file names

### URLs
* ❌ Using `/path/index.xml` or `/path/index.html` instead of `/path`
* ❌ Inconsistent URL formats across content items

### Content Types
* ❌ **PUTTING CONTENT TYPES IN `/site/content-types/` - THEY MUST GO IN `/config/studio/content-types/`**
* ❌ Not creating content types for nested components (items in collections)
* ❌ Referencing components in form definitions without creating their form definitions
* ❌ Not adding datasources to form definitions
* ❌ Not adding `iceId` attributes to form definition fields

### Content Items
* ❌ **NOT creating page content items for each HTML template**
* ❌ **Leaving `sections_o` fields empty on pages**
* ❌ **NOT creating content items for nested components with actual data**
* ❌ **NOT populating component collections with item references**
* ❌ **Using `renderComponentCollection` with `$containerAttributes` for grid items - must manually wrap each item in the exact wrapper divs from the original HTML**

---

## Naming Conventions

* **Template Files**: kebab-case (e.g., `product-section.ftl`)
* **Display Template Paths**: kebab-case (e.g., `/templates/web/components/product-section.ftl`)
* **Content Type Names**: snake_case with prefix (e.g., `/component/cmp_product_section`)
* **Field Names**: use camelCase for the fieldName and snake_case with suffix (e.g., `title_t`, `description_html`, `seoKewords_t`,`items_o`)
* **Content Item Files**: kebab-case (e.g., `home-products.xml`)
* **URLs**: kebab-case, simplified (e.g., `/cart`, `/about-us`)

---

## Summary: Critical Rules

1. **HTML Structure**: Must match original exactly - pixel perfect parity
2. **XML Structure**: All content files need root elements (`<page>` or `<component>`) and internal-name
3. **Content Types**: Use attribute format in config.xml, include all required elements in form-definition.xml, **MUST be in `/config/studio/content-types/`**
4. **Field Types**: Use `rte` for rich text (NOT `rich-text`), `repeatable-group` for collections
5. **Display Templates**: Use kebab-case, match actual file names
6. **FreeMarker**: Use bracket notation for hyphenated properties, direct comparison for booleans, never use `?then()` in strings
7. **XB Markup**: One field per element, wrap correct HTML elements, preserve original structure
8. **URLs**: Use simplified paths (no `/index.xml` or `/index.html`)
9. **Content Items**: Create for EVERY HTML page, populate `sections_o` fields, create nested component items with actual data
10. **Component Collections**: Populate all collections with item references in correct order
11. **Grid Structure**: Wrap each grid item in exact wrapper divs from original HTML
12. **Datasources**: All form definitions must have datasources for image-picker and node-selector fields
