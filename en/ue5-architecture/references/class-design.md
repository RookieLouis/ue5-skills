# Inheritance, Composition, and UE Object Selection

## Selection

| Question | Prefer | Do not choose when |
| --- | --- | --- |
| Is this truly a kind of the base type with the same lifecycle and behavioral contract? | Inheritance | Reusing a few lines or allowing subclasses to diverge greatly. |
| Must multiple Actors compose and reuse a capability? | `UActorComponent` | It does not belong to an Actor and needs no Actor/World lifecycle. |
| Is it logic/runtime data needing reflection and GC? | `UObject` or `USTRUCT` | It needs Transform, Collision, or Actor behavior. |
| Is it a world entity needing Transform, Collision, Replication, etc.? | `AActor` / `APawn` / `ACharacter` | It is pure data or an algorithm. |
| Is it a shared service with a clear scope? | An appropriate Subsystem, e.g. `UGameInstanceSubsystem` / `UWorldSubsystem` | It serves just one Character or its lifetime differs from the Subsystem. |
| Do different classes share one capability? | A small `UInterface` | The target is always a known owned Component; direct calls are clearer. |

## Inheritance

- Use for stable type hierarchies, such as a specific character derived from `ACharacter` or PickupActor from `AActor`.
- Put only genuinely shared invariants, behavioral contracts, and lifecycle logic in a base class. Do not create five inheritance levels to reuse configuration fields.
- When overriding `virtual` / `BlueprintNativeEvent`, make the parent contract explicit. Call `Super` appropriately for engine lifecycle methods such as `BeginPlay`, `EndPlay`, and `Tick`.
- Respect LSP: callers of a base type should not need to inspect the concrete subtype to avoid failure.
- Consider composition, DataAsset, or Strategy for varying behavior; do not create a C++ subclass for every weapon/enemy property variation.

## Composition / ActorComponent

- Suitable for reusable Health, Inventory, Interaction, and Equipment capabilities.
- A Component owns only its own state. `UInventoryComponent` may emit inventory change events but should not find the HUD and manipulate UI.
- The Actor configures owned Components and coordinates clear one-to-one operations. Consider a coordinator for complex flows between unrelated Actors.
- Components consistently attached to one Actor may be obtained and cached through the Owner. Consider Interface/Coordinator when cross-system dependencies grow.
- Do not force data structures or stateless algorithms into ActorComponents; `USTRUCT`, ordinary C++ functions, or `UObject` may be more appropriate.

## Examples and Restraint

- Player Character + InventoryComponent + InteractionComponent: prefer composition.
- `AWorldPickupActor` deriving from `AActor`: normal use of UE inheritance.
- Enemies differing only in damage and speed: prefer parameters or a DataAsset to two C++ subclasses.
- A boss with a distinct lifecycle and AI flow may be simpler as a dedicated subclass.

First ask whether `Actor + 1–3 Components + a few direct references` clearly solves the problem. Add abstraction only for an observed responsibility conflict, reuse need, or variant.
