# models, database, and migrations

dframework features an active record ORM, fluent query builder, and migration runner supporting mysql, postgresql, and sqlite.

## models mandate: prioritize models over raw DB access

always prioritize active record `Model` subclasses for application data access, entity relationships, business logic, and mutations over raw `DB.table()` or `DB.query()`.

### why models are strongly recommended

1. **domain encapsulation and hydration**: queries executed through `Model` (`User.where('status', 'active')`, `User.find(id)`, `User.all()`) return hydrated model instances carrying relationships, instance methods, getters, and setters.
2. **relationship traversal and eager loading**: models provide declarative `.with('relation')` and `.load('relation')` eager loading that prevents n+1 query problems with transparent in memory caching.
3. **automatic serialization security**: models automatically hide sensitive fields (`password`, `token`, `secret`, `api_key`, `remember_token`, and custom `static hidden = []`) during `toJSON()` serialization, preventing accidental credential exposure in API responses.
4. **active record convenience**: provides concise single line methods like `Model.find(id)`, `Model.findByEmail('user@example.com')`, `Model.firstOrCreate(attributes, values)`, `Model.updateOrCreate(attributes, values)`, `Model.paginate(20)`, and instance `.save()`, `.update()`, and `.delete()`.
5. **full query builder parity**: `Model` proxies 100% of fluent query builder methods (`join`, `leftJoin`, `whereBetween`, `whereExists`, `when`, `unless`, `selectSub`, `count`, `sum`), retaining maximum querying flexibility with model hydration.

reserve raw `DB.table()` and `DB.query()` strictly for reporting aggregations spanning arbitrary non-model views or low level maintenance tasks.

## debugging and inspecting models with tinker

never create custom scratch test scripts to query models or database state. use `dstrn tinker` with inline code directly from the command line:

```bash
dstrn tinker "await User.count()"
dstrn tinker "await User.with('profile').first()"
dstrn tinker --execute="await DB.table('users').where('role', 'admin').get()"
```

all models in `models/` and framework facades are preloaded automatically into global scope with top level await support.

## defining models

models live in `models/` and extend `Model` from `'dframework'`.

```javascript
// models/User.js
import { Model } from 'dframework';
import Post from './Post.js';
import Profile from './Profile.js';

export default class User extends Model {
  static table = 'users'; // optional (defaults to lowercase plural of class)
  static primaryKey = 'id'; // optional (auto detected from schema, composite keys supported)
  static keyType = 'uuid'; // optional (handles uuid generation on create and save)
  static fillable = ['name', 'email', 'metadata']; // allowed mass assignable attributes
  static guarded = ['is_admin']; // protected attributes
  static hidden = ['password', 'secret', 'api_key']; // extra attributes hidden from toJSON()
  static casts = {
    metadata: 'json',
    tags: 'array',
    is_active: 'boolean',
    login_count: 'int',
    joined_at: 'datetime'
  };

  profile() {
    return this.hasOne(Profile, 'user_id');
  }

  posts() {
    return this.hasMany(Post, 'user_id');
  }
}
```

- call `user.getKey()` to retrieve the primary key value regardless of column name
- default auto hidden fields during `toJSON()` serialization: `password`, `token`, `secret`, `api_key`, `remember_token`

### mass assignment protection (`static fillable` and `static guarded`)

models protect against mass assignment vulnerabilities during `create()` and `update()`. by default, when neither `fillable` nor `guarded` is defined (or when `guarded = []`), all attributes are fillable. you may restrict assignable fields using `static fillable` as an allowlist, or `static guarded` as a denylist (or `['*']` to protect all attributes). creating a new model with a predefined primary key and calling `save()` attempts an insert and throws a `DiagnosticError` on collision:

```javascript
export default class User extends Model {
  static fillable = ['name', 'email'];
  // or: static guarded = ['is_admin', 'role'];
}
```

### model lifecycle hooks

models support mutation lifecycle hooks returning `false` to abort the operation:

- `creating()` / `created()`: before/after record creation
- `updating()` / `updated()`: before/after record update
- `saving()` / `saved()`: before/after save (both insert and update)
- `deleting()` / `deleted()`: before/after record deletion

### attribute casting (`static casts`)

models support automatic attribute casting on hydration and persistence:

| cast                            | behavior                                                                                                                                                                                                                                                                  |
| :------------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `json`, `array`, `object`       | stringified json in database is parsed into javascript objects/arrays on model hydration; javascript objects/arrays are serialized to json strings during `save()` and `create()`. in-place mutations (`user.metadata.theme = 'dark'`) persist automatically on `save()`. |
| `boolean`, `bool`               | converts `1`, `'1'`, `'true'`, `true` to `true`, and `0`, `'0'`, `'false'`, `false` to `false`.                                                                                                                                                                           |
| `int`, `integer`                | parses integer values via `parseInt()`.                                                                                                                                                                                                                                   |
| `float`, `double`, `real`       | parses floating point values via `parseFloat()`.                                                                                                                                                                                                                          |
| `date`, `datetime`, `timestamp` | converts strings to `Date` objects on hydration and formats to standard sql timestamp format on save.                                                                                                                                                                     |
| `string`                        | casts values to primitive strings.                                                                                                                                                                                                                                        |

subclasses inherit and merge `static casts` from parent classes automatically.

### static model methods

| method                                      | arguments                                                                                                  | returns                                                                  |
| :------------------------------------------ | :--------------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------- |
| `Model.all()`                               | none                                                                                                       | `Promise<Model[]>` (hydrated instances, `[]` when no rows)               |
| `Model.query()`                             | none                                                                                                       | `ModelQueryBuilder` (fresh query builder instance)                       |
| `Model.find(id)`                            | primary key value, or `{ pk1, pk2 }` for composite keys                                                    | `Promise<Model\|null>`                                                   |
| `Model.findBy<Column>(value)`               | value                                                                                                      | `Promise<Model\|null>` (magic finder e.g. `User.findByEmail('a@b.com')`) |
| `Model.first(where?)`                       | optional `{ column: value }` object                                                                        | `Promise<Model\|null>`                                                   |
| `Model.firstWhere(column, op?, val?)`       | column name, operator/value, optional value                                                                | `Promise<Model\|null>`                                                   |
| `Model.latest(count?, column?)`             | `null` or number; optional column name                                                                     | `null` count: `Promise<Model\|null>`; numeric count: `Promise<Model[]>`  |
| `Model.where(column, operator, value)`      | column/operator/value, object, or closure                                                                  | `ModelQueryBuilder` (chainable, thenable, async iterable)                |
| `Model.whereRaw(sql, bindings)`             | raw sql where string, bindings array                                                                       | `ModelQueryBuilder` (chainable)                                          |
| `Model.orWhereRaw(sql, bindings)`           | raw sql or where string, bindings array                                                                    | `ModelQueryBuilder` (chainable)                                          |
| `Model.having(col, op, val)`                | column, operator, value                                                                                    | `ModelQueryBuilder` (chainable)                                          |
| `Model.havingRaw(sql, bindings)`            | raw sql having string, bindings array                                                                      | `ModelQueryBuilder` (chainable)                                          |
| `Model.orderByRaw(sql)`                     | raw sql order by expression string                                                                         | `ModelQueryBuilder` (chainable)                                          |
| `Model.with('rel1', 'rel2')`                | relation names, array, closure constraints `('posts', q => q.where(...))`, or object `{ rel: (q) => ... }` | `ModelQueryBuilder` (chainable)                                          |
| `Model.paginate(perPage, pageName?)`        | rows per page (default 10), optional page param name                                                       | `Promise<Paginator>`                                                     |
| `Model.create(data)`                        | column/value object                                                                                        | `Promise<Model>` (hydrated instance with generated id and defaults)      |
| `Model.isFillable(key)`                     | attribute name                                                                                             | `boolean` (true if attribute is mass assignable)                         |
| `Model.filterAttributes(data)`              | column/value object                                                                                        | `object` (shallow copy containing only fillable attributes)              |
| `Model.getCasts()`                          | none                                                                                                       | `object` (merged cast definitions resolved across inheritance chain)     |
| `Model.firstOrCreate(attributes, values?)`  | search attributes; optional creation values                                                                | `Promise<Model>` (matched or newly created instance)                     |
| `Model.updateOrCreate(attributes, values?)` | search attributes; values to update/create                                                                 | `Promise<Model>` (updated or newly created instance)                     |
| `builder.clone()`                           | none                                                                                                       | `ModelQueryBuilder` (isolated copy preserving relations and constraints) |

### instance model methods

