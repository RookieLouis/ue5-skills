---
name: ue5-epic-cpp-standard
description: "编写、修改、重构、Debug、Review 或解释 Unreal Engine C++（.h/.cpp、UCLASS/USTRUCT/UENUM/UINTERFACE、UFUNCTION/UPROPERTY、UE C++ API）时使用：按 Epic UE5.8 C++ Coding Standard 及标注的项目约定检查命名、注释、const、现代 C++ 与 Lambda 捕获、格式、依赖和 API 设计。纯 Blueprint 节点配置、资产/美术调整、与 UE C++ 无关的任务不要自动使用；职责划分、跨系统通信等架构问题使用 ue5-architecture。"
---

# UE5 Epic C++ Coding Standard

## 目标与适用范围

本 Skill 用于 Unreal Engine C++ 的编写、修改、重构、Debug、Code Review 和代码讲解，以 `references/Epic_CPP_Coding_Standard.md`（Epic UE5.8 C++ Coding Standard 导出副本）为规范依据。不要把一般性编程建议、项目约定或个人习惯冒充 Epic 原文规定。

- **适用**：`.h` / `.cpp`、`UCLASS`、`USTRUCT`、`UENUM`、`UINTERFACE`、`UFUNCTION`、`UPROPERTY`、Unreal C++ API、模块的 C++ 接口与实现。
- **不单独触发**：纯 Blueprint 节点配置、资产/美术调整、与 UE C++ 无关的任务；这些任务若同时涉及 C++ 代码，仍对 C++ 部分应用本 Skill。
- **分工**：职责划分、通信方式、数据组织、C++ / Blueprint 分工由 `ue5-architecture` 负责；本 Skill 只管代码层面的规范。

## 规则优先级（与 Custom Instructions 保持一致）

0. 编译器 / UHT / UBT 技术约束：不参与排序；用户或项目要求与之冲突时，说明原因并给出可行替代。
1. 用户当前任务的明确要求。
2. 项目专属规范与架构约定（含 `ue5-architecture`，以及本 Skill 中标注“项目约定”的条目）。
3. 本 Skill 的 Epic 原文摘要（Epic C++ Coding Standard）。
4. 其他通用 C++ 实践（不得冒充 Epic 规范）。

存在真实冲突时，指出冲突并给出解决方案，不得擅自忽略项目要求。

## 规范来源与阅读策略

- 完整原文：[`references/Epic_CPP_Coding_Standard.md`](references/Epic_CPP_Coding_Standard.md)，原样副本，不要修改。
- 默认只读本文件，不要为简单修改把 1,354 行原文全部读入上下文。
- 遇到争议、例外、复杂代码，或要给出“符合 Epic 标准”的结论时，按下方索引的行号读取对应章节全文，不要只依赖速查摘要。
- 原文是 UE5.8 时点的快照；与项目实际 UE 版本、引擎头文件、UHT 或编译器行为冲突时，先核实再说明区别。
- **规则文字优先于示例代码**。原文示例有笔误和旧 API，不要原样复制，例如：Range-Based For 示例的 `TFieldIterator<UProperty>`（UE5 反射属性已是 `FProperty`）、同节旧写法 `It.Key(), *It.Value()` 解引用写反、Const 示例的 `TArray<Int32>` 和缺少变量名的 `for (const FString& : StringArray)`。
- **面向引擎代码的条款**（版权声明、`//@UE5` 标记、`UE::` namespace、平台目录）用于项目代码前先确认是否适用。版权声明只要求 Epic 对外发布的源文件首行写 `// Copyright Epic Games, Inc. All Rights Reserved.`，项目自建文件不需要这一行。

### 原文索引（行号对应当前 references 文件，替换原文后需同步更新）

