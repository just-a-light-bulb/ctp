**Comic Trans Studio**

Addendum — Design & Technical Deep Dive

Routes · UI System · Canvas Editor · Database Schema

| **§ A**
Route & API Design | **§ B**
Manga UI Design System | **§ C**
Canvas Editor Deep Dive | **§ D**
Optimised DB Schema |
| --- | --- | --- | --- |

**§ A Route & API Design**

SvelteKit page routes + serverless API endpoints — complete path inventory

---

**A.1 SvelteKit Page Routes**

Every URL a user can navigate to is listed below. SvelteKit uses file-based routing inside src/routes/. Routes inside (app)/ are protected by a Kinde auth hook that redirects unauthenticated visitors to /login. Routes inside (auth)/ are public and handle the OAuth flow. The route groups in parentheses are layout-only directories — they do not appear in the URL.

| **Route Path** | **File Location** | **Description** | **Auth** |
| --- | --- | --- | --- |
| **/** | src/routes/+page.svelte | Landing page: hero, feature list, demo GIF, Sign Up CTA | Public |
| **/login** | src/routes/(auth)/login/+page.svelte | Kinde sign-in widget (Google OAuth + magic link) | Public |
| **/auth/callback** | src/routes/(auth)/callback/+server.ts | Kinde OAuth redirect handler — sets session cookie, redirects to /projects | Public |
| **/logout** | src/routes/(auth)/logout/+server.ts | Invalidates session, redirects to / | Public |
| **/projects** | src/routes/(app)/projects/+page.svelte | Project list grid. Each card shows title, source→target lang, chapter count, last updated | Required |
| **/projects/new** | src/routes/(app)/projects/new/+page.svelte | Create project: title, source language, target language, style preset, character glossary upload (CSV/JSON) | Required |
| **/projects/[pid]** | src/routes/(app)/projects/[pid]/+page.svelte | Project home: chapter cards with status, progress bar, quick-translate button | Required |
| **/projects/[pid]/edit** | src/routes/(app)/projects/[pid]/edit/+page.svelte | Edit project metadata and character glossary | Required |
| **/projects/[pid]/chapters/new** | src/routes/(app)/projects/[pid]/chapters/new/+page.svelte | Create chapter: number, title, bulk image upload via Uploadcare widget | Required |
| **/projects/[pid]/chapters/[cid]** | src/routes/(app)/projects/[pid]/chapters/[cid]/+page.svelte | Chapter dashboard: page thumbnail grid, per-page status badges, batch-process all button | Required |
| **/projects/[pid]/chapters/[cid]/edit** | src/routes/(app)/projects/[pid]/chapters/[cid]/edit/+page.svelte | Edit chapter title, reorder pages, delete pages | Required |
| **/editor/[pageId]** | src/routes/(app)/editor/[pageId]/+page.svelte | Unified editor: canvas panel (left) + translation spreadsheet (right). Core app surface. | Required |
| **/editor/[pageId]/preview** | src/routes/(app)/editor/[pageId]/preview/+page.svelte | Full-screen preview of rendered final page (cleaned image + translated text overlaid) | Required |
| **/settings** | src/routes/(app)/settings/+page.svelte | User profile, OpenRouter API key, default translation style, notification preferences | Required |
| **/settings/plan** | src/routes/(app)/settings/plan/+page.svelte | Storage usage meter, plan comparison, upgrade button | Required |
| **/404** | src/routes/+error.svelte | Custom 404 / error page styled as a manga panel with an confused character illustration | Public |

**A.2 API Route Design**

All API routes are SvelteKit server endpoints living in src/routes/api/. They are deployed as Vercel serverless functions. Each endpoint follows a consistent convention: POST for create/trigger actions, GET for reads, PATCH for partial updates, DELETE for removals. All endpoints return JSON with a consistent envelope: { ok: boolean, data?: any, error?: string }.

**Authentication Middleware**

A hooks.server.ts guard runs before every request. For routes under /api/, it validates the Kinde session JWT from the Authorization header (Bearer token). If invalid or missing, it returns 401. This single check means no individual API route needs to repeat auth logic.

