# Loupe: Addendum — Features, Screens and Technical Spec

*Document 2 of 2 · Paste together with the main PRD · v1.1 · Written for the Google AI Studio builder agent*

## 0. How the two documents fit together

Document 1 (the main PRD) defines scope, pages, the extraction pipeline, prompts, schemas, the design.md template, error messages, Firebase data and build stages. This document adds: the product name, the feature list, screen-by-screen specs, and the technical details a builder needs (folder layout, libraries, API handling, data tables, code contracts).

Where the two documents disagree, **this document wins**. Overrides:

1. **App name** is **Loupe**. Wherever Document 1 says `{{APP_NAME}}`, use Loupe.
2. **Generated file title line** is `# DESIGN.md: <title>` (not REBUILD.md). The download file name is `<slug>-design.md`.
3. **Shared images:** the `shares` document stores only the small cover thumbnail. All full images go in the `images` sub-collection, each at most 600 KB binary (about 800 KB as base64). Ignore the "700 KB inline" wording in Document 1, Section 4.
4. **State management** uses Zustand (Section 5.2 below).

Everything else in Document 1 stays exactly as written.

---

## 1. Name and brand

**Name: Loupe.** A loupe is the small magnifying lens designers, photographers and jewelers use to inspect fine detail. That is exactly what the product does: look closely at a screenshot and write down every detail. It is a real word, short, easy to say and spell, and says nothing about "AI".

**Tagline:** "See the design. Rebuild it exactly."

**Backup names** (if Loupe is taken when you check domains and trademarks): Kern, Tracer.

**Brand rules for the build**

- The name lives in one constant: `APP_NAME = 'Loupe'` in `config.ts`. Never type it elsewhere.
- Logo for the MVP is a simple wordmark: the text "Loupe" in bold Space Grotesk, preceded by a small SVG lens icon (a circle with a short diagonal handle) drawn in `--accent`. No other logo work.
- Page titles follow the pattern `Page name · Loupe`. Default title: `Loupe: Screenshot to design.md`.
- Favicon: the lens icon as an inline SVG data URL.

---

## 2. Main features (MVP)

| ID | Feature | Priority | Done when |
|---|---|---|---|
| F1 | Upload 1 to 8 screenshots (drop, click, paste) | P0 | Valid files appear as ordered, reorderable tiles |
| F2 | Quality gate with specific rejection messages | P0 | Each rejection code in Doc 1 Section 11 can be triggered |
| F3 | Multi-site detection and click-to-choose screen | P0 | Two different sites open the choice screen |
| F4 | Generate `design.md` (measure, analyze, extract, validate, render) | P0 | A real screenshot yields a complete file |
| F5 | Stable output (cache by image hash) | P0 | Same files twice give a byte-identical file |
| F6 | Result viewer (Preview and Raw tabs) | P0 | Tables and headings render; raw shows exact text |
| F7 | Copy, Download .md, Export PDF | P0 | Clipboard and file match raw text exactly |
| F8 | History in the browser | P0 | Results survive reload; delete works |
| F9 | Share link (public page with images) | P0 | Link opens on another device |
| F10 | Library (publish, browse, search, filter, sort) | P0 | Published item appears immediately for others |
| F11 | Landing page and Tutorial page | P1 | Match Doc 1 Sections 5.2 and 5.3 |
| F12 | Debug mode (`?debug=1`) | P1 | Panel shows raw JSON, timings, cache status |
| F13 | Site queue ("Next up") after multi-site runs | P1 | Banner appears and continues correctly |

P0 means the app is not testable without it. P1 is built after all P0 items pass.

---

## 3. User journeys (use these as manual test scripts)

**J1 First-time user.** Landing, "Try it now", drop one screenshot, tiles appear, "Generate", watch five real steps, land on Result, press Copy, see "Copied".

**J2 Mixed screenshots.** Drop a hero from site A and an about section from site B, press Generate, see the choice screen, pick site A, get Result A, see banner "Next up: site B", press it, get Result B.

**J3 Bad image.** Drop a photo of a dog. After the Analyze step the card shows "Please send a better image" with the reason. No result is created.

**J4 Sharing.** On Result, press Share. Link is copied. Open it in a private window: images and file are visible. Press "Create your own".

