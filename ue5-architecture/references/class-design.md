# Inheritance、Composition 与 UE 对象选型

## 选型规则

| 问题 | 优先方案 | 何时不选 |
|---|---|---|
| 这个类型确实是某基类的一种（is-a）并遵守相同生命周期/行为契约？ | `Inheritance` | 只是想复用几行代码，或不同子类行为大幅分叉 |
| 一项功能要被多个 Actor 组合复用（has-a）？ | `UActorComponent` | 功能不属于 Actor、无需 Actor/World 生命周期 |
| 单纯逻辑/Runtime Data，需要反射和 GC？ | `UObject` 或 `USTRUCT` | 本身需要 Transform、Collision、Actor 行为 |
| 游戏世界中的实体，需要 Transform、Collision、Replication 等 Actor 能力？ | `AActor` / `APawn` / `ACharacter` | 纯数据/算法不应创建 Actor |
| 共享服务且有明确作用域？ | `UGameInstanceSubsystem` / `UWorldSubsystem` 等合适 Subsystem | 仅服务一个 Character 或生命周期与 Subsystem 不一致 |
| 不同类都具备统一能力？ | 小型 `UInterface` | 目标永远是已知的自有 Component，直接调用更好 |

## Inheritance（继承）

- 用于真正稳定的类型层次，例如 `ACharacter` 衍生具体角色，`AActor` 衍生 PickupActor。
- 公共基类只放真正共享的**不变量、行为契约和生命周期逻辑**。不要为了复用配置字段创建 5 层继承。
- 重写 `virtual` / `BlueprintNativeEvent` 时明确父类契约；涉及 `BeginPlay`、`EndPlay`、`Tick` 等 Engine 生命周期时按需求正确调用 `Super`。
- 关注 LSP：父类型的使用方不应依赖判断哪一个具体子类才不会出错。
- 可变化行为优先考虑组合、DataAsset、Strategy；不要把每一种武器/敌人属性都做一个 C++ 子类。

## Composition（组合 / ActorComponent）

- 适用于可复用的 Health、Inventory、Interaction、Equipment 能力。
- Component 只管理自己应有的状态。`UInventoryComponent` 可发出库存变化事件，但不要主动寻找 HUD 并操纵 UI。
- Actor 负责配置自有 Component 和协调明确的一对一操作；多个无关 Actor 间的复杂流程再考虑 Coordinator。
- Component 之间如果稳定地共同依附一个 Actor，允许 Owner 获取并缓存对应 Component；跨系统依赖增大时再切 Interface/Coordinator。
- 不要强制把数据结构、无状态算法都做成 ActorComponent；可能 `USTRUCT`、普通 C++ 函数或 `UObject` 更合适。

## 选择示例

- Player Character + InventoryComponent + InteractionComponent：组合优先。
- `AWorldPickupActor` 派生 `AActor`：继承是正常 UE 类型系统用法。
- 若两种敌人仅伤害/移动速度不同：优先参数或 DataAsset，而不是马上创建两个 C++ 子类。
- Boss 有显著独立生命周期和 AI 流程：专用子类可能更简单。

## 反过度设计

先问：`Actor + 1~3 个 Component + 少量直接引用` 是否足以清晰实现？只有看见真实职责冲突、复用需求或扩展变体后，才增加新抽象层。
