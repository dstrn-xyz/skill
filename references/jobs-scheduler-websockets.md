# background jobs, scheduler, and websockets

dframework includes integrated asynchronous infrastructure with zero external daemon requirements like redis or celery.

## background jobs (worker threads)

jobs live in `jobs/` and execute in dedicated background worker threads.

```javascript
// jobs/SendWelcomeEmailJob.js
export default class SendWelcomeEmailJob {
  async handle(payload) {
    const user = await User.find(payload.userId);
    if (!user) return;

    await Mailer.send(user.email, 'welcome to the platform', {
      name: user.name
    });
  }
}
```

### dispatching jobs

use the global `Job` facade from anywhere (controllers, middlewares, socket handlers, commands):

```javascript
// in a controller
await Job.dispatch('SendWelcomeEmailJob', { userId: user.id }, { timeout: 30000, tries: 3 });
```

- payload must be json serializable (pass record ids instead of large active record model instances)
- options parameter supports `timeout` (milliseconds, default 60000), `tries` (max retry count, default 1), and `backoff` (exponential backoff base in ms)
- persistent worker pool: worker threads are only warmed and started if the application contains jobs in `jobs/`, meaning applications with zero jobs pay zero performance or memory overhead
- persistent workers remain alive across sequential jobs to eliminate thread creation and database connection overhead
- `Socket.broadcast()` and `Socket.setState()` called inside jobs are automatically relayed to the main process websocket server via internal IPC
- configure worker pool concurrency in `config/app.js` via `queue.maxWorkers` (default 4)

### broadcasting from jobs

jobs can dispatch real time websocket events or update client component state using the global `Socket` facade:

```javascript
// jobs/ExportReportJob.js
export default class ExportReportJob {
  async handle(payload) {
    Socket.broadcast('export:progress', { percent: 50 });
    const report = await this.generate(payload.reportId);
    Socket.broadcast('export:ready', { url: report.url });
    Socket.setState('#export-status', { ready: true });
  }
}
```

## task scheduler

scheduled tasks are defined in `console/Schedule.js`. the scheduler starts automatically with the server.

```javascript
// console/Schedule.js
export default function (scheduler) {
  // schedule command classes from console/commands/
  scheduler.command('PruneExpiredSessions').hourly();
  scheduler.command('GenerateDailyReport').daily();

  // schedule inline closures
  scheduler.command(async () => {
    await DB.table('logs').where('created_at', '<', new Date(Date.now() - 30 * 86400000)).delete();
  }).weekly();
}
```

### frequency methods

- `everyMinute()` (60s)
- `everyFiveMinutes()` (5m)
- `everyTenMinutes()` (10m)
- `everyFifteenMinutes()` (15m)
- `everyThirtyMinutes()` (30m)
- `hourly()` (1h)
- `daily()` (24h)
- `weekly()` (7d)
- `every(ms)` (custom milliseconds)

### command classes (scheduler tasks)

developers cannot define new top level `dstrn` cli commands. instead, command classes live in `console/commands/` and represent scheduler tasks that can be scheduled or run manually:

```javascript
// console/commands/PruneExpiredSessions.js
export default class PruneExpiredSessions {
  async handle(args, options) {
    const isDryRun = this.hasOption('dry-run');
    const batch = parseInt(this.option('batch', 500), 10);
    const table = this.argument(0, 'sessions');

    const count = await DB.table(table).where('expires_at', '<', new Date()).delete();
    Log.info('pruned expired sessions', { count, batch, isDryRun });
  }
}
```

- scheduled in `console/Schedule.js`
- run manually from CLI: `dstrn run PruneExpiredSessions [args]` (supports `--key value`, `--key=value`, boolean flags, and positional args accessible via `this.option()`, `this.hasOption()`, and `this.argument()`)

## websockets (socket router)

socket events are defined in `routes/wire.js` using the global `Socket` facade.

```javascript
// routes/wire.js
Socket.on('ping', () => json({ pong: true }));

// group with authentication middleware
Socket.group({ middleware: ['AuthMiddleware@requireAuth'] }, (auth) => {
  auth.on('chat:message', async (req, res, ws) => {
    const message = await Message.create({
      user_id: Auth.user().id,
      content: req.body.content
    });

    // broadcast to all clients except current connection
    Socket.broadcast('chat:new', {
      id: message.id,
      user: Auth.user().name,
      content: message.content
    }, ws);

    return json({ success: true });
  }).targets(['#chat-messages']);
});
```

- `req` in socket handlers contains session data, cookies, and `Auth.user()`
- `.targets(['#selector'])` enforces that client morph updates are only applied to authorized dom targets
- `Socket.toUser(userId, event, data)` / `Socket.toUserState(userId, target, state)` / `Socket.toUser(userId).emit(event, data)` targets a specific user across connections
- `Socket.toUsers([id1, id2], event, data)` / `Socket.toUsersState([id1, id2], target, state)` targets multiple users
- `Socket.toSession(sessionId, event, data)` / `Socket.toSessionState(sessionId, target, state)` targets a specific session id
- `Socket.to(ws, event, data)` / `Socket.to(ws).emit(event, data)` targets a specific connection instance directly
- `Socket.guard(guard).toUser(userId, event, data)` / `Socket.guard(guard).toUsers([id1, id2], event, data)` scopes dispatch to an authentication guard
- `Socket.toGuardUser(guard, userId, event, data)` / `Socket.toGuardUserState(guard, userId, target, state)` targets a specific user on a named guard
- `Socket.toGuardUsers(guard, [id1, id2], event, data)` / `Socket.toGuardUsersState(guard, [id1, id2], target, state)` targets multiple users on a named guard
- all targeted socket methods called inside jobs are automatically relayed to the main process via IPC