# Authentication setup

Use this only when the spec covers pages that need a signed-in user. Sign in once in a setup project, save the browser storage state to `.auth/`, and have the test projects start from it.

If the base URL, credentials or login flow are unknown, ask the user before writing this. Credentials go in `.env` (ignored by git), never in code.

## `tests/auth.setup.ts`

```typescript
import { test as setup, expect } from '@playwright/test';

const authFile = '.auth/user.json';

setup('sign in', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('Email').fill(process.env.E2E_USER_EMAIL!);
  await page.getByLabel('Password').fill(process.env.E2E_USER_PASSWORD!);
  await page.getByRole('button', { name: 'Sign in' }).click();
  await expect(page.getByRole('heading', { name: 'Dashboard' })).toBeVisible();
  await page.context().storageState({ path: authFile });
});
```

Replace the route, labels and post-login check with the real ones found by the probe script.

## `projects` in `playwright.config.ts`

```typescript
projects: [
  { name: 'setup', testMatch: /.*\.setup\.ts/ },
  {
    name: 'chromium',
    use: { ...devices['Desktop Chrome'], storageState: '.auth/user.json' },
    dependencies: ['setup'],
  },
],
```

## Tests that must start signed out

Override the storage state in the spec file:

```typescript
test.use({ storageState: { cookies: [], origins: [] } });
```

## Several roles

Add one setup test and one state file per role (`.auth/admin.json`, `.auth/customer.json`). Select a role per spec file with `test.use({ storageState: '.auth/admin.json' })`. Record which state each flow needs under "Prerequisites" in `exploration-knowledge/<flow-name>.md`.
