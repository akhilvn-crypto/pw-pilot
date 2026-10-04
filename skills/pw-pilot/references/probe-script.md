# Headless probe script

The probe script is the main exploration tool. It opens a page in headless Chromium and prints:
- the page's accessibility snapshot, which lists roles and accessible names that map directly onto `getByRole(role, { name })`, `getByLabel` and `getByText`, and
- every element that has a `data-testid`, for `getByTestId`. Test ids do not appear in the snapshot.

It needs no GUI, so any agent can run it.

## Create `scripts/probe.mjs` in the project

```javascript
// Usage: node scripts/probe.mjs <url> [storageStatePath]
import { chromium } from '@playwright/test';

const [url, storageState] = process.argv.slice(2);
if (!url) {
  console.error('Usage: node scripts/probe.mjs <url> [storageStatePath]');
  process.exit(1);
}

const browser = await chromium.launch();
const context = await browser.newContext(storageState ? { storageState } : {});
const page = await context.newPage();
try {
  await page.goto(url);
  // Give client-rendered apps a moment to settle without hanging on pages that poll.
  await page.waitForLoadState('networkidle', { timeout: 5_000 }).catch(() => {});
  console.log(`# ${await page.title()}\nURL: ${page.url()}\n`);

  // Roles and accessible names map directly to getByRole(role, { name }).
  console.log('## Accessibility snapshot');
  console.log(await page.locator('body').ariaSnapshot());

  // Test ids do not appear in the snapshot; list them for getByTestId.
  const testIds = await page.locator('[data-testid]').evaluateAll((els) =>
    els.map((el) => `${el.tagName.toLowerCase()} data-testid="${el.getAttribute('data-testid')}"`),
  );
  if (testIds.length) console.log(`\n## Test ids\n${testIds.join('\n')}`);
} finally {
  await browser.close();
}
```

## Run it

```bash
node scripts/probe.mjs http://localhost:3000/login
node scripts/probe.mjs http://localhost:3000/account .auth/user.json   # signed in
```

## Sample output

```text
# Login
URL: http://localhost:3000/login

## Accessibility snapshot
- main:
  - heading "Sign in" [level=1]
  - textbox "Email"
  - textbox "Password"
  - button "Sign in"
  - link "Need help?":
    - /url: /help
  - checkbox "Remember me"
  - button

## Test ids
button data-testid="icon-only"
```

Translate the snapshot into locators:

| Snapshot line | Locator |
|---|---|
| `textbox "Email"` | `page.getByLabel('Email')`, or `page.getByRole('textbox', { name: 'Email' })` |
| `button "Sign in"` | `page.getByRole('button', { name: 'Sign in' })` |
| `link "Need help?"` | `page.getByRole('link', { name: 'Need help?' })` |
| `button` with no name | has no accessible name, so use its test id: `page.getByTestId('icon-only')` |

## Reaching states behind interactions

The script shows the page as it loads. To inspect a modal, a later wizard step or a validation message, copy the script and add the actions before the snapshot, for example:

```javascript
await page.getByRole('button', { name: 'Sign in' }).click();
await page.getByRole('alert').waitFor();
```

To inspect only part of the page, snapshot a narrower locator, for example `page.getByRole('dialog').ariaSnapshot()`.

Verify a candidate locator before writing it into a test:

```javascript
console.log(await page.getByRole('button', { name: 'Sign in' }).count()); // expect 1
```
