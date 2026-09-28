# dspa client router

dSPA is dframework's zero config client side router that transforms server rendered applications into fast single page applications with DOM morphing, scoped script lifecycle management, and automatic CSRF handling.

## how dspa works

when navigating, the router fetches the destination page in the background, parses the response, extracts targeted content, and surgically morphs only the matching regions of the active page. the browser URL and history are updated via `pushState`, document title is synced, CSRF tokens are refreshed, and page scripts execute in an auto cleanup scope.

layout state, audio playback, and WebSocket connections remain uninterrupted across navigations.

## links and navigation

### anchor interception rules

standard `<a>` tags are only intercepted if marked with the `d-link` attribute:

```html
<!-- intercepted by dSPA -->
<a href="/dashboard" d-link>dashboard</a>

<!-- standard full page browser navigation (skipped by dSPA) -->
<a href="/dashboard">dashboard</a>
```

the router skips interception for:

- standard `<a>` tags without `d-link`
- cross origin links
- links with `target="_blank"`
- links starting with `#`, `mailto:`, `tel:`, or `javascript:`
- clicks while holding modifier keys (ctrl, meta, shift, alt)

### the d-link custom element

`<d-link>` is a dedicated custom element that is always intercepted and provides granular swap control:

```html
<!-- standard spa navigation -->
<d-link href="/dashboard">dashboard</d-link>

<!-- targeting a specific container -->
<d-link href="/admin/users" target="#spa-container">users</d-link>

<!-- extracting specific source selector from fetched page -->
<d-link href="/settings" target="#panel" src="#settings-panel">settings</d-link>

<!-- swap modes: inner, outer, append, prepend -->
<d-link href="/notifications" target="#feed" mode="prepend">load new</d-link>

<!-- preserve scroll position after navigation -->
<d-link href="/feed?page=2" target="#feed" preserve-scroll>next page</d-link>

<!-- force hard full page reload on d-link -->
<d-link href="/logout" d-full-reload>logout</d-link>
```

### active link highlighting

declaratively style active navigation items based on current URL:

```html
<!-- prefix matching (matches /dashboard and /dashboard/settings) -->
<d-link href="/dashboard" d-nav-url="/dashboard" d-nav-active="active-link">dashboard</d-link>

<!-- exact matching -->
<d-link href="/dashboard" d-nav-url="/dashboard" d-nav-active="active-link" d-nav-exact>dashboard</d-link>
```

## default swap targets

by default, the router dynamically checks `dSPA.targets` and `window.dSPA_DEFAULT_TARGETS`, falling back to candidate selectors `['#spa-container', 'main', 'body']`. the router resolves the first selector matching in both pages. to override globally across your layout:

```html
<script defer>
  dSPA.targets = ['main'];
</script>
```

## programmatic navigation

navigate from javascript via `dSPA.navigate()`:

```javascript
// simple navigation
dSPA.navigate('/dashboard');

// advanced navigation with options and transition hooks
dSPA.navigate('/profile', {
  targets: ['#content'],
  srcTargets: ['#profile-view'],
  mode: 'inner',
  preserveScroll: true,
  transitions: {
    beforeSwapTarget(oldEl, meta) {
      oldEl.style.opacity = '0';
    },
    afterSwapTarget(newEl, meta) {
      newEl.style.opacity = '1';
    }
  }
});
```

### transition hooks

register global transitions on `dSPA.transitions` in your layout:

```html
<script defer>
  dSPA.transitions = {
    beforeSwap(rootNode, meta) {
      // animate old content out (async aware, router directly awaits returned promise)
      return rootNode.animate([{ opacity: 1 }, { opacity: 0 }], { duration: 150 }).finished;
    },
    afterSwap(rootNode, meta) {
      // animate new content in (directly awaited before finishing navigation)
      return rootNode.animate([{ opacity: 0 }, { opacity: 1 }], { duration: 150 }).finished;
    }
  };
</script>
```

per navigation transitions passed to `dSPA.navigate()` override global transitions for that navigation.

## d-form (ajax form submission)

`<d-form>` intercepts form submissions, submits via `fetch()`, auto injects CSRF headers, and disables inputs while submitting:

```html
<d-form action="/login" method="POST" class="flex-column g-1">
  <input type="email" name="email" required>
  <input type="password" name="password" required>
  <button type="submit">sign in</button>
</d-form>
```

### client side pre validation

```html
<d-form action="/register" method="POST" rules='{"email": "required|email", "password": "required|min:8"}'>
  <input type="email" name="email">
  <input type="password" name="password">
  <button type="submit">register</button>
</d-form>
```

### handling server responses in d-form

- JSON redirect: if server returns `redirect('/path')`, form navigates via dSPA. if `force: true` was set (`redirect('/path', 302, true)`), performs full browser reload.
- JSON errors: if server returns validation errors, form automatically highlights invalid inputs and renders error messages.
- HTML response: replaces content of `target` element (`target="#results"`). add `navigate` attribute to run the full dSPA lifecycle instead of simple element replace.
- force reload: `<d-form action="/save" force-reload="success">` forces hard reload on success.
- javascript callback: `<d-form action="/api/items" callback="onItemCreated">` calls global function with response data.

## script execution and scope lifecycle

scripts inside swapped content execute automatically in an isolated scope. all timers, event listeners, and pending fetch requests are automatically destroyed when the owning DOM node is removed on navigation.

### scoped browser apis

inline page scripts receive local scoped versions of browser utilities:

- `setTimeout`, `setInterval`, `clearTimeout`, `clearInterval`
- `requestAnimationFrame`, `cancelAnimationFrame`
- `fetch` (automatically injects `X-CSRF-TOKEN`)
- `listen(target, event, handler)`, `listenAll`, `unlisten`, `unlistenAll`
- `nextFrame()`, `sleep(ms)`
- `Socket` (framework frontend socket client)
- `dSPA` (router instance)

```html
<!-- in a swapped page view -->
<div id="status">active</div>
<script>
  // this timer is automatically cleared when navigating away from this view
  setInterval(() => {
    select('#status').textContent = new Date().toLocaleTimeString();
  }, 1000);
</script>
```

### external scripts and persistent elements

- standard `<script>` tags: scripts (inline and `<script src="...">`) inside swapped regions automatically re-execute on every navigation in document order within an auto cleanup scope.
- persistent initial scripts: `<script type="text/dspa">` is used for initial layout scripts that need to defer execution until `Socket.connect()` completes on first page boot.
- ignore script: add `d-spa-ignore` (`<script src="/tracking.js" d-spa-ignore>`) to prevent dSPA execution on navigation.
- persistent elements / scripts (`d-spa-keep`): elements or scripts with `d-spa-keep` survive body swaps and initial startup cleanup.
- module scripts: add `d-spa-scope` (`<script type="module" d-spa-scope>`) to opt into scoped APIs.