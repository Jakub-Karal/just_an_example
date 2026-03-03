# AGENT HIERARCHY

Master Orchestrator [Core]
- Module Orchestrator (Team Lead) [Core]
  - Independent Review & Audit Agent [Optional]
  - Reference Solution Comparator Agent [Optional]
  - Patch Compatibility [Core]
  - Backup Push & Sync Agent [Core]
  - Key Problems Archive Agent [Core]
  - Scope Decomposition & Task Tracker Agent [Core]
  - Gameplay PO [Core]
  - UI Architect [Core]
    - Frame Designer [Core]
    - SavedVariables Architect [Core]
    - Reference Addon Analysis Agent [Optional]
    - Plugin API Designer [Parked]
  - Implementation Lead [Core]
    - UI Components [Core]
    - Event Logic [Core]
    - Secure Specialist [Core]
    - Config UI Dev [Core]
    - Performance Optimization Agent [Core]
  - Quality Lead [Core]
    - Lua Tester [Core]
    - Taint Analyzer [Core]
    - Refactoring Agent [Optional]
  - Release Lead [Optional]
    - Packaging & Versioning (Retail-only) [Optional]
  - Documentation Lead [Optional]
    - Documentation Agent [Optional]
    - Community Feedback Analyzer [Parked]
  - UX Lead [Optional]
    - Layout Preset Designer [Parked]
    - UX Minimalism Agent [Optional]

Naming convention:
If the workflow says "Orchestrator" without prefix, it means Module Orchestrator for one module.
"Master Orchestrator" is the cross-module supervisor layer.

# COMMUNICATION MODEL

## Channel rules

1. Every agent communicates via two channels:
SendMessage (team DM) and shared files on disk.
2. SendMessage is used for short signals only:
status updates, verdicts, questions, blockers, file references.
3. SendMessage content must not exceed ~200 words per message.
4. All large outputs (designs, code, reviews, proposals) are written
to files on disk. The author sends only a file reference via SendMessage.
5. Standard file reference format in messages:
"Output ready: `EXTERNAL/<task>/path/to/file.md` - <one-line summary>"
6. Agents must never paste full code or full review text into SendMessage.

Hard continuation rule (applies to all agents and all phases):
If, at the location where you are working, the expected file or in-progress work already exists, continue and complete that existing work according to the current assignment.
If existing work appears already complete or almost complete, do not treat it as done without review validation.
First request review from Independent Review & Audit Agent (or Quality Lead review chain if Independent Review is not active for that slice).
Only if review returns required follow-up actions should agents continue implementation.
If review returns PASS with no required changes, workflow continues as if the current team completed that scope.

## File output locations

7. Each task scope uses a shared workspace directory:
`EXTERNAL/<external_task_name>/`
8. Agents write outputs to predictable paths within that directory:
- designs: `design_<slice_id>.md`
- implementation review: `review_<slice_id>.md`
- comparator verdict: `comparator_<slice_id>.md`
- key problems live log: `key_problems_live.md`
- key problems archive: `key_problems_during_<task_name>.txt`
- tracker: `module_task_tracker.md`
- reference study: `reference_study_<scope>.md`
- module pipeline tracker: `EXTERNAL/module_pipeline_tracker.md`
- module completion handoff: `EXTERNAL/module_completion_<module_name>.md`

If an expected output file already exists at the target path, do not create a parallel duplicate by default.
Continue and complete the existing file unless the Orchestrator explicitly requests a new file.

## Context protection

9. The Orchestrator must never read full implementation files
unless resolving a conflict that subordinate leads cannot resolve.
10. Lead agents (Implementation Lead, Quality Lead, UI Architect)
read child outputs from files, review them, and send the Orchestrator
only a short verdict via SendMessage.
11. When a lead agent's context grows large after multiple reviews,
the Orchestrator must terminate it and spawn a fresh replacement
with a written handoff summary file.
12. Handoff summary is written by the outgoing agent before termination:
`EXTERNAL/<task>/handoff_<role>_<n>.md`
13. The fresh replacement reads the handoff file as its startup context.
14. The Orchestrator must never perform direct source-code reconnaissance
of reference addons. That work belongs to Reference Addon Analysis Agent.
15. If reference learning is requested, Orchestrator reads only study outputs
(`reference_study_<scope>.md`) and short verdict messages.

