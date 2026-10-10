# globals, facades, and helper functions

dframework attaches core facades, response functions, and utilities to the global scope on startup. you do not need to import these in controllers, routes, views, middlewares, or jobs.

## global facades (never import)

these objects are always available on globalThis:

- `Route`: route registration facade (`Route.get`, `Route.post`, `Route.put`, `Route.delete`, `Route.group`, `Route.prefix`, `Route.middleware`, `Route.shield`, `Route.csrf`)
- `Socket`: websocket route registration facade (`Socket.on`, `Socket.group`, `Socket.middleware`, `Socket.broadcast`, `Socket.setState`, `Socket.to`, `Socket.toUser`, `Socket.toUserState`, `Socket.toUsers`, `Socket.toUsersState`, `Socket.toSession`, `Socket.toSessionState`, `Socket.guard`, `Socket.toGuardUser`, `Socket.toGuardUserState`, `Socket.toGuardUsers`, `Socket.toGuardUsersState`, `Socket.liveNotify`)
- `DB`: database query builder and connection facade (`DB.table`, `DB.query`, `DB.insert`, `DB.transaction`)
- `Auth`: user authentication facade (`Auth.check()`, `Auth.user()`, `Auth.id()`, `Auth.guard('admin')`, `Auth.login(user, data, perm)`, `Auth.logout()`, `Auth.flush()`)
- `Session`: session storage facade (`Session.get()`, `Session.set()`, `Session.has()`, `Session.flash()`, `Session.forget()`, `Session.permanent()`, `Session.id()`, `Session.regenerate()`)
- `Job`: background queue facade (`Job.dispatch('JobName', payload, { timeout, tries, backoff })`)
- `Log`: application logging facade (`Log.debug`, `Log.info`, `Log.warn`, `Log.error`, `Log.table`)
- `Config`: configuration facade (`Config.get('app.key', default)`, `Config.has('app.key')`, `Config.register(name, config)`)

```javascript
// good (in any controller, middleware, or job)
const user = Auth.user();
const admin = Auth.guard('admin').user();
const setting = Config.get('app.env');
Log.info('user action registered', { id: user?.id });
await Job.dispatch('SendEmailJob', { userId: 1 }, { timeout: 30000, tries: 3 });
```

## global response helpers (never import)

only call these from within request context (controllers, middlewares, socket handlers):

- `render(viewPath, data)`: compiles and sends `.d` view template
- `json(data)`: sends json response (`application/json; charset=utf-8`)
- `redirect(url, code = 302, force = false)`: navigates to url (spa aware, blocks cross origin)
- `back(fallback, code, force)`: redirects to previous url
- `abort(code, message)`: halts execution and sends http error (json for ajax, html for browser)
- `status(code)`: sets status code and returns chainable builder (`status(422).json({ error: 'invalid' })`)
- `cookie(name, value, options)`: sets response cookie
- `stream(writer)`: streams response chunks with backpressure handling (writer signature: `async (write, close) => void`)

## global url and routing helpers

- `route('name', params)`: generates url from named route with parameter substitution
- `routeIs('name', params)`: checks if active route matches name or pattern (supports wildcards like `'admin.*'`)
- `url('/path/:id', { id: 10 })`: builds path with parameter substitution
- `asset('css/app.css')`: returns root relative asset url
- `storage('avatars/user.jpg')`: returns web accessible storage url
- `old('fieldName', default)`: reads flashed input from previous request in templates

## global validation and sanitization helpers

- `validate(dataOrReq, rules, customMessages)`: validates input and returns result object with `.passes()`, `.fails()`, `.errors()`, `.first(field)`
- `sanitizeBody(data, fieldDefs)`: casts raw fields according to type definitions (`number`, `integer`, `boolean`, `array`, `json`)

## global translation and ruby helpers

- `t(req, 'file.key', params, fallback)`: async translation lookup from `lang/{locale}/{file}.json` with parameter interpolation and furigana parsing
- `ruby(base, reading)`: returns html ruby tag or parses inline bracket format (`'日本[にほん]語'`)

## global object and array utilities

- `get(obj, 'a.b.c', default)`: safe nested property read via dot notation
- `set(obj, 'a.b.c', value)`: safe nested property write (creates intermediate objects)
- `has(obj, 'a.b.c')`: checks existence of nested key
- `forget(obj, 'a.b.c')`: deletes nested property
- `pluck(arr, 'key')`: extracts property from array of objects into flat array
- `merge(...objects)`: deep recursive merge of objects and concatenation of arrays
- `arrayFirst(arr, predicate)`: returns first matching item or undefined
- `arrayLast(arr, predicate)`: returns last matching item or undefined
- `move(listOrStr, from, to)`: non mutating index move
- `remove(listOrStr, index)`: non mutating item removal
- `replace(listOrStr, index, val)`: non mutating replacement
- `limit(listOrStr, maxLen)`: slices list or string to max length
- `uniquify(listOrStr)`: removes duplicates

## global type and value checks

- `isEmpty(val)`: returns true for `undefined`, `null`, `''`, `[]`, `{}`
- `isString(val)`, `isNumber(val)`, `isInteger(val)`, `isBoolean(val)`, `isArray(val)`, `isObject(val)`, `isFunction(val)`
- `isUndefined(val)`, `isNull(val)`
- `random(min, max)`: inclusive random integer
- `now()`: current timestamp in iso format
- `escapeHtml(val)`: escapes html special characters
- `escapeJs(val)`: escapes javascript string and template literal special characters

## explicit imports from 'dframework'

only these classes and utilities need explicit import:

```javascript
import { Model } from 'dframework';          // for models/
import { Paginator } from 'dframework';      // for manual pagination construction
import { Storage } from 'dframework';        // for disk file operations
import { RateLimiter } from 'dframework';    // for rate limiting middleware
import { Hash } from 'dframework';           // for manual bcrypt verify or make
import { Locale } from 'dframework';         // for Locale.set / Locale.get
import { Inflector } from 'dframework';      // for pluralize, snakeCase, pascalCase, inferTableName
import { Env } from 'dframework';            // for config/*.js reading
import { SqlHelpers } from 'dframework';     // for raw SQL literals like NOW()
import { RequestContext } from 'dframework'; // for accessing req outside controllers
```