# Unreal Engine C++ 实现与质量规则

## Engine 优先

- 尊重项目实际 UE 版本、模块和 C++ 标准。面向 UE5.8 开发时仍应核实本机引擎源码/官方 API，尤其是 Plugin、Experimental / Beta 功能。
- 优先使用 UE 原生 `AActor`、`UActorComponent`、`UObject`、`USTRUCT`、`UInterface`、`Delegate`、`Subsystem`、`DataAsset`、`DataTable` 和反射宏；不照搬与 UE 生命周期冲突的传统 C++ 框架。
- 修改 Build.cs 依赖要有理由；避免为了一个小功能造成循环模块依赖。
- 不为了“统一”把所有 `Tick` 改成事件；检查实时连续更新需求。相反，状态变化通知通常不应靠每帧轮询。

## 对象引用 / 生命周期

- 明确 Actor Owner、Outer、World、生命周期范围、GC 与软硬引用区别。
- 持久 UObject 引用应按 UE 反射 GC 规则保存；弱引用、Soft Pointer 用在对应不持有与按需加载场景。
- Delegate、Timer、Async 请求、Level Unload 和 EndPlay 要考虑回调目标有效性，避免重复注册。
- 游戏逻辑严格区分“可加载的静态定义”和“运行时可变状态”；切关卡时不要让 World 作用域 Actor 失效后仍被长期服务访问。

## 断言、日志和失败可见性（项目约定）

以下是本项目的约定，不是 Epic Coding Standard 原文要求；代码风格以 `ue5-epic-cpp-standard` Skill 为准。

- `check/checkf`：开发期不变量绝对必须成立，违反后不应继续；**不可**用于玩家输入、网络包、Save 文件等可恢复错误。
- `ensure/ensureMsgf`：本应成立但仍可继续的异常，保留明确诊断；不要把 `ensure` 当作正常的 `if` 条件分支。
- `UE_LOGFMT`：诊断运行时可恢复失败、资源缺失、上下文状态；必要时定义项目 Log Category、标明 ItemId/Owner/World。
- 失败必须有合理行为：例如 TryAddItem 返回失败而非只写日志、静默略过。校验所有关键 Blueprint 指派/配置。
- `UE_LOGFMT` 的 Header 通常需要 `#include "Logging/StructuredLog.h"`；格式化参数/宏签名按项目 Engine 版本核实。

示意：

```cpp
if (!IsValid(ItemDefinition))
{
    UE_LOGFMT(LogTemp, Error, "Item definition is invalid for pickup {PickupName}",
        ("PickupName", GetNameSafe(this)));
    return false;
}
```

`ensureMsgf` 的用途是暴露“设计上应该配置却没有配置”的异常；用户可恢复、常见的库存满等状态不应触发断言。

## 可验证实现

- UI 事件：验证只在成功变更时广播、监听者重复创建不重复订阅、取消监听时不遗留失效回调。
- DataTable：验证无效 RowName、空表、重新导入、Soft Asset 未加载、失败回退。
- DataAsset：验证同一 Definition 被两个物品实例引用时 Runtime 状态不会串改。
- Composition/Inheritance：验证基类初始化、Owner/Component 生命周期、销毁/EndPlay 顺序。
- Pool：按 [object-pooling.md](object-pooling.md) 的完整状态复位清单做压力测试。
- 性能：只在采样数据支持时做池化/复杂缓存方案，并比较帧时间而非只比较代码感觉。
- 具体测试可优先使用 UE Automation Tests、功能测试、PIE，能跑则跑；不能跑则明确“未验证”。

## 与 AI 编辑器工具协作

- 如果 Aura Editor Agent、Python、MCP 能访问 UE Editor，可核实 Blueprint Class、Asset、DataTable Schema、引用关系和编辑配置再动手。
- 工具不可用时只输出可执行的 Editor 操作清单或说明哪些部分需人工检查；不得假装已经打开 Editor 修改过资产。
- 对从 Fab 获取的系统尽量在边界增加 Adapter/独立 C++ Layer，记录 Vendor 升级和蓝图引用迁移成本。