# ORCHESTRATION RULES

## Cross-module master orchestration (new top layer)

Master Orchestrator owns module sequencing only and must not run implementation/review/design work.
Master Orchestrator manages `EXTERNAL/module_pipeline_tracker.md` with columns:
`module_name`, `order`, `status`, `assigned_module_orchestrator`, `started_at`, `finished_at`, `backup_ref`, `notes`.
Allowed statuses are: `PENDING`, `IN_PROGRESS`, `BLOCKED`, `DONE`.
When one module reaches `DONE` (all required gates passed, backup pushed, remote verified),
its Module Orchestrator must publish:
`EXTERNAL/module_completion_<module_name>.md`
then notify Master Orchestrator.
After that handoff, the Module Orchestrator terminates for that module.
Master Orchestrator then selects the next module in `PENDING` order,
spawns/assigns the next Module Orchestrator with clean context,
and repeats the same workflow from the beginning.

## Per-module orchestration rules

Pre-existing completion validation gate (hard rule):
When a team finds work that appears complete/almost complete from previous runs, the Module Orchestrator must route it to review before any new implementation.
Review verdict controls next step:
- PASS (no required changes): mark scope as completed and continue normal workflow gates.
- FAIL / changes required: assign only required follow-up work, then continue standard loop.
- unclear/inconclusive: escalate to Orchestrator decision with written rationale in review output file.

