# PRD: Arabic→English Translation Review Tool (MVP)

**Date:** 2026-10-16 · **Author:** Helina Azer

---

## Overview

The MVP is a single-tenant web app. A reviewer uploads one text-based Arabic PDF (up to 30 pages), gets an automatic paragraph-by-paragraph English translation, corrects it side by side with the source, checks off every paragraph, and downloads a publishable English-only file.

**Any Arabic text.** The tool is not tied to one kind of content. It must work for any Arabic text: religious, literary, historical, legal, or general. Domain-specific rules, such as how to render scripture, live in swappable translation profiles (TR-2), not in the product's code. The first planned use is Christian texts, which is why the wireframes and examples use the Gospel of John. That is sample content only.

**Problem.** Translating Arabic texts into English is slow and hard to verify. Machine translation is a useful first draft, but every paragraph still needs a careful human check against the original, and many texts follow domain conventions a generic translation misses. Reviewers also need to see exactly what they changed.

**Long-term vision.** A full translation and publishing pipeline: upload, auto-translate, first review, second-phase review, then publishing, with admins and project managers coordinating the work. The MVP builds only the core loop (upload, translate, review, export) so the most important workflow can be validated first.

---

## Goals and scope

**Goals**

1. A reviewer can turn a 30-page Arabic PDF into a reviewed English translation without leaving the app.
2. Every paragraph is checked against its source before export.
3. Every change from the machine translation is visible at a glance.
4. No work is lost between sessions.
5. Any Arabic text can be processed; supporting a new kind of text means adding a translation profile, not changing code.

**In scope**

- Upload of one text-based Arabic PDF, up to 30 pages
- Automatic paragraph splitting and machine translation
- Paragraph navigator with checkboxes
- Side-by-side Arabic / English view with in-place editing
- Color-coded diff of the reviewer's edits
- Autosave and persistence
- Done button, enabled only when every paragraph is checked
- Export of the English-only translation

**Out of scope**

| Item | Why it waits |
| --- | --- |
| Login, user accounts, roles | Single-tenant MVP; anyone with the app can use it |
| Multi-tenancy | Single-tenant MVP |
| Scanned (image) PDFs and OCR | Arabic OCR with diacritics is unreliable |
| Other file types (.docx, .txt) | PDF only to start |
| Documents over 30 pages | Keeps processing and review scope manageable |
| Multiple documents at once | One document at a time |
| Manual merge or split of paragraphs | The automatic split is accepted as-is |
| Translation speed and performance tuning | Correctness comes first |
| 2nd-phase review, publishing workflow, project management | Later phases |

---

## Personas

The MVP has one effective user. With no login, the person who uploads is the person who reviews.

| Persona | Full-product role | MVP |
| --- | --- | --- |
| Reviewer (P0) | Compares the translation with the Arabic, edits it, and checks off paragraphs | Yes: the primary user |
| Admin (P1) | Manages users, uploads source material | Upload only, done by the reviewer |
| 2nd Phase Reviewer | Second quality check on the reviewer's work | Later |
| Publisher | Publishes verified translations | Later |
| Project Manager | Assigns work, tracks progress | Later |

---

## User flow

The reviewer goes from upload to download in six steps. Step 4 repeats for every paragraph, in any order.

1. **Upload.** The reviewer selects a PDF. The system checks that it is a PDF, 30 pages or fewer, and has extractable text.
2. **Extract and split.** The system pulls out the Arabic text and splits it into paragraphs.
3. **Translate.** The system machine-translates each paragraph into English and shows progress until all are ready.
4. **Review each paragraph.** The reviewer picks a paragraph in the navigator, compares Arabic with English, edits the English in place, checks the diff, and ticks the checkbox. Ticks can be undone at any time.
5. **Done.** When every paragraph is ticked, the Done button becomes active and the reviewer clicks it.
6. **Export.** The reviewer downloads the English-only translation as a publishable file.

---

## Functional requirements

IDs are referenced by the acceptance criteria below.

**Upload (UP)**

- UP-1: Accept a single `.pdf` file by file picker or drag-and-drop.
- UP-2: Reject files that are not PDFs, have more than 30 pages, or contain no extractable text, with a specific message for each.
- UP-3: Only one document exists at a time. Uploading a new PDF replaces the current one after the reviewer confirms that its work will be discarded.

**Extraction and splitting (EX)**

- EX-1: Extract Arabic text from the PDF in correct reading order (right to left).
- EX-2: Split the text into paragraphs automatically and number them in document order.
- EX-3: Drop running headers, footers, and page numbers where they can be detected.
- EX-4: Preserve Arabic diacritics (tashkeel) exactly as extracted. Many texts, such as scripture, classical works, and poetry, use them heavily.

**Translation (TR)**

- TR-1: Translate each paragraph independently from Arabic to English and keep it linked to its source paragraph.
- TR-2: The translation prompt has two layers:
    - **General rules** that apply to any Arabic text: translate the meaning faithfully, write natural English, keep names and terms consistent across the document, and never drop content.
    - **A translation profile** with optional domain conventions: instructions plus a glossary of fixed term renderings.
