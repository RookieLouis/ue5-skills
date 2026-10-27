# Data-Driven Design：DataAsset、PrimaryDataAsset、DataTable

## 先区分三类数据

- **Definition / Static Configuration**：武器基础伤害、物品重量、Icon、Niagara、怪物基础配置。来源可为 DataAsset/DataTable；游戏过程原则上只读。
- **Runtime State**：弹药、耐久、当前血量、库存数量、任务进度。由 Actor/Component/UObject/UStruct 等运行时实例拥有，不放入共享 Definition Asset。
- **Save Data**：用于重建进度的稳定 ID + 必要可变状态；不要直接把指向临时 Actor 的指针当作可持久化数据。

典型结构：`ItemDefinition`（基础定义） + `FInventoryEntry`（ItemId、Quantity、Durability 等 Runtime） + `FSaveInventoryEntry`（可序列化状态）。

## DataAsset vs DataTable 决策

| 场景                                      | 推荐                                        | 理由 / 注意                                    |
| --------------------------------------- | ----------------------------------------- | ------------------------------------------ |
| 每件物品有独立配置与多种 Asset 引用，Designer 经常单独编辑   | `UDataAsset`                              | 内容浏览器中逐项编辑，适合复杂定义与资源引用                     |
| 需要 AssetManager 标识、Asset Bundle、按需加载与卸载 | `UPrimaryDataAsset`                       | 有 PrimaryAssetId；仍须按需配置 AssetManager 和加载策略 |
| 数百行结构统一，需要 CSV 导入、集中平衡                  | `UDataTable` + `FTableRowBase`            | 列结构固定，易做批量数值调整                             |
| 少量简单参数                                  | `UPROPERTY(EditDefaultsOnly)` / `USTRUCT` | 没必要为每个小参数新建表或 Asset                        |
| 大量复杂定义 + 大量平衡数值                         | 允许混合 DataAsset + DataTable                | 仅在明确划分权威字段与引用关系时使用，避免两处重复同一参数              |

**不要设定所有武器必须用 DataAsset 或所有物品必须用 DataTable 的硬规则。** 依据内容量、编辑方式、依赖管理和加载性能决策。

## DataAsset 规则

- 有 C++ 模块时，在 C++ 中定义 `UDataAsset` / `UPrimaryDataAsset` 子类；用 `UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category=...)` 等暴露内容编辑字段，必要时加 `meta=(ClampMin=..., UIMin=...)`。纯 Blueprint 项目创建继承 `PrimaryDataAsset` 的 Blueprint Class，再用它创建 Data Asset（`UDataAsset` 本身不能作为 Blueprint 父类）。

- 定义应默认当作不可变共享配置；动态状态另存在组件或实例结构中。

- 需要按需流送时，考虑 `TSoftObjectPtr` / `TSoftClassPtr`、AssetManager、异步加载与缺失资产处理；不要假设“使用 PrimaryDataAsset 就自动异步加载”。

- `UPrimaryDataAsset` 支持 PrimaryAssetId 和 Asset Bundle，适用于复杂 Asset 管理；简单项目不要无故强制替换所有 `UDataAsset`。

- 父子配置继承有需求时，评估 Data Only Blueprint，避免靠手动复制数值造成漂移。

## DataTable 规则

- 行结构通常为 `USTRUCT(BlueprintType) struct FItemRow : public FTableRowBase { GENERATED_BODY() ... };`。
- 每行列类型必须稳定；CSV 导入时默认第一列作为 RowName（与表头文字无关），设置了 `UDataTable::ImportKeyField` 时改用表头与之同名的列；通过 `FDataTableRowHandle` / `FindRow<T>()` 访问。
- `FindRow` 可能失败：区分非法编辑数据（开发期 `ensureMsgf` + 诊断）与运行时可恢复缺失（安全回退与日志）。
- **不要长期缓存 `FindRow()` 返回的指针**：重新导入会改变行数据；若需长期使用，按规则复制必要值或通过 RowHandle 再查询。
- DataTable 中的硬引用可能造成资源一并加载；大资源引用优先评估 Soft Reference；即使是 Soft Reference 也要设计首次加载和占位表现。
- 编辑器创建/重导表后要验证行名唯一、字段默认值、迁移兼容性、空资源引用及目标平台 Cook。

## 推荐的 C++ 类型示意（按项目模块名调整）

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

上面的例子展示的是 **DataTable Row Schema**；真实项目必须补齐所需 Header、模块依赖、读表 API、Data validation 和调用方的错误处理后才能编译运行。

## 游戏示例

- Pickup / Item：大量同构数值若需要批量调平衡，可存 `DT_ItemBalance`；独立 Mesh、Icon、Description、Ability 等丰富内容可存 `DA_ItemDefinition`。但同一字段应有唯一的 Source of Truth。
- 武器：`DA_WeaponDefinition` 描述可编辑参数和 Asset；`UWeaponComponent` 维护当前弹药、状态、CD。
- Loot Table：可用 DataTable 管理权重/数量，Spawner 负责 Roll；存档只记录所需的持久化结果，不把 DataTable 行当运行时记录。

## 官方参考（核实适用版本）

- https://dev.epicgames.com/documentation/unreal-engine/data-assets-in-unreal-engine
- https://dev.epicgames.com/documentation/unreal-engine/data-driven-gameplay-elements-in-unreal-engine
- https://dev.epicgames.com/documentation/unreal-engine/asset-management-in-unreal-engine