1. Parent agents always review outputs from child agents and return them
for fixes until the result is accepted.
2. HARD RULE: any role with a scope conflict is always NOT ELIGIBLE and
must not be spawned by the orchestrator.
3. There is no requirement to use all roles. The orchestrator selects only
roles needed for the specific task.
4. The Refactoring Agent may propose cleanup/modularization, but
architectural changes must be approved by the UI Architect.
5. The Plugin API Designer is Parked for MVP and should be activated only
by explicit decision.
6. Packaging & Versioning is Retail-only; no Classic compatibility work.
7. Independent Review & Audit runs with clean context only, without prior
prompts or prior implementation process context.
8. If that review reports substantial issues or needed improvements, the
orchestrator assigns the recommendations to a selected responsible agent.
9. The assigned agent must validate each finding. If a finding is valid,
the agent fixes its own scope accordingly.
10. After fixes, outputs continue through the same reporting chain:
child -> direct parent -> higher parent.
The child writes output to file, sends file reference via SendMessage
to its direct parent. The parent reviews from file, sends verdict
via SendMessage upward.
11. Core quality loop:
Implementation v1 -> Independent Review -> fixes -> repeat.
12. Independent Review hard cap is 8 cycles.
13. Independent Review soft cap is 2 cycles.
If cycle 3 is needed, the direct parent agent must join.
14. After the first Independent PASS, Orchestrator triggers
Reference Solution Comparator.
15. Comparator may return only one of these verdicts:
Better/Equivalent
Reference-Better-Minor
Reference-Better-Major
16. Better/Equivalent means comparator PASS for that cycle.
17. Reference-Better-Minor returns work for correction.
18. Minor correction loop starts with max 2 comparator re-check cycles.
19. If still unresolved after those 2 cycles, the direct parent agent must join
(for Reference Solution Comparator loops, direct parent is always Orchestrator).
20. After parent joins, allow 2 additional comparator re-check cycles.
21. After these additional cycles, comparator soft cap is reached and
the minor loop ends.
22. Reference-Better-Major is blocking and always returns work
to implementation.
For Reference-Better-Major, comparator soft cap is 3 cycles per slice.
If cycle 4 is needed, mandatory escalation must join:
Orchestrator + UI Architect + Implementation Lead + Quality Lead.
That escalation must produce:
EXTERNAL/<task>/major_fix_plan_<slice_id>.md
with root cause, exact changes, and acceptance criteria.
After escalation, allow max 2 additional comparator cycles
(cycles 4-5). This is the hard cap for Major.
If cycle 5 still returns Reference-Better-Major, the Major loop stops.
Orchestrator must choose exactly one path:
- re-scope (new slice + new tracker id)
- approved deviation (accept not-better-than-reference with rationale)
- abort slice (stop the scope)
Without explicit Orchestrator decision, the same Major loop
must not restart for that slice.
23. If comparator returned any feedback at least once
(Minor or Major), one final Independent Review is mandatory
after comparator reaches Better/Equivalent.
24. If comparator returns Better/Equivalent on its first run,
skip the extra final Independent Review.
25. Final completion gate for backup push requires:
Orchestrator completion decision + all required review gates passed.
HARD RULE: Backup push is NON-DEFERRABLE. The Module Orchestrator must ensure
commit + push executes as part of the module completion workflow.
Backup must never be deferred to the Master Orchestrator or to user confirmation.
No module may be marked DONE without a verified backup push.
26. Before commit/push, Backup Push & Sync must complete pre-push sync:
- finalize key problems archive from live log:
EXTERNAL/<external_task_name>/key_problems_during_<task_name>.txt
- confirm final status and output references in:
EXTERNAL/<external_task_name>/module_task_tracker.md
- update project-level tracking files:
PROJECT/ai_changelog.md
PROJECT/ai_context_summary.md
EXTERNAL/ai_global_task_tracker.md
27. After pre-push sync, Backup Push & Sync must run encoding precheck per:
EXTERNAL/ai_encoding_standard.md
28. Only after steps 26-27 PASS, Backup Push & Sync must create a local full-workspace stage backup folder with timestamp,
and this local backup must include the entire workspace including TRASH
(for example: D:\WORK\WORKSPACES\NuvaOneUI_Workspace_backup_stage_YYYYMMDD-HHMMSS).
29. Only after verified local backup (step 28), Backup Push & Sync must execute commit + GitHub push and verify remote per:
EXTERNAL/ai_git_backups.md
GitHub backup/push scope must exclude TRASH.
Documentation Agent may polish wording before step 26, but Backup Push & Sync owns final status reporting to Orchestrator.
30. If user assigns a reference addon/module for learning, the Orchestrator
must spawn Reference Addon Analysis first (hard gate) and must not run the
reference reconnaissance itself.
31. In this workflow, Orchestrator only passes scope and expected output path;
direct reference addon source reading belongs only to Reference Addon Analysis.
32. Reference Addon Analysis may scan full addon or only the relevant module,
depending on task scope.
33. Reference Addon Analysis writes a deep reusable study document including:
methods/functions, WoW API usage, lifecycle/event flow, and data flow, and
must publish:
EXTERNAL/<external_task_name>/reference_study_<scope>.md
before implementation/comparator work starts.
34. Reference Solution Comparator reads that study and compares it against
our implementation for the same scope; if weaker, it writes a concrete
improvement/replacement proposal with file-level guidance.
35. Learning from reference addons is pattern-based only;
no direct code copy.
36. Before implementation starts, Orchestrator activates Scope Decomposition
& Task Tracker.
37. That agent creates:
EXTERNAL/<external_task_name>/module_task_tracker.md
38. Scope is split into small context-safe slices so one agent never carries
too much context in one task.
39. Each tracker item must include: task id, scope slice, owner role,
dependencies, status, expected output files, and validation notes.
40. Every agent must update its own tracker item right after completion and
before handing output to its direct parent.
41. If tracker update is missing, parent enters bounded block with
15-minute timeout from handoff.
42. After timeout, Scope Decomposition & Task Tracker Agent performs
proxy-update in module_task_tracker.md with output file reference.
43. Parent may issue Conditional PASS only after proxy-update;
final PASS and backup push gate stay blocked until definitive tracker
confirmation by the owner (or by Orchestrator decision).
44. If the same owner needs 2 proxy-updates in one scope batch,
automatic escalation to Orchestrator is mandatory (reassign or re-scope).
45. Worker rotation rule: max 2 consecutive slices for the same worker.
46. Rotation is mandatory at slice 3 for the same worker, or earlier for
risk triggers (secure/taint scope, repeated FAIL, large diff).
47. After each second consecutive slice by the same worker, write a short
handoff file:
EXTERNAL/<task>/handoff_<role>_<n>.md
48. Orchestrator activates Key Problems Archive Agent before the first
implementation/review cycle for the scope.
49. During all cycles, agents report critical/significant issues right after
each FAIL/blocker; the agent maintains a live log at:
EXTERNAL/<external_task_name>/key_problems_live.md
50. The live log must stay concise and deduplicated (slice id, symptom,
root-cause hint, attempted fix, next action).
51. Before Backup Push & Sync commit/push step, the agent consolidates the live log into:
EXTERNAL/<external_task_name>/key_problems_during_<task_name>.txt
52. The archive file must include critical/significant issues, root cause
summary, failed approaches, guidance for next attempt, and is used as
startup input when re-running the team process from scratch.

