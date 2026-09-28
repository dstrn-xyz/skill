# frontend, dspa, and reactivity

dframework provides an integrated frontend stack: compiled custom utility CSS (`dstrn.css`), the `dSPA` client router, WebSocket reactivity (`d-wire` and `d-live`), built in custom web components, and the `dComponent` reactive element base class.

## utility css classes (dstrn.css)

dframework provides its own custom compiled utility classes. do not assume tailwind or bootstrap classes exist.

### layout and flex

- flex container: `.flex-row`, `.flex-column`, `.flex-center`, `.flex-wrap`, `.flex-1`, `.flex-auto`, `.flex-none`
- alignment: `.justify-start`, `.justify-center`, `.justify-end`, `.justify-between`, `.justify-around`, `.justify-evenly`
- items: `.align-start`, `.align-center`, `.align-end`, `.align-stretch`, `.align-baseline`
- grid: `.grid-center`, `.grid-2`, `.grid-3`, `.grid-4`, `.grid-auto`, `.col-span-1` through `.col-span-4`, `.col-span-full`
- position: `.relative`, `.absolute`, `.fixed`, `.sticky`, `.inset-0`, `.absolute-center`, `.absolute-fill`

### spacing and sizing

- padding: `.p-0`, `.p-05`, `.p-1`, `.p-15`, `.p-2`, `.p-3`, `.px-1`, `.py-2`, `.pt-1`, `.pb-1`
- margin: `.m-0`, `.m-auto`, `.mx-auto`, `.my-1`, `.mt-2`, `.mb-2`, `.ml-auto`
- gap: `.g-05`, `.g-1`, `.g-15`, `.g-2`, `.g-3`, `.gx-1` (column gap), `.gy-1` (row gap)
- width: `.w-100` (100%), `.w-50`, `.w-auto`, `.w-100vw`, `.min-w-0`, `.max-w-100`
- height: `.h-100` (100%), `.h-100svh`, `.min-h-100svh`, `.h-auto`

### typography and appearance

- size: `.fs-08`, `.fs-1`, `.fs-12`, `.fs-15`, `.fs-2`, `.fs-3`
- weight: `.fw-400`, `.fw-500`, `.fw-600`, `.bold` (`fw-700`), `.thin` (`fw-300`)
- alignment: `.text-left`, `.text-center`, `.text-right`
- text wrap: `.truncate`, `.overflow-ellipsis`, `.break-words`
- radius: `.bdr-05`, `.bdr-1`, `.bdr-2`, `.bdr-circle`
- border: `.border-s`, `.border-m`, `.border-bottom-s`, `.border-top-s`
- shadow: `.shadow-s`, `.shadow-m`, `.shadow-l`
- cursor and effects: `.pointer`, `.hover:bg-accent`, `.hover:text-accent`, `.shadow-hover-s`
- responsive prefixes: `sm:` (min 480px), `md:` (min 768px), `lg:` (min 1024px), `xl:` (min 1280px)

```html
<!-- card layout example -->
<div class="flex-column g-1 p-2 bdr-1 border-s bg-container shadow-s md:flex-row">
  <div class="flex-1">
    <h3 class="fs-12 bold">system overview</h3>
    <p class="fs-09">live server telemetry</p>
  </div>
  <button class="pointer bdr-05 p-05 bg-accent text-white hover:bg-accent-dark">view</button>
</div>
```

## global frontend dom utilities

available globally in client side scripts:

- `select(selector, parent)`: `querySelector` shorthand
- `selectAll(selector, parent)`: `querySelectorAll` returning true array
- `listen(element, event, handler)`: adds event listener, returns cleanup function
- `addClass(element, className)` / `removeClass(element, className)` / `toggleClass(element, className)`
- `notify(message, duration)`: toast notification
- `modal(htmlString, callback)`: display interactive modal dialog
- `createFragment(html)`: parses html into a cached template fragment
- `appendMany(container, ...nodes)`: appends multiple nodes, fragments, or html strings in a single dom operation
- `escapeHtml(val)`: escapes html special characters
- `escapeJs(val)`: escapes javascript string and template literal special characters
- `sleep(ms)`: promise delay
- `nextFrame()`: promise resolving after animation frames

