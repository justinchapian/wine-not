# Agent Guide

## How to navigate this repo

1. **Start with `INDEX.md`** — it maps tasks to the right steering file.
2. **Check `decisions.md`** before proposing architectural alternatives.
3. **Follow `feature-planning.md`** rules when modifying any docs or steering files.

## Conflict resolution

1. Code is authoritative over documentation.
2. Steering files are authoritative over `docs/` files.
3. `decisions.md` explains "why" — it wins when a rule seems contradictory.

## Verifying your work

- `npm run build` must pass
- `npm run lint` must pass
- If you modified steering/docs, check anti-drift rules in `feature-planning.md`
