# built in frontend components

dframework includes a comprehensive library of zero dependency, high performance custom web components compiled into the frontend bundle. always prioritize using these built in elements rather than authoring ad hoc HTML/JS inputs or widgets from scratch.

## core mandate

when building interfaces, always use dframework components:

- use `d-text-input` instead of raw `<input type="text">` when searching, formatting numbers, or handling shortcuts
- use `d-combobox` for searchable, dynamic dropdown selects with automatic viewport positioning
- use `d-hamburger` for responsive mobile navigation menus and multi level sliding subcategory overlays
- use `d-dropdown` and `d-drawer` for navigation, accordions, and offcanvas panels
- use `d-hold-button` for destructive or critical actions (delete, reset, prune)
- use `d-image-input` and `d-file-input` for drag and drop file uploads with preview and size formatting
- use `d-modal` and `d-notification` for dialogs and toasts
- use `d-skeleton` during content loading states
- use `d-slider` for interactive numeric range inputs with automatic indicator binding
- use `d-color-picker` for interactive hex color selection
- use `d-toggle` for segmented switches with spring and hover animations
- use `d-morph` for smooth physics based transitions between element states
- use `d-context-menu` for right click contextual actions with boundary clamping

## component catalog

### d-checkbox

accessible, form associated checkbox switch:

```html
<d-checkbox text="enable two factor authentication" checked name="two_factor" value="1"></d-checkbox>
```

- attributes/props:
  - `checked`: `boolean` (default `false`, reflected to form value)
  - `text`: `string` (default `"checkbox"`)
  - `name`: `string` (default `""`)
  - `value`: `string` (default `"on"`, form payload)
- programmatic api:
  - `.checked`, `.text`, `.name`, `.value` (getters and setters)
- events: `change`
- behavior: automatically synchronizes checked state, label text, and form submission values when toggled or updated programmatically.

### d-color-picker

interactive hex color picker featuring 2D saturation/brightness, hue, and alpha controls:

```html
<d-color-picker value="#3b82f6" name="brand_color" label="brand color"></d-color-picker>
```

- attributes/props:
  - `value`: `string` (default `"#d3ac5f"`, supports 3, 4, 6, 8 hex digits, form value)
  - `name`: `string` (default `""`)
  - `id`: `string` (default `""`)
  - `label`: `string` (default `"color picker"`)
  - `opened`: `boolean` (default `false`)
- programmatic api:
  - `toggle()`: opens or closes popup panel
  - `setHex(hex)`: sets hex color and updates visuals
  - `.value`, `.hex`, `.h`, `.s`, `.b`, `.a`, `.isOpen`
- events: `change` (detail: hex string), `input` (detail: hex string)
- behavior: interactive 2D color plane, hue bar, and alpha slider with smooth dragging and resize responsiveness.

### d-combobox

searchable, extensible select component with automatic floating placement:

```html
<d-combobox
  placeholder="select department"
  allow-search
  allow-input
  name="department">
  <option value="eng" selected>engineering</option>
  <option value="des">design</option>
  <option value="ops">operations</option>
</d-combobox>
```

- attributes/props:
  - `placeholder`: `string` (default `""`)
  - `value`: `string` (default `null`, form value)
  - `allow-search` / `allowSearch`: `boolean` (default `false`)
  - `allow-input` / `allowInput`: `boolean` (default `false`)
  - `horizontal` / `isHorizontal`: `boolean` (default `false`)
  - `name`: `string` (default `""`)
  - `id`: `string` (default `""`)
  - `options`: `array` (default `[]`, array of `{value, text, content}`)
- static methods:
  - `dCombobox.closeAll(except)`: closes all open comboboxes across document
- programmatic api:
  - `addOption(value, text)`: appends option dynamically
  - `removeOption(value)`: removes option by value
  - `clearOptions()`: clears all options
  - `destroy()`: cleans up dropdown
  - `.value`, `.options`, `.placeholder`, `.allowSearch`, `.allowInput`, `.isHorizontal`
- events: `change` (detail: value), `input`, `add` (detail: item), `open`, `close`
- behavior: parses child `<option>` tags, calculates viewport space to open above or below without overflowing, supports full keyboard navigation and search filtering.

### d-context-menu

right click contextual menu with automatic boundary collision clamping:

