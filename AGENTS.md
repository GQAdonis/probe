# Project Instructions

Probe is a native, local-first API client built with Rust, GPUI, Longbridge
gpui-base, and OpenCollection YAML. The CLI and desktop are equal adapters over
the same application and domain implementation.

## Read Only What the Task Needs

After this file, inspect the affected code and read only the matching reference:

| Work | Reference |
| --- | --- |
| Domain, repositories, persistence, HTTP, imports | [Architecture](docs/ARCHITECTURE.md) |
| CLI commands, JSON, selectors, exit codes | [CLI](docs/CLI.md) |
| Desktop interaction, themes, accessibility | [Design](docs/DESIGN.md) |
| Rust workflow, dependencies, GPUI, tests | [Development](docs/DEVELOPMENT.md) |
| Errors or logging | [Errors and logging](docs/ERRORS_AND_LOGGING.md) |
| Benchmarks or optimization | [Performance](docs/PERFORMANCE.md) |
| Future scope | [Roadmap](IMPLEMENTATION_PLAN.md) |

Do not read every document by default. The [documentation index](docs/README.md)
identifies the canonical source for each topic. README is product onboarding, not
required implementation context. For unfamiliar GPUI APIs, inspect the exact pinned
source and examples; pinned revisions are authoritative.

## Priorities

Resolve conflicts in this order: data compatibility, correctness, data safety,
programmatic/API stability, UI responsiveness, cross-platform compatibility,
maintainability, memory efficiency, visual polish.

## Invariants

- Business logic belongs in application/core code, never a frontend. CLI, GPUI, and
  future interfaces must share OpenCollection parsing, environment resolution,
  request construction and execution, authentication, and persistence operations.
- The domain must not depend on GPUI, gpui-base, CLI libraries, YAML, HTTP-client
  types, or filesystem APIs. Keep crate dependencies directed inward.
- OpenCollection YAML is canonical. Do not duplicate collection structure in a
  proprietary database or invent format extensions without approval.
- Preserve unknown YAML where practical. Writes must be atomic, report failures,
  and refuse to overwrite externally modified sources.
- Runtime RequestKey and FolderKey values are session-only. Persistent CLI and
  desktop-session references use repository locators.
- Request selection in an open desktop workspace is an O(1) in-memory operation
  with no filesystem, parsing, database, or network work.
- Filesystem, network, and expensive parsing or highlighting work must not block
  the GPUI thread.
- The CLI is non-interactive by default. Structured stdout is deterministic,
  versioned JSON; diagnostics go to stderr. Keep stable error categories and exit
  codes and bound large output.
- Desktop components consume Probe semantic tokens. Use Longbridge gpui-base
  (crate gpui-base / import gpui_base), never the separate gpui-component crate.
  Do not change pinned GPUI dependencies during unrelated work.

## Data Safety and Tests

Every newly supported OpenCollection feature needs fixture-based tests, including
load → modify → save → reload where relevant. Core behavior must be testable without
launching GPUI or invoking the CLI binary. Preserve unrelated worktree changes and
never use unsafe merely to bypass ownership problems.

Before completing a code change, run:

```bash
unset CARGO_TARGET_DIR
cargo fmt --check
cargo clippy --all-targets --all-features -- -D warnings
cargo test --all-targets --all-features
cargo deny check advisories
```

For changes to Rust production behavior, also run the coverage and CRAP review in
[Development](docs/DEVELOPMENT.md). Run diff-scoped mutation testing on changed
production functions in the CLI, core, HTTP, OpenCollection, Postman, or Yaak
crates. Investigate each surviving mutant: it may expose a missing behavior
assertion, an equivalent change, platform glue, or a design problem. Add tests
only for contracts, invariants, compatibility, and regressions; never mirror
the implementation or add assertions merely to raise coverage. Do not split a
function solely to lower CRAP. Preserve the architecture and requested scope;
stop once the requested work and applicable checks pass, without unrelated
cleanup. Do not report completion while a required check fails.

## Scope

Implement only the requested feature. Do not add cloud sync, accounts, telemetry,
analytics, plugins, GraphQL, streaming protocols, Git provider APIs, or MCP without
explicit scope. Avoid unrelated refactors, inspect source instead of guessing, and
make the smallest coherent change.

<!-- prometheus-base:start v1 -->
# Agent Operating Rules