| 主题                                         | 章节                                                                  | 行号                  |
| ------------------------------------------ | ------------------------------------------------------------------- | ------------------- |
| 类成员排序、封装                                   | Class Organization / Encapsulation                                  | L66–87 / L1066–1073 |
| 版权声明                                       | Copyright Notice                                                    | L90–99              |
| 类型、变量、函数、布尔、宏命名                            | Naming Conventions                                                  | L102–264            |
| 用词                                         | Inclusive Word Choice                                               | L267–343            |
| 数值类型                                       | Portable C++ code                                                   | L346–377            |
| 标准库、容器、字符串                                 | Use of standard libraries                                           | L380–411            |
| 注释                                         | Comments / Guidelines / Example Formatting                          | L414–468 / L556–589 |
| const                                      | Const Correctness                                                   | L471–553            |
| C++20、static_assert、override/final、nullptr | Modern C++ Language Syntax 及子节                                      | L592–619            |
| auto、range-based for                       | Auto / Range-Based For                                              | L622–682            |
| Lambda 与捕获                                 | Lambdas and Anonymous Functions / Captures and Return Types         | L685–720            |
| enum、move、默认成员初始化                          | Strongly-Typed Enums / Move Semantics / Default Member Initializers | L723–840            |
| 第三方代码                                      | Third Party Code                                                    | L843–859            |
| 大括号、if-else、缩进、switch                      | Code Formatting 及子节                                                 | L862–968            |
| namespace、using                            | Namespaces                                                          | L971–1010           |
| include、前置声明、模块目录、inline                   | Physical Dependencies                                               | L1013–1063          |
| TEXT()、变量遮蔽、警告、字面量、static、diff 噪音          | General Style Issues                                                | L1076–1207          |
| bool 参数、参数列表、重载、接口、UObject 传参              | API Design Guidelines                                               | L1210–1304          |
| 平台相关代码                                     | Platform-Specific Code                                              | L1307–1354          |

## 规则强度

速查规则按原文用词标注强度，Code Review 按强度定级。编译 / UHT 错误和对象生命周期缺陷不在此表内，始终最先处理。

| 标签              | 原文用词                                                | Review 处理        |
| --------------- | --------------------------------------------------- | ---------------- |
| `[必须]` / `[禁止]` | must、never、not allowed、always、do not，以及 UHT / 编译器限制 | 必须修复             |
| `[应当]`          | should、avoid、prefer 及一般祈使句                          | 应修复；有正当理由时可保留并说明 |
| `[建议]`          | encourage、can、consider、use your best judgment       | 建议项，不阻塞          |

## 核心速查规则（Epic 原文摘要）

以下摘录原文；标注“项目约定”或“技术约束”的条目不是原文。细节与例外以对应章节为准。

### 命名（Naming Conventions / Inclusive Word Choice / Physical Dependencies）

- [必须] 类型名用 PascalCase 并加前缀：`T` 模板、`U` 继承 UObject、`A` 继承 AActor、`S` 继承 SWidget、`I` 抽象接口、`C` concept 类 struct、`E` 枚举，其余多数用 `F`。UHT 多数情况下依赖正确前缀。
- [必须] bool 变量加 `b` 前缀，如 `bPendingDestruction`。
- [建议] 按引用传入并由函数写回的参数加 `Out` 前缀；若该参数是 bool，`b` 放最前面：`bOutResult`。
- [应当] typedef 按目标类型加前缀（模板实例化的 typedef 不再算模板，如 `FArrayOfMyTypes`）；类型模板参数及基于它的嵌套别名不受前缀规则约束；模板参数与别名同名时用 `In` 前缀区分（`InElementType`）。
- [应当] 名称清晰、无歧义、有描述性，避免过度缩写，作用域越大越重要；类型和变量用名词，方法用动词描述其效果，或描述无副作用方法的返回值。
- [应当] 返回 bool 的函数用是非问句命名，如 `IsVisible()`、`ShouldClearBuffer()`；无返回值的过程用“强动词 + 宾语”；避免以 `Handle`、`Process` 开头的名字（动词含义模糊）。
- [应当] 每个变量单独一行声明。
- [应当] 宏名全大写、下划线分词、加 `UE_` 前缀（宏不能放进 namespace）。
- [应当] 文件名不加类型前缀：`Scene.cpp` 而不是 `UScene.cpp`。
- [应当] 标识符（类型、变量、函数、宏、文件名）使用美式英语拼写和语法（原文 L104）。注释语言见“### 注释”。
- [禁止] 使用强化刻板印象或指涉歧视的词，如 blacklist / whitelist / master / slave；替换词见 Word List。