```html
<div class="card p-2">
  right click inside this card
  <d-context-menu>
    <span>copy link</span>
    <span>duplicate</span>
    <span>delete</span>
  </d-context-menu>
</div>
```

- attributes/props:
  - `opened`: `boolean` (default `false`)
  - `parent`: `string` selector (defaults to parent element)
- programmatic api:
  - `open(e)`: opens menu with optional pointer event coordinates
  - `close(e)`: closes menu
  - `toggle(e)`: toggles menu
- events: `click` (detail: clicked element)
- behavior: opens on right-click within its parent container, bounds coordinates so the menu never overflows parent or screen edges, dismisses on outside click without triggering actions on underlying elements. when creating dynamically, listen to `'click'` directly on the component element.

### d-drawer

sliding offcanvas navigation drawer with wave animations:

```html
<d-drawer direction="left" id="sidebar-drawer">
  <nav class="flex-column g-1 p-2">
    <a href="/dashboard">dashboard</a>
    <a href="/settings">settings</a>
  </nav>
</d-drawer>
```

- attributes/props:
  - `opened`: `boolean` (default `false`)
  - `direction`: `string` (`"left"`, `"right"`, `"top"`, `"bottom"`, default `"left"`)
- programmatic api:
  - `open()`: returns Promise resolving after animation completes
  - `close()`: returns Promise
  - `toggle()`: returns Promise
  - `.isOpened`, `.isAnimating`, `.direction`, `.opened`
- events: `open`, `close`
- behavior: slides into view from the configured edge with multi layer wave animations.

### d-dropdown

collapsible accordion dropdown container:

```html
<d-dropdown header="account security">
  <div class="p-1 flex-column g-1">
    <a href="/profile">profile settings</a>
    <a href="/sessions">active sessions</a>
  </div>
</d-dropdown>
```

- attributes/props:
  - `header`: `string` (default `"dropdown"`)
  - `opened`: `boolean` (default `false`)
- programmatic api:
  - `open()`: expands dropdown
  - `close()`: collapses dropdown
  - `toggle()`: toggles state
  - `.header`, `.opened`, `.isOpened`
- behavior: clicking the header expands or collapses child content with an animated chevron indicator.

### d-file-input

drag and drop file upload container with interactive item list:

```html
<d-file-input accept=".pdf,.docx,.zip" name="attachments" multiple compact></d-file-input>
```

- attributes/props:
  - `accept`: `string` (default `"*/*"`)
  - `icon`: `string` (default `"dstrn-folder-line"`)
  - `placeholder`: `string` (default `"drag & drop file"`)
  - `subtitle`: `string` (default `"or click to browse"`)
  - `compact`: `boolean` (default `true`)
  - `multiple`: `boolean` (default `false`)
  - `name`: `string` (default `""`)
  - `id`: `string` (default `""`)
  - `value`: `object`/`array` (default `null`, `File`, `File[]`, or URL string)
- programmatic api:
  - `reset()`: clears files, value, and input
  - `.value`, `.accept`, `.compact`, `.multiple`
- events: `change` (detail: `File`, `File[]`, or `null`)
- behavior: drag and drop file upload with file list display, formatted sizes (Bytes, KB, MB, GB), item deletion, and form integration.

### d-hamburger

responsive mobile navigation hamburger trigger and full screen navigation overlay with programmable media query breakpoints and multi level sliding subcategory drilldown panels:

```html
<nav class="nav bg-container bdr border-s px-1 py-05 flex-row justify-between align-center">
  <a d-link href="/" class="flex-row align-center">
    <span class="bold text-content-l">app name</span>
  </a>

  <!-- desktop container: automatically hidden below breakpoint and restored above breakpoint -->
  <div id="nav-links" class="flex-row align-center g-1">
    <a d-link href="/features" class="text-content hover:text-content-l fs-09">features</a>
    <div d-submenu="framework">
      <a d-link href="/architecture" class="text-content hover:text-content-l fs-09">architecture</a>
      <a d-link href="/benchmarks" class="text-content hover:text-content-l fs-09">benchmarks</a>
    </div>
    <a d-link href="/pricing" class="text-content hover:text-content-l fs-09">pricing</a>
  </div>

  <!-- responsive mobile hamburger overlay -->
  <d-hamburger target="#nav-links" breakpoint="md">
    <div slot="header" class="flex-row align-center g-05">
      <span class="bold text-content-l">app name</span>
    </div>
    <div slot="footer">
      <a d-link href="/login" class="btn btn-sm w-100">sign in</a>
    </div>
  </d-hamburger>
</nav>
```

