# Blueprint → C++ 迁移

适用：把现有 Blueprint 系统（包括第三方 BP 框架）的权威逻辑和数据类型逐步下沉到 C++。前提是项目已有 C++ 模块；没有时按 SKILL.md 第 1 步先征得用户确认。

## 原则

- **保留资产，换父类**：为 BP 新建 C++ 基类，再把 BP Reparent 到它，而不是删除重建；关卡实例、DataAsset 引用和 BP 子类因此得以保留。
- **一次只迁一个单元**：一个 Component，或一组 struct/enum。每一步都能编译、能跑 PIE、能单独回滚。
- **先迁被依赖的类型**：C++ 无法把 UserDefinedStruct / UserDefinedEnum 当作类型使用，所以先迁 struct/enum，再迁 Component，最后迁 Character、Ability 等上层类型。
- **对 Blueprint 的接口保持不变**：函数名、参数、Event Dispatcher 名称不变；必须改名时用 CoreRedirects。

## 步骤

1. 用 Reference Viewer 列出迁移单元的所有引用者，记录 BP 中对它的调用、变量读写和事件绑定。
2. 编写 C++ 基类或类型，按 [balancing-cpp-blueprint.md](balancing-cpp-blueprint.md) 的暴露规则设置 `UPROPERTY` / `UFUNCTION`。
3. 把 BP Reparent 到 C++ 基类，逐个用 C++ 成员替换 BP 变量和函数，再删除 BP 中的旧定义。
4. 打开并编译所有引用者，修复断开的节点。
5. PIE 回归：覆盖该单元相关的核心用例，以及存档加载。

## 风险点

- **同名冲突**：C++ 父类新增的属性或函数与 BP 子类已有成员同名时会冲突。迁移前规划命名，替换后检查所有引用。
- **UserDefinedStruct / UserDefinedEnum 换成 native 类型**：引用它们的 BP、DataTable、DataAsset 中已保存的值不会自动对应到新类型，可能丢失默认值和配置。先在少量资产上确认数据完整，再批量替换；替换前提交版本控制。
- **CoreRedirects**：类、属性、函数、struct、enum 改名或移动时，在 `Config/DefaultEngine.ini` 的 `[CoreRedirects]` 段添加对应条目，重新保存引用资产后确认生效。
- **第三方框架**：直接修改 Vendor BP 后无法再直接合并上游更新；记录被修改的资产清单。
- **存档兼容**：涉及 SaveGame 中的类型或字段时，验证旧存档仍能加载。

## 项目实测（UE 5.8，项目约定，非 Epic 原文）

以下来自本项目 Attributes 第一、二阶段迁移；升级引擎版本后需重新核实。

- **UserDefinedStruct → USTRUCT**：`StructRedirects` 未生效，改用 `ObjectRedirects`；旧成员名带 GUID 后缀，用 `PropertyRedirects` 映射到新成员名，结构体节点引脚才能匹配。
- **UserDefinedEnum → UENUM**：用 `EnumRedirects` + `ValueChanges`；原生枚举按旧枚举的**数值**排列，不按显示顺序。
- **默认值与精度**：原生类型的默认值必须与旧 UDS 一致；旧成员为 real/double 时 C++ 用 `double`。
- **编译加载**：新增反射类型不要依赖 Live Coding，关闭编辑器完整编译后再重启。
- **Tick**：BP 组件改挂到自身不 Tick 的原生中间父类后，该父类 UCLASS 需加 `meta=(ChildCanTick)`，否则 BP 子类的 Event Tick 会被禁用。
- **删除旧 UDS**：关闭编辑器后离线删除；5.8 中用 Python 删除可能崩溃。
- **快照范围**：迁移前后的快照除 BP 变量和默认值外，还要覆盖蒙太奇中的 AnimNotify 实例、关卡 Actor 实例和 Widget 树控件。

## 验收

- 所有引用者编译无错误。
- 迁移前后，同一组 PIE 用例的行为一致。
- 这一步迁移可以在版本控制中单独回滚。
