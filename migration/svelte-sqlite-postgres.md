Migrate Existing Svelte + Node.js + TypeScript Application from SQLite to PostgreSQL 18

Role and Authority

You are a Principal PostgreSQL Architect, Senior TypeScript Engineer, Svelte/SvelteKit Specialist, Database Migration Engineer, DevOps Engineer, and Test Automation Engineer.

You have full authority to inspect and modify the repository.

Your mandate is to completely migrate the existing application from SQLite to PostgreSQL 18, including:

* Application source code
* Database access layers
* SQL queries
* Schema definitions
* Migration files
* Seed data
* Data migration tooling
* Unit, integration, and end-to-end tests
* Test fixtures and test infrastructure
* Docker and Docker Compose configuration
* Environment configuration
* Package dependencies
* Build scripts
* Shell scripts
* Makefiles
* CI/CD workflows
* Operational tooling
* Markdown documentation
* Developer instructions
* Any supplementary files that reference or depend on SQLite

This is an implementation task, not merely an architectural assessment.

Do not stop after producing a plan. Inspect the repository, implement the migration, run validation, fix defects, and leave the repository in a working state.

⸻

Primary Objective

Convert the existing Svelte-based Node.js TypeScript web application from SQLite to PostgreSQL 18 while preserving all existing functionality, data semantics, API contracts, user journeys, security controls, and observable application behaviour.

The PostgreSQL database will run in a container.

The completed repository must:

1. Use PostgreSQL 18 as its only production runtime database.
2. No longer require SQLite during normal application execution.
3. Provide a safe, repeatable method for migrating existing SQLite data into PostgreSQL.
4. Provide deterministic schema migrations.
5. Provide containerised PostgreSQL development and test environments.
6. Pass all applicable tests, builds, type checks, linting, and migration verification.
7. Contain updated operational and developer documentation.
8. Avoid introducing unnecessary frameworks, abstractions, or architectural churn.

⸻

Non-Negotiable Requirements

Preserve Existing Behaviour

You must not:

* Delete existing functionality.
* Remove application features because they are difficult to migrate.
* Change public API contracts unless technically unavoidable.
* Change business rules without explicitly documenting the reason.
* Replace working application architecture unnecessarily.
* Rewrite unrelated modules.
* silently alter query semantics.
* Leave unresolved placeholders or migration TODOs.
* leave the application supporting SQLite and PostgreSQL indefinitely.
* use SQLite as a fallback when PostgreSQL is unavailable.
* hard-code credentials.
* expose database credentials to client-side Svelte code.
* use the PostgreSQL superuser as the normal application role.

Repository Instructions

Before making changes, locate and obey all repository instructions, including:

* AGENTS.md
* README.md
* CONTRIBUTING.md
* Architecture decision records
* Project-specific coding standards
* Package manager configuration
* Workspace configuration
* Existing migration conventions
* Existing test conventions

Where repository instructions conflict with this migration mandate, preserve the intent of the repository instructions while completing the PostgreSQL migration.

No Assumptions

Determine the actual implementation from the repository.

Do not assume:

* Whether this is Svelte, SvelteKit, or a custom Svelte server.
* Whether the project uses npm, pnpm, yarn, or another package manager.
* Whether it uses Prisma, Drizzle, Kysely, Knex, Sequelize, TypeORM, MikroORM, raw SQL, better-sqlite3, sqlite3, or a custom persistence layer.
* Whether SQLite access is synchronous or asynchronous.
* Whether migrations already exist.
* Whether tests use real databases, mocks, temporary SQLite files, or fixtures.
* Whether there is one database or multiple SQLite files.
* Whether IDs are generated internally or externally.
* Whether SQLite-specific behaviour is relied upon intentionally.

Inspect and prove each of these before selecting the implementation approach.

⸻

PostgreSQL Target

Use PostgreSQL major version 18.

At execution time:

* Use the latest stable, supported PostgreSQL 18.x patch release allowed by repository dependency-pinning policy.
* Never select a beta, release candidate, development, or PostgreSQL 19 image.
* Prefer a pinned PostgreSQL 18 patch tag or immutable digest for CI and production-like environments.
* A floating postgres:18 tag may be used for local development only when consistent with repository policy.
* Do not use PostgreSQL 18 releases older than 18.4 unless an explicit repository constraint makes that unavoidable. PostgreSQL 18.4 includes security and defect corrections affecting earlier 18.x releases. (PostgreSQL)

For the official PostgreSQL 18 container image:

* Account for PostgreSQL 18’s version-specific PGDATA layout.
* Mount the persistent named volume at /var/lib/postgresql.
* Do not blindly reuse the PostgreSQL 17-and-earlier /var/lib/postgresql/data volume target.
* Do not manually override PGDATA unless there is a demonstrated repository requirement.
* Document the selected image tag, volume target, persistence behaviour, and upgrade implications. (Docker Hub)

⸻

Phase 1: Forensic Repository Discovery

Before modifying code, perform a repository-wide audit.

Identify the Application Architecture

Determine and document:

* Svelte or SvelteKit version.
* Node.js version.
* TypeScript version and configuration.
* Server adapter and deployment model.
* SSR, SPA, or hybrid rendering mode.
* Package manager and workspace structure.
* Existing server-only module conventions.
* Application startup lifecycle.
* Development, test, build, and production commands.
* Existing Docker and Compose topology.
* Existing CI/CD workflows.

