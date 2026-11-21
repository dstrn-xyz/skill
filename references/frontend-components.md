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
- use `d-popover` for top layer floating overlays, dropdowns, and contextual flyouts
- use `d-menubar` for desktop application menubars with submenus and shortcuts
- use `d-navmenu` for horizontal website navigation with shared morphing viewports and automatic `d-hamburger` mobile drilldown
- use `d-tooltip` for hover and focus hints, keyboard shortcut indicators, and disabled action explanations
- use `d-morph` for smooth physics based transitions between element states
- use `d-link` or `<a d-link>` for client side navigation without full page reloads
- use `d-form` for ajax form submission, automatic csrf injection, and inline error handling
- use `d-context-menu` for right click contextual actions with boundary clamping
- built in components support native client side localization out of the box with english and japanese dictionaries; use `this.t()` in custom dcomponents (see the localization reference for details)

## component catalog

### d-breadcrumb

hierarchical navigation trail with custom separators, live child node preservation, and collapsible middle items:

```html
<d-breadcrumb separator="/" max-items="3">
  <a d-link href="/">home</a>
  <a d-link href="/workspace">workspace</a>
  <a d-link href="/projects">projects</a>
  <span>settings</span>
</d-breadcrumb>
```

- attributes/props:
  - `separator`: `string` (default `""`, defaults to `dstrn-right-small-line`; accepts text or `dstrn-*` icon class)
  - `max-items` / `maxItems`: `number` (default `0`, collapses middle items into an interactive ellipsis popover)
  - `items`: `array` (default `null`, array of `{ label, href, link }`)
- programmatic api:
  - `expand()`, `collapse()`, `.separator`, `.maxItems`, `.items`
- events: `expand`, `collapse`
- behavior: wraps live child nodes into an ordered list with separators, highlights the final item, and reveals collapsed middle items on click.

### d-buttongroup

horizontal or vertical button group fusing adjacent `.btn` buttons, inputs, text addons, and popover/tooltip triggers:

```html
<d-buttongroup>
  <button class="btn btn-outline">archive</button>
  <button class="btn btn-outline">report</button>
</d-buttongroup>

<d-buttongroup orientation="vertical">
  <button class="btn btn-outline"><i class="dstrn-add-line"></i></button>
  <button class="btn btn-outline"><i class="dstrn-subtract-line"></i></button>
</d-buttongroup>
```

- attributes/props:
  - `orientation`: `string` (`"horizontal"` default, or `"vertical"`)
  - `disabled`: `boolean` (default `false`, disables descendant controls while preserving previously disabled ones)
  - `dividers`: `boolean` (default `false`, adds automatic hairline dividers between solid buttons)
- helper classes:
  - `.d-btngroup-sep` (or `<hr>`): visual hairline separator between solid buttons
  - `.d-btngroup-text`: inline text/label addon matching button height and borders
- behavior: collapses overlapping borders, preserves outer radius with logical CSS properties (RTL ready), spaces nested `<d-buttongroup>` toolbars automatically, and fuses `<d-popover>`, `<d-tooltip>`, `<d-text-input>`, and `<d-combobox>` triggers cleanly.

### d-calendar

zero dependency interactive calendar featuring single, range, and multiple date selection, native `Intl.DateTimeFormat` month and weekday localization, smooth carousel month transitions, and keyboard navigation:

```html
<!-- single date selection -->
<d-calendar mode="single" value="2026-11-19"></d-calendar>

<!-- date range selection with continuous track highlight -->
<d-calendar mode="range" value="2026-11-10,2026-11-20"></d-calendar>

<!-- multiple date selection -->
<d-calendar mode="multiple" value="2026-11-05,2026-11-12,2026-11-19"></d-calendar>

<!-- quick month and year dropdown jump -->
<d-calendar caption-layout="dropdown"></d-calendar>
```

- attributes/props:
  - `mode`: `string` (`"single"` default, `"range"`, or `"multiple"`)
  - `value`: `string` (selected ISO date `YYYY-MM-DD`, range `from,to`, or comma list)
  - `month`: `number` (`1` to `12`)
  - `year`: `number` (e.g. `2026`)
  - `min`: `string` (`YYYY-MM-DD` lower bound)
  - `max`: `string` (`YYYY-MM-DD` upper bound)
  - `locale`: `string` (BCP 47 code, defaults to document or system language)
  - `week-starts-on`: `number` (`0` Sunday or `1` Monday)
  - `show-outside-days`: `boolean` (default `true`)
  - `caption-layout`: `string` (`"label"` default with arrow buttons, or `"dropdown"` month/year selectors)
  - `disabled`: `boolean` (default `false`)