| **Endpoint** | **Method + Purpose** | **Key Request / Response** |
| --- | --- | --- |
| **POST /api/pages** | Create page record after Uploadcare upload completes. Called by the chapter creation form's onUpload callback. | body: { chapterId, uploadcareUrl, pageNumber } → { page: Page } |
| **GET /api/pages/[id]** | Fetch a single page with all its TextRegions and job status. Used by the editor on mount. | → { page, regions: TextRegion[], job: TranslationJob | null } |
| **DELETE /api/pages/[id]** | Delete a page and all its TextRegions. Triggers Uploadcare file deletion via their REST API. | → { ok: true } |
| **POST /api/jobs/process** | Queue a full translation job for one page. Writes TranslationJob row with status='queued'. Returns immediately; work happens async. | body: { pageId, modelOverride? } → { jobId } |
| **POST /api/jobs/process-batch** | Queue translation jobs for all pages in a chapter that haven't been processed yet. | body: { chapterId } → { queued: number } |
| **GET /api/jobs/[id]** | Poll job status. Used by the progress indicator on the chapter dashboard. | → { job: TranslationJob } |
| **GET /api/cron/run-jobs** | Called by Vercel Cron every 30 seconds. Picks the oldest queued job, runs the 3-step AI pipeline, saves results, marks job done. | Vercel Cron secret in header. Returns { processed: jobId | null } |
| **POST /api/ai/vision** | Run Step 1 Vision Agent on a page image URL. Returns raw bounding boxes + OCR text. Internal — called only by /api/cron/run-jobs. | body: { pageId, imageUrl } → { regions: RawRegion[] } |
| **POST /api/ai/characters** | Run Step 2 Character ID Agent. Takes page image + region list. Returns speaker metadata per region. | body: { pageId, imageUrl, regions } → { speakers: SpeakerMeta[] } |
| **POST /api/ai/translate** | Run Step 3 Translation Agent. Takes regions + speaker meta + project glossary. Returns translated text per region. | body: { pageId, regions, speakers, glossary, memory } → { translations: Translation[] } |
| **PATCH /api/regions/[id]** | Update a single TextRegion. Used by the editor spreadsheet when a translator edits a cell. | body: Partial<TextRegion> (translatedText, isApproved, speakerGender, fontSizeOverride) → { region } |
| **POST /api/regions/bulk-approve** | Approve all regions for a page that meet a minimum confidence threshold. | body: { pageId, minConfidence: 0-100 } → { approved: number } |
| **POST /api/regions/manual** | Create a manually drawn text region (no AI detection). Optionally trigger single-bubble translation. | body: { pageId, bbox, originalText, translate?: boolean } → { region } |
| **DELETE /api/regions/[id]** | Delete a single TextRegion (e.g. a falsely detected region). | → { ok: true } |
| **POST /api/render/page** | Compose final image for one page: inpaint original text → overlay translated text. Stores result as Page.renderedImageUrl. | body: { pageId } → { renderedUrl } |
| **POST /api/render/chapter** | Trigger render for all approved pages in a chapter in sequence. | body: { chapterId } → { rendered: number } |
| **POST /api/export/zip** | Generate ZIP archive of all rendered page PNGs for a chapter. Returns a signed Uploadcare CDN URL. | body: { chapterId } → { downloadUrl } |
| **POST /api/export/pdf** | Combine all rendered pages into a single PDF using pdf-lib. Returns CDN URL. | body: { chapterId } → { downloadUrl } |
| **GET /api/projects** | List all projects for authenticated user. | → { projects: Project[] } |
| **POST /api/projects** | Create project. | body: { title, srcLang, tgtLang, glossary? } → { project } |
| **PATCH /api/projects/[id]** | Update project metadata. | → { project } |
| **DELETE /api/projects/[id]** | Delete project and cascade all chapters/pages/regions. | → { ok: true } |
| **POST /api/chapters** | Create chapter. | body: { projectId, title, number } → { chapter } |
| **PATCH /api/chapters/[id]** | Update chapter title or status. | → { chapter } |
| **DELETE /api/chapters/[id]** | Delete chapter and cascade. | → { ok: true } |
| **POST /api/storage/sign** | Generate Uploadcare signed upload URL. Called client-side before uploading so the secret API key stays server-only. | body: { fileName, mimeType } → { uploadUrl, fileId } |
| **DELETE /api/storage/[fileId]** | Delete a file from Uploadcare. Called when pages are deleted. | → { ok: true } |

**A.3 Cron Job Flow (Detailed)**

The most critical API route is /api/cron/run-jobs because it is the engine that drives the entire AI pipeline. Understanding it in detail prevents the most common async bugs. The cron job is configured in vercel.json and fires every 30 seconds. It is protected by a shared secret header (CRON_SECRET env var) so only Vercel's scheduler can call it — any external call returns 401.

// Vercel Cron + Job Queue Pattern

---

// vercel.json

{

"crons": [{ "path": "/api/cron/run-jobs", "schedule": "*/30 * * * * *" }]

}

// /api/cron/run-jobs logic (pseudocode)

1. SELECT oldest TranslationJob WHERE status = 'queued' LIMIT 1 (for concurrency safety)

2. UPDATE job SET status = 'running', startedAt = now()

3. CALL /api/ai/vision → save TextRegion rows (status='detected')

4. CALL /api/ai/characters → update TextRegion rows (speakerName, genderCode, thaiParticle)

5. CALL /api/ai/translate → update TextRegion rows (translatedText, confidence)

6. UPDATE job SET status = 'done', completedAt = now()

ON ERROR: UPDATE job SET status = 'error', errorMessage = err.message

→ client polls GET /api/jobs/[id] every 3s to detect completion

---

**§ B Manga UI Design System**

Visual language, colour palette, typography, components — inspired by ink, paper & panels

---

**B.1 Design Philosophy**

The aesthetic direction is 'Ink on Paper' — a UI that feels like you are working inside a manga itself, not just a tool about manga. Every design decision should reinforce this: the colours come from a printing workshop (not a tech company), the typography feels like it belongs in a tankobon volume, and interactive elements echo the visual language of manga panels, speech bubbles, and speed lines.

The key tension to navigate is usability versus aesthetics. A manga-inspired UI that makes text hard to read or buttons hard to find is a failure. The goal is selective theatricality: the structural chrome (sidebars, headers, panels) carries the aesthetic weight, while the working surfaces (the translation spreadsheet, the canvas) stay clean and functional.

**B.2 Colour Palette**

| **Token Name** | **Value & Usage** |
| --- | --- |
| **--ink-black** | #1A1A2E — primary text, borders, logo. The ink of a pen on paper. |
| **--panel-red** | #C0392B — primary accent. Action buttons, active states, error states. The red stamp (hanko) of manga editors. |
| **--cream-paper** | #FDF6E3 — app background and page surfaces. The warm off-white of aged manga paper. |
| **--screen-blue** | #4A90D9 — secondary accent for info states, links, AI-suggested content. The blue pencil used for non-photo editing marks. |
| **--gold-tone** | #D4A853 — tertiary accent for highlights, gold-star ratings, premium features. The gilt edges of a deluxe edition. |
| **--manga-navy** | #2C2C54 — sidebar backgrounds, table headers, modal overlays. The deep blue-black of a manga cover. |
| **--panel-light** | #FFF8F0 — alternating table row backgrounds, card surfaces. Slightly warmer than white. |
| **--tone-gray** | #C8B8A2 — screentone pattern fills, disabled states, placeholder text. The halftone dot grey of manga shadows. |
| **--ink-muted** | #8B8680 — secondary text, metadata labels, timestamps. |
| **--border-line** | #E8E0D5 — subtle panel borders, dividers, input outlines. The lighter ruled line of a sketchbook. |