Find Every SQLite Dependency

Search all tracked source and supplementary files for:

* sqlite
* sqlite3
* better-sqlite3
* libsql
* bun:sqlite
* SQLite ORM drivers
* SQLite dialect selections
* SQLite connection strings
* .db
* .sqlite
* .sqlite3
* file:
* database file paths
* temporary database creation
* in-memory SQLite usage
* PRAGMA
* AUTOINCREMENT
* last_insert_rowid
* INSERT OR
* SQLite-specific SQL functions
* SQLite migration tooling
* SQLite backup and restore scripts

Search:

* TypeScript
* JavaScript
* Svelte files
* SQL files
* Test files
* Shell scripts
* Python scripts
* Makefiles
* Dockerfiles
* Compose files
* CI workflows
* Environment files
* JSON, YAML, and TOML configuration
* Markdown documentation
* Generated schema definitions
* Seed and fixture files

Do not rely solely on textual dependency names. Trace database abstractions, factory functions, repositories, adapters, service constructors, imports, and injected dependencies.

Build a Database Usage Inventory

Create or update:

docs/postgresql-migration-audit.md

The inventory must include:

Area	Current implementation	SQLite dependency	PostgreSQL replacement	Files affected	Risk

Include at minimum:

* Database creation
* Connection lifecycle
* Transactions
* Schema creation
* Migrations
* Query execution
* Prepared statements
* Repositories
* Services
* API handlers
* Svelte server load functions
* Svelte form actions
* Hooks and middleware
* Background workers
* Authentication/session storage
* Test setup
* Fixtures
* Seeds
* CLI utilities
* Backup scripts
* CI services
* Docker configuration
* Documentation

Inventory the Existing Schema

Extract the authoritative SQLite schema.

Record:

* Tables
* Columns
* Declared types
* Actual stored value types where relevant
* Primary keys
* Foreign keys
* Unique constraints
* Check constraints
* Defaults
* Generated columns
* Indexes
* Partial indexes
* Triggers
* Views
* Virtual tables
* FTS tables
* WITHOUT ROWID tables
* Strict tables
* Cascade behaviour
* Deferrable relationships
* Implicit assumptions enforced only by application code

Where a live or fixture SQLite database is available, inspect:

* sqlite_master
* PRAGMA table_info
* PRAGMA table_xinfo
* PRAGMA foreign_key_list
* PRAGMA index_list
* PRAGMA index_info
* PRAGMA index_xinfo
* Trigger and view definitions

Do not assume migration files accurately represent the deployed SQLite database. Compare migrations, current schema code, and available database files.

⸻

Phase 2: Select the PostgreSQL Data Access Strategy

Preserve the Existing Abstraction Where Practical

If the repository already uses an ORM or query builder with mature PostgreSQL support:

* Retain it.
* Change its dialect, provider, or driver.
* Regenerate metadata or clients where required.
* Convert SQLite-specific raw SQL separately.
* Avoid replacing the ORM merely because another option is preferred.

If the repository directly uses a SQLite-only driver:

* Replace it with an appropriate PostgreSQL driver or a minimal query abstraction.
* Prefer the smallest change that results in a clean, maintainable PostgreSQL implementation.
* Do not introduce multiple competing database access libraries.
* Do not add an ORM solely to avoid understanding the existing SQL.

For direct PostgreSQL access in Node.js, use a maintained, TypeScript-compatible PostgreSQL client that supports:

* Connection pooling
* Parameterised queries
* Transactions
* Savepoints where required
* Row count metadata
* RETURNING
* Graceful shutdown
* TLS configuration
* Connection timeout configuration
* Pool timeout configuration
* Typed results or safe result validation

Async Propagation

SQLite libraries such as better-sqlite3 may use synchronous calls. PostgreSQL access is normally asynchronous.

When converting synchronous database access:

* Propagate async/await through the complete call chain.
* Update repository interfaces.
* Update service methods.
* Update API handlers.
* Update SvelteKit load functions and actions.
* Update hooks and middleware.
* Update tests and mocks.
* Do not suppress returned promises.
* Do not create fire-and-forget writes.
* Do not block the Node.js event loop to simulate synchronous behaviour.

Server-Only Isolation

Ensure:

* Database clients are imported only by server-side modules.
* Database credentials cannot enter browser bundles.
* SvelteKit server modules use appropriate server-only locations and naming.
* Shared domain types remain independent of database drivers.
* Client code receives serialisable DTOs rather than driver-specific row objects.

⸻

Phase 3: PostgreSQL Connection Architecture

Implement one canonical database connection subsystem.

It must provide:

* A singleton or lifecycle-safe connection pool.
* Hot-module-reload safety during local development.
* Explicit configuration validation.
* Parameterised query execution.
* Transaction helpers.
* Graceful pool shutdown.
* Connection error reporting without leaking credentials.
* Configurable pool sizing.
* Configurable connection and idle timeouts.
* Optional TLS configuration.
* Health/readiness checking.
* Test injection or isolation without duplicating production logic.

Use one canonical environment variable:

DATABASE_URL=postgresql://application_user:password@postgres:5432/application_db

Additional environment variables may be supported when already required by repository conventions, but DATABASE_URL must remain the canonical assembled connection value.

Validate configuration at startup.

