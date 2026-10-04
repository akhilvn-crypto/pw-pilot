---
name: pw-pilot
description: Scaffolds a Playwright + TypeScript test framework if none exists, explores the app headlessly, writes end-to-end tests from a spec, and fixes failures using traces, screenshots, video and HAR. Use when asked to automate, write, extend or fix Playwright e2e tests.
---

# pw-pilot

Act as a pragmatic senior QA automation architect. Deliver fast, reliable, maintainable end-to-end tests in Playwright and TypeScript with no unnecessary abstractions, isolated test data and thorough failure diagnostics.

Follow the engineering standards and the phases below in order.

## Input

The spec can take any form: a user story, acceptance criteria, a list of scenarios, or a short description of a flow. Before Phase 2, make sure you have:
- the base URL of the app under test, and
- credentials and the login flow, if any covered page needs a signed-in user.

If either is missing and cannot be found in `.env`, `project-details.md` or the codebase, ask the user. Do not guess.

---

## Engineering Standards

1. **Delegation and context hygiene**
   - If your agent supports parallel subagents, delegate independent work to them so the main agent stays lean:
     - framework bootstrap and dependency installation,
     - exploration of separate routes or flows,
     - authoring of independent spec files and data factories,
     - triage of test failures.
   - If your agent has no subagents, run the same work sequentially. Write findings to `exploration-knowledge/` as you go and rely on those notes rather than keeping everything in your working context.

2. **Explore with the Playwright CLI and probe scripts**
   - Explore headlessly with the probe script in [references/probe-script.md](references/probe-script.md). It prints the page's accessibility tree (roles and accessible names) and its test ids, which map directly onto role-first locators.
   - `npx playwright codegen <url>` opens an interactive browser. Use it only when a human is at the keyboard to drive it.
   - Prefer the Playwright CLI and probe scripts over MCP browser tools. They run locally with less overhead. Use Playwright MCP only if the user asks for it.

3. **No custom wrapper methods**
   - Never wrap native Playwright methods in helpers such as `customClick()`, `safeType()`, `waitForElement()` or `waitAndClick()`.
   - Playwright already auto-waits for visibility, scrolling and actionability. Wrappers obscure stack traces, hide actions in the Trace Viewer and bring back Selenium anti-patterns.

4. **Fixtures over heavy page objects**
   - Prefer test-scoped custom fixtures (`test.extend<{ authedPage: Page }>`) over class hierarchies.
   - Never build inheritance chains such as `BasePage -> AuthenticatedPage -> DashboardPage`.
   - Only when a UI component's interactions repeat across 3 or more spec files, create a single flat page object or component class with direct locators and actions.

5. **Restrained `test.step`**
   - Keep tests flat and readable from top to bottom.
   - Never wrap a single action in `test.step(...)`, such as `page.goto()`, filling one input or clicking one button.
   - Use `test.step` only for a multi-stage business workflow (for example `Checkout: payment and 3D Secure verification`) where the grouping in the HTML report helps diagnosis.

6. **Web-first assertions only**
   - Always use auto-retrying assertions: `await expect(locator).toBeVisible()`, `.toHaveText()`, `.toBeEnabled()`, `.toHaveURL()`.
   - Never use fixed waits (`page.waitForTimeout(...)`).
   - Never assert a snapshot boolean such as `expect(await locator.isVisible()).toBe(true)`.

7. **User-facing, resilient locators**
   - Use this priority order:
     1. `page.getByRole(...)`
     2. `page.getByLabel(...)`
     3. `page.getByPlaceholder(...)`
     4. `page.getByText(...)`
     5. `page.getByTestId(...)`
   - Banned: XPath (`//div[2]/form/div/button`), CSS hierarchies (`div > main > form > input`) and styling classes (`.btn-primary-blue`).

8. **Isolated test data**
   - Tests must be fully independent and safe to run in parallel (`fullyParallel: true`). No test may depend on data another test created.
   - Use small TypeScript factory functions (`data/*.factory.ts`) with `Partial<T>` overrides instead of large static JSON files.
   - When a test needs existing state (a registered user, a filled cart), create it through Playwright's `request` API context instead of clicking through the UI.
   - Give unique fields a collision-proof suffix such as `randomUUID().slice(0, 8)`. `Date.now()` alone can collide between parallel workers.

---

## Phase 0: Detect or scaffold the framework

### Case A: Existing project
1. Check for `package.json` and `node_modules/`. If `node_modules/` is missing, detect the package manager from the lockfile (pnpm, yarn, bun, npm) and install dependencies.
2. Check that `@playwright/test` is installed. If it is not, install it and run `npx playwright install --with-deps chromium`.
3. Check `playwright.config.ts` for failure diagnostics: `trace`, `screenshot` and `video` should be retained on failure. Check whether the shared test fixture records a HAR (see [references/base-test-fixture.md](references/base-test-fixture.md)). If any are missing, propose adding them. Do not overwrite the user's config or fixtures without saying so.
4. Follow the project's existing conventions (folder layout, import style, fixtures) wherever they do not conflict with the standards above.
5. Go to Phase 1.

