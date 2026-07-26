# dev-loop config

## Artifacts
- plans_dir: docs/plans
- phase_folder: <NN>-<slug>
- design_doc: design.md
- plan_doc: implementation.md
- product_design: docs/design.md

## Run
- web: `pnpm dev` → http://localhost:3000

## Automated gate
- test: `pnpm test`
- lint: `pnpm lint`
- typecheck: `pnpm typecheck`

## Visual gate
- enabled: true
- engine: browse                    # gstack browse daemon; chrome-plugin as fallback
- refs_dir: .claude-design          # created when the design direction is approved (design-directions phase)
- spec_docs:
  - docs/design.md
  - docs/design-system.md           # written once the design direction is approved
- max_iterations_per_screen: 6

## Screens map
| screen | web route | reference                                  |
|--------|-----------|--------------------------------------------|
| chat   | /         | .claude-design/ (after approved direction) |

## End of phase
- qa: http://localhost:3000
- code_review: on                   # /review
- security_review: on               # /cso
- adherence: not configured yet — revisit after the design system exists

## Ship
- enabled: true                     # /ship via gstack; merge is always human

## Extra rules
- All UI work loads the design skills: `frontend-design`, `impeccable`, `taste-skill` (project) and `ui-ux-pro-max` (global). Before build, produce 2-3 design directions and get human approval; the approved direction becomes `.claude-design/` refs + `docs/design-system.md`.
- Final audit before ship: `/web-interface-guidelines` (global command).
- UI copy in English. No emojis in production UI; icons from one icon family only.
- WCAG AA contrast on every interactive element; loading/empty/error states are mandatory for every surface.
- Mobile-first: layouts are designed at 390px first, then scaled up.
