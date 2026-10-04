# Base test fixture with HAR recording

Create `fixtures/base-test.ts`. Every spec imports `test` and `expect` from here instead of from `@playwright/test`.

```typescript
import { test as base, expect } from '@playwright/test';
import fs from 'fs';

export const test = base.extend({
  // Record a HAR for every test. The browser context depends on contextOptions, so this
  // teardown runs after the context has closed and the HAR has been written to disk.
  contextOptions: async ({ contextOptions }, use, testInfo) => {
    const harPath = testInfo.outputPath('network.har');
    await use({ ...contextOptions, recordHar: { path: harPath, content: 'embed' } });
    if (testInfo.status === testInfo.expectedStatus) fs.rmSync(harPath, { force: true });
  },
});

export { expect };
```

## How it behaves

- The HAR is written to the test's own output folder, `test-results/<test-folder>/network.har`, next to the trace, video and screenshot.
- When a test ends as expected, the HAR is deleted. When it fails, the HAR is kept.
- `content: 'embed'` stores response bodies inside the HAR, so a single JSON file holds the whole network log.

## Why not `page.routeFromHAR(..., { update: true })`

A `page` fixture that records with `routeFromHAR` and deletes the file in its own teardown does not work. Playwright writes that HAR only when the browser context closes, and the context closes after the `page` fixture's teardown. The delete runs before the file exists, so HAR files from passing tests pile up. Overriding `contextOptions` avoids the problem, because Playwright tears it down only after the context that depends on it has closed.

## Adding fixtures

Add further fixtures to the same `base.extend` call, for example a pre-seeded user:

```typescript
import { test as base, expect, type APIRequestContext } from '@playwright/test';
import { buildUserData, type UserData } from '@data/user.factory';

async function createUser(request: APIRequestContext): Promise<UserData> {
  const user = buildUserData();
  const response = await request.post('/api/users', { data: user });
  expect(response.ok()).toBeTruthy();
  return user;
}

export const test = base.extend<{ seededUser: UserData }>({
  // contextOptions override from above goes here as well
  seededUser: async ({ request }, use) => {
    await use(await createUser(request));
  },
});
```

Replace `/api/users` with the app's real endpoint, as recorded in `exploration-knowledge/`.
