# Grimoire

Ask the rules, get the source. Grimoire is a mobile-first PWA chat that answers D&D 5e rules questions with streaming, source-cited answers — every rules claim links back to the SRD passage it came from.

Built for the table: mid-session, you need the grappling rules in ten seconds, not a forum thread. And a wrong rule is worse than no answer, which is why the assistant searches the SRD before it speaks and shows you exactly where each answer comes from.

**Status:** in development.

## How it works

- You ask a question in plain English; the model decides when to search the System Reference Document through a `searchSRD` tool.
- Search runs over a local SRD index built at development time (no runtime dependency on third-party APIs).
- Answers stream in with inline citations and tappable source cards linking to the SRD.

## Stack

Next.js (App Router) · React · TypeScript · Tailwind CSS · Vercel AI SDK + Claude · Motion · Serwist (PWA) · MiniSearch · Vitest

See [docs/design.md](docs/design.md) for the full design.

## Content license

Rules content comes from the D&D 5e System Reference Document, used under its open license. Grimoire is an unofficial project, not affiliated with Wizards of the Coast.
