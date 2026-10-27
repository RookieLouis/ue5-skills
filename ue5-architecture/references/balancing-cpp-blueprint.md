# C++ 与 Blueprint 的平衡 / Balancing C++ and Blueprint

参考来源：Epic Games 的 [Balancing Blueprint and C++（UE 4.27，Architecture Notes）](https://dev.epicgames.com/documentation/unreal-engine/balancing-blueprint-and-cplusplus?application_version=4.27#architecturenotes)。本文结合其他 UE5 文档与项目约定整理，并非 Epic 原文；使用时需核实项目的引擎版本。

目标：**C++ 提供稳定行为契约与权威状态，Blueprint 提供内容装配、配置、适当的视觉与演出扩展**。这不是“所有 Blueprint 逻辑都禁止”，而是避免关键 Gameplay 规则分散且无法测试。

这里的“权威”定义见 [SKILL.md](../SKILL.md)：规则与状态的唯一来源，不等同于网络 Authority。

## 分工决策表

| 内容                                                   | 首选位置                             | 说明                                            |
| ---------------------------------------------------- | -------------------------------- | --------------------------------------------- |
| 生命值计算、库存堆叠、交易合法性、伤害规则                                | C++                              | 权威 Gameplay Logic，需可重复验证                      |
| 输入到 Gameplay 命令的分发                                   | C++ 或简明 Blueprint                | 复杂时 C++，内容试验时 Blueprint 也可；避免 Widget 决定核心规则   |
| Item/Weapon/Enemy 配置                                 | DataAsset / DataTable / Editor   | 设计师无需改源代码即可调数值                                |
| Mesh、Animation、Montage、Niagara、Sound、Widget Class 指派 | Blueprint Class / Editor         | 让 Asset 直接可视化配置                               |
| Camera Shake、受击 Flash、音效、VFX、非权威演出                   | Blueprint / AnimBP / Niagara     | 可通过 C++ 事件触发；避免阻塞核心游戏流程                       |
| AI 状态/行为                                             | BT/StateTree/Blueprint 与 C++ 可混合 | 选最可维护的表现形式，状态权威与关键算法可下沉 C++                   |
| UI 布局及动画                                             | UMG / Blueprint                  | 核心数据由 ViewModel/Gameplay 状态驱动，避免 UI 维护第二份真实数据 |
| Save/Load、稳定 ID、合法性检查                                | C++                              | 与 Runtime State 和 Data Definition 分离          |

纯 Blueprint 项目：表中"C++"表示权威层，由 BP ActorComponent 承担（无状态的计算可放 Blueprint Function Library），同样要求权威状态唯一、UI 不做规则判断。是否新增 C++ 模块见 [SKILL.md](../SKILL.md) 第 1 步。

## C++ 暴露规则

- 可配置定义优先 `EditDefaultsOnly`；需要关卡逐实例调节才考虑 `EditInstanceOnly` / `EditAnywhere`。
- 核心状态优先 `VisibleInstanceOnly`、`BlueprintReadOnly` 或 private getter；禁止为方便调整把所有字段 `BlueprintReadWrite`。
- 用 `UFUNCTION(BlueprintCallable)` 暴露命令或受控 API，用 `BlueprintPure` 暴露无副作用查询；不要将修改状态的方法标成 Pure。
- C++ 可定义 `BlueprintNativeEvent`（有默认实现）或 `BlueprintImplementableEvent`（纯 BP 表现 Hook）；使用时明确执行路径及是否调用父实现。
- 为 Blueprint 需要监听的状态使用 `BlueprintAssignable` 的 Dynamic Multicast Delegate；只在 C++ 消费时可使用原生 Delegate。
- 设置 Category、DisplayName/ToolTip、范围约束、Asset 类型过滤（适用时）；Designer-facing 配置提供稳妥默认值与数据验证。
- 谨慎允许 Blueprint 重写核心规则：不变量与最终提交操作尽可能留在 C++，允许 Blueprint 调整配置或做明确 Hook。

## 对 Blueprint / Asset 的尊重

- 重构 C++ 类型、UPROPERTY/UFUNCTION、类名、路径、Component 名称和 Blueprint 父类前，先核实现有 BP/Asset 引用影响；需要时规划重定向、迁移和回归验证。
- 不要声称仅凭 C++ 源码就完整理解 Blueprint Graph。
- 可将 Blueprint 业务逻辑逐步迁出：先抽取 C++ 权威状态和最小公共 API；再替换调用；最后验证 BP 行为和 Editor 配置。具体步骤见 [bp-to-cpp-migration.md](bp-to-cpp-migration.md)。
- 第三方 Marketplace/Fab 插件优先用 Adapter、独立 Component、继承或合法扩展点隔离；直接修改 Vendor 源码前评估更新成本和许可。

## Architecture Notes：类型依赖与资源加载

以下结合 Epic 的 [Architecture Notes（UE4.27）](https://dev.epicgames.com/documentation/unreal-engine/balancing-blueprint-and-cplusplus?application_version=4.27#architecturenotes) 与 UE5 资源管理文档，整理为 UE5 项目的决策规则；按实际依赖和内容规模应用。

1. **公共契约放在轻量基础类型中**：跨系统使用的属性、函数优先定义在 C++ 基类、轻量 BP 基类或接口。Cast 到具体 BP 类、把它用作变量或函数参数类型，都可能使调用方依赖该 BP 及其硬引用资源。关注的是依赖链，不能据此断言 Cast 运算本身昂贵；轻量基类也应避免引用具体内容子类。

2. **减少 BP 循环依赖**：检查类型、默认值和资源引用形成的环。通过职责拆分、基础类型、接口或事件明确依赖方向，降低加载与编译耦合；只把调用改成 Interface 并不能保证其参数和默认值没有硬引用。C++ 同样需要管理模块与类型依赖。

3. **资源配置与 C++ 行为分离**：通用基类优先暴露可配置属性，由 BP 默认值或 DataAsset 指派内容。谨慎在构造器中用 `ConstructorHelpers::FObjectFinder` / `FClassFinder` 写死可选资源路径：它们可能在相关类/CDO 构造时触发同步加载，并增加重命名维护成本。把硬引用搬进 BP 或 DataAsset 后，仍会随持有者加载；小型、必需、常驻资源可以使用硬引用。参见 [UE5 Referencing Assets](https://dev.epicgames.com/documentation/en-us/unreal-engine/referencing-assets-in-unreal-engine)。

4. **用可追踪的资源引用，并核实 Cook**：按需引用资源用 `TSoftObjectPtr`，引用类用 `TSoftClassPtr`，需要路径值时用 `FSoftObjectPath`；优先在可被 Cook 扫描到的资产属性中配置。避免把运行时拼接字符串后 `LoadObject` 作为唯一资源发现方式。软引用不能覆盖所有 Cook 场景；由配置、运行时生成或动态注册引入的内容，应核实 Asset Manager 扫描与 Cook 规则，并在打包版本验证可加载。纯 Blueprint 项目使用 Soft Object Reference / Soft Class Reference 类型的变量。参见 [UE5 Asset Management](https://dev.epicgames.com/documentation/en-us/unreal-engine/asset-management-in-unreal-engine)。

5. **稳定共享类型优先原生定义**：已有 C++ 项目中，跨 C++ / BP 边界、用于关键存档或长期公共契约的类型，优先 `USTRUCT(BlueprintType)` / `UENUM(BlueprintType)`。BP UserDefinedStruct / UserDefinedEnum 可用于局部原型，但不能直接作为常规静态 C++ 类型使用；不要按“被几个 BP 使用”机械决定迁移。已有类型替换时先核实资产默认值、DataTable 和旧存档兼容，详见 [bp-to-cpp-migration.md](bp-to-cpp-migration.md)。

6. **异步加载需要明确的完成与生命周期契约**：软引用只推迟加载；`LoadSynchronous` 仍会阻塞，不能等同异步。大型可选 Mesh、音效、敌人类等可通过 Asset Manager / `FStreamableManager::RequestAsyncLoad` 按需加载，说明完成前的行为、失败处理和请求对象失效时的处理；加载后按使用期保留资源引用或有效加载句柄。Blueprint 中对应 `Async Load Asset` / `Async Load Class Asset` / `Async Load Assets` 节点；`Load Asset Blocking` / `Load Class Asset Blocking` 是同步加载。少量简单内容不必为此新增自定义 Manager。参见 [UE5 Asynchronous Asset Loading](https://dev.epicgames.com/documentation/en-us/unreal-engine/asynchronous-asset-loading-in-unreal-engine)。

评审时用 [Reference Viewer / Size Map](https://dev.epicgames.com/documentation/en-us/unreal-engine/reference-viewer-in-unreal-engine) 核实硬/软依赖链及资源规模，再决定是否解耦或改成按需加载；不要只凭 Graph 外观推断加载成本。

## Designer 验收问题

1. 添加新物品/敌人是否可以只创建 DataAsset / Blueprint 配置而不编辑 C++？
2. 改一个 Icon、VFX、数值是否能在 Editor 中找到清晰入口？
3. Designer 是否能误改权威 Runtime State 或使游戏进入不一致状态？
4. 缺失 Asset、无效 RowName、越界参数是否会提供清晰、可定位的错误？
5. 改动后的旧 Blueprint 及关卡实例是否还能正确加载？

依赖链与软引用加载的工程审查项见 [review-checklist.md](review-checklist.md) 的 A、D 节。

## 不建议的极端

- “Blueprint 仅能放美术资源，任何逻辑都不能写”：阻碍快速迭代和原生工作流。
- “C++ 只写组件壳，所有逻辑都交给 Blueprint”：影响系统级维护、Code Review 和确定性测试。
