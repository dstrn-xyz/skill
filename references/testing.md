# testing and quality assurance reference

dframework provides a built in zero dependency testing framework built directly on node's native test runner (node:test). applications perform fast in process http requests without socket allocation overhead, assert database state, and simulate browser interactions with full form and navigation fidelity in continuous integration pipelines.

the `dframework/testing` module re-exports all test runner and assertion primitives (test, describe, it, before, after, beforeEach, afterEach, only, skip, todo, and strict assert). test files import everything directly from `dframework/testing` without requiring any direct node:test or node:assert imports.

## cli test runner

the runner is invoked via `dstrn test`:

```bash
# run all test files matching tests/**/*.test.js
dstrn test

# run specific file or directory
dstrn test tests/feature/UserTest.test.js
dstrn test tests/unit

# watch mode for re running on file modifications
dstrn test --watch

# run only tests marked with { only: true }
dstrn test --only

# output raw unformatted node tap or spec stream
dstrn test --raw
```

`dstrn test` automatically configures `APP_ENV=testing` and activates experimental sqlite flags on node 22 and 23. tests strictly require a `.env.testing` file in the project root; the framework does not fall back to `.env`, preventing accidental data mutations in your development or production databases.

## test generator

scaffold test stubs using `dstrn make:test`:

```bash
# creates tests/feature/UserTest.test.js
dstrn make:test UserTest

# creates tests/browser/LoginTest.test.js
dstrn make:test LoginTest --browser

# creates tests/unit/CalculatorTest.test.js
dstrn make:test CalculatorTest --unit

# nested directories are created automatically
dstrn make:test Auth/RegistrationTest
```

## in process http integration tests

the `testApp()` client dispatches synthetic requests directly through the application pipeline with zero socket binding overhead.

### request methods and session continuity

sequential requests within the same `testApp()` client share cookie jars and session state:

```javascript
import { describe, it, assert, testApp } from 'dframework/testing';

describe('user profile feature', () => {
  it('updates profile and preserves session cookies', async () => {
    const app = await testApp();

    const loginRes = await app.post('/login', {
      email: 'alex@example.com',
      password: 'secret'
    });
    loginRes.assertRedirect('/dashboard');

    const profileRes = await app.get('/profile');
    profileRes.assertOk();
    profileRes.assertSee('alex@example.com');
  });
});
```

supported methods include `app.get()`, `app.post()`, `app.put()`, `app.patch()`, `app.delete()`, and `app.head()`.

client configuration methods:
- `app.withHeaders(headers)`: attaches custom headers to requests
- `app.withCookies(cookies)`: attaches custom cookies to client jar
- `app.withoutCsrf()`: disables automated csrf token generation on mutating requests

### user authentication

use `actingAs` to authenticate a user before executing requests:

```javascript
import { describe, it, testApp } from 'dframework/testing';

describe('dashboard access', () => {
  it('allows access to authenticated user', async () => {
    const app = await testApp();
    app.actingAs({ id: 42, role: 'admin' });

    const res = await app.get('/admin/dashboard');
    res.assertOk();
  });
});
```

### response assertions

| assertion | description |
| --------- | ----------- |
| `assertStatus(code)` | assert response status code matches expected number |
| `assertOk()` | assert response status is 200 |
| `assertCreated()` | assert response status is 201 |
| `assertNotFound()` | assert response status is 404 |
| `assertForbidden()` | assert response status is 403 |
| `assertUnauthorized()` | assert response status is 401 |
| `assertRedirect(uri)` | assert status is redirect (301, 302, 303, 307, 308) and location matches |
| `assertJson(expected)` | assert response json contains expected object properties |
| `assertJsonMissing(key)` | assert response json does not contain specified key |
| `assertSee(text)` | assert response body string contains substring |
| `assertDontSee(text)` | assert response body string does not contain substring |
| `assertHeader(name, value)` | assert response header matches expected value |

response accessors:
- `res.status`: numeric status code
- `res.json()`: parsed json response body object
- `res.text()`: raw body string
- `res.headers`: response headers object

## database assertions

test app instances provide assertion helpers for database verification:

```javascript
import { describe, it, testApp } from 'dframework/testing';

describe('registration flow', () => {
  it('creates user in database', async () => {
    const app = await testApp();
    await app.migrate();
    await app.rollback();
    await app.migrate();

    await app.post('/register', {
      name: 'Sam',
      email: 'sam@example.com'
    });

    await app.assertDatabaseHas('users', { email: 'sam@example.com' });
    await app.assertDatabaseMissing('users', { email: 'ghost@example.com' });
    await app.assertDatabaseCount('users', 1);
  });
});
```

## browser and page interaction tests

the `browse()` client runs an in memory dom parser simulating browser rendering, form filling, dropdown selection, checkbox toggling, and button clicks without external browser binaries in ci. the simulated dom matches server rendered html via whitespace separated descendant selectors and `:not()` without running a client javascript runtime or complex combinators (such as `>`, `+`, or `~`):

```javascript
import { describe, it, browse } from 'dframework/testing';

describe('contact page interaction', () => {
  it('submits contact form and verifies thank you page', async () => {
    const page = await browse('/contact');

    page.assertSee('contact our team');
    page.assertElement('form');

    page.type('input[name="email"]', 'visitor@example.com');
    page.type('textarea[name="message"]', 'hello world');
    page.check('input[name="newsletter"]');
    page.select('select[name="topic"]', 'support');

    await page.click('button[type="submit"]');

    page.assertPath('/contact/success');
    page.assertSee('thank you for reaching out');
  });
});
```

### browser actions and assertions

actions:
- `page.type(selector, text)`: sets input or textarea value
- `page.check(selector)`: checks checkbox input
- `page.uncheck(selector)`: unchecks checkbox input
- `page.select(selector, value)`: selects option in select dropdown
- `await page.click(selector)`: clicks button (submitting containing form) or anchor link (navigating to target href)

assertions:
- `page.assertPath(expectedPath)`: asserts current pathname matches expected
- `page.assertSee(text)`: asserts rendered text contains substring
- `page.assertDontSee(text)`: asserts rendered text does not contain substring
- `page.assertElement(selector)`: asserts element matching css selector exists
- `page.assertMissing(selector)`: asserts element matching css selector does not exist
- `page.assertValue(selector, expected)`: asserts form control value matches expected
- `page.assertChecked(selector)`: asserts checkbox is checked
- `page.assertNotChecked(selector)`: asserts checkbox is unchecked

## request tracking and correlation

routes opt into tracking using `.track()`:

```javascript
Route.get('/api/orders', 'OrderController@index').track();

Route.track(() => {
  Route.get('/checkout', 'CheckoutController@show');
  Route.post('/checkout', 'CheckoutController@process');
});
```

when active, correlation tracking id is available on `req.trackingId`, forwarded via `X-Request-Id` response header, prepended to file logs (`[<requestId>]`), and attached to 500 error views and json error responses.
