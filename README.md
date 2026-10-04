# pw-pilot

An Agent Skill that turns a spec into a working Playwright + TypeScript test suite. It scaffolds the framework if there is none, explores the app headlessly, writes the tests, and fixes failures from their trace, screenshot, video and HAR.

Works in Claude Code, Codex, Gemini CLI and Antigravity CLI, and in any agent that reads `SKILL.md` skills.

## Install

```bash
# Into the current project, for the agents you choose
npx skills add akhilvn-crypto/pw-pilot

# Globally, for specific agents
npx skills add akhilvn-crypto/pw-pilot --skill pw-pilot -g -a claude-code -a codex -a gemini-cli
```

Update later with `npx skills update`.

### Antigravity CLI

For a single project, install with `-a antigravity-cli`. The skill goes into `.agents/skills/`, which Antigravity CLI reads:

```bash
npx skills add akhilvn-crypto/pw-pilot --skill pw-pilot -a antigravity-cli
```

For every project, don't use `-g -a antigravity-cli`. That puts the skill in `~/.gemini/antigravity-cli/skills/`, but Antigravity CLI loads global skills only from `~/.gemini/config/skills/`. Clone the repo and link the skill folder into that directory instead:

```powershell
# Windows (PowerShell)
git clone https://github.com/akhilvn-crypto/pw-pilot.git
New-Item -ItemType Directory -Force "$HOME\.gemini\config\skills" | Out-Null
New-Item -ItemType Junction -Path "$HOME\.gemini\config\skills\pw-pilot" -Target "$PWD\pw-pilot\skills\pw-pilot"
```

```bash
# macOS / Linux
git clone https://github.com/akhilvn-crypto/pw-pilot.git
mkdir -p ~/.gemini/config/skills
ln -s "$PWD/pw-pilot/skills/pw-pilot" ~/.gemini/config/skills/pw-pilot
```

Start a new `agy` session afterwards. To update, run `git pull` in the clone.

## Use

Ask your agent in plain language:

> Use pw-pilot to automate the login and checkout flows in `specs/checkout.md`. The app runs at http://localhost:3000.

The spec can be a user story, acceptance criteria or a list of scenarios. If the base URL or the login details are missing, the agent asks for them.

## What it does

| Phase | |
|---|---|
| 0. Detect or scaffold | Uses an existing Playwright project, or creates a lean one: config, `tsconfig`, a HAR-recording base fixture, a data factory and an optional auth setup project. |
| 1. Project memory | Reads or creates `project-details.md`, the suite's notes between sessions. |
| 2. Explore | Runs a headless probe script that prints each page's roles and accessible names, and caches the findings in `exploration-knowledge/<flow>.md`. Later runs reuse them. |
| 3. Write | Writes specs and data factories, in parallel subagents where the agent supports them. |
| 4. Run and fix | Runs the specs and diagnoses failures from screenshot, video, trace and HAR. Fixes test problems and reports genuine app bugs. |
| 5. Sync | Updates `project-details.md` with what changed. |

## Rules the generated tests follow

- No wrappers around Playwright actions (`safeClick()` and the like).
- Fixtures over page-object inheritance. A flat page object only for UI used in 3+ specs.
- `test.step` only for multi-stage business workflows.
- Web-first assertions only. No `waitForTimeout`, no `expect(await x.isVisible())`.
- Locators in the order role, label, placeholder, text, test id. No XPath or CSS chains.
- Independent, parallel-safe tests with factory-built unique data, seeded through the API.

## Layout

```
skills/pw-pilot/
├── SKILL.md                      rules and the phase workflow
└── references/                   loaded only when needed
    ├── scaffold-config.md        package.json scripts, tsconfig, playwright.config.ts, .gitignore
    ├── base-test-fixture.md      HAR-recording fixture
    ├── data-factory.md
    ├── auth-setup.md             setup project and storage state
    ├── probe-script.md           headless exploration script
    ├── exploration-template.md
    └── project-details-template.md
```

## License

MIT