- attributes/props:
  - `opened`: `boolean` (default `false`)
  - `breakpoint`: `string` (default `"md"`, tokens: `sm`, `md`, `lg`, `xl`, `xxl`, `ultra` or arbitrary query like `"(max-width: 48em)"`)
  - `target`: `string` (selector of desktop container to clone links from and automatically toggle visibility)
  - `animation`: `string` (`slide-down`, `fade`, `slide-right`, `slide-left`)
  - `direction`: `string` (`top`, `right`, `left`, `bottom`)
  - `size`: `string` (default `"1.5em"`)
  - `color`: `string` (default `"currentColor"`)
  - `activecolor` / `activeColor`: `string` (default `"var(--accent)"`)
  - `autohidetarget` / `autoHideTarget`: `boolean` (default `true`, automatically toggles desktop container display)
  - `title`: `string` (default `""`, fallback header title when `slot="header"` is omitted)
- slots: `slot="header"`, `slot="footer"`
- submenus:
  - nested: wrap links with `d-submenu="category name"` to create interactive drilldown panels with back navigation
  - linked: use `d-submenu="#element-id"` on a trigger button to slide into panel populated from target ID
- static methods: `dHamburger.closeAll(except)`
- programmatic api:
  - `open()`: returns Promise
  - `close()`: returns Promise
  - `toggle()`: returns Promise
  - `navigateTo(subPanel, title)`: slides into subcategory panel
  - `navigateBack()`: returns to parent panel
  - `destroy()`: cleans up overlay and media listeners
- events: `open`, `close`
- behavior: watches viewport width, automatically hides the desktop container and reveals the hamburger button below the configured breakpoint, restores desktop container above breakpoint, clones interactive items into sliding category panels, locks page scrolling while open, and closes automatically on SPA navigation (`dspa:navigated`), link clicks, `Escape`, or backdrop clicks.

### d-hold-button

press and hold button preventing accidental activation of destructive actions:

```html
<d-hold-button delay="1500" class="btn btn-red" type="submit">
  hold to delete
</d-hold-button>
```

- attributes/props:
  - `delay`: `number` (default `800`, ms hold duration)
  - `type`: `string` (`"button"`, `"submit"`, default `"button"`)
  - `disabled`: `boolean` (default `false`)
- programmatic api: `.delay`, `.type`, `.disabled`
- events: `click` (fires only after holding for full duration)
- behavior: requires continuous press for the configured duration before triggering action, and automatically submits surrounding form when `type="submit"`.

### d-icon-button

icon button wrapper:

```html
<d-icon-button icon="dstrn-heart-line" size="2em" color="var(--content-l)" activecolor="var(--red)"></d-icon-button>
```

- attributes/props:
  - `icon`: `string` (CSS class name)
  - `size`: `string` (default `"4.2em"`)
  - `color`: `string` (default `"var(--content-l)"`)
  - `activecolor` / `activeColor`: `string` (default `"var(--accent)"`)
  - `id`: `string` (default `""`)

### d-image-input

drag and drop image upload container with client preview:

```html
<d-image-input accept="image/png,image/jpeg" name="avatar" fit="cover" placeholder="drop photo"></d-image-input>
```

- attributes/props:
  - `accept`: `string` (default `"image/png, image/jpeg, image/gif"`)
  - `icon`: `string` (default `"dstrn-pic-line"`)
  - `placeholder`: `string` (default `"drag & drop image"`)
  - `subtitle`: `string` (default `"or click to browse"`)
  - `replace-text` / `replaceText`: `string` (default `"replace"`)
  - `delete-text` / `deleteText`: `string` (default `"delete"`)
  - `no-replace` / `noReplace`: `boolean` (default `false`)
  - `no-delete` / `noDelete`: `boolean` (default `false`)
  - `fit`: `string` (`"contain"`, `"cover"`, default `"contain"`)
  - `name`: `string` (default `""`)
  - `id`: `string` (default `""`)
  - `value`: `object`/`string` (default `null`, `File` object or image URL string)
