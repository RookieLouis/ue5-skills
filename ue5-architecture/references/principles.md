# 基础架构原则：SOLID / SoC / Loose Coupling

原则只用于改进设计决策，不是强制增加类型数量的评分体系。**清晰、正确、可配置 > 模式数量**。

## SOLID

| 原则 | UE5 中的行动准则 | 典型反例 |
|---|---|---|
| SRP | 一个类/模块围绕明确的变化原因组织；Combat、Inventory、UI 分别维护自身规则 | Character 同时处理伤害公式、存档、UI 创建、物品堆叠 |
| OCP | 在确定有真实变体时，通过 DataAsset、组合、Interface 或受控继承提供扩展点 | 为假设中的几十种敌人预先写复杂基类 |
| LSP | 子类保持基类行为契约，不让调用者必须判断特殊子类才能安全运行 | 子类重写核心函数后悄悄拒绝原本有效操作 |
| ISP | Interface 应描述一个明确能力，例如 Interactable，而不是通用万能接口 | IEverything 含拾取、受击、交易、任务方法 |
| DIP | 跨独立系统的稳定策略优先依赖小型契约，允许明确 Ownership 下的直接依赖 | Inventory 硬依赖某个具体 Widget 蓝图类型 |

## Separation of Concerns（SoC）

- 以职责和变化原因划分 Gameplay、Presentation、配置、Persistence，而不是单纯按文件长度拆分。
- CombatComponent 负责战斗状态/规则；Widget 负责展示；配置来自 DataAsset/DataTable；Save System 负责持久化。
- Character/Pawn 可以作为协调者保存对自有 Component 的引用，**不是任何协调代码都属于 God Object**。
- 每个跨系统数据流应有明确的权威状态持有者，避免 UI、副本 Actor 同时维护另一份“真实库存”。

## Loose Coupling（松耦合）

- 降低不必要的编译依赖、运行时对象依赖和对具体 Blueprint Class 的依赖。
- 相同目标不必一律选 Interface：Owner 直接调用自有 Component 是自然、清晰且便于 Debug 的实现。
- 使用前必须考虑事件解绑、对象销毁、Asset 软硬引用、模块 Build.cs 依赖、Subsystem 生命周期。
- **低耦合不等于零依赖**。滥用全局服务定位、字符串消息、万能事件总线往往降低可追踪性。

## 设计优先级与冲突解决

1. Gameplay 正确性和数据一致性。
2. UObject 生命周期、安全性、Editor/Blueprint 资产兼容。
3. 可读、可验证、职责与依赖透明。
4. Designer Convenience、性能和扩展成本。
5. 对未来扩展的预估；不以猜测覆盖前四项。

架构 Review 必须说明：现有具体问题是什么、为什么要改变、最小方案为何不足、抽象引入了什么维护成本。
