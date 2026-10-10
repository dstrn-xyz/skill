# views and templating

dframework uses a compiled server side template engine with `.d` file extension, located in the `views/` directory. templates compile to efficient javascript functions.

## referencing views

view paths use dot notation matching the directory tree under `views/`:

- `views/home.d` is referenced as `'home'`
- `views/app/users/index.d` is referenced as `'app.users.index'`
- `views/layouts/app.d` is referenced as `'layouts.app'`

## zero config automatic compilation

view compilation is handled 100% automatically by the framework runtime (`ViewEngine`). templates are compiled in memory on first access and hot reloaded automatically in local development.

never write custom view compilation scripts, run manual compiler tools, or modify core framework rendering logic. there is zero build step or manual compilation script needed for views.

## directive reference

| directive | description |
| :-------- | :---------- |
| `@extends('layout')` | extends a parent layout template |
| `@section('name') ... @endsection` | defines a section of content for a layout placeholder |
| `@yield('name')` | renders a named section placeholder in a layout |
| `@include('partial')` | renders a partial view inheriting parent variables |
| `@if (expr) ... @elseif (expr) ... @else ... @endif` | conditional rendering |
| `@foreach (arr as item) ... @endforeach` | array iteration |
| `@forelse (arr as item) ... @empty ... @endforelse` | array iteration with empty fallback |
| `@for (init; cond; step) ... @endfor` | loop iteration |
| `@pagination(paginator, spa, target)` | renders pagination navigation controls |
| `@js ... @endjs` | executes localized presentation compute in template |
| `@json(variable)` | serializes server data to json in script tags or attributes |
| `@t('key', params)` | renders translated string |
| `@ruby('text', 'rt')` | renders pronunciation ruby tags |

## basic output and escaping

- `{{ expression }}`: html escaped output (safe against xss)
- `{!! rawHtml !!}`: unescaped raw html output
- `@json(variable)`: serializes variable to json in javascript context

```html
<!-- escaped -->
<div class="user-name">{{ user.name }}</div>

<!-- unescaped raw html -->
<div class="content">{!! post.renderedBody !!}</div>

<!-- embedded in javascript -->
<script>
  const initialData = @json(user);
</script>
```

## control structures

```html
<!-- conditionals -->
@if (user && user.isAdmin)
  <span class="badge">admin</span>
@elseif (user)
  <span class="badge">member</span>
@else
  <a href="/login" class="link">login</a>
@endif

<!-- loops ($index, $first, and $last are automatically available) -->
@foreach (posts as post)
  <article class="flex-column g-1">
    <h3>{{ post.title }}</h3>
    <p>{{ post.excerpt }}</p>
    @if ($last)
      <span>end of feed</span>
    @endif
  </article>
@endforeach

<!-- loops with empty state fallback ($index, $first, and $last are available) -->
@forelse (notifications as note)
  <div class="notification-item">{{ note.text }}</div>
@empty
  <div class="empty-state">no notifications found</div>
@endforelse

<!-- standard for loop -->
@for (let i = 0; i < count; i++)
  <span class="dot"></span>
@endfor
```

## layouts and sections

layouts establish reusable page wrappers with `@yield` placeholders. child views extend layouts with `@extends` and supply content with `@section`.

```html
<!-- views/layouts/app.d -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>{{ title || 'dframework app' }}</title>
</head>
<body class="flex-column min-h-100svh">
  @include('partials.nav')

  <main class="flex-1 p-2">
    @yield('content')
  </main>

  @include('partials.footer')
</body>
</html>
```

```html
<!-- views/app/users/index.d -->
@extends('layouts.app')

@section('content')
  <div class="flex-column g-2">
    <h1>users</h1>
    @foreach (users as user)
      <div class="user-row">{{ user.name }} ({{ user.email }})</div>
    @endforeach

    @pagination(users)
  </div>
@endsection
```

## pagination

the view engine integrates with the framework `Paginator` class returned by model `.paginate()` methods. paginators are directly iterable in loops and support automatic or custom markup:

```html
<!-- automatic controls with standard navigation (standalone directive) -->
@pagination(users)

<!-- or direct unescaped links output -->
{!! users.links() !!}

<!-- opt-in dspa navigation -->
@pagination(users, true)

<!-- dspa targeting specific container -->
@pagination(users, true, '#users-container')

<!-- custom html with block scoped variables (prev, next, current, last, total, perPage, paginator) -->
@pagination(users)
  <div class="custom-pagination">
    <span>page {{ current }} of {{ last }} ({{ total }} records)</span>
    @if (prev)
      <a href="{{ prev }}" class="btn">previous</a>
    @endif
    @if (next)
      <a href="{{ next }}" class="btn">next</a>
    @endif
  </div>
@endpagination
```

## includes and pure ui partials

use `@include('partial.name')` for pure ui and rendering components (cards, list rows, headers, navigation widgets). `@include` takes only the template name and automatically inherits all variables from the parent view scope. do not pass data objects as a second argument:

```html
@include('partials.header')
@include('app.components.card')
```

- if a widget requires heavy client javascript, canvas rendering, audio loops, or carousels, author a custom `dComponent` instead

## server side javascript compute blocks

heavy lifting (data loading, domain calculations, DB queries) belongs in controllers. 100% of design and frontend rendering compute belongs in localized `@js` blocks in the template:

```html
@js
  // format data and compute presentation tiers near usage point
  const isOptimal = ping < 50;
  const statusColor = isOptimal ? 'text-green' : 'text-red';
  const statusBg = isOptimal ? 'background: rgba(21, 128, 61, 0.2)' : 'background: rgba(220, 38, 38, 0.2)';
  const formattedDate = new Date(timestamp).toLocaleDateString('en-GB', { day: '2-digit', month: 'short' });
@endjs

<div class="flex-row align-center g-1 p-1 bdr {{ statusColor }}" style="{{ statusBg }}">
  <span>{{ ping }}ms</span>
  <span class="fs-08 text-content">{{ formattedDate }}</span>
</div>
```

- keep `@js` blocks near where variables are consumed
- avoid redundant recalculations across template loops

## localization and furigana

- `@t('file.key')`: sync translation in views
- `@t('file.key', { name: user.name })`: with parameter interpolation
- `@ruby('漢字', 'かんじ')`: renders `<ruby>漢字<rt>かんじ</rt></ruby>`
- `@ruby('日本[にほん]語[ご]')`: parses inline bracket annotations into ruby tags

```html
<h1>@t('common.welcome', { name: user.name })</h1>
<p>@ruby('日本語[にほんご]')</p>
```

## forms and automatic csrf

the template compiler automatically injects the hidden `_csrf` input into every `<form>` tag and adds the `<meta name="csrf-token">` tag into `<head>`.

```html
<form action="/login" method="POST" class="flex-column g-1">
  <!-- _csrf input is injected automatically by compiler -->
  <input type="email" name="email" value="{{ old('email') }}" required>
  <input type="password" name="password" required>
  <button type="submit">login</button>
</form>
```