Reject:

* Missing database URL
* Unsupported URL scheme
* Invalid pool values
* Insecure production defaults
* Client-side access to database configuration

Do not log:

* Passwords
* Full connection URLs containing credentials
* Authentication tokens
* Secret environment variables

⸻

Phase 4: Containerised PostgreSQL

Add or update the repository’s Compose configuration.

Do not include the obsolete Compose schema version field.

PostgreSQL Service

Create a PostgreSQL service with:

* Official PostgreSQL 18 image
* Stable 18.x patch tag or digest
* Named persistent volume
* Correct PostgreSQL 18 volume target
* Internal Compose network
* POSTGRES_DB
* Bootstrap administrator or migration credentials
* Secure password injection
* pg_isready health check
* Restart policy appropriate to repository conventions
* Sensible shm_size
* Explicit locale only where application behaviour requires it
* No POSTGRES_HOST_AUTH_METHOD=trust
* No hard-coded production password
* No unnecessary public host binding

Expose port 5432 to the host only when required for local development.

When exposed, make the host port configurable to avoid collisions.

Application Dependency

Where the application also runs in Compose:

* Configure its DATABASE_URL using the PostgreSQL service hostname.
* Add a health-based dependency where supported.
* Do not use localhost from one container to reach another.
* Ensure application startup does not race database readiness.
* Still retain application-level retry/backoff for brief connection failures.

Credentials and Roles

Separate concerns where practical:

* Bootstrap administrator role
* Migration/schema-owner role
* Runtime application role
* Read-only operational role only when the repository requires one

The normal application role must not be a PostgreSQL superuser.

Grant only required privileges.

Do not make the application runtime depend on POSTGRES_USER being its permanent superuser identity.

Initialisation Scripts

Do not use /docker-entrypoint-initdb.d as the primary application schema migration system.

It may be used only for minimal bootstrap actions that occur before the repository migration tool can run, such as:

* Creating a non-superuser application role
* Creating a dedicated schema
* Granting initial connection privileges

All versioned application schema changes must run through the repository’s migration system.

Remember that entrypoint initialisation scripts only execute against an empty PostgreSQL data directory. Never rely on them for repeatable upgrades.

⸻

Phase 5: Schema Conversion

Create a deterministic PostgreSQL schema using the project’s existing migration framework where viable.

If no reliable migration framework exists, introduce one appropriate to the selected data access stack.

There must be one authoritative migration history.

Do not maintain parallel schema definitions that can drift.

Type Mapping

Do not perform blind textual substitutions. Determine the semantic intent of every column.

Use mappings such as:

SQLite usage	PostgreSQL candidate	Required analysis
INTEGER PRIMARY KEY	bigint or integer identity	Determine ID range, external ID preservation, and insert behaviour
AUTOINCREMENT	GENERATED ... AS IDENTITY	Do not use legacy serial unless required by existing tooling
Integer 0/1 boolean	boolean	Validate all existing values before conversion
TEXT timestamp	timestamptz or timestamp	Determine timezone semantics
Unix epoch integer	bigint or timestamp type	Preserve public API representation
JSON stored as TEXT	jsonb	Validate every existing value before conversion
BLOB	bytea	Verify encoding and driver return type
REAL	double precision or numeric	Use numeric for exact financial/decimal values
Decimal stored as text	numeric(p,s) where valid	Determine precision and scale from data
UUID stored as text	uuid	Validate existing UUIDs
Date-only text	date	Verify no time components exist
Duration integer	integer, bigint, or interval	Preserve units and external contracts
Case-insensitive text	citext or normalised index strategy	Do not change comparison semantics accidentally

Do not use PostgreSQL enum types automatically. Evaluate whether:

* A check constraint
* Lookup table
* Domain type
* PostgreSQL enum

best matches the project’s change and deployment requirements.

Identity Columns and Sequences

For generated numeric IDs:

* Use PostgreSQL identity columns where compatible.
* Preserve all existing SQLite IDs during migration.
* Reset each identity sequence after import.
* Set the next generated value safely above the current maximum.
* Handle empty tables correctly.
* Test the first insert after migration.

Do not convert externally supplied IDs into generated IDs.

Timestamps

Determine the application’s actual timezone contract.

Prefer:

* timestamptz for real-world instants.
* UTC at storage and application boundaries.
* Explicit serialisation at API boundaries.

Do not assume SQLite timestamp strings contain timezone data.

Detect and handle:

* Naive timestamps
* ISO 8601 strings
* Unix seconds
* Unix milliseconds
* Mixed formats
* Invalid timestamps
* SQLite-generated UTC values
* Application-generated local values

Document all timestamp conversion decisions.

JSON

Convert textual JSON to jsonb only when:

* The column semantically contains JSON.
* All non-null existing values are valid JSON.
* Application code expects structured JSON.
* Indexing or querying semantics are compatible.

Do not automatically convert arbitrary text columns containing occasional JSON.

Define how driver-returned JSON objects are validated and serialised.

Constraints

PostgreSQL must enforce domain integrity that SQLite may have treated loosely.

Implement and verify:

* Primary keys
* Foreign keys
* Unique constraints
* Check constraints
* Non-null constraints
* Default expressions
* Cascade behaviour
* Restrict behaviour
* Deferred constraints where genuinely required

