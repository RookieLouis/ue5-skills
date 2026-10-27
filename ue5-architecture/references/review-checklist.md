# 架构 Code Review / 重构验收清单

只审查与当前任务相关的项；不因项目没有使用每一种模式而扣分。每条发现都应附：**证据位置、实际风险、最小修复、验证方法**。

## A. 职责与依赖

- [ ] Gameplay 规则、状态、UI、配置、存档是否各有明确 Authority？
- [ ] Character 是否成了库存、Combat、Quest、Save 的 God Object？已有 Coordinator 是否仅负责流程协调？
- [ ] 不同 Component 是否存在双向循环依赖？Build.cs 是否引入不必要的模块依赖？
- [ ] Stable Ownership 的对象是否用了简单直接引用，而非多余的 Interface/Event Bus？
- [ ] 公共基类、接口和 UI 是否因 Cast、变量或参数类型依赖了具体 BP 的整条内容链，或越过了独立模块边界？（用 Reference Viewer / Size Map 核实）

## B. SOLID / Inheritance / Composition

- [ ] 每个类是否有明确变更原因，而不是追求无意义单函数类？
- [ ] 是否存在不合理的多层继承/为了参数差异增加子类？
- [ ] ActorComponent 拆分是否带来真实复用或职责隔离？
- [ ] 子类是否遵守父类行为约定与 Engine 生命周期？
- [ ] Subsystem 的生命周期是否匹配业务作用域，避免滥用 Global Manager？

## C. Observer / Mediator

- [ ] 命令与事件是否分离？业务成功后才发广播？
- [ ] 是否明确同步成功/失败和失败补偿；是否错误依赖 Delegate 监听顺序？
- [ ] 监听者是否在适当时机绑定、解绑；Widget 重建/Actor Pool 复用是否重复绑定？
- [ ] Mediator 是否真的在协调复杂用例，还是仅多了一层转发？
- [ ] 事件的数据有效性与所有权是否清楚？
- [ ] GameplayTag / State Machine 事件是否有明确的接收方，而不是接收方不确定的全局广播？
- [ ] AnimNotify 窗口的状态是否由 Component/Ability 持有，并在 Montage 中断、Ability 结束时关闭？

## D. Data-Driven / Editor

- [ ] DataAsset、DataTable 是否按内容规模与编辑需求选择？
- [ ] 静态 Definition、Runtime State、Save Data 是否明确分离，字段是否有唯一 Source of Truth？
- [ ] DataTable Row 缺失、空表、CSV 重导、行名变更、指针失效是否妥善处理？
- [ ] PrimaryDataAsset / Soft References 是否有实际 AssetManager/加载方案：加载完成前的表现、失败处理、资源持有期是否明确，打包版本是否包含所需资源？
- [ ] 能否在不重编 C++ 的情况下增添同类新 Item / Weapon 内容？
- [ ] Blueprint 暴露是否最小化；ToolTip/Category、默认值、范围和 Data Validation 是否清晰？
- [ ] 改名/更换父类/迁移 Component 后，旧 Blueprint、关卡和存档是否兼容？

## E. Object Pool / 性能

- [ ] 是否有 Profile 数据证明引入 Pool？是否检查现成 UE 池化功能？
- [ ] Pool Actor 的 Collision、Timer、Delegate、Instigator、Velocity、FX、UI/GC 等状态能否完全 Reset？
- [ ] Pool Active/Available 状态能否防止重复释放与交叉使用？
- [ ] World 销毁、PIE 重启、容量耗尽时是否安全？
- [ ] Pool 之后的帧时间/内存是否优于原方案？

## F. 稳定性与交付

- [ ] 核心失败是否能通过日志/断言暴露，而不是静默跳过？
- [ ] `check` 是否只用于不可恢复的内部不变量，`ensure` 是否用于非预期但可继续的问题？
- [ ] UE C++、GC、Owner/Outer、Timer、EndPlay 生命周期是否正确？
- [ ] 是否保留旧功能行为、第三方插件边界和资产迁移路径？
- [ ] 实际运行了哪些 Build / Automation Test / PIE / Profiler 测试？未做的标记出来。

## 推荐输出模板

### 审查结果

| 级别                             | 位置                                                 | 已确认问题 / 证据 | 影响  | 最小修改建议 | 验证  |
| ------------------------------ | -------------------------------------------------- | ---------- | --- | ------ | --- |
| Critical / High / Medium / Low | `Source/...:Line` 或 `/Game/...资产 → Graph/Function` | 具体失效场景     | ... | ...    | ... |

### 推荐依赖与数据流

- 系统 A：负责何种状态/行为。
- 系统 B：负责何种状态/行为。
- A → B：直接调用 / Interface / Coordinator；原因。
- B → UI：Delegate；何时通知。

### 备选与理由

- 最简可行方案：……
- 仅在 XX 条件成立时升级到 Mediator / Pool / PrimaryDataAsset：……

### 测试与剩余风险

- 已执行：……
- 未执行：……
- Blueprint/Asset / 性能尚需验证：……
