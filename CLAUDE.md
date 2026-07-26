# Grimoire — Claude Instructions

## Documentation
- `docs/design.md` — product design: what Grimoire is, architecture, scope. Read it before planning anything.
- `docs/design-system.md` — visual tokens and rules (exists after the design direction is approved).
- `docs/plans/<NN>-<slug>/` — per-phase design + implementation plan (dev-loop artifacts).

## Workflow (dev-loop)
Feature phases run through the dev-loop framework. Project contract: `.loop/config.md` — read it fresh at the start of every run; state logs live in `.loop/runs/`.

Stages: `/think-phase` → `/plan-phase` (human gate) → `/execute-phase` (automated gate: `pnpm test` / `pnpm lint` / `pnpm typecheck`, atomic commit per task, visual verify per screen) → ship on explicit human go.

- Commits are atomic on the working branch; never push, merge, or open PRs without the user's explicit go.
- One phase at a time; when an `implementation.md` is done, stop and tell the user.

## Design
- Every UI task loads the design skills: `frontend-design`, `impeccable`, `taste-skill` (in `.claude/skills/`) and `ui-ux-pro-max` (global).
- Before building UI, produce 2-3 design directions for human approval. The approved direction is the visual source of truth (`.claude-design/` + `docs/design-system.md`).
- Final pass before ship: `/web-interface-guidelines` audit.

## Hard rules
- UI copy in English.
- `ANTHROPIC_API_KEY` lives in `.env.local` only — never committed, never logged, never read into chat output.
- SRD content: only data from the ingest script (`scripts/ingest-srd.ts`); the model must cite retrieved passages, never invent SRD text.
