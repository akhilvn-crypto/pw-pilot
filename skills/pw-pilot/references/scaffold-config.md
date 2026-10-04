# Scaffold configuration

Load this file only when scaffolding a new framework (Phase 0, Case B).

## `package.json` scripts

Merge these into the `scripts` section:

```json
"scripts": {
  "test": "playwright test",
  "test:headed": "playwright test --headed",
  "test:ui": "playwright test --ui",
  "test:codegen": "playwright codegen",
  "test:report": "playwright show-report",
  "typecheck": "tsc --noEmit"
}
```

## `tsconfig.json`

Playwright runs TypeScript itself, so this file serves type checking and the import aliases. It uses `paths` without `baseUrl`, and `bundler` resolution, both of which current TypeScript versions require.

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "preserve",
    "moduleResolution": "bundler",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "noEmit": true,
    "types": ["node"],
    "paths": {
      "@fixtures/*": ["./fixtures/*"],
      "@data/*": ["./data/*"]
    }
  },
  "include": ["tests/**/*.ts", "fixtures/**/*.ts", "data/**/*.ts", "playwright.config.ts"]
}
```

Specs import with the aliases: `import { test, expect } from '@fixtures/base-test'` and `import { buildUserData } from '@data/user.factory'`. Playwright resolves `paths` at run time, so no extra tooling is needed.

## `playwright.config.ts`

```typescript
import { defineConfig, devices } from '@playwright/test';
import dotenv from 'dotenv';

dotenv.config({ quiet: true });

export default defineConfig({
  testDir: './tests',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  // One worker on CI suits small shared runners. Tests must still be parallel-safe:
  // raise this when the CI machine has spare cores.
  workers: process.env.CI ? 1 : undefined,
  reporter: [['html', { open: 'never' }], ['list']],
  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    // Failure diagnostics. The HAR is recorded by fixtures/base-test.ts.
    trace: 'retain-on-failure',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    actionTimeout: 10_000,
    navigationTimeout: 15_000,
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
  ],
});
```

When the app needs login, extend `projects` as shown in [auth-setup.md](auth-setup.md).

## `.gitignore`

```gitignore
node_modules/
test-results/
playwright-report/
blob-report/
playwright/.cache/
.auth/
.env
.env.local
```

## `.env.example`

Commit this file. Copy it to `.env` (ignored) and fill in real values.

```dotenv
BASE_URL=http://localhost:3000
# Only when the app needs login:
E2E_USER_EMAIL=
E2E_USER_PASSWORD=
```
