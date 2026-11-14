# Architecture Code Review / Refactoring Acceptance Checklist

Review only items relevant to the task. Do not penalize a project for not using every pattern. Each finding needs **evidence location, actual risk, minimal fix, and verification method**.

## A. Responsibilities and Dependencies

- [ ] Do Gameplay rules, state, UI, configuration, and saves have clear authority?
- [ ] Has Character become a God Object for inventory, Combat, Quest, and Save? Does a Coordinator only orchestrate flow?
- [ ] Are there bidirectional Component dependencies or unnecessary Build.cs module dependencies?
- [ ] Do stable ownership relationships use simple direct references instead of unnecessary Interface/Event Bus layers?
- [ ] Do public bases, interfaces, or UI depend on a concrete BP's entire content chain through casts, variables, or parameter types, or cross independent module boundaries? Verify with Reference Viewer / Size Map.

## B. SOLID / Inheritance / Composition

- [ ] Does each class have a clear reason to change, without meaningless single-function classes?
- [ ] Are there unjustified inheritance layers or subclasses created for parameter differences?
- [ ] Does ActorComponent decomposition provide real reuse or responsibility isolation?
- [ ] Do subclasses honor parent behavior and engine lifecycle contracts?
- [ ] Does a Subsystem's lifetime match business scope, avoiding Global Manager misuse?

## C. Observer / Mediator

- [ ] Are commands and events separate, with broadcasts only after business success?
- [ ] Are synchronous results and failure compensation explicit, without depending on Delegate listener order?
- [ ] Are listeners bound/unbound appropriately, without duplicates after Widget recreation or pooled Actor reuse?
- [ ] Does the Mediator coordinate a complex use case rather than simply forward calls?
- [ ] Are event data validity and ownership clear?
- [ ] Do GameplayTag / State Machine events have explicit recipients instead of global broadcasts with unknown recipients?
- [ ] Does a Component/Ability own AnimNotify window state and close it on Montage interruption or Ability end?

## D. Data-Driven / Editor

- [ ] Are DataAsset/DataTable choices suited to scale and editing needs?
- [ ] Are static Definition, Runtime State, and Save Data separate with one source of truth per field?
- [ ] Are missing rows, empty tables, CSV reimport, renamed rows, and invalid pointers handled?
- [ ] Do PrimaryDataAsset / soft references have an actual Asset Manager/loading plan, including pre-completion behavior, failures, retention, and packaged content?
- [ ] Can similar Items/Weapons be added without recompiling C++?
- [ ] Is Blueprint exposure minimal, with clear ToolTip/Category, defaults, ranges, and validation?
- [ ] Are old Blueprints, levels, and saves compatible after renames, reparenting, or Component migration?

## E. Object Pool / Performance

- [ ] Does profiling justify pooling, and was built-in UE pooling checked?
- [ ] Can Collision, Timer, Delegate, Instigator, Velocity, FX, UI/GC, etc. be completely reset?
- [ ] Do Active/Available states prevent double release and concurrent reuse?
- [ ] Are World destruction, PIE restart, and exhaustion safe?
- [ ] Are frame time and memory better after pooling?

## F. Stability and Delivery

- [ ] Are core failures visible through logs/assertions instead of silently skipped?
- [ ] Is `check` reserved for unrecoverable internal invariants and `ensure` for unexpected but continuable issues?
- [ ] Are UE C++, GC, Owner/Outer, Timer, and EndPlay lifetimes correct?
- [ ] Are existing behavior, third-party plugin boundaries, and asset migration paths preserved?
- [ ] Which Build / Automation Test / PIE / Profiler checks actually ran? Mark those not run.

## Suggested Output

### Findings

| Severity | Location | Confirmed issue / evidence | Impact | Minimal fix | Verification |
| --- | --- | --- | --- | --- | --- |
| Critical / High / Medium / Low | `Source/...:Line` or `/Game/... asset → Graph/Function` | Concrete failure scenario | ... | ... | ... |

### Dependencies and Data Flow

- System A and B: state/behavior each owns.
- A → B: direct call / Interface / Coordinator and why.
- B → UI: Delegate and notification timing.

### Alternatives and Rationale

- Simplest viable solution; conditions under which Mediator / Pool / PrimaryDataAsset is justified.

### Tests and Remaining Risks

- Executed; not executed; Blueprint/Asset/performance still requiring verification.