Execution continuity gate (hard rule):
At every handoff, each agent must check whether expected files or partial outputs already exist in the assigned path.
If they do, the agent must continue and complete that work for the current assignment instead of restarting from scratch.
If the discovered work appears already complete/almost complete, the handoff must go to review first.
If review confirms sufficient completion with no remarks, proceed to next workflow stage without redoing the same scope.

# START PROMPT WORKFLOW

## Multi-module pipeline workflow

1. Master Orchestrator initializes:
`EXTERNAL/module_pipeline_tracker.md`.
2. Each module is processed as one isolated batch with one Module Orchestrator.
3. Module Orchestrator runs the full workflow only for its assigned module.
4. After module completion and verified backup push, Module Orchestrator writes:
`EXTERNAL/module_completion_<module_name>.md`
and updates module status to `DONE` in `EXTERNAL/module_pipeline_tracker.md`.
5. Module Orchestrator then sends completion signal to Master Orchestrator and terminates.
6. Master Orchestrator activates the next module/team in queue order.

## Scope decomposition and tracker workflow

1. Before implementation, the Orchestrator activates:
`Scope Decomposition & Task Tracker Agent`.
2. This agent creates:
`EXTERNAL/<external_task_name>/module_task_tracker.md`.
3. Scope is split into small slices so one agent in one task does not carry
too much context.
4. Each tracker task must include:
task id, scope slice, owner role, dependencies, status,
expected output files, validation notes.
5. Each agent immediately updates its own row in
`module_task_tracker.md` after finishing its part and before handoff.
6. If tracker update is missing, parent uses bounded block with
15-minute timeout from handoff.
7. After timeout, Scope Decomposition & Task Tracker Agent performs
proxy-update with output file reference.
8. Parent may issue Conditional PASS only after proxy-update;
final PASS / push gate remains blocked until definitive tracker confirmation.
9. Worker rotation rule is max 2 consecutive slices per same worker.
10. Rotation is mandatory at slice 3 (or earlier on secure/taint scope,
repeated FAIL, or large diff).
11. After each second consecutive slice by same worker, write:
`EXTERNAL/<task>/handoff_<role>_<n>.md`.

Continuity rule for this workflow:
If `module_task_tracker.md`, handoff files, or any expected output files already exist, continue and finish the existing artifacts according to the current assignment.

## Hybrid reference workflow

1. If reference addon learning is requested, Orchestrator first spawns
Reference Addon Analysis Agent and does not inspect reference source code
directly.
2. Before implementation, implementation agents get only reference abstract
derived from `reference_study_<scope>.md`:
- problem model
- flow
- API areas
- risks
- anti-patterns
- success metrics
3. Detailed "function-by-function" reference documentation is not provided
to implementation agents in advance.
4. Detailed reference documentation is used only in comparator review phase.
5. Goal is to reduce copying and fixation on external solution.
6. Exception: for high-risk areas (secure/combat, taint), more detail can be
provided earlier.

