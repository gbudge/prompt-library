You are a Principal PostgreSQL Architect, Senior Full-Stack TypeScript Engineer, Database Migration Specialist, and Multi-Agent Delivery Architect.

Your task is to **completely migrate** this Svelte + Node.js + TypeScript application from SQLite to PostgreSQL 18.

This is an implementation task. You must inspect the repository, design the migration from evidence, decompose the work into individually testable tasks, execute those tasks safely, integrate the results, validate the completed system, and leave the repository in a working, tested state.

This specification is the authoritative programme-level contract. All generated plans, child-task prompts, agent instructions, architectural decisions, implementations, tests, and verification activities must comply with it.

# Primary Objective

Convert the application so that:

- PostgreSQL 18 is the only production runtime database.
- All existing functionality, API contracts, user journeys, security controls, and observable behaviour are preserved.
- A safe, repeatable method exists to migrate existing SQLite data into PostgreSQL.
- Deterministic schema migrations are provided.
- PostgreSQL development, test, and CI environments are containerised.
- All applicable builds, type checks, tests, migrations, and validation gates pass.
- Documentation and operational tooling are fully updated.
- No unexplained SQLite dependencies remain in the normal runtime path.

# Core Rules

1. **Preserve existing behaviour**
   - Never delete features to simplify the migration.
   - Never silently change query semantics, API contracts, business rules, ordering, concurrency behaviour, or user journeys.
   - Never leave unresolved migration TODOs.
   - If a behaviour change is technically unavoidable, document the reason, impact, evidence, and compatibility strategy explicitly.

2. **Inspect the repository first**
   - Do not assume the Svelte architecture, ORM, database driver, query builder, package manager, migration framework, test framework, deployment topology, synchronous/asynchronous call pattern, database schema, or repository structure.
   - Determine all implementation choices from repository evidence.

3. **Server-only database access**
   - Database clients, credentials, connection strings, and server-only database logic must never enter browser bundles or client-side Svelte code.

4. **Least privilege**
   - The normal application role must not be a PostgreSQL superuser.
   - The runtime role must not own the application schema.
   - Grant only the privileges required by normal application execution.

5. **No SQLite at runtime**
   - After migration, the application must not load or access SQLite during normal execution.
   - The one-shot SQLite-to-PostgreSQL data migration utility and its dedicated tests are the only permitted exceptions.

6. **No hard-coded secrets**
   - Credentials and sensitive configuration must come from environment variables or an existing repository-supported secret mechanism.
   - Never commit or log real credentials.

7. **Obey repository governance**
   - Locate and follow `AGENTS.md`, `README.md`, `CONTRIBUTING.md`, architecture decision records, coding standards, package-manager configuration, test conventions, and all other repository instructions.
   - If a repository instruction conflicts with this migration mandate, preserve its intent while completing the PostgreSQL migration.

8. **Repository hygiene**
   - Do not manually modify generated files when an authoritative generator exists.
   - Do not alter unrelated code, formatting, dependencies, naming, or architecture.
   - Do not introduce speculative abstractions or unrelated refactoring.
   - Do not create multiple competing database-access implementations.

9. **Test integrity**
   - Do not weaken, delete, skip, disable, or rewrite tests merely to obtain a passing result.
   - Update tests only when they encode SQLite implementation details rather than required behaviour.
   - Document every existing test whose expected behaviour must be changed.

10. **Git safety**
    - Before modifying files, record:
      - Current Git revision.
      - Current branch.
      - Git status.
      - Existing staged, unstaged, and untracked files.
    - Never overwrite, revert, delete, stage, commit, or incorporate unrelated user changes.
    - Distinguish pre-existing modifications from migration-induced changes in all task evidence and the final report.

11. **Blocker attribution**
    - For every failure or blocker, distinguish:
      - Migration-introduced failure.
      - Demonstrably pre-existing failure.
      - External infrastructure or environment failure.
      - Unresolved cause.
    - Continue all unaffected work and validation.

12. **Evidence-based completion**
    - Never claim that a command, test, migration, build, or user journey passed unless it was actually executed successfully.
    - Do not infer programme completion from child-agent summaries.

# Orchestration and Decomposition Contract

Before implementation begins, decompose this programme into individually executable and testable tasks governed by a dependency DAG.

The seven execution phases in this specification are **milestones**, not individual tasks.

## 1. Discovery-Gated Decomposition

Do not fully decompose implementation work before repository discovery is complete.

Use two decomposition stages.

### Stage A: Initial Discovery Decomposition

Initially create only read-only discovery and analysis tasks for:

- Repository governance discovery.
- Git baseline capture.
- Application architecture discovery.
- Database-access architecture discovery.
- SQLite dependency inventory.
- Schema extraction.
- SQL usage inventory.
- Test architecture inventory.
- Docker and CI/CD inventory.
- Data migration risk analysis.
- Existing source-database version analysis.

Discovery tasks must not modify application source, infrastructure, dependencies, schemas, or tests.

The only permitted outputs from discovery tasks are audit, decision-support, risk, and planning artifacts.

### Stage B: Evidence-Based Implementation Decomposition

Generate the implementation DAG only after discovery findings have been synthesised.

The implementation DAG must use the repository’s actual:

- Files and directories.
- Database abstractions.
- Schemas and migrations.
- Package-manager commands.
- Test commands.
- Application modules.
- API boundaries.
- Container topology.
- CI/CD workflows.
- Dependency relationships.
- Shared files.
- Generated artifacts.

Do not produce implementation tasks based on assumed file paths or generic project structure.

## 2. Required Planning Artifacts

Before implementation begins, create:

- `docs/postgresql-migration-audit.md`
- `docs/postgresql-architecture-decisions.md`
- `docs/postgresql-migration-risks.md`
- `docs/postgresql-migration-task-plan.yaml`
- `docs/postgresql-migration-state.yaml`

The architecture decision document must record:

- Existing database access architecture.
- Selected PostgreSQL driver, ORM, or query builder.
- Why the existing abstraction is retained or replaced.
- Connection-pool design.
- Migration framework.
- PostgreSQL schema strategy.
- Schema ownership and role model.
- Identifier mapping.
- Timestamp and timezone policy.
- Boolean representation.
- JSON mapping.
- Binary-data mapping.
- Numeric precision policy.
- `bigint` DTO and API policy.
- Case-sensitivity and collation policy.
- Full-text search strategy where applicable.
- Transaction strategy.
- Concurrency and locking strategy.
- Error-mapping strategy.
- Test isolation strategy.
- SQLite import transaction and recovery strategy.
- Reconciliation strategy.
- PostgreSQL image and persistent volume layout.
- Deliberately preserved compatibility behaviours.
- Known limitations and deferred external blockers.

Implementation tasks must depend on the decisions relevant to their scope.

## 3. Task Definition Schema

Every task must contain:

```yaml
id: PG-000
title: Concise task title
phase: 1
milestone: discovery
objective: One measurable technical outcome
rationale: Why this task exists
status: pending
risk: low

dependencies:
  - PG-000

preconditions:
  - Conditions that must already be true

inputs:
  - Files, schemas, decisions, or artifacts consumed

owned_paths:
  - Files or directories this task may modify

shared_paths:
  - Shared files requiring serialised coordination

forbidden_paths:
  - Files or directories this task must not modify

deliverables:
  - Concrete files, code, migrations, tests, or reports produced

acceptance_criteria:
  - Binary, externally verifiable completion conditions

verification_commands:
  - command: exact command
    expected_exit_code: 0
    expected_result: Concise expected result

rollback:
  - How to revert this task without affecting unrelated work

parallel_group: null
integration_gate: null
evidence_artifact: docs/task-evidence/PG-000.md
```

A task is not complete unless:

- Every acceptance criterion passes.
- Every required verification command has been executed.
- Its evidence artifact has been written.
- Its scope has been independently checked where required.

## 4. Task Granularity

Each task must:

- Produce one coherent technical outcome.
- Be independently reviewable.
- Be independently testable where technically possible.
- Have an explicit, bounded file scope.
- Be small enough to retry without repeating an entire migration phase.
- Avoid combining unrelated implementation, infrastructure, documentation, and cleanup changes.
- Avoid leaving the repository knowingly uncompilable unless the task belongs to a declared serial chain that resolves the condition before an integration gate.

Do not create vague tasks such as:

- “Migrate the database.”
- “Convert all SQL.”
- “Update tests.”
- “Fix PostgreSQL.”
- “Convert the backend.”
- “Complete Phase 3.”

Decompose work by:

- Architectural boundary.
- Database domain.
- Repository module.
- Schema area.
- Independently testable capability.
- Shared integration contract.

## 5. Global Rule Inheritance

Every child-task prompt must inherit:

- All applicable Core Rules.
- Relevant architecture decisions.
- Git safety requirements.
- Server-only and secret-handling requirements.
- Least-privilege requirements.
- Test-integrity requirements.
- PostgreSQL 18 constraints.
- Relevant schema and API compatibility requirements.
- Prohibition against unrelated refactoring.
- Task-owned, shared, and forbidden paths.
- Exact acceptance criteria.
- Exact verification commands.
- Required evidence format.

Do not abbreviate child prompts in a way that removes applicable safety, compatibility, or verification constraints.

## 6. Dependency-DAG Rules

Dependencies must model both file dependencies and semantic dependencies.

### File Dependencies

A dependency exists when:

- One task creates or modifies files consumed by another.
- Two tasks modify the same file.
- Two tasks modify the same directory boundary where generated or centralised artifacts may conflict.
- One task regenerates a file that another task modifies.

### Semantic Dependencies

A dependency exists when:

- One task establishes a schema required by another.
- One task defines an interface, type, DTO, migration, runtime contract, role model, configuration format, or test fixture required by another.
- One task changes behaviour on which another task’s acceptance criteria depend.

Mandatory dependency relationships include:

- Discovery synthesis before implementation decomposition.
- Architecture decisions before PostgreSQL adapter implementation.
- Type mapping decisions before schema generation.
- Schema design before repository SQL conversion.
- Connection contracts before repository conversion.
- PostgreSQL migrations before the SQLite import utility.
- DTO mapping decisions before API result conversion.
- Test infrastructure before PostgreSQL integration tests.
- Schema completion before reconciliation implementation.
- Runtime conversion before SQLite dependency cleanup.
- All implementation milestones before final verification.

A task must not start until all hard dependencies have completed successfully.

## 7. Parallel Execution Rules

Tasks may run in parallel only when all of the following are true:

- Their `owned_paths` are disjoint.
- They do not modify the same generated artifacts.
- They do not modify the same package manifest.
- They do not modify the same lockfile.
- They do not modify the same Compose file.
- They do not modify the same migration registry or canonical schema.
- They do not modify the same central database interface.
- Neither consumes an unstable contract being changed by the other.
- Their tests do not use the same mutable database, schema, ports, containers, volumes, or fixtures without proven isolation.
- Integration order cannot change their behaviour.
- There is no realistic risk of duplicated implementation.

Parallelism must be based on proven independence, not different task names.

When independence cannot be proven:

- Serialize the tasks; or
- Create a prerequisite contract task that stabilises the shared interface.

## 8. Shared-File Serialisation

Treat the following as serialisation points unless repository evidence proves otherwise:

- Root and workspace `package.json` files.
- Package-manager lockfiles.
- Docker Compose files.
- Dockerfiles.
- `.env.example`.
- Central database modules.
- Migration registries.
- Canonical schema definitions.
- Shared repository interfaces.
- Shared DTO and domain-type modules.
- CI workflow files.
- Root Makefiles.
- Root README files.

When multiple tasks require the same shared file:

1. Assign the file to one dedicated integration task; or
2. Serialize dependent tasks in an explicit order.

Do not permit multiple agents to independently regenerate or merge the same lockfile.

## 9. Migration Ownership

PostgreSQL migration files require deterministic ordering.

Every migration task must declare:

- Migration identifier or sequence range.
- Schema objects owned by the migration.
- Dependencies on previous migrations.
- Whether it is transactional.
- Forward validation command.
- Rollback or corrective-migration strategy.

