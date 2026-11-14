# Data-Driven Design: DataAsset, PrimaryDataAsset, DataTable

## Separate Three Data Categories

- **Definition / static configuration:** base weapon damage, item weight, Icon, Niagara, monster defaults. DataAsset/DataTable may supply these; treat as read-only during play.
- **Runtime state:** ammo, durability, current health, inventory quantity, quest progress. Owned by runtime Actor/Component/UObject/UStruct instances, not shared Definition assets.
- **Save data:** stable IDs and necessary mutable state for reconstruction, not pointers to transient Actors.

For example: `ItemDefinition` + `FInventoryEntry` (ItemId, Quantity, Durability) + serializable `FSaveInventoryEntry`.

## DataAsset vs DataTable

| Scenario | Choice | Consideration |
| --- | --- | --- |
| Individual definitions with several asset references, edited separately | `UDataAsset` | Individual Content Browser editing and rich definitions |
| Asset Manager identity, bundles, on-demand load/unload | `UPrimaryDataAsset` | PrimaryAssetId; configure manager and loading policy as needed |
| Hundreds of uniform rows, CSV and centralized balancing | `UDataTable` + `FTableRowBase` | Fixed schema and bulk tuning |
| Few simple parameters | `UPROPERTY(EditDefaultsOnly)` / `USTRUCT` | No need for a new asset/table for every parameter |
| Many complex definitions and balance values | Mix DataAsset + DataTable | Define authoritative fields and references; avoid duplicate values |

Do not mandate one format for all weapons or items. Decide from content volume, editing, dependency management, and load performance.

## DataAsset

- In a C++ project define a `UDataAsset` / `UPrimaryDataAsset` subclass and expose fields via `UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category=...)`, adding `meta=(ClampMin=..., UIMin=...)` as needed. In Blueprint-only projects create a Blueprint Class derived from `PrimaryDataAsset`, then create Data Assets from it (`UDataAsset` itself cannot be a Blueprint parent).
- Treat definitions as immutable shared configuration; keep dynamic state in components or instance structs.
- For on-demand streaming consider `TSoftObjectPtr` / `TSoftClassPtr`, Asset Manager, asynchronous loading, and missing-asset handling. PrimaryDataAsset does not automatically imply asynchronous loading.
- `UPrimaryDataAsset` supports PrimaryAssetId and Asset Bundles for more complex asset management. Do not force replacement of all `UDataAsset`s in a simple project.
- Consider Data Only Blueprint for inherited configuration rather than manually copying values and allowing drift.

## DataTable

- Typically derive the row `USTRUCT(BlueprintType)` from `FTableRowBase`. Keep column types stable. CSV import uses the first column as RowName by default regardless of header; if `UDataTable::ImportKeyField` is set, it uses the column with that header. Access using `FDataTableRowHandle` / `FindRow<T>()`.
- `FindRow` may fail. Distinguish invalid editor data (development `ensureMsgf` and diagnostics) from recoverable runtime absence (safe fallback and logging).
- Do not cache `FindRow()` pointers long-term: reimport can change row data. Copy needed values or look up again via RowHandle.
- Hard references in a table can load assets together. Consider soft references for large assets and design first-load and placeholder behavior even for soft references.
- After creation/reimport verify unique row names, defaults, migration compatibility, null asset references, and target-platform Cook.

```cpp
USTRUCT(BlueprintType)
struct FItemBalanceRow : public FTableRowBase
{
    GENERATED_BODY()

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Balance")
    float Weight = 0.0f;

    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Balance", meta=(ClampMin="1"))
    int32 MaxStackSize = 1;
};
```

This illustrates a DataTable row schema; a real project needs headers, module dependencies, reading API, data validation, and caller error handling before it can compile and run.

- Item: `DT_ItemBalance` may hold bulk-tuned homogeneous numbers; `DA_ItemDefinition` may hold per-item Mesh, Icon, Description, Ability, etc. Each field has one source of truth.
- Weapon: `DA_WeaponDefinition` describes parameters/assets; `UWeaponComponent` owns current ammo, state, and cooldown.
- Loot Table: DataTable holds weights/quantities, Spawner performs rolls, and saves record only required persistent results rather than treating rows as runtime records.

Official references, verify version: [Data Assets](https://dev.epicgames.com/documentation/unreal-engine/data-assets-in-unreal-engine), [Data-Driven Gameplay Elements](https://dev.epicgames.com/documentation/unreal-engine/data-driven-gameplay-elements-in-unreal-engine), [Asset Management](https://dev.epicgames.com/documentation/unreal-engine/asset-management-in-unreal-engine).