### 类组织与封装（Class Organization / Encapsulation）

- [应当] 面向读者组织类声明：public 接口在前，private 实现在后（通常 public → protected → private）。
- [应当] 成员默认 private，除非属于 public/protected 接口；只供派生类使用的字段设为 private 并提供 protected 访问函数。
- [应当] 不打算被继承的类使用 `final`。

### 类型与现代 C++（Portable C++ / Modern C++ / Enums / Default Member Initializers）

- [应当] 语言基线为 C++20；编译器特有的语言特性只能包在预处理宏或条件中，并谨慎使用。
- [应当] 一律用 `nullptr`，不用 `NULL`。
- [必须] 重写虚函数时同时写 `virtual` 和 `override`。
- [应当] `auto` 只用于原文列出的三种情况：绑定 lambda 到变量；迭代器类型冗长且影响可读性；模板代码中表达式类型难以写出。使用时照样写 `const`、`&`、`*`。
- [应当] 不使用 structured binding。
- [应当] 优先用 range-based for；只读遍历用 const 引用。
- [应当] 用 `enum class` 取代旧式 namespace enum；`UPROPERTY` 直接用 enum class，不再用 `TEnumAsByte<>`。
- [必须] 暴露给 Blueprint 的 enum 以 `uint8` 为底层类型（UE 5.8 UHT 对其他底层类型直接报错）。
- [应当] Flag enum 定义值为 0 的 `None`，用 `ENUM_CLASS_FLAGS(EnumType)` 生成位运算符，并写成 `(Flags & EFlags::Flag1) != EFlags::None`。
- [应当] 序列化、复制等宽度重要的数据用 `int32`、`uint8` 等显式宽度类型；宽度无关时可以用 `int`。
- [禁止] 假设 `bool`、`TCHAR`、`PTRINT` 的大小。
- [建议] 默认成员初始化器在游戏代码中比在引擎代码中更合适；基类、UObject 子对象、依赖构造参数的值等不能这样初始化；避免同一类的默认值一半写在头文件、一半写在构造函数。

### 标准库（Use of standard libraries）

- [应当] 避免标准容器和字符串，互操作代码除外；同一 API 中不混用 UE 与标准库的写法。
- [应当] 新代码用 `<atomic>` 而非 `TAtomic`（旧代码改动时迁移）；与 UE trait 重叠时用 `<type_traits>`；`std::numeric_limits` 和 `<cmath>` 浮点函数可直接使用；`<regex>` 只能封装在 editor-only 代码中；`memcpy` / `memset` 仅在有可证明的性能收益时替代 `FMemory::Memcpy` / `FMemory::Memset`。
- [应当] 头文件中避免 include 标准库头文件。

### Const（Const Correctness）

- [应当] 不修改的参数用 const 指针或 const 引用传递；不修改对象的方法标 `const`；只读遍历用 const。
- [应当] 按值参数和局部变量也优先加 `const`，声明与定义保持一致；要 `MoveTemp` 移入容器或成员的按值参数例外。
- [应当] 指针本身为 const 时把 `const` 写在末尾：`T* const Ptr`。
- [禁止] 给返回类型本身加 `const`（如 `const TArray<FString> Get()`、`const T* const Get()`）；返回指向 const 的指针或引用不受影响。