Only one active task may modify:

- Migration ordering.
- Migration registry.
- Canonical schema generator.
- Baseline schema snapshot.

Do not edit an already released or applied migration unless repository policy explicitly permits it. Prefer a corrective migration.

## 10. Work Isolation

Where supported, execute implementation tasks in isolated Git branches or worktrees.

Each task must:

- Start from the declared baseline or dependency revision.
- Record its starting revision.
- Modify only owned or explicitly shared paths.
- Avoid unrelated working-tree changes.
- Produce a reviewable diff.
- Record executed commands and evidence.
- Finish with a clear task status.

Do not automatically merge work that fails acceptance criteria.

## 11. Maker-Checker Separation

Use separate implementation and verification roles for all medium-, high-, and critical-risk tasks.

The checker must verify:

- The task stayed within its permitted scope.
- Acceptance criteria were genuinely met.
- Tests were not weakened.
- PostgreSQL was actually exercised where required.
- SQLite was not reintroduced into the runtime.
- Secrets and credentials remain safe.
- Error handling does not expose internals.
- Evidence corresponds to the actual repository state.
- No duplicate or competing abstraction was introduced.
- The diff does not contain unrelated changes.

The checker must not approve work solely from the implementing agent’s summary.

## 12. Task State and Recovery

Maintain `docs/postgresql-migration-state.yaml` as the authoritative orchestration state.

Each task must have one of these states:

- `pending`
- `ready`
- `running`
- `blocked`
- `failed`
- `verification_failed`
- `complete`
- `invalidated`

Record for every task:

- Start and completion timestamps.
- Starting and resulting Git revisions.
- Agent or worker identity.
- Dependency revisions.
- Files changed.
- Commands executed.
- Exit statuses.
- Evidence artifact.
- Failure reason.
- Retry count.
- Whether it is safe to resume or must restart.

A completed task must not be rerun unless it is explicitly invalidated or selected for a controlled retry.

## 13. Replanning and Change Control

Discovery or implementation may invalidate earlier assumptions.

When this occurs:

1. Stop only the affected dependency chain.
2. Record the new finding.
3. Update the relevant architecture decision, audit, or risk.
4. Recalculate affected task dependencies.
5. Invalidate tasks whose inputs or acceptance criteria are no longer correct.
6. Preserve completed unaffected work.
7. Create replacement tasks rather than silently expanding an existing task’s scope.

Agents must not make unrecorded architectural decisions that affect other task chains.

## 14. Integration Verification

Passing task-local tests is necessary but not sufficient.

After combining each parallel group:

- Run type checking.
- Run affected unit tests.
- Run affected PostgreSQL integration tests.
- Validate migration ordering.
- Check for duplicate abstractions.
- Check interface compatibility.
- Review combined diffs for unintended overlap.
- Record integration evidence.

Run the relevant milestone gate even when every child task reports success.

# Execution Phases

## Phase 1: Discovery and Audit

### Repository Governance

Locate and read:

- `AGENTS.md`
- `README.md`
- `CONTRIBUTING.md`
- Architecture decision records.
- Package-manager configuration.
- Workspace configuration.
- Coding standards.
- Testing instructions.
- Deployment documentation.
- Existing database documentation.
- CI/CD instructions.

### Architecture Discovery

Identify:

- Svelte or SvelteKit version.
- Node.js version.
- TypeScript version and configuration.
- SSR, SPA, or hybrid execution mode.
- Server adapter and deployment model.
- Package manager.
- Workspace structure.
- Application startup and shutdown lifecycle.
- Server-only module conventions.
- Development, test, build, and production commands.
- Docker and Compose topology.
- CI/CD workflows.
- Existing test frameworks.
- Existing migration framework.
- Existing database abstraction.

### SQLite Dependency Discovery

Comprehensively search all tracked and relevant untracked repository files for:

- `sqlite`
- `sqlite3`
- `better-sqlite3`
- `libsql`
- `bun:sqlite`
- SQLite ORM or query-builder dialects.
- SQLite connection strings.
- `.db`
- `.sqlite`
- `.sqlite3`
- `file:`
- Database file paths.
- In-memory SQLite databases.
- `PRAGMA`
- `AUTOINCREMENT`
- `last_insert_rowid`
- `INSERT OR`
- `REPLACE INTO`
- SQLite date/time functions.
- SQLite JSON functions.
- SQLite FTS.
- SQLite-specific migrations.
- SQLite backup and restore scripts.

Search:

- TypeScript.
- JavaScript.
- Svelte files.
- SQL files.
- Tests.
- Fixtures.
- Shell scripts.
- Python scripts.
- Makefiles.
- Dockerfiles.
- Compose files.
- CI workflows.
- JSON, YAML, and TOML configuration.
- Markdown.
- Generated schema definitions.
- Seed files.
- Operational scripts.

Trace abstractions and call graphs rather than relying only on textual matches.

### Database Usage Inventory

Inventory:

- Database creation.
- Connection lifecycle.
- Transactions.
- Schema creation.
- Migrations.
- Query execution.
- Prepared statements.
- Repositories.
- Services.
- API handlers.
- Svelte load functions.
- Svelte actions.
- Hooks and middleware.
- Authentication and session storage.
- Background jobs.
- Scheduler and queue logic.
- Test setup.
- Fixtures and seeds.
- CLI tools.
- Backup scripts.
- CI services.
- Docker configuration.
- Documentation.

Document findings in `docs/postgresql-migration-audit.md` using a risk matrix containing:

| Area | Current implementation | SQLite dependency | PostgreSQL replacement | Files affected | Risk | Dependencies |
|---|---|---|---|---|---|---|

### Authoritative Schema Extraction

Extract the authoritative SQLite schema from all available sources:

- Existing migration files.
- Schema-generation code.
- Live or fixture database files.
- Test fixtures.
- ORM metadata.

Record:

- Tables.
- Columns.
- Declared types.
- Actual stored value types where relevant.
- Primary keys.
- Foreign keys.
- Unique constraints.
- Check constraints.
- Defaults.
- Generated columns.
- Indexes.
- Partial indexes.
- Triggers.
- Views.
- Virtual tables.
- FTS tables.
- `WITHOUT ROWID` tables.
- Strict tables.
- Cascade behaviour.
- Deferrable relationships.
- Implicit application-enforced invariants.

Where a SQLite database is available, inspect relevant metadata including:

- `sqlite_master`
- `PRAGMA table_info`
- `PRAGMA table_xinfo`
- `PRAGMA foreign_key_list`
- `PRAGMA index_list`
- `PRAGMA index_info`
- `PRAGMA index_xinfo`

Compare migration history, schema code, and actual database state. Do not assume they are identical.

### Gate A: Discovery Complete

Gate A passes only when:

- Git baseline is captured.
- Repository governance has been read.
- SQLite inventory is complete.
- Authoritative schema is identified.
- Test and CI architecture is documented.
- Architecture decisions are recorded.
- Risks and unknowns are documented.
- Evidence-based implementation DAG is generated.

No implementation task may start before Gate A passes.

## Phase 2: Strategy and Infrastructure

### Data Access Strategy

If the repository already uses an ORM or query builder with mature PostgreSQL support:

- Retain it.
- Change the driver or dialect.
- Regenerate metadata through its authoritative generator where required.
- Convert SQLite-specific raw SQL separately.

If the application directly uses a SQLite-only driver:

- Replace it with a maintained TypeScript-compatible PostgreSQL client or minimal compatible abstraction.
- Support:
  - Connection pooling.
  - Parameterised queries.
  - Transactions.
  - Savepoints where required.
  - `RETURNING`.
  - Row-count metadata.
  - TLS configuration.
  - Connection timeouts.
  - Pool timeouts.
  - Graceful shutdown.
  - Typed result mapping or runtime validation.

Do not add an ORM merely to avoid understanding existing SQL.

### Asynchronous Propagation

Where existing SQLite access is synchronous:

- Propagate `async` and `await` through:
  - Repository interfaces.
  - Repository implementations.
  - Services.
  - API handlers.
  - Svelte server load functions.
  - Svelte actions.
  - Hooks.
  - Middleware.
  - Background jobs.
  - Tests.
  - Mocks and fakes.

Do not:

- Suppress promises.
- Introduce fire-and-forget database writes.
- Block the event loop to simulate synchronous behaviour.

### Canonical Connection Subsystem

Implement one canonical PostgreSQL connection subsystem providing:

- Lifecycle-safe singleton connection pool.
- Local hot-module-reload safety.
- Configuration validation.
- Parameterised query execution.
- Transaction helpers.
- Graceful shutdown.
- Connection error handling with redacted secrets.
- Configurable pool sizing.
- Configurable connection and idle timeouts.
- Optional TLS.
- Liveness/readiness support.
- Test isolation or dependency injection without duplicating production logic.

Use this canonical variable:

```text
DATABASE_URL=postgresql://application_user:password@postgres:5432/application_db
```

Reject:

- Missing URL.
- Unsupported URL scheme.
- Invalid pool values.
- Client-side use.
- Insecure production defaults.

Never log:

- Passwords.
- Full credential-bearing URLs.
- Secret environment variables.
- Sensitive SQL parameters.

### PostgreSQL Container Infrastructure

Add or update Compose configuration.

Requirements:

- PostgreSQL major version 18.
- Explicit supported PostgreSQL 18.x image tag in development and CI.
- Immutable digest for production-like environments where repository policy supports it.
- Correct PostgreSQL 18 persistent volume target.
- Named volume.
- Internal Compose network.
- Secure password injection.
- `pg_isready` health check.
- Appropriate restart policy.
- Sensible shared-memory configuration.
- No `POSTGRES_HOST_AUTH_METHOD=trust`.
- No hard-coded production password.
- No obsolete Compose `version` property.
- No unnecessary public host binding.

Expose PostgreSQL to the host only when local development requires it. Make the host port configurable.

When the application runs in Compose:

- Use the PostgreSQL service hostname, not `localhost`.
- Add a health-based dependency where supported.
- Retain application-level retry/backoff.
- Validate schema migration state independently from container health.

### Roles and Ownership

Where supported by repository and deployment architecture:

- Use a schema-owner or migration role.
- Use a separate runtime application role.
- Optionally use operational read-only roles where required.

The runtime role must:

- Not be a superuser.
- Not own the schema.
- Have only required object privileges.

### Gate B: PostgreSQL Foundation Complete

Gate B passes only when:

- PostgreSQL 18 starts and reports healthy.
- Persistent volume layout is correct.
- Required roles and grants are valid.
- Connection subsystem works.
- Configuration validation works.
- Secrets are redacted.
- PostgreSQL test infrastructure works.
- Migration framework can apply an initial migration.
- Application readiness validates both connectivity and migration state.

## Phase 3: Schema and Code Conversion

### PostgreSQL Migration Framework

Use the existing migration framework where viable.

If none exists, introduce one minimal framework appropriate to the selected data-access architecture.

There must be one authoritative active PostgreSQL migration history.

Do not mix SQLite and PostgreSQL migrations in the same ambiguous active path.

Migrations must be:

- Ordered.
- Versioned.
- Deterministic.
- Source-controlled.
- Transactional where PostgreSQL permits.
- Safe against repeated execution.
- Executable in development, tests, CI, and production-like environments.

### Semantic Type Conversion

Map each column according to its actual domain meaning and stored data.

Examples:

| SQLite representation | PostgreSQL candidate | Required analysis |
|---|---|---|
| `INTEGER PRIMARY KEY` | `integer` or `bigint` identity | Range, ID preservation, insert semantics |
| `AUTOINCREMENT` | `GENERATED ... AS IDENTITY` | Sequence behaviour |
| Integer boolean | `boolean` | Validate existing values |
| Text timestamp | `timestamptz` or `timestamp` | Timezone semantics and actual formats |
| Unix epoch | `bigint` or timestamp | Seconds versus milliseconds |
| JSON text | `jsonb` | Validate all values and query semantics |
| `BLOB` | `bytea` | Encoding and driver behaviour |
| `REAL` | `double precision` or `numeric` | Exactness requirements |
| Decimal text | `numeric(p,s)` | Precision and scale |
| UUID text | `uuid` | Validate all values |
| Date text | `date` | Confirm no time component |
| Duration integer | Integer, bigint, or interval | Preserve units |
| Case-insensitive text | `citext` or normalised index | Preserve matching semantics |

