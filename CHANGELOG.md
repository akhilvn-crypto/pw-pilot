# Changelog

Rule changes alter how agents write tests, so every change to them is listed here.

## 0.1.0 - 2026-10-04

First public release, adapted from the private `playwright-automator` skill.

- Renamed to `pw-pilot`. Frontmatter cut to `name` and a short `description`.
- Split into `SKILL.md` (rules and phases) and `references/` (code loaded only when scaffolding or exploring).
- Delegation is now agent-neutral: parallel subagents where supported, sequential with notes otherwise.
- Exploration runs through a headless probe script (accessibility snapshot plus test ids). `codegen` is kept for human-driven sessions. MCP browser tools are no longer forbidden, just not preferred.
- HAR is recorded through a `contextOptions` fixture and removed on pass. The previous `routeFromHAR` fixture left HAR files behind for passing tests.
- `tsconfig.json` updated for current TypeScript (`bundler` resolution, `paths` without `baseUrl`). Specs import through the `@fixtures/*` and `@data/*` aliases.
- Factories use `randomUUID()` suffixes instead of `Date.now()`.
- Added an auth setup project with storage state, and `.env.example`.
- Defined what counts as a spec, and when the agent asks for the base URL and credentials.
- Documented why CI runs with one worker while tests stay parallel-safe.
