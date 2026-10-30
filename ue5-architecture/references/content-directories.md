# Content Directories：UE 内容目录组织

本指南参考 [Tom Looman 的 Content Directories](https://tomlooman.com/unreal-engine-naming-convention-guide/#content-directories)，并结合 UE 5.8 官方文档。下表的目录名称和顺序完全按 Tom 的原表列出，描述包含补充说明；这不是 Epic 强制目录规范。

## 规划原则

- 先检查项目当前 Content、插件和第三方包的结构、资产依赖与约定；新项目可考虑将项目自有内容集中在 `Content/<ProjectName>/`（编辑器中的 `/Game/<ProjectName>/`），与 Marketplace 内容分开，便于识别和迁移。插件内容使用其自身的挂载点，不能假定都在 `/Game`。
- 按功能或内容归属组织，保持同一内容的静态网格、材质、纹理等基本依赖就近；多个内容复用的资产放在清晰的共享位置。仅在数量或工作流确有需要时再细分 `Meshes`、`Materials`、`Textures`，避免无意义的层级。
- 优先延续项目已有结构与第三方包边界，不把下列目录机械地全部创建出来，也不因本指南自动重排项目。大功能可以拥有自己的顶层目录。

`MyGame` 是示例项目名，可替换为实际项目名；表中 `../` 表示相对于 `Content/MyGame` 的路径，`...` 表示其下的内容。

| Sub-Folder | Description |
| --- | --- |
| `Content/MyGame` | 项目自有内容的顶层目录；其余行均相对该目录。 |
| `../AI` | AI 相关资产，例如行为树、黑板和 AI 配置。 |
| `../Art` | 网格、纹理、材质等美术内容；UI 与特效专用内容放在各自目录。 |
| `../Audio` | 音频资产，例如音效、音乐及相关配置。 |
| `../Core` | 项目核心及共享的基础内容。 |
| `../Input` | Enhanced Input 相关资产，例如 Input Action 和 Input Mapping Context。 |
| `../Characters` | 角色相关内容。 |
| `../Characters/Animations` | 角色动画相关资产。 |
| `../Effects` | 视觉特效及其相关内容。 |
| `../Effects/Flares` | 示例：各类光晕特效专用的系统、材质和纹理。 |
| `../Maps` | 所有关卡，包括开发专用关卡。 |
| `../Maps/Dev/...` | 共享的开发专用关卡；排除 Cook 和打包需另行配置与验证。 |
| `../UI` | 用户界面相关资产。 |
| `../UI/Materials` | UI 专用材质。 |
| `../UI/Fonts` | UI 字体资产。 |
| `../Actions` | Action 资产示例，类似 GAS Abilities。 |
| `../Expeditions` | 适合独立顶层目录的大型功能示例，主要存放 Blueprint。 |

同一资源只选一个明确归属；这些文件夹不暗示必须按资产类型拆分，也不规定具体 Gameplay 架构。

## 开发内容与打包

- `Content/Developers/<User>/` 可作为个人实验区；按 [Epic Developers Folder](https://dev.epicgames.com/documentation/unreal-engine/developers-folder-in-unreal-engine) 在 Content Browser 启用 Show Developer Content。正式内容不要引用此目录中的资产，避免 Cook 失败或错误依赖。
- `Maps/Dev` 和开发目录只是组织约定，**目录名称本身不保证排除 Cook/打包**。要排除时，结合项目 Packaging 设置、Asset Manager Cook 规则、引用关系与目标构建核实；例如评估 Directories to Never Cook，避免排除运行时需要的内容，并通过实际 Cook/打包验证。参考 [Epic Packaging Settings](https://dev.epicgames.com/documentation/unreal-engine/project-section-of-the-unreal-engine-project-settings)。

## 已有项目调整

仅在用户要求的范围内执行迁移，先备份受影响内容或确认可恢复的版本控制状态；检查引用、软路径、配置、存档以及第三方内容更新边界。资产移动/重命名使用 UE 编辑器或经验证的 UE 资产工具，不直接用文件系统批量移动 `.uasset`。

移动后检查 [Asset Redirectors](https://dev.epicgames.com/documentation/unreal-engine/asset-redirectors-in-unreal-engine)：按受影响范围修复重定向器，并确认引用包能保存；还要检查配置/代码中的字符串路径、软引用和存档兼容性。验证相关 Blueprint、关卡和必要的 Cook；未执行的验证要如实说明。资产命名参见 `ue5-epic-cpp-standard` 的资产命名指南。