- TR-2a: Profiles are configuration (a file of instructions and glossary entries), not code. Adding or changing a profile needs no code change. In the MVP, the active profile is set in configuration, and the document records which profile was used.
- TR-2b: The MVP ships two profiles:
    - **General** (default): no domain conventions; for any Arabic text.
    - **Christian / Bible:** الله is rendered as "God", not "Allah"; biblical names use their established English forms (يوحنا as "John", not "Yohanna"); scripture references use standard English citation (e.g. John 1:6); and scripture quotations follow the **New King James Version (NKJV)**, including capitalized pronouns for God ("He", "His") and NKJV phrasing (e.g. "Most assuredly, I say to you").
- TR-3: Store the original machine translation permanently. It is never overwritten.
- TR-4: Show translation progress (e.g. 18 of 42 paragraphs translated).
- TR-5: A paragraph whose translation fails is flagged and can be retried on its own.

**Review (RV)**

- RV-1: The left panel lists every paragraph in order with its number, the first words of the Arabic, and a checkbox.
- RV-2: Clicking a paragraph shows it in the main panel: Arabic source on one side (read-only, RTL), English translation on the other (editable, LTR).
- RV-3: The reviewer edits the English directly in place.
- RV-4: Edits autosave without a save button.
- RV-5: The navigator shows progress (e.g. 12 / 42 checked).
- RV-6: The reviewer can move to the previous and next paragraph without returning to the navigator.

**Diff (DF)**

- DF-1: Once a paragraph's English differs from its original machine translation, a diff section appears under the side-by-side view.
- DF-2: The diff compares the original machine translation with the current text at word level. Removed words are red with strikethrough; added words are green.
- DF-3: The diff updates live as the reviewer types.
- DF-4: If the reviewer's edits return the text to the original, the diff section hides again.
- DF-5: The navigator marks paragraphs that have been edited, so changed paragraphs can be found quickly.

**Checkboxes (CK)**

- CK-1: Each paragraph has a checkbox meaning "reviewed."
- CK-2: A checkbox can be ticked and unticked any number of times.
- CK-3: A checked paragraph stays editable.

**Done and export (EXP)**

- EXP-1: The Done button is disabled until every paragraph is checked, with a tooltip saying how many remain.
- EXP-2: Clicking Done generates the English-only translation, paragraphs in original order, and downloads it.
- EXP-3: The export contains only the final English text: no Arabic, no diff markup, no paragraph numbers.
- EXP-4: Unticking a paragraph after Done disables Done again until it is re-checked. Re-exporting is allowed.

---

## Review screen

The review screen has two panels. The navigator on the left picks the paragraph; the main panel on the right shows it.

```
┌──────────────────────┬──────────────────────────────────────────────────┐
│ PARAGRAPHS           │  Paragraph 4 of 42                [Prev] [Next]  │
│                      │ ┌──────────────────────┬───────────────────────┐ │
│ ☑ Paragraph 1        │ │ Original Arabic      │ English translation   │ │
│ ☑ Paragraph 2 edited │ │ Read-only, RTL       │ Editable, autosaves   │ │
│ ☑ Paragraph 3        │ │          ...النص     │ The English text...|  │ │
│▶☐ Paragraph 4        │ └──────────────────────┴───────────────────────┘ │
│ ☐ Paragraph 5        │                                                  │
│ ☐ Paragraph 6        │  YOUR CHANGES (shown only after an edit)         │
│   ⋮                  │ ┌──────────────────────────────────────────────┐ │
│ ☐ Paragraph 42       │ │ There was a man sent ~~by Allah~~ from God   │ │
│ ──────────────────── │ │ whose name was ~~Yohanna.~~ John.            │ │
│ 3 / 42 checked       │ │ (removed = red strikethrough, added = green) │ │
│ [ Done and export ]  │ └──────────────────────────────────────────────┘ │
│ Enabled once all     │                                                  │
│ checked              │                                                  │
└──────────────────────┴──────────────────────────────────────────────────┘
```

The sample text is from the Gospel of John, using the Christian / Bible profile. The screen works the same for any Arabic text. The highlighted row in the navigator is the paragraph shown on the right. The English box is the only editable area. The diff box appears only after the reviewer changes that paragraph's English. Done stays disabled until every paragraph is checked.

---

## Data model and persistence

Two records hold everything. The original machine translation is stored separately from the edited text, so the diff always has a fixed baseline.

**Document**

| Field | Description |
| --- | --- |
| id | Unique identifier |
| file_name | Original PDF name |
| page_count | 1 to 30 |
| translation_profile | Profile used to translate it (e.g. general, christian-bible) |
| status | uploading, extracting, translating, in_review, done, failed |
| created_at / updated_at | Timestamps |

**Paragraph**

