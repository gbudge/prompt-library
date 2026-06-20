You are an AI Orchestration Agent. Your role is to coordinate the construction of a production-grade logging subsystem for this application.

Your task is to create a reusable, centrally managed logging module that can be used consistently throughout the entire codebase.

Objectives

Design and implement a logging framework that:

* Supports standard logging levels:
    * TRACE
    * DEBUG
    * INFO
    * WARN
    * ERROR
* Supports multiple output destinations:
    * Console (stdout/stderr)
    * Log file
    * Both simultaneously
* Can be configured through:
    * Command-line startup parameters
    * Environment variables
    * .env files
    * Application configuration files (if present)
* Allows runtime configuration during application startup without requiring code changes.
* Provides a single shared logger interface used by all application components.
* Is designed for production use.

⸻

Functional Requirements

1. Central Logging Module

Create a dedicated logging package/module that becomes the single source of truth for logging.

Requirements:

* No component should instantiate its own logger.
* All application components must obtain loggers from the central logging package.
* Logging configuration must be initialized once during application startup.
* Logging behaviour must be consistent across the entire application.

Example:

```
Application
 ├── Configuration
 ├── Logging
 ├── API
 ├── Database
 ├── MCP Server
 ├── Indexer
 ├── Scheduler
 └── Workers
```

All modules use the same logging subsystem.

⸻

2. Logging Levels

Support:

```
TRACE
DEBUG
INFO
WARN
ERROR
```

Level hierarchy:

```
TRACE > DEBUG > INFO > WARN > ERROR
```

Examples:

```
TRACE = highly verbose diagnostics
DEBUG = developer troubleshooting
INFO  = normal operational events
WARN  = unexpected but recoverable conditions
ERROR = failures requiring attention
```

Filtering must occur centrally.

Examples:

```
LOG_LEVEL=INFO
```

Produces:

```
INFO
WARN
ERROR
```

but suppresses:

```
TRACE
DEBUG
```

⸻

3. Configuration Sources

Configuration must be loaded in the following precedence order:

```
1. Command-line arguments
2. Environment variables
3. .env file
4. Application defaults
```

Later sources are overridden by earlier sources.

Example:

```bash
app --log-level debug
```

must override:

```env
LOG_LEVEL=info
```

⸻

4. Environment Variable Support

Support at minimum:

```env
LOG_LEVEL=info
LOG_OUTPUT=console
LOG_FILE=logs/application.log
LOG_FORMAT=text
LOG_CONSOLE=true
LOG_FILE_ENABLED=false
```

Additional variables may be added if beneficial.

⸻

5. Command-Line Parameters

Provide startup arguments.

Minimum requirements:

```bash
--log-level
--log-output
--log-file
--log-format
```

Examples:

```bash
app --log-level debug
app --log-level trace --log-output console
app --log-output file --log-file ./logs/app.log
app --log-output both
```

⸻

6. Output Destinations

Support:

**Console**

```
stdout/stderr
```

**File**

Write to:

```
configurable path
```

Example:

```env
LOG_FILE=logs/app.log
```

Requirements:

* Automatically create parent directories.
* Handle missing directories gracefully.
* Fail with clear errors if file cannot be opened.

**Dual Output**

Support:

```
console + file simultaneously.
```

Example:

```env
LOG_OUTPUT=both
```

⸻

7. Log Formatting

Support at least:

**Text Format**

Example:

```
2026-06-20T10:15:23Z INFO api.server Request completed
```

**Structured JSON Format**

Example:

```json
{
  "timestamp": "2026-06-20T10:15:23Z",
  "level": "INFO",
  "component": "api.server",
  "message": "Request completed"
}
```

Configuration:

```env
LOG_FORMAT=text
```

or

```env
LOG_FORMAT=json
```

⸻

8. Component-Aware Logging

Support named loggers.

Examples:

```
api.server
database
scheduler
mcp
indexer
github
worker
```

Usage:

```
[api.server]
[database]
[indexer]
```

This enables tracing issues to specific components.

⸻

9. Contextual Logging

Support structured fields.

Examples:

```
repository=my-repo
branch=main
commit=abc123
request_id=xyz789
```

Example JSON:

```json
{
  "repository": "my-repo",
  "branch": "main",
  "request_id": "xyz789"
}
```

Fields should be attachable without string concatenation.

⸻

10. Error Logging

Support:

```
message
error object
stack trace (where available)
```

Example:

```
ERROR failed to open database
error="permission denied"
```

⸻

11. Startup Logging

At application startup emit configuration information.

Example:

```
Logging initialized
Level=DEBUG
Output=BOTH
Format=JSON
File=logs/app.log
```

This should occur once after configuration is finalized.

⸻

12. Thread Safety

The logging subsystem must be safe for:

* concurrent requests
* worker pools
* background jobs
* asynchronous execution

No message corruption or interleaving should occur.

⸻

13. Performance Requirements

Requirements:

* Minimal allocation overhead.
* Avoid expensive formatting when log level is disabled.
* Efficient file writing.
* Suitable for long-running services.

⸻

14. Developer Experience

Provide concise usage examples.

Example:

```go
logger := logging.GetLogger("database")
logger.Debug("opening connection")
logger.Info(
    "repository indexed",
    "repository", repo,
    "branch", branch,
)
```

Error example:

```go
logger.Error(
    "database query failed",
    "error", err,
)
```

⸻

Non-Functional Requirements

The implementation must be:

* Production-ready
* Fully documented
* Modular
* Testable
* Reusable
* Maintainable
* Consistent across the application
* Easy to extend with future logging sinks

Future sinks may include:

* Syslog
* CloudWatch
* OpenTelemetry
* Loki
* Elasticsearch
* Splunk

Design the architecture so these can be added without changing application code.

⸻

Deliverables

Produce:

1. Complete logging architecture design.
2. Configuration model.
3. Environment variable specification.
4. CLI parameter specification.
5. Module/package structure.
6. Public API design.
7. Initialization flow.
8. Logger lifecycle design.
9. File logging implementation.
10. Console logging implementation.
11. Structured JSON logging implementation.
12. Error handling strategy.
13. Concurrency considerations.
14. Unit test strategy.
15. Integration test strategy.
16. Complete production-ready source code integrated into the existing application.
17. Required updates to startup/bootstrap code.
18. Required updates to configuration loading.
19. Required updates to application documentation.
20. Example configurations and usage examples.

Do not provide placeholders, pseudo-code, stubs, TODOs, or partial implementations. Produce complete, runnable, production-grade code and all required integration changes.