- programmatic api:
  - `prevMonth()`, `nextMonth()`
  - `.selectedDate`, `.range`, `.selectedDates`
- events:
  - `select`: dispatched on day cell click `{ value, date }`
  - `change`: dispatched on selection state update `{ value, date, from, to, fromDate, toDate, dates }`
  - `month-change`: dispatched on month navigation `{ year, month }`
- behavior: directional carousel slide transitions on month switch, interactive month and year grid zoom on header text click, continuous highlighted track in range mode, compact 2-letter weekday headers (`Su`, `Mo`, `Tu`, `We`, `Th`, `Fr`, `Sa`), full arrow/PageUp/PageDown keyboard navigation, and automatic `min`/`max` cell disabling.

### d-datepicker

form-associated date picker combining `<d-popover>` and `<d-calendar>` with native trigger styling, clear button, single and range modes, localized formatting, and keyboard shortcuts:

```html
<!-- single date selection with placeholder -->
<d-datepicker name="due_date" placeholder="pick a date"></d-datepicker>

<!-- localized date display with initial value -->
<d-datepicker name="event_date" value="2026-11-19" format="localized" clearable></d-datepicker>

<!-- date range picker -->
<d-datepicker name="reservation" mode="range" value="2026-11-10,2026-11-20" clearable></d-datepicker>
```

- attributes/props:
  - `name`: `string` (form field name)
  - `value`: `string` (`YYYY-MM-DD` or range `from,to`, form value)
  - `mode`: `string` (`"single"` default, or `"range"`)
  - `placeholder`: `string` (defaults to `"pick a date"` or `"pick date range"`)
  - `format`: `string` (`"localized"` for formatted date, or empty for ISO)
  - `min`: `string` (`YYYY-MM-DD` lower bound)
  - `max`: `string` (`YYYY-MM-DD` upper bound)
  - `locale`: `string` (BCP 47 code)
  - `clearable`: `boolean` (default `false`, shows clear button when date is selected)
  - `disabled`: `boolean` (default `false`)
  - `week-starts-on`: `number` (`0` Sunday or `1` Monday)
  - `show-outside-days`: `boolean` (default `true`)
  - `close-on-select`: `boolean` (default `true`, auto-closes popover on selection)
  - `side`: `string` (`"bottom"` default)
  - `align`: `string` (`"start"` default)
- programmatic api:
  - `open()`, `close()`, `toggle()`, `clear()`
  - `.isOpen`, `.selectedDate`, `.range`, `.value`
- events:
  - `change`: dispatched when selection changes `{ value, from, to, date, dates }`
  - `input`: dispatched when value changes `{ value }`
  - `open`: dispatched when calendar popover opens
  - `close`: dispatched when calendar popover closes
- behavior: renders an interactive button trigger with calendar icon, date preview or placeholder, and clear action. opens `<d-calendar>` inside floating `<d-popover>`. auto-closes on date or range selection with spring transitions. full keyboard accessibility (`Enter`/`Space`/`ArrowDown` opens, `Escape` closes). fully form-associated (`static form = true`).

### d-checkbox

accessible, form associated checkbox switch:

```html
<d-checkbox text="enable two factor authentication" checked name="two_factor" value="1"></d-checkbox>
<d-checkbox text="accept terms" placement="before" name="terms"></d-checkbox>
```

- attributes/props:
  - `checked`: `boolean` (default `false`, reflected to form value)
  - `text`: `string` (default `"checkbox"`)
  - `name`: `string` (default `""`)
  - `value`: `string` (default `"on"`, form payload)
  - `placement`: `string` (default `"after"`, set `"before"` to place the toggle before the label)
  - `disabled`: `boolean` (default `false`, disables interaction and excludes from form submission)
- programmatic api:
  - `.checked`, `.text`, `.name`, `.value`, `.placement`, `.disabled` (getters and setters)
- events: `change`
- behavior: automatically synchronizes checked state, label text, and form submission values when toggled or updated programmatically.

### d-codeblock

syntax-highlighted code block with animated copy button, title header bar, language badge, line numbers, line highlighting, and diff mode:

```html
<!-- code block with title and language -->
<d-codeblock title="Server.js" lang="javascript" copyable>
const port = 3000;
function listen() {
  console.log("listening on", port);
}
</d-codeblock>

<!-- line highlighting and line numbers -->
<d-codeblock lang="javascript" highlight="2,4-5">
const a = 1;
const b = 2;
console.log(a + b);
</d-codeblock>

<!-- diff mode with floating copy button -->
<d-codeblock diff copyable>
 initial config
-const timeout = 1000;
+const timeout = 5000;
 return timeout;
</d-codeblock>
```

