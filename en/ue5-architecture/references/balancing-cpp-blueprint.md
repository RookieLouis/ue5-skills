# Balancing C++ and Blueprint

This guide draws on Epic's UE 4.27 Architecture Notes, UE5 documentation, and project conventions. It is not a verbatim copy of Epic's guidance; verify recommendations against the project's actual engine version.

The goal is stable behavioral contracts and authoritative state in C++, with content assembly, configuration, and suitable visual/presentation extensions in Blueprint. This does not prohibit Blueprint logic; it avoids scattering critical Gameplay rules beyond testing. “Authority” means the source of truth defined in [SKILL.md](../SKILL.md), not network Authority.

## Division of Responsibilities

| Concern | Preferred location | Notes |
| --- | --- | --- |
| Health calculations, stacking, trade validity, damage | C++ | Authoritative, reproducibly testable Gameplay logic |
| Input dispatch to Gameplay commands | C++ or concise Blueprint | C++ for complexity; BP for content experiments; UI should not own core rules |
| Item/Weapon/Enemy configuration | DataAsset / DataTable / Editor | Designers tune without editing code |
| Mesh, Animation, Montage, Niagara, Sound, Widget Class assignments | Blueprint Class / Editor | Visual asset configuration |
| Camera Shake, hit flash, audio, VFX, non-authoritative presentation | Blueprint / AnimBP / Niagara | Can be triggered by C++ events; do not block core flow |
| AI states/behavior | BT/StateTree/Blueprint and C++ | Choose maintainability; authoritative state and key algorithms may live in C++ |
| UI layout/animation | UMG / Blueprint | Drive from ViewModel/Gameplay state; no second source of truth |
| Save/Load, stable IDs, validation | C++ | Separate from runtime state and definitions |

For Blueprint-only projects, “C++” in the table means an authoritative BP ActorComponent; stateless computations may use a Blueprint Function Library. Keep one authoritative state and do not let UI decide rules. See step 1 of SKILL.md before adding a C++ module.

## Exposing C++

- Prefer `EditDefaultsOnly` for configurable definitions; use `EditInstanceOnly` / `EditAnywhere` when level instances need tuning.
- Prefer `VisibleInstanceOnly`, `BlueprintReadOnly`, or private getters for core state; do not make everything `BlueprintReadWrite` for convenience.
- Expose commands/controlled APIs with `UFUNCTION(BlueprintCallable)` and side-effect-free queries with `BlueprintPure`; never mark state-mutating methods Pure.
- `BlueprintNativeEvent` has a default implementation; `BlueprintImplementableEvent` is a BP hook. Specify execution paths and whether the parent implementation is called.
- Use a `BlueprintAssignable` dynamic multicast delegate for state BP must observe; native delegates suffice for C++ consumers.
- Set Category, DisplayName/ToolTip, bounds, and asset filters as applicable; provide safe defaults and validation for designer-facing settings.
- Allow core-rule overrides cautiously: keep invariants and final commits in C++ when practical, with explicit hooks or configurable parameters for Blueprint.

## Respect Existing Blueprints and Assets

- Before changing C++ types, UPROPERTY/UFUNCTION, class/path/Component names, or Blueprint parents, inspect references and plan redirects, migration, and regression testing.
- Do not claim to understand an entire Blueprint graph from C++ alone.
- Migrate gradually: extract authoritative state and minimal public API, replace calls, then validate BP behavior and Editor configuration; see [bp-to-cpp-migration.md](bp-to-cpp-migration.md).
- Isolate Marketplace/Fab plugins with adapters, separate Components, inheritance, or supported extension points. Assess upgrade and license costs before editing vendor source.

## Architecture Notes: Type Dependencies and Asset Loading

The following combines Epic's [Architecture Notes (UE 4.27)](https://dev.epicgames.com/documentation/unreal-engine/balancing-blueprint-and-cplusplus?application_version=4.27#architecturenotes) with UE5 asset-management documentation as project decision rules; apply according to actual dependencies and content scale.

