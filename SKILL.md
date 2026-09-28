---
name: dframework
description: >-
  comprehensive guide and rules for writing dframework applications, controllers, routing, views, models, migrations, reactivity, frontend, jobs, and native plugins. Use whenever developing, reviewing, refactoring, or generating code in dframework projects.
---
# dframework guidelines

core engineering principles and architectural standards for dframework projects.

## core mandate

dframework is not a traditional express or node framework. it uses a laravel inspired architecture with global facades, active record models, compiled views, and return based response helpers.

every agent working on dframework projects must follow these fundamental rules:

1. never use `res.json()`, `res.send()`, `res.status()`, or `res.redirect()`; controllers must return global response helpers (`return json()`, `return render()`, `return redirect()`, `return back()`, `return abort()`, `return status()`, `return stream()`)
2. never import global facades or response helpers (`Route`, `Socket`, `DB`, `Auth`, `Session`, `Job`, `Log`, `Config`, `render`, `json`, `redirect`, `back`, `abort`, `status`, `stream`, `validate`, `sanitizeBody`, `t`, `ruby`, `escapeHtml`, `escapeJs`); they are attached to the global scope on startup
3. only import specific framework components when required (`Model`, `Paginator`, `Storage`, `RateLimiter`, `Hash`, `Locale`, `Inflector`, `Env`, `SqlHelpers` from `'dframework'`)
4. always prioritize active record `Model` subclasses (`User.find(id)`, `User.with('profile').where('status', 'active').get()`, `static casts = { metadata: 'json' }`, `static fillable = ['name', 'email']`) over raw `DB.table()` or `DB.query()` for entity data access, relationships, and mutations; define `static fillable` or `static guarded` on models to protect against mass assignment vulnerabilities; all database queries must use parameterized bindings (`?` placeholders) and never interpolate raw variables into sql strings
5. when consuming model relationships, always use standard async method calls (`await model.relation()`), concurrent resolvers (`await Promise.all([this.album(), this.genre()])`), or direct property access (`model.relation`); preloaded relations via `.with()` or `.load()` are cached in memory and resolve in zero database queries; never access or inspect private `_relations` directly
6. verify frontend utility classes exist in `dstrn.css` before using them; never assume tailwind or bootstrap classes exist
7. use `select()`, `selectAll()`, or `querySelector` and `querySelectorAll` for DOM manipulation
8. always prioritize and use built in frontend components (`d-text-input`, `d-combobox`, `d-dropdown`, `d-drawer`, `d-hamburger`, `d-hold-button`, `d-image-input`, `d-file-input`, `d-modal`, `d-notification`, `d-skeleton`, `d-toggle`, `d-slider`, `d-color-picker`, `d-morph`, `d-context-menu`, `d-checkbox`, `d-icon-button`, `d-loader`); for responsive mobile navigation menus, always use `<d-hamburger target="#nav-links" breakpoint="md">` instead of writing custom JavaScript menu toggles
9. never manually include `<link href="/css/dstrn.css">` or `<script src="/js/dstrn.js">` in `<head>`; the framework automatically injects them, and manual inclusion will crash runtime JavaScript
10. adhere to the 3-tier surface elevation system (`bg-container-d`, `bg-container`, `bg-container-l`), text contrast hierarchy (`text-content`, `text-content-l`), and micro-gap spacing; place localized frontend compute in view `@js` blocks near usage
11. use `@include('partial.name')` for pure UI components without passing data arguments; partials inherit parent view scope automatically. Reserve `dComponent` custom elements for canvas rendering, audio/timer loops, or heavy client javascript. For dynamic list and component mounting, leverage `dComponent.fragment()` and `dComponent.appendMany()` (or `createFragment()` and `appendMany()` from utils) to benefit from template caching and single operation DOM batching
12. always use the `Locale` facade (`import { Locale } from 'dframework'`; `Locale.set('ja')`) to switch or persist user locale; never set `locale` cookies or parse headers manually
13. always ask the user whether Japanese text should use `@ruby` directives and furigana bracket annotations (`漢字[かんじ]`) or plain Japanese text
14. always ask the user whether the application/implementation should use the runtime SPA or classic web navigation
15. to check missing translations and ensure key parity across locales, use the `dstrn lang:lint` command
16. middleware references must use explicit `ClassName@method` string syntax (e.g. `'AuthMiddleware@requireAuth'`) or instantiated objects (`RateLimiter`); short alias strings like `'auth'` do not exist; middleware can pass downstream data to controllers and views using `return next({ key: value })` or `return next().with({ key: value })`
17. for SPA navigation, use `<d-link>` or `<a d-link>` (standard `<a>` tags without `d-link` perform full page reloads); use `<d-form>` for AJAX form submission
18. never use native browser `alert()` or `confirm()`; use `notify()`, `modal()`, or `d-modal` instead
19. keep controllers small and focused; delegate heavy background tasks to `Job.dispatch()` and scheduled tasks to `console/Schedule.js`; jobs can broadcast real time websocket events or target specific users via `Socket.broadcast()`, `Socket.toUser()`, and `Socket.setState()`
20. never write custom scripts to manually compile views or alter view compiler logic; views are compiled 100% automatically by the framework runtime
21. never edit `dstrn.js` or `dstrn.css` directly; these bundles are regenerated on server start and edits will be wiped. If the bundles need to be regenerated, ask the user to restart their server
22. never search for or execute frontend build, rebuild, or compile commands (e.g. `npm run build` or `dstrn build`); the asset compiler runs automatically when starting `dstrn serve`
23. never edit framework source code (`dframework/core/` or `node_modules/dframework/`) to fix project related issues; work entirely within user project files, and rely on the SKILL and documentation rather than inspecting internal implementations
24. never run dframework internal source code tests (`npm test` in `dframework/`) for user projects; internal tests are completely unrelated to user applications
25. never write custom one off test scripts to query the database or inspect models; always use `dstrn tinker` (e.g. `dstrn tinker "await Model.method()"` or `dstrn tinker --execute="await Model.count()"`) to inspect models, test queries, and debug state since all models and facades are fully loaded
26. all comments must strictly be explaining why rather than what
27. prioritize clean, expressive code rather than heavily relying on comments
28. avoid excessive type checks, again clean expressive code does not require type definitions

