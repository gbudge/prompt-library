You are a senior Go systems engineer. Build a production-grade, Git-aware Repository Intelligence Server exposed via Model Context Protocol (MCP) using JSON-RPC 2.0 over Streamable HTTP.

This is a local-first, multi-repository, multi-branch, deterministic code intelligence engine. SQLite is the only source of truth. All intelligence comes from a fully indexed graph. No filesystem scans during queries.

0. NON-NEGOTIABLE GUARANTEES
- Deterministic outputs: identical inputs produce byte-identical JSON outputs
- Incremental updates only: no full rebuild unless full_reindex=true
- Complete isolation: data is scoped to repo_id and branch_id, no cross-scope reads
- SQLite holds ALL mutable state
- MCP query tools MUST NOT access filesystem or git; they read SQLite only
- Core logic MUST NOT use heuristics, embeddings, or LLM inference
- MCP layer is stateless: no session memory affects correctness

1. IDENTITY MODEL
All data is scoped: repo_id → branch_id → entity.

Normalization:
- repo path: filepath.Abs, then filepath.Clean, convert to forward slashes, lowercase on Windows
- remote URL: trim trailing .git, lowercase scheme and host

IDs (lowercase hex SHA256):
- repo_id = SHA256(normalized path OR normalized remote URL)
- branch_id = SHA256(repo_id + ":" + branch_name)
- file_id = SHA256(repo_id + ":" + branch_id + ":" + file_path)
- symbol_id = SHA256(file_id + ":" + symbol_name + ":" + signature)

Branch identity is immutable. A rename creates a new branch_id. Old branch_id is preserved, never migrated or deleted automatically.

2. COMPONENTS (single Go module)
- cmd/server/main.go
- internal/mcp: JSON-RPC handler, tool dispatch
- internal/git: branch listing, diff, working tree reads
- internal/repo: registration and lifecycle
- internal/parser/go: Go AST parser
- internal/parser/interface.go: parser interface
- internal/indexer: incremental state machine
- internal/store/sqlite: schema, migrations, queries
- internal/query: pure SQL for tools
- internal/graph: types and validation
- internal/cache: read-through cache, optional v1

3. SQLITE CONSISTENCY
Required PRAGMAs on open:
PRAGMA journal_mode=WAL;
PRAGMA synchronous=NORMAL;
PRAGMA foreign_keys=ON;
PRAGMA busy_timeout=5000;

Writes: BEGIN IMMEDIATE; COMMIT or ROLLBACK. One writer at a time.
Reads: read-only transactions, see only committed data.
MCP queries during indexing see the last committed state. Per-file updates are atomic.

4. MCP TRANSPORT (MCP 2024-11-05)
Endpoint: POST /mcp only. Accept: application/json. Content-Type: application/json. No SSE. No batching.

Request:
{"jsonrpc":"2.0","id":"string|number","method":"string","params":{...}}

Implement: initialize, initialized (notification), tools/list, tools/call, shutdown, exit (notification).

tools/list returns capabilities: {"tools":{}}.

tools/call params:
{"name":"tool_name","arguments":{"repo_id":"...","branch_id":"...", ...}}

Response envelope:
{"jsonrpc":"2.0","id":1,"result":{"content":[{"type":"text","text":"<JSON string>"}]}}

The text field MUST parse to either:
Success: {"ok":true,"repo_id":"...","branch_id":"...","data":{}}
Error: {"ok":false,"error":{"code":"...","message":"..."}}

Determinism rules:
- Marshal with sorted map keys
- Sort all arrays before marshal:
    * files by path
    * symbols by name then symbol_id
    * edges by from_id, to_id, edge_type
- Use case-sensitive Go string compare
- No timestamps, random IDs, or map iteration order in output
- Measure response size on UTF-8 bytes of text field. If >2MB, return error code RESPONSE_TOO_LARGE

5. ERROR CODES (only these)
REPO_NOT_FOUND, BRANCH_NOT_FOUND, FILE_NOT_FOUND, SYMBOL_NOT_FOUND, INVALID_ARGS, RESPONSE_TOO_LARGE, INDEXING_IN_PROGRESS, INDEX_DISABLED, INTERNAL_ERROR

6. GRAPH MODEL
Nodes: File, Symbol, Module
Edges (directed): IMPORTS, DEFINES, CALLS, REFERENCES, CONTAINS, DEPENDS_ON
Uniqueness: UNIQUE(repo_id, branch_id, from_id, to_id, edge_type)

