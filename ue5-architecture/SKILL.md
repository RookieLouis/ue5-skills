---
name: ue5-architecture
description: 为 Unreal Engine 5 C++/Blueprint 项目设计、重构或审查 Gameplay 架构。当任务涉及新系统规划、多个 Actor/Component/Subsystem 的职责边界、Inventory/Combat/Interaction/Save 等跨系统通信、SOLID/SoC/Loose Coupling、Observer/Mediator、DataAsset/DataTable、Inheritance/Composition、Object Pooling、BP 依赖链与 Hard/Soft Reference、异步加载、Designer Convenience、Blueprint → C++ 迁移或架构 Code Review 时使用。普通语法解释、单点 Bug 修复、简单属性修改、仅添加日志时不要自动使用。
---

# UE5 Architecture Design & Review

目标：构建适合中小型、以 AI 辅助开发为主的 Unreal Engine 游戏架构——职责清楚、依赖可控、易于 Editor/Blueprint 配置、便于演进，且不为了模式本身而增加抽象。使用用户的语言说明，保留 Actor、Component、Projectile、Collision、Delegate、DataAsset、Blueprint 等 UE 技术术语英文。

本 Skill 中的“权威”（Authority）指规则与状态的唯一来源，不等同于网络复制中的 Authority；单机项目不因此需要服务器、RPC 或复制系统。

## 任务处理流程（必须遵循）

1. **核实现状**：先确认项目形态：`.uproject` 是否有 `Modules`、项目是否有 `Source/`。再检查相关 C++、Blueprint、Build.cs、已有 Plugin/Asset、既有架构及目标 UE 版本。不能访问项目时标注假设；绝不声称已检查未看到的 Blueprint Graph、Asset 内容或编译结果。优先复用已有系统，尊重第三方资产的维护和升级边界。
   
   - **纯 Blueprint 项目**：原则不变，用 BP 的对应物落地：BP ActorComponent、Blueprint Interface、Event Dispatcher、继承 `PrimaryDataAsset` 的 BP 子类。本 Skill 中由 C++ 承担的权威层，在这类项目里由 BP Component 承担。
   - **新增 C++ 模块**是项目级决策：单独提出，说明理由和成本，经用户确认后再执行；不要夹带在普通设计或审查任务里。

2. **确定范围**：列出业务规则、运行时状态、配置数据、表现/UI、存档、Owner 与生命周期；仅解决当前任务确实需要的扩展点。单机优先，不无端引入网络同步基础设施。

3. **提出轻量方案**：给出 Actor / ActorComponent / UObject / Subsystem / Interface / Delegate / DataAsset / DataTable 的职责分配、调用方向和需要的数据流。标出哪些是直接引用、命令调用、事件通知；比较最简单可行方案与必要的抽象方案。

4. **判定模式，而非套模式**：先判断 Direct Reference 是否足够，再评估 Interface / Observer / Mediator；先判断现有类是否可以扩展，再决定 Inheritance / Composition；只有基于性能证据才引入 Object Pooling。参照本 Skill 的决策资料。

5. **明确 C++ / Blueprint 边界与 Designer 接口**：根据项目形态明确核心规则和权威状态的归属、Blueprint 的流程/表现扩展，以及 DataAsset、DataTable、Editor 参数。已有 C++ 项目优先把稳定的公共契约与关键规则放在 C++；纯 Blueprint 项目按第 1 步由 BP Component 承担，不为分工原则强行新增 C++。公开最小必要 API，明确状态读写权限与数据验证方式。涉及跨 BP 类型、共享 Struct/Enum 或资源加载时，按 [balancing-cpp-blueprint.md](references/balancing-cpp-blueprint.md) 检查加载依赖、类型归属和按需加载边界。

6. **实现或评审**：任务要求编码时做最小可验证改动；先检查依赖与生命周期，再处理状态与数据，最后处理表现和调试。不无授权重构无关模块。不在设计任务中擅自大范围修改代码。

7. **验证和汇报**：说明风险、依赖变化、必要测试与未验证项；在有构建/Editor 环境时运行匹配测试并报告结果，禁止虚构已通过的验证。架构审查应按严重程度报告问题及其文件位置，再给精简修复建议。

## 按需读取 Reference

仅加载与任务直接相关的文件，不要默认一次读取所有 Reference：

