# Single-Player Extraction Example: Pickup → Inventory → UI → Save

This illustrates SOLID, separation of concerns, loose coupling, Observer, data-driven design, designer convenience, composition, and Mediator only when necessary. It is not the only required architecture.

## Requirements and Responsibilities

In a linear level, the player picks up world items subject to stacking and weight limits, the HUD updates, and inventory persists on extraction/save. Designers can add items with modest visual/data differences.

| Type | Responsibility | Not responsible for |
| --- | --- | --- |
| `APlayerCharacter` | Own/configure Components, input, simple pickup coordination | Item table definitions, HUD creation details, all save serialization |
| `UInteractionComponent` | Trace/Overlap candidates, focus, interaction intent | Mutating inventory, refreshing UI, destroying arbitrary targets |
| `IInteractable` (optional) | Shared interaction contract for Chest/NPC/Pickup | General message bus |
| `AWorldPickup` | World entity, definition reference and remaining amount | Permanent authority for player inventory |
| `UInventoryComponent` | Runtime inventory, validate capacity/stacking, return results, broadcast changes | Finding Widgets or playing specific UI animations |
| `UItemDefinition` / `FItemRow` | Static item data such as weight, stack limit, Icon | Current quantity, durability, mutable state |
| `UInventoryWidget` / ViewModel | Render state, handle presentation interaction | Deciding whether an item was added successfully |
| `USaveGame` + Save service | Persist stable Item IDs, counts, necessary state; reconstruct on load | Persisting transient level Actor pointers directly |

## Simple Flow

Input → Player / InteractionComponent → optionally `IInteractable` → `AWorldPickup` reads definition and remaining count → Character or interaction implementation calls `TryAddItem` synchronously → Inventory validates and commits → on success update remaining Pickup count and remove it at zero. After a state change Inventory broadcasts `OnInventoryChanged` for UI refresh. Save converts inventory runtime state to stable ItemId, Count, and necessary state.

Prevent duplicate pickups if inventory is updated but world removal fails. In single-player, use a clear synchronous flow and commit the world update after successful validation. For complex transactions, split drops, Quest counts, or rollback, explicitly define commit order and introduce a Coordinator.

## Mediator and Data Placement

For only `TryAddItem` plus updating the pickup, Character or Pickup can call Inventory directly. If Inventory, Quest, Audio, Loot Ownership, trade rules, and rollback need coordination, use a local `LootCoordinator` for orchestration while InventoryComponent retains capacity rules.

- C++: `InventoryComponent::TryAddItem`, weight/stacking, validation, save conversion, state events.
- Blueprint: `BP_WorldPickup` configures StaticMesh, Interact Widget, VFX/SFX; `WBP_Inventory` handles layout and refresh animation.
- DataAsset: item Icon, Mesh, rarity presentation, Description, and related soft assets.
- DataTable: optional centralized Weight, MaxStack and sale values with one authoritative source per field. A few dozen items and few designers do not require both systems.
- Runtime Inventory: e.g. `FInventoryEntry{ ItemId, Quantity, Durability }`; never write back to the shared Definition.

## Pooling and Acceptance

Rare long-lived ground items usually need no pool. Profile short-lived projectiles with Unreal Insights first; pool only if frequent and safely resettable. Evaluate Niagara's built-in pooling before a custom manager.

1. Pickup into an empty inventory, save, reload, and check quantity.
2. A full inventory returns failure, leaves the world item intact, and does not refresh UI incorrectly.
3. Partial pickup leaves the correct world quantity; repeated input does not duplicate commits.
4. A designer can create a new DataAsset and `BP_WorldPickup` without modifying C++.
5. Widget destruction/recreation leaves neither duplicate subscriptions nor dangling callbacks.
6. If using DataTable, an invalid RowName produces clear diagnostics and safe behavior.
