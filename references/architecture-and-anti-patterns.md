# architecture and anti patterns

dframework enforces clear architectural boundaries and predictable conventions.

## project directory layout

the root folder structure is fixed and discovered automatically:

- `config/`: configuration modules (`app.js`, `auth.js`, `database.js`, `session.js`, `storage.js`, `native.js`)
- `console/`: background scheduler (`Schedule.js`) and command classes (`commands/`)
- `controllers/`: request handlers (subdirectories map via dot notation)
- `database/`: migrations (`database/migrations/`) and seeders (`database/seeders/`)
- `jobs/`: background worker thread jobs
- `lang/`: localization json files (`lang/{locale}/{file}.json`)
- `middlewares/`: request filter classes
- `models/`: active record ORM models
- `native/`: native plugins (`native/plugins/`) and assets (`native/assets/`)
- `public/`: static files
- `routes/`: route definition files (auto loaded: `web.js`, `wire.js`)
- `storage/`: uploads (`storage/public/`), private files (`storage/local/`), framework cache (`storage/framework/`)
- `views/`: server side `.d` templates

## architectural boundaries

enforce these separation of concerns rules (checked by `dstrn lint` and `dstrn doctor`):

- controllers must never call or import other controllers
- models must never import or depend on controllers
- middlewares must never depend on controllers
- routes should reference controller strings rather than instantiating models directly
- user projects must never modify framework core source files (`dframework/core/` or `node_modules/dframework/`)
- user projects must never run internal framework test suites (`npm test` in `dframework/`)

## strict anti patterns

avoid these common mistakes:

| bad pattern                                                                     | good practice                                                                               | reason                                                                                                                                                          |
| :------------------------------------------------------------------------------ | :------------------------------------------------------------------------------------------ | :-------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `res.json({ ok: true })`                                                        | `return json({ ok: true })`                                                                 | dframework controllers return response helper objects                                                                                                           |
| `res.status(404).send('err')`                                                   | `return abort(404, 'err')`                                                                  | dframework manages response pipeline automatically                                                                                                              |
| `json({ ok: true })` (missing return)                                           | `return json({ ok: true })`                                                                 | omitting return causes request timeouts                                                                                                                         |
| modifying framework core source code to fix project issues                      | resolve within application files                                                            | framework core is immutable and published independently                                                                                                         |
| writing custom one-off test scripts to query models or database                 | run `dstrn tinker "await Model.method()"` or `dstrn tinker --execute="await Model.count()"` | all models and facades are automatically loaded with top level await support                                                                                    |
| inspecting framework source internals instead of docs                           | refer to SKILL and documentation                                                            | the SKILL provides full specifications and contracts                                                                                                            |
| running dframework internal test suite on user projects                         | run project specific tests if authored                                                      | framework tests in `dframework/` are completely unrelated                                                                                                       |
| manually checking translation keys across files                                 | run `dstrn lang:lint`                                                                       | framework cli provides automated parity and view key linting                                                                                                    |
| directly editing `js/dstrn.js` or `css/dstrn.css`                               | author in `public/js/components/` or `public/css/`                                          | `dstrn.js` and `dstrn.css` are regenerated on start; direct edits are wiped                                                                                     |
| searching for build, rebuild, or compile commands                               | run `dstrn serve`; compilation is automatic                                                 | dframework has no manual frontend build step or `npm run build` command                                                                                         |
| writing custom view compilation scripts or altering view compiler               | let `ViewEngine` compile views automatically                                                | view compilation is 100% automated in memory at runtime                                                                                                         |
| manually adding `<link href="/css/dstrn.css">` or `<script src="/js/dstrn.js">` | omit them completely                                                                        | framework automatically injects runtime CSS and JS; manual inclusion crashes JS                                                                                 |
| assuming ruby or plain text for Japanese                                        | ask the user before writing Japanese                                                        | user must decide whether Japanese requires furigana `@ruby` tags or plain text                                                                                  |
| `.middleware('auth')` or `middleware: ['auth']`                                 | `.middleware('AuthMiddleware@requireAuth')`                                                 | middleware is resolved from `middlewares/` via `ClassName@method`; aliases do not exist                                                                         |
| `import { DB, Route, Auth } from 'dframework'`                                  | use globals directly without import                                                         | facades and helpers are registered globally on boot                                                                                                             |
| accessing or mutating req for auth                                              | `Auth.user()` or `Auth.guard('admin').user()`                                               | authentication is handled cleanly via global Auth facade without touching req                                                                                   |
| `cookie('locale', 'ja')`                                                        | `Locale.set('ja')`                                                                          | `Locale` facade sets cookie, session, and validates format                                                                                                      |
| flat, single tone background without depth                                      | 3-tier elevation (`bg-container-d`, `bg-container`, `bg-container-l`)                       | established dframework UI hierarchy and depth standard                                                                                                          |
| complex inline ternary styling in HTML                                          | localized `@js` compute blocks                                                              | keeps view templates clean, readable, and declarative                                                                                                           |
| authoring `dComponent` for static markup                                        | use `@include('partial')`                                                                   | `dComponent` is reserved for canvas, audio loops, or heavy JS                                                                                                   |
| passing data to `@include('partial', { key: val })`                             | use `@include('partial')` directly                                                          | `@include` takes only the template name and inherits all parent view scope variables automatically                                                              |
| `document.getElementById('btn')`                                                | `select('#btn')` or `querySelector('#btn')`                                                 | clean code DOM querying mandate                                                                                                                                 |
| `window.alert('error')`                                                         | `notify('error')` or `modal('error')`                                                       | no native alerts in refined production apps                                                                                                                     |
| `DB.query(\`SELECT * FROM u WHERE id = ${id}\`)`                                | `DB.query('SELECT * FROM u WHERE id = ?', [id])`                                            | always use parameterized queries to prevent SQL injection                                                                                                       |
| using raw `<select>` or custom dropdown js                                      | `<d-combobox>` or `<d-dropdown>`                                                            | dframework provides optimized built in web components                                                                                                           |
| using raw `DB.table()` or `DB.query()` for entity data                          | use `Model` subclasses (e.g. `User.find(id)`, `User.with('posts').get()`)                   | models provide eager loading to prevent n+1 queries, automatic hidden attribute filtering on JSON serialization, and full query builder parity                  |
| manual `fs.writeFile` or custom path resolution for uploads                     | use `Storage.disk().put()` and `Storage.disk().get()`                                       | `Storage` manages path traversal protection, disk space safety buffers, automatic directory creation, multipart uploads, and transparent AES-256-GCM encryption |
| checking or accessing private `this._relations` directly                        | `await this.relation()` or `await Promise.all([this.a(), this.b()])`                        | eager loaded relationships are cached in memory automatically and resolve in zero queries                                                                       |
| looping over models and calling relations without `with()`                      | `Model.with('relation').get()` or `await model.load('relation')`                            | eager loading prevents n+1 queries with single batch lookup                                                                                                     |
| ad hoc JS toggles or drawer hacks for mobile navigation                         | `<d-hamburger target="#nav-links" breakpoint="md">`                                         | `<d-hamburger>` manages media queries, target DOM cloning, nested subpanels, scroll locking, and SPA auto closing automatically                                 |
| using tailwind classes (e.g. `bg-blue-500`, `items-center`)                     | use `dstrn.css` classes (e.g. `.bg-blue`, `.align-center`)                                  | dframework has its own custom compiled CSS utility set                                                                                                          |
| creating unhandled promise catch blocks                                         | let errors propagate or handle specific exceptions                                          | fail loudly when failure matters                                                                                                                                |