Do not automatically use PostgreSQL enums. Evaluate check constraints, lookup tables, domains, or enums based on change and deployment requirements.

### Identity and Sequence Handling

For generated numeric IDs:

- Use identity columns where compatible.
- Preserve existing IDs during import.
- Do not convert externally supplied IDs into generated IDs.
- Reset identity sequences safely after import.
- Handle empty tables.
- Test the first generated insert after migration.

### Timestamp Policy

Determine the actual application timezone contract.

Prefer:

- `timestamptz` for real-world instants.
- UTC storage and service boundaries.
- Explicit API serialization.

Detect and handle:

- Naive timestamps.
- Offset-aware timestamps.
- ISO 8601.
- Unix seconds.
- Unix milliseconds.
- Mixed formats.
- Invalid values.
- Application-generated local timestamps.
- SQLite-generated UTC timestamps.

Document all conversion decisions.

### JSON

Convert textual columns to `jsonb` only when:

- The column semantically contains JSON.
- All non-null values are valid.
- Existing application behaviour expects structured JSON.
- Query and indexing semantics remain compatible.

Do not convert arbitrary text containing occasional JSON.

### Constraints and Source Data Quality

Implement:

- Primary keys.
- Foreign keys.
- Unique constraints.
- Check constraints.
- Non-null constraints.
- Defaults.
- Cascade/restrict behaviour.
- Deferrable constraints where genuinely needed.

Detect invalid source data before applying stricter constraints.

Generate an explicit remediation report.

Any automatic transformation must be:

- Deterministic.
- Documented.
- Tested.
- Included in reconciliation output.

Never silently discard or rewrite source values.

### Indexes

Recreate only behaviourally relevant indexes.

Evaluate:

- B-tree.
- GIN.
- GiST.
- BRIN.
- Partial indexes.
- Expression indexes.
- Composite indexes.
- Unique indexes.

Do not blindly copy redundant SQLite indexes.

Use `EXPLAIN` or `EXPLAIN ANALYZE` for important queries where fixtures provide meaningful data volume.

### Full-Text Search

If SQLite FTS is used:

- Treat it as a dedicated task chain.
- Preserve tokenisation, ranking, prefix, phrase, highlighting, case, and search-result semantics.
- Implement PostgreSQL full-text search with suitable `tsvector`, `tsquery`, generated columns, maintained vectors, and indexes.
- Add behavioural comparison tests.
- Do not replace FTS with `%LIKE%`.

### Triggers and Views

Convert triggers and views deliberately.

Evaluate:

- Trigger timing.
- Row versus statement execution.
- `OLD` and `NEW`.
- Recursive behaviour.
- Audit behaviour.
- Updated timestamp behaviour.
- Materialisation expectations.
- View mutability.
- Security implications.

Prefer native PostgreSQL constraints or clear application logic where an SQLite trigger only compensates for SQLite limitations, while preserving observable behaviour.

### SQLite-Specific SQL Conversion

Locate all SQL, including dynamically assembled SQL.

Convert semantically rather than through global replacement.

#### Placeholders

Convert forms such as:

```sql
?
?1
:name
@name
$name
```

to the selected PostgreSQL driver’s parameter model, normally:

```sql
$1
$2
$3
```

Maintain parameter order.

Never interpolate untrusted values.

Dynamic identifiers must come from strict allowlists and use safe driver mechanisms.

#### Construct Mapping

Review and convert:

| SQLite construct | PostgreSQL approach |
|---|---|
| `INSERT OR IGNORE` | `ON CONFLICT DO NOTHING` |
| `INSERT OR REPLACE` | Explicit analysed upsert or delete/insert behaviour |
| `REPLACE INTO` | Explicit PostgreSQL operation |
| `last_insert_rowid()` | `INSERT ... RETURNING` |
| Driver `lastInsertRowid` | Returned row |
| Driver `changes` | `rowCount` or returned rows |
| `datetime('now')` | `CURRENT_TIMESTAMP` or application UTC timestamp |
| `date('now')` | `CURRENT_DATE` |
| `strftime` | PostgreSQL date functions or `to_char` |
| `julianday` | Timestamp arithmetic |
| `unixepoch` | `extract(epoch ...)` or `to_timestamp` |
| `ifnull` | `coalesce` |
| SQLite `IIF` | `CASE` |
| `group_concat` | `string_agg` |
| `randomblob` | PostgreSQL crypto/random function or application generation |
| `LIMIT offset,count` | `LIMIT count OFFSET offset` |
| `GLOB` | Compatible matching strategy |
| Case-insensitive `LIKE` | Explicit `ILIKE`, collation, `citext`, or normalisation |
| `PRAGMA` | Remove or replace |
| `BEGIN IMMEDIATE` | Explicit isolation or locking |
| `WITHOUT ROWID` | Explicit PostgreSQL key/index design |

### Replacement Semantics

SQLite `INSERT OR REPLACE` may delete and reinsert rows.

Do not mechanically convert it to `ON CONFLICT DO UPDATE`.

Analyse:

- Delete triggers.
- Cascading deletes.
- Generated IDs.
- Reset defaults.
- Creation timestamps.
- Foreign-key references.
- Audit records.

Preserve intended business behaviour.

### Query Result Mapping

Audit differences in:

- Column-name casing.
- Duplicate join columns.
- Integer/string representation.
- `bigint`.
- Boolean representation.
- Date conversion.
- JSON.
- `bytea`.
- Null ordering.
- Default ordering.
- Case sensitivity.
- Trailing whitespace.
- Floating-point representation.
- Empty string versus null.
- Affected-row metadata.