**B.3 Typography**

Typography is one of the most powerful signals of a theme. The font pairing below is chosen to feel editorial and slightly literary — the kind of refinement you would find in a high-end manga magazine (like Big Comic Original), not a typical SaaS dashboard.

| **Role** | **Font & Rationale** |
| --- | --- |
| **Display / H1** | Shippori Mincho B1 (Google Fonts) — a high-contrast Japanese-influenced serif with strong ink-stroke character. Used for page titles, project names, and chapter numbers. The vertical-stroke contrast directly echoes the kanji brushwork heritage of manga typography. |
| **Headings / H2–H3** | Cormorant Garamond Bold — an extremely refined, high-contrast Western serif that pairs with Shippori's aesthetic. Used for section headings, panel headers, and card titles. |
| **Body text / Labels** | Noto Serif — the universal serif designed to display all scripts correctly, including Thai (essential since the UI is bilingual Thai/English). Used for all body copy, table content, and form labels. |
| **UI Controls / Tags** | DM Mono — a geometric monospaced font for badges, confidence scores, keyboard shortcut labels, and code blocks. The monospace feel is deliberately technical, contrasting with the handcraft serif aesthetic. |
| **Thai text rendering** | Noto Serif Thai — must be loaded explicitly alongside Noto Serif so that Thai characters in translated content render with the same weight and baseline as the surrounding Latin characters. |

**⚠️ Font Loading Strategy**

Load all four fonts via a single @font-face import from the Google Fonts CDN, using the display=swap parameter to prevent layout shift. In the SvelteKit app.html, add the preconnect link tags for fonts.googleapis.com and fonts.gstatic.com before the font link element. This reduces Time to First Meaningful Paint from ~400ms to ~80ms on repeat visits because browsers cache the font files.

---

**B.4 Spacing & Layout Scale**

Use a base unit of 4px (var(--unit)). All spacing values are multiples of this unit: 4, 8, 12, 16, 24, 32, 48, 64, 96. This creates a consistent vertical rhythm and makes it easy to align components across the two-panel editor layout. The exception is the canvas panel, which uses pixel-exact positioning driven by the image dimensions.

**B.5 Core UI Component Guide**

**Panel Frame (the fundamental container)**

Every major content area is wrapped in a panel-frame — a div with a solid 2px border in --border-line, a subtle box shadow (2px 4px 0 rgba(26,26,46,0.08)), and a paper-cream background. The border-radius is exactly 0 for all primary panels. Zero border-radius is a deliberate aesthetic choice that recalls the sharp-cornered panels of a physical manga page. Only secondary UI elements (tooltips, dropdown menus, badges) use a small 3px radius.

CSS — panel-frame

---

.panel-frame {

border: 2px solid var(--border-line);

box-shadow: 2px 4px 0 rgba(26,26,46,0.08);

background: var(--cream-paper);

border-radius: 0; /* intentional — no rounded corners on panels */

position: relative;

}

/* Panel with red accent border on the left (for active/focused panels) */

.panel-frame--active {

border-left: 4px solid var(--panel-red);

}

---

**Speed-Line Divider**

Between major sections of the sidebar and at the top of the editor toolbar, use a speed-line divider — a thin horizontal rule with 5–7 diagonal dashes radiating outward from the left edge, drawn as a CSS border gradient. This is purely decorative but creates a strong manga identity moment that users immediately associate with the aesthetic.

CSS — speed-line divider

---

.speed-divider {

height: 2px;

background: repeating-linear-gradient(

- 60deg,

var(--ink-black) 0px,

var(--ink-black) 2px,

transparent 2px,

transparent 8px

);

opacity: 0.15;

margin: 16px 0;

}

---

**Speech Bubble Badge**

Confidence score badges and speaker labels are styled as manga speech bubbles — a pill shape (border-radius: 50vh) with a tiny triangular tail pointing downward. The tail is a pure CSS :after pseudo-element trick. High confidence (>85%) uses --screen-blue fill; medium (60–85%) uses --gold-tone; low (<60%) uses --panel-red. This lets translators see confidence at a glance without reading any numbers.

CSS — speech bubble badge

---

.bubble-badge {

display: inline-block;

padding: 2px 10px;

border-radius: 50vh;

font-family: 'DM Mono', monospace;

font-size: 11px;

color: white;

position: relative;

}

.bubble-badge::after { /* the tail */

content: '';

position: absolute;

bottom: -5px; left: 50%;

transform: translateX(-50%);

border: 3px solid transparent;

border-top-color: inherit; /* inherits the badge background via border trick */

}

.bubble-badge--high { background: var(--screen-blue); }

.bubble-badge--medium { background: var(--gold-tone); }

.bubble-badge--low { background: var(--panel-red); }

---

**Action Button**

Primary buttons use a bold black fill with white text and a hard offset shadow (3px 3px 0 var(--panel-red)) — a technique borrowed from retro manga lettering and zine design. On hover, the shadow offset reduces to 1px 1px, creating a 'press down' tactile feel. There is no border-radius. Secondary buttons are outlined with a 2px ink-black border and no fill.

CSS — action button

---

.btn-primary {

background: var(--ink-black);

color: white;

border: none;

border-radius: 0;

padding: 10px 24px;

font-family: 'DM Mono', monospace;

font-size: 13px;

letter-spacing: 0.05em;

text-transform: uppercase;

box-shadow: 3px 3px 0 var(--panel-red);

cursor: pointer;

transition: box-shadow 80ms ease, transform 80ms ease;

}

.btn-primary:hover {

box-shadow: 1px 1px 0 var(--panel-red);

transform: translate(2px, 2px);

}

---

**Halftone Texture Overlay**

The app background (behind all panels) uses a CSS-only halftone dot pattern at very low opacity (4%). This is the single most impactful detail for establishing the manga printing aesthetic — it immediately reads as 'screentone' to any manga reader. It is applied as a radial-gradient repeating background on the body element and has no performance cost.

