# Fluidez

Spanish for English speakers, in about twenty minutes a day. Text only, works offline, no account, no ads, no hearts.

**Play it:** https://matt-mccrea.github.io/fluidez-next/

This repository holds the published build of the Fluidez rebuild. The source lives on the `rebuild` branch of [Matt-McCrea/Fluidez](https://github.com/Matt-McCrea/Fluidez/tree/rebuild), under `app/`, and every push there rebuilds this site. The notes below explain what the app is trying to do and how it does it.

---

## The educational philosophy

Fluidez is built around one question: what does the evidence say actually moves an adult from zero to holding a real conversation in Spanish, and what would an app look like if it took that seriously?

The answers it commits to:

**Retrieval beats recognition.** You learn a word by pulling it out of your own head, not by seeing it again. Most app exercises are recognition dressed up as practice. Fluidez makes production, English to Spanish, the default direction for grammar, patterns and useful chunks, and for vocabulary as soon as a word has been met once in the other direction. The aim is that at least half of all the retrievals a learner does are productive.

**Spacing is a schedule, not a feature.** Every item you learn is a card in a real spaced repetition model, and the whole daily session is built around what that model says is due. The app never trims reviews to make room for new material. When the backlog grows, new material is cut first, then lighter days convert to review days. The path only advances when the memory work is keeping up.

**Nothing appears before it is taught.** The course is a graph of about 6,100 skills: words, grammar points, sentence patterns, fixed chunks, contrasts such as ser versus estar, and can-do statements. Every sentence, story chapter, dialogue scene and writing task is tagged with exactly which skills it needs. The session compiler will only show you content whose requirements you have already met. Readings land at 95 to 98 percent known words by construction, not by editorial care.

**A correct chunk is not mastery.** Getting "¿Cómo te llamas?" right proves you remember a chunk. Producing "¿Cómo se llama tu hermana?" proves you own the pattern. Practice climbs a ladder for each skill: choose the right form, then type it, then transform it, then recombine it into a sentence you have never seen. Only novel recombinations count as evidence of mastery.

**Interleave on purpose.** Drill days mix the newest grammar point with older ones and with a known contrast, so you practise choosing between forms, not just producing one form on demand. The rhythm changes every few cards, and no more than four cards of the same type run in a row.

**Honest checking, honest feedback.** If the app says it checks something, it checks it. Typed Spanish is matched against the model answer with a matcher that understands dropped subject pronouns, clitic position, adverb placement and the tú versus usted choice. When you miss, you get one plain-English hint that names the kind of error without giving the answer, and one attempt to fix it before the reveal. "I don't know" is always available on a pick card, and it counts as a miss rather than a lucky guess.

**One story, one voice.** A story set in Bogotá runs through all sixty units, so there is always a reason to come back. The Spanish is Latin American, Colombian where it matters. There is no jargon on screen, no block names, no points, no streak punishment. The close of each day tells you what came back from last week and what the story's cliffhanger is.

**Text only, by design.** No audio, no listening track, no AI feedback in the app. That keeps it fast, offline and private, and it keeps the learning loop measurable.

The two research briefs the design is drawn from are in the source repository under `docs/research/`.

---

## What the course contains

| | Count |
|---|---|
| Units (A1 to C1) | 60 |
| Skill graph nodes | 6,143 (4,554 words, 650 patterns, 404 chunks, 269 can-dos, 172 grammar points, 94 contrasts) |
| Practice sentences | 71,390 |
| Story chapters | 219 |
| Dialogue scenes | 270 |
| Writing tasks | 875 |
| Teaching scripts | 916 |

Units are nine or ten days. Days are five to eight blocks. Blocks are short runs of four to twelve cards. A card is one question, one answer.

---

## How a day is built

Each unit file lists its days and what they introduce. The **session compiler** turns that plan plus your memory state into today's session, deterministically, so a given learner on a given day always gets the same session and it can be reproduced in a test.

There are seven kinds of day: introduce, drill, input, patterns, output, review and checkpoint, plus a minimum day for when you are short of time. The compiler chooses by precedence: spacing rules first (one new grammar point per day, never two days running, always followed by a drill within three days), then the backlog throttle, then whatever the unit plan says.

A typical day:

1. **Warm-up.** Reviews due from the memory model, lowest retrievability first. Both directions of the same word never appear in the same session.
2. **Teach.** A short script with a human reason for the point, a few examples to compare, the rule in one line with a "why" underneath, a read-and-pick, your turn, the pitfalls, and a personal sentence that is checked for form.
3. **Practice.** The ladder for the new point, interleaved with older points and a contrast.
4. **Input or output.** A story chapter in beats with a check between beats, a dialogue scene, or a writing task with a rubric that is saved and shown next time.
5. **Before you go.** Everything introduced today, and anything you missed, served once more as production.

Introduction is not a single exposure. A new item is retrieved three to five times inside its first lesson, and the first spaced repetition rating is derived from how that went.

---

## The memory model

Every skill has up to two cards: receptive (Spanish to English) and productive (English to Spanish). Cards are scheduled with FSRS, the open spaced repetition algorithm, at a target retention of 90 percent. With consecutive correct answers that gives intervals of roughly 3, 11, 35 and 101 days.

Reviewing one sentence gives bounded partial credit to every other skill in that sentence, so the arithmetic works at tens of thousands of sentences without tens of thousands of reviews.

Every outcome, graded or not, is written to an append-only review event log. That is what makes the progress screen honest: it reports delayed retrieval accuracy on items at least seven days old, production share, and backlog against budget, rather than a count of sessions completed. It is also what will let the scheduler be fitted to each learner later.

Multi-skill items are graded per skill: one wrong word lapses one skill, not three. Self-check and checkpoint results feed a remediation set that the next drill draws from first. Leeches are detected from the event log.

---

## Checking

- **Translation matcher.** Canonicalises subject pronoun drop, clitic climbing, adverb position, question word order, tú versus usted when the English is open, and accepts any other bank sentence with the same English. Peninsular forms are accepted and flagged, not failed.
- **Diagnosis.** Names the first real error in plain English: gender, one versus many, who is doing it versus when it happens, ser versus estar, por versus para, a missing small word.
- **Accents.** Strict where they change meaning, such as verb endings. Lenient on chunk and word cards. Every Spanish field has an accent bar that keeps the caret.
- **Output.** Personal sentences and scene replies must be mostly Spanish and contain the form the day teaches. The app says what it can judge and what it cannot.

---

## Reference and free practice

A reference tab with verb tables by any form, a grammar index grouped by what you want to say, and a word list with strength and last seen sentence. Every entry has "Practise this". Practice modes: quick review of your weakest items, a conjugation grind, a translation grind, and a drill on any set of skills. Home shows up to three things that keep tripping you with a "Train it" button.

---

## Content pipeline

The content was written by language models against a strict contract and a strict gate. Each unit's words, grammar and patterns come from the course graph; a tagger computes what every sentence requires; and `npm run gate` refuses anything that breaks schema, dialect, accent rules, legality (grammar ahead of schedule), reading coverage, near-duplicate limits, frame monotony, typeability or chapter length. A per-unit reviewer and a story audit run on top of the gate. Nothing lands without passing it.

---

## Architecture

- **Stack.** TypeScript, Preact, Vite, a progressive web app with a service worker. Works offline after first load. No server, no account; state is in IndexedDB on the device.
- **Content.** JSON under `content/`: `graph/` (the skill graph), `units/` (the 60 unit plans), `bank/`, `scripts/`, `story/`, `scenes/`, `writing/`. Each is bundled as a lazily loaded chunk per unit so the first load is small and later units arrive as needed.
- **Compiler.** `src/compiler/` builds a session from a unit plan, the learner state and a seed. Pure and deterministic.
- **Learner.** `src/learner/` holds the FSRS scheduler, the two-card model, implicit credit, the review event log, weak-spot detection and versioned persistence.
- **Checking.** `src/check/` holds the translation matcher, the diagnosis, the writing constraints and the verb analysis.
- **UI.** `src/ui/` holds screens, blocks and cards, the session runner, placement, reference and practice.
- **Tooling.** `tools/` holds the gate, the tagger, the simulator (four learner profiles over 400 days), the supply checker and the content job queue.
- **Tests.** About 2,500 unit and property tests, including property tests over random courses (no content before its requirements, no repeats, throttle and spacing hold) and a Playwright run through ten real sessions.

---

## Status

The course is content-complete from A1 to C1 and the app is feature-complete for its first version. It is in playtesting. Known open items: the upper units have been machine-reviewed but not yet read by a native speaker, the gate still emits advisory warnings that deserve a cleanup pass, and the first install is larger than it should be because the whole skill graph ships up front.