**J5 Library.** On Result, turn on "Publish to library". Open Library on another device: the item is at the top. Open it, press Copy, copy count increases once.

**J6 Repeat.** Generate from the same file again. It says "Loaded your saved result" and shows an identical file instantly.

---

## 4. Screen specifications

Conventions: every interactive element carries a `data-testid` (listed per screen) so tests can find it. Every screen has the three async states (loading, empty or success, error) described in Document 1.

### 4.0 Screen inventory

| # | Screen | Route | Needs Firebase |
|---|---|---|---|
| S1 | Landing | `/` | Only the teaser |
| S2 | Generate: empty | `/generate` | No |
| S3 | Generate: files selected | `/generate` | No |
| S4 | Generate: choose a site | `/generate` | No |
| S5 | Generate: processing | `/generate` | No |
| S6 | Generate: send a better image | `/generate` | No |
| S7 | Result | `/result/:id` | Only Share and Publish |
| S8 | History | `/history` | No |
| S9 | Library | `/library` | Yes |
| S10 | Share page | `/s/:shareId` | Yes |
| S11 | Tutorial | `/tutorial` | No |
| S12 | Not found | `*` | No |

Modals and overlays: image lightbox, delete confirmation, toast stack.

### 4.1 S1 Landing

Content and order are in Document 1, Section 5.2. Extra rules:

- Hero headline max width 14 ch per line on desktop, three lines at most.
- The CSS-only hero visual: left card shows a faint browser-window outline; right card shows lines of monospace text (use fixed, honest sample text such as `## Colors`, `--bg: #050505`); a thin arrow between them. No fake output claims.
- Library teaser calls `listPublic({ sort: 'new', pageSize: 6 })`. While loading show 6 skeleton cards.
- `data-testid`: `cta-try`, `cta-example`, `faq-item-1` to `faq-item-6`, `teaser-grid`.

### 4.2 S2 and S3 Generate: upload

```
┌────────────────────────────────────────────────────┐
│  Loupe   Generate  Library  Tutorial  History      │
├────────────────────────────────────────────────────┤
│                                                    │
│   ┌────────────────────────────────────────────┐   │
│   │  ⬆  Drop website screenshots here,         │   │
│   │     click to upload, or paste (Ctrl+V)     │   │
│   │  PNG, JPG or WEBP · up to 8 · 10 MB each   │   │
│   └────────────────────────────────────────────┘   │
│   Upload screenshots you are happy to share.       │
│                                                    │
│   [1 img] [2 img] [3 img]    Order matters: top    │
│                              of the page first.    │
│   [Generate design.md]   Clear all                 │
└────────────────────────────────────────────────────┘
```

- Tiles: 160 px wide thumbnails (object-fit contain on `--surface-2`), order badge top-left, remove button top-right, file name and size below, red badge plus reason if rejected.
- Drag-to-reorder: use native HTML5 drag events, no extra library. Also give each tile two small buttons "Move earlier" and "Move later" for touch and keyboard.
- Validation runs the moment files are added (Doc 1 Section 11 client rules). The blur check runs on a 512 px copy in a canvas.
- The page remembers nothing after reload (no draft persistence).
- `data-testid`: `dropzone`, `file-input`, `tile-<n>`, `tile-remove-<n>`, `btn-generate`, `btn-clear`.

### 4.3 S4 Choose a site

```
┌────────────────────────────────────────────────────┐
│  We found screenshots from more than one website.  │
│  Which one should we extract first?                │
│                                                    │
│  ( ) Aeline: AI consulting landing page            │
│      [thumb][thumb]   Hero · About                 │
│                                                    │
│  (•) Northfield: Fintech pricing page              │
│      [thumb]          Pricing · FAQ                │
│                                                    │
│  [Extract this one]   Extract all, one after       │
│                       another                      │
└────────────────────────────────────────────────────┘
```

- Whole card is clickable; the radio reflects selection. Default selection: first site.
- "Extract this one": runs Step 6 for the selected site and saves the others to the `siteQueue` in the `meta` store.
- "Extract all, one after another": runs sites in order of their first image; each produces its own result; after the last one, open the first result and show a banner listing the others.
- `data-testid`: `site-card-<siteId>`, `btn-extract-one`, `btn-extract-all`.

### 4.4 S5 Processing