This region is the standing contract. It holds only invariants that must
survive compaction. Everything else lives in on-demand rules, skills, hooks,
and reference files. Where a hook enforces a rule stated here, the hook wins.

Managed by `prometheus-context-bootstrap`. Edits inside these markers are
overwritten on re-run. Write project prose outside them.

## Position and authority

- `.kbd-orchestrator/current-waypoint.json` is authoritative for position.
- `versions.toml` is authoritative for architecture decisions and dependency pins.
- READMEs go stale. Do not trust one over the two files above.
- Read the waypoint at session start. State the current phase before executing.

## Capability inversion

Agent kernels do not write. Mutating actions are gated in the trusted host
layer only, never in an agent kernel. Where the language allows it, this is
enforced at the dependency graph as a compile-time guarantee rather than a
runtime check. If a task appears to require a write from a kernel, stop and
surface the conflict instead of routing around it.

## Phase order

Task loop: Spec, Plan, Execute, Reflect.
Evolution loop: Compile, Evaluate, Optimize, Promote.

Running a phase out of order is a quality failure, not a shortcut. Name the
phase you are in. Do not execute before a plan exists.

## Verification boundaries

Finish a coherent implementation set before testing it. During implementation,
use static inspection and reasoning; use a narrow compiler check only when it is
required to unblock progress. Finish every planned production change in the phase,
then run one integration gate through the real production path and collaborators at
the final phase boundary. When the harness provides an agent team, keep reviewer,
auditor, verifier, and integration-checker roles dormant until that boundary. Unit,
mock-only, filtered-function, and per-edit tests are not completion evidence.
Per-stack commands live in `.claude/rules/`, loaded only when a matching file is read.

- `.claude/rules/rust.md` — rust tiers and hard rules

## Evidentiary standard

Address observed problems. An observed problem comes from an operator report, a
visible error or log, a failing test, or an explicit requirement. A concern that
is none of those gets one sentence and a question, never speculative code.

Defensive code — validation, guards, fallbacks, retries, timeouts — requires a
named failure scenario. Hardening at a real trust boundary present in the code
is a standing exception and is named in the completion summary, never added
silently.

## Evidence over assertion

Show the command and its output, the test result, or the artifact. "Looks done"
is not done. Report what was actually run and at which boundary. If a check could
not run, say which claims are therefore unverified. An unverified claim reported
as verified is worse than no check at all.

## Anti-sycophancy

Critics never see generation history. Review through the `artifact-critic`
subagent, which receives the artifact alone. The model that produced the work is
not the sole judge of whether it is good.

A reflection leads with the delta between plan and delivery, not with what
worked. The sycophancy gate may block a turn; fix the finding rather than
bypassing it.

## Learning and memory

Learning is append-only under `.prometheus/`: `session-log.md`, `decisions.md`,
`gotchas.md`, `postmortems/`, `knowledge/`. Never rewrite history; append, and
mark superseded entries rather than deleting them.

Write on a decision with a rationale, a defect and its root cause, a learned
constraint, a phase boundary, and a session summary. Read `gotchas.md` before
touching a subsystem.

Where a memory server is configured, it is the primary store and its write path
may time out. On failure, log to the markdown files above and continue. Never
block a task on the memory server.

## Architecture

- Single-writer build discipline within one build or target directory.
- Feature-based organization by capability, not by technical layer.
- Strict layering: UI, then hooks or view models, then stores, then services,
  then external. Reverse flow only through reactive state or events.
- Business state lives in explicit, inspectable systems, never in UI components
  or agent-only memory.
- Open standards first. Avoid lock-in unless explicitly required.
- Verify dependency versions against official sources before introducing them.
  Do not rely on training-era version knowledge.

## Scope

Minimum change that solves the problem. Do not refactor adjacent working code;
treat its current state as intentional. Mention unrelated issues, do not fix
them unasked. Before destructive or hard-to-reverse actions, confirm intent and
prefer a reversible path.

## Skills may be absent

Harnesses drop skill descriptions past a context budget, so a skill you expect
may not be listed. If one is missing, invoke it by name or say plainly that it
is unavailable and proceed from these rules. Never invent what an absent skill
would have done.

## Communication

Direct and execution-first. Structure claims as statement, mechanism, stakes.
Short declarative sentences. No marketing language.

Avoid: leverage as a verb, utilize, synergy, roadmap as a verb, journey,
harness as a verb, delve, revolutionary.

