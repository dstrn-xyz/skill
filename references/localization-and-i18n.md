# localization and internationalization

dframework includes built in multi locale support with dictionary caching, dynamic URL interpolation, client side language switching, and first class Japanese furigana ruby annotations.

## dictionary structure

translation files live in `lang/{locale}/{filename}.json`:

```
lang/
├── en/
│   ├── auth.json
│   ├── common.json
│   └── messages.json
└── ja/
    ├── auth.json
    ├── common.json
    └── messages.json
```

example translation dictionary:

```json
// lang/en/common.json
{
  "welcome": "welcome back, :name!",
  "actions": {
    "save": "save changes",
    "cancel": "cancel"
  }
}
```

```json
// lang/ja/common.json
{
  "welcome": "おかえりなさい、:nameさん！",
  "actions": {
    "save": "保存[ほぞん]",
    "cancel": "キャンセル"
  }
}
```

## switching and reading locales (Locale facade)

always use the `Locale` facade. never write to `locale` cookies or parse request headers manually.

```javascript
import { Locale } from 'dframework';

// set locale permanently for user (writes cookie and session)
Locale.set('ja');

// set locale for current request only (without modifying cookie or session)
Locale.set('fr', true);

// get active locale for current request
const current = Locale.get(); // 'ja'

// get default fallback locale configured in config/app.js
const defaultLocale = Locale.getDefault(); // 'en'
```

### locale controller pattern

```javascript
// controllers/LocaleController.js
import { Locale } from 'dframework';

export default class LocaleController {
  async switch(req) {
    const { lang } = req.params;
    
    // validate against supported locales
    if (!['en', 'ja'].includes(lang)) {
      return abort(400, 'unsupported locale');
    }

    Locale.set(lang);
    return back();
  }
}
```

## translating strings

### in javascript (controllers, models, jobs)

use the global `t()` helper:

```javascript
// basic translation
const welcome = await t(req, 'common.welcome', { name: user.name });

// nested keys
const btnText = await t(req, 'common.actions.save');

// with fallback text
const label = await t(req, 'common.missing_key', 'fallback label');
```

- key format: first segment is filename without `.json`, remaining segments form nested path in json object
- `t()` is asynchronous because it reads files from disk on initial request (cached in memory afterwards)

### in view templates (.d files)

use the `@t()` directive in views. request object is injected automatically:

```html
<h1>@t('common.welcome', { name: user.name })</h1>
<button>@t('common.actions.save')</button>
```

## japanese furigana (ruby annotations)

dframework provides first class support for japanese furigana annotations.

> [!IMPORTANT]
> always ask the user whether japanese text should use @ruby directives and furigana bracket annotations (`漢字[かんじ]`) or just plain japanese text without annotations.

### inline notation in translation json

add bracket notation `漢字[かんじ]` inside translation strings:

```json
// lang/ja/messages.json
{
  "greet": "東京[とうきょう]へようこそ、:nameさん！"
}
```

```javascript
await t(req, 'messages.greet', { name: '花子' });
// outputs: "<ruby>東京<rt>とうきょう</rt></ruby>へようこそ、花子さん！"
```

### ruby helper and view directive

```javascript
// in javascript
ruby('漢字', 'かんじ'); // "<ruby>漢字<rt>かんじ</rt></ruby>"
ruby('日本[にほん]語[ご]'); // "<ruby>日本<rt>にほん</rt></ruby><ruby>語<rt>ご</rt></ruby>"
```

```html
<!-- in view templates -->
<h1>@ruby('漢字', 'かんじ')</h1>
<p>@ruby('日本[にほん]語')</p>
```

## locale detection order

dframework resolves `req.locale` automatically during request pipeline boot:

1. `locale` cookie (checked first; must match alphanumeric locale code pattern)
2. `Accept-Language` HTTP header (primary language code parsed automatically)
3. default locale configured in `config/app.js` (`locale` key)

## translation linting and parity (dstrn lang:lint)

to verify translation consistency and discover missing keys across locales, run:

```bash
dstrn lang:lint
```

the linter performs comprehensive checks:
- validates key parity between the default locale and all translation dictionaries in `lang/`
- scans `.d` view templates and javascript files to confirm all `@t('...')` and `t(req, '...')` keys exist
- flags missing translation keys, unused dictionary keys, and syntax errors

## caching and hot reload

- translation json files are cached in memory after first load
- in local development (`app.env = 'local'`), file watchers automatically invalidate cache on file edit without server restart
- in production, files are cached in memory for process lifetime with zero filesystem overhead