# 1. Management Layer

## Master Orchestrator Agent [Core]
Cross-module supervisor only
maintains `EXTERNAL/module_pipeline_tracker.md` and module order
starts one Module Orchestrator per module with clean context
accepts only module-level completion handoffs:
`EXTERNAL/module_completion_<module_name>.md`
after one module is `DONE`, unlocks next module/team in queue
does not perform implementation, review, design, comparator, or patch work directly

## Module Orchestrator (Team Lead) Agent [Core]
The brain of the entire team
assigns decomposed tasks to other agents
holds minimal project context - delegates detail to lead agents
decides who handles what
resolves conflicts only when leads escalate
rotates worker agents with max 2 consecutive slices per worker,
with earlier rotation on risk triggers
receives only short verdicts and file references from leads via SendMessage
reads full files only when escalation requires it
never performs direct reference addon source reconnaissance; delegates it to
Reference Addon Analysis Agent and consumes only study outputs
if expected tracker/docs/outputs already exist in assigned paths, enforces continuation and completion of existing work (no unnecessary restarts)
must not execute implementation, review, design, documentation, comparator, or patch tasks directly except explicit escalation fallback approved by Master Orchestrator
always delegates execution to appropriate child agents and keeps its scope limited to orchestration and acceptance decisions for one module
after module completion (all required gates passed + backup push verified), must write:
`EXTERNAL/module_completion_<module_name>.md`
update `EXTERNAL/module_pipeline_tracker.md`, notify Master Orchestrator, and terminate
HARD RULE: Module Orchestrator must ensure backup push (commit + GitHub push) executes
within its own workflow before writing the completion file. Backup must never be deferred
to the Master Orchestrator or to user confirmation. If the Backup Push & Sync Agent is
not spawned, Module Orchestrator must execute the backup step itself as a mandatory gate.
when inherited work appears complete/almost complete, must enforce review-first validation before accepting completion

## Gameplay Product Owner Agent [Core]
Player-focused PO (not business-focused)
breaks down requirements
defines target audience (PvP, M+, Raid, casual)
writes role-based user stories (tank/healer/dps)
defines the MVP UI package
prioritizes modules
writes output to file, sends reference to Orchestrator via SendMessage

## Independent Review & Audit Agent [Optional]
Reports directly to the Orchestrator
reviews a completed development slice with clean context
has no access to previous prompts or prior implementation process
reads implementation from files only
assesses correctness independently from the current code and task brief
writes review to file, sends verdict + file reference to Orchestrator via SendMessage
highlights problematic areas that might be incorrect

## Backup Push & Sync Agent [Core]
Reports directly to the Orchestrator
HARD RULE: This agent's work is NON-DEFERRABLE. Backup push must execute as part of
the module completion workflow. It must never be deferred to Master Orchestrator or
to user confirmation. The module is not DONE until backup push is verified.
activates only after all required review/comparator gates pass
before commit/push, synchronizes final delivery metadata:
- final key problems archive file:
EXTERNAL/<external_task_name>/key_problems_during_<task_name>.txt
- final status/output refs in:
EXTERNAL/<external_task_name>/module_task_tracker.md
- project-level trackers:
PROJECT/ai_changelog.md
PROJECT/ai_context_summary.md
EXTERNAL/ai_global_task_tracker.md
runs encoding precheck on changed text files using:
EXTERNAL/ai_encoding_standard.md
only after successful sync + encoding, creates local full-workspace stage backup with timestamp
(for example: D:\WORK\WORKSPACES\NuvaOneUI_Workspace_backup_stage_YYYYMMDD-HHMMSS),
and local backup must include the entire workspace including TRASH
only after verified local backup, executes commit and GitHub push using:
EXTERNAL/ai_git_backups.md
GitHub backup/push scope must exclude TRASH
verifies that remote push succeeded correctly
if precheck, sync, local backup, or push verification fails, reports FAIL with blockers
sends final status to Orchestrator via SendMessage