Before adding stricter constraints, inspect existing data and provide a deterministic remediation path for invalid rows.

Do not silently discard invalid data.

Indexes

Recreate all behaviourally relevant indexes.

Review each index for PostgreSQL suitability:

* B-tree
* GIN
* GiST
* BRIN
* Partial indexes
* Expression indexes
* Composite indexes
* Unique indexes

Do not copy redundant SQLite indexes without analysis.

Use EXPLAIN or EXPLAIN ANALYZE on important queries where fixtures provide meaningful volumes.

Full-Text Search

If SQLite FTS3, FTS4, or FTS5 is used:

* Treat it as a dedicated migration workstream.
* Reproduce the user-visible search semantics.
* Determine tokenisation, ranking, prefix, phrase, highlighting, and case behaviour.
* Implement PostgreSQL full-text search using appropriate tsvector, tsquery, indexes, generated columns, or maintained vectors.
* Add behavioural tests comparing representative old and new search results.
* Do not replace full-text search with %LIKE% queries.

Triggers and Views

Convert SQLite triggers and views deliberately.

Account for:

* Trigger timing
* Row versus statement execution
* OLD and NEW semantics
* Recursive behaviour
* Update timestamps
* Audit events
* Materialisation expectations
* View mutability
* Security-barrier implications

Prefer application logic or native PostgreSQL constraints where an existing trigger merely compensates for SQLite limitations, but preserve observable behaviour.

⸻

Phase 6: Convert SQLite-Specific SQL

Locate every SQL statement, including dynamically assembled queries.

Convert SQL semantically, not through global search-and-replace.

Parameter Placeholders

Convert SQLite placeholders such as:

?
?1
:name
@name
$name

to the selected PostgreSQL client’s supported parameterisation model, normally:

$1
$2
$3

Maintain correct parameter ordering.

Never interpolate user-controlled data into SQL.

Identifiers cannot be parameterised. Any dynamic identifier must come from a strict allowlist and use the driver’s safe identifier mechanism where available.

Required Semantic Conversions

Review and convert all applicable constructs:

SQLite construct	PostgreSQL approach
INSERT OR IGNORE	INSERT ... ON CONFLICT ... DO NOTHING
INSERT OR REPLACE	Explicit ON CONFLICT ... DO UPDATE, after analysing replacement semantics
REPLACE INTO	Explicit PostgreSQL upsert
last_insert_rowid()	INSERT ... RETURNING
Driver lastInsertRowid	Returned row from RETURNING
Driver changes	PostgreSQL command rowCount or returned rows
datetime('now')	CURRENT_TIMESTAMP or application-supplied UTC timestamp
date('now')	CURRENT_DATE
strftime(...)	PostgreSQL date/time functions or to_char
julianday(...)	PostgreSQL timestamp arithmetic
unixepoch(...)	extract(epoch from ...) or to_timestamp(...)
ifnull(a,b)	coalesce(a,b)
SQLite IIF	CASE
group_concat	string_agg
randomblob	Appropriate PostgreSQL random/crypto function or application generation
hex/unhex	PostgreSQL encoding functions
LIMIT offset,count	LIMIT count OFFSET offset
GLOB	PostgreSQL-compatible matching implementation
Case-insensitive LIKE assumption	Explicit ILIKE, collation, citext, or normalisation
PRAGMA	Remove or replace with PostgreSQL configuration/schema introspection
VACUUM invocation	PostgreSQL operational strategy, not request-path SQL
ANALYZE invocation	PostgreSQL operational strategy
SQLite transaction mode	Explicit PostgreSQL isolation and locking semantics
BEGIN IMMEDIATE	Appropriate transaction/locking design
WITHOUT ROWID	Normal PostgreSQL table with explicit key/index design

Replacement Semantics

SQLite INSERT OR REPLACE may delete the conflicting row and insert a new row.

Do not translate it mechanically to PostgreSQL ON CONFLICT DO UPDATE.

Analyse whether existing behaviour depends on:

* Delete triggers
* Cascading deletes
* New generated IDs
* Reset defaults
* Changed creation timestamps
* Foreign-key relationships
* Audit records

Choose and document the PostgreSQL operation that preserves intended business behaviour.

Query Result Differences

Audit code for assumptions about:

* Column name casing
* Duplicate column names in joins
* Integer versus string representation
* bigint deserialisation
* Boolean representation
* Date object conversion
* JSON parsing
* bytea buffers
* NULL ordering
* Default sort order
* Case sensitivity
* Trailing whitespace
* Floating-point values
* Empty strings versus null
* Affected-row metadata

Define explicit row mappings where driver return types differ from existing application DTOs.

Ordering

SQL result order is undefined without ORDER BY.

Add deterministic ordering where:

* Tests depend on order.
* API responses promise order.
* UI rendering expects order.
* Pagination requires stable order.
* SQLite happened to return insertion or rowid order.

For pagination, include a deterministic tie-breaker.

Concurrency and Locking

SQLite often serialises writes differently from PostgreSQL.

Review all workflows involving:

* Read-modify-write sequences
* Counters
* Task claiming
* Queues
* Scheduler state
* Session updates
* Optimistic concurrency
* Idempotency keys
* Singleton records
* Upserts
* Uniqueness checks followed by insert
* State transitions

Use PostgreSQL mechanisms where required:

* Atomic UPDATE
* INSERT ... ON CONFLICT
* RETURNING
* Row locks
* FOR UPDATE
* FOR UPDATE SKIP LOCKED
* Advisory locks
* Serializable transactions
* Optimistic version columns
* Unique constraints

Do not retain race-prone “check then write” patterns merely because SQLite previously reduced concurrency.

⸻

Phase 7: Transactions

Implement explicit transaction handling.

The transaction API must:

* Obtain one pooled client.
* Begin the transaction.
* Execute all transaction operations on that same client.
* Commit on success.
* Roll back on failure.
* Release the client in finally.
* Preserve the original error.
* Support transaction-scoped repository access.
* Avoid accidentally executing transaction queries through the global pool.
* Support nested operations through transaction propagation or savepoints where required.

Review transaction isolation for each important workflow.

Do not globally increase isolation without evidence.

Test:

* Successful commit
* Rollback
* Constraint failure
* Concurrent updates
* Deadlock or serialization retry where applicable
* Nested service calls
* Resource release after exceptions

⸻

Phase 8: PostgreSQL Error Handling

Replace SQLite-specific error handling.

Implement stable application-level error mapping for relevant PostgreSQL conditions, such as:

* Unique constraint violation
* Foreign-key violation
* Check constraint violation
* Not-null violation
* Serialization failure
* Deadlock
* Connection failure
* Statement timeout
* Invalid input syntax
* Undefined table or column during migration failure

Do not expose raw SQL, connection details, or internal database structure to clients.

Where existing APIs return domain-specific errors, preserve those contracts.

Use named constraints where constraint identity is needed for reliable error mapping.

Do not depend on parsing human-readable PostgreSQL error messages when SQLSTATE or structured driver fields are available.

⸻

Phase 9: Existing SQLite Data Migration

Presume existing SQLite data must be preserved unless the repository explicitly proves the database is disposable.

Create a production-quality, one-shot migration utility in TypeScript unless repository conventions require another language.

Example command shape:

pnpm db:migrate:sqlite \
  --source ./data/application.sqlite \
  --database-url "$DATABASE_URL"

Adapt the command to the repository’s package manager.

Required Migration Utility Capabilities

The utility must support:

* --source
* Destination through DATABASE_URL
* --dry-run
* --verify-only
* --batch-size
* --verbose
* Optional --allow-non-empty-target
* Explicit destructive confirmation for any overwrite behaviour
* Machine-readable exit status
* Structured progress logging
* Redacted configuration output

Safety Requirements

The utility must:

1. Open SQLite in read-only mode.
2. Refuse to mutate the source database.
3. Verify the source database is readable.
4. Verify the destination is PostgreSQL 18.
5. Verify destination schema migration status.
6. Detect whether target application tables are empty.
7. Refuse unsafe duplicate imports by default.
8. Support repeatable verification.
9. Record migration metadata.
10. Preserve source IDs.
11. Preserve null values.
12. Preserve binary values.
13. Preserve timestamps according to documented conversion rules.
14. Preserve JSON meaning.
15. Roll back failed destination writes.
16. Never delete the SQLite source.
17. Never overwrite a backup.
18. Produce a final reconciliation report.

Migration Ordering

Determine table load order from foreign-key relationships.

Use one of:

* Topological insertion order
* Deferred constraints
* Controlled temporary constraint strategy

Do not globally disable integrity enforcement without post-load validation.

Handle cycles explicitly.

Batch Loading

Use bounded batches to prevent excessive memory consumption.

For each table:

* Read deterministic batches.
* Transform values through explicit conversion functions.
* Insert through parameterised statements.
* Preserve original primary keys.
* Record source and destination counts.
* Surface the exact table and source key on failure.
* Avoid logging sensitive row contents.

Optimised bulk loading may be used only if it preserves correctness and transaction safety.

Sequence Repair

After inserting explicit IDs:

* Find each generated identity sequence.
* Set it to a safe value based on imported data.
* Handle empty tables.
* Verify a subsequent generated insert does not conflict.

Verification

The migration utility must verify at minimum:

* Table row counts
* Primary-key coverage
* Foreign-key integrity
* Required non-null columns
* Unique constraints
* Invalid boolean values
* Timestamp conversion failures
* Invalid JSON
* Binary length consistency
* Min/max numeric IDs
* Sequence state
* Orphan records
* Known domain invariants

Where practical, compute deterministic per-table reconciliation hashes over canonicalised row values.

Canonicalisation must define:

* Column order
* Null representation
* Timestamp representation
* JSON key ordering
* Binary encoding
* Boolean representation
* Numeric representation

A hash mismatch must be treated as a failed migration, not a warning.

Migration Report

Produce a report containing:

* Source database path
* Source database size
* Destination identity with credentials redacted
* Migration start and completion timestamps
* Schema version
* Tables migrated
* Rows read
* Rows inserted
* Rows rejected
* Sequence resets
* Verification results
* Hash comparisons
* Warnings
* Errors
* Overall pass/fail status

⸻

Phase 10: Migrations and Schema Lifecycle

Provide canonical commands for:

* Apply migrations
* Show migration status
* Create a migration
* Roll back where supported and safe
* Reset a local development database
* Seed development data
* Validate schema drift

Migrations must be:

* Ordered
* Versioned
* Deterministic
* Committed to source control
* Transactional where PostgreSQL permits
* Safe against accidental repeated execution
* Independent of container first-start behaviour
* Executable in development, tests, CI, and production-like deployment

Do not modify an already-released migration when the repository’s migration policy requires immutable migrations. Add a corrective migration instead.

If existing SQLite migrations remain valuable as migration history:

* Archive them clearly as SQLite-only history.
* Ensure they are not executed against PostgreSQL.
* Do not mix SQLite and PostgreSQL SQL within the same ambiguous migration path.

⸻

Phase 11: Tests

PostgreSQL compatibility must be proven through automated tests.

Real PostgreSQL Required

Database integration tests must use a real PostgreSQL 18 instance.

Acceptable approaches include:

* Testcontainers
* A dedicated Compose test service
* A PostgreSQL CI service container

Do not claim PostgreSQL compatibility based solely on:

* SQLite tests
* Mocked repositories
* SQL string snapshots
* In-memory substitutes

Test Isolation

Implement deterministic isolation using one strategy appropriate to the repository:

* Dedicated database per test worker
* Dedicated schema per test worker
* Transaction rollback per test
* Truncation with identity reset
* Ephemeral Testcontainers instance

Parallel tests must not interfere with one another.

Required Test Coverage

Add or update tests for:

* Database startup
* Configuration validation
* Connection pooling
* Graceful shutdown
* Migration from an empty database
* Re-running migrations
* Repository CRUD
* Foreign keys
* Unique constraints
* Check constraints
* Identity generation
* RETURNING
* Transactions
* Rollback
* Upserts
* Pagination ordering
* Boolean conversion
* Timestamp conversion
* JSON/JSONB behaviour
* Binary data
* Nullable values
* Error mapping
* Concurrent state changes
* SQLite-to-PostgreSQL data migration
* Sequence reset after imported IDs
* Application startup with PostgreSQL unavailable
* Application recovery after PostgreSQL becomes available
* Existing API and UI flows that depend on persistence

Where existing tests encode SQLite-specific behaviour, determine whether they represent:

* A required application contract
* An accidental SQLite implementation detail

Preserve required contracts. Replace accidental implementation assumptions with PostgreSQL-correct tests.

Fixture Migration Test

Create a representative SQLite fixture containing:

* Multiple related tables
* Null values
* Boolean values
* Timestamps
* JSON
* Unicode
* Empty strings
* Binary data where applicable
* Explicit IDs
* Foreign keys
* Uniqueness boundaries
* Maximum representative numeric values
* Any application-specific edge cases

Automate:

1. Start empty PostgreSQL 18.
2. Apply PostgreSQL migrations.
3. Run SQLite-to-PostgreSQL migration.
4. Run reconciliation verification.
5. Run application repository tests against the migrated data.
6. Verify the next generated identity value.

⸻

Phase 12: Scripts, Makefile, and Developer Commands

Update all scripts and Makefile targets that currently assume SQLite.

Provide repository-consistent commands equivalent to:

db-up
db-down
db-restart
db-status
db-logs
db-shell
db-migrate
db-migrate-status
db-seed
db-reset
db-test
db-backup
db-restore
db-migrate-sqlite
db-verify

Use the project’s package manager and naming conventions.

Safety Gates

Destructive commands such as reset, drop, or restore must:

* Refuse production environments by default.
* Require explicit confirmation or a force flag.
* Display the target host, port, database, and user with secrets redacted.
* Avoid ambiguous defaults.
* Never target a database inferred solely from an unvalidated environment variable.

PostgreSQL Shell

Provide a simple command to open psql inside the PostgreSQL container without requiring a host-installed client.

Backup and Restore

Provide documented commands using PostgreSQL-native tooling such as:

* pg_dump
* pg_restore

Backups must:

* Include schema and data.
* Fail on command errors.
* Avoid embedding credentials in command history where practical.
* Use timestamped filenames.
* Never overwrite an existing backup silently.

Restore documentation must clearly state whether the destination must be empty.

⸻

Phase 13: Environment Configuration

Update:

* .env.example
* Development environment documentation
* Test environment variables
* Compose environment
* CI secrets/variables
* Deployment examples
* Any configuration schemas
* Runtime validation code

Remove obsolete variables such as:

* SQLite file paths
* SQLite journaling configuration
* Busy timeout settings
* SQLite WAL toggles
* In-memory SQLite flags

Do not remove variables until all references have been eliminated.

Example development configuration:

DATABASE_URL=postgresql://application_user:development_password@localhost:5432/application
POSTGRES_DB=application
POSTGRES_USER=postgres_admin
POSTGRES_PASSWORD=development_admin_password

These values must be clearly identified as local-development examples only.

Do not commit real secrets.

⸻

Phase 14: CI/CD

Update CI to provision PostgreSQL 18.

CI must:

1. Start PostgreSQL 18.
2. Wait for a valid health check.
3. Install dependencies using the repository lockfile.
4. Apply migrations to an empty database.
5. Verify migration status.
6. Run type checking.
7. Run linting.
8. Run unit tests.
9. Run PostgreSQL integration tests.
10. Run SQLite-to-PostgreSQL fixture migration tests.
11. Run application build.
12. Run end-to-end tests where already supported.
13. Fail on schema drift or migration verification failure.

Use unique test databases or schemas when jobs execute concurrently.

Do not rely on external persistent databases for normal CI.

