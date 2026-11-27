# testing and quality assurance reference

dframework provides a built in zero dependency testing framework built directly on node's native test runner (node:test). applications perform fast in process http requests without socket allocation overhead, assert database state, and simulate browser interactions with full form and navigation fidelity in continuous integration pipelines.

the `dframework/testing` module re-exports all test runner and assertion primitives (`test`, `describe`, `it`, `before`, `after`, `beforeEach`, `afterEach`, `only`, `skip`, `todo`, and strict `assert`). in addition, it provides the unified `Tester` facade (`Tester.get`, `Tester.post`, `Tester.put`, `Tester.patch`, `Tester.delete`, `Tester.route`, `Tester.actingAs`, `Tester.fake`, `Tester.assertDispatched`, `Tester.assertDatabaseHas`) with zero wrapper boilerplate. like other framework facades, `Tester` is also available globally on globalThis in the test environment. test files import everything directly from `dframework/testing` without requiring any direct node:test or node:assert imports.

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

the `Tester` facade dispatches synthetic requests directly through the application pipeline with zero socket binding overhead.

### request methods and session continuity

sequential requests share cookie jars and session state through the `Tester` facade:

```javascript
import { test, Tester } from 'dframework/testing';

test('updates profile and preserves session cookies', async () => {
  const loginRes = await Tester.post('/login', {
    email: 'alex@example.com',
    password: 'secret'
  });
  loginRes.assertRedirect('/dashboard');

  const profileRes = await Tester.get('/profile');
  profileRes.assertOk();
  profileRes.assertSee('alex@example.com');
});
```

supported methods include `Tester.get()`, `Tester.post()`, `Tester.put()`, `Tester.patch()`, `Tester.delete()`, `Tester.head()`, and `Tester.options()`.

configuration methods:
- `Tester.actingAs(user)`: authenticates user for subsequent requests
- `Tester.withHeaders(headers)`: attaches custom headers to requests
- `Tester.withCookies(cookies)`: attaches custom cookies to client jar
- `Tester.withoutCsrf()`: disables automated csrf token generation on mutating requests

### user authentication

use `actingAs` to authenticate a user before executing requests:

```javascript
import { test, Tester } from 'dframework/testing';

test('allows access to authenticated user', async () => {
  Tester.actingAs({ id: 42, role: 'admin' });

  const res = await Tester.get('/admin/dashboard');
  res.assertOk();
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

## service fakes

dframework provides test doubles and spies for background jobs, websocket events, queues, and mail via `fake({ jobs: true, socket: true, mail: true })`, `fakeJobs()` / `fakeSocket()` / `fakeMail()`, or passing service names like `fake('jobs')` or `fake(['jobs', 'mail'])`. assertions are verified using top-level helpers or client methods, never called on global facades.

### faking background jobs

calling `Tester.fake({ jobs: true })` (or `Tester.fakeJobs()`) intercepts dispatches and records jobs in memory:

```javascript
import { test, Tester } from 'dframework/testing';

test('dispatches confirmation email on checkout', async () => {
  Tester.fake({ jobs: true });

  const res = await Tester.post('/checkout', { cartId: 10 });
  res.assertOk();

  Tester.assertDispatched('SendOrderConfirmationJob');
  Tester.assertDispatched('SendOrderConfirmationJob', (payload) => payload.cartId === 10);
  Tester.assertDispatchedTimes('SendOrderConfirmationJob', 1);
  Tester.assertNotDispatched('CancelOrderJob');
});
```

job assertions:
- `Tester.assertDispatched(job, callback)`: assert job was dispatched, optionally matching predicate
- `Tester.assertNotDispatched(job, callback)`: assert job was not dispatched
- `Tester.assertDispatchedTimes(job, count)`: assert exact dispatch count
- `Tester.assertNothingDispatched()`: assert no jobs were dispatched
- `Tester.dispatched(job, callback)`: retrieve dispatched jobs matching filter

### faking websocket events

calling `Tester.fake({ socket: true })` (or `Tester.fakeSocket()`) intercepts websocket events without requiring a live socket server:

```javascript
import { test, Tester } from 'dframework/testing';