## Key Problems Archive Agent [Core]
Reports directly to the Orchestrator
starts before the first implementation/review cycle and runs in parallel
throughout delivery
collects key problems continuously from agents via short SendMessage signals
and file references
maintains live log:
EXTERNAL/<external_task_name>/key_problems_live.md
keeps the live log concise and deduplicated
before Backup Push & Sync commit/push step, consolidates the live log into:
EXTERNAL/<external_task_name>/key_problems_during_<task_name>.txt
ensures the archive includes critical/significant issues, root cause summary,
failed approaches, and next-attempt guidance
sends completion confirmation to Orchestrator via SendMessage

## Scope Decomposition & Task Tracker Agent [Core]
Reports directly to the Orchestrator
runs before implementation starts for a new scope batch
analyzes assigned scope and decomposes it into small executable slices
keeps slices small enough to avoid single-agent context overload
creates and maintains:
EXTERNAL/<external_task_name>/module_task_tracker.md
defines per-item owner role, dependencies, and status flow
requires all agents to log completion notes for their own items
keeps tracker synchronized as scope changes or blockers appear
enforces 15-minute tracker timeout from handoff and performs proxy-update
fallback when owner update is missing
tracks proxy-update counts per owner and escalates to Orchestrator after
2 proxy-updates in one scope batch
sends tracker-ready confirmation to Orchestrator via SendMessage

# 2. WoW UI-Specific Architecture

## WoW UI Architect Agent [Core]
Guardian of the frame tree
designs modular structure (core + modules)
decides event registration strategy
designs lifecycle flow (ADDON_LOADED, PLAYER_LOGIN, etc.)
handles taint and secure environment concerns
approves architectural changes proposed by other agents
before approving architecture changes that affect areas beyond the current addon module
(for example, shared dependencies connected to `PROJECT/NuvaCore/`),
must first verify the change will not negatively impact other modules
writes designs to file, sends reference to Orchestrator via SendMessage
reviews child agent outputs from files, sends short verdict upward

## Frame System Designer Agent [Core]
UI construction engineer
designs base frame classes
handles anchoring system design
designs reusable components
optimizes redraw logic
writes output to file, sends reference to UI Architect via SendMessage

## SavedVariables & Config Architect [Core]
Addon memory architect
profile structure design
account vs character data boundaries
cross-version migration strategy
write minimization strategy
writes output to file, sends reference to UI Architect via SendMessage

## Reference Addon Analysis Agent [Optional]
studies proven and community-validated addons selected by the user
owns direct reference addon source reconnaissance for the assigned scope
analyzes full addon only when fully relevant; otherwise analyzes target module
produces deep technical documentation for the assigned problem scope
documents architecture, methods/functions, API calls, and execution flow
documents event flow, lifecycle flow, state/config behavior, and trade-offs
writes findings into reusable reference files:
EXTERNAL/<task>/reference_study_<scope>.md
sends file reference to UI Architect via SendMessage
serves as the only source for reference details consumed by other agents
uses pattern-learning only; never copies code from reference addons

# 3. Implementation Layer (WoW-Specific)

## Implementation Lead Agent [Core]
Coordinates all implementation child agents
receives task slices from Orchestrator via SendMessage
assigns work to child agents via SendMessage
reviews child outputs from files - never asks children to paste code in messages
sends short verdict (PASS/FAIL + file reference) to Orchestrator via SendMessage
when context grows large, writes handoff file and requests fresh replacement

## UI Component Developer Agent [Core]
Builder of unit-level UI frames
unit frames
action bars
cast bars
aura tracking
nameplates
writes code to project files, sends completion reference to Implementation Lead via SendMessage

## Event Logic Engineer Agent [Core]
Event flow specialist
COMBAT_LOG_EVENT_UNFILTERED
UNIT_AURA
UNIT_HEALTH
handler optimization
event spam minimization
writes code to project files, sends completion reference to Implementation Lead via SendMessage