Every significant document names the uncomfortable thing — the scenario that
hurts the author's own position.

## Done

A task is done when its stated integration exit criteria pass at the applicable boundary, not when
the output looks plausible. Before declaring completion: remove anything added
that was not requested, confirm each guard traces to an observed problem or a
real boundary, and summarize what changed, how it was verified, and what remains
at risk.

## Execution scaffold

This section exists because the fleet is mixed. Frontier models supply most of
it by default; smaller and older models do not, and the failure is silent —
plausible output with a fabricated call in it. Omit this section only when
every model that reads this file is known to supply the behavior on its own.

### Before executing

Restate the task in one sentence, and name the phase. If the restatement does
not match what was asked, stop and ask rather than proceeding on the closer
reading. Name the files you intend to touch before touching them.

### Do not fabricate

Never invent an API, a file path, a package name, a command flag, or a
configuration key. If you have not read it in this session or it is not pinned
in `versions.toml`, verify it before using it. "I could not confirm this
exists" is a correct answer. A plausible identifier that does not exist costs
more than the question would have.

Do not guess at a tool's parameters. Read its schema. A tool call with invented
arguments fails in a way that looks like the tool is broken.

### Verification is explicit

Run the check. Paste the command and its actual output. Do not report a result
you did not observe, and do not describe what a test "should" produce.

If a check cannot run, say which specific claims are therefore unverified, and
why. Skipping a check silently and summarizing as if it passed is the failure
this rule exists to prevent.

### Code output

Never elide code with `...`, `// rest unchanged`, or a similar placeholder in a
file you are writing. Emit the complete content of every file you write.

When editing, change the minimum span. Do not reformat, reorder imports, or
rename adjacent symbols while making an unrelated change.

Match the file's existing conventions over your own defaults.

### Complete coherent sets

Batch related implementation work until a meaningful production path is complete.
Do not interrupt every edit with a build or test. Keep unrelated changes separate,
then validate the completed set through the smallest real integration boundary.

Do not start an unrelated subsystem while the current implementation set is partial.

### Stop conditions

Stop and ask when: the requirement is ambiguous in a way that changes the
design, two readings of the task lead to different files, the change would
break an existing behavior, or you are about to do something hard to reverse.

Stop when the goal is met. Do not continue into adjacent improvements.

### Format contracts

When a specific output format is requested — JSON, a table, a diff, a schema —
emit exactly that format with no preamble, no trailing commentary, and no
markdown fence unless the fence was asked for. A parser is often reading it.

### Self-check before reporting completion

State each of these explicitly, not as a claim that you did them:

1. What changed, file by file.
2. What was run to verify it, and the observed output.
3. What was added that was not requested — remove it, or list it and ask.
4. Which guards trace to an observed failure, and which do not.
5. What remains unverified, and why.

<!-- profile: mixed — see references/MODEL-PROFILES.md before changing -->
<!-- prometheus-base:end -->

<!-- uiux-routing:start v1 -->
## UI/UX routing
UI, styles, tokens, motion or copy → `prometheus-ui-ux`. Read `.agents/UI_UX_PROTOCOL.md` or its bundled default; preserve design authority.
All code: detect `.agent-team/project-routing.json` and real team manifests. Preserve selection; adopt a sole team; ask if ambiguous. Use relevant roles, disclosing sequential fallback.
Backend work loads no UI guidance. Review respects user-only skills and the completed-phase boundary.
<!-- uiux-routing:end -->

<!-- prometheus-team-routing:start v1 -->
For every code task, read `.agent-team/project-routing.json`, then its active team manifest and the relevant role instructions. Default to that team, selecting only roles whose responsibilities and ownership match the work. Preserve native permissions, models, concurrency limits and existing project instructions.
For UI work, load the role-bound `prometheus-ui-ux` or `prometheus-ui-review` skill. Prefer `.agents/UI_UX_PROTOCOL.md` when present; otherwise use the installed `prometheus-ui-ux/references/UI_UX_PROTOCOL.md`. Backend work must not load UI guidance.
Use native delegation when available. If unavailable, follow the selected role instructions sequentially and report that limitation. Review in the builder context is not independent review. Keep reviewers dormant until the complete implementation phase; allow one batched correction/confirmation cycle. Respect user-only skill invocation restrictions. Zed external ACP agents use their own native configuration; parallel UI threads are not an automatic delegation API.
<!-- prometheus-team-routing:end -->