test('broadcasts message and notifies recipient', async () => {
  Tester.fake({ socket: true });

  await Tester.post('/messages', { toUser: 42, text: 'hello' });

  Tester.assertBroadcast('message.sent');
  Tester.assertSentToUser(42, 'notification.new');
  Tester.assertNotSentToUser(99, 'notification.new');
});
```

socket assertions:
- `Tester.assertBroadcast(event, callback)`: assert event was broadcast to all clients
- `Tester.assertNotBroadcast(event, callback)`: assert event was not broadcast
- `Tester.assertSentToUser(userOrId, event, callback)`: assert event was sent to specific user id
- `Tester.assertNotSentToUser(userOrId, event, callback)`: assert event was not sent to user
- `Tester.assertSentToSession(sessionId, event, callback)`: assert event was sent to specific session
- `Tester.assertSent(event, callback)`: assert event was sent across any channel
- `Tester.assertNotSent(event, callback)`: assert event was not sent across any channel
- `Tester.assertSentTimes(event, count)`: assert event was sent an exact number of times
- `Tester.assertNothingSent()`: assert no socket events were emitted
- `Tester.assertRevoked(userOrId, guardName)`: assert user websocket session was revoked
- `Tester.sent(event, callback)`: retrieve sent socket events matching filter
- `Tester.broadcasted(event, callback)`: retrieve broadcasted socket events matching filter

### faking mail

calling `Tester.fake({ mail: true })` (or `Tester.fakeMail()`) intercepts mail delivery:

```javascript
import { test, Tester } from 'dframework/testing';

test('sends welcome email on registration', async () => {
  Tester.fakeMail();

  await Tester.post('/register', { email: 'user@example.com' });

  Tester.assertSentMail('emails.welcome');
  Tester.assertSentMailCount(1);
});
```

mail assertions:
- `Tester.assertSentMail(criteria)`: assert email was sent matching template name or predicate callback receiving `{ to, cc, bcc, from, subject, view, template, data, replyTo, html, text }`
- `Tester.assertQueuedMail(criteria)`: assert email was queued matching template name or predicate callback
- `Tester.assertNothingMailSent()`: assert no email was sent or queued
- `Tester.assertSentMailCount(count)`: assert exact number of sent emails

## database assertions and transactions

wrap tests in automatic database rollbacks using `Tester.useDatabaseTransactions()` (or disable with `Tester.withoutDatabaseTransactions()`) or per callback with `Tester.useTransaction()`. tests execute inside an isolated transaction that rolls back upon test completion:

```javascript
import { test, Tester } from 'dframework/testing';

Tester.useDatabaseTransactions();

test('creates user in database', async () => {
  await Tester.post('/register', {
    name: 'sam',
    email: 'sam@example.com'
  });

  await Tester.assertDatabaseHas('users', { email: 'sam@example.com' });
  await Tester.assertDatabaseMissing('users', { email: 'ghost@example.com' });
  await Tester.assertDatabaseCount('users', 1);
});
```

## browser and page interaction tests

the `Tester.browse()` (or exported `browse()`) client runs an in memory dom parser simulating browser rendering, form filling, dropdown selection, checkbox toggling, and button clicks without external browser binaries in ci. the simulated dom matches server rendered html via whitespace separated descendant selectors and `:not()` without running a client javascript runtime or complex combinators (such as `>`, `+`, or `~`):

```javascript
import { describe, it, Tester } from 'dframework/testing';

describe('contact page interaction', () => {
  it('submits contact form and verifies thank you page', async () => {
    const page = await Tester.browse('/contact');

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

when active, correlation tracking id is available on `req.trackingId`, forwarded via `X-Request-Id` response header, prepended to file logs (`[<requestId>]`), and attached to 500 error views and json error responses. request state can be recorded at any point using `Tracker.capture()`, and exported asynchronously using registered sinks configured via `Tracker.sink()`.