- attributes/props:
  - `title`: `string` (filename or title displayed in header)
  - `lang`: `string` (code language name)
  - `code`: `string` (programmatic code text)
  - `line-numbers`: `boolean` (default `true`)
  - `highlight`: `string` (comma-separated line numbers or ranges e.g. `"2,4-6"`)
  - `diff`: `boolean` (default `false`)
  - `copyable`: `boolean` (default `true`, copy to clipboard button)
  - `wrap`: `boolean` (default `false`, word wrap)
  - `max-height`: `string` (maximum scrollable height)
  - `code-font-size`: `string` (custom font size applied specifically to code text)
- programmatic api:
  - `copy()`: copies code text to clipboard and animates checkmark icon
  - `.codeText`: returns raw plain code string
- events:
  - `copy`: `{ text: string }`
- behavior: renders an integrated header bar with title and language badge when provided, or floating copy button in top-right when omitted. animates copy button with spring transition from `dstrn-copy-line` to `dstrn-check-line` for 2 seconds. includes safe zero-injection tokenizer. adopts server-rendered markup smoothly.

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
  - `open()`, `close()`, `toggle()`: controls popup panel visibility
  - `setHex(hex)`: sets hex color and updates visuals
  - `.value`, `.hex`, `.h`, `.s`, `.b`, `.a`, `.isOpen`
- events: `change` (detail: hex string), `input` (detail: hex string)
- behavior: interactive 2D color plane, hue bar, and alpha slider rendered inside an internal top layer `d-popover` (never clipped by `overflow: hidden`, flips automatically, closes on outside click or `Escape`).

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
  - `empty-text` / `emptyText`: `string` (default `"no results"`)
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
- events: `change` (detail: value), `open`, `close`
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
- static methods:
  - `dContextMenu.closeAll(except)`: closes all open context menus across document
- programmatic api:
  - `open(e)`: opens menu with optional pointer event coordinates
  - `close(e)`: closes menu
  - `toggle(e)`: toggles menu
- events: `click` (detail: clicked element), `open`, `close`
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

### d-form

ajax form container with automatic csrf token injection, validation, and error display:

```html
<d-form action="/register" method="POST" rules='{"email": "required|email"}' class="flex-column g-1">
  <input type="email" name="email" placeholder="email address">
  <input type="password" name="password" placeholder="password">
  <button type="submit" class="btn">register</button>
</d-form>
```

- attributes/props:
  - `action`: `string` (submission destination url)
  - `method`: `string` (default `"POST"`, supports `"GET"`, `"POST"`, `"PUT"`, `"DELETE"`)
  - `rules`: `string` (json string of client side validation rules)
  - `target`: `string` (selector for html response swapping, defaults to body)
  - `navigate`: `boolean` (default `false`, pushes to history on html response)
  - `force-reload`: `string` (`""`, `"true"`, `"all"`, `"success"`, `"error"`)
  - `callback`: `string` (global window function called on success)
- programmatic api:
  - `submit()`: submits form
  - `serialize()`: returns `FormData` instance
  - `showErrors(errors)`: displays inline field errors or top error banner
  - `clearErrors()`: clears active errors
  - `setDisabled(disabled)`: enables or disables form controls
- events: `d-submit`, `d-success`, `d-error`, `d-response`
- behavior: wraps internal form, automatically injects csrf token, disables inputs while submitting, evaluates client side rules, sends multipart when files are present, displays server validation errors inline, and handles redirect or reload flags.

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
      <a d-link href="/login" class="btn fs-08 w-100">sign in</a>
    </div>
  </d-hamburger>
