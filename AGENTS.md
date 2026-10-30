# Unreal Engine Skills

## 架构设计：ue5-architecture

- 性质：项目自定义的架构规范，属于“项目专属规范与架构约定”，不是 Epic 官方规范。
- 使用场景：Content 目录规划、资产归类；新系统规划；多个 Actor / Component / Subsystem 的职责边界；Inventory / Combat / Interaction / Save 等跨系统通信；SOLID / SoC / Loose Coupling；Observer / Mediator；DataAsset / DataTable；Inheritance / Composition；Object Pooling；Blueprint 依赖和资源加载；Designer Convenience；Blueprint → C++ 迁移；架构 Code Review。C++ 与纯 Blueprint 项目均适用。
- 不自动使用：普通语法解释、单点 Bug 修复、简单属性修改、仅添加日志。
- 新增 C++ 模块属于项目级决策：先说明理由和成本，经用户确认后再执行；任何任务都不得顺带新增。
- 按需读取：[SKILL.md](ue5-architecture/SKILL.md) 为入口，按任务读取 `references/` 中对应文件；目录组织读取 [Content Directories](ue5-architecture/references/content-directories.md)。

## Coding规范与资产命名：ue5-epic-cpp-standard

- C++ 规范来源：Epic Games Unreal Engine 5.8 C++ Coding Standard；Skill 中标注“项目约定”的条目（如注释语言跟随用户语言）不是 Epic 原文，按第 2 级处理。
- 使用场景：生成、修改、重构、Debug、Review 或解释 UE C++（.h / .cpp、反射宏、UE C++ API）；规划、命名或审查 UE Content 资产名称和前缀。
- 不自动使用：不涉及资产命名的纯 Blueprint 节点配置或美术调整；涉及 C++ 时，仍对 C++ 部分应用本 Skill。
- 新增和本次改动的代码遵循规范；既有代码只在改动行附近修正，不做全量整改。
- 按需读取：[SKILL.md](ue5-epic-cpp-standard/SKILL.md) 为入口，按任务读取 `references/`；资产命名读取 [Naming Convention Guide](ue5-epic-cpp-standard/references/asset-naming-guide.md)。纯资产命名任务不套用 C++ 工作流或编译检查。
- 资产命名指南参考 Epic 和 Tom Looman 的推荐；项目已有约定及用户明确要求优先。资产名称与 C++ 标识符使用不同规范，不因指南自动批量重命名已有资产。

## 协作规则

- 先设计后Coding：任务既涉及架构又要写 C++ 时，先按 ue5-architecture 确定方案，再按 ue5-epic-cpp-standard 编写代码。
- 分工：职责划分、通信方式、数据组织、Content 目录和 C++ / Blueprint 分工以 ue5-architecture 为准；C++ 代码风格与资产命名以 ue5-epic-cpp-standard 为准；断言、日志、失败可见性策略以 ue5-architecture 的 [ue5-implementation.md](ue5-architecture/references/ue5-implementation.md) 为准（项目约定）。
- Review 顺序：正确性、编译、UObject 生命周期 → 架构问题（职责、依赖方向、配置与运行时状态分离）→ Coding Standard。
- 修改后执行与改动相关且可用的验证；涉及 C++ 时运行可用的编译与测试；无法验证时明确说明。

## 项目约定

- 避免过度设计，不为设计模式引入不必要的抽象；判断标准见 ue5-architecture 的“判定模式，而非套模式”与“必须避免”。
- 优先 UE 原生 API；标准库与 UE 实现之间的取舍以 Epic 原文 Use of standard libraries 为准。
- 不得将个人编程偏好或项目约定冒充 Epic 官方规范。
- 优先采用详细注释，用户正在学习 C++。

## 规范优先级

编译器 / UHT / UBT 技术约束不参与排序：任何要求与之冲突时，说明原因并给出替代方案。其余按以下顺序：

1. 用户当前任务的明确要求。
2. 项目专属规范与架构约定（ue5-architecture 的架构决策及其标明的项目约定；ue5-epic-cpp-standard 中标注“项目约定”的条目）。
3. Epic C++ Coding Standard（ue5-epic-cpp-standard 的 Epic 原文摘要，约束代码层面）。
4. 其他通用 C++ 实践。

资产命名及 Content 目录指南是独立的推荐资料，不属于 Epic C++ Coding Standard；应在用户明确要求和项目已有约定下使用。存在真实冲突时，指出冲突并给出解决方案，不得擅自忽略项目要求。
