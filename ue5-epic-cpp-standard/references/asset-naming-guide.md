# Naming Convention Guide：UE 资产命名

参考 [Epic UE 5.8 Recommended Asset Naming Conventions](https://dev.epicgames.com/documentation/unreal-engine/recommended-asset-naming-conventions-in-unreal-engine-projects) 和 [Tom Looman Naming Convention Guide](https://tomlooman.com/unreal-engine-naming-convention-guide/)。两者都是推荐，部分前缀不同；用户明确要求及项目已有一致规范优先。此指南针对 Content 资产，不替代 C++ 标识符命名规则。

## 基本形式

`[AssetTypePrefix]_[AssetName]_[Descriptor]_[OptionalVariant]`。名称使用清晰的英文单词和一致的大小写；下划线分隔组成部分。描述符说明用途或类型，变体字母/编号仅在需要区分时添加；Tom 建议通用数字变体使用两位数，例如 `SM_Pipe_01`。名称无需为了填满模板而添加无意义字段。

示例：`SM_Pipe_Long`、`T_Pipe_Long_N`、`WBP_Inventory`、`ABP_Player`。UE 编辑器自动生成的关联资产、插件或第三方内容可保留其一致约定。

## 常用资产前缀

下表默认选 Epic 的推荐前缀；如果项目已采用 Tom 或其他一致方案，则继续遵循项目方案，不把差异当成错误。

| 类型 | 默认前缀 | 示例 / 说明 |
| --- | --- | --- |
| Blueprint | `BP_` | `BP_Enemy` |
| Actor Component | `AC_` | `AC_Inventory` |
| Blueprint Interface | `BI_` | Tom 使用 `BPI_` |
| Widget Blueprint | `WBP_` | Tom 使用 `Widget_`；Lyra 使用 `W_` |
| Enum / Structure 资产 | `E_` / `F_` | 与 C++ 类型的 `E` / `F` 前缀区分 |
| Data Table / Curve Table | `DT_` / `CT_` | |
| Static Mesh / Skeletal Mesh | `SM_` / `SK_` | |
| Material / Material Instance | `M_` / `MI_` | |
| Texture | `T_` | 按需添加下方纹理描述符 |
| Physics Asset / Physics Material | `PHYS_` / `PM_` | Tom 的物理材质使用 `_PhysMat` 后缀 |
| Animation Blueprint | `ABP_` | Tom 使用 `_AnimBP` 后缀 |
| Animation Montage / Sequence | `AM_` / `AS_` | Tom 的 Montage 使用 `_Montage` 后缀 |
| Skeleton / Blend Space | `SKEL_` / `BS_` | Tom 的 Skeleton 使用 `_SKEL` 后缀 |
| Niagara Emitter / System / Function | `FXE_` / `FXS_` / `FXF_` | Tom 的 Niagara System 使用 `NS_` |
| Level Sequence | `LS_` | |

Epic 列表并不穷尽所有类型；已有约定优先。若项目没有相应约定，可参考 Tom 的 `MF_`（Material Function）、`MPC_`（Material Parameter Collection）、`MSS_`（MetaSound Source）或 Lyra 的 `GA_`（Gameplay Ability）、`GE_`（Gameplay Effect）、`GCN_`（Gameplay Cue Notify），并保持一致；这些不应冒充上述 Epic 推荐表的条目。

## 纹理描述符

参考 Tom 的示例：`_D` 颜色/漫反射、`_N` 法线、`_E` 自发光、`_M` 遮罩、`_R` 粗糙度、`_MT` 金属度、`_AO` 环境光遮蔽。项目已有后缀和通道打包规范优先；对打包纹理明确通道含义，避免仅凭 `_M` 或 `_R` 猜测实际内容。

## 使用与变更

先核实现有项目同类资产的约定、来源和依赖，再建议名称；项目混用前缀时明确指出并提供一致选择，不自行做全项目整改。新建资产遵循已确定的约定；第三方内容保留其维护边界。

重命名既有资产前先备份受影响内容或确认可恢复的版本控制状态，检查引用、软路径、配置和存档；使用 UE 编辑器或经验证的 UE 资产工具，按范围检查并修复 [Asset Redirectors](https://dev.epicgames.com/documentation/unreal-engine/asset-redirectors-in-unreal-engine)，验证引用包、Blueprint/关卡和需要时的 Cook。不要混淆资产 Redirector 与 C++ 反射类型重命名使用的 Core Redirects；不能验证时如实说明。
