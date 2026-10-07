# command line interface reference

dframework provides a built in cli runner invoked with `dstrn <command>`. all commands except `dstrn init`, `dstrn help`, and `dstrn version` (-v) are gated and require execution within a valid dframework project directory containing `config/app.js`.

## project lifecycle

- `dstrn init <name>`: scaffolds a fresh dframework application
- `dstrn serve`: boots the development server with live reload and watcher
- `dstrn tinker [code]`: interactive repl with all models, facades, and helpers preloaded
- `dstrn key:generate`: generates and updates the 32 byte APP_KEY encryption key in .env
- `dstrn run <command>`: runs a user command from console/commands/
- `dstrn dispatch <job> [--payload=json]`: queues a background job from the command line

## database and migrations

- `dstrn migrate`: runs all pending database migrations
- `dstrn migrate:status`: displays batch status of all migrations
- `dstrn migrate:rollback`: rolls back the latest migration batch
- `dstrn migrate:make <table>`: creates a timestamped migration file
- `dstrn seed [seeder]`: executes database seeders
- `dstrn seed:make <name>`: creates a new database seeder class
- `dstrn drop`: drops all tables in the database with confirmation prompt
- `dstrn advisor`: analyzes database schemas and queries to recommend optimal indexes

## code generators

- `dstrn make:model <name>`: generates a model class in models/
- `dstrn make:controller <name>`: generates a controller in controllers/
- `dstrn make:middleware <name>`: generates a middleware class in middlewares/
- `dstrn make:command <name>`: generates a console command in console/commands/
- `dstrn make:job <name>`: generates an asynchronous job class in jobs/
- `dstrn make:component <name>`: generates a view component template
- `dstrn make:plugin <name>`: scaffolds a cross platform native plugin

## system and certificates

- `dstrn logs:clear`: archives or purges application log files
- `dstrn docs:publish`: copies framework documentation into the local project
- `dstrn deploy`: generates systemd unit and deployment files
- `dstrn cert`: inspects ssl certificates
- `dstrn cert:obtain`: provisions automated ssl certificates
- `dstrn cert:renew`: renews installed ssl certificates
- `dstrn cert:status`: checks validity and expiration of domain certificates

## native platforms

- `dstrn simulate [--ios|--android|--desktop]`: runs the project in native mobile or desktop simulator
- `dstrn build [--ios|--android|--desktop]`: compiles production binaries for target platforms
- `dstrn native:status`: inspects native project configuration and platform status
- `dstrn native:doctor`: verifies native toolchains and sdk installations
- `dstrn native:clean`: cleans native build caches and derived data

## architecture and quality

- `dstrn graph`: analyzes application dependency graph and routes
- `dstrn lint`: checks architecture rules and separation of concerns
- `dstrn dead`: identifies unused views, dead controllers, and orphaned code
- `dstrn doctor`: comprehensive health score across security, database, and code quality
- `dstrn lang:lint`: verifies translation keys and language parity across dictionaries