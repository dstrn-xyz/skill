# controllers, routing, and responses

controllers and route definitions in dframework follow laravel style conventions rather than express patterns.

## controller design

controllers live in `controllers/` and can be organized in subdirectories. each file exports a default class with async methods.

```javascript
// controllers/app/UserController.js
export default class UserController {
  async index(req) {
    const users = await User.paginate(20);
    return render('app.users.index', { users });
  }

  async store(req) {
    const v = validate(req.body, {
      email: 'required|email',
      name: 'required|string|min:2'
    });
    if (v.fails()) {
      return status(422).json({ errors: v.errors() });
    }

    const user = await User.create(req.body);
    return status(201).json({ user });
  }

  async show(req) {
    const user = await User.find(req.params.id);
    if (!user) return abort(404, 'user not found');
    return render('app.users.show', { user });
  }
}
```

## response helpers mandate

in dframework, you must return global response helpers from your controller methods.

never use express methods on `res`:

```javascript
// forbidden (express style)
async badHandler(req, res) {
  res.status(200).json({ ok: true });
  res.send('hello');
  res.redirect('/home');
}

// required (dframework style)
async goodHandler(req) {
  return json({ ok: true });
  return render('home');
  return redirect('/home');
  return back();
  return abort(403, 'unauthorized');
  return status(422).json({ error: 'invalid payload' });
  return stream(async (write, close) => { await write(chunk); close(); });
}
```

### response helpers reference

- `return render('view.name', data)`: compiles `.d` template, injects data, returns html
- `return json(data)`: serializes object, sets application/json content type
- `return redirect(url, code = 302, force = false)`: handles navigation (spa aware)
- `return back(fallback)`: redirects to previous url based on session referrer
- `return abort(code, message)`: returns structured json error for ajax or error page for browser
- `return status(code).json(data)`: sets http status and returns json
- `return status(code).set('Header', 'val').send(content)`: custom headers and raw body
- `return back().withErrors(v.errors()).withInput(req.body)`: flashes validation errors to view
- `return stream(async (write, close) => { await write(chunk); close(); })`: streams response chunks with automatic backpressure support; `write(chunk)` awaits transport readiness before resolving and `close()` releases the pool

### streaming responses

ideal for proxying large files (audio, video, documents) without buffering the entire payload into memory:

```javascript
// StreamController.js
export default class StreamController {
  async audio(req) {
    const file = await AudioFile.find(req.params.id);
    if (!file) return abort(404);

    const upstream = await fetch(file.url, {
      headers: { range: req.headers['range'] }
    });

    return status(upstream.status)
      .set('accept-ranges', 'bytes')
      .set('content-range', upstream.headers.get('content-range'))
      .type('audio/mpeg')
      .stream(async (write, close) => {
        const reader = upstream.body.getReader();
        while (true) {
          const { done, value } = await reader.read();
          if (done) break;
          await write(Buffer.from(value));
        }
        close();
      });
  }
}
```

- if the writer function completes without calling `close()`, the framework automatically ends the response and releases the connection pool
- if the writer throws an error, the response terminates and the pool is released immediately

## request object (req)

the request object is injected into controller and middleware methods:

- `req.body`: parsed json or form fields on mutating requests (post, put, patch, delete)
- `req.query`: parsed query parameters object
- `req.params`: route url parameters (`:id` in route paths)
- `req.input('key', fallback)`: unified input resolver (priority: params > body > query)
- `req.all()`: all unified inputs merged as an object (params > body > query)
- `req.files`: uploaded multipart files (`req.files.fieldName[0]`)
- `req.file('key')`: returns first uploaded file for field name or null
- `req.hasFile('key')`: true if file was uploaded for field name
- `req.cookies`: parsed cookies object
- `req.page`: integer from `?page=` query parameter (defaults to 1)
- `req.isAjax`: true for xhr, fetch, or json accept header
- `req.isSPA`: true for dSPA router requests
- `req.isNative`: true for native mobile or desktop webview requests
- `req.header('name')`: case insensitive header reader
- `req.locale`: active detected locale code

## middleware registration syntax (no short aliases)

middleware classes live in `middlewares/`. each method acts as an independent handler:

```javascript
// middlewares/AuthMiddleware.js
export default class AuthMiddleware {
  async requireAuth(req, next) {
    if (!Auth.check()) {
      return redirect('/login', true);
    }
    const devices = await Device.where({ user_id: Auth.user().id, revoked: false }).get();
    return next({ devices });
  }

  async requireGuest(req, next) {
    if (Auth.check()) {
      return redirect('/dashboard', true);
    }
    return next();
  }
}
```

pass data to downstream controllers and views using `return next({ key: value })` or `return next().with({ key: value })`. any data passed is automatically merged onto `req` and injected into view template scopes.

### strictly resolved like controllers