### Lambda 与对象生命周期（Lambdas / Captures and Return Types）

- [应当] lambda 保持简短，作为谓词时不超过几条语句；非平凡 lambda 按普通函数写注释。
- [应当] 用显式捕获，不用 `[&]` / `[=]`；`[=]` 只要引用成员变量就会隐式捕获 `this`。
- [禁止] 延迟执行的 lambda 按引用捕获局部变量或成员变量。按值捕获指针（包括 `this`）在延迟执行时同样可能悬空，被捕获的 UObject 指针对 GC 不可见。
- [应当] 延迟执行使用 `CreateWeakLambda` / `CreateSPLambda`（委托上用 `BindWeakLambda`），对象释放后自动解绑；其他共享对象用 `TWeakObjectPtr` / `TWeakPtr` 捕获，并在 lambda 内校验。
- [必须] 不符合以上做法的延迟 lambda，注释说明捕获为何安全。
- [应当] 大型 lambda，或返回另一个函数调用结果的 lambda，写显式返回类型。

### API 设计（API Design Guidelines）

- [应当] UObject 用指针传递，不用引用；函数不接受 null 时在 API 中注明或妥善处理。
- [应当] 避免 bool 参数，尤其是作为标志的 bool，改用 flag enum。例外：bool 就是传入的完整状态时可以用，如 `SetEnabled(bool bEnabled)`。
- [应当] 避免过长的参数列表，改用专门的参数 struct。
- [应当] 不要同时按 `bool` 和 `FString` 重载同名函数：`Func(TEXT("String"))` 会调用 bool 版本。
- [必须] 接口类（`I` 前缀）不得有成员变量。
- [应当] 接口类始终为抽象类；非纯虚、非虚或 static 方法须在类内 inline 实现。

### 格式（Code Formatting）

- [必须] 大括号总是另起一行，不论函数或代码块多小（包括头文件中的单行访问函数）；单语句代码块也加大括号；`else if` 与首个 `if` 对齐。
- [应当] 行首缩进用 Tab，Tab 宽度设为 4；非 Tab 字符之后的对齐可以用空格。
- [必须] `switch` 总有 `default`，且 `default` 里也写 `break`。
- [应当] 非空 case 以 `break` / `return` / `continue` 结束，或注释 `// falls through`。
- [应当] 指针和引用符号紧贴类型，右侧只留一个空格：`FShaderType* Ptr`，不写 `FShaderType *Ptr` 或 `FShaderType * Ptr`。
- [禁止] 函数声明或调用处在函数名与 `(` 之间加空格。
- [应当] 文件末尾留一个空行。

### 注释（Comments / Example Formatting）

- [应当] 代码自解释；注释写意图，不复述代码，不与代码矛盾；坏代码应重写而不是多加注释。
- [应当] 方法注释只在 public 声明处写一次，只写调用者需要的信息（含与调用者相关的 override 说明）；实现细节写在实现内部。
- [应当] 类注释说明它解决什么问题、为什么创建；参数注释写明单位、取值范围、“不可能”的值和状态/错误码含义；需要时用 `@param`、`@return`、`@warning`、`@note`、`@see`、`@deprecated`，各占一行，写在主描述之后；函数唯一作用就是返回该值且主描述已说明时，不写 `@return`。
- [应当]（项目约定，替代原文 L104 对注释的要求）注释语言跟随用户在当前对话中使用的语言；用户另行指定时按指定。非英文注释中，UE 类型、API、宏、术语和代码标识符保留英文原文。只对新写和本次改动的注释生效，不批量翻译既有注释；标识符、`TEXT()` 字符串和日志文本不受本条影响。
- [必须]（技术约束）`@param`、`@return` 等标签保持英文，`@param` 后的参数名与实际参数一致：编辑器据此生成 Blueprint Pin 提示（`ObjectTools.cpp`）。
- [必须]（技术约束）注释含非 ASCII 字符时，源文件保存为 UTF-8：UBT 给 MSVC 传 `/utf-8`（`VCToolChain.cs`）。
- [应当]（项目约定）紧贴 `UCLASS` / `USTRUCT` / `UENUM` / `UPROPERTY` / `UFUNCTION` 的注释会被 UHT 写成编辑器 ToolTip，只写面向调用者/设计师的职责与契约；内部设计取舍写在 `.cpp`，必须留在头文件时用 UHT 忽略的 `//~`。
- [应当]（项目约定）类和 public / protected API 写职责注释，自解释的 override 和简单访问函数不强制写；函数体注释只在读者无法从代码看出原因时写“为什么”，例如 workaround 及其原因、执行顺序或生命周期依赖、性能取舍、为兼容 Blueprint / 已序列化数据保留的写法。

