---
name: ue5-architecture
description: Design, refactor, or review Gameplay architecture and Content directory organization for Unreal Engine 5 C++ and Blueprint projects. Use for content planning, asset organization, new systems, responsibilities across Actors, Components and Subsystems, cross-system communication, data-driven design, inheritance and composition, asset dependencies, or Blueprint-to-C++ migration. Do not automatically use for ordinary syntax explanations, isolated bug fixes, simple property changes, or adding logs only.
---

# UE5 Architecture Design & Review

Build an architecture suited to small and medium Unreal Engine games developed with AI assistance: clear responsibilities, controlled dependencies, convenient Editor/Blueprint configuration, and room to evolve, without adding abstractions merely to apply patterns. Explain in the user's language while retaining English UE terms such as Actor, Component, Projectile, Collision, Delegate, DataAsset, and Blueprint.

In this skill, “authority” means the single source of truth for rules and state, not network-replication Authority. A single-player project does not need servers, RPCs, or replication on this account.

For Content directory planning or asset organization alone, use [content-directories.md](references/content-directories.md) without applying the Gameplay architecture workflow below. Follow the workflow for system architecture.

## Required Workflow

1. **Verify the current state.** Determine whether the `.uproject` has `Modules` and whether the project has `Source/`. Inspect relevant C++, Blueprints, Build.cs, existing plugins and assets, architecture, and the target UE version. State assumptions when the project is inaccessible; never claim to have inspected Blueprint graphs, asset contents, or build results that you have not seen. Prefer existing systems and respect third-party maintenance and upgrade boundaries.
   - **Blueprint-only projects:** Apply the same principles using BP ActorComponents, Blueprint Interfaces, Event Dispatchers, and BP subclasses of `PrimaryDataAsset`. BP Components provide the authoritative layer attributed to C++ elsewhere in this skill.
   - **Adding a C++ module** is a project-level decision. Propose it separately, explain the rationale and cost, and obtain user confirmation before proceeding; do not slip it into ordinary design or review work.
2. **Define scope.** Identify business rules, runtime state, configuration, presentation/UI, saves, owners, and lifetimes. Address only extension points required by the task. Prefer single-player design; do not introduce network synchronization without a reason.
3. **Propose a lightweight design.** Allocate responsibilities among Actor, ActorComponent, UObject, Subsystem, Interface, Delegate, DataAsset, and DataTable as appropriate, and describe call direction and data flow. Distinguish direct references, commands, and event notifications; compare the simplest viable solution with any necessary abstraction.
4. **Evaluate patterns instead of applying them mechanically.** First decide whether a direct reference suffices before considering Interface, Observer, or Mediator. Consider extending an existing class before choosing inheritance or composition. Introduce object pooling only on performance evidence. Consult the relevant references.
5. **Define C++ / Blueprint boundaries and designer interfaces.** Assign core rules and authoritative state, Blueprint flow and presentation extensions, and DataAssets, DataTables, and Editor parameters according to the project. In existing C++ projects, prefer stable public contracts and critical rules in C++; in Blueprint-only projects use BP Components as in step 1, without adding C++ merely to enforce this division. Expose the minimum necessary API, define state read/write access and data validation, and consult [balancing-cpp-blueprint.md](references/balancing-cpp-blueprint.md) for cross-BP types, shared structs/enums, or asset loading.
6. **Implement or review.** When coding is requested, make the smallest verifiable change. Check dependencies and lifetimes first, then state and data, and finally presentation and debugging. Do not refactor unrelated modules without authorization or make broad code changes during a design task.
7. **Verify and report.** Explain risks, dependency changes, required tests, and unverified items. Run suitable tests when a build or Editor environment is available and report actual results; never invent successful verification. For architecture reviews, report issues by severity and location, followed by concise fixes.

## References: Read as Needed

Load only files directly relevant to the task:

