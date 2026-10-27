# 单人撤离游戏示例：拾取 → 库存 → UI → 存档

此示例展示 Skill 如何在 UE5 组合设计中同时使用 SOLID、SoC、Loose Coupling、Observer、Data-Driven、Designer Convenience、Composition，以及在确有必要时使用 Mediator。**它不是强制项目采用的唯一架构。**

## 需求

线性关卡中的玩家可从世界拾取物品；按堆叠与重量限制加入 Inventory；HUD 更新；撤离或存档时保留库存。不同物品由 Designer 扩展，支持少量外观与数据差异。

## 推荐角色划分

| 类型                             | 职责                                       | 不该负责                         |
| ------------------------------ | ---------------------------------------- | ---------------------------- |
| `APlayerCharacter`             | 拥有/配置自有 Component，接受输入，必要时编排简单拾取         | 物品表定义、HUD 构建细节、所有 Save 序列化规则 |
| `UInteractionComponent`        | Trace/Overlap 候选检测、关注目标、发出交互意图           | 修改库存、刷新 UI、直接 Destroy 任意目标   |
| `IInteractable`（可选）            | 对 Chest、NPC、Pickup 等提供统一交互契约             | 通用消息总线                       |
| `AWorldPickup`                 | 世界中的拾取实体、持有定义引用与待拾取数量                    | 作为玩家库存的永久权威状态                |
| `UInventoryComponent`          | 拥有库存 Runtime State；校验容量、堆叠，返回操作结果；广播状态变更 | 查找 Widget、播放具体 UI 动画         |
| `UItemDefinition` / `FItemRow` | 静态 Item 数据，例如重量、堆叠上限、Icon                | 玩家当前数量/耐久/可变状态               |
| `UInventoryWidget` / ViewModel | 根据库存状态刷新显示、处理表现交互                        | 擅自决定是否成功添加物品                 |
| `USaveGame` + Save 服务          | 保存持久 Item ID、数量和必要状态；加载时重建               | 对关卡临时 Actor 指针直接持久化          |

## 数据流（简单实现）

```
Input: Interact
      |
      v
Player / InteractionComponent
      |  对不同对象调用 IInteractable（需要时）
      v
AWorldPickup（读取 ItemDefinition + 当前待拾取数量）
      |  由 Character 或交互实现发起 TryAddItem（同步返回结果）
      v
InventoryComponent（规则权威：合法性、容量、堆叠、提交状态）
      |                 |
      | success         `-- OnInventoryChanged (Delegate) --> UI 刷新
      v
更新 AWorldPickup 的剩余数量；为零则移除世界实体

存档时：Save System <- Inventory Runtime State -> 稳定 ItemId + Count ...
```

说明：真实实现需要避免“已添加库存但世界对象删除失败”造成重复拾取。单机游戏可把交互过程作为一个明确同步流程，验证成功后再提交世界更新；若需复杂事务、掉落拆分、Quest 计数、失败回滚，则明确统一提交次序并引入 Coordinator。

## 何时加入 Mediator

不需要：只有 `TryAddItem` + 更新 Pickup 状态这两步，Character 或 Pickup 与 Inventory 的清晰调用就够了。

可以加入：拾取需要协调 `Inventory`、`Quest`、`Audio`、`Loot Ownership`、交易规则、失败回滚；选一个 **局部 `LootCoordinator`**，只负责编排，库存容量规则仍由 InventoryComponent 持有。

## Blueprint / Data 位置

- C++：`InventoryComponent::TryAddItem`、重量/堆叠计算、合法性验证、Save 转换、状态事件。
- Blueprint：`BP_WorldPickup` 配置 StaticMesh、Interact Widget、拾取 VFX/SFX；`WBP_Inventory` 的布局与刷新动画。
- DataAsset：每个物品 Icon、Mesh、稀有度表现、Description 和特殊关联的 Soft Asset。
- DataTable：可选集中管理大量物品的 Weight、MaxStack、售卖数值（保持字段权威唯一）。如果只是几十个物品并且只有少量 Designer，不必同时建立两套定义。
- Runtime Inventory：例如 `FInventoryEntry{ ItemId, Quantity, Durability }`，不能写回共享 Definition Asset。

## Object Pooling 的应用判定

- 地面物品很少且长时间存在：**通常不需要 Pool**。
- 大量短寿命 Projectile：先用 Unreal Insights 量化 Spawn/Destroy 成本；若频繁且状态可彻底重置，再选择 Pool。
- Niagara：先评估 Niagara 自带池化，而不是直接生成自定义 Niagara Pool Manager。

## 验收案例

1. 空库存拾取成功，存档后重新加载数量正确。
2. 库存满时 `TryAddItem` 返回失败，世界拾取物保持不变，UI 不错误刷新。
3. 部分拾取时世界剩余数量正确；重复按键不会重复提交。
4. Designer 创建新 DataAsset 和 `BP_WorldPickup`，不修改 C++ 即可生成另一种物品。
5. UI Widget 销毁重建后事件订阅无重复/无悬空回调。
6. 若使用 DataTable，故意使用错误 RowName 时有清晰诊断与安全行为。