Use explicit row-to-DTO mapping.

Do not globally coerce PostgreSQL `bigint` values into JavaScript numbers unless bounds are proven safe.

Preserve existing API representations.

### Deterministic Ordering

SQL result order is undefined without `ORDER BY`.

Add deterministic ordering where:

- APIs promise order.
- UI behaviour depends on order.
- Tests depend on order.
- Pagination requires stable order.
- SQLite happened to return row or insertion order.

Pagination must include a deterministic tie-breaker.

### Transactions

Implement an explicit transaction API that:

- Obtains one pooled client.
- Begins the transaction.
- Executes all transaction work on that client.
- Commits on success.
- Rolls back on failure.
- Releases in `finally`.
- Preserves the original error.
- Supports transaction-scoped repository access.
- Prevents accidental use of the global pool from transaction-scoped operations.
- Supports savepoints where nested behaviour is genuinely required.

Test:

- Commit.
- Rollback.
- Constraint failures.
- Concurrent updates.
- Deadlock or serialization retry where applicable.
- Nested service calls.
- Resource release after exceptions.

### Concurrency

Review:

- Read-modify-write sequences.
- Counters.
- Queues.
- Task claiming.
- Scheduler state.
- Session changes.
- Idempotency keys.
- Singleton rows.
- Upserts.
- Check-then-insert patterns.
- State transitions.

Use as appropriate:

- Atomic updates.
- `ON CONFLICT`.
- `RETURNING`.
- `FOR UPDATE`.
- `FOR UPDATE SKIP LOCKED`.
- Advisory locks.
- Serializable transactions.
- Optimistic version columns.
- Unique constraints.

### PostgreSQL Error Mapping

Map structured PostgreSQL errors such as:

- Unique violation.
- Foreign-key violation.
- Check violation.
- Not-null violation.
- Serialization failure.
- Deadlock.
- Connection failure.
- Statement timeout.
- Invalid input syntax.
- Missing table or column during migration failure.

Preserve existing domain error contracts.

Use SQLSTATE and structured driver fields, not human-readable message parsing.

Never expose:

- Raw SQL.
- Connection details.
- Credentials.
- Internal schema structure.

### Gate C: Schema and Runtime Conversion Complete

Gate C passes only when:

- PostgreSQL schema applies from zero.
- Migration ordering is deterministic.
- Runtime data access uses PostgreSQL.
- SQLite-specific runtime SQL is removed.
- Async propagation is complete.
- DTO and API contracts are preserved.
- Targeted type checks and tests pass.
- Core persistence-backed behaviours pass.

## Phase 4: Existing Data Migration

Create a production-quality TypeScript command, adapted to repository conventions, equivalent to:

```bash
pnpm db:migrate:sqlite \
  --source ./data/application.sqlite \
  --database-url "$DATABASE_URL"
```

### Required Options

Support:

- `--source`
- `--dry-run`
- `--verify-only`
- `--batch-size`
- `--verbose`
- Optional `--allow-non-empty-target`
- Explicit confirmation for destructive or overwrite behaviour
- Machine-readable exit status
- Structured progress output
- Redacted configuration output

### Source Schema Compatibility

The tool must:

- Detect and record the SQLite source schema version.
- Support all known deployed source versions; or
- Fail with an explicit compatibility error.
- Document any prerequisite upgrade path.

### Safety

The tool must:

1. Open SQLite read-only.
2. Never mutate the source.
3. Verify source readability.
4. Verify destination PostgreSQL major version 18.
5. Verify destination migration state.
6. Detect non-empty target tables.
7. Refuse unsafe duplicate imports by default.
8. Record migration metadata.
9. Preserve source IDs.
10. Preserve nulls.
11. Preserve binary values.
12. Preserve timestamp meaning.
13. Preserve JSON meaning.
14. Roll back failed writes where transaction boundaries permit.
15. Never delete the SQLite source.
16. Never overwrite backups.
17. Produce a reconciliation report.

### Transaction and Recovery Strategy

Provide either:

- Whole-import atomicity when data volume permits; or
- Checkpointed table- or batch-level transactions with:
  - Migration-state tracking.
  - Deterministic resume.
  - Clear rollback or cleanup behaviour.
  - Duplicate prevention.
  - Safe retry semantics.

A failed import must not leave the target in an ambiguous state that cannot be cleanly resumed or rolled back.

The utility must be safe against accidental re-execution.

### Ordering

Determine insertion order from foreign-key relationships.

Use:

- Topological order.
- Deferred constraints.
- Explicitly controlled temporary constraint strategy.

Handle cycles explicitly.

Do not globally disable integrity enforcement without post-import validation.

### Batch Processing

Use bounded batches.

For each table:

- Read deterministic batches.
- Transform values using explicit conversion functions.
- Insert using parameterised statements.
- Preserve primary keys.
- Record source and destination counts.
- Report the failing table and source key.
- Avoid logging sensitive row content.

### Sequence Repair

After explicit ID import:

- Resolve each identity sequence.
- Set a safe next value.
- Handle empty tables.
- Verify the first generated insert.

### Reconciliation

Reconciliation must include:

- Row counts per table.
- Primary-key sets or representative ranges, as appropriate.
- Foreign-key integrity.
- Unique constraints.
- Non-null constraints.
- Invalid boolean detection.
- Timestamp conversion failures.
- Invalid JSON detection.
- Binary length consistency.
- Minimum and maximum numeric IDs.
- Table-specific domain invariants.
- Orphan detection.
- Canonicalised per-table hashes.
- Sampled row comparisons.
- Sequence state validation.

Canonicalisation must define:

- Column ordering.
- Null representation.
- Timestamp representation.
- JSON key ordering.
- Binary encoding.
- Boolean representation.
- Numeric representation.

A reconciliation mismatch is a migration failure, not merely a warning.

### Migration Report

Include:

- Source database path.
- Source size.
- Source schema version.
- Destination identity with credentials redacted.
- Start and completion timestamps.
- Destination schema version.
- Tables migrated.
- Rows read.
- Rows inserted.
- Rows rejected.
- Sequence resets.
- Constraint validation.
- Hash comparisons.
- Sampled comparison results.
- Warnings.
- Errors.
- Overall pass/fail status.