</nav>
```

- attributes/props:
  - `opened`: `boolean` (default `false`)
  - `breakpoint`: `string` (default `"md"`, tokens: `sm`, `md`, `lg`, `xl`, `xxl`, `ultra` or arbitrary query like `"(max-width: 48em)"`)
  - `target`: `string` (selector of desktop container to clone links from and automatically toggle visibility)
  - `animation`: `string` (`slide-down`, `fade`, `slide-right`, `slide-left`, default `"slide-down"`)
  - `direction`: `string` (`top`, `right`, `left`, `bottom`, default `"top"`)
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

### d-key

tactile keyboard keycap badge and optional form associated shortcut recorder input:

```html
<d-key>cmd+k</d-key>
<d-key separator="">cmd,shift,p</d-key>
<d-key live>cmd+s</d-key>
<d-key editable name="shortcut" value="cmd+b"></d-key>
```

- attributes/props:
  - `value`: `string` (default `""`, form value; readable even when initialized via text content)
  - `name`: `string` (default `""`)
  - `separator`: `string` (default `"+"`, set `""` for adjacent keycaps without `+`)
  - `editable`: `boolean` (default `false`, turns into a click to record shortcut input)
  - `live`: `boolean` (default `false`, highlights keycaps when matching physical keys are pressed)
  - `placeholder`: `string` (default `"press key"`)
  - `disabled`: `boolean` (default `false`)
  - `clearable`: `boolean` (default `true`, clears recorded shortcut on `Backspace`/`Delete`)
- programmatic api:
  - `.value`, `.keys` (getter returning normalized token array), `clear()`
- events: `change` (detail: normalized shortcut string)
- behavior: splits `+` or `,` chords into platform aware keycap pills, adapts styling inside `.btn` and `.d-key-group`, and records live key combos in `editable` mode.

### d-loader

canvas arc spinner:

```html
<d-loader style="width: 2em; height: 2em;"></d-loader>
```

- attributes/props:
  - `color`: `string` (stroke color, falls back to `--accent` or text color)
- programmatic api: `destroy()`
- behavior: lightweight canvas spinner that automatically pauses animation when scrolled offscreen to conserve CPU and adapts to container dimensions.

### d-link

client side navigation link component and directive:

```html
<d-link href="/dashboard" class="nav-link">dashboard</d-link>
<a d-link href="/settings" target="#panel" mode="inner">settings</a>
```

- attributes/props:
  - `href`: `string` (destination url)
  - `target`: `string` (selectors for target elements to swap)
  - `src`: `string` (selectors for elements to extract from response)
  - `mode`: `string` (`"inner"`, `"outer"`, `"append"`, `"prepend"`, default `"inner"`)
  - `preserve-scroll`: `boolean` (default `false`, preserves scroll position)
  - `d-full-reload`: `boolean` (default `false`, forces standard page reload)
- programmatic api: `dSPA.navigate(url, options)`
- behavior: intercepts clicks on `<d-link>` and `<a d-link>`, validates same origin, ignores modifier clicks or external protocols, and fetches content asynchronously to perform dom swaps without full page reloads.

### d-menubar

desktop style application menu bar with top level triggers, instant hover switching, submenus, check/radio items, and keyboard navigation:

```html
<d-menubar>
  <div d-menu="file">
    <span slot="trigger">file</span>
    <div slot="content">
      <div d-item value="new-tab" shortcut="cmd+t">new tab</div>
      <div d-item value="new-window" shortcut="cmd+n">new window</div>
      <div d-item value="incognito" disabled>new incognito window</div>
      <hr>
      <div d-submenu="share">
        <span slot="trigger">share</span>
        <div slot="content">
          <div d-item value="email-link">email link</div>
          <div d-item value="copy-link" shortcut="cmd+c">copy link</div>
        </div>
      </div>
    </div>
  </div>
  <div d-menu="view">
    <span slot="trigger">view</span>
    <div slot="content">
      <div d-check value="show-bookmarks" checked shortcut="cmd+b">show bookmarks</div>
    </div>
  </div>
  <div d-menu="profiles">
    <span slot="trigger">profiles</span>
    <div slot="content">
      <div d-radio-group name="profile" value="work">
        <div d-radio value="personal">personal</div>
        <div d-radio value="work">work</div>
      </div>
    </div>
  </div>
</d-menubar>
```

- attributes/props:
  - `items`: `array` (default `null`, json menu structure alternative)
  - `disabled`: `boolean` (default `false`)
- declarative attributes: `[d-menu]`, `[slot="trigger"]`, `[slot="content"]`, `[d-item]`, `[d-check]`, `[d-radio-group]`, `[d-radio]`, `[d-submenu]`, `<hr>`
- programmatic api:
  - `.openMenu(index)`: open top menu by index
  - `.closeAllMenus()`: close open menus
  - `dMenubar.closeAll()`: close all menubars
- events:
  - `select`: detail `{ value, item, menu }`
  - `change`: detail `{ value, checked }` or `{ name, value }`
  - `open`: detail `{ menu, index }`
  - `close`: detail `{ menu, index }`
- behavior: renders top layer dropdown panels via `d-popover`, provides desktop instant hover switching across top triggers, and supports full keyboard arrow and shortcut navigation.

### d-modal

modal dialog container:

```html
<d-modal id="dialog" persistent>
  <div class="flex-column g-1">
    <h3>dialog title</h3>
    <p class="text-content-l">dialog content goes here</p>
    <button class="btn fs-08" onclick="this.closest('d-modal').close()">close</button>
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
- styling boundaries: `<d-modal>` automatically wraps content in `.d-modal-wrapper` (container background, border, radius, shadow) and `.d-modal-content`. avoid adding outer container classes like `bg-container-l bdr` inside the template to prevent duplicate borders. target `d-modal .d-modal-wrapper` in CSS to customize styling.

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