| method                                        | arguments                                                               | returns                                                                               |
| :-------------------------------------------- | :---------------------------------------------------------------------- | :------------------------------------------------------------------------------------ |
| `instance.save(newData?)`                     | optional object to merge before saving                                  | `Promise<void>`                                                                       |
| `instance.update(data)`                       | column/value object                                                     | `Promise<void>` (mutates instance attributes in place and executes UPDATE)            |
| `instance.increment(column, amount?, extra?)` | column name, optional amount (default 1), optional extra columns object | `Promise<object>` (mutates instance attribute in place and executes increment UPDATE) |
| `instance.decrement(column, amount?, extra?)` | column name, optional amount (default 1), optional extra columns object | `Promise<object>` (mutates instance attribute in place and executes decrement UPDATE) |
| `instance.delete()`                           | none                                                                    | `Promise<void>` (deletes row by primary key)                                          |
| `instance.hash(field)`                        | attribute name on the instance                                          | `Promise<void>` (mutates instance in place with bcrypt hash)                          |
| `instance.load('rel1', 'rel2')`               | relation names                                                          | `Promise<Model>` (lazy eager loads relations on existing instance)                    |
| `instance.toJSON()`                           | none                                                                    | plain object with hidden attributes removed and relations serialized                  |

### retrieving records

```javascript
// single records
const user = await User.find(1);
const user = await User.findByEmail('user@example.com'); // magic finder
const latest = await User.latest(); // most recent single record

// multiple records and collections
const all = await User.all();
const active = await User.where('status', 'active').get();
const topTen = await User.where('points', '>', 100).orderBy('points', 'DESC').limit(10).get();
const recentFive = await User.latest(5);

// pagination (returns Paginator instance, reading page from context automatically)
const paged = await User.where('status', 'active').paginate(20);
// paged is an iterable Paginator instance with .total, .currentPage, .lastPage, .links(), etc.
// custom page parameter name: await User.paginate(20, 'user_page')

// eager loading (prevents n+1 queries)
const users = await User.with('profile', 'posts').get();
const user = await User.with('profile').find(1);
const nested = await User.with('posts.comments').get();
```

### creating, updating, and deleting

```javascript
// create new record (returns hydrated instance reloaded with generated id)
const user = await User.create({
  name: 'alice',
  email: 'alice@example.com',
  password: 'plainpassword' // auto hashed per app.hashFields (default ['password'])
});

// find or create / update or create
const user = await User.firstOrCreate({ email: 'alice@example.com' }, { name: 'alice' });
const user = await User.updateOrCreate({ email: 'alice@example.com' }, { status: 'verified' });

// update instance
user.status = 'inactive';
await user.save();
// or
await user.update({ status: 'inactive' });

// delete instance
await user.delete();
```

## relationships

define relationships as methods returning `hasOne`, `hasMany`, or `belongsTo`.

```javascript
// models/Post.js
import { Model } from 'dframework';
import User from './User.js';
import Comment from './Comment.js';

export default class Post extends Model {
  author() {
    return this.belongsTo(User, 'user_id');
  }

  comments() {
    return this.hasMany(Comment, 'post_id');
  }

  roles() {
    return this.belongsToMany(Role); // many to many with pivot table inference (role_user)
  }
}
```

- many to many relationships use `this.belongsToMany(Role, pivotTable, foreignPivotKey, relatedPivotKey)`
- pivot table names default to alphabetically sorted snake_case model names (e.g. `role_user`), foreign keys to `${singular}_id`
- load additional pivot columns using `.withPivot('expires_at', 'level')`
- pivot data attaches cleanly to `model.pivot` without polluting primary model attributes
- pivot management helpers on relation proxy:
  - `await user.roles().attach(roleId, { level: 'admin' })` (accepts scalar, array of IDs, or object map)
  - `await user.roles().detach(roleId)` (or `detach([1, 2])` or `detach()` for all)
  - `await user.roles().sync({ 1: { level: 'admin' }, 2: { level: 'editor' } })` (returns `{ attached, detached, updated }`)

### consuming relationships and eager loading

relationships should always be consumed through standard async method calls or properties. the framework manages memory caching automatically:

```javascript
// eager load relationships when querying models
const posts = await Post.with('author', 'comments', 'roles').get();

// access directly as properties or await relationship methods
for (const post of posts) {
  const author = await post.author(); // 0 queries (resolved from memory cache)
  const comments = await post.comments(); // 0 queries (resolved from memory cache)
  console.log(post.author.name); // direct property access
}

// inside model methods (e.g. toCard, serialize, formatting)
export default class Post extends Model {
  async toCard() {
    const [author, comments, roles] = await Promise.all([
      this.author(),
      this.comments(),
      this.roles()
    ]);

    return {
      id: this.id,
      title: this.title,
      author: author ? { id: author.id, name: author.name } : null,
      comments_count: comments.length,
      roles: roles.map((role) => role.name)
    };
  }
}
```

