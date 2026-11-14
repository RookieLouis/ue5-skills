# Blueprint → C++ Migration

Use for gradual migration of authoritative logic and data types in existing Blueprint systems, including third-party BP frameworks. Requires an existing C++ module; otherwise obtain confirmation as in step 1 of [SKILL.md](../SKILL.md).

## Principles

- Preserve assets and change their parent: create a C++ base and reparent the BP rather than deleting/recreating it, preserving level instances, DataAsset references, and BP subclasses.
- Migrate one unit at a time, e.g. one Component or a group of structs/enums. Each step must compile, run in PIE, and be independently reversible.
- Migrate depended-on types first: C++ cannot use UserDefinedStruct / UserDefinedEnum as static C++ types. Migrate structs/enums, then Components, then higher-level types such as Character or Ability.
- Preserve Blueprint-facing function names, parameters, and Event Dispatcher names; use CoreRedirects if renaming is necessary.

## Steps

1. Use Reference Viewer to list all referencers; record calls, variable reads/writes, and event bindings.
2. Write the C++ base/type, exposing `UPROPERTY` / `UFUNCTION` as in [balancing-cpp-blueprint.md](balancing-cpp-blueprint.md).
3. Reparent the BP, replace BP variables/functions one by one with C++ members, then remove old BP definitions.
4. Open and compile all referencers and repair broken nodes.
5. Run PIE regression for relevant core use cases and save loading.

## Risks

- Same-name C++ parent properties/functions conflict with existing BP child members. Plan names and inspect references after replacement.
- Replacing UserDefinedStruct / UserDefinedEnum with native types does not automatically map saved BP, DataTable, or DataAsset values and may lose defaults/configuration. Check a small set first and commit version control before bulk replacement.
- For renamed or moved classes, properties, functions, structs, or enums, add appropriate `[CoreRedirects]` entries in `Config/DefaultEngine.ini`, resave referenced assets, and confirm effectiveness.
- Directly changing vendor BPs complicates upstream updates; record affected assets. Verify old saves if SaveGame types or fields change.

## Project Experience (UE 5.8; Project Convention, Not Epic Text)

Observed in this project's Attributes migration phases 1 and 2; recheck after engine upgrades.

- UserDefinedStruct → USTRUCT: `StructRedirects` did not work; `ObjectRedirects` did. Old member names had GUID suffixes; map them with `PropertyRedirects` for struct node pins to match.
- UserDefinedEnum → UENUM: use `EnumRedirects` + `ValueChanges`; order native values by the old enum's numeric values, not display order.
- Match native defaults to the old UDS; use C++ `double` for old real/double members.
- Do not rely on Live Coding for new reflected types: close the Editor, build fully, and restart.
- If a BP Component is reparented to a native intermediate parent that does not Tick, add `meta=(ChildCanTick)` to its UCLASS or BP child Event Tick is disabled.
- Delete old UDS offline with Editor closed; Python deletion may crash in 5.8.
- Before/after snapshots should cover BP variables/defaults, AnimNotify instances in montages, level Actor instances, and Widget tree controls.

## Acceptance

All referencers compile without errors; the same PIE cases behave consistently before and after; this migration step can be independently rolled back in version control.