### d-navmenu

horizontal website navigation menu with shared morphing viewport panel and seamless drilldown integration with `d-hamburger`:

```html
<nav class="nav">
  <d-navmenu id="site-nav">
    <a d-link href="/" class="nav-link">overview</a>
    <div d-submenu="getting started">
      <span slot="trigger">getting started</span>
      <div slot="content" class="flex-column g-05" style="width: 280px;">
        <a d-link href="/docs" class="d-navmenu-sublink">
          <span class="d-navmenu-sublink-title">introduction</span>
          <span class="d-navmenu-sublink-desc">reusable frontend component library</span>
        </a>
      </div>
    </div>
  </d-navmenu>

  <!-- automatically converts d-navmenu into mobile drilldown drawer -->
  <d-hamburger target="#site-nav" breakpoint="md"></d-hamburger>
</nav>
```

- attributes/props:
  - `items`: `array` (default `null`, json structure alternative)
  - `delay`: `number` (default `150`, hover grace delay before closing)
  - `orientation`: `string` (default `"horizontal"`)
  - `align`: `string` (default `"center"`)
  - `disabled`: `boolean` (default `false`)
- declarative attributes: `[d-submenu]`, `[slot="trigger"]`, `[slot="content"]`, `.d-navmenu-sublink`
- programmatic api:
  - `.open(index)`: open submenu by index
  - `.close()`: close active submenu
  - `.getHamburgerItems()`: returns original element clones for mobile drawer
  - `dNavMenu.closeAll()`: close all instances
- events:
  - `open`: detail `{ index, title, item }`
  - `close`: detail `null`
- behavior: features an animated hover pill, shared morphing viewport panel with directional cross-slide (`translateX` in direction of travel) and `@starting-style` spring reveal. Pairs with `<d-hamburger target="#site-nav">` to auto hide on mobile and convert menus into drilldown panels.

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

### d-otp

form associated one time password and verification code input backed by a single native input:

```html
<d-otp length="6" separator="3" name="code"></d-otp>
<d-otp length="4" mask name="pin"></d-otp>
```

- attributes/props:
  - `length`: `number` (default `6`, 1 to 16)
  - `value`: `string` (default `""`, form value)
  - `name`: `string` (default `""`)
  - `pattern`: `string` (`"digits"` default, `"alphanumeric"`, `"alpha"`, or custom regex)
  - `separator`: `string` (default `""`, interval like `"3"` or comma indices like `"2,4"`)
  - `mask`: `boolean` (default `false`)
  - `disabled`: `boolean` (default `false`)
  - `invalid`: `boolean` (default `false`)
  - `autofocus`: `boolean` (default `false`)
  - `uppercase`: `boolean` (default `false`)
- programmatic api:
  - `focus()`, `clear()`, `error(message)`, `clearError()`
- events: `input`, `change`, `complete` (detail: sanitized string)
- behavior: supports SMS `one-time-code` autofill and paste sanitization, renders segmented slot groups with animated caret and character pop, and clears errors on next input.

### d-popover

top layer floating popover container (`popover="manual"`) with batched `translate3d` scroll tracking, viewport flipping, click/hover/manual triggers, and nested popover support:

```html
<d-popover side="bottom" align="start">
  <button slot="trigger" class="btn btn-outline">open popover</button>
  <div slot="content">
    <div class="d-popover-header">
      <h4 class="d-popover-title">title</h4>
      <p class="d-popover-desc">description</p>
    </div>
  </div>
</d-popover>
```

- attributes/props:
  - `opened`: `boolean` (default `false`)
  - `trigger`: `string` (`"click"` default, `"hover"`, `"manual"`)
  - `side`: `string` (`"bottom"` default, `"top"`, `"left"`, `"right"`, `"inline-start"`, `"inline-end"`)
  - `align`: `string` (`"start"` default, `"center"`, `"end"`)
  - `offset`: `number` (default `8`)
  - `match-width` / `matchWidth`: `boolean` (default `false`)
  - `hover-delay` / `hoverDelay`: `number` (default `120`)
  - `target`: `string` (default `""`, external anchor CSS selector)
  - `close-on-click-outside` / `closeOnClickOutside`: `boolean` (default `true`)
  - `close-on-escape` / `closeOnEscape`: `boolean` (default `true`)
  - `exclusive`: `boolean` (default `true`)
  - `unstyled`: `boolean` (default `false`)