### Gate D: Data Migration Complete

Gate D passes only when:

- Representative SQLite data migrates successfully.
- Source database remains unchanged.
- Reconciliation passes.
- Sequence state is valid.
- Failure handling is tested.
- Resume or rollback behaviour is tested.
- Unsafe re-execution is prevented.

## Phase 5: Tests

### Real PostgreSQL Requirement

All database integration tests must use a real PostgreSQL 18 instance through:

- Testcontainers.
- Dedicated Compose service.
- CI service container.

Do not claim compatibility using only:

- SQLite tests.
- Mocked repositories.
- SQL snapshots.
- In-memory substitutes.

### Test Isolation

Use an appropriate deterministic strategy:

- Database per worker.
- Schema per worker.
- Transaction rollback.
- Truncation with identity reset.
- Ephemeral container per suite.

Parallel tests must not interfere.

### Required Coverage

Add or update tests for:

- Configuration validation.
- Database startup.
- Pool lifecycle.
- Graceful shutdown.
- Migration from zero.
- Migration repeat execution.
- Repository CRUD.
- Foreign keys.
- Unique constraints.
- Check constraints.
- Identity generation.
- `RETURNING`.
- Transactions.
- Rollback.
- Upsert semantics.
- Stable pagination.
- Deterministic ordering.
- Boolean conversion.
- Timestamp conversion.
- JSON and JSONB.
- Binary data.
- Null values.
- `bigint` DTO mapping.
- PostgreSQL error mapping.
- Concurrent state changes.
- Connection unavailability.
- Recovery after PostgreSQL becomes available.
- SQLite-to-PostgreSQL migration.
- Sequence repair.
- Existing persistence-backed application flows.

### Migration Fixture

Create a representative SQLite fixture containing:

- Related tables.
- Null values.
- Boolean values.
- Multiple timestamp formats where applicable.
- JSON.
- Unicode.
- Empty strings.
- Binary data.
- Explicit IDs.
- Foreign keys.
- Unique-boundary cases.
- Representative maximum numeric values.
- Application-specific edge cases.

Automate:

1. Start clean PostgreSQL 18.
2. Apply PostgreSQL migrations.
3. Run SQLite migration.
4. Run reconciliation.
5. Run application repository tests against migrated data.
6. Verify subsequent generated IDs.

### Gate E: Application Integration Complete

Gate E passes only when:

- Full applicable test suite passes against PostgreSQL 18.
- Persistence-backed user journeys pass.
- Migration fixture pipeline passes.
- CI configuration passes.
- Operational commands work.
- No tests were improperly weakened.

## Phase 6: Operational Updates

### Scripts and Makefile

Update all scripts, Makefiles, and package commands that assume SQLite.

Provide repository-consistent commands equivalent to:

```text
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
```

### Destructive Command Safety

Destructive commands must:

- Refuse production by default.
- Require explicit confirmation or force flag.
- Display target host, port, database, and user with secrets redacted.
- Avoid ambiguous defaults.
- Validate target configuration before execution.

### PostgreSQL Shell

Provide a command to open `psql` inside the container without requiring a host-installed PostgreSQL client.

### Backup and Restore

Use PostgreSQL-native tools such as:

- `pg_dump`
- `pg_restore`

Backups must:

- Include schema and data.
- Fail on errors.
- Avoid exposing credentials.
- Use timestamped filenames.
- Never silently overwrite files.

Document restore prerequisites and target-state expectations.

### Environment Configuration

Update:

- `.env.example`
- Development configuration.
- Test variables.
- Compose environment.
- CI variables.
- Deployment examples.
- Configuration validation.

Remove obsolete runtime settings such as:

- SQLite paths.
- SQLite journal configuration.
- Busy timeout.
- WAL toggles.
- In-memory runtime flags.

Do not remove variables until all references are removed.

### CI/CD

CI must:

1. Start PostgreSQL 18.
2. Wait for health.
3. Install dependencies using the lockfile.
4. Apply migrations from zero.
5. Verify migration state.
6. Run type checking.
7. Run linting.
8. Run unit tests.
9. Run PostgreSQL integration tests.
10. Run SQLite-to-PostgreSQL fixture migration.
11. Run reconciliation.
12. Build the application.
13. Run E2E tests where supported.
14. Fail on schema drift or verification failure.

Use isolated databases or schemas for concurrent jobs.

### Documentation

Update all documentation that references SQLite.

Create or update:

- `README.md`
- Developer setup documentation.
- Test documentation.
- Deployment documentation.
- `docs/postgresql-migration.md`

Document:

- PostgreSQL 18 requirement.
- Container startup.
- Environment configuration.
- Connection architecture.
- Migration commands.
- Development workflow.
- Test workflow.
- Shell access.
- Seeding.
- Reset.
- Backup.
- Restore.
- SQLite data migration.
- Reconciliation.
- Troubleshooting.
- Role separation.
- Pool sizing.
- TLS.
- Patch upgrades.
- Major-version upgrade implications.
- PostgreSQL 18 volume layout.

### Cutover Runbook

Document:

1. Announce maintenance window where required.
2. Stop application writes.
3. Back up SQLite.
4. Start PostgreSQL.
5. Apply PostgreSQL migrations.
6. Run dry-run import.
7. Run actual import.
8. Run reconciliation.
9. Verify sequences.
10. Start the application.
11. Execute smoke tests.
12. Monitor errors and database health.
13. Preserve SQLite backup unchanged.
14. Define rollback criteria.
15. Define rollback procedure.

Do not imply that writes made after PostgreSQL cutover can be automatically reverse-migrated unless that capability is implemented and tested.

## Phase 7: Cleanup and Final Verification

### Dependency Cleanup

After successful PostgreSQL validation:

- Remove unused SQLite runtime dependencies.
- Remove obsolete SQLite type packages.
- Remove old connection factories.
- Remove SQLite runtime migration code.
- Remove PRAGMA configuration.
- Remove runtime `.db` creation.
- Remove obsolete Docker mounts.
- Remove stale test setup.
- Remove stale scripts.
- Update the lockfile through the repository package manager.