⸻

Phase 15: Documentation

Update every document that references SQLite or old database commands.

At minimum, document:

* PostgreSQL 18 requirement
* Container startup
* Environment configuration
* Database connection architecture
* Migration commands
* Development workflow
* Test workflow
* PostgreSQL shell access
* Seeding
* Resetting local data
* Backup
* Restore
* Existing SQLite data migration
* Reconciliation verification
* Troubleshooting
* Production deployment considerations
* Credential separation
* Pool sizing
* TLS configuration
* Upgrade procedure for future PostgreSQL 18 patch releases
* Major-version upgrade warning
* Correct PostgreSQL 18 volume layout

Create or update:

docs/postgresql-migration.md

Include a cutover runbook:

1. Announce maintenance window where required.
2. Stop application writes.
3. Back up SQLite.
4. Start PostgreSQL.
5. Apply PostgreSQL migrations.
6. Run dry-run migration.
7. Run actual migration.
8. Run verification.
9. Reset and verify identity sequences.
10. Start application against PostgreSQL.
11. Execute smoke tests.
12. Monitor errors and database health.
13. Retain SQLite backup unchanged.
14. Define rollback criteria.
15. Define rollback procedure.

A rollback must restore the application configuration and the untouched SQLite source. Do not imply that writes made to PostgreSQL after cutover can automatically be reverse-migrated unless such tooling is actually implemented and tested.

⸻

Phase 16: Dependency Cleanup

After the PostgreSQL implementation is validated:

* Remove unused SQLite runtime dependencies.
* Remove unused SQLite development dependencies.
* Remove obsolete type packages.
* Remove old SQLite connection factories.
* Remove SQLite-only migration execution code.
* Remove SQLite PRAGMA configuration.
* Remove runtime creation of .db files.
* Remove obsolete Docker mounts.
* Remove old test setup.
* Remove stale scripts.
* Update the lockfile using the repository package manager.
* Confirm no browser bundle contains PostgreSQL or SQLite server dependencies.

Retain only the SQLite dependency required by the one-shot data migration utility, if that utility reads SQLite directly.

If retained:

* Keep it isolated from application runtime code.
* Classify it as a development or migration dependency where appropriate.
* Document why it remains.
* Ensure production runtime startup does not load it.
* Ensure production container builds may omit it if migration is performed separately.

⸻

Phase 17: Static Verification

Perform repository-wide searches after implementation.

There must be no unexplained runtime references to:

better-sqlite3
sqlite3
bun:sqlite
libsql
PRAGMA
last_insert_rowid
INSERT OR REPLACE
INSERT OR IGNORE
.sqlite
.sqlite3
DATABASE_PATH
SQLITE_

Any remaining match must be one of:

* The intentional SQLite data migration utility
* Archived migration documentation
* Migration test fixtures
* Historical documentation clearly marked as such

Document every intentional remaining reference.

⸻

Phase 18: Runtime Verification

Execute all applicable repository commands.

At minimum:

1. Install dependencies.
2. Start PostgreSQL 18 from a clean volume.
3. Confirm health.
4. Apply migrations.
5. Check migration status.
6. Seed data if supported.
7. Start the application.
8. Execute a database-backed smoke test.
9. Run type checking.
10. Run linting.
11. Run tests.
12. Run integration tests.
13. Run migration fixture tests.
14. Run the production build.
15. Start the production build where practical.
16. Exercise primary persistence-backed user journeys.
17. Stop and recreate the PostgreSQL container.
18. Verify named-volume persistence.
19. Reset the development database.
20. Reapply migrations from zero.
21. Verify no manual intervention is required.

Do not report a command as passing unless it was actually executed successfully.

If an existing unrelated failure prevents complete validation:

* Prove it existed independently where possible.
* Document the exact command and failure.
* Continue all other possible validation.
* Do not hide it.
* Do not weaken tests to force a pass.

⸻

PostgreSQL Operational Requirements

Pool Sizing

Do not use arbitrary pool sizes.

Choose conservative defaults based on:

* Number of application processes
* Number of Node.js workers
* Test concurrency
* PostgreSQL max_connections
* Deployment topology

Document:

total potential connections =
application instances × workers per instance × pool maximum

Ensure the total leaves capacity for:

* Migrations
* Operational access
* Monitoring
* Maintenance

Timeouts

Support appropriate configuration for:

* Connection timeout
* Idle pool timeout
* Statement timeout
* Lock timeout where needed
* Transaction timeout strategy

Do not globally set aggressive query timeouts without reviewing long-running workflows.

Application Health

Differentiate:

* Liveness: the Node.js process is functioning.
* Readiness: the application can perform required PostgreSQL operations.

Readiness checks must be lightweight and bounded by timeout.

Do not expose database errors or credentials through public health endpoints.

Logging

Log useful database lifecycle events:

* Pool initialised
* Migration started/completed
* Connection failure
* Transaction retry
* Graceful shutdown
* Verification failure

Do not log:

* Passwords
* Raw secrets
* Full sensitive SQL parameters
* Complete connection URLs
* Entire migrated records

Security

Ensure:

* All values use parameterised queries.
* Dynamic SQL identifiers use strict allowlists.
* The runtime role is least privilege.
* The database port is not publicly exposed by default.
* TLS can be enabled for external PostgreSQL deployments.
* Production does not use development passwords.
* Error responses do not reveal schema details.
* SQL migration files are reviewed for unsafe privilege grants.
* User-controlled values cannot alter search_path.
* Application queries do not rely on a mutable or unsafe search_path.

Prefer explicit schema qualification where the application uses a non-default schema.

⸻

Required Repository Deliverables

The completed implementation must include, where applicable:

1. PostgreSQL container configuration.
2. Correct PostgreSQL 18 persistent volume configuration.
3. Environment examples.
4. PostgreSQL connection pool module.
5. Runtime configuration validation.
6. Converted schema migrations.
7. Converted database access layer.
8. Converted SQL.
9. Converted transaction handling.
10. PostgreSQL error mapping.
11. SQLite-to-PostgreSQL data migration utility.
12. Data reconciliation utility.
13. PostgreSQL seed process.
14. Updated tests.
15. Real PostgreSQL integration test infrastructure.
16. Migration fixture tests.
17. Updated CI/CD.
18. Updated Makefile and scripts.
19. Backup and restore commands.
20. Updated README and database documentation.
21. Migration audit document.
22. Cutover and rollback runbook.
23. Dependency cleanup.
24. Final verification report.

⸻

Acceptance Criteria

The migration is complete only when all applicable criteria pass.

Runtime

* The application starts with PostgreSQL 18.
* The application does not access SQLite during normal execution.
* Database-backed user journeys work.
* Application shutdown closes PostgreSQL resources.
* Database unavailability produces controlled failures.
* Credentials remain server-side.

Schema

* A clean PostgreSQL instance can be migrated from zero.
* All tables, indexes, constraints, triggers, and views are present.
* Schema migrations are deterministic.
* Schema drift checks pass.
* Foreign keys are enforced.
* Identity generation works after imported IDs.
* Domain constraints match intended behaviour.

Data

* Existing SQLite data can be migrated.
* IDs are preserved.
* Relationships are preserved.
* Row counts reconcile.
* Verification checks pass.
* Sequence state is correct.
* Invalid source data is reported, not silently dropped.
* The SQLite source remains untouched.

SQL and Application Behaviour

* All SQL is PostgreSQL-compatible.
* All external values are parameterised.
* Upsert semantics are intentionally preserved.
* Pagination is deterministic.
* Timezone behaviour is documented and tested.
* Boolean, JSON, binary, numeric, and null handling are tested.
* Concurrency-sensitive workflows are safe.

Containers

* PostgreSQL health checks pass.
* Persistent data survives container recreation.
* No obsolete Compose version field exists.
* No hard-coded production secrets exist.
* The application uses the Compose service hostname inside containers.
* PostgreSQL 18 volume layout is correct.

Quality

* Type checking passes.
* Linting passes.
* Tests pass.
* Integration tests use real PostgreSQL.
* Production build passes.
* No unexplained SQLite runtime references remain.
* No unresolved migration TODOs remain.
* Documentation matches actual commands.

⸻

Required Execution Order

Execute the work in this order:

1. Read repository instructions.
2. Audit architecture and SQLite usage.
3. Capture the existing schema and behavioural contracts.
4. Create the migration audit document.
5. Select the PostgreSQL driver/dialect strategy.
6. Add PostgreSQL container infrastructure.
7. Add connection and configuration infrastructure.
8. Implement PostgreSQL schema migrations.
9. Convert the data access layer.
10. Convert raw SQL.
11. Propagate asynchronous API changes.
12. Convert transactions and error handling.
13. Implement SQLite data migration.
14. Implement reconciliation verification.
15. Update seeds and fixtures.
16. Update tests.
17. Update scripts and Makefiles.
18. Update CI/CD.
19. Update documentation.
20. Remove obsolete SQLite runtime code.
21. Run static verification.
22. Run complete runtime validation.
23. Fix all migration-related defects.
24. Produce the final report.

Commit-sized changes should remain logically grouped even when no actual Git commits are requested.

Do not mix broad unrelated refactoring into the migration.

⸻

Final Response Format

After completing the implementation, provide a concise but complete report using these sections:

Migration Status

State one of:

* COMPLETE
* COMPLETE WITH DOCUMENTED EXTERNAL BLOCKERS
* INCOMPLETE

Do not report COMPLETE unless the acceptance criteria were actually validated.

Architecture Selected

Include:

* Existing data access technology
* PostgreSQL driver/dialect selected
* Why it was retained or replaced
* Connection pool design
* Migration framework
* PostgreSQL container image/tag
* Database role model

Files Changed

Group by:

* Application source
* Schema and migrations
* Data migration
* Tests
* Containers
* CI/CD
* Scripts
* Documentation
* Removed files

Schema Decisions

List important decisions for:

* IDs
* Timestamps
* Booleans
* JSON
* Binary data
* Numeric precision
* Case-insensitive text
* Full-text search
* Constraints
* Indexes

SQLite Data Migration

Include:

* Command
* Safety behaviour
* Verification method
* Sequence handling
* Known source-data issues

Validation Results

For every command run, include:

Command:
Result:
Relevant output:

Do not claim tests were run when they were not.

Remaining SQLite References

List each intentional remaining reference and why it remains.

Risks and Follow-Up

Include only genuine unresolved risks or external blockers.

Do not include speculative enhancements unrelated to the PostgreSQL migration.