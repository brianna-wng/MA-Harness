# MA-Harness

A complete orchestration and memory system for teams of coding agents.

The **Orchestrator Harness** runs each campaign as a fresh epoch: ROOT splits the
work into parallel lanes, and every lane gets its own Git branch, isolated
worktree, task card, and provider session so workers never interfere. A
super-cache stages the shared tools, hooks, skills, and provider config; a
persistent monitor tracks processes, results, leases, and lane health; and
workers report progress or request help through queue notifications that ROOT
answers without restarting sessions. The harness owns the full result
lifecycle — evidence tied to an exact lane and run, a reviewer verdict
(pass/fail/blocked) separate from ROOT's accept/reject, plus resume, correction,
retirement, and identity-exact cleanup.

On top of this runs a **memory feedback loop**. Each reviewed task becomes a
trajectory; EverOS turns reviewed experience into historical cases and reusable
skills; and a trust system verifies their origin and promotes approved skills
into **trusted procedures** — immutable versioned records with permissions,
applicability rules, current-version status, and revocation. Trusted procedures
publish to **MongoDB Atlas** and are found via real Vector Search. A new task
searches EverOS for local experience and Atlas for shared procedures, validates
every result, builds a safe task-specific plan and worker context, launches a
real coding worker, and feeds the reviewed outcome back into memory. Every
memory feature is optional: with them all disabled, the harness still works
normally.

On **SWE-Marathon v1.1 (ZSTD)** it scored 47.3% above the official *high*
average and 7.3% above the official *xhigh* average, while using 91% fewer
uncached tokens than xhigh (11.14× lower), though runs took longer.

## `memory_harness` package

This repo's core deliverable is `src/memory_harness/` — the Stage-A Python
package implementing the deterministic memory and plan-reuse contracts. Its
guiding principle is **determinism and fail-closed safety**: the same inputs
yield the same decision, side effects are recorded before they happen, and any
capability that cannot be authorized is refused rather than approximated.

- **Canonical records** — task, plan, decision, dispatch, operation, and outcome
  objects with stable content hashes, so identical work is reused, not recomputed.
- **Bounded prepare, one safe dispatch** — a single fault-isolated search before
  any optional call, then at-most-once execution with explicit reconciliation.
- **Durable SQLite store** — the single source of truth for provenance and
  approval evidence, with verifiable snapshot/restore.
- **Privacy guards** — credentials and known secrets are kept out of query
  payloads and persisted records.
- **Gated integrations** — EverOS (reviewed experience) and MongoDB Atlas
  (trusted-procedure vector search) are optional and skip cleanly when absent.
