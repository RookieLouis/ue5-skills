# System Communication: Direct Reference, Interface, Observer, Mediator

## Selection

| Need | Choice | Reason |
| --- | --- | --- |
| Player owns InventoryComponent and calls `TryAddItem` | Direct reference | One target, synchronous result, easy to debug |
| Interact with Chest, NPC, Pickup of different types | `UINTERFACE` / `I...` | Common contract without a common inheritance tree |
| UI or sound responds to inventory changes | Delegate / Observer | Broadcast a completed change without knowing listeners |
| Pickup coordinates Interaction, Inventory, Quest, removal and failure compensation | Mediator / Coordinator as needed | Centralize orchestration without mutual dependencies |
| One target returns success, failure, or remainder | Synchronous command or interface function | Broadcast is not a transaction API and promises neither order nor success |
| Existing GameplayTag / State Machine / AnimNotify flow | Retain and review existing mechanism | Correct unclear routing or misplaced authority rather than introducing another mechanism |

## Observer

- Prefer `DECLARE_MULTICAST_DELEGATE...` for C++ only, or `DECLARE_DYNAMIC_MULTICAST_DELEGATE...` plus `UPROPERTY(BlueprintAssignable)` for Blueprint subscribers. Dynamic delegates have reflection overhead; choose for actual need.
- Name past-tense/state-change events such as `OnInventoryChanged` and `OnHealthChanged`. The state owner emits them after successful commit.
- Listeners/UI should not mutate the sender's internal data and cause recursive loops.
- Match `AddDynamic`, `AddUObject`, `AddWeakLambda`, etc. to lifetimes; manage `FDelegateHandle`, unbinding and duplicate binding. Verify BeginPlay/EndPlay, Widget recreation, and pool reactivation.
- Delegates do not guarantee business ordering among observers. Execute core actions synchronously before broadcasting results. Avoid Tick polling for event-driven state, while considering frequency/batching for high-frequency updates.

## Mediator

Put orchestration of a multi-system use case at a clear coordination point; do not create a global Manager that knows everything.

- Example: validate item → check inventory capacity → commit item → notify quest → remove world item, including intermediate failure.
- Prefer an existing `PlayerController`, `Character`, or appropriately scoped `ULootCoordinatorComponent`. Call small participant command APIs and identify state owners, committers, and rollback behavior.
- Use one coordinator per use case/bounded context; do not create one without actual orchestration complexity. Do not replace typed interfaces with a global string-topic Event Bus or duplicate subsystem rules inside the mediator.

## GameplayTag / State Machine / AnimNotify

- GameplayTags are hierarchical and Editor-validated, not automatically a string bus. An event with explicit sender and recipient, e.g. activating a tagged Ability on one's own component, is normal; global broadcast with unknown recipients is the Event Bus antipattern.
- Do not rely on events for synchronous results such as Ability activation or damage success: return the result or reflect it in authoritative state.
- AnimNotify / AnimNotifyState only send triggers. A Component or Ability owns hit-detection, invulnerability, or input-buffer window state; Notify opens/closes it.
- The state owner must also close windows and clear hit records on Montage interruption or early Ability end, without relying solely on NotifyEnd.
- The State Machine owner changes state; other systems request transitions via events instead of directly rewriting it.

## Loot → Inventory → UI

1. `InteractionComponent` identifies a target and requests `IInteractable` behavior.
2. Character or local `LootCoordinator` calls `InventoryComponent::TryAddItem(...)` and receives a synchronous result.
3. Only on success update/remove the World Pickup; Inventory broadcasts `OnInventoryChanged` after its state changes.
4. UI subscribes to refresh; sound/VFX respond to suitable Gameplay or presentation events. InventoryComponent does not hard-reference a concrete Widget.
5. For simple pickup, Character can call inventory directly without a Coordinator.

Avoid Inventory casting to a concrete HUD Widget, wrapping every function in an Interface, broadcasting “try add item” and hoping a listener commits it, or building a globally accessible GenericMediator God Object.
