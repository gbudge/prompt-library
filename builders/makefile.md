Makefile Engineering Standard

You are responsible for creating, reviewing, refactoring, or updating Makefiles for any software project regardless of programming language, framework, operating system, database, deployment model, or runtime.

The resulting Makefile must provide a consistent, production‑grade developer experience across all projects.

Core Objectives

The Makefile must:

· Be language agnostic.
· Be operating system agnostic wherever practical.
· Prefer cross‑platform tools and commands.
· Be deterministic and repeatable.
· Be idempotent wherever possible.
· Be CI/CD friendly.
· Be suitable for both local development and automation pipelines.
· Fail fast on errors.
· Return proper exit codes.
· Avoid duplicated logic.
· Minimise onboarding effort.
· Be self‑documenting.
· Be maintainable and extensible.
· Use dependency chaining instead of duplicated commands.
· Use variables instead of hardcoded values.
· Prefer explicitness over hidden behaviour.

Shell & Global Settings

Canonical shell detection (adapt if project already specifies a shell):

```makefile
HAS_BASH := $(shell command -v bash 2>/dev/null)
ifdef HAS_BASH
  SHELL := /bin/bash
  .SHELLFLAGS := -euo pipefail -c
else
  SHELL := /bin/sh
  .SHELLFLAGS := -ec
endif
```

If bash is unavailable (e.g., Alpine, minimal containers), the fallback to POSIX /bin/sh ensures portability. Recipes must avoid bashisms when using the fallback.

Single .PHONY declaration at the top listing every phony target:

```makefile
.PHONY: help setup clean reset upgrade build all unit-test coverage system-test format lint typecheck scan quality start stop dev db-init db-reset …
```

Terminal Colours (Auto‑Detection)

Embed these colour variables at the top of the Makefile. They degrade gracefully to plain text when output is not a terminal.

```makefile
ifneq ($(shell [ -t 1 ] && echo true),true)
  RESET :=
  BOLD  :=
  DIM   :=
  RED   :=
  GREEN :=
  YELLOW:=
  BLUE  :=
  CYAN  :=
  WHITE :=
else
  RESET := \033[0m
  BOLD  := \033[1m
  DIM   := \033[2m
  RED   := \033[31m
  GREEN := \033[32m
  YELLOW:= \033[33m
  BLUE  := \033[34m
  CYAN  := \033[36m
  WHITE := \033[37m
endif
```

All recipes that emit coloured output MUST use these variables. Never hardcode ANSI codes.

Default Target

```makefile
.DEFAULT_GOAL := help
```

The first user‑visible experience must always be a discoverable command list.

Logging Standard

All user‑facing recipes must emit structured log messages using the defined colour variables.

Supported levels:

· [INFO] – use $(GREEN)
· [WARN] – use $(YELLOW)
· [ERROR] – use $(RED)

Requirements:

· Log what is starting, major steps, completion, and failures clearly.
· Surface command failures immediately using the pattern:
  some_command || { echo "$(RED)[ERROR]$(RESET) ..."; exit 1; }
· Example: echo "$(GREEN)[INFO]$(RESET) Installing dependencies..."

Environment Configuration

```makefile
ENV_FILE ?= .env
-include $(ENV_FILE)
export
```

Variables

Use variables for all project‑specific values: tool paths, versions, ports, container/image names, directories, output paths, database locations, environment settings. Avoid hardcoded values.

Internal Targets

Targets not intended for direct user invocation must be prefixed with _ and excluded from help output. Examples: _prepare, _validate, _check_dependencies, _confirm_reset.

Dependency Chaining

Prefer all: build lint test over duplicating commands. Targets should compose other targets wherever possible.

Self‑Documenting Help System

