# Prototype

UI/UX prototype for the project. The prototyping tool is selected during `/docs-prototype`
(see the **Prototyping Tool** field in `prototype-brief.md`) and can be Stitch, pen.dev
(Pencil), simple ASCII mockups, draw.io, or another tool — this section stays agnostic to
that choice.

## Files

| File                  | Purpose                                                                                                  |
| --------------------- | -------------------------------------------------------------------------------------------------------- |
| `README.md`           | This file — directory overview and quick start                                                           |
| `prototype-brief.md`  | Authoritative source of truth: page definitions, user flows, scope, requirements coverage matrix         |
| `design-direction.md` | Visual direction, color strategy, typography, component inventory, accessibility, responsive breakpoints |
| `pen/`                | Only if pen.dev/Pencil is selected — place your `.pen` design file here (not committed if >50 MB)        |
| `drawio/`             | Only if draw.io is selected — place your `.drawio` mockup file here                                      |

Stitch prototypes are hosted — link the project URL from `prototype-brief.md` instead of
storing a local file. ASCII mockups live inline, directly in `prototype-brief.md`.

## Opening the Prototype

- **pen.dev / Pencil** — open the `.pen` file from `pen/` via **File → Open** in the Pencil
  app, or the Pencil CLI:

  ```bash
  open docs/05-prototype/pen/<your-prototype>.pen
  ```

- **draw.io** — open the `.drawio` file from `drawio/` in the draw.io desktop app or
  [diagrams.net](https://app.diagrams.net).
- **Stitch** — follow the project URL recorded in `prototype-brief.md`.
- **Simple mockups (ASCII)** — read them directly in `prototype-brief.md`; no external
  tool required.

## Scope

Define what the prototype covers (e.g. Discovery and Planning workflows only) and what is out of scope (e.g. delivery, sprint, maintenance). This prevents scope creep during the design phase.

## Source References

- [Prototype Brief](./prototype-brief.md)
- [Design Direction](./design-direction.md)
- [Project Overview](../00-context/overview.md)
- [Requirements by Feature](../01-requirements/README.md)
- [Architecture](../03-architecture/README.md)
