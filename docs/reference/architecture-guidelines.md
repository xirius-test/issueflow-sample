# Architecture guidelines

This file is a **starter stub**. It ships empty on purpose — this workflow
does not know your system's architecture or module boundaries.

Producing-stage workflow guidance and quality-gate agent charters reference
this file **conditionally**: *"if `docs/reference/architecture-guidelines.md`
exists and has content, follow it; otherwise, use general architectural
judgment."* Filling this in raises the bar every agent is held to — for THIS
repo specifically — with zero charter changes required.

## Fill this in with your system's own rules, for example:

- Layering / module boundaries (what may depend on what)
- Where business logic must live vs. where it must never live
- Package/service boundaries and the contracts between them
- Patterns your team has standardized on (and anti-patterns to avoid)
- Anything your team currently enforces only in code review

Until you fill this in, agents fall back to general architectural best
practice — never a hardcoded assumption about your system's shape.