Hardcoded help output is prohibited. The help target MUST use the following canonical snippet (adjust indentation to match the project's Makefile):

```makefile
help: ## @Meta @Show this help message
  @printf "%s\n" ""
  @printf "$(CYAN)%-20s$(RESET) %s\n" "TARGET" "DESCRIPTION"
  @printf "%s\n" "──────────────────────────────────────────────────────"
  @grep -E '^[a-zA-Z_-]+:.*?## @' $(MAKEFILE_LIST) \
    | grep -v '^_' \
    | sed 's/:.*## @/|/' \
    | awk -F'|' '{groups[$$2]=groups[$$2] sprintf("  $(YELLOW)%-18s$(RESET) %s\n", $$1, $$3)} END {n=asorti(groups,s); for(i=1;i<=n;i++) {printf "$(CYAN)%s$(RESET)\n", s[i]; printf "%s", groups[s[i]]}}' \
    || grep -E '^[a-zA-Z_-]+:.*?## @' $(MAKEFILE_LIST) \
    | grep -v '^_' \
    | sed 's/:.*## @/|/' \
    | sort -t'|' -k2,2 -k1,1 \
    | awk -F'|' 'prev!=$$2{printf "\n$(CYAN)%s$(RESET)\n",$$2}{printf "  $(YELLOW)%-18s$(RESET) %s\n",$$1,$$3;prev=$$2}'
  @printf "%s\n" ""
```

Documentation format for every public target: target-name: ## @Group @Description

Note: @Meta is the group for the help target itself. This snippet handles both GNU and BSD/macOS awk gracefully.

Destructive Operation Guards

Targets that destroy data/state MUST include a confirmation prompt bypassable via FORCE=1. Use the following canonical guard, which uses the colour variables and is safe in non‑interactive environments:

```makefile
_confirm_reset:
  @if [ -z "$${FORCE:-}" ]; then \
    if [ -t 0 ]; then \
      printf "$(YELLOW)[WARN]$(RESET) This will destroy local state. Continue? [y/N] "; \
      read -r confirm && [ "$$confirm" = "y" ] || { echo "$(GREEN)[INFO]$(RESET) Aborted."; exit 1; }; \
    else \
      echo "$(RED)[ERROR]$(RESET) Destructive operation requires FORCE=1 in non-interactive environments."; \
      exit 1; \
    fi; \
  fi
```

Required Targets

Only include targets relevant to the project. No DB targets without a database; no container targets without containers.

Environment Management

· setup – ## @Environment @Install project dependencies (idempotent)
· clean – ## @Environment @Remove generated files
· reset – ## @Environment @Reset local environment
  Dependency chain: reset: _confirm_reset clean setup
· upgrade – ## @Environment @Upgrade dependencies (no major bumps unless explicitly configured)

Build

· build – ## @Build @Build production artifacts
· all – ## @Build @Run full CI pipeline (build + quality gates + unit tests + coverage)
  Executes the full verification pipeline in the correct dependency order, failing immediately on errors.

Testing

· unit-test – ## @Testing @Run unit tests
· coverage – ## @Testing @Generate coverage reports
· system-test – ## @Testing @Run system/integration tests

Code Quality

· format – ## @Quality @Format source code
· lint – ## @Quality @Run linting
· typecheck – ## @Quality @Run static type analysis
· quality – ## @Quality @Run advanced quality analysis

Security

· scan – ## @Security @Run security scanning (SAST, deps, secrets, containers; fail on critical)

Version Management

Follows Semantic Versioning. Discovery priority:

1. VERSION file
2. package.json
3. pyproject.toml
4. Cargo.toml
5. pom.xml
6. .csproj
7. Fallback: create a VERSION file.

Expose the current version via the variable CURRENT_VERSION.

· version – ## @Versioning @Display current version
· patch – ## @Versioning @Increment patch version
· minor – ## @Versioning @Increment minor version
· major – ## @Versioning @Increment major version

Increment targets must update the discovered storage and/or create git tags accordingly.

Release Management

release – ## @Release @Create GitHub release

Workflow:

1. Verify clean git working tree.
2. Run full pipeline (make all).
3. Verify version tag exists; create and push if missing.
4. Create GitHub release with gh release create.
5. Upload release artifacts where applicable.

Must fail if any prerequisite is not met.

Container Management (when applicable)

Uses Docker Compose exclusively.

· build-container – ## @Container @Build container images
· start-container – ## @Container @Start containers
· stop-container – ## @Container @Stop containers
· restart-container – ## @Container @Restart containers

Production Runtime (when applicable)

· start-api / stop-api / restart-api – ## @Runtime @Manage production API
· start-ui / stop-ui / restart-ui – ## @Runtime @Manage production UI
· start – ## @Runtime @Start production services
  Orchestrate start-api and start-ui with readiness checks, correct startup ordering, consolidated logging, proper signal handling, and correct exit code propagation. Use process‑parallel tools native to the project’s stack (e.g., concurrently for Node, Makefile jobserver with &/wait for POSIX shells, or ecosystem‑specific orchestrators) with dependency readiness checks.

Development Runtime (when applicable)

· dev-api – ## @Development @Start API in dev mode (auto‑reload, watch, debug logging)
· dev-ui – ## @Development @Start UI in dev mode (hot reload, watch mode)
· dev – ## @Development @Start full dev environment
  Orchestrate dev-api and dev-ui with the same requirements as start.

Database Targets (when applicable)

· db-init – ## @Database @Initialize schema (idempotent)
· db-reset – ## @Database @Reset to base schema (destructive guard)
· db-backup – ## @Database @Create timestamped backup
· db-restore – ## @Database @Restore from backup
· list-backups – ## @Database @List available backups

Implementation Discovery

When generating a Makefile, you must:

· Detect the project’s language, tooling, and ecosystem.
· Produce working commands for every target (never stubs).
· Use idiomatic tools for the detected stack (e.g., ruff for Python, cargo fmt for Rust, dotnet format for .NET, golangci-lint for Go).
· Infer version storage using the discovery priority above.
· Embed the canonical help, colour, and guard snippets verbatim (adjusted only for variable naming consistency).
· Ensure all recipes log with the structured [INFO]/[WARN]/[ERROR] format using the colour variables.

Review Checklist

When reviewing or refactoring an existing Makefile:

1. Required targets present where applicable
2. No duplicated logic
3. Portable shell usage (bash with fallback)
4. Structured logging on all user‑facing targets
5. Proper error handling and exit codes
6. Canonical comment‑driven help system present
7. All public targets documented; internal targets prefixed _
8. Dependency chaining used
9. CI/CD compatible (non‑TTY safe guards, colour fallback)
10. Variables replace hardcoded values
11. Target naming matches this standard
12. Destructive guards with FORCE bypass
13. ENV_FILE loading present
14. Single .PHONY declaration
15. Backward compatibility preserved where practical
16. Colour auto‑detection and colour variables used everywhere

The final Makefile must be production‑grade, self‑documenting, deterministic, maintainable, portable, CI/CD‑friendly, and provide a consistent developer experience across all projects.