- Five steps (labels from Doc 1 Section 5.6). Each step has a status icon: pending (gray dot), active (spinner), done (check). Status changes only on real `PipelineEvent`s (Section 5.4 below).
- Under the list show the elapsed time as `mm:ss`. If a step takes longer than 20 s, add the line "Still working. The free testing quota can be slow."
- If a retry wait happens, show "The service is busy. Retrying in N s." with a live countdown.
- Cancel button calls `AbortController.abort()`. After cancel, go back to S3 with files still loaded.
- `data-testid`: `step-check`, `step-measure`, `step-analyze`, `step-extract`, `step-validate`, `btn-cancel`.

### 4.5 S6 Send a better image

Layout and texts are in Document 1, Section 11. For each rejected file show a row: small thumbnail, file name, reason text. Primary button "Choose another image" clears the rejected files and returns to S2 or S3.

### 4.6 S7 Result

```
┌────────────────────────────────────────────────────┐
│  Loupe   Generate  Library  Tutorial  History      │
├────────────────────────────────────────────────────┤
│  Aeline: AI consulting landing                     │
│  [dark] [ai-tech] [hero] [pricing]                 │
│  [Copy] [Download .md] [Export PDF] [Share]        │
│  Publish to library  (o──)                         │
├───────────────┬────────────────────────────────────┤
│ [ image 1 ]   │  [ Preview | Raw .md ]             │
│ [ image 2 ]   │  ## Style Summary                  │
│ 3 Nov 2026    │  ...                               │
│               │  | Token | Value | Usage |         │
└───────────────┴────────────────────────────────────┘
 Paste the whole file into your AI builder as the first message.
```

- Title, tags and images come from the saved result.
- Preview tab renders markdown with `react-markdown` plus `remark-gfm`. Raw HTML is never enabled.
- Tables sit inside a wrapper with `overflow-x: auto`. Code blocks scroll horizontally. Table header row is sticky inside the wrapper only on screens wider than 768 px.
- Raw tab shows monospace text with line numbers (render as a two-column grid; line numbers are not selectable, so copying text does not include them).
- Banner "Next up: <site label>. Extract it now" appears if `siteQueue` has items for this run.
- Share and Publish show their disabled state with the Firebase message when remote storage is unavailable.
- `data-testid`: `btn-copy`, `btn-download`, `btn-pdf`, `btn-share`, `toggle-publish`, `tab-preview`, `tab-raw`, `result-title`, `banner-next`.

### 4.7 S8 History

Layout in Document 1, Section 5.8. Cards are `<a>` links to `/result/:id`. Delete opens a confirm dialog with the buttons "Delete" (danger) and "Keep". `data-testid`: `history-card-<id>`, `history-delete-<id>`, `confirm-delete`.

### 4.8 S9 Library

```
┌────────────────────────────────────────────────────┐
│  Library                                           │
│  [ Search prompts... ]       Sort: [Newest ▾]      │
│  Industry: [ai-tech][saas][agency]...              │
│  Style:    [dark][light][minimal]...               │
│  Sections: [hero][pricing][faq]...                 │
│                                                    │
│  ┌──────┐ ┌──────┐ ┌──────┐                        │
│  │ img  │ │ img  │ │ img  │                        │
│  │title │ │title │ │title │                        │
│  │tags ⧉12│ │tags ⧉3│ │tags ⧉0│                      │
│  └──────┘ └──────┘ └──────┘                        │
│              [ Load more ]                         │
└────────────────────────────────────────────────────┘
```

- Cards show cover thumbnail, title, up to 3 tags, copy count (a small copy icon plus number), date.
- Filter chips are multi-select within a group (match any), and across groups they combine with AND. A "Clear filters" link appears when any chip is active.
- Search is case-insensitive over title and all tags.
- Filters and search apply to items already loaded (48 first, then 24 per click). Show the line "Showing N of the loaded prompts" and a note when "Load more" is available: "Load more to search further."
- `data-testid`: `library-search`, `library-sort`, `chip-<group>-<value>`, `library-card-<shareId>`, `btn-load-more`.

### 4.9 S10 Share page