- programmatic api:
  - `open()`, `close()`, `toggle()`, `reposition()`, `setAnchor(el)`, `getAnchor()`, `dPopover.closeAll(except)`
  - `.isOpened`, `.portal`, `.surface`
- events: `open`, `close`
- behavior: promotes content into the native top layer without clipping inside `overflow: hidden` ancestors, tracks scroll across nested containers via single pass rAF `translate3d`, and pairs cleanly inside `<d-buttongroup>`.

### d-progress

gpu composited progress bar indicator with optional label, percentage readout, and indeterminate mode:

```html
<d-progress value="56" label="upload progress" show-value></d-progress>
<d-progress indeterminate label="syncing assets"></d-progress>
```

- attributes/props:
  - `value`: `number` (default `0`)
  - `min`: `number` (default `0`)
  - `max`: `number` (default `100`)
  - `indeterminate`: `boolean` (default `false`)
  - `label`: `string` (default `""`)
  - `show-value` / `showValue`: `boolean` (default `false`)
  - `color`: `string` (default `""`, defaults to `var(--accent)`)
  - `size`: `string` (default `""`, defaults to `0.25em`)
- programmatic api:
  - `.value`, `.min`, `.max`, `.indeterminate`, `.label`, `.showValue`, `.color`, `.size`
  - `.ratio` (read only `0` to `1` float), `.percentage` (read only `0` to `100` integer)
- events: `complete` (detail: `value`, emitted once when reaching `max`)
- behavior: animates strictly with `transform: scaleX()`, pauses offscreen automatically, and plays a subtle completion sheen.

### d-radiobutton

form associated radio button with sibling group coordination, description subtext, placement control, and arrow key navigation:

```html
<d-radiobutton name="plan" value="free" text="starter" description="for personal projects" checked></d-radiobutton>
<d-radiobutton name="plan" value="pro" text="pro" description="unlimited workspaces"></d-radiobutton>
```

- attributes/props:
  - `checked`: `boolean` (default `false`, reflected to form value)
  - `name`: `string` (default `""`, coordinates peers sharing the same name)
  - `value`: `string` (default `"on"`, form payload when checked)
  - `text`: `string` (default `""`, falls back to inner text if omitted)
  - `description`: `string` (default `""`)
  - `placement`: `string` (default `"before"`, or `"after"`)
  - `disabled`: `boolean` (default `false`)
- programmatic api:
  - `.checked`, `.name`, `.value`, `.text`, `.description`, `.placement`, `.disabled`
- events: `change` (detail: `value`)
- behavior: unchecks sibling radios sharing the same `name` in the same form/document, supports arrow key navigation skipping disabled peers, and matches `d-checkbox` spring animation.

### d-questionnaire

interactive multi step questionnaire composing progress, radio buttons, checkboxes, text inputs, and keyboard shortcuts:

```html
<d-questionnaire name="survey">
  <div d-step name="role" title="what is your role?" type="single" required>
    <div d-option value="dev" text="developer" description="building backends"></div>
    <div d-option value="design" text="designer" description="crafting ui"></div>
  </div>
  <div d-step name="tools" title="tools you use" type="multiple" skippable>
    <div d-option value="git" text="git"></div>
    <div d-option value="docker" text="docker"></div>
  </div>
  <div d-step name="feedback" title="extra feedback" type="text" placeholder="type thoughts"></div>
</d-questionnaire>
```

- attributes/props:
  - `name`: `string` (default `""`)
  - `value`: `string` (default `"{}"`, form associated serialized json)
  - `current-step`: `number` (default `0`)
  - `auto-advance`: `boolean` (default `false`)
  - `show-progress`: `boolean` (default `true`)
  - `show-shortcuts`: `boolean` (default `true`)
  - `questions`: `string` (json array of question definitions)
- programmatic api:
  - `.next()`, `.prev()`, `.skip()`, `.goTo(index)`, `.reset()`, `.answers`, `.setAnswer(name, val)`
- events: `step` (`{ step, index, total, answers }`), `answer` (`{ name, value, answers }`), `submit` (`{ answers, json }`)
- behavior: numeric keys 1-9 choose options, Enter proceeds, Backspace/ArrowLeft navigates back, and invalid required steps trigger a shake animation.