middleware is resolved dynamically from `middlewares/` using `ClassName@method` (or `subdir.ClassName@method`). there is no alias registry:

```javascript
// forbidden (short alias string does not exist and will crash)
Route.get('/app', 'AppController@index').middleware('auth');
Route.group({ middleware: ['auth'] }, (r) => {});

// required (explicit ClassName@method syntax)
Route.middleware('AuthMiddleware@requireAuth').get('/app', 'AppController@index');
Route.group({ middleware: ['AuthMiddleware@requireAuth'] }, (r) => {
  r.get('/dashboard', 'DashboardController@index');
});

// inline array syntax (all entries except last are middleware)
Route.get('/profile', ['AuthMiddleware@requireAuth', 'UserProfileController@show']);

// instantiated middleware objects (like RateLimiter: windowMs, max, message, keyGenerator, trustedProxies, maxEntries)
import { RateLimiter } from 'dframework';
const limiter = new RateLimiter({ windowMs: 60000, max: 10, maxEntries: 100000 });
Route.middleware(limiter).post('/login', 'AuthController@login');
```

## routing system

all files in `routes/*.js` are auto loaded on boot. do not import route files manually.

```javascript
// routes/web.js
Route.get('/', 'HomeController@index').name('home');
Route.get('/users', 'app.UserController@index').name('users.index');
Route.get('/users/:id', 'app.UserController@show').name('users.show');
Route.post('/users', 'app.UserController@store').name('users.store');
Route.put('/users/:id', 'app.UserController@update').name('users.update');
Route.delete('/users/:id', 'app.UserController@destroy').name('users.destroy');

// route groups with prefix, middleware, domain, basic auth, shield, or csrf
Route.group({ prefix: '/admin', middleware: ['AuthMiddleware@requireAuth', 'admin.AdminMiddleware@requireAdmin'] }, (admin) => {
  admin.get('/dashboard', 'admin.DashboardController@index').name('admin.dashboard');
  admin.post('/settings', 'admin.SettingsController@save').name('admin.settings');
});

// group level shield and csrf deactivation
Route.group({ prefix: '/webhooks', shield: false, csrf: false }, (hooks) => {
  hooks.post('/stripe', 'StripeController@handle');
  hooks.post('/github', 'GitHubController@handle');
});

// standalone shield and csrf group wrappers
Route.shield(false, (r) => {
  r.post('/api/external', 'ExternalController@handle');
});
Route.csrf(false, (r) => {
  r.post('/api/webhook', 'WebhookController@handle');
});

// fluent chainable builders
Route.prefix('/webhooks')
  .shield(false)
  .csrf(false)
  .group((hooks) => {
    hooks.post('/stripe', 'StripeController@handle');
  });

// domain and port binding
Route.domain('api.example.com', (api) => {
  api.get('/health', () => json({ status: 'healthy' }));
});

// basic auth protection (chainable builder, single route modifier, or custom validator)
Route.basicAuth('admin', 'secret').get('/metrics', 'MetricsController@export');
Route.get('/secret', 'app.SecretController@show').basicAuth('admin', 'secret');
Route.basicAuth(async (user, pass) => user === 'admin' && pass === 'secret').get('/admin', 'AdminController@index');

// parameter constraints with regex patterns
Route.get('/users/:id', 'UserController@show').where('id', '[0-9]+');
Route.get('/posts/:year/:slug', 'PostController@show').where({
  year: '^[0-9]{4}$',
  slug: '^[a-z0-9-]+$'
});

// individual route modifier overrides
Route.post('/webhooks/stripe', 'WebhookController@handle')
  .csrf(false)
  .shield(false);
```

### route naming and matching

- always name routes with `.name('unique.name')`
- generate urls in controllers and views with `route('users.show', { id: user.id })`
- check active route in views or controllers with `routeIs('users.*')`

## request input retrieval

use `req.input()` or `req.all()` to access parameters, body data, and query strings with unified priority (params > body > query):

```javascript
// single input with optional fallback
const id = req.input('id');
const search = req.input('q', '');

// all merged inputs
const allInputs = req.all(); // or req.input()
```

## input validation

use the global `validate()` helper. rule chains are separated by pipes (`|`).

```javascript
const v = validate(req.body, {
  name: 'required|string|min:3|max:50',
  email: 'required|email',
  role: 'required|in:admin,editor,viewer',
  age: 'nullable|integer|min:18'
});

if (v.fails()) {
  // for api response
  return status(422).json({ errors: v.errors() });

  // for html forms
  return back().withErrors(v.errors()).withInput(req.body);
}
```

### file validation

pass `req` instead of `req.body` to validate uploaded files:

```javascript
const v = validate(req, {
  avatar: 'file_required|file_size:4mb|file_mimes:jpg,png,webp'
});
```