CSS — halftone body texture

---

body {

background-color: var(--cream-paper);

background-image: radial-gradient(

circle,

var(--tone-gray) 1px,

transparent 1px

);

background-size: 12px 12px;

}

---

**B.6 Page-by-Page UI Specification**

| **Page / Route** | **UI Description** |
| --- | --- |
| **/ (Landing)** | Full-width manga-panel grid layout. Four panels show screenshots of the app with caption boxes beneath each. Hero headline uses Shippori Mincho at 64px. CTA button is panel-red. Background has the halftone texture. A 'screentone' gradient fades from cream to white near the bottom. |
| **/projects (List)** | Cards arranged in a masonry grid (3 columns desktop, 2 tablet, 1 mobile). Each card has: project cover thumbnail (first page of chapter 1), a title in Shippori Mincho, language pair shown as JP → TH with an arrow glyph, chapter count and last-updated date in DM Mono. Card hover lifts with the box-shadow press effect. |
| **/projects/[pid] (Detail)** | Chapter cards in a 2-column grid. Each card has a thumbnail strip of the first 3 page images, a chapter number in a large Cormorant numeral (the manga volume-number style), and a progress bar. The progress bar is styled as a thin ink stroke that fills left to right. |
| **/projects/[pid]/chapters/[cid] (Dashboard)** | Page thumbnails in a scrollable 4-column grid. Each thumbnail has a coloured status badge in the bottom-right corner (a speech bubble badge: pending, processing, translated, done). Clicking a thumbnail navigates to the editor. A sticky toolbar at the top has the Translate All and Export buttons. |
| **/editor/[pageId] (Main Editor)** | Two-panel layout with a draggable resize handle. Left panel: the Canvas. Right panel: the Translation Spreadsheet. Both panels are panel-frame containers. The top bar shows the chapter/page breadcrumb in Shippori Mincho and the action buttons (Render, Export). A thin speed-line divider separates the top bar from the panels. |
| **/settings** | Single-column form layout with section headings as panel-frame headers. API key field uses a show/hide toggle. Storage meter is a horizontal bar with an ink-stroke style, not a rounded pill. |

**§ C Canvas Editor — Step-by-Step Implementation**

How to build the split-pane manga page editor from first principles

---

**C.1 Architecture Decision: Fabric.js vs Raw Canvas API**

Before writing a single line of canvas code, the most important decision is whether to use Fabric.js (a Canvas abstraction layer) or the browser's raw Canvas 2D API. Both are valid. Understanding the trade-offs prevents you from having to rewrite the editor midway through.

| **Fabric.js** | **Raw Canvas 2D API** |
| --- | --- |
| **Gives you free object model: every shape, image, and text box is a JS object with properties. Selection, dragging, and resize handles work out of the box.** | You manage everything yourself: which objects exist, where they are, which one is selected, how to redraw them. |
| **Built-in event system: object:selected, object:modified, object:moving work immediately without custom hit-testing code.** | You must implement hit-testing manually: on mousedown, iterate through all regions and check if the click point is inside any bounding box. |
| **50 KB gzipped bundle cost. Some quirks with high-DPI (Retina) screens that require manual pixel ratio handling.** | Zero bundle cost. Full control. Retina/HiDPI is handled once by setting canvas.width = naturalWidth * devicePixelRatio. |
| **RECOMMENDATION: Use Fabric.js for Phase 1 MVP. Switch to raw Canvas only if you hit a specific performance ceiling (typically > 500 objects per page, which is unrealistic for manga).** | Better choice if you are building a generalized raster editor. Overkill for the focused use case of manga bubble overlays. |

The steps below use Fabric.js because it dramatically reduces implementation time for the overlay, selection, and resize interactions that are the core of the editor experience.

**C.2 Step 1 — Project Setup**

Install Fabric.js and its types. Because Fabric has a complex object model, TypeScript types are essential to avoid runtime errors when accessing object properties.

Terminal + SvelteKit SSR guard

---

# Install Fabric.js (v5 — stable, well-documented)

npm install fabric

npm install --save-dev @types/fabric

# In the SvelteKit editor route, Fabric must be imported dynamically

# because it uses browser-only APIs and will crash during SSR:

# src/routes/(app)/editor/[pageId]/+page.svelte

import { onMount } from 'svelte';

let fabricModule: typeof import('fabric') | null = null;

onMount(async () => {

fabricModule = await import('fabric');

initCanvas();

});

---

**C.3 Step 2 — Canvas Initialisation & HiDPI Handling**

The single most common canvas bug is blurry rendering on Retina screens. It happens because the HTML canvas element has two separate sizes: the CSS display size (what you set with width/height CSS) and the internal pixel buffer size (what you set with canvas.width and canvas.height attributes). On a 2x display, you must set the buffer to 2x the display size, then scale the drawing context to match.

SvelteKit component — canvas init

---

// src/lib/components/editor/CanvasEditor.svelte

let canvasEl: HTMLCanvasElement;

let fc: fabric.Canvas; // the Fabric canvas instance

function initCanvas() {

const { Canvas } = fabricModule!;

const dpr = window.devicePixelRatio || 1;

const container = canvasEl.parentElement!;

const displayW = container.clientWidth;

const displayH = container.clientHeight;

// Set the HTML attribute sizes to the physical pixel count

canvasEl.width = displayW * dpr;

canvasEl.height = displayH * dpr;

// Set the CSS display size back to logical pixels

canvasEl.style.width = `${displayW}px`;

canvasEl.style.height = `${displayH}px`;

fc = new Canvas(canvasEl, {

selection: true, // allow multi-select with shift-click

preserveObjectStacking: true,

renderOnAddRemove: false, // batch renders for performance

});

// Scale all Fabric drawing operations to match device pixel ratio

fc.setDimensions({ width: displayW, height: displayH });

fc.setZoom(1); // will be adjusted when image loads

}

---

**C.4 Step 3 — Loading the Manga Page Image**

