# frontend design system

dframework applications follow a polished, dark mode first visual hierarchy built on custom utility classes, 3-tier surface elevation, micro typography, and clean separation between controller logic and localized view compute.

## 3-tier surface elevation

interfaces establish visual depth using three surface tiers:

- `bg-container-d`: deep backdrop canvas (root application background)
- `bg-container`: structural panels, sidebars, base cards, and navigation wrappers
- `bg-container-l`: elevated elements, top headers, metric tiles, hover states, and active items

```html
<!-- layered surface hierarchy -->
<main class="bg-container-d flex-column w-100 h-100svh p-1 g-1">
  <div class="bg-container bdr p-2 flex-column g-1">
    <div class="bg-container-l bdr p-1 flex-row justify-between align-center">
      <span class="text-content-l bold">elevated header</span>
      <span class="text-content fs-08">status indicator</span>
    </div>
  </div>
</main>
```

## typography and text hierarchy

text contrast reflects content priority:

- `text-content-l`: primary high contrast text (headings, active titles, key values)
- `text-content`: secondary muted text (labels, descriptions, metadata)
- `text-white`: high emphasis labels and buttons
- `text-accent` / `text-accent-l`: primary brand accent elements
- `text-green` / `text-red` / `text-yellow`: status indicators, paired with translucent 20% alpha backgrounds

### typographic scale

combine fractional font sizing with uppercase tracking and truncation:

```html
<!-- section header with subtext -->
<div class="flex-column g-025">
  <span class="text-content-l fs-115 bold uppercase">system health</span>
  <span class="text-content fs-08">real time latency metrics</span>
</div>

<!-- metric card -->
<div class="bg-container-l bdr p-1 flex-column g-025 flex-1">
  <span class="fs-06 text-content uppercase overflow-ellipsis">response time</span>
  <div class="flex-row align-end g-05 pt-05">
    <span class="fs-15 bold text-content-l">42<span class="fs-05 text-content">ms</span></span>
    <span class="fs-06 text-green uppercase mb-025">optimal</span>
  </div>
</div>
```

- size classes: `.fs-05`, `.fs-06`, `.fs-07`, `.fs-08`, `.fs-09`, `.fs-1`, `.fs-115`, `.fs-15`, `.fs-2`, `.fs-3`
- text helpers: `.bold`, `.thin`, `.uppercase`, `.text-center`, `.overflow-ellipsis`

## spacing, borders, and interactive transitions

- micro-gaps: `.g-025` (0.25em), `.g-05` (0.5em), `.g-075` (0.75em), `.g-1` (1em), `.g-15` (1.5em), `.g-2` (2em)
- borders: `.border-s` (thin border), `.border-m`, `.border-bottom-s`, `.border-top-s`
- border radius: `.bdr` (0.5em), `.bdr-05`, `.bdr-08`, `.bdr-circle` (50%)
- interaction: `.pointer`, `.hover:bg-container-l`, `.tr-03` (0.3s transition), `.btn-transparent`, `.shadow-s`, `.shadow-hover-s`

```html
<!-- interactive action tile -->
<div class="relative flex-center flex-column bdr-08 border-s pointer bg-container p-1 hover:bg-container-l tr-03 g-05 flex-1">
  <i class="dstrn-waveform-path fs-15 text-content-l"></i>
  <span class="text-content text-center fs-06 uppercase">equalizer</span>
</div>
```

## app shell layout pattern

modern dframework applications use a fixed full-viewport height shell with internal scrollable regions:

