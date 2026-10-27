# 系统通信：Direct Reference、Interface、Observer、Mediator

## 通信选型表

| 需求                                                  | 推荐方式                                                    | 关键理由                        |
| --------------------------------------------------- | ------------------------------------------------------- | --------------------------- |
| Player 明确持有 InventoryComponent，需要执行 `TryAddItem`    | Direct Reference                                        | 目标唯一、同步结果可见，最易调试            |
| 玩家可和 Chest、NPC、Pickup 等不同类型对象交互                     | Unreal Interface (`UINTERFACE` / `I...`)                | 统一行为契约，不要求相同继承树             |
| Inventory 状态变化后 UI、音效等自行响应                          | Delegate / Observer                                     | 广播“已发生”状态变化，发送者不认识监听者       |
| 拾取需协调 Interaction、Inventory、Quest、世界道具移除，存在顺序/失败补偿  | Mediator/Coordinator（按需）                                | 集中用例编排，不让每个模块相互依赖           |
| 单个目标需要返回成功/失败/剩余数量                                  | 直接同步命令或 Interface 函数                                    | 广播不是事务 API，不保证处理顺序和成功响应     |
| 项目已有 GameplayTag / State Machine / AnimNotify 驱动的流程 | 用现有机制，按下文「GameplayTag / State Machine / AnimNotify」一节审查 | 不另起一套通信方式；只修正路由不清或状态权威错位的地方 |

## Observer Pattern（观察者模式）

- UE 优先使用 `DECLARE_MULTICAST_DELEGATE...`（仅 C++）或 `DECLARE_DYNAMIC_MULTICAST_DELEGATE...` + `UPROPERTY(BlueprintAssignable)`（Blueprint 需要订阅）。动态 Delegate 有反射开销；按真实需求选型。
- 使用过去式/状态变化命名，例如 `OnInventoryChanged`、`OnHealthChanged`；事件由**状态持有者在成功提交更改后**发出。
- 不要由 UI/Listener 负责修改发送方内部数据，避免事件引发递归循环。
- `AddDynamic`、`AddUObject`、`AddWeakLambda` 等绑定方式应与对象生命周期匹配；管理 `FDelegateHandle`、解绑与重复绑定，验证 BeginPlay/EndPlay、Widget 重建、池化重激活时的行为。
- Delegate 不承诺跨多个观察者的业务顺序；核心动作必须先由同步函数明确执行，再广播结果。
- 避免用 Tick 轮询可直接由事件得知的状态，但高频更新需考虑事件频率/批量刷新。

## Mediator Pattern（中介者模式）

Mediator 的目的：**把多系统参与的一个业务流程的编排放在清楚的协调点上**，而不是创造一个知道游戏所有事情的全局 Manager。

- 适用：一次拾取需要检查物品有效性 → 库存容量 → 提交物品 → 通知任务 → 从世界移除，且需处理中途失败。
- 优先从已有 `PlayerController`、`Character` 或专用 `ULootCoordinatorComponent` 中选择生命周期合理的协调者。
- Mediator 调用参与方提供的小型命令 API，并明确谁拥有权威状态、谁提交操作、失败是否回滚。
- 保持 **一个用例/有界上下文一个协调者**；如没有多方编排复杂度，不要独立建 Mediator 类。
- 不能用全局 Event Bus + 字符串 Topic 取代有类型的接口；不要让 Mediator 复制各子系统内部规则。

## GameplayTag / State Machine / AnimNotify

战斗框架常用这三种机制串联流程，按以下规则判断：

- **Tag 事件不等于字符串总线**：GameplayTag 有层级，可在 Editor 中校验。每类 Tag 事件有明确的发送方和接收方（例如"在自身的 Ability 组件上激活带某 Tag 的 Ability"）时属于正常用法；只有向全局广播、接收方不确定的用法才按 Event Bus 反模式处理。
- **需要同步结果时不用事件**：Ability 是否激活成功、伤害是否生效，应由被调用方返回或由权威方的状态体现，不依赖"发出事件后总会有监听者处理"。
- **AnimNotify / AnimNotifyState 只发触发信号**：命中检测、无敌、输入缓冲等窗口状态由 Component 或 Ability 持有，Notify 只负责打开和关闭窗口。
- **窗口要兜底关闭**：Montage 被打断或 Ability 提前结束时，由持有状态的一方关闭窗口并清理命中记录，不只依赖 NotifyEnd。
- **State Machine 由所有者切换**：其他系统发事件请求切换，不直接改写当前状态。

## 示例：Loot → Inventory → UI

1. `InteractionComponent` 识别目标并请求互动能力（`IInteractable`）。
2. Character 或局部 `LootCoordinator` 解析“拾取”请求，调用 `InventoryComponent::TryAddItem(...)`，同步获得成功/失败。
3. 成功后才更新/移除 World Pickup；Inventory 在自身状态变化后发出 `OnInventoryChanged`。
4. UI 订阅事件刷新；音效/VFX 在合适的 Gameplay 或表现事件响应，不由 InventoryComponent 硬引用具体 Widget。
5. 若拾取逻辑很简单，可以直接由 Character 调用库存，无须引入 Coordinator。

## 避免的陷阱

- `UInventoryComponent` `Cast<UMyHUDWidget>` 去刷新界面。
- 为每个函数都加一层 Interface，导致跳转困难。
- 用 Delegate 广播“尝试添加物品”，期望其中某一个监听者确保物品已经添加。
- 用拥有全局访问权的 GenericMediator 把模块变成中心化的巨型 God Object。