The manga page image is stored on Uploadcare's CDN. Fabric.js loads it as a background image. The key detail is calculating a zoom level that fits the image within the canvas container while preserving its aspect ratio. This initial zoom value is stored and used as a reference for all subsequent zoom operations.

Loading image from Uploadcare CDN

---

let baseZoom = 1; // the fit-to-container zoom level

let currentZoom = 1;

async function loadPageImage(imageUrl: string) {

return new Promise<void>((resolve) => {

fabric.Image.fromURL(

imageUrl,

(img) => {

const containerW = fc.getWidth();

const containerH = fc.getHeight();

const imgW = img.naturalWidth || img.width!;

const imgH = img.naturalHeight || img.height!;

// Calculate zoom to fit the image in the container

baseZoom = Math.min(containerW / imgW, containerH / imgH);

currentZoom = baseZoom;

fc.setZoom(baseZoom);

// Position image at top-left of the zoomed viewport

img.set({ left: 0, top: 0, selectable: false, evented: false });

fc.setBackgroundImage(img, fc.renderAll.bind(fc));

resolve();

},

{ crossOrigin: 'anonymous' } // required for Uploadcare CDN images

);

});

}

---

**C.5 Step 4 — Rendering Text Region Overlays**

Each TextRegion from the database has a bounding box (x, y, w, h) in the coordinate space of the original image. Since the canvas is zoomed, we must NOT multiply these coordinates by the zoom level — Fabric.js handles that automatically when you set the canvas zoom. You simply draw the rectangle at the original image coordinates and Fabric scales it correctly.

Each overlay is a Fabric Rect (the coloured bounding box) grouped with a Fabric Text (the region index number). The Rect's stroke colour encodes status: green for approved, amber for pending, red for low confidence. The group is stored in a Map keyed by regionId so we can find and update it later.

Rendering bounding box overlays

---

// Colour constants matching the design system

const COLOUR = { approved: '#22c55e', pending: '#f59e0b', low: '#c0392b' };

const regionObjects = new Map<string, fabric.Group>(); // regionId → Fabric group

function renderRegion(region: TextRegion) {

const { x, y, w, h } = region.bbox; // original image coords

const colour = region.isApproved ? COLOUR.approved

: region.confidence > 75 ? COLOUR.pending : COLOUR.low;

const rect = new fabric.Rect({

left: 0, top: 0, width: w, height: h,

fill: `${colour}22`, // 13% opacity fill

stroke: colour,

strokeWidth: 2 / currentZoom, // keep stroke 2px regardless of zoom

rx: 0, ry: 0,

selectable: true,

hasControls: true, // show resize handles

hasBorders: true,

cornerStyle: 'circle',

cornerColor: colour,

cornerSize: 8,

transparentCorners: false,

});

const label = new fabric.Text(String(region.bubbleIndex), {

left: 4, top: 4,

fontSize: 14 / currentZoom, // scale label to stay readable

fill: colour,

fontFamily: 'DM Mono',

selectable: false,

evented: false,

});

const group = new fabric.Group([rect, label], {

left: x, top: y,

data: { regionId: region.id }, // store id for event handlers

});

regionObjects.set(region.id, group);

fc.add(group);

}

// Call for each region after image loads:

regions.forEach(renderRegion);

fc.requestRenderAll();

---

**C.6 Step 5 — Bidirectional Sync (Canvas ↔︎ Spreadsheet)**

The bidirectional sync is the single most important UX feature of the editor and also the most common source of infinite-loop bugs if not implemented carefully. The pattern to use is an activeRegionId Svelte store. When the user clicks in the canvas, the Fabric event handler writes to this store. When the user clicks a spreadsheet row, the Svelte handler writes to this store. Both panels read from the store — they never communicate with each other directly.

Bidirectional sync via Svelte store

---

// src/lib/stores/editor.ts

import { writable } from 'svelte/store';

export const activeRegionId = writable<string | null>(null);

// ─── In CanvasEditor.svelte ───────────────────────────────────────────────

// When the user clicks an overlay in the canvas:

fc.on('selection:created', (e) => {

const obj = e.selected?.[0];

if (obj?.data?.regionId) {

activeRegionId.set(obj.data.regionId); // write to store

}

});

fc.on('selection:cleared', () => activeRegionId.set(null));

// When the store changes from the OUTSIDE (spreadsheet click),

// select the matching Fabric object — but only if we didn't just set it:

let suppressCanvasSync = false;

activeRegionId.subscribe((id) => {

if (suppressCanvasSync || !id) return;

const group = regionObjects.get(id);

if (group) {

suppressCanvasSync = true;

fc.setActiveObject(group);

// Pan to centre the selected region in the viewport

const vpt = fc.viewportTransform!;

const centre = group.getCenterPoint();

vpt[4] = fc.getWidth() / 2 - centre.x * currentZoom;

vpt[5] = fc.getHeight() / 2 - centre.y * currentZoom;

fc.requestRenderAll();

suppressCanvasSync = false;

}

});

// ─── In TranslationTable.svelte ──────────────────────────────────────────

// When a spreadsheet row is clicked:

function selectRow(regionId: string) {

activeRegionId.set(regionId);

// Svelte will auto-scroll the row into view via bind:this + scrollIntoView

}

// Highlight active row:

$: activeId = $activeRegionId; // reactive to store changes

---

**C.7 Step 6 — Zoom & Pan**

Zoom and pan are essential for working with high-resolution manga pages. Fabric.js provides setZoom() and relativePan() but you need to add the mouse and touch event listeners yourself. The pattern below implements zoom-to-cursor, which means the image appears to expand outward from the cursor position rather than from the top-left corner.

Zoom-to-cursor + pan implementation

---

const ZOOM_MIN = 0.1;

const ZOOM_MAX = 5;

let isPanning = false;

let lastPanPoint = { x: 0, y: 0 };

// ─── Zoom with mouse wheel ────────────────────────────────────────────────