- programmatic api:
  - `reset()`: clears image, preview, value, and input
  - `.preview`: getter/setter for preview URL string
  - `.value`, `.fit`
- events: `change` (detail: `File` or `null`)
- behavior: provides drag and drop image selection with instant preview, action buttons to replace or remove images, and form submission integration.

### d-loader

canvas arc spinner:

```html
<d-loader style="width: 2em; height: 2em;"></d-loader>
```

- attributes/props:
  - `color`: `string` (stroke color, falls back to `--accent` or text color)
- programmatic api: `destroy()`
- behavior: lightweight canvas spinner that automatically pauses animation when scrolled offscreen to conserve CPU and adapts to container dimensions.

### d-modal

modal dialog container:

```html
<d-modal id="dialog" persistent>
  <div class="flex-column g-1">
    <h3>dialog title</h3>
    <p class="text-content-l">dialog content goes here</p>
    <button class="btn btn-sm" onclick="this.closest('d-modal').close()">close</button>
  </div>
</d-modal>
```

- attributes/props:
  - `opened`: `boolean` (default `false`)
  - `persistent`: `boolean` (default `false`)
- programmatic api:
  - `open()`: opens modal and locks body scroll
  - `close()`: closes modal with exit animation
  - `destroy()`: closes and removes element from DOM
  - `.opened`, `.isOpened`
- events: `open`, `close`
- behavior: displays an overlay dialog and prevents page scrolling while visible. when `persistent` is true, clicking the backdrop calls `close()` to hide the modal without removing the element from the DOM. when `persistent` is false (default), clicking the backdrop calls `destroy()` and removes the element.
- styling boundaries: `<d-modal>` automatically wraps content in `.d-modal-wrapper` (container background, border, radius, shadow, padding) and `.d-modal-content`. avoid adding outer container classes like `bg-container-l bdr` inside the template to prevent duplicate borders and padding. target `d-modal .d-modal-wrapper` in CSS to customize styling.

### d-morph

physics based morphing animation engine (`window.dMotion`):

```html
<d-morph name="summary" d-action="click" d-to="detail" d-duration="350" d-visible>
  <div class="card p-2">summary card</div>
</d-morph>

<d-morph name="detail" d-action="click" d-to="summary" d-duration="350">
  <div class="card p-4">detail card</div>
</d-morph>
```

- `<d-morph>` attributes: `name`, `d-visible`, `d-action` (`click`, `long-press`, `click-out`), `d-to`, `d-duration`
- `dMotion` API:
  - `dMotion.get(name)`: retrieves morph instance
  - `dMotion.getPrevious()`: returns previously active morph element
  - `dMotion.transition(from, to, triggerNode)`: triggers programmatic morph transition
  - `dMotion.listen(from, to, callback, hook)`: hooks (`'before'` or `'after'`)
  - `dMotion.unlisten(from, to, callback, hook)`
- behavior: creates smooth animated morph transitions between two elements anywhere on the page, preserving visual styles and form states during animation. child elements with `d-no-morph` remain interactive without triggering the transition.

### d-notification

toast notification element:

```html
<d-notification timer="4000" opened>
  <p>changes saved successfully</p>
</d-notification>
```

- attributes/props:
  - `opened`: `boolean` (default `false`)
  - `timer`: `number` (default `0`, auto-dismiss timeout in ms)
- programmatic api:
  - `show()`: displays toast and begins countdown
  - `hide()`: hides toast
  - `destroy()`: awaits animation and removes from DOM
- behavior: displays toast notifications with an optional countdown progress bar that automatically dismisses after the timer expires.

### d-skeleton

loading placeholder shapes:

```html
<d-skeleton type="text" lines="3" gap="0.5em"></d-skeleton>
<d-skeleton type="circle" size="3em"></d-skeleton>
<d-skeleton type="rect" width="100%" height="150px" radius="0.5em"></d-skeleton>
<d-skeleton type="card"></d-skeleton>
```

- attributes/props: `type` (`text`, `circle`, `rect`, `card`), `lines`, `width`, `height`, `size`, `gap`, `radius`

### d-slider

custom range slider:

```html
<d-slider id="vol" name="volume" min="0" max="1" step="0.01" value="0.5"></d-slider>
<span data-slider-vol>50%</span>
```

