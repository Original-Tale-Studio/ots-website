# OTS Architecture — Conceptual View (Fast Track, Literary Track & Gutenberg Track)

> High-level, non-technical overview of the Fast Track (FT), Literary Track (LT),
> and Gutenberg Track (GT) architecture and workflows.

The system is an **online translation service** built around three big pieces:

1. **Web portal (Frontend + API)** — customers place orders, upload files, pay, and download results. Admins manage quotes, pricing, and human workers.
2. **Automated pipeline (Pipeline jobs)** — the "machine room": each translation order is processed as a chain of self-contained stages (preprocess → translate → QA → deliver).
3. **Orchestrator (Workflow)** — the "conveyor belt": when a payment is confirmed, a message kicks off the orchestrator, which decides which track to use (Fast, Literary, Gutenberg) and runs the stages one by one, waiting for each to finish before starting the next.

The source material is Taiwanese Hokkien (台語) writing; tracks differ in *how much human care* is layered on top of automated translation.

---

## Fast Track (FT) — "Fully automated, speed first"

**Who for:** standard documents, small-to-medium size, straightforward content.

**Customer journey:**
1. Upload file, get instant upfront price, pay.

**Pipeline stages (linear, hands-off):**

```
File prep → Machine translation → Auto QA → (conditional human QA) → Delivery
```

- **File prep**: Text is extracted and segmented into clean units.
- **Machine translation**: Each unit is translated automatically.
- **Auto QA**: Automated checks validate the output (completeness, consistency, terminology, etc.).
- **Human QA (conditional)**: Only if QA produces serious "must-fix" issues, a human reviewer is notified and the workflow *pauses* (polls periodically, up to ~24h) until all serious issues are resolved.
- **Delivery**: Final files in multiple formats (plain text, formatted HTML, bilingual parallel view) are generated and stored; the customer is notified.

**Character:** a straight conveyor belt. Humans are the exception, not the rule.

---

## Literary Track (LT) — "Human-in-the-loop, quality first"

**Who for:** literary works, long manuscripts (10K+ words) where nuance matters.

**Customer journey (quote-first):**
1. Upload file → order sits in **awaiting quote** state (no upfront pricing).
2. Admin reviews and sends a **quotation**.
3. Customer pays after accepting the quote → pipeline starts.

**Pipeline stages (machine first, then humans):**

```
File prep + Machine draft → Auto QA → [Editor] → [Proofreader] → Final QA → Delivery
```

- **Machine draft**: A combined stage produces an AI first-draft translation of the whole work.
- **Auto QA (early)**: Automated checks run *on the machine draft before humans see it* — the flagged issues are surfaced to the editor so they can address them while editing.
- **Editor (human)**: Admin assigns a professional **editor**, who reworks the AI draft into publishable literary translation. The workflow patiently waits (up to ~a week).
- **Proofreader (human)**: The admin next assigns a **native-language proofreader**, who polishes the edited text. Another waiting period (up to ~2 days).
- **Final QA checklist**: One more automated pass over the human-polished text.
- **Delivery**: Same multi-format output package as FT.

**Character:** same conveyor belt, but with long "human workstations" inserted; the orchestrator simply waits patiently and alerts admins if humans exceed their time budget.

---

## Gutenberg Track (GT) — "Public-domain books, three editions, human gates"

**Who for:** public-domain books sourced from **Project Gutenberg** — long, chaptered works where the same book is published in **three parallel editions** (standard translation, simplified youth edition, and a Tâi-lô romanization annotated edition).

**Customer journey:**
1. A book (Gutenberg ID) is submitted as an order — the content is pre-existing public domain, so there is no file upload; pricing/payment happens at order creation.

**Pipeline stages (longest conveyor belt, with two human "quality gates"):**

```
Fetch & split → Chapter detection → Glossary building → Machine translation
      → Auto QA → [Gate 1: human review of translation]
      → Simplification (youth edition) → [Gate 2: human review of simplified prose]
      → Romanization annotation (Tâi-lô edition) → Delivery
```