### relationship caching and query rules

1. **automatic in memory caching**: when a relationship is preloaded with `Model.with('relation')` or `instance.load('relation')`, awaiting the relationship method (`await model.relation()`) or accessing the property (`model.relation`) resolves instantly from in memory cache with zero database queries.
2. **never access private `_relations`**: developers and application code must never inspect or access `this._relations` directly (e.g. `this._relations && 'author' in this._relations ? this._relations.author : await this.author()` is an anti pattern). always call `await this.author()` or use `Promise.all([this.author(), this.comments()])`.
3. **unconstrained queries**: calling an unconstrained relation method on a model with `null` foreign/local keys (e.g. `track.genre()` when `genre_id` is null, or on unsaved models) immediately resolves to `null` (for `belongsTo`/`hasOne`) or `[]` (for `hasMany`/`belongsToMany`) without executing unconstrained full table database queries.
4. **constrained queries**: chaining query constraints (e.g. `await post.comments().where('approved', true).get()`) marks the query builder as constrained and executes a targeted SQL query with the specified filters without corrupting or returning stale cached memory.
5. **dual accessor proxies**: relation properties are callable proxies; `Array.isArray(post.comments)` evaluates to `false`. iterate directly or `await post.comments` without manual `Array.isArray` branching.
6. **relation property assignment**: relation properties support direct assignment (`post.comments = [c1, c2]`, `post.author = new User({ name: 'alice' })`), persisting into internal state for serialization.
7. **lazy eager loading**: load relations on existing instances after fetching with `await post.load('author', 'comments', 'roles')`.
8. **eager load constraints**: pass a query builder closure to filter or sort eager loaded models; sorting specified in the closure is strictly preserved:

```javascript
// single relation constraint
const users = await User.with('posts', (q) => q.where('published', true).orderBy('views', 'desc')).get();

// multiple relation constraints using object syntax
const authors = await Author.with({
  articles: (q) => q.where('status', 'live').orderBy('created_at', 'desc'),
  comments: (q) => q.orderBy('id', 'asc')
}).get();
```

## fluent query builder (DB facade and Model)

all query builder methods are chainable and available on both `DB.table('table_name')` and directly on `Model` classes (e.g. `User.leftJoin('profiles', 'users.id', '=', 'profiles.user_id')`, `Post.whereBetween('views', [10, 100])`). the global `DB` facade requires no import.

### select, joins, and relations

```javascript
// basic select and joins
const rows = await DB.table('users')
  .select('users.id', 'users.name', 'profiles.avatar', 'roles.title')
  .join('roles', 'users.role_id', '=', 'roles.id') // inner join
  .leftJoin('profiles', 'users.id', '=', 'profiles.user_id') // left join
  .rightJoin('accounts', 'users.account_id', '=', 'accounts.id') // right join
  .crossJoin('flags') // cross join
  .where('users.status', 'active')
  .orderBy('users.created_at', 'DESC')
  .limit(20)
  .offset(40)
  .get();

// advanced join with multi condition closure
const results = await DB.table('users')
  .join('posts', (join) => {
    join.on('users.id', '=', 'posts.user_id')
      .orOn('users.alt_id', '=', 'posts.author_id')
      .where('posts.is_published', '=', 1)
      .whereNull('posts.deleted_at');
  })
  .get();

// raw join clause
const data = await DB.table('orders')
  .joinRaw('LEFT JOIN discounts ON discounts.order_id = orders.id AND discounts.valid = ?', [1])
  .get();
```

### conditional where clauses

```javascript
const query = DB.table('products')
  .where('status', 'in_stock') // 2 args (= inferred)
  .where('price', '>', 50) // 3 args
  .whereNot('category', 'archived') // negated condition
  .whereNull('deleted_at') // IS NULL
  .whereNotNull('published_at') // IS NOT NULL
  .whereIn('tag_id', [1, 2, 3]) // IN array of values
  .whereNotIn('vendor_id', [9, 10]) // NOT IN array
  .whereBetween('price', [10, 100]) // BETWEEN bounds
  .whereNotBetween('stock', [0, 5]) // NOT BETWEEN bounds
  .whereColumn('updated_at', '>', 'created_at') // compare two columns
  .whereHashed('pin', '1234') // bcrypt hashed comparison
  .whereExists((q) => { // subquery EXISTS
    q.table('reviews').whereColumn('reviews.product_id', 'products.id').where('rating', '>=', 4);
  });

// or conditions
query.orWhere('featured', 1)
  .orWhereIn('category_id', [4, 5])
  .orWhereNull('discontinued_at')
  .orWhereNotNull('published_at')
  .orWhereBetween('price', [10, 50]);

// nested where grouping
const grouped = await DB.table('users')
  .where('active', 1)
  .where((q) => {
    q.where('role', 'admin').orWhere('points', '>', 1000);
  })
  .get();

// conditional query builder branching
const filtered = await DB.table('users')
  .when(searchQuery, (q, val) => q.where('name', 'LIKE', `%${val}%`))
  .unless(includeBanned, (q) => q.where('status', '!=', 'banned'))
  .get();
```

