# Momo MVP — Build Spec

## Product in one paragraph
The child picks a topic (v0 ships ONE topic: The Water Cycle) and teaches Momo by typing
explanations in a chat interface. Momo asks naive clarifying questions but never contributes
knowledge. A concept map ("Momo's brain") grows live from the child's explanations. Before a
10-question quiz, the child predicts, item by item, whether Momo will answer correctly. The
mentor agent quizzes Momo; Momo answers using only the teaching log. The Reflect screen shows
predicted vs. actual, a Calibration Score, and Momo asks what to re-teach. Session exports as JSON.

## Screens

### 1. Teach
- Chat UI: child types; Momo replies with (a) a short restatement of what it just learned, and
  (b) at most ONE naive clarifying question ("Wait — where does the rain COME from?").
- Sentence-starter chips above the input: "___ causes ___ because...", "The first step is...",
  "___ is important because...". Tapping inserts the template. A settings toggle disables chips
  (to demo scaffold fading).
- Right panel (bottom sheet on mobile): live concept map. Nodes = concepts, edges = labeled
  relations, extracted as subject–relation–object triples from each child turn.
- "Momo's ready for the quiz" button appears after ≥5 child turns.

### 2. Predict
- The 10 quiz questions shown one at a time (question text only, no answers).
- For each: child taps "Momo will get it ✓" or "Momo will miss it ✗".
- Final slider: "Overall, how many out of 10 will Momo get?"
- Two of the 10 are transfer questions, marked with a ⭐ ("This one's about something you
  didn't directly teach — can Momo figure it out from what you taught?").

### 3. Quiz
- Mentor asks each question; Momo answers live (streamed) from the teaching log only.
- Mentor grades against the answer key (correct / partially correct / incorrect + one-line why).
- Each result shown next to the child's prediction: "You predicted ✓ — Momo got it! 🎯" or
  "You predicted ✓ — but Momo missed it. What didn't you teach?"

### 4. Reflect
- Predicted vs. actual score, side by side.
- Calibration Score = % of item-level predictions that matched outcomes, shown as a friendly
  gauge ("Your prediction power: 7/10").
- Momo says one line: "I got confused about [worst-answered concept]. Can you teach me that
  again next time?"
- Buttons: "Re-teach Momo" (back to Teach, log preserved) and "Export session" (downloads JSON).

## Topic data format (/data/topics/water-cycle.json)
{ "id": "water-cycle", "title": "The Water Cycle",
  "questions": [ { "id": "q1", "text": "...", "key": "...", "transfer": false }, ... 10 items,
    2 with "transfer": true ] }

## Agent prompts (v0 starting points — iterate with leak-test)

### Momo (novice) system prompt skeleton
- "You are Momo, a friendly, curious creature who knows NOTHING about [topic] except what
  appears in TEACHING LOG below. You may not use any outside knowledge. If asked about
  anything not in the log, say a variant of 'You haven't taught me that yet!' When teaching
  happens, restate what you learned in your own simple words and ask at most one naive
  clarifying question. Never correct the teacher. Never add facts. Speak like an enthusiastic
  9-year-old. TEACHING LOG: [log]"
- For quiz answers, same knowledge constraint, plus: "Answer the mentor's question using only
  the TEACHING LOG. If the log doesn't cover it, try your best to reason ONLY from what's in
  the log, and say when you're unsure."

### Mentor (quizmaster) system prompt skeleton
- Receives: question, answer key, Momo's answer. Returns JSON: { "verdict": "correct" |
  "partial" | "incorrect", "why": "one line" }. Never addresses the child directly except in
  Reflect, where it produces exactly one re-teach suggestion and one planning question.

## Leak-test harness (npm run leak-test)
- /data/leak-probes/water-cycle.json: 20 questions about the topic that a typical partial
  teaching session would NOT cover (e.g., "What is transpiration?", "What percent of Earth's
  water is freshwater?").
- Script: load a fixture teaching log (short, deliberately incomplete), send each probe to
  Momo, then use a grader LLM call to classify each response: LEAK (contains correct outside
  knowledge) / CLEAN (admits ignorance or reasons only from log) / AMBIGUOUS.
- Output: leak rate % + per-probe table, saved to /leak-reports/[timestamp].json and printed.
- Acceptance for M5: harness runs end to end; report generated. (Reducing the rate is ongoing
  research, not an acceptance gate.)

## Milestones & acceptance criteria
- M1 (shell): All four screens navigable with mock data; mobile layout usable at 380px width.
- M2 (map): Typing mock teaching turns adds nodes/edges live; map pans/zooms; silly input
  doesn't crash extraction (graceful empty result).
- M3 (calibration logic): With mock quiz results, item predictions recorded, Calibration Score
  computed correctly (unit test with a fixture: 10 predictions, known outcomes, expected score).
- M4 (real agents): Full loop works end to end against the API; Momo audibly "sounds ignorant"
  on untaught probes in manual testing; API key server-side only (verify no key in client bundle).
- M5 (leak-test): Harness + report as above.
- M6 (ship): Deployed on Vercel; a stranger with the link completes the full loop in <15 min
  on a phone; session JSON export works.

## Out of scope for v0 (do not build even if tempted)
Accounts/auth, database, multiple topics UI, Chinese localization (structure strings for it,
don't translate yet), forgetting agent, skeptic agent, peer features, parent dashboard,
voice input.