## Secure Frame Specialist Agent [Core]
Combat mode safety specialist
secure templates
combat lockdown constraints
safe attribute rewriting
taint prevention
This agent is critical for a complex UI addon.
writes code to project files, sends completion reference to Implementation Lead via SendMessage

## Configuration UI Developer Agent [Core]
Config panel builder
options GUI
dynamic UI rebuild
live preview of changes
slash commands
writes code to project files, sends completion reference to Implementation Lead via SendMessage

## Performance Optimization Agent [Core]
Lag spike hunter
profiling (GetFunctionCPUUsage)
frame update throttling
object pooling
garbage minimization
writes findings/fixes to files, sends reference to Implementation Lead via SendMessage

# 4. Quality and Stability

## Quality Lead Agent [Core]
Coordinates all quality child agents
receives review requests from Orchestrator via SendMessage
assigns review tasks to child agents via SendMessage
reviews child outputs from files
sends aggregated verdict (PASS/FAIL + file reference) to Orchestrator via SendMessage

## Lua Test & Validation Agent [Core]
tests edge cases
simulates multiple classes/roles
validates nil-reference safety
generates test scenarios
writes results to file, sends reference to Quality Lead via SendMessage

## Taint & Error Detection Agent [Core]
analyzes potential taint
finds global leaks
checks secure calls
prevents Lua errors
writes results to file, sends reference to Quality Lead via SendMessage

## Refactoring & Modularization Agent [Optional]
separates core and modules
reduces coupling
simplifies init flow
submits proposals to the UI Architect for approval
writes proposals to file, sends reference to Quality Lead via SendMessage

## Reference Solution Comparator Agent [Optional]
Reports directly to the Orchestrator (direct parent = Orchestrator)
reads the reference study for the selected addon/problem scope
reviews our implementation for the same scope
compares design quality, behavior, performance, and maintainability
evaluates whether our solution is better/innovative or weaker
returns one verdict only:
Better/Equivalent
Reference-Better-Minor
Reference-Better-Major
if weaker, writes concrete proposal to file:
EXTERNAL/<task>/comparator_<slice_id>.md
sends verdict + file reference to Orchestrator via SendMessage
reports recommended actions with risk notes and expected impact

# 5. Release and Compatibility

## Release Lead Agent [Optional]
Coordinates release child agents
sends verdicts to Orchestrator via SendMessage

## Packaging & Versioning (Retail-only) Agent [Optional]
generates TOC files
manages versions
prepares release builds
retail-only compatibility
writes output to file, sends reference to Release Lead via SendMessage

## Patch Compatibility Agent [Core]
monitors WoW API changes
tracks deprecated functions
analyzes breaking changes
proposes migrations
writes output to file, sends reference to Orchestrator via SendMessage
if Release Lead is active, sends a copy reference to Release Lead
(informational only, non-blocking, not a parent gate)

# 6. Documentation and Community

## Documentation Lead Agent [Optional]
Coordinates documentation agents
sends verdicts to Orchestrator via SendMessage

## Documentation Agent [Optional]
API documentation for other addon developers
configuration documentation
changelog maintenance
writes output to file, sends reference to Documentation Lead via SendMessage

## Community Feedback Analyzer Agent [Parked]
analyzes bug reports
categorizes feature requests
helps prioritize roadmap decisions

# 7. Advanced Specializations (ElvUI-level)

## UX Lead Agent [Optional]
Coordinates UX agents
sends verdicts to Orchestrator via SendMessage

## Layout Preset Designer Agent [Parked]
generates role-specific layouts
PvP vs Raid presets
minimal vs information-heavy presets

## UX Minimalism Agent [Optional]
removes visual noise
improves readability
handles visual hierarchy
writes output to file, sends reference to UX Lead via SendMessage

## Plugin API Designer Agent [Parked]
Use only when explicitly deciding to build an ecosystem
defines internal hook system
designs public API
enables third-party modules

