# Momo — Learn by Teaching (MVP)

A web demo where a child (age 10–14) teaches "Momo," a faithfully ignorant AI agent,
a school topic. Momo's knowledge = ONLY the child's teaching log. Core loop:
Teach → Predict → Quiz → Reflect. Full product spec: see SPEC.md (read it before planning any milestone).

## Non-negotiable rules
1. FIDELITY OF IGNORANCE: Momo must never use knowledge outside the teaching log.
   All Momo responses are generated with a system prompt containing only the log + persona rules.
   Any change to Momo's prompting must be followed by `npm run leak-test` and the leak rate reported to me.
2. API keys live ONLY in server-side code (Next.js API routes / env vars). Never in client code. Never committed.
3. This is a research prototype for children: no dark patterns, no open-ended chat outside the teaching task,
   no data collection beyond the session JSON export.
4. Plan mode first for every milestone. Show me the plan; wait for my approval before editing files.
5. After each milestone: run the app, show me evidence it works (test output or screenshot description), then commit.

## Stack
- Next.js (App Router) + React, deployed on Vercel
- Anthropic Messages API via a single server-side route (/api/agent)
- React Flow for the concept map; no database (React state + end-of-session JSON export)
- Plain CSS or Tailwind; mobile-first (kids will use phones/tablets)

## Commands
- `npm run dev` — local dev server
- `npm run build` — production build (must pass before any commit)
- `npm run leak-test` — fidelity-of-ignorance harness: probes Momo with untaught questions, prints leak rate

## Architecture notes
- Two agent personas, both served by /api/agent with different system prompts:
  - momo (novice): knowledge = teaching log only; asks naive clarifying questions; says
    "You haven't taught me that yet!" for untaught material
  - mentor (quizmaster): holds ground-truth question bank in /data/topics/*.json; grades Momo's
    answers against the key; never reveals answers to Momo
- Concept map triples are extracted server-side after each teaching turn (structured output call)
- Session state lives in a single React context; exportSession() dumps JSON (teaching log, predictions,
  quiz results, calibration score, leak-test results if run)

## Build order (one milestone per session — see SPEC.md for acceptance criteria)
M1: Four-screen shell with mock data (no API). M2: Concept map rendering from mock triples.
M3: Prediction + calibration scoring logic with mock quiz. M4: Real API wiring (Momo + mentor).
M5: Leak-test harness. M6: Polish + Vercel deploy.

## Style
- Small components, plain code, no clever abstractions — the human reviewing this is a psychology
  student learning to build; prefer readable over elegant, and add brief comments explaining WHY.