1. **Lightweight shared contracts:** Put cross-system properties/functions in C++ base types, lightweight BP bases, or interfaces. Casting to a concrete BP, or using it as a variable/parameter type, can make callers depend on the BP and its hard-referenced content. The concern is the dependency chain, not an assertion that the cast operation itself is expensive. Lightweight bases should avoid referencing concrete content subclasses.
2. **Reduce BP cycles:** Inspect cycles from types, defaults, and asset references. Clarify dependency direction through responsibilities, base types, interfaces, or events to reduce loading/compilation coupling. An Interface call alone does not eliminate hard references in parameters or defaults. Manage C++ module/type dependencies too.
3. **Separate asset configuration from C++ behavior:** Expose configurable properties in generic base classes and assign content in BP defaults or DataAssets. Be cautious about hardcoded optional paths using constructor `ConstructorHelpers::FObjectFinder` / `FClassFinder`: class/CDO construction may synchronously load them and renames become harder. A hard reference moved to BP/DataAsset still loads with its owner. Small essential resident assets may use hard references. See [Referencing Assets](https://dev.epicgames.com/documentation/en-us/unreal-engine/referencing-assets-in-unreal-engine).
4. **Trace references and verify Cook:** Use `TSoftObjectPtr` for on-demand assets, `TSoftClassPtr` for classes, and `FSoftObjectPath` for a path value; prefer properties discoverable by Cook. Do not rely solely on dynamically constructed strings and `LoadObject` for discovery. Soft references do not cover every Cook scenario: verify Asset Manager scanning and Cook rules for configuration-driven, runtime-generated, or dynamically registered content, and test loading in packaged builds. BP-only projects use Soft Object Reference / Soft Class Reference variables. See [Asset Management](https://dev.epicgames.com/documentation/en-us/unreal-engine/asset-management-in-unreal-engine).
5. **Prefer native stable shared types:** In existing C++ projects, prefer `USTRUCT(BlueprintType)` / `UENUM(BlueprintType)` for types crossing C++/BP boundaries, key saves, or long-lived public contracts. BP UserDefinedStruct/Enum can serve local prototypes but cannot directly serve as normal static C++ types. Do not decide migration mechanically by counting BP users. Verify asset defaults, DataTables, and old saves before replacing; see [bp-to-cpp-migration.md](bp-to-cpp-migration.md).
6. **Define async completion and lifetime:** Soft references delay loading; `LoadSynchronous` still blocks. Use Asset Manager / `FStreamableManager::RequestAsyncLoad` for large optional meshes, sounds, enemy classes, etc., and define behavior before completion, failure handling, and what happens if the requester becomes invalid. Retain an asset reference or valid loading handle for its required lifetime. Blueprint provides `Async Load Asset` / `Async Load Class Asset` / `Async Load Assets`; `Load Asset Blocking` / `Load Class Asset Blocking` are synchronous. A few simple assets do not justify a custom Manager. See [Asynchronous Asset Loading](https://dev.epicgames.com/documentation/en-us/unreal-engine/asynchronous-asset-loading-in-unreal-engine).

Use [Reference Viewer / Size Map](https://dev.epicgames.com/documentation/en-us/unreal-engine/reference-viewer-in-unreal-engine) to verify hard/soft dependency chains and asset size before deciding on decoupling or on-demand loading; do not infer cost from graph appearance alone.

## Designer Acceptance

1. Can a new item/enemy be added by creating DataAsset/Blueprint configuration without editing C++?
2. Is there a clear Editor entry for changing an Icon, VFX, or value?
3. Can a designer accidentally alter authoritative runtime state or create inconsistency?
4. Do missing assets, invalid RowNames, and out-of-range parameters produce clear, traceable errors?
5. Do old Blueprints and level instances still load correctly after changes?

See sections A and D of [review-checklist.md](review-checklist.md) for dependency-chain and soft-loading review. Avoid both extremes: “Blueprint can hold art only and no logic,” which harms iteration, and “C++ only supplies Component shells while Blueprint contains all logic,” which harms system maintenance, review, and deterministic testing.