### raw expressions and subqueries

```javascript
// selectRaw with parameter bindings
const stats = await DB.table('orders')
  .select('user_id')
  .selectRaw('COUNT(id) AS total_orders, SUM(amount) AS total_spent')
  .groupBy('user_id')
  .get();

// subquery in select
const users = await DB.table('users')
  .select('id', 'name')
  .selectSub((q) => {
    q.table('orders').count().whereColumn('orders.user_id', 'users.id');
  }, 'orders_count')
  .get();

// distinct columns
const countries = await DB.table('users').distinct('country').get();
```

### aggregates and calculations

```javascript
const hasUsers = await DB.table('users').where('role', 'admin').exists();
const noUsers = await DB.table('users').where('role', 'banned').doesntExist();
const totalCount = await DB.table('orders').count();
const sumTotal = await DB.table('orders').sum('amount');
const avgRating = await DB.table('reviews').avg('rating');
const minPrice = await DB.table('products').min('price');
const maxPrice = await DB.table('products').max('price');
const names = await DB.table('users').pluck('name');
const first = await DB.table('users').where('id', 1).first();
const sql = DB.table('users').where('role', 'admin').toSql();
```

### insert, update, and delete

```javascript
// single insert
const { insertId } = await DB.table('logs').insert({ action: 'login', ip: '127.0.0.1' });

// batch insert returning array of inserted IDs
const ids = await DB.table('tags').insert([
  { name: 'electronics' },
  { name: 'audio' }
]);

// update with conditions
await DB.table('users').where('id', 1).update({ status: 'active' });

// atomic increment and decrement
await DB.table('users').where('id', 1).increment('total_listens', 1);
await User.where('id', 1).decrement('credits', 5);

// delete requires where clause
await DB.table('sessions').where('expires_at', '<', new Date()).delete();
```

## raw database queries

always use parameter bindings. never interpolate variables directly into sql strings.

```javascript
// parameterized queries (safe)
const rows = await DB.query('SELECT * FROM users WHERE status = ? AND role = ?', ['active', 'admin']);

// with query caching
const cached = await DB.query('SELECT * FROM settings', [], { cache: true, ttl: 5000 });

// raw sql literals for updates
import { SqlHelpers } from 'dframework';
await DB.update('users', { updated_at: SqlHelpers.raw('NOW()') }, { id: 1 });
```

## migrations

migration files live in `database/migrations/` and run in alphabetical order.

```javascript
// database/migrations/2026_01_01_000001_create_users_table.js
export async function up({ Schema }) {
  await Schema.create('users', (table) => {
    table.increments('id');
    table.string('name');
    table.string('email', 191).unique();
    table.string('password');
    table.enum('role', ['admin', 'editor', 'user']).defaultTo('user');
    table.boolean('is_active').defaultTo(true);
    table.json('settings').nullable();
    table.timestamps();
  });
}

export async function down({ Schema }) {
  await Schema.dropIfExists('users');
}
```

### foreign key constraints

```javascript
export async function up({ Schema }) {
  await Schema.create('posts', (table) => {
    table.increments('id');
    table.integer('user_id').unsigned().notNullable();
    table.string('title');
    table.text('content');
    table.timestamps();

    table.foreign('user_id')
      .references('id', 'users')
      .onDelete('cascade')
      .onUpdate('cascade');
  });
}
```

### modifying existing tables

```javascript
export async function up({ Schema }) {
  await Schema.table('users', (table) => {
    table.string('phone').nullable();
    table.string('email', 150).nullable().modify(); // fluent modify
    table.modify('bio', 'TEXT', { nullable: true }); // direct modify
    table.renameColumn('username', 'handle');
    table.dropColumn('temp_token');
    table.dropColumns('old_a', 'old_b');
    table.index('phone');
  });
}

export async function down({ Schema }) {
  await Schema.table('users', (table) => {
    table.dropForeign('fk_users_org_id');
    table.dropIndex('idx_users_phone');
    table.renameColumn('handle', 'username');
  });
}
```