### d-sidebar

application shell sidebar designed for zero layout thrashing with offcanvas sliding, icon collapse mode, and responsive mobile drawers:

```html
<d-sidebar id="app-sidebar" collapsible="icon" variant="sidebar" persist="app_sidebar">
  <div slot="header">
    <span class="d-sidebar-label">workspace</span>
  </div>
  <div d-sidebar-group>
    <div d-sidebar-group-label>platform</div>
    <div d-sidebar-menu>
      <a d-sidebar-item href="/dashboard" active>
        <i class="dstrn-dashboard-line"></i>
        <span class="d-sidebar-label">dashboard</span>
      </a>
      <div d-sidebar-item d-collapsible>
        <div d-sidebar-item-trigger>
          <i class="dstrn-folder-line"></i>
          <span class="d-sidebar-label">projects</span>
          <i class="dstrn-down-small-line d-sidebar-chevron"></i>
        </div>
        <div d-sidebar-sub>
          <a d-sidebar-sub-item href="/projects/active">active</a>
        </div>
      </div>
    </div>
  </div>
  <div slot="footer">
    <span class="d-sidebar-label">profile</span>
  </div>
  <div d-sidebar-rail></div>
</d-sidebar>
```

- attributes/props:
  - `side`: `string` (`"left"` or `"right"`, default `"left"`)
  - `collapsible`: `string` (`"offcanvas"`, `"icon"`, or `"none"`, default `"offcanvas"`)
  - `variant`: `string` (`"sidebar"`, `"floating"`, or `"inset"`, default `"sidebar"`)
  - `opened`: `boolean` (default `true`)
  - `breakpoint`: `string` (default `"md"`, switches to mobile overlay)
  - `shortcut`: `string` (default `"cmd+b"`)
  - `persist`: `string` (default `""`, localstorage key)
- programmatic api:
  - `.open()`, `.close()`, `.toggle()`, `.isOpen`, `.isMobile`
- events: `open` (`{ mobile }`), `close` (`{ mobile }`)
- behavior: uses static layout spacer with fixed css variable widths and gpu transforms to eliminate reflows; in icon mode hides labels and attaches tooltips; clicking rail or external `[d-sidebar-trigger]` toggles sidebar; converts to offcanvas drawer below breakpoint.

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
- programmatic api: `.value`, `.placeholder`, `.icon`, `.disabled`, `.readonly`, `.focus()`, `.blur()`
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
  - `textcolor` / `textColor`: `string` (default `""`, auto contrast against background)
  - `textactivecolor` / `textActiveColor`: `string` (default `""`, auto contrast against active background)
  - `bgcolor` / `bgColor`: `string` (default `"var(--container-l)"`)
  - `hovercolor` / `hoverColor`: `string` (default `""`, theme adaptive subtle hover)
- programmatic api:
  - `.selected`: getter returns active element, setter accepts index (number/string) or button element
  - `.value`: selected index number
- events: `change` (detail: selected index number)
- behavior: renders interactive segmented options with animated hover and selection transitions, and supports initial selection via the `default` attribute or numeric value.

### d-tooltip

lightweight popup displaying helpful context and keyboard shortcuts on hover or keyboard focus, built upon the top layer `d-popover` positioning engine:

```html
<d-tooltip text="save document" shortcut="cmd+s" side="top">
  <button class="btn btn-outline">save</button>
</d-tooltip>

<d-tooltip text="action unavailable" side="right">
  <button class="btn btn-outline" disabled>disabled action</button>
</d-tooltip>

<d-tooltip>
  <button slot="trigger" class="btn btn-gray"><i class="dstrn-add-line"></i></button>
  <span slot="content">custom tooltip markup</span>
</d-tooltip>
```

- attributes/props:
  - `text`: `string` (default `""`, text label)
  - `shortcut`: `string` (default `""`, rendered automatically as `<d-key>` glyphs)
  - `side`: `string` (`"top"` default, `"bottom"`, `"left"`, `"right"`, `"inline-start"`, `"inline-end"`)
  - `align`: `string` (`"center"` default, `"start"`, `"end"`)
  - `delay`: `number` (default `150`ms hover grace delay)
  - `offset`: `number` (default `6`px gap)
  - `disabled`: `boolean` (default `false`, disables reveal)
- programmatic api:
  - `.open()`, `.close()`, `.toggle()`, `.setAnchor(el)`, `.getAnchor()`, `.text`, `.shortcut`, `.disabled`, `.isOpened`
  - `dTooltip.closeAll(except)`: static method closing all active tooltips