| Focus | Reference |
| --- | --- |
| SOLID, separation of concerns, loose coupling, responsibilities | [principles.md](references/principles.md) |
| Inheritance, composition, ActorComponent, Subsystem selection | [class-design.md](references/class-design.md) |
| Observer, Mediator, Interface, Delegate, direct calls, GameplayTag events, State Machine, AnimNotify | [communication.md](references/communication.md) |
| DataAsset, PrimaryDataAsset, DataTable, configuration / runtime / save separation | [data-driven.md](references/data-driven.md) |
| C++ vs Blueprint, designer convenience, Editor API, architecture notes, BP dependencies, shared types, asset loading | [balancing-cpp-blueprint.md](references/balancing-cpp-blueprint.md) |
| Object pooling for projectiles, effects, and similar objects | [object-pooling.md](references/object-pooling.md) |
| Ownership/GC, logging, assertions, performance, and testing; C++ style belongs to `ue5-epic-cpp-standard` | [ue5-implementation.md](references/ue5-implementation.md) |
| Architecture review and delivery acceptance | [review-checklist.md](references/review-checklist.md) |
| Single-player extraction game Loot → Inventory → UI example | [extraction-example.md](references/extraction-example.md) |
| Blueprint-to-C++ migration, Reparent, CoreRedirects, UserDefinedStruct/Enum replacement | [bp-to-cpp-migration.md](references/bp-to-cpp-migration.md) |
| Content directory planning, asset organization, development folders, migration risks | [content-directories.md](references/content-directories.md) |

Use the asset naming guide in `ue5-epic-cpp-standard` for asset names and prefixes. Do not automatically rename or move existing assets as a consequence of directory planning.

## Default Decision Order

- **Responsibilities:** Establish a clear reason for each module to change, then use composition for reusable capabilities. Inheritance is appropriate for a natural `is-a` relationship that honors the behavioral contract; do not ban it mechanically.
- **Communication:** Use direct references for stable ownership, interfaces for common behavior across types, delegates/observers for state-change notifications, and a mediator/coordinator only for complex cross-system flows. Notifications do not replace required business commands.
- **Data:** Consider DataAssets for a small number of individually edited items or complex references; DataTables for many homogeneous rows and bulk balancing; and PrimaryDataAssets when on-demand loading or Asset Manager management is needed. Separate configuration, runtime state, and save data.
- **Performance:** Object pooling trades lifecycle complexity for performance stability. Do not use it by default; profile first. Consider UE's built-in pooling support, such as for particles, before building another system.
- **Engine constraints:** Prefer native UE concepts and lifecycles and respect the project's actual version. Verify newer engine APIs and third-party plugin behavior against source or official documentation; do not present speculation as fact.

## Output Format: Adapt to Task Complexity

### New System Design

1. Requirements and assumptions: Gameplay scope, single-player/multiplayer, content volume, and performance hotspots.
2. Recommended design: components and responsibilities, key data types, dependencies/events/calls, and C++ / Blueprint division.
3. Rationale for using or not using Interface, Mediator, pooling, DataTable, inheritance, and so on.
4. Implementation order in minimal working increments.
5. Acceptance and risks: correctness, lifecycle, Editor configuration, performance, and testability.

### Refactoring / Code Review

1. List confirmed issues first, including severity, evidence and impact; distinguish inference from evidence.
2. Give a minimal change plan, preserved behavior, and migration risks. Check Blueprint references, data asset migration, and save compatibility in particular.
3. Give concrete verification steps and identify tests not run. Do not substitute generic pattern terminology for actionable advice.

## Must Avoid

- Generating many Managers, Interfaces, BaseClasses, or forwarding classes merely to satisfy SOLID.
- Replacing every cross-component call with a Delegate, turning every system into a Subsystem, or pooling every object.
- Letting Widgets, presentation Blueprints, or concrete content BP subclasses such as a specific enemy or weapon bypass their authoritative layer to rewrite core rules or maintain a second source of truth. In Blueprint-only projects a BP Component may be the authoritative layer. Do not store mutable runtime inventory or character state in DataAssets used as static configuration.
- Using `check` for recoverable errors such as player input, or silently swallowing important invariant failures with an `if`.
- Claiming an architecture exists or has been verified without inspecting the relevant existing plugins and Blueprints.