Layout: image strip on the left (or on top on mobile), file on the right, same Copy / Download / PDF actions, plus "Create your own". Title and tags at the top. Show the small line "Shared with Loupe". If the item has more than one image show a simple lightbox with next and previous.
`data-testid`: `share-title`, `share-image-<n>`, `btn-create-own`.

### 4.10 S11 Tutorial and S12 Not found

Content in Document 1, Section 5.3. Not found: a heading "This page is not available." and a button "Go home".

---

## 5. Technical architecture

### 5.1 Folder structure

```
src/
  main.tsx
  App.tsx                 router and layout
  config.ts               every constant, the two prompts, vocabularies
  types.ts                shared TypeScript types
  services/
    ai.ts                 Gemini calls (only file that talks to the SDK)
    pipeline.ts           orchestrator (runs Steps 1 to 9)
    hash.ts               SHA-256 helpers
    prepare.ts            decode, slice, resize images
    measure.ts            palette and region measurement
    validate.ts           zod schemas and normalization
    render.ts             spec to design.md
    compress.ts           compress images for storage and sharing
    errors.ts             AppError and error mapping
    usage.ts              canGenerate() stub
    identity.ts           getUser() stub
  storage/
    local.ts              IndexedDB adapter
    remote.ts             Firestore adapter
    firebase.ts           Firebase init
  store/
    runStore.ts           current generation run (Zustand)
    libraryStore.ts       library list, filters, paging (Zustand)
  pages/                  one file per screen
  components/             Header, Footer, Toast, Dialog, Tile, SiteCard,
                          StepList, MarkdownView, RawView, TagChip, LibraryCard
  tests/
    fixtures/             sample spec JSON, expected design.md, test images
```

### 5.2 Libraries (use only these)

| Purpose | Library |
|---|---|
| UI | react, react-dom, react-router-dom |
| Build and style | vite, typescript, tailwindcss |
| Gemini | @google/genai |
| Validation | zod |
| Local storage | idb-keyval |
| Remote storage | firebase (Firestore only) |
| Ids | nanoid |
| Markdown preview | react-markdown, remark-gfm |
| State | zustand |
| Icons | lucide-react |

Do not add any other library without asking first. No UI kits, no animation libraries, no chart libraries.

### 5.3 Constants in `config.ts`

| Constant | Value |
|---|---|
| `APP_NAME` | `'Loupe'` |
| `PIPELINE_VERSION` | `'1.0.0'` (bump whenever prompts, schema, validation or renderer change) |
| `GEMINI_MODEL` | one pinned id from the AI Studio model picker |
| `GEN_CONFIG` | `{ temperature: 0, topK: 1, seed: 42, maxOutputTokens: 16000 }` |
| `MAX_FILES` / `MAX_FILE_MB` | `8` / `10` |
| `MIN_WIDTH` / `MIN_HEIGHT` / `MAX_DIM` | `800` / `300` / `12000` |
| `BLUR_THRESHOLD` | `40` |
| `MAX_SEGMENTS` | `12` |
| `SEGMENT_RATIO` / `TALL_RATIO` | `1.25` / `2.2` |
| `AI_MAX_SIDE` / `AI_JPEG_QUALITY` | `2048` / `0.9` |
| `RETRY_DELAYS_MS` | `[5000, 15000, 30000]` |
| `CALL_TIMEOUT_MS` | `90000` |
| `SCHEMA_RETRIES` | `2` |
| `COLOR_SNAP_DISTANCE` | `40` |
| `COLOR_MERGE_DISTANCE` | `24` |
| `HISTORY_MAX` | `50` |
| `SHARE_IMAGE_MAX_BYTES` | `600000` |
| `COVER_WIDTH` / `COVER_MAX_BYTES` | `480` / `50000` |
| `LIBRARY_FIRST_PAGE` / `LIBRARY_PAGE` | `48` / `24` |
| `AUTH_ENABLED` / `PAYMENTS_ENABLED` | `false` / `false` |

### 5.4 Pipeline orchestrator

`services/pipeline.ts` exports one function. The UI only talks to this function and to the run store.

