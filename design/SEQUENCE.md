# Translation Review MVP: Sequence with Wireframes

Companion to [PRD.md](../PRD.md). Shows how the reviewer, the web app, and the backend services interact at each step, with the screen the reviewer sees at that point.

The tool works with any Arabic text. The wireframes use sample content only: the Gospel of John, chapter 1, translated with the Christian / Bible profile. The Arabic is from the Van Dyck Arabic Bible (public domain), and the reviewed English matches the NKJV (PRD TR-2b). With the General profile or another profile, the flow and screens are the same.

**Files in this folder**

| File | What it is |
| --- | --- |
| [sequence-diagram.excalidraw](sequence-diagram.excalidraw) | Full sequence diagram with every wireframe screenshot embedded beside its step |
| [sequence-diagram.png](sequence-diagram.png) | PNG export of the diagram above |
| [wireframes/](wireframes/) | One editable `.excalidraw` file per screen (01 to 08) |
| [screenshots/](screenshots/) | PNG of each wireframe |

Open `.excalidraw` files at [excalidraw.com](https://excalidraw.com) (File > Open) or with the Excalidraw VS Code extension. Opening a file locally does not upload it anywhere.

**Participants**

| Participant | Role |
| --- | --- |
| Reviewer | The single MVP user (uploads and reviews) |
| Web app | Browser front end: navigator, side-by-side view, live diff |
| Backend API | Validates uploads, coordinates extraction, translation, persistence, export |
| PDF extractor | Library that counts pages and extracts Arabic text |
| Translation engine (LLM) | Translates one paragraph at a time using the general rules plus the active translation profile (TR-2) |
| Database | Stores the document and paragraphs (see PRD data model) |

---

## Full sequence

![Full sequence diagram with wireframes](sequence-diagram.png)

The same flow as text (renders as a diagram on GitHub, or in VS Code with a Mermaid preview extension):

```mermaid
sequenceDiagram
    actor R as Reviewer
    participant W as Web app
    participant B as Backend API
    participant X as PDF extractor
    participant T as Translation engine (LLM)
    participant D as Database

    Note over R,D: 1 · Open the app (Screen 1)
    R->>W: Open app
    W->>B: GET /api/document
    B->>D: Load current document
    D-->>B: None
    B-->>W: 404: no document
    W-->>R: Show upload screen

    Note over R,D: 2 · Upload and validate (Screen 2 if rejected)
    R->>W: Drop PDF
    W->>W: Check it is a .pdf
    W->>B: POST /api/documents (file)
    B->>X: Count pages, extract text
    alt scanned, over 30 pages, or unreadable
        X-->>B: No text found / 45 pages
        B-->>W: 422 + reason
        W-->>R: Show error banner
    end

    Note over R,D: 3 · Extract and translate (Screen 3)
    X-->>B: Arabic text
    B->>B: Split into paragraphs
    B->>D: Save document + paragraphs
    B-->>W: 202: status = translating
    loop each paragraph
        B->>T: Translate (active profile prompt)
        T-->>B: English text
        B->>D: Save machine_translation
    end
    loop every few seconds
        W->>B: GET /api/document/status
        B-->>W: 18 of 42 translated
    end
    W-->>R: Show progress

    Note over R,D: 4 · Review a paragraph (Screen 4)
    B-->>W: status = in_review
    W->>B: GET /api/paragraphs
    B->>D: Load paragraphs
    B-->>W: 42 paragraphs
    R->>W: Click paragraph 4
    W-->>R: Show Arabic + English side by side

    Note over R,D: 5 · Edit and see the diff (Screen 5)
    R->>W: Edit English text
    W->>W: Compute word diff (live)
    W-->>R: Show diff in red / green
    W->>B: PATCH /api/paragraphs/4 (autosave)
    B->>D: Save current_translation
    R->>W: Tick checkbox
    W->>B: PATCH is_checked = true
    B->>D: Save is_checked

    Note over R,D: 6 · Retry a failed paragraph (Screen 6)
    R->>W: Click paragraph 7 (failed)
    W-->>R: Show Retry button
    R->>W: Click Retry
    W->>B: POST /api/paragraphs/7/retry
    B->>T: Translate paragraph 7
    T-->>B: English text
    B->>D: Save machine_translation
    B-->>W: Paragraph 7 translated

    Note over R,D: 7 · Check every paragraph (Screen 7)
    R->>W: Tick remaining paragraphs
    W->>B: PATCH is_checked (each)
    B->>D: Save is_checked
    W->>W: 42 / 42 checked: enable Done
    W-->>R: Done button enabled

    Note over R,D: 8 · Done and export (Screen 8)
    R->>W: Click Done & export
    W->>B: POST /api/document/export
    B->>D: Load current_translation in order
    B->>B: Build English-only .docx
    B-->>W: gospel-of-john-sample-en.docx
    W-->>R: Download file + confirmation
```

---

## Step by step

### 1 · Open the app
With no document in the database, the reviewer lands on the upload screen. The accepted input is spelled out under the drop zone (UP-1).

![Screen 1 · Upload](screenshots/01-upload.png)

### 2 · Upload rejected
The browser checks the file type first. The backend then checks page count and text layer. A rejected file gets a specific reason (UP-2).

![Screen 2 · Upload rejected](screenshots/02-upload-rejected.png)

### 3 · Extract and translate
After a valid upload, the backend splits the text into paragraphs and translates them one by one. The web app polls for progress. A paragraph that fails to translate is counted but doesn't stop the rest (TR-4, TR-5).

![Screen 3 · Translating](screenshots/03-translating.png)

### 4 · Review a paragraph
The navigator lists every paragraph with a checkbox, the first words of the Arabic, and an `edited` or `failed` tag. Clicking a row shows that paragraph side by side (RV-1, RV-2).

![Screen 4 · Review a paragraph](screenshots/04-review.png)

### 5 · Edit and see the diff
Typing in the English box updates a word-level diff against the original machine translation: removed words in red with strikethrough, added words in green. Changes autosave, and the navigator marks the paragraph as `edited` (RV-3, RV-4, DF-1 to DF-5).

![Screen 5 · Edit and diff](screenshots/05-edit-diff.png)

### 6 · Retry a failed paragraph
A paragraph whose translation failed shows a Retry button in place of the English text. Retrying translates only that paragraph (TR-5).

![Screen 6 · Retry a failed paragraph](screenshots/06-retry-failed.png)

### 7 · All checked, Done enabled
Done & export stays disabled until all 42 paragraphs are checked. Unticking any paragraph disables it again (EXP-1, CK-2).

![Screen 7 · All checked, Done enabled](screenshots/07-all-checked.png)

### 8 · Exported
Clicking Done & export builds an English-only file in paragraph order and downloads it. Re-exporting is allowed (EXP-2 to EXP-4). The `.docx` format follows the PRD's current assumption and is still an open question.

![Screen 8 · Exported](screenshots/08-exported.png)
