# MA-Harness

`memory_harness` is a **Stage-A Python package for deterministic memory- and
plan-reuse contracts**. It gives an agent harness a durable, auditable memory
of tasks, plans, decisions, and outcomes, plus a bounded "prepare once, dispatch
at most once" flow that reuses prior work without ever guessing.

Its guiding principle is **determinism and fail-closed safety**: the same inputs
always yield the same decision, every side effect is recorded before it happens,
and any capability that cannot be authorized is refused rather than approximated.

## Features

- **Canonical records.** Task, plan, decision, dispatch, operation, and outcome
  objects with stable content hashes, so identical work is recognized and reused
  instead of recomputed.
- **Integrity-bound execution envelope.** A parent envelope binds one integrity
  over the task, objective, repository base, accepted plan, and rendered
  context — tampering or drift is detected, not silently accepted.
- **Durable SQLite storage.** Decisions, operations, and outcomes persist in a
  local store that is the single source of truth for provenance and approval
  evidence.
- **Isolated snapshot &amp; restore.** `snapshot.SnapshotService` captures a whole
  memory store into a verifiable artifact and restores it into a fresh path,
  checking every integrity and compatibility fact before installing — it never
  merges, overwrites, or replays work.
- **Bounded preparation, one safe dispatch.** `preparation.PreparationService`
  resolves the exact state and runs a single fault-isolated search before any
  optional call; `runtime.MemoryRuntime` persists launch intent before dispatch
  and records the observed invocation after, guaranteeing at-most-once execution
  with explicit reconciliation of ambiguous effects.
- **Plan reuse with safe fallback.** Versioned local templates support direct
  fill or a bounded adaptation draft, with a clean fresh-plan fallback when no
  prior work fits.
- **Privacy guards.** Control credentials and known synthetic secrets are kept
  out of query payloads and persisted records by design.
- **Gated optional integrations.** A reviewed-experience adapter (EverOS) and a
  trusted-procedure vector-search adapter (MongoDB Atlas) are available but fully
  optional — absent installs skip honestly rather than degrade.

It deliberately does **not** provide a second launcher, scheduler, review
system, evidence ledger, secret manager, learned selector, or benchmark runner.

## Repository layout

| Path | Contents |
|------|----------|
| `src/memory_harness/` | The core package (the deliverable). |
| `tests/` | `unittest` suites — `tests/local/` (dependency-light) and `tests/live/` (network, env-gated). |
| `harness/` | Bundled orchestrator harness, tooling, docs, and vendored deps (`everos`, `langchain-mongodb`). |
| `.agent/ .agents/ .claude/ .codex/ .qwen/` | Portable agent/editor config (see `workspace-aid/README.md`). |
| `.secrets/` | Local credentials — **gitignored, never committed**. |

## Requirements

- **Python 3.11+** for the core package (`pyproject.toml`).
- **Python 3.12+** additionally for the optional `everos` extra.

## Install

```bash
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate

pip install -e .                   # core package
```

The core package is dependency-light. Optional integrations ship as extras
(`.[atlas]`, `.[everos]`); see `src/memory_harness/README.md` for their setup.

## Running the tests

Tests use the standard-library `unittest` runner.

```bash
# Dependency-light local suite (optional integrations skip themselves cleanly)
python -m unittest discover -s tests/local -p 'test_*.py'

# Everything discoverable
python -m unittest discover -s tests -p 'test_*.py'
```

Live suites under `tests/live/` are **opt-in** and skip unless their environment
gate is set (e.g. `MEMORY_HARNESS_RUN_LIVE_ATLAS=1`).

## Security

Credentials live only in the gitignored `.secrets/` directory and are never
committed. Build artifacts (`*.egg-info/`, `.venv/`, `__pycache__/`) are ignored
and regenerate on demand.
