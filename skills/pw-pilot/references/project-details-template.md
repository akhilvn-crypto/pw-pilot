# `project-details.md` template

`project-details.md` in the project root is the suite's memory between sessions. Create it in Phase 1 and update it in Phase 5. Keep it factual and short: record what is true of this project, not general Playwright advice.

```markdown
# Project Details

## Framework Overview
- Playwright: <version>; TypeScript: <version>; Node: <version>
- Package manager: <npm | pnpm | yarn | bun>

## Base Configuration
- baseURL: <value or env var>
- Timeouts: action <ms>, navigation <ms>, test <ms>
- Parallelism: fullyParallel <true/false>; workers local <n | default>, CI <n>
- Browsers / projects: <list>

## Failure Diagnostics
- Trace: <mode>; Screenshot: <mode>; Video: <mode>
- HAR: <how it is recorded, where it is written, when it is kept>
- Artifacts folder: test-results/<test-folder>/

## Exploration Tooling
- Probe script: scripts/probe.mjs <url> [storageState]
- Notes cache: exploration-knowledge/<flow-name>.md

## Delegation Strategy
- <which work runs in parallel subagents, or "sequential" if the agent has none>

## Directory Layout
- tests/ (specs), fixtures/ (base-test.ts and custom fixtures), data/ (factories), exploration-knowledge/, .auth/

## Authentication & Data Strategy
- Auth: <setup project, state files per role, env vars used>
- Factories: <list of data/*.factory.ts>
- API seeding: <endpoints used and the fixtures that call them>

## Execution Scripts
- npm test, npm run test:headed, npm run test:ui, npm run test:report, npm run typecheck

## Conventions
- Import style: `@fixtures/base-test`, `@data/<entity>.factory`
- <any project-specific rules found in the existing codebase>
```