- events: `open`, `close`
- behavior: renders in the browser top layer via `d-popover`, immune to `overflow: hidden` clipping, reliably shows on disabled trigger buttons, and integrates with keyboard `focusin` / `focusout`.

### d-virtual-list

lightweight DOM virtualization component skipping layout and paint for offscreen elements using CSS content-visibility and automatic bottom threshold detection:

```html
<d-virtual-list item-size="64px" buffer="800px 0px 800px 0px" threshold="600">
  @foreach ($items as $item)
    <div class="p-3 bg-container mb-2 rounded">{{ $item->title }}</div>
  @endforeach
</d-virtual-list>
```

- attributes/props:
  - `item-size` / `itemSize`: `string` (default `"52px"`, estimated item height fallback)
  - `buffer`: `string` (default `"600px 0px 600px 0px"`, intersection observer root margin)
  - `threshold`: `number` (default `500`, scroll distance in pixels to trigger reachbottom)
  - `cooldown`: `number` (default `250`, minimum delay between reachbottom triggers in milliseconds)
  - `scroll-parent` / `scrollParent`: `string` (default `""`, optional CSS selector for custom scroll container)
- programmatic api:
  - `.itemSize`, `.buffer`, `.threshold`, `.cooldown`, `.scrollParent`
  - `.checkBottom()`: manually evaluates proximity to bottom
- events: `reachbottom` (detail: `{ distance, threshold }`, dispatched when user scrolls near the end of the list)
- behavior: dynamically hides offscreen items with `content-visibility: hidden` while retaining intrinsic sizing to avoid layout shifts, and notifies parent listeners when approaching bottom for infinite scrolling.

---

## zero build automatic asset compilation

`storage/framework/public/js/dstrn.js` and `storage/framework/public/css/dstrn.css` are automatically compiled on server start and regenerated automatically by the framework runtime watcher whenever any change in `public/css/components/` or `public/js/components/` is detected (served at `/js/dstrn.js` and `/css/dstrn.css`).

strictly enforce these rules:

- never edit compiled `dstrn.js` or `dstrn.css` directly; any edits are wiped when bundles regenerate
- never search for or run build, rebuild, or compile commands (e.g. `npm run build` or `dstrn build`); the compiler runs automatically
- author custom or override components in `public/js/components/<ComponentName>.js`
- author custom component css in `public/css/components/<ComponentName>.css` or application styles in `public/css/app.css`
- elements with `btn` and `btn-` classes already have inline-flex layout, padding, and border radius built in; do not attach redundant padding, border radius, or flex classes. there are no button size classes like `.btn-sm`; apply `fs-` font size classes (e.g. `fs-08`, `fs-09`) to scale buttons and components proportionally because all dimensions use em units

## overriding built in components

all built in UI component behaviors can be patched or overridden by placing a matching file in `public/js/components/` (e.g. `public/js/components/dCheckbox.js`).

the compiler automatically replaces the core component with the user implementation when building `dstrn.js`.

### dComponent is immutable

`dComponent` is the foundational base class for all custom elements and reactivity. it cannot be overridden or shadowed by user code. the compiler always enforces that the core `dComponent` base class is bundled first so all components inherit properly.

## custom dComponent lifecycle and reconnection behavior

when building custom UI components that extend `dComponent`, understand the lifecycle distinctions:

- `mount()`: called exactly once when the component is first mounted into the DOM. use this hook for initial template rendering, DOM caching, and one time setup.
- `mountSync()`: forces immediate synchronous initialization of attributes, template, and mount hooks without waiting for requestAnimationFrame. use when creating child custom elements programmatically inside mount(). also available statically as `dComponent.mountSync(element)`.
- `connected()`: called whenever the component attaches to the DOM. if a component has already initialized and is reconnected (such as moving within the DOM tree or during client side SPA transitions), `connected()` runs while `mount()` is skipped. use this hook to restore active subscriptions or listeners.
- `destroy()`: called when the component is removed from the DOM. `dComponent` automatically clears registered intervals, timeouts, animation frames, and DOM listeners, and invokes `destroy()` for custom cleanup.
- `syncForm()`: called automatically during microtask update flushes for form-associated components. can be invoked manually or overridden to customize ElementInternals form value serialization.
- `reflectAttribute(name, value)`: immediately reflects a property or attribute value to the host DOM element safely without triggering recursive `attributeChangedCallback` execution.
- `isProgrammaticUpdate`: read-only boolean getter indicating if property mutations are currently occurring via property setters and `onPropChanged()`.
- `update()`: schedules an asynchronous reactive rerender when state properties change.