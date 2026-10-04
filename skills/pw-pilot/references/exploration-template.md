# Exploration knowledge template

Write one file per flow or page: `exploration-knowledge/<flow-name>.md`. Later runs read these files instead of exploring again, so record only what is verified.

```markdown
# Exploration Knowledge: <Feature / Flow Name>

## Route & Entry Point
- URL / Path: `/checkout`
- Query Parameters: `?step=payment` (if applicable)

## Verified Stable Locators (captured with the probe script)
- Submit button: `page.getByRole('button', { name: 'Complete Order' })`
- Card number: `page.getByLabel('Card Number')`
- Status banner: `page.getByRole('alert')`

## Dynamic States & Asynchronous Behavior
- Loader: `page.getByRole('progressbar')` disappears after the cart API responds.
- Transitions: the payment modal animates in before its submit button is enabled; assert `toBeEnabled()` before clicking.

## Test Data Requirements
- Entity shape: user with a billing profile and one item in the cart.
- Uniqueness rules: email must be unique per registration; coupon codes cannot be reused.
- API seeding route: `POST /api/cart/items` with payload `{ itemId: 123, qty: 1 }`.

## Prerequisites
- Session: signed-in customer (`storageState: '.auth/customer.json'`).

## Last Verified
- <date>, against <base URL / environment>
```

Update the file whenever Phase 4 finds a changed locator or behavior.
