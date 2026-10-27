# Object Pooling（对象池）：何时用、如何安全实现

## 核心立场

**默认不做 Object Pooling。先建立基准、Profile 再决定。** 频繁 `SpawnActor/Destroy` 可能产生开销，但池化也引入 Actor 生命周期、状态复位、GC、内存占用和 Debug 复杂度。性能瓶颈可能在 Niagara、碰撞、Tick、材质/渲染，而非 Spawn 本身。

## 决策条件

较适合：大量、短寿命、同类型、持续反复创建的 Projectile、某些临时 Actor/VFX，在 Unreal Insights / Stats / Trace 中能观察到 Spawn/Destroy 开销且可安全重用。

不优先适合：低频任务/宝箱/拾取、复杂 AI、状态众多的 Enemy（复位成本高）、关卡常驻 Actor、无法安全复位的第三方类。

**先核实引擎自带能力**：例如 Niagara 系统/Component 的内建池化选项；如果能满足需求，不必再单独写 Niagara Pool Manager。自定义 Pool 不保证必然更快。

## 简单优先的设计

- 一个 Pool 管理一种明确的对象契约，必要时按类/种类分池；池归属于与该 World / Level 一致的生命周期。
- 定义 `Acquire`（取出）、`Activate`（重新初始化）、`Deactivate`（停用）、`Release`（归还）。
- Pool 必须显式区分 **Available** 和 **Active**，防止重复释放、重复出借、已 Destroy 对象被重新使用。
- 由 Pool 或明确 Owner 控制 Spawn 数量、预热数量、Max Capacity，以及耗尽策略（临时创建 / 拒绝视觉效果 / 其他根据 Gameplay 需求选择）。
- `Release` 不等于 `Destroy`。调用 `Destroy()` 的 Actor **不能**作为仍可回收的有效实例继续放回池。
- 保证重入安全：同一对象在 Acquire→Release→Acquire 循环后，不留前一次使用的 Timer、Hit 绑定、Instigator、Damage、运动速度、Transform、Visual 状态。

## Pool Actor 释放检查清单

停用并确认：

1. `SetActorHiddenInGame(true)`、相关组件 Visibility、`SetActorEnableCollision(false)`，停止 Movement / ProjectileMovement 并处理残留 Overlap/Hit。
2. 关闭不需要的 Tick、Timer、Latent Action，以及异步回调或需防止复用后旧回调运行的 generation token。
3. 清理/复位 Gameplay State（Damage、Target、Instigator、Owner、Velocity、LifeTime、Team、命中标志、碰撞忽略列表等）。
4. Niagara / Audio / Decal / Trail 等表现按各自生命周期停止、重置或重建，确认不会将上一轮残影附到下一轮。
5. 不要在每次 Acquire 中重复 `AddDynamic` 导致绑定数量累积；回调绑定策略要稳定，释放/销毁时解绑必要句柄。
6. Pool Manager 必须正确追踪对象生命周期，校验 `IsValid` 与 `EndPlay`；World/Level 卸载、PIE 重启、切图时统一清理。

激活时：设置新的 Transform、Owner/Instigator、Collision、速度、生命期、特效和所有 Gameplay 参数；任何项若没有被可靠复位应默认禁止复用。

## UObject / GC、性能和测试

- UObject/Actor 引用使用符合 UE 反射/GC 规则的引用类型，如 `UPROPERTY()` 托管的 `TObjectPtr` 容器；Actor 已标记销毁时，持有指针并不会让它重新成为有效 Actor。
- Pool 空闲对象仍可能占内存并保有昂贵的 Component；需要容量限制、按场景关闭和真实的内存测量。
- 测量指标：每帧 Spawn 数、GT 峰值时间、GC 负担、内存占用、Actor 数、Collision 与 Niagara 成本。做池化前后同样场景可重复测试。
- 必须测试：连射→同时命中→连续释放、池耗尽、场景切换、PIE 重启、异步回调到达旧对象、反复 Acquire/Release 100 次后状态相同。

## 与其他模式关系

- Pooling 是**生命周期/性能优化**，不是 SOLID 的强制组成。
- 池可以是 World 级的服务，也可以局部 Owned Component；不要为拿到 Pool 到处创建无类型 Global Singleton。
- Projectile 的“返回池”流程中，Projectile 自身负责报告死亡/结束，实际释放由 Pool/Owner 执行；不要使 Projectile 依赖具体 UI 或全局 Gameplay Manager。