### column types and modifiers

- column types: `id`, `increments`, `bigIncrements`, `integer`, `bigInteger`, `float`, `decimal`, `boolean`, `string`, `text`, `json`, `enum`, `datetime`, `date`, `time`, `timestamps`
- modifiers: `.nullable()`, `.notNullable()`, `.unsigned()`, `.defaultTo(val)`, `.unique()`, `.index()`, `.primary()`, `.comment('text')`, `.modify()`
- table alterations: `dropColumn(name)`, `dropColumns(names)`, `renameColumn(from, to)`, `modify(name, type, options)`, `index(cols, name?)`, `unique(cols, name?)`, `dropIndex(name)`, `dropForeign(name)`

## storage and file system

dframework provides a unified file system abstraction through the `Storage` service. it manages multiple disks, path traversal protection, automatic directory creation, disk space verification, multipart upload handling, and transparent AES-256-GCM encryption.

import `Storage` explicitly from `'dframework'`:

```javascript
import { Storage } from 'dframework';
```

### configuration

storage disks are configured in `config/storage.js` under `storage.disks`:

```javascript
// config/storage.js
import { Env } from 'dframework';

export default {
  disks: {
    local: {
      root: './storage/local', // private files, not web accessible
    },
    public: {
      root: './storage/public', // web accessible files
    },
    secure: {
      root: './storage/secure',
      encryptionKey: Env.value('APP_KEY'), // transparent aes-256-gcm encryption
    },
  },
};
```

- dframework automatically symlinks `storage/public` to `public/storage` on boot
- public disk files are accessible in templates via `storage('avatars/1.jpg')` helper or browser path `/storage/avatars/1.jpg`

### obtaining a disk

call `Storage.disk(name)`. omitting the disk name defaults to the `'public'` disk:

```javascript
const publicDisk = Storage.disk(); // defaults to 'public'
const localDisk = Storage.disk('local');
const secureDisk = Storage.disk('secure');
```

### storing files and handling uploads

`put(relativePath, data)` creates parent directories automatically and accepts strings, `Buffer` instances, or multipart upload objects from `req.files`:

```javascript
// write string content
await Storage.disk('local').put('exports/report.csv', 'id,name\n1,alice');

// write buffer
await Storage.disk().put('images/logo.png', imageBuffer);

// direct multipart file upload handling (req.files.avatar from form upload)
const filePath = await Storage.disk().put('avatars/user.jpg', req.files.avatar);
```

### retrieving files

```javascript
// returns Buffer by default
const buffer = await Storage.disk('local').get('documents/contract.pdf');

// pass encoding to return string
const text = await Storage.disk('local').get('exports/report.csv', 'utf8');

// check file existence
const exists = await Storage.disk().exists('avatars/user.jpg');

// get absolute server path
const absolutePath = Storage.disk('local').path('exports/report.csv');
```

### file management and deletion

```javascript
// move or rename file within disk
await Storage.disk().move('temp/draft.txt', 'published/final.txt');

// duplicate file within disk
await Storage.disk().copy('templates/base.pdf', 'invoices/1001.pdf');

// delete file (returns true if deleted, false if file did not exist)
await Storage.disk().delete('temp/draft.txt');
```

### transparent encryption

disks configured with `encryptionKey` automatically encrypt on write and decrypt on read using AES-256-GCM with tamper detection (12-byte IV + 16-byte auth tag + ciphertext). the API is identical to unencrypted disks:

```javascript
// encrypted transparently on disk
await Storage.disk('secure').put('secrets/token.txt', 'secret-key-value');

// decrypted transparently on read
const secret = await Storage.disk('secure').get('secrets/token.txt', 'utf8');
```

### built in safety protections

- **disk capacity protection**: before writing, `Storage` verifies that the target filesystem has sufficient space for the payload plus a 100 megabyte safety buffer to prevent server disk exhaustion
- **path traversal mitigation**: blocks null bytes (`\0`), resolves canonical paths within the disk root, rejects escaping path segments (`..`), and limits path depth to 10 segments

## running migrations and seeders

```bash
dstrn migrate
dstrn migrate:rollback
dstrn seed
```

in production (`APP_ENV=production`), interactive confirmation prompts are displayed. pass `--force` to run non-interactively in automated deployment pipelines:

```bash
dstrn migrate --force
dstrn seed --force
```