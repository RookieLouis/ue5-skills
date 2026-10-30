# UE5 Skills

两个用于 Unreal Engine 5 项目开发的 AI 技能，分别处理 Gameplay 架构和 UE C++ 编码规范。技能正文以中文为主，保留 Unreal Engine 的英文术语。

## 包含的技能

| 技能 | 用途 |
| --- | --- |
| [ue5-architecture](ue5-architecture/SKILL.md) | 设计、重构或审查 Gameplay 架构及 Content 目录组织，包括类与组件职责、系统通信、数据驱动、C++ 与 Blueprint 分工，以及 Blueprint 向 C++ 迁移。 |
| [ue5-epic-cpp-standard](ue5-epic-cpp-standard/SKILL.md) | 编写、修改或审查 UE C++ 时，参考 Epic UE 5.8 C++ Coding Standard 检查命名、注释、格式、API 设计等；也提供 Content 资产命名指南，并明确标注项目约定。 |

两个技能可以配合使用：架构技能侧重系统设计和目录组织，C++ 技能侧重代码规范和资产命名。架构技能也支持纯 Blueprint 项目，不要求为了使用技能而新增 C++ 模块。

仓库根目录的 [AGENTS.md](AGENTS.md) 包含技能调用、协作和规范优先级的中文约定，可作为项目的 `AGENTS.md` 使用，也可将内容复制到支持 Custom Instructions 的工具中。使用前请按项目需求调整。

## 安装

将需要的技能文件夹完整复制到 Aura 项目的技能目录，保留各自的 `SKILL.md` 和 `references/`：

```text
<项目目录>/Saved/.Aura/Skills/
├── ue5-architecture/
│   ├── SKILL.md
│   └── references/
└── ue5-epic-cpp-standard/
    ├── SKILL.md
    └── references/
```

如果技能列表没有更新，请按 Aura 的实际行为重新加载项目或重启 Aura。其他兼容 `SKILL.md` 的工具可能采用不同的安装位置和触发方式，请参照对应工具的文档。

## 使用

在 Aura 中可以通过 `/ue5-architecture` 或 `/ue5-epic-cpp-standard` 显式调用，也可以在描述相关任务时让支持自动调用的工具根据技能描述选择。建议同时提供项目背景、引擎版本、目标和需要检查的范围。例如：

```text
/ue5-architecture 帮我审查背包系统的组件职责与 UI 通信方式。
/ue5-epic-cpp-standard 审查这个 ActorComponent 的 UE C++ 命名、生命周期与接口。
```

技能中的参考资料按任务需要读取。具体规则、适用边界和验证要求请见各技能的 `SKILL.md`。使用前请根据项目现有规范、实际引擎版本和工具能力核实建议；技能不能替代编译、编辑器验证或测试。

## 参考来源

`ue5-architecture` 的 [C++ 与 Blueprint 的平衡](ue5-architecture/references/balancing-cpp-blueprint.md) 参考了 Epic 的 [Balancing Blueprint and C++（UE 4.27）](https://dev.epicgames.com/documentation/unreal-engine/balancing-blueprint-and-cplusplus?application_version=4.27#architecturenotes) 及其他 UE5 文档，并结合项目约定。

`ue5-epic-cpp-standard` 附带的 `references/Epic_CPP_Coding_Standard.md` 是 Epic Games 的 UE 5.8 文档导出快照，来源为 [Epic C++ Coding Standard](https://dev.epicgames.com/documentation/unreal-engine/epic-cplusplus-coding-standard-for-unreal-engine)。Epic 原文与其权利归原权利人所有；项目约定在技能正文中另有标识。