fc.on('mouse:wheel', (opt) => {

const delta = opt.e.deltaY;

let zoom = fc.getZoom();

zoom *= 0.999 ** delta; // smooth logarithmic zoom

zoom = Math.min(Math.max(zoom, ZOOM_MIN), ZOOM_MAX);

// zoom-to-cursor: keep the point under the cursor stationary

fc.zoomToPoint({ x: opt.e.offsetX, y: opt.e.offsetY }, zoom);

currentZoom = zoom;

opt.e.preventDefault();

opt.e.stopPropagation();

updateOverlayLineWidths(); // rescale stroke widths after zoom

});

// ─── Pan with middle-mouse or spacebar+drag ───────────────────────────────

fc.on('mouse:down', (opt) => {

if (opt.e.button === 1 || isSpaceHeld) { // middle-click or space+drag

isPanning = true;

fc.setCursor('grabbing');

lastPanPoint = { x: opt.e.clientX, y: opt.e.clientY };

fc.selection = false; // disable selection while panning

}

});

fc.on('mouse:move', (opt) => {

if (!isPanning) return;

const dx = opt.e.clientX - lastPanPoint.x;

const dy = opt.e.clientY - lastPanPoint.y;

fc.relativePan({ x: dx, y: dy });

lastPanPoint = { x: opt.e.clientX, y: opt.e.clientY };

});

fc.on('mouse:up', () => {

isPanning = false;

fc.selection = true;

fc.setCursor('default');

});

---

**C.8 Step 7 — Manual Region Drawing**

When the AI misses a text region, the translator needs to draw a new one. The drawing mode is toggled by a toolbar button. When active, mousedown starts recording the start corner and mouseup commits the rectangle as a new TextRegion.

Manual region drawing mode

---

let isDrawingMode = false;

let drawStart: { x: number; y: number } | null = null;

let drawPreview: fabric.Rect | null = null;

export function enterDrawMode() {

isDrawingMode = true;

fc.selection = false; // disable normal selection

fc.setCursor('crosshair');

}

fc.on('mouse:down', (opt) => {

if (!isDrawingMode) return;

// Convert screen coords to image coords (accounting for zoom + pan)

const pointer = fc.getPointer(opt.e);

drawStart = { x: pointer.x, y: pointer.y };

drawPreview = new fabric.Rect({

left: drawStart.x, top: drawStart.y,

width: 0, height: 0,

stroke: '#c0392b', strokeWidth: 2 / currentZoom,

fill: 'rgba(192,57,43,0.1)',

selectable: false,

});

fc.add(drawPreview);

});

fc.on('mouse:move', (opt) => {

if (!isDrawingMode || !drawStart || !drawPreview) return;

const pointer = fc.getPointer(opt.e);

const w = pointer.x - drawStart.x;

const h = pointer.y - drawStart.y;

drawPreview.set({ width: Math.abs(w), height: Math.abs(h),

left: w < 0 ? pointer.x : drawStart.x,

top: h < 0 ? pointer.y : drawStart.y });

fc.requestRenderAll();

});

fc.on('mouse:up', async (opt) => {

if (!isDrawingMode || !drawStart || !drawPreview) return;

fc.remove(drawPreview);

const bbox = { x: drawPreview.left!, y: drawPreview.top!,

w: drawPreview.width!, h: drawPreview.height! };

if (bbox.w > 10 && bbox.h > 10) { // ignore accidental clicks

// POST to /api/regions/manual — server creates the DB row

const region = await createManualRegion(pageId, bbox);

renderRegion(region); // add it to the canvas immediately

}

drawStart = null; drawPreview = null;

isDrawingMode = false;

fc.selection = true;

fc.setCursor('default');

});

---

**C.9 Step 8 — Undo / Redo Stack**

Undo/redo must be custom-implemented because Fabric.js does not have a built-in history. The strategy is a command pattern: every mutation (region created, translated text changed, bbox moved) is represented as an object with an execute() and an undo() method. A stack of executed commands is kept in memory, and Ctrl+Z pops and undoes the most recent one.

Command-pattern undo/redo

---

interface Command {

execute(): Promise<void>;

undo(): Promise<void>;

description: string;

}

const undoStack: Command[] = [];

const redoStack: Command[] = [];

export async function executeCommand(cmd: Command) {

await cmd.execute();

undoStack.push(cmd);

redoStack.length = 0; // clear redo stack on new action

}

export async function undo() {

const cmd = undoStack.pop();

if (!cmd) return;

await cmd.undo();

redoStack.push(cmd);

}

export async function redo() {

const cmd = redoStack.pop();

if (!cmd) return;

await cmd.execute();

undoStack.push(cmd);

}

// Example: wrapping a translation edit as a Command

function makeEditTranslationCommand(regionId: string, oldText: string, newText: string): Command {

return {

description: `Edit translation for region ${regionId}`,

execute: () => patchRegion(regionId, { translatedText: newText }),

undo: () => patchRegion(regionId, { translatedText: oldText }),

};

}

---

**C.10 Step 9 — Preview Mode (Cleaned Image + Text Overlay)**

Preview mode shows the final output: the cleaned image (with original text removed) with the translated Thai text rendered inside each bubble. This is a read-only canvas mode — all Fabric overlays are hidden and replaced with a single composited preview image. Toggling back to edit mode restores the overlays.

Preview mode toggle

---

let isPreviewMode = false;

async function togglePreview() {

isPreviewMode = !isPreviewMode;

if (isPreviewMode) {

// Hide all region overlays

regionObjects.forEach(group => group.set('visible', false));

// Load the rendered image (POST /api/render/page first if not yet rendered)

const { renderedUrl } = await renderPage(pageId);

fabric.Image.fromURL(renderedUrl, (img) => {

img.set({ left: 0, top: 0, selectable: false, evented: false,

data: { isPreview: true } });

fc.add(img);

fc.sendToBack(img); // behind overlays (which are hidden anyway)

fc.requestRenderAll();

}, { crossOrigin: 'anonymous' });

} else {

// Remove preview image and restore overlays

fc.getObjects().filter(o => o.data?.isPreview).forEach(o => fc.remove(o));

regionObjects.forEach(group => group.set('visible', true));

fc.requestRenderAll();

}

}