### 依赖、模块与 namespace（Physical Dependencies / Namespaces）

- [应当] 头文件用 `#pragma once`。
- [应当] 优先前置声明而不是 include；include 尽量细粒度（例如不 include `Core.h`）；直接 include 所需的每个头文件，不依赖间接 include。
- [必须] 其他模块需要的定义放在 `Public` 目录的头文件中；其余放 `Private`。
- [应当] 大函数拆成合乎逻辑的子函数；inline 只用于简单访问函数或有 profiling 证据的场景；谨慎使用 `FORCEINLINE`。
- [必须] UCLASS、USTRUCT 等反射类型不能定义在 namespace 中（UHT 不支持 namespace）。
- [禁止] 在全局作用域写 `using` 声明，`.cpp` 中也不行（会影响 Unity Build）；可以写在其他 namespace 或函数体内。
- [必须] 前置声明必须写在类型所属的 namespace 内，否则会出现链接错误。
- [应当] 非反射的新 API 放入 `UE::` 下的嵌套 namespace，实现细节放 `Private` 子 namespace（如 `UE::Audio::Private::`）。这是面向引擎的条款，项目代码是否沿用 `UE::` 先确认项目约定。

### 通用风格（General Style Issues）

- [必须] 不允许变量遮蔽。UE 5.8 中 BuildSettingsVersion ≥ V2 的 Target 默认 `ShadowVariableWarningLevel = Error`，按编译问题处理。
- [应当] 处理编译警告；确实无法解决时才用 `#pragma` 屏蔽，作为最后手段。
- [必须] 字符串字面量总是用 `TEXT()` 包裹。已知例外：`UE_LOGFMT` 的格式串和字段名按其 API 使用普通字面量（见 UE 5.8 `Logging/StructuredLog.h`）。
- [应当] 不在循环中重复相同运算，把公共子表达式移到循环外；必要时用 static 避免跨调用的重复工作，如用字符串字面量构造 `FName`。
- [应当] 复杂表达式拆成有描述性名字的中间变量，如 `const bool bIsPlayerDead = ...;`。
- [应当] 调用处避免匿名字面量，改用说明含义的具名常量。
- [应当] 头文件中不定义非平凡 static 变量，改为头文件 `extern` 声明 + `.cpp` 定义。
- [应当] 变量在临近使用处赋值，缩短依赖距离；方法尽量拆成命名清晰的子方法。
- [应当] 调试代码要么完善可用，要么不提交。
- [应当] 避免不改变行为的大范围修改（空白、批量重命名私有变量）；确有必要时单独提交，不与行为修改混在一起；小的规范问题只在改动行或附近代码顺手修正。

### 平台与第三方代码（Platform-Specific Code / Third Party Code）

- [应当] 平台相关代码放在对应平台目录，并通过 HAL（如 `FPlatformMisc`）抽象；避免在平台目录外新增 `PLATFORM_[PLATFORM]` 判断，改用描述特性的 define（如 `PLATFORM_USE_PTHREADS`）。
- [应当] 修改第三方库代码时加 `//@UE5` 注释并写明原因；第三方代码用 `// @third party code - BEGIN …` / `END …` 标记。

