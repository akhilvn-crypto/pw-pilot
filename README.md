# pw-pilot

An Agent Skill that turns a spec into a working Playwright + TypeScript test suite. It scaffolds the framework if there is none, explores the app headlessly, writes the tests, and fixes failures from their trace, screenshot, video and HAR.

Works in Claude Code, Codex and Antigravity CLI, and in any agent that reads `SKILL.md` skills.

## Install

Installs use the [`skills`](https://www.npmjs.com/package/skills) CLI through `npx`, so Node.js 18+ is the only requirement. A **project** install makes the skill available in the current repo only. A **global** install (`-g`) makes it available in every project. Start a new agent session after installing.

### Claude Code

```bash
# Project: installs into .claude/skills/
npx skills add akhilvn-crypto/pw-pilot --skill pw-pilot -a claude-code

# Global: installs into ~/.claude/skills/
npx skills add akhilvn-crypto/pw-pilot --skill pw-pilot -g -a claude-code
```

Check it with `/skills` inside Claude Code, or run it directly with `/pw-pilot`.

### Codex

```bash
# Project: installs into .agents/skills/
npx skills add akhilvn-crypto/pw-pilot --skill pw-pilot -a codex

# Global: installs into ~/.codex/skills/ (or $CODEX_HOME/skills)
npx skills add akhilvn-crypto/pw-pilot --skill pw-pilot -g -a codex
```

### Antigravity CLI

```bash
# Project: installs into .agents/skills/
npx skills add akhilvn-crypto/pw-pilot --skill pw-pilot -a antigravity-cli
```

For a global install, don't use `-g -a antigravity-cli`. That puts the skill in `~/.gemini/antigravity-cli/skills/`, but Antigravity CLI loads global skills only from `~/.gemini/config/skills/`. Clone the repo and link the skill folder into that directory instead:

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

To update, run `git pull` in the clone.

### Several agents at once

```bash
# Project
npx skills add akhilvn-crypto/pw-pilot --skill pw-pilot -a claude-code -a codex -a antigravity-cli

# Global (Antigravity CLI still needs the link step above)
npx skills add akhilvn-crypto/pw-pilot --skill pw-pilot -g -a claude-code -a codex
```

Update `npx skills` installs with `npx skills update`. Other agents work too: pass their name to `-a`, for example `-a gemini-cli` for Gemini CLI.

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