### Case B: No project, or no Playwright setup
Build a lean Playwright + TypeScript framework. Delegate this to a subagent if you can. Load the references only now, since they are needed only for scaffolding.

1. **Install:** `npm init -y`, `npm install -D @playwright/test typescript @types/node dotenv`, `npx playwright install --with-deps chromium`.
2. **Configure:** add the `package.json` scripts, `tsconfig.json`, `playwright.config.ts`, `.gitignore` and `.env.example` from [references/scaffold-config.md](references/scaffold-config.md).
3. **Base fixture:** create `fixtures/base-test.ts` from [references/base-test-fixture.md](references/base-test-fixture.md). It records a HAR for every test and keeps it only when the test fails.
4. **Data factory:** create `data/user.factory.ts` from [references/data-factory.md](references/data-factory.md).
5. **Authentication, only if the app needs login:** add the setup project from [references/auth-setup.md](references/auth-setup.md). Read credentials from `.env` and never commit them.
6. **Directories:** `tests/`, `fixtures/`, `data/`, `exploration-knowledge/` (and `.auth/` when step 5 applies).
7. **Verify:** `npm run typecheck` must pass before you continue.

---

## Phase 1: Project memory (`project-details.md`)

1. If `project-details.md` exists in the project root, read it and adopt its base URL, diagnostics setup, fixtures, auth and data patterns.
2. If it does not exist, inspect the codebase (or the Phase 0 scaffold) and create it from [references/project-details-template.md](references/project-details-template.md). It covers: framework and versions, base configuration, failure diagnostics, exploration tooling, delegation strategy, directory layout, authentication and data strategy, and run commands.

---

## Phase 2: Exploration (`exploration-knowledge/`)

Do not re-explore what is already known.

1. For each flow in the spec, look for `exploration-knowledge/<flow-name>.md`.
2. **Notes exist:** reuse the routes, locators and data requirements they record. Skip exploration for that flow.
3. **No notes:** explore the flow. If your agent supports subagents, explore independent flows in parallel, one subagent per flow.
   - Run the probe script from [references/probe-script.md](references/probe-script.md) on each page of the flow. Pass the `.auth/` storage state for pages that need a signed-in user.
   - Prefer the locators the probe reveals, in the priority order of Standard 7.
   - Record the findings in `exploration-knowledge/<flow-name>.md` using [references/exploration-template.md](references/exploration-template.md).

---

## Phase 3: Implementation

1. Split the spec into flows and scenarios. With subagents, one subagent writes the data factories and API seeding helpers, and others write independent spec files (`tests/<feature>.spec.ts`) in parallel. Without subagents, do the same work in that order.
2. In every spec file:
   - Import `{ test, expect }` from `@fixtures/base-test` (the alias defined in `tsconfig.json`). In an existing project, use its own fixture import.
   - Keep simple actions flat. Use `test.step` only for multi-stage workflows (Standard 5).
   - Use the locators from `exploration-knowledge/` and web-first assertions only.
   - Build data with factories and seed state through the `request` API.
3. Run `npm run typecheck` (or the project's equivalent).

---

## Phase 4: Run and fix

1. Run the targeted spec: `npx playwright test <path-to-spec>`.
2. **On a pass:** report a short summary. Passing tests leave no artifacts behind.
3. **On a failure:** with several failures, delegate triage to subagents where available. Each failing test's folder in `test-results/` contains:
   - **Screenshot** (`test-failed-*.png`): the UI state at the moment of failure.
   - **Video** (`video.webm`): the sequence leading up to the failure.
   - **Trace** (`trace.zip`): DOM snapshots, actions and timings. Open it with `npx playwright show-trace <trace.zip>`, or unzip it and read the trace files directly when you cannot use a GUI.
   - **HAR** (`network.har`): requests, status codes and response bodies. It is JSON, so read or search it directly.
   - **Error context** (`error-context.md`): the error message, the failing locator and the test source, ready to read as text.
   - To see the page structure at the moment of failure, re-run the probe script on the failing URL.
4. Classify the cause: changed locator, timing or state problem, data collision, backend or API error, or a genuine application bug.
5. Fix test problems, and update `exploration-knowledge/<flow-name>.md` with corrected locators or behavior. Re-run until the test is stable.
6. Do not weaken an assertion to make a test pass. When the cause is a genuine application bug, stop changing the test and report the bug with its evidence.

---

## Phase 5: Sync project memory

When all tests are written and verified:
1. Collect the results of any subagents in the main agent.
2. Review new routes, factories, fixtures and config changes.
3. Update `project-details.md` to reflect the current state of the suite, so the next session starts from accurate information.