The SQLite driver may remain only when required by the dedicated import utility.

If retained:

- Isolate it from runtime code.
- Classify it appropriately as a migration or development dependency.
- Document why it remains.
- Ensure production runtime startup does not load it.
- Ensure production builds can omit it where migration is executed separately.

### Static Verification

Search the complete repository for:

```text
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
```

Every remaining match must be one of:

- Dedicated migration utility.
- Migration test fixture.
- Archived historical migration.
- Documentation explicitly describing migration history.

Document each intentional remaining reference.

### Runtime Verification

Execute the full workflow from scratch:

1. Install dependencies.
2. Start PostgreSQL 18 from a clean volume.
3. Verify container health.
4. Apply migrations.
5. Verify migration status.
6. Seed data where supported.
7. Start the application.
8. Run persistence-backed smoke tests.
9. Run type checking.
10. Run linting.
11. Run unit tests.
12. Run PostgreSQL integration tests.
13. Run migration fixture tests.
14. Run reconciliation.
15. Build production artifacts.
16. Start production build where practical.
17. Exercise primary persistence-backed user journeys.
18. Stop and recreate PostgreSQL container.
19. Verify volume persistence.
20. Reset the development database.
21. Reapply migrations from zero.
22. Confirm no undocumented manual steps are required.

### Operational Verification

Where practical, verify:

- `db-shell`.
- Backup creation.
- Restore into a clean destination.
- Graceful application shutdown.
- PostgreSQL unavailability behaviour.
- Recovery after PostgreSQL availability.
- Readiness versus liveness behaviour.

### Gate F: Cleanup and Release Readiness

Gate F passes only when:

- SQLite runtime dependencies are removed.
- Static reference audit passes.
- Clean PostgreSQL bootstrap passes.
- Container restart and persistence pass.
- Full test suite passes.
- Documentation matches executable commands.
- Cutover and rollback runbooks are complete.
- Final integrated repository verification passes.

# Final Completion Authority

Only the final integration and verification stage may declare the programme:

- `COMPLETE`
- `COMPLETE WITH DOCUMENTED EXTERNAL BLOCKERS`
- `INCOMPLETE`

Individual agents may declare only their assigned tasks complete.

The programme must not be declared complete from:

- Agent summaries.
- Task-local success alone.
- Static analysis alone.
- Speculative reasoning.
- Tests that did not use PostgreSQL where required.

# Required Deliverables

The completed repository must contain:

1. PostgreSQL 18 container and persistent volume configuration.
2. Canonical connection module with lifecycle and health handling.
3. Configuration validation using `DATABASE_URL`.
4. Deterministic PostgreSQL schema migrations.
5. Fully converted database access layer.
6. Complete async propagation through the server stack.
7. PostgreSQL transaction and error-mapping infrastructure.
8. Production-quality SQLite-to-PostgreSQL import utility.
9. Reconciliation and migration reporting.
10. PostgreSQL seed and reset processes.
11. Real PostgreSQL integration-test infrastructure.
12. Migration fixture and automated migration tests.
13. Updated CI/CD.
14. Updated scripts and Makefile targets.
15. Backup and restore commands.
16. Updated environment examples.
17. Updated README and developer documentation.
18. Migration audit.
19. Architecture decision document.
20. Risk register.
21. Dependency DAG and task plan.
22. Authoritative orchestration state.
23. Cutover and rollback runbook.
24. Task-level evidence artifacts.
25. Final verification report.
26. No unexplained SQLite runtime remnants.

# Final Report Format

Produce a concise final report using these sections.

## Migration Status

State exactly one:

- `COMPLETE`
- `COMPLETE WITH DOCUMENTED EXTERNAL BLOCKERS`
- `INCOMPLETE`

## Git Baseline and Change Safety

Include:

- Initial revision.
- Final revision where applicable.
- Initial branch.
- Pre-existing staged changes.
- Pre-existing unstaged changes.
- Pre-existing untracked files.
- Migration-induced changes.
- Confirmation that unrelated user changes were preserved.

## Architecture Selected

Include:

- Existing data-access technology.
- Selected PostgreSQL technology.
- Reason it was retained or replaced.
- Connection-pool design.
- Migration framework.
- PostgreSQL image tag or digest.
- Schema and role model.
- Test isolation strategy.
- Data-import transaction and recovery strategy.

## Orchestration Summary

Include:

- Number of tasks generated.
- Tasks completed.
- Tasks failed.
- Tasks blocked.
- Tasks invalidated.
- Parallel groups executed.
- Milestone gate results.
- Replanning events.

## Files Changed

Group by:

- Application source.
- Database infrastructure.
- Schema and migrations.
- Data migration.
- Tests.
- Containers.
- CI/CD.
- Scripts.
- Documentation.
- Removed files.

## Schema Decisions

Include:

- IDs.
- Timestamps.
- Booleans.
- JSON.
- Binary data.
- Numeric precision.
- `bigint` API mapping.
- Case sensitivity.
- Full-text search.
- Constraints.
- Indexes.
- Trigger and view handling.

## SQLite Data Migration

Include:

- Command.
- Supported source schema versions.
- Safety behaviour.
- Transaction or checkpoint model.
- Resume or rollback behaviour.
- Reconciliation method.
- Sequence handling.
- Source-data issues.
- Final verification result.

## Validation Evidence

For each command run, include:

```text
Command:
Exit status:
Relevant output:
Failure attribution:
Evidence artifact:
```

Store extensive output in repository log or report artifacts rather than flooding the final response.

## Remaining SQLite References

List every intentional remaining reference and why it remains.

## Blockers and Risks

For every blocker, state:

- Category.
- Whether it is migration-introduced, pre-existing, external, or unresolved.
- Evidence.
- Affected tasks.
- What validation continued.
- Required remediation.

## Final Acceptance Result

State whether each milestone gate passed.

The migration is complete only when all required acceptance criteria and milestone gates have been validated through actual command execution.