| Field | Description |
| --- | --- |
| id | Unique identifier |
| document_id | Parent document |
| position | Order in the document, starting at 1 |
| source_text | Arabic paragraph as extracted |
| machine_translation | Original English output, never changed |
| current_translation | Reviewer's latest text, initially equal to machine_translation |
| translation_status | pending, translated, failed |
| is_checked | Reviewed checkbox state |
| updated_at | Last edit time |

Derived values are computed, not stored: "edited" means current_translation differs from machine_translation, and the diff is computed from those two fields. All changes save to the server database immediately, so closing the browser or reloading never loses work.

---

## Error handling

The rule: the reviewer is always told what went wrong and what to do next, and saved work is never lost.

| Situation | System behavior | Message to reviewer (draft) |
| --- | --- | --- |
| File is not a PDF | Reject before upload | "Only PDF files are supported." |
| PDF over 30 pages | Reject after page count | "This PDF has 45 pages. The limit is 30." |
| PDF is scanned or has no text layer | Reject after extraction finds no text | "This PDF appears to be scanned. Please upload a text-based PDF." |
| PDF is corrupt or password-protected | Reject | "This PDF can't be opened. Check that it isn't damaged or password-protected." |
| Upload interrupted | Discard partial file | "Upload failed. Please try again." |
| Extraction fails | Document marked failed | "We couldn't read the text in this PDF." |
| Translation fails for one paragraph | Paragraph flagged in navigator; others continue | "Translation failed for this paragraph." + Retry button |
| Translation service unavailable | Pause and retry automatically, then flag | "Translation is temporarily unavailable. Retrying..." |
| Autosave fails | Keep edit locally and retry; show unsaved indicator | "Changes not saved. Retrying..." |
| Reviewer leaves with unsaved changes | Browser warning | Standard "leave page?" prompt |
| Export fails | Keep document state; allow retry | "Export failed. Please try again." |

---

## Non-functional requirements and technology

- **Platform:** a web app in current desktop browsers (Chrome, Edge, Firefox, Safari). Mobile layout is not required.
- **Bidirectional text:** Arabic renders right to left with a font that shows diacritics correctly; English renders left to right; neither breaks the layout.
- **Text fidelity:** Arabic text and special characters are stored as UTF-8 and never altered.
- **Persistence:** all state lives in a server-side database. The browser holds only unsaved edits waiting for retry.
- **Responsiveness:** switching paragraphs and showing the diff feel instant for a 30-page document. Translation time itself is out of scope.
- **Access:** no authentication. The app is deployed for internal use only and is not exposed publicly.
- **Translation engine:** a large language model (e.g. Claude) is recommended, because the prompt can carry the active profile's conventions and glossary (TR-2). The engine sits behind one interface so it can be swapped.
- **Suggested stack (non-binding):** React front end; a Node or Python back end; a PDF text extraction library with Arabic support; a word-level diff library; SQLite or Postgres.

---

## Acceptance criteria

The MVP is complete when a reviewer can do all of the following with a real 30-page Arabic PDF.

- [ ] Upload the PDF and see every paragraph translated, with no manual steps (UP-1, EX-2, TR-1)
- [ ] Get a clear rejection for a scanned PDF, a non-PDF, and a 31-page PDF (UP-2)
- [ ] Click any paragraph in the navigator and see Arabic and English side by side (RV-2)
- [ ] Edit the English and see the red/green word diff appear immediately (RV-3, DF-1 to DF-3)
- [ ] Close the browser, reopen it, and find every edit and checkbox as it was left (RV-4)
- [ ] Tick and untick paragraphs freely; see the progress count update (CK-2, RV-5)
- [ ] See Done disabled with paragraphs unchecked, then enabled once all are checked (EXP-1)
- [ ] Download an English-only file with paragraphs in original order and no diff markup (EXP-2, EXP-3)
- [ ] Retry a paragraph whose translation failed, without re-uploading (TR-5)
- [ ] Translate a non-religious Arabic PDF (e.g. a news article) with the General profile, and a Bible passage with the Christian / Bible profile, each following its own conventions (TR-2)

---

## Open questions and future phases

**Open questions**

- [ ] Export file format: .docx (editable before publishing) or PDF (final layout)? This draft assumes .docx.
- [x] English Bible translation for the Christian / Bible profile: NKJV (decided; see TR-2b)
- [ ] Which glossary terms does the Christian / Bible profile need (e.g. denomination-specific terminology)?
- [ ] Which other profiles are needed after General and Christian / Bible?
- [ ] Should the reviewer pick the profile at upload (a dropdown), or is setting it in configuration enough for the MVP?
- [ ] Should the export keep basic structure from the source, such as headings, verse numbers, or lists, or be plain paragraphs?
- [ ] Who hosts the app, and is an internal network deployment enough to stand in for login?

**Future phases**

1. Accounts and roles (Admin, Reviewer), then multiple documents per workspace
2. 2nd-phase review, building on the stored diffs
3. Manual merge and split of paragraphs; scanned PDF support with OCR
4. Project management: assignment, deadlines, progress dashboards
5. Publishing workflow and output formats
6. Profile picker at upload, plus an in-app editor for profiles and glossaries