---

**§ D Storage-Optimised Database Schema**

Prisma schema designed to minimise database size and query cost

---

**D.1 Optimisation Principles**

The naive schema for this application would be expensive in storage because of three patterns: storing large JSON blobs in every row (bounding boxes, glossaries), storing the same string values repeated across thousands of rows (language codes, AI model names, status strings), and storing floating-point numbers where integers would suffice. The schema below addresses all three with specific techniques.

| **Problem** | **Solution Applied** |
| --- | --- |
| **Status fields stored as VARCHAR ('queued', 'running', 'done') — wastes 5–10 bytes per row on string overhead vs a single byte for an integer.** | Use Prisma's native enum type, which maps to a SMALLINT (2 bytes) in Postgres. All status/type/language fields are enums. |
| **Bounding boxes stored as a JSON column { x, y, w, h } — JSON has ~40 bytes overhead per column for the structural characters and key names, even for 4 numbers.** | Store bbox as four separate SMALLINT columns (bboxX, bboxY, bboxW, bboxH). SMALLINT is 2 bytes each = 8 bytes total for the bbox vs ~60 bytes as JSON. At 50 regions/page × 500 pages, this saves ~1.3 MB per project. |
| **Confidence stored as Float (8 bytes) — six decimal places of precision are useless for a 0-100% score.** | Store confidence as a SMALLINT (0–100). Reduces from 8 bytes to 2 bytes per region row. |
| **Image URLs stored in full in every Page row — Uploadcare URLs follow a predictable pattern (https://ucarecdn.com/{fileId}/) so only the 36-character UUID fileId needs to be stored.** | Store only the fileId (UUID, 16 bytes in Postgres UUID type vs ~60 bytes as text). Reconstruct the full URL at query time with a computed field or application-layer function. |
| **Translation memory stored as repeated column values across many TextRegion rows with the same original text.** | Add a TranslationMemory table with a unique constraint on (projectId, sourceText). TextRegion rows reference it by FK instead of duplicating the original text. This also makes cross-chapter consistency queries trivial. |
| **Character glossary stored as a JSON blob on the Project row — hard to query and grows unbounded.** | Normalise into a ProjectCharacter table with indexed columns for characterName and genderCode. Enables efficient lookup during translation and clean audit trails when names change. |

**D.2 Enum Definitions**

Prisma enums (all map to SMALLINT)

---

// All enums are stored as SMALLINT in Postgres — not VARCHAR

enum Lang {

JA // Japanese

ZH // Chinese

KO // Korean

TH // Thai

EN // English

VI // Vietnamese

}

enum PageStatus {

PENDING // uploaded, not yet processed

PROCESSING // AI job running

DETECTED // OCR done, awaiting translation

TRANSLATED // translation done, awaiting review

APPROVED // all regions approved

RENDERED // final image composed

}

enum JobStatus {

QUEUED

RUNNING

DONE

ERROR

}

enum GenderCode {

MALE

FEMALE

NEUTRAL

UNKNOWN

}

enum RegionType {

DIALOGUE // speech bubble

THOUGHT // thought balloon

NARRATION // caption/narration box

SFX // sound effect

SIGNAGE // background text (signs, posters)

}

enum TranslationStyle {

NATURAL // idiomatic, natural Thai

LITERAL // closer to source structure

FORMAL // polite/formal register

CASUAL // informal/youth register

}

---

**D.3 Full Schema**

schema.prisma — full optimised schema

---

// schema.prisma — storage-optimised

generator client {

provider = 'prisma-client-js'

}

datasource db {

provider = 'postgresql'

url = env('DATABASE_URL')

}

model User {

id String @id @default(cuid()) // cuid = 25 chars, indexed efficiently

kindeId String @unique // the Kinde user ID

email String @unique

isPro Boolean @default(false)

apiKey String? @db.VarChar(200) // encrypted OpenRouter key

createdAt DateTime @default(now())

projects Project[]

}

model Project {

id String @id @default(cuid())

userId String

user User @relation(fields: [userId], references: [id], onDelete: Cascade)

title String @db.VarChar(120) // cap title length

srcLang Lang

tgtLang Lang

style TranslationStyle @default(NATURAL)

createdAt DateTime @default(now())

chapters Chapter[]

characters ProjectCharacter[]

memory TranslationMemory[]

@@index([userId])

}

model ProjectCharacter {

id Int @id @default(autoincrement()) // Int not String: saves 21 bytes/row

projectId String

project Project @relation(fields: [projectId], references: [id], onDelete: Cascade)

name String @db.VarChar(80)

gender GenderCode @default(UNKNOWN)

particle String? @db.VarChar(10) // Thai polite particle override

notes String? @db.VarChar(300)

@@unique([projectId, name]) // no duplicate character names per project

@@index([projectId])

}

model Chapter {

id Int @id @default(autoincrement())

projectId String

project Project @relation(fields: [projectId], references: [id], onDelete: Cascade)

number SmallInt // chapter number (saves 2 bytes vs Int)

title String @db.VarChar(120)

status PageStatus @default(PENDING)

createdAt DateTime @default(now())

pages Page[]

@@unique([projectId, number]) // no duplicate chapter numbers

@@index([projectId])

}

model Page {

id Int @id @default(autoincrement())

chapterId Int

chapter Chapter @relation(fields: [chapterId], references: [id], onDelete: Cascade)

pageNumber SmallInt // SmallInt: pages rarely exceed 32767

srcFileId String @db.Uuid // Uploadcare UUID (16 bytes) not full URL

cleanFileId String? @db.Uuid // after inpainting

renderedFileId String? @db.Uuid // after final compositing

status PageStatus @default(PENDING)

createdAt DateTime @default(now())

regions TextRegion[]

jobs TranslationJob[]

@@unique([chapterId, pageNumber])

@@index([chapterId])

@@index([status]) // for job queue: WHERE status = PENDING

}

model TextRegion {

id Int @id @default(autoincrement())

pageId Int

page Page @relation(fields: [pageId], references: [id], onDelete: Cascade)

bubbleIndex SmallInt // reading order (1, 2, 3...)

regionType RegionType @default(DIALOGUE)

// Bounding box as 4 SMALLINT columns (8 bytes total) vs JSON blob (~60 bytes)

bboxX SmallInt

bboxY SmallInt

bboxW SmallInt

bboxH SmallInt

originalText String @db.Text

translatedText String? @db.Text

// Speaker info

speakerName String? @db.VarChar(80)

speakerGender GenderCode @default(UNKNOWN)

thaiParticle String? @db.VarChar(10)

// Quality

confidence SmallInt @default(0) // 0–100 (SMALLINT = 2 bytes vs Float = 8 bytes)

isApproved Boolean @default(false)

isManual Boolean @default(false) // true = drawn by human, not AI

// Render overrides

fontSizeOverride SmallInt? // null = auto-fit

// Translation memory FK (prevents duplicate storage of repeated dialogue)

memoryId Int?

memory TranslationMemory? @relation(fields: [memoryId], references: [id])

createdAt DateTime @default(now())

updatedAt DateTime @updatedAt

@@index([pageId])

@@index([pageId, isApproved]) // for bulk-approve queries

}

model TranslationMemory {

id Int @id @default(autoincrement())

projectId String

project Project @relation(fields: [projectId], references: [id], onDelete: Cascade)

sourceText String @db.Text

sourceHash String @db.VarChar(64) // SHA-256 of sourceText for fast lookup

targetText String @db.Text

useCount Int @default(1) // how many regions reference this memory

createdAt DateTime @default(now())

regions TextRegion[]

@@unique([projectId, sourceHash]) // deduplicate on content hash, not full text

@@index([projectId])

}

model TranslationJob {

id Int @id @default(autoincrement())

pageId Int

page Page @relation(fields: [pageId], references: [id], onDelete: Cascade)

status JobStatus @default(QUEUED)

aiModel String @db.VarChar(60)

step SmallInt @default(0) // 0=pending, 1=vision done, 2=chars done, 3=translation done

errorMessage String? @db.VarChar(500)

startedAt DateTime?

completedAt DateTime?

createdAt DateTime @default(now())

@@index([status, createdAt]) // for job queue: ORDER BY createdAt ASC WHERE status=QUEUED

}

---

**D.4 Storage Savings Analysis**

To make the optimisations concrete, the table below compares the storage cost per manga chapter (20 pages, 30 regions per page = 600 TextRegion rows) between the naive schema and the optimised schema.

| **Field / Pattern** | **Naive → Optimised savings per 600 rows** |
| --- | --- |
| **Status enum fields (×4 per TextRegion row)** | VARCHAR avg 8 bytes → SMALLINT 2 bytes per enum field. 4 enums × 6 bytes saved × 600 rows = 14.4 KB saved per chapter. |
| **Bounding box storage** | JSON blob ~60 bytes → 4×SMALLINT = 8 bytes. 52 bytes saved × 600 rows = 31.2 KB per chapter. |
| **Confidence score** | Float 8 bytes → SMALLINT 2 bytes. 6 bytes saved × 600 rows = 3.6 KB per chapter. |
| **Image file ID (×3 per Page row)** | Full URL ~65 bytes → UUID 16 bytes. 49 bytes saved × 3 fields × 20 pages = 2.9 KB per chapter. |
| **Translation Memory deduplication** | Common lines repeated verbatim in every chapter. For a series with 20 chapters, recurring dialogue stored once instead of 20 times. Estimated 60 KB saved per 20-chapter series. |
| **Integer PKs for high-volume tables** | Using Int (4 bytes) or SmallInt (2 bytes) instead of cuid/String (25 bytes) for TextRegion, Page, Chapter, Job PKs saves significant index space. 21 bytes saved × 4 tables × 600 rows = ~50 KB per chapter. |
| **Total estimated saving** | ~160 KB per chapter vs naive schema. For a 50-chapter series this is ~8 MB saved — meaningful on a 1 GB Prisma free tier. |

**D.5 Critical Indexes**

Indexes are the other major lever for database performance. Each index below is justified by a specific query pattern that occurs in the application's hot paths. Adding unnecessary indexes wastes write performance and storage, so every index listed has a concrete reason.

| **Index** | **Justified by Query Pattern** |
| --- | --- |
| **User.kindeId (unique)** | auth hook runs on every page load and looks up the user by kindeId — this must be sub-millisecond. |
| **Page.status** | The Cron job polls for the oldest PENDING or QUEUED page. Without this index, it does a full table scan every 30 seconds. |
| **Page.chapterId** | Chapter dashboard loads all pages for a chapter. FK index ensures this is a fast index scan. |
| **TextRegion.pageId** | Editor loads all regions for a page on mount — this is the most frequent query in the app. |
| **TextRegion(pageId, isApproved)** | Bulk-approve endpoint: UPDATE regions WHERE pageId = ? AND isApproved = false AND confidence >= ?. Composite index covers both filter columns. |
| **TranslationJob(status, createdAt)** | Cron queue query: SELECT * FROM jobs WHERE status = QUEUED ORDER BY createdAt ASC LIMIT 1. Composite index makes this O(log n) instead of O(n). |
| **TranslationMemory(projectId, sourceHash)** | Before inserting a new translation, check for duplicates by hash, not full text comparison. Composite unique index doubles as the lookup index. |
| **ProjectCharacter(projectId, name)** | Unique constraint doubles as the index for glossary lookups by character name during translation. |

*Comic Trans Studio — Design & Technical Deep Dive | Addendum v1.1*