- attributes/props:
  - `value`: `number` (default `0.5`, form value)
  - `min`: `number` (default `0`)
  - `max`: `number` (default `1`)
  - `step`: `number` (default `0.01`)
  - `name`: `string` (default `""`)
  - `id`: `string` (default `""`)
  - `disabled`: `boolean` (default `false`)
  - `thumbcolor` / `thumbColor`: `string` (default `"var(--accent)"`)
  - `fillcolor` / `fillColor`: `string` (default `"var(--accent-d)"`)
  - `trackcolor` / `trackColor`: `string` (default `"var(--border-muted)"`)
  - `sfx`: `string` (optional audio feedback URL)
- programmatic api:
  - `getValue()`: returns numeric value
  - `setValue(v)`: updates value and visual indicators
- events: `input`, `change`
- behavior: supports smooth dragging and keyboard arrow navigation, updates text in any element with `data-slider-{id}` with the current percentage value, and integrates with form submission.

### d-text-input

enhanced text input:

```html
<d-text-input placeholder="search" icon="dstrn-search-line" shortcut="cmd+k" name="q"></d-text-input>
```

- attributes/props:
  - `placeholder`: `string` (default `""`)
  - `icon`: `string` (default `""`)
  - `type`: `string` (`"text"`, `"password"`, `"search"`, `"number"`, default `"text"`)
  - `autocomplete`: `boolean` (default `false`)
  - `value`: `string` (default `""`, form value)
  - `name`: `string` (default `""`)
  - `id`: `string` (default `""`)
  - `shortcut`: `string` (default `""`, e.g. `"cmd+k"`, `"ctrl+f"`)
  - `disabled`: `boolean` (default `false`)
  - `readonly`: `boolean` (default `false`)
  - `min`, `max`, `step`: `string` (number controls)
- programmatic api: `.value`, `.placeholder`, `.icon`, `.disabled`, `.readonly`
- events: `change`
- behavior: supports global keyboard shortcuts (e.g. `cmd+k`) that focus the field automatically, and displays increment/decrement stepper controls when configured with `type="number"`.

### d-toggle

segmented control with magnetic hover and spring transitions:

```html
<d-toggle name="plan" value="0">
  <div default>monthly</div>
  <div>annually</div>
</d-toggle>
```

- attributes/props:
  - `value`: `number` (default `0`, selected option index, form value)
  - `name`: `string` (default `""`)
  - `id`: `string` (default `""`)
  - `color`: `string` (default `"var(--container-l)"`)
  - `activecolor` / `activeColor`: `string` (default `"var(--accent)"`)
  - `textcolor` / `textColor`: `string` (default `"var(--content-l)"`)
  - `textactivecolor` / `textActiveColor`: `string` (default `""`)
  - `bgcolor` / `bgColor`: `string` (default `"var(--container-l)"`)
  - `hovercolor` / `hoverColor`: `string` (default `"rgba(255, 255, 255, 0.05)"`)
- programmatic api:
  - `.selected`: getter returns active element, setter accepts index (number/string) or button element
  - `.value`: selected index number
- events: `change` (detail: selected index number)
- behavior: renders interactive segmented options with animated hover and selection transitions, and supports initial selection via the `default` attribute or numeric value.

---

## zero build automatic asset compilation

`public/js/dstrn.js` and `public/css/dstrn.css` are automatically compiled and regenerated by the framework runtime when running `dstrn serve`.

strictly enforce these rules:

- never edit `public/js/dstrn.js` or `public/css/dstrn.css` directly; any edits are wiped on server restart
- never search for or run build, rebuild, or compile commands (e.g. `npm run build` or `dstrn build`); the compiler runs automatically on boot
- author custom or override components in `public/js/components/<ComponentName>.js`
- author custom styling in `public/css/app.css`

## overriding built in components

all built in UI component behaviors can be patched or overridden by placing a matching file in `public/js/components/` (e.g. `public/js/components/dCheckbox.js`).

the compiler automatically replaces the core component with the user implementation when building `public/js/dstrn.js`.

### dComponent is immutable

`dComponent` is the foundational base class for all custom elements and reactivity. it cannot be overridden or shadowed by user code. the compiler always enforces that the core `dComponent` base class is bundled first so all components inherit properly.