- **Fetch & split**: The book is downloaded from Project Gutenberg and broken into translation-sized pieces.
- **Chapter detection**: The book's chapter structure is discovered — first attempted intelligently (AI), falling back to simple heading patterns, and finally to grouping paragraphs into "Parts" when the book has no headings at all.
- **Glossary building**: Key terms and character names are extracted up front so the **whole book stays consistent** (e.g., a name is translated the same way in chapter 1 and chapter 40).
- **Machine translation**: All chapters are translated using the shared glossary.
- **Auto QA (pre-Gate)**: Automated checks run on the raw translation before any human looks at it.
- **Gate 1 — human review (translation)**: A human reviewer approves/fixes the **standard edition** segment by segment. The conveyor belt waits here.
- **Simplification (youth edition)**: From whole chapters of the *reviewed* translation, a simplified, easier-to-read version is produced (chapter-level context, not sentence-by-sentence).
- **Gate 2 — human review (simplified)**: A human reviews the **youth edition** chapter by chapter. The belt waits again.
- **Romanization (Tâi-lô edition)**: A third edition is generated — the text annotated with Tâi-lô romanization, segment by segment across the full book.
- **Delivery**: All three editions are packaged together (plain text + formatted HTML per edition); the customer is notified.

**Character:** FT's automation combined with LT's patience — machine-heavy stages, each with its own tuning, punctuated by **two mandatory human review gates**. Because all three editions derive from one shared, human-approved master translation, quality work is only done once.

---

## Key contrasts

| | Fast Track | Literary Track | Gutenberg Track |
|---|---|---|
| Pricing | Instant, upfront | Quotation → approve → pay | Paid at order creation (public-domain source) |
| Automation | End-to-end | Machine draft + human finishing | Machine stages + two mandatory review gates |
| Humans | Only on serious QA flags | Editor + proofreader always involved | Human review of translation and youth edition |
| Speed | Minutes–hours | Days | Longest (full-book, 3 editions) |
| QA timing | After machine output (blocking) | Early pass shown to editor + final checklist | Auto QA after translation, before Gate 1 |
| Time budgets | Human QA window ~24h | Editor ~7 days; proofreader ~2 days (system alerts admins on timeout) | Gates wait for human review on the portal |
| Output | Multi-format single translation | Multi-format polished translation | Three editions (standard / youth / Tâi-lô) |

Everything else (order states, payments, delivery) is shared infrastructure; the tracks are essentially **different recipes** running through the same orchestrator.

---

## The GT Video Storyboard Feature — "From book to screen, on demand"

> Conceptual view of the user-triggered video storyboarding capability that
> lives *alongside* the Gutenberg Track. Not part of the main conveyor belt —
> it's a second, optional production line.

### When it happens

After a Gutenberg order has finished the normal text pipeline (delivered) and a
human has verified the output, someone (admin/authorized user) **chooses** to
turn the book into video content by clicking "Generate Storyboard". Nothing
starts automatically — video is a deliberate, on-demand project layered on top
of a finished book.

### The idea

The book → screen flow works like a small film production, where AI plays the
creative crew and humans play the director:

```
Finished book ──▶ (user clicks "Generate Storyboard")
        │
        ▼
  AI Film Director ──▶ Storyboard (scene-by-scene shooting script) ──▶ Human edit/sign-off
        │
        ▼
  Per-scene production (AI studio):  narration audio · scene artwork · animated clips
        │
        ▼
  Assembly: chapter videos · subtitles ──▶ Finished video
```

### The stages, conceptually

1. **The director reads the youth edition.** The machine (AI) reads the simplified,
   human-approved edition of the book — the one designed to be easy to follow —
   because that's the version that works best as narration.
2. **The lookbook.** The AI first creates a "visual style guide": who the
   characters are, how they look, what the world feels like — so that scene 40
   looks like scene 1. Same philosophy as the GT glossary, but for images.
3. **The shooting script (the storyboard).** The AI breaks each chapter into
   distinct scenes, and for each scene writes: the narration line, what we see
   (visual prompt), and how it flows. This scene-by-scene script is the
   **storyboard** — the thing the user asked the system to generate.
4. **The editing room (human).** The user works in an interactive studio page:
   chapter tabs, one card per scene. They can edit narration text, regenerate
   art, preview each piece, until it feels right. This is also a formal human
   review gate (Gate 3) like the translation/simplify reviews.
5. **The dailies.** For each scene, on demand: spoken narration (two language
   tracks available), scene artwork, and an animated clip. Preview it right in
   the card.
6. **The final cut.** Scenes are assembled into chapter videos, subtitles
   generated, and everything packaged into the finished video deliverable.

### Character

- **Out of band**: unlike FT/LT/GT text pipelines, this is *not* triggered by
  payment or the conveyor belt. A human presses the button, when they're ready.
- **Generate → curate → produce**: AI produces the draft creative work cheaply;
  humans curate it; then (and only then) is the expensive per-scene media
  (audio/video) generated.
- **Reuse of approved work**: it builds on the already-human-approved youth
  edition, so no re-translation of the text happens here.
- **Bilingual narration**: the same storyboard supports two narration language
  tracks; pick a track, generate its audio, its subtitles, its chapter videos.