## detailed references

load and review the relevant reference guide for deep specifications:

- [frontend design system](./references/frontend-design-system.md): 3-tier surface elevation, typographic hierarchy, micro-gaps, app shell layout, and view compute separation
- [dspa client router](./references/spa-router.md): anchor interception, `<d-link>`, `<d-form>`, transition hooks, swap targets, and scoped script lifecycle
- [built in frontend components](./references/frontend-components.md): complete catalog of integrated UI elements (inputs, dropzones, modals, drawers, comboboxes, hold buttons, morphing)
- [localization and i18n](./references/localization-and-i18n.md): Locale facade, t() helper, @t directive, furigana ruby support, dstrn lang:lint, and dictionary structure
- [globals, facades, and helpers](./references/globals-facades-helpers.md): complete inventory of global helpers, facades, type checks, and explicit imports
- [controllers, routing, and responses](./references/controllers-routing-responses.md): controller class conventions, route definitions, group modifiers, validation, and response helpers
- [models, database, and migrations](./references/models-database-migrations.md): active record model methods, relationships, fluent query builder, raw queries, migrations, and storage disk file management
- [views and templating](./references/views-templates.md): `.d` template directives, loops, conditionals, layouts, slots, server javascript blocks, and automatic CSRF
- [frontend, dspa, and reactivity](./references/frontend-spa-reactivity.md): `dstrn.css` utility classes, dSPA router, `d-wire`, `d-live`, and `dComponent` custom elements
- [jobs, scheduler, and websockets](./references/jobs-scheduler-websockets.md): worker thread jobs, task scheduler commands, frequencies, and websocket routes
- [native bridge and plugins](./references/native-plugins-bridge.md): `App.native` APIs, zero config 4-file plugins, and platform simulation tools
- [cli commands and tools](./references/cli-commands.md): built in commands for lifecycle, migrations, database, generators, system, and native platforms
- [architecture and anti patterns](./references/architecture-and-anti-patterns.md): project directory layout, architectural boundaries, and common mistakes table

## execution checklist

before finalizing code in any dframework application:

- [ ] verify all controller and middleware handlers use `return` with response helpers (no `res.json()` or `res.send()`)
- [ ] verify no global facades (`DB`, `Route`, `Auth`, `Log`, `Config`, `Job`, `Session`) were unnecessarily imported
- [ ] verify middleware references use 'ClassName\@method' (e.g. 'AuthMiddleware\@requireAuth') and never short aliases like 'auth'
- [ ] verify no manual `<link href="/css/dstrn.css">` or `<script src="/js/dstrn.js">` tags were added in `<head>`
- [ ] verify `js/dstrn.js` and `css/dstrn.css` were not edited directly
- [ ] verify no unnecessary frontend build scripts or commands were executed
- [ ] verify framework core source code was not modified to solve application issues
- [ ] verify dframework internal tests were not run on user project tasks
- [ ] verify translations were checked for missing keys using dstrn lang:lint
- [ ] verify no custom view compilation scripts were created or core view rendering logic modified
- [ ] verify user was asked whether Japanese text requires @ruby furigana annotations or plain text
- [ ] verify locale switching uses Locale.set() and never manual cookie writing
- [ ] verify UI adheres to 3-tier surface elevation (bg-container-d, bg-container, bg-container-l) and text hierarchy
- [ ] verify localized frontend compute is placed in view @js blocks and pure UI components use @include partials without data arguments
- [ ] verify SPA links use `<d-link>` or `<a d-link>` and forms use `<d-form>`
- [ ] verify entity data operations, relationships, and attribute casting (static casts) prioritize active record Model subclasses over raw DB calls
- [ ] verify models declare static fillable or static guarded to safeguard mass assignment on create() and update()
- [ ] verify production migrations and seeders execute non-interactively via --force flag in deployment scripts
- [ ] verify relationships are eager loaded with with() or load() and consumed via standard async methods (await model.relation()), concurrent resolvers (await Promise.all([this.album(), this.genre()])), or property access without accessing private _relations
- [ ] verify file operations and uploads use Storage.disk() rather than manual filesystem writes
- [ ] verify all database queries use parameterized `?` bindings
- [ ] verify utility CSS classes match `dstrn.css` conventions and not tailwind
- [ ] verify built in UI elements (d-text-input, d-combobox, d-modal, d-image-input, etc.) are used for widgets
- [ ] verify responsive mobile navigation uses <d-hamburger> with target and breakpoint rather than custom JavaScript menu toggles
- [ ] verify model and database inspection uses dstrn tinker with inline code rather than scratch test scripts
- [ ] verify no emojis, use the framework's <i class="dstrn-*"> icon font instead
- [ ] verify comments clearly explain why
- [ ] verify functions adhere to maximum nesting depth of 3 levels