## dspa router summary

see [spa-router.md](./spa-router.md) for full documentation:

- standard `<a>` tags are only intercepted when marked with `d-link` attribute (`<a href="/url" d-link>`)
- `<d-link href="/url">` is always intercepted and supports `target`, `src`, `mode`, and `preserve-scroll`
- `<d-form action="/url" method="POST">` submits via fetch and auto injects CSRF headers
- all scripts inside swapped content re-execute automatically on navigation in an auto cleanup scope; use `d-spa-ignore` to skip and `d-spa-keep` to preserve elements/scripts across body swaps; persistent SPA scripts use `<script type="text/dspa">`

## real time reactivity (d-wire and d-live)

dframework provides two distinct reactivity systems:

### d-wire (user action to server event)

use `d-wire` when user interaction triggers a server action:

```html
<!-- frontend template -->
<button d-wire="counter:increment" d-target="#counter-box">+1</button>
<div id="counter-box">{{ count }}</div>
```

```javascript
// backend handler in routes/wire.js
Socket.on('counter:increment', async (req) => {
  const count = await Counter.increment();
  return render('partials.counter', { count });
}).targets(['#counter-box']);
```

### d-live (database change stream)

use `d-live` to automatically refresh a DOM region whenever a database table changes. no backend handler needed:

```html
<!-- updates automatically when records in 'posts' table are created or updated -->
<div d-live="posts" d-live-filter="status:published">
  @foreach (posts as post)
    <article>{{ post.title }}</article>
  @endforeach
</div>
```

## authoring custom web components (dComponent)

all custom components extend `dComponent`:

```javascript
// public/js/components/UserBadge.js
class UserBadge extends dComponent {
  static tag = 'd-user-badge';
  static props = {
    userId: { type: 'number', default: 0 }
  };

  template() {
    return `<span class="badge">loading</span>`;
  }

  mount() {
    this.effect(async () => {
      const user = await fetch(`/api/users/${this.state.userId}`).then(r => r.json());
      this.setState({ user });
    });
  }

  render() {
    const el = this.querySelector('.badge');
    if (this.state.user && el) {
      el.textContent = this.state.user.name;
    }
  }
}

dComponent.define(UserBadge);
```

### performance characteristics

- **batched mount & cached template stamping:** initial `mount()` is batched via `requestAnimationFrame` to ensure child nodes are parsed in light DOM before initialization. `template()` strings are preparsed once on `define()` and stamped via cached `cloneNode(true)`.
- **static fragment caching & safe dom insertion:** `dComponent.fragment(html)` caches parsed templates in an LRU map. `dComponent.appendMany(target, ...nodes)` mounts multiple nodes in a single batched DOM insertion. `this.setChildren(content)` replaces component child nodes safely using cached template fragments.
- **3-phase microtask update queue:** state and prop mutations coalesce into an ordered 3-phase microtask flush: attribute sync, surgical render, effects and form sync.

### overriding built in components

user components in `public/js/components/` can override any built in UI component (such as `dCheckbox.js`). `dComponent` itself is immutable and cannot be overwritten by user code.

## zero build automatic asset compilation

the framework compiles `public/js/dstrn.js` and `public/css/dstrn.css` automatically on startup when running `dstrn serve`.

- never edit `public/js/dstrn.js` or `public/css/dstrn.css` directly; changes will be overwritten on next server start
- there are no manual frontend build or compile commands (e.g. `npm run build` or `dstrn build`); the compiler runs on boot
- if a rebuild is needed, ask the user to restart their development server
- author modular components in `public/js/components/<ComponentName>.js`
- author custom styles in `public/css/app.css` or other files