```ts
export type StepName = 'check' | 'measure' | 'analyze' | 'extract' | 'validate';

export type PipelineEvent =
  | { type: 'step'; step: StepName; status: 'active' | 'done' }
  | { type: 'retry'; inSeconds: number; attempt: number }
  | { type: 'cache_hit'; resultId: string }
  | { type: 'rejected'; items: { fileName: string; code: string }[] }
  | { type: 'done'; resultIds: string[] }
  | { type: 'error'; error: AppError };

export interface PipelineOptions {
  signal: AbortSignal;
  onEvent: (e: PipelineEvent) => void;
  // Called when more than one site is found. Resolves with the user's choice.
  requestChoice: (sites: SiteGroup[]) => Promise<{ siteIds: string[] }>;
}

export async function runPipeline(files: File[], opts: PipelineOptions): Promise<void>;
```

Rules:

- Events are emitted only when the work actually starts or finishes. No timers that fake progress.
- The `step` events map to the five visible steps: `check` (hash, validate, prepare), `measure`, `analyze`, `extract`, `validate` (validate, render, save).
- Pipeline code never touches React. The run store subscribes through `onEvent`.
- Only one run at a time. The Generate button is disabled while a run is active.

### 5.5 Gemini API handling (`services/ai.ts`)

This is the only file that imports `@google/genai`.

**Client.** Create it once. Read the key from the same environment variable the AI Studio project template already uses (do not invent a new name, and never print it).

```ts
import { GoogleGenAI } from '@google/genai';
const ai = new GoogleGenAI({ apiKey: /* the key the AI Studio template provides */ });
```

**Request shape.** Images go in as inline base64 parts, each preceded by a text label so the model can refer to them.

```ts
const parts = [
  { text: 'img1' }, { inlineData: { mimeType: 'image/jpeg', data: base64One } },
  { text: 'img2' }, { inlineData: { mimeType: 'image/jpeg', data: base64Two } },
  { text: extraTextSuchAsMeasurementsJson },
];

const res = await ai.models.generateContent({
  model: GEMINI_MODEL,
  contents: [{ role: 'user', parts }],
  config: {
    systemInstruction: PROMPT_TEXT,
    ...GEN_CONFIG,
    responseMimeType: 'application/json',
    responseSchema,          // see note below
    abortSignal: signal,     // if the SDK version supports it
  },
});
const raw = res.text;        // JSON string
```

**Schema note.** Pass `responseSchema` if converting the zod schema to Gemini's schema format is straightforward. If it causes errors, drop `responseSchema`, keep `responseMimeType`, and rely on the zod check below. The zod validation is the real safety net, never skip it.

**Size guard.** Add up the base64 lengths of all parts. If the total is above 18 MB, reduce `AI_MAX_SIDE` in steps of 256 px and re-encode, then try again. This keeps the request under the inline limit.

**Parsing.** `JSON.parse(raw)`. If parsing throws, treat it as a schema failure. If the response was cut off (`finishReason` equal to `MAX_TOKENS`), retry once with the extra line "Keep every description under 25 words."

**Blocked responses.** If the response has a block reason or a `SAFETY` finish reason, throw `AppError('adult_or_unsafe')`.

**Retry wrapper.**

```ts
async function withRetry<T>(fn: () => Promise<T>, signal: AbortSignal, onRetry: (s: number, a: number) => void): Promise<T> {
  for (let attempt = 0; ; attempt++) {
    try {
      return await withTimeout(fn(), CALL_TIMEOUT_MS, signal);
    } catch (err) {
      if (signal.aborted) throw new AppError('cancelled');
      const status = (err as any)?.status;
      const retryable = [429, 500, 503, 504].includes(status) || err instanceof TypeError;
      if (!retryable || attempt >= RETRY_DELAYS_MS.length) throw mapError(err);
      onRetry(RETRY_DELAYS_MS[attempt] / 1000, attempt + 1);
      await sleep(RETRY_DELAYS_MS[attempt], signal);
    }
  }
}
```

**Error mapping (`mapError`).**

| Situation | `AppError` code | User message |
|---|---|---|
| HTTP 429 or 503 after all retries | `quota` | Doc 1 Section 11 |
| Network failure, timeout | `network` | Doc 1 Section 11 |
| HTTP 401 or 403 | `ai_config` | The AI service is not set up correctly. Check the API key. |
| HTTP 400 (bad request) | `unreadable` | We could not read enough detail from this image. Please send a clearer screenshot. |
| Safety block | `adult_or_unsafe` | We cannot process this image. |
| User pressed Cancel | `cancelled` | (no message, return to S3) |
| Anything else | `network` | Doc 1 Section 11 (and log the technical error) |

Add `ai_config` to the messages table.

**Call budget per run:** Analyze 1 call, Extract 1 call per chosen site, up to 2 schema retries, retries for HTTP errors on top. Calls are always sequential, never parallel.

### 5.6 State management

- `runStore` (Zustand) holds: `files`, `fileIssues`, `stepStatus`, `retryInfo`, `sites`, `phase` (`idle`, `checking`, `choosing`, `processing`, `rejected`, `done`, `error`), `error`, `resultIds`, and the `AbortController`.
- `libraryStore` holds: `items`, `cursor`, `sort`, `filters`, `search`, `status` (`idle`, `loading`, `error`), `hasMore`.
- Anything that must survive a reload lives in IndexedDB or Firestore, never only in a store.

### 5.7 Data tables

**IndexedDB (via idb-keyval, one database named `loupe`)**

| Store | Key | Value fields |
|---|---|---|
| `results` | `id` (nanoid) | `id`, `createdAt` (ms), `cacheKey`, `title`, `tags`, `markdown`, `spec`, `images[]` (`name`, `width`, `height`, `thumbDataUrl`, `dataUrl`), `shareId?` |
| `cache` | `cacheKey` | `resultId` |
| `meta` | fixed keys | `deviceId` (string), `siteQueue` (array of pending site groups with their image refs), `copiedShares` (array of share ids) |

History order is by `createdAt` descending. When `results` exceeds `HISTORY_MAX`, delete the oldest and its cache entries.

**Firestore (`shares/{shareId}`)**

| Field | Type | Notes |
|---|---|---|
| `title` | string | from `spec.meta.title` |
| `markdown` | string | at most 300 KB |
| `tags` | map of three string arrays | `industry`, `style`, `sections` |
| `imageCount` | number | |
| `coverImage` | string | data URL, 480 px wide, at most 50 KB |
| `deviceId` | string | anonymous device id |
| `createdAt` | timestamp | server timestamp |
| `copyCount` | number | starts at 0 |
| `visibility` | string | `unlisted` or `public` |
| `publicAt` | timestamp | exists only while public |
| `publicCopyCount` | number | exists only while public |

**Firestore (`shares/{shareId}/images/{n}`)**

| Field | Type |
|---|---|
| `index` | number |
| `mime` | string |
| `width`, `height` | number |
| `dataUrl` | string (at most about 800 KB) |

**TypeScript types to define in `types.ts`:** `Result`, `ShareDoc`, `ShareImageDoc`, `Measurements`, `AnalyzeResult`, `SiteGroup`, `DesignSpec` (as in Document 1, Section 8), `AppError`, `PipelineEvent`.

### 5.8 Handling markdown tables (renderer and viewer)

The renderer writes many tables. It must never produce a broken table.

- `esc(text)`: replace `|` with `\|`, replace line breaks with a space, and trim.
- Every table has a header row and a separator row `|---|---|---|`.
- Column order is fixed per table (Document 1, Section 9).
- Colors print as uppercase hex in backticks, sizes as integers followed by `px`.
- Cells with no value print a dash only inside tables where a column is optional (for example `weight`). Never write "N/A".
- Viewer: wrap each rendered `<table>` in a div with horizontal scroll, `min-width` on cells of 96 px, and `word-break: break-word` for long CSS strings.
- The Raw tab must show the exact string that Copy and Download use. They all read the same `markdown` field. Never regenerate it in the viewer.

### 5.9 Formatting helper rules for the renderer

| Helper | Rule |
|---|---|
| `hex(v)` | uppercase `#RRGGBB`, wrapped in backticks |
| `px(n)` | `Math.round(n)` plus `px` |
| `em(n)` | up to 2 decimals plus `em` |
| `slug(title)` | lowercase, spaces to `-`, only `a-z`, `0-9`, `-`, max 40 chars |
| `title` | trimmed and limited to 60 characters |
| line endings | `\n` only, file ends with a single newline |

### 5.10 Image compression (`services/compress.ts`)

```ts
compressImage(blob: Blob, opts: { maxSide: number; maxBytes: number }):
  Promise<{ dataUrl: string; width: number; height: number; bytes: number }>
```

Draw to a canvas at the longest side `maxSide`, encode WebP. Try quality 0.8, 0.7, 0.6, 0.5. If still above `maxBytes`, reduce `maxSide` to 1400 and repeat. If WebP is not supported by the browser, use JPEG. Calculate bytes from the base64 length (`length * 3 / 4`). Throw `AppError('storage')` if nothing fits.

### 5.11 Share and publish flows

**Share (button on Result):**

1. If the result already has a `shareId`, copy the existing link and stop.
2. Compress the cover thumbnail and each image (`SHARE_IMAGE_MAX_BYTES`).
3. `createShare` writes the document and image documents in one batch, `visibility: 'unlisted'`.
4. Save `shareId` on the local result.
5. Copy `${location.origin}${location.pathname}#/s/${shareId}` and show a toast with the link.

**Publish toggle:**

- On: create the share first if needed, then `setPublic(shareId, true)` (sets `publicAt`, `publicCopyCount = copyCount`, `visibility = 'public'`).
- Off: `setPublic(shareId, false)` (deletes the two fields, sets `visibility = 'unlisted'`).
- The toggle shows a spinner while writing and rolls back with an error toast if the write fails.

**Copy counting:** on Copy from S10 (or a library item), if the share id is not in `copiedShares`, call `incrementCopy`, then add the id to `copiedShares`.

### 5.12 PDF export

The Result and Share pages have print CSS. `window.print()` is called after switching to the Preview tab. Print rules: hide header, footer, action bar, tabs and image column; white background, black text; `@page { margin: 16mm }`; avoid page breaks inside table rows and code blocks; wrap long lines.

### 5.13 Security and safety

- Markdown is rendered without raw HTML.
- Image sources must start with `data:image/webp`, `data:image/jpeg` or `data:image/png`; anything else is not rendered.
- Titles and tags are shown as text only, never as HTML.
- Never log the API key. In debug mode, log responses but not keys.

### 5.14 Accessibility basics

Buttons are real `<button>` elements, tiles and cards are links where they navigate, dialogs trap focus and close on Esc, images have alt text (`Uploaded screenshot 1`), color contrast of text on dark surfaces is at least 4.5:1, focus rings are visible.

---

## 6. Test fixtures the owner should prepare

Create a folder of real test images before Stage 2:

1. A clean 1440 px landing page screenshot.
2. The same screenshot heavily blurred.
3. A 400 px wide thumbnail of a website.
4. A photo of an animal.
5. Screenshots from two different websites.
6. A very tall full-page screenshot (4000 px or more).
7. A website shown inside a framed mockup photo (like a template marketplace preview).
8. A solid red image and a three-color image (red, green, blue stripes).

In `src/tests/fixtures/` keep one fixed `sample-spec.json` and the expected `sample-design.md` produced from it. These two files are test data only and are never shown to users.

---

## 7. First message to send to AI Studio

Paste this as your first message together with both documents:

> You are building the product described in the two attached documents. Read both fully before writing code. Then reply with (1) a list of the 13 stages from Document 1, Section 14 in your own words, (2) any question that blocks you, and (3) which parts you think cannot work in AI Studio and what you will do instead. Do not write code yet. After I confirm, build **Stage 0 only**, run its tests, and report using the format in Document 1. Never move to the next stage until I say "go". Do not add any feature, library, page or setting that is not in the documents. Do not fake anything: no hardcoded results, no timer-based progress, no buttons without real behavior. If something fails, tell me plainly.

---

## 8. Common failure modes to avoid (checklist for the builder)

- Showing a made-up `design.md` when the AI call fails. Show the error state instead.
- Fake progress using `setTimeout`. Steps change only on real events.
- Using the "latest" model alias. Use one pinned id.
- Skipping zod validation because the model "usually returns correct JSON".
- Letting the model write the final file. The renderer writes it.
- Putting dates, ids or model names inside `design.md`.
- Storing images in `localStorage`, or using Firebase Storage buckets.
- Rendering markdown with raw HTML enabled.
- Buttons that look finished but have no handler.
- Adding login, pricing, plans, trial counters or payment text.
- Swallowing errors in empty `catch` blocks.
- Declaring a stage "done" without running its tests.
- Changing the output structure or prompts without bumping `PIPELINE_VERSION`.
