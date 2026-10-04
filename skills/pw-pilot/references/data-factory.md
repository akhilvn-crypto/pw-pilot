# Data factories

Each entity gets one small factory in `data/<entity>.factory.ts`: an interface, defaults that are unique on every call, and `Partial<T>` overrides.

## Baseline: `data/user.factory.ts`

```typescript
import { randomUUID } from 'node:crypto';

export interface UserData {
  email: string;
  name: string;
  role: 'admin' | 'user';
}

export function buildUserData(overrides: Partial<UserData> = {}): UserData {
  const id = randomUUID().slice(0, 8);
  return {
    email: `testuser-${id}@example.com`,
    name: `Test User ${id}`,
    role: 'user',
    ...overrides,
  };
}
```

## Rules

- **Unique by default.** Every call returns values no other test will use. A random suffix avoids the collisions that `Date.now()` causes when parallel workers build data in the same millisecond.
- **Override only what the test cares about.** `buildUserData({ role: 'admin' })` says what matters to the test. The rest stays default.
- **No shared fixtures in JSON.** Do not keep large static data files. A test that needs a specific value passes it as an override.
- **Respect app constraints.** If a field has validation rules (length, format, allowed domain), encode them in the factory and note them in `exploration-knowledge/`.
- **Seed through the API.** When a test needs the entity to exist on the server, build it with the factory and create it through Playwright's `request` context, as a fixture or at the start of the test. See [base-test-fixture.md](base-test-fixture.md).