7. SQLITE SCHEMA
CREATE TABLE repos (repo_id TEXT PRIMARY KEY, path TEXT NOT NULL, remote_url TEXT, registered_at INTEGER NOT NULL, enabled INTEGER NOT NULL DEFAULT 1);
CREATE TABLE branches (branch_id TEXT PRIMARY KEY, repo_id TEXT NOT NULL, branch_name TEXT NOT NULL, head_commit TEXT, last_scan_time INTEGER, FOREIGN KEY(repo_id) REFERENCES repos(repo_id));
CREATE TABLE files (id TEXT PRIMARY KEY, repo_id TEXT NOT NULL, branch_id TEXT NOT NULL, path TEXT NOT NULL, hash TEXT NOT NULL, language TEXT, last_modified INTEGER);
CREATE INDEX idx_files_scope ON files(repo_id, branch_id, path);
CREATE INDEX idx_files_hash ON files(hash);
CREATE TABLE symbols (id TEXT PRIMARY KEY, repo_id TEXT NOT NULL, branch_id TEXT NOT NULL, file_id TEXT NOT NULL, name TEXT NOT NULL, type TEXT NOT NULL, signature TEXT, start_line INTEGER, end_line INTEGER);
CREATE INDEX idx_symbols_scope_name ON symbols(repo_id, branch_id, name);
CREATE INDEX idx_symbols_file ON symbols(file_id);
CREATE TABLE edges (id INTEGER PRIMARY KEY AUTOINCREMENT, repo_id TEXT NOT NULL, branch_id TEXT NOT NULL, from_id TEXT NOT NULL, to_id TEXT NOT NULL, edge_type TEXT NOT NULL, UNIQUE(repo_id, branch_id, from_id, to_id, edge_type));
CREATE INDEX idx_edges_from ON edges(repo_id, branch_id, from_id);
CREATE INDEX idx_edges_to ON edges(repo_id, branch_id, to_id);

8. LIFECYCLE
enabled flag: 1 = active, 0 = disabled. When disabled, all query tools return INDEX_DISABLED. Indexer skips it.

index_repo(path, branch?, full_reindex?, language?):
1. Normalize path, compute repo_id. Insert if new.
2. If exists and enabled=0, set enabled=1.
3. If full_reindex=true, DELETE FROM files, symbols, edges WHERE repo_id=?; DELETE FROM branches WHERE repo_id=?;
4. For target branches, walk files, hash content, compare to stored hash.
5. For NEW or UPDATED Go files: parse, in one transaction delete old symbols and edges for file_id, insert new, update files table.
6. Update branches.head_commit and last_scan_time.

list_indexes(): return repos sorted by path with repo_id, path, enabled, branch_count, last_indexed, head_commit.

disable_index(repo_id): set enabled=0
enable_index(repo_id): set enabled=1
unregister_repo(repo_id): delete all rows for repo_id
rescan_repo(repo_id, branch_id?): same as index_repo with full_reindex=true

OpenCode mapping: /index = index_repo, /index fresh = full_reindex=true, /index full = full_reindex=true, /list = list_indexes, /disable = disable_index, /enable = enable_index, /rm = unregister_repo

9. GIT TRACKING
Incremental: git diff --name-status <stored_head> HEAD. Read changed files from working tree. Hash and compare.
Branch rename: new branch_id on next index. Old data stays.
Isolation: no shared state between branches.

10. GO PARSER (go/parser, go/ast only)
Extract: packages, imports, functions, methods with receiver, structs, interfaces, static direct calls.
DO NOT: resolve interfaces, dynamic dispatch, reflection, function pointers, embedded promotion, cross-package type inference.
Method calls resolved by lexical receiver type name only.

11. INDEXER STATE MACHINE
States: NEW, UNCHANGED, UPDATED, DELETED
Per file:
- NEW or UPDATED: BEGIN IMMEDIATE, delete old, insert new, update file, COMMIT. On any error: ROLLBACK, keep prior state, log, continue.
- DELETED: BEGIN IMMEDIATE, delete symbols, delete edges where from_id=file_id or to_id=file_id, delete file, COMMIT.
- UNCHANGED: skip
Never span a transaction across files. Worker pool 8 to 16, single SQLite writer.

12. TOOL CONTRACTS
All tools except lifecycle are read-only SQLite queries.

Indexing: index_repo, list_indexes, disable_index, enable_index, unregister_repo, rescan_repo
Repo info: get_repo_summary, get_repo_state
Files: get_file, get_file_summary, list_files
Symbols: find_symbol, get_symbol, list_symbols
Graph: get_callers, get_callees, get_dependencies, get_references
Impact: what_breaks_if, find_impact_of_file_change, find_unused_symbols
Architecture: get_service_map (main packages only), get_dependency_graph

All list outputs sorted as in section 4.

13. PERFORMANCE
- Index ≤2s per 1000 LOC Go file
- Symbol lookup <10ms
- No full scans after initial ingest
- Graph queries use indexes only

14. VALIDATION CHECKLIST
- Multiple repos and branches indexed independently
- Edit branch A does not affect branch B
- Queries return correct graphs with deterministic ordering
- No filesystem access in query tools
- Incremental update works without full_reindex
- full_reindex wipes and rebuilds
- disable_index returns INDEX_DISABLED, enable_index restores
- unregister_repo deletes DB rows only
- Branch rename creates new branch_id, preserves old
- Failed file transaction leaves zero partial state
- Response >2MB returns RESPONSE_TOO_LARGE
- MCP initialize advertises tools correctly
- WAL mode active, queries never see uncommitted writes