# Unreal Engine C++ Implementation and Quality

## Engine First

- Respect the project's actual UE version, modules, and C++ standard. Even for UE 5.8, verify local engine source or official APIs, especially plugins and Experimental/Beta features.
- Prefer native `AActor`, `UActorComponent`, `UObject`, `USTRUCT`, `UInterface`, Delegate, Subsystem, DataAsset, DataTable, and reflection macros; do not transplant traditional C++ frameworks that conflict with UE lifecycles.
- Justify Build.cs dependency changes and avoid circular module dependencies for small features.
- Do not replace every `Tick` with events merely for uniformity; check whether continuous real-time updates are needed. State-change notifications generally should not be polled every frame.

## References and Lifetimes

- Identify Actor Owner, Outer, World, lifetime scope, GC, and hard versus soft references.
- Store persistent UObject references according to UE reflection/GC rules. Use weak references for non-ownership and soft pointers for on-demand loading.
- Check callback target validity for delegates, timers, async requests, level unload, and EndPlay; avoid duplicate registrations.
- Separate loadable static definitions from mutable runtime state. A long-lived service must not access a World-scoped Actor after it becomes invalid during a level transition.

## Assertions, Logging, and Failure Visibility: Project Conventions

These are project conventions, not Epic Coding Standard requirements; C++ style is covered by `ue5-epic-cpp-standard`.

- `check/checkf`: development invariants that must hold and after whose violation execution should not continue. Never use for recoverable errors such as player input, network packets, or save files.
- `ensure/ensureMsgf`: unexpected conditions that should hold but allow continuation, with clear diagnostics. Do not use `ensure` as a normal `if` branch.
- `UE_LOGFMT`: diagnose recoverable runtime failures, missing assets, and context. Define a project Log Category and include ItemId/Owner/World when needed.
- Failures need reasonable behavior: `TryAddItem` should return failure rather than only logging or silently skipping. Validate critical Blueprint assignments/configuration.
- `UE_LOGFMT` usually needs `#include "Logging/StructuredLog.h"`; verify format arguments and macro signatures against the project's engine version.

```cpp
if (!IsValid(ItemDefinition))
{
    UE_LOGFMT(LogTemp, Error, "Item definition is invalid for pickup {PickupName}",
        ("PickupName", GetNameSafe(this)));
    return false;
}
```

Use `ensureMsgf` to expose unexpectedly missing required configuration, not normal recoverable states such as a full inventory.

## Verifiable Implementation

- UI events: broadcast only after successful changes; avoid duplicate subscriptions on recreation and dangling callbacks on unsubscribe.
- DataTables: test invalid RowName, empty tables, reimport, unloaded soft assets, and fallback.
- DataAssets: two item instances referencing one Definition must not corrupt each other's runtime state.
- Composition/inheritance: check base initialization, Owner/Component lifetimes, and destruction/EndPlay order.
- Pools: stress test the full reset checklist in [object-pooling.md](object-pooling.md).
- Performance: introduce pooling or complex caching only with profiling evidence and compare frame times.
- Prefer UE Automation Tests, functional tests, or PIE as appropriate. Run available tests and state explicitly what remains unverified.

## AI Editor Tools

- If Aura Editor Agent, Python, or MCP can access the UE Editor, verify Blueprint classes, assets, DataTable schemas, references, and Editor configuration before editing.
- If tools are unavailable, provide actionable Editor steps or identify manual checks; never pretend to have edited assets in the Editor.
- For Fab systems, prefer an Adapter or independent C++ layer at the boundary and record vendor-upgrade and Blueprint-reference migration costs.