| 任务重点                                                                              | 读取文件                                                        |
| --------------------------------------------------------------------------------- | ----------------------------------------------------------- |
| SOLID、SoC、Loose Coupling、职责划分                                                     | [principles.md](references/principles.md)                   |
| Inheritance、Composition、ActorComponent、Subsystem 选型                               | [class-design.md](references/class-design.md)               |
| Observer、Mediator、Interface、Delegate、直接调用、GameplayTag 事件、State Machine、AnimNotify | [communication.md](references/communication.md)             |
| DataAsset、PrimaryDataAsset、DataTable、配置/Runtime/Save 分离                           | [data-driven.md](references/data-driven.md)                 |
| C++ vs Blueprint、Designer Convenience、Editor API、Architecture Notes（BP 依赖、共享类型、资源加载） | [balancing-cpp-blueprint.md](references/balancing-cpp-blueprint.md) |
| Projectile、特效等对象的 Object Pooling                                                  | [object-pooling.md](references/object-pooling.md)           |
| Ownership/GC、日志、断言、性能与测试（C++ 代码风格见 ue5-epic-cpp-standard Skill）                   | [ue5-implementation.md](references/ue5-implementation.md)   |
| 架构审核与交付验收                                                                         | [review-checklist.md](references/review-checklist.md)       |
| 单机撤离游戏 Loot → Inventory → UI 实际示例                                                 | [extraction-example.md](references/extraction-example.md)   |
| Blueprint → C++ 迁移、Reparent、CoreRedirects、UserDefinedStruct/Enum 替换               | [bp-to-cpp-migration.md](references/bp-to-cpp-migration.md) |

## 架构选型默认顺序

- **职责**：先让每个模块只有清晰的变化原因；再用 Composition 拆分可复用能力；能自然表达 `is-a` 关系且遵守行为契约时可以 Inheritance。不要机械禁止继承。
- **通信**：稳定拥有关系用 Direct Reference；不同类型执行统一行为用 Interface；状态变更通知用 Delegate / Observer；复杂跨系统流程才考虑 Mediator / Coordinator。通知事件不代替必须执行的业务命令。
- **数据**：少量每项独立编辑或复杂引用数据，评估 DataAsset；大量同构行/批量平衡，评估 DataTable；按需加载、AssetManager 管理需求再评估 PrimaryDataAsset。配置、Runtime State、Save Data 必须区分。
- **性能**：Object Pooling 是以额外生命周期复杂度换取性能稳定性，默认不用；先 Profile。优先评估 UE 自带的粒子等池化支持，避免重复造轮子。
- **引擎约束**：尽量使用原生 UE 概念与生命周期，尊重现有项目版本。对于 Engine 新版本 API 或第三方插件行为，优先核实实际源码/官方文档；不把推测当事实。

## 输出格式（按任务复杂度裁剪）

### 新系统设计

1. **需求与假设**：Gameplay 范围、单机/多人、内容量、性能热点。
2. **推荐方案**：组件与职责表；关键数据类型；依赖/事件/调用流程；C++/Blueprint 分工。
3. **选择理由**：为什么使用或不使用 Interface、Mediator、Pooling、DataTable、Inheritance 等。
4. **实现次序**：按最小可运行闭环拆分步骤。
5. **验收与风险**：正确性、生命周期、Editor 调整、性能、可测试性。

### 重构 / Code Review

1. 先列**已确认的问题**（严重级别、证据位置、影响）；区分猜测与实证。
2. 给出最小改动方案、保留行为与迁移风险，特别检查 Blueprint 引用、数据资产迁移和 Save 兼容性。
3. 给出具体验证步骤及未执行的测试；避免用泛泛的模式术语代替可操作建议。

## 必须避免

- 为了符合 SOLID 而生成大量 Manager、Interface、BaseClass 或转发类。
- 把所有跨 Component 调用改成 Delegate、把每个系统改成 Subsystem、把每个对象都放进 Object Pool。
- 让 Widget、表现层 Blueprint 或具体内容 BP 子类（如某个敌人/武器 BP）绕过所属权威层改写核心规则或维护第二份真实状态；纯 Blueprint 项目的 BP Component 可以承担权威层。不要让静态 DataAsset 承载可变库存/角色状态。
- 通过 `check` 处理玩家输入等可恢复错误，或通过静默 `if` 吞掉重要 invariant 失败。
- 未检查现有 Plugin/Blueprint 便断言某个架构已经存在或已经验证。
