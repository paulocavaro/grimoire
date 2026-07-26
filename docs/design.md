# Grimoire — Design

A mobile-first PWA chat that answers D&D 5e rules questions with source-cited answers from the SRD. One screen, maximum polish — a rules oracle you can pull out at the table mid-session.

## What it does

The user asks a rules question in plain English ("How does grappling work?", "What does Fireball do at 5th level?"). The assistant streams an answer grounded in the System Reference Document (SRD) — the openly licensed portion of the D&D 5e rules — and every answer cites the SRD sections it came from, rendered as tappable source cards.

One screen. No accounts, no history persistence, no settings page.

## Why these choices

- **Source-cited answers via tool use** (not prompt stuffing, not bare model knowledge): the model decides when to search, the UI shows it searching, and citations link to real SRD passages. Trustworthy answers are the whole product: at the table, a wrong rule is worse than no answer.
- **Local SRD index** (not live third-party API calls at runtime): the app never depends on someone else's uptime, and search quality is ours to control.
- **One screen**: polish concentrates instead of spreading.

## Stack

| Layer | Choice |
|---|---|
| Framework | Next.js 15 (App Router) + React 19 + TypeScript |
| Styling | Tailwind CSS v4, mobile-first |
| AI | Vercel AI SDK (`ai` + `@ai-sdk/anthropic`), Claude with streaming + tool use |
| Motion | Motion (`motion/react`) for micro-interactions |
| PWA | Serwist (manifest, service worker, installability, offline shell) |
| Search | MiniSearch over a local JSON index, in-memory on the server |
| Tests | Vitest (search index unit tests) |
| Deploy | Vercel |

## Architecture

```
scripts/ingest-srd.ts     build-time: dnd5eapi data → data/srd-index.json
app/page.tsx              the chat screen (server shell + client chat)
app/api/chat/route.ts     streamText + searchSRD tool (Anthropic key lives here)
lib/srd-search.ts         MiniSearch index load + query (top-k passages)
lib/prompt.ts             system prompt (grounding + citation rules)
```

### SRD data pipeline (build-time)

`scripts/ingest-srd.ts` runs once (and on demand) at development time, never at runtime:

1. Fetch from dnd5eapi (or the 5e-bits source JSON): rule sections, spells, conditions, monsters (core fields), equipment basics.
2. Normalize into flat passages: `{ id, category, title, text, srdUrl }`. Long sections are split into passages of roughly 200–400 words so retrieval returns focused chunks.
3. Write `data/srd-index.json`, committed to the repo.

### Chat flow (runtime)

1. Client uses `useChat` (AI SDK) → `POST /api/chat` with the message history.
2. Route handler calls `streamText` with Claude, the system prompt, and a `searchSRD(query)` tool. `maxSteps: 3` (the model may search up to 3 times before answering).
3. `searchSRD` queries the MiniSearch index and returns top-k passages (id, title, excerpt, srdUrl).
4. The model writes the answer with inline citation markers (`[1]`, `[2]`) mapped to the passages it used.
5. The stream carries text deltas AND tool events; the UI renders a "Searching the SRD…" state while the tool runs, then the answer, then source cards.

System prompt rules: answer only from retrieved passages when a rule is specific; say so when the SRD does not cover the question (e.g. content outside the SRD license); keep answers tight; always attach citations for rules claims.

## The screen

Mobile-first, one column, full height (`100dvh`):

- **Header** — wordmark + install hint (subtle).
- **Empty state** — composed intro with 3–4 suggestion chips ("How does grappling work?", "Explain the Prone condition", "What does Fireball do?", "How do death saving throws work?"). Tapping a chip sends it.
- **Message list** — user/assistant bubbles. Assistant messages render markdown, inline citation markers, and a row of source cards (title + category + SRD link) under the answer. While a tool call is in flight: "Searching the SRD…" indicator with subtle motion.
- **Input bar** — fixed to the bottom, safe-area aware, send button, disabled state while streaming, stop button during generation.

States: loading (skeleton shaped like the final layout), empty (the composed intro), error (inline message with retry — never a dead end). Buttons and text meet WCAG AA contrast.

Visual direction: decided at build start — 2–3 design directions are produced first (using the design skills toolchain), one is approved, then implemented consistently. The approved direction's tokens (palette, type pair, radius scale, motion signature) are documented in `docs/design-system.md` once chosen.

## PWA

- `manifest.json` with icons, name, theme color; installable on iOS/Android home screen.
- Serwist service worker: precache the app shell and static assets. Chat requires network (LLM + server-side search); offline shows a friendly "you're offline" state instead of a broken screen.

## Error handling

- Missing/invalid `ANTHROPIC_API_KEY` → clear server log + friendly inline error in chat.
- Anthropic rate limit / overload → inline error with retry, streamed partial answer preserved.
- Tool/search failure → the model is told the search failed and answers accordingly (or asks the user to retry); never a silent empty result presented as fact.

## Quality gates

- `pnpm typecheck`, `pnpm lint`, `pnpm test` (Vitest: ingest normalization + search relevance smoke tests).
- Visual gate: screen compared against the approved design direction (dev-loop visual verify).
- Final pass: accessibility/interface audit (keyboard focus, ARIA, contrast) before deploy.

## Development workflow

Built with dev-loop (staged gates: think → plan → build → ship) via a thin `.loop/config.md` adapter. This design doc is the Think-stage artifact.

## Out of scope (v1)

- Accounts, chat history persistence, multi-conversation.
- Content beyond the SRD (no Tormenta/GURPS/homebrew; the architecture leaves room for other rule sets later).
- Portuguese UI (English only).
- Voice input, dice rolling, character sheets.