## 项目约定（非 Epic 原文）

- 用用户的语言解释 C++/UE 语义；Unreal API、类名和技术术语保留英文（Actor、Component、Projectile 等），不生硬翻译。
- 注释语言跟随用户语言，规则见“### 注释”中标注“项目约定”的条目（替代原文 L104 对注释的要求）。
- 断言（`check` / `ensure`）、日志（`UE_LOGFMT`）和失败可见性策略以 `ue5-architecture` 的 `references/ue5-implementation.md` 为准，不要标成 Epic 原文要求。
- 项目还没有 C++ 模块时，不要自行新增；新增模块是项目级决策，先说明理由和成本，经用户确认后再执行（见 `ue5-architecture`）。

## 执行步骤

1. **确认范围**：识别相关 `.h/.cpp`、原有项目惯例、UE 版本、修改目的和已有行为。先读必要的上下文，再提出改动。
2. **实施或审查**：先处理正确性、UHT/编译、对象生命周期（含 Lambda 规则）和用户的明确需求，再按速查规则检查命名、可读性、依赖、API 和格式。
3. **最小化修改**：新写和本次改动的代码遵循规范；既有代码只在改动行附近修正，不做全文件重排、批量重命名或纯格式化。
4. **按需查证**：非显而易见的规则按索引读原文章节；版本敏感的 UE API 核对实际引擎源码或文档。
5. **验证**：环境允许时运行 UBT 编译（含 UHT）和相应测试，修复相关 warnings；未执行编译/测试时必须明确说未验证，不能宣称“编译通过”。
6. **反馈**：说明实质性改动、规则依据（强度标签 + 原文章节）、风险和剩余验证工作。

## 处理不同任务

**生成代码**

- 先给能集成进项目的最小方案；头文件与源文件配套，写明前置声明和必要的 include。
- 不虚构项目模块名、现有成员、UE API 签名或构建状态；无法确认的条件明确标注。
- 只在需求确实需要时引入新抽象，不为“合规”引入模板、模块或框架；架构取舍交给 `ue5-architecture`。

**重构**

- 默认保持可观察行为，识别 Blueprint / UHT 绑定和公开 API 的兼容风险；重命名反射类型、属性、函数或枚举值会影响已序列化的资产和 Blueprint，需要 CoreRedirects（见 `ue5-architecture` 的 `references/bp-to-cpp-migration.md`）。
- 区分“修复规范问题”和“改变设计/行为”，分开提交或分开展示。

**Code Review**

- 顺序：正确性、编译 / UHT、UObject 生命周期 → 架构问题（涉及多个类或系统的职责与依赖时按 `ue5-architecture` 检查）→ Coding Standard（按强度标签定级）。
- 每条结论给出：位置（文件/行号）、问题、风险、强度标签与原文章节、最小修复建议；没有证据时不武断报错。
- 不把项目约定（如 `UE_LOGFMT`、`ensureMsgf`、`checkf` 的使用策略）标成 Epic 原文要求。

## 完成前检查

- [ ] 修改满足用户需求，没有无意中改变行为。
- [ ] Lambda 捕获、委托回调与 UObject 生命周期安全。
- [ ] 类型前缀、`b` / `Out` 前缀、`const`、`virtual` + `override`、`final`、`enum class`、`nullptr`、`auto` 符合规则。
- [ ] 大括号、Tab、指针/引用位置、`switch` 的 `default`、`TEXT()` 符合规则。
- [ ] include、前置声明、Public/Private 位置、namespace 与 UHT 约束合理。
- [ ] 没有无关的大范围格式化、无端抽象或多余 include。
- [ ] 依据不明确的判断已按索引查阅原文；Review 结论标注了强度与章节。
- [ ] 如实说明编译/测试结果，未验证的写明“未验证”。