```html
<!-- views/layout/app.d -->
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <title>@yield('title')</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <link rel="stylesheet" href="{{ asset('css/app.css') }}">
</head>
<body>
  <main class="bg-container-d flex-column w-100 h-100svh p-1 g-1">
    <!-- main area: sidebar navigation + content drawer -->
    <div class="main-container relative flex-row w-100 g-1" style="height: calc(100% - 7em);">
      <aside class="navigation-container relative h-100 flex-column" style="width: 4em;">
        @include('app.components.navigation')
      </aside>

      <d-drawer opened direction="bottom" class="content-container relative h-100 bdr overflow-hidden flex-1">
        <div id="spa-container" class="relative h-100 w-100 flex-column bg-container overflow-y-auto overflow-x-hidden">
          @yield('content')
        </div>
      </d-drawer>
    </div>

    <!-- persistent bottom widget bar -->
    <footer class="extra-container relative flex-row w-100 g-1" style="height: 6em;">
      @include('app.components.player')
    </footer>
  </main>
</body>

<script defer>
  dSPA_DEFAULT_TARGETS = ['#spa-container'];
  dSPA.transitions = {
    beforeSwap(oldNode, meta) {
      const drawer = select('d-drawer');
      return drawer?.close();
    },
    afterSwap(newNode, meta) {
      const drawer = select('d-drawer');
      return drawer?.open();
    }
  };
</script>
</html>
```

## responsive navigation with d-hamburger

for top navigation and responsive headers, pair desktop link containers with `<d-hamburger>`:

```html
<!-- views/partials/nav.d -->
<nav class="nav bg-container bdr border-s px-1 py-05 flex-row justify-between align-center">
  <a d-link href="/" class="flex-row align-center g-05">
    <span class="bold text-content-l">app name</span>
  </a>

  <!-- desktop container: automatically managed by d-hamburger -->
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

- `<d-hamburger>` watches viewport width via `breakpoint="md"` and automatically hides `#nav-links` on mobile while revealing the hamburger button
- child elements with `d-submenu="title"` automatically expand into drilldown sliding panels with back button navigation
- never author custom javascript menu toggles or ad hoc drawer hacks for navigation bars

## separation of concerns for compute

follow strict boundaries for computation:

1. **controllers**: handle 100% of data retrieval, database queries, authentication checks, and business logic. controllers never generate HTML strings or style definitions.
2. **view** **`@js`** **blocks**: handle 100% of design and frontend rendering compute (e.g. status badge color mapping, date formatting, chart series mapping, tier calculation). `@js` blocks must be placed near where they are consumed and remain non redundant.
3. **`@include`** **partials**: use for pure UI and rendering components (cards, list rows, headers, navigation widgets); `@include('partial.name')` takes only the template name and automatically inherits all variables from the parent view scope.
4. **custom** **`dComponent`** **custom elements**: use for widgets requiring heavy javascript, canvas rendering, audio/timer animation loops, or carousels (`d-graph`, `d-waveform`, `d-carousel`).

### localized view compute pattern

```html
<!-- views/app/status.d -->
@extends('layout.app')

@section('content')
<div class="flex-column h-100 w-100 bg-container">
  @js
    // pure design and frontend compute placed directly near template usage
    const isDegraded = systemHealth.errorRate > 0.05;
    const badgeColor = isDegraded ? 'text-red' : 'text-green';
    const badgeBg = isDegraded ? 'background: rgba(220, 38, 38, 0.2)' : 'background: rgba(21, 128, 61, 0.2)';
    const statusText = isDegraded ? 'degraded performance' : 'all systems operational';
  @endjs

  <div class="bg-container-l bdr p-1 flex-row align-center g-1">
    <div class="flex-center bdr-circle p-1 {{ badgeColor }}" style="{{ badgeBg }}; width: 3em; height: 3em;">
      <i class="dstrn-check fs-15"></i>
    </div>
    <div class="flex-column">
      <span class="fs-115 bold {{ badgeColor }} uppercase">{{ statusText }}</span>
      <span class="fs-08 text-content-l">{{ systemHealth.activeNodes }} active streaming nodes</span>
    </div>
  </div>
</div>
@endsection
```

## extra design requirements
- do not over use shadows, keep it contrained and clean
- do not generate AI-like ticking badges unless explicitly asked
- do not generate standard, generic AI-like designs