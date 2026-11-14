---
name: ue5-epic-cpp-standard
description: Apply the Epic UE 5.8 C++ Coding Standard when writing, changing, refactoring, debugging, reviewing, or explaining Unreal Engine C++; use the asset naming guide when planning, naming, or reviewing UE asset names and prefixes. Do not automatically use for Blueprint-only node configuration or art changes without asset naming. Use ue5-architecture for Gameplay architecture and Content directory organization.
---

# UE5 Epic C++ Coding Standard

## Purpose and Scope

Use this skill for writing, changing, refactoring, debugging, reviewing, and explaining Unreal Engine C++. Its standard is the exported UE 5.8 snapshot at [Epic_CPP_Coding_Standard.md](references/Epic_CPP_Coding_Standard.md). Do not pass off generic programming advice, project conventions, or personal preferences as Epic's requirements.

- Applies to `.h` / `.cpp`, `UCLASS`, `USTRUCT`, `UENUM`, `UINTERFACE`, `UFUNCTION`, `UPROPERTY`, UE C++ APIs, and module C++ interfaces and implementations.
- For UE Content asset names, read the [asset naming guide](references/asset-naming-guide.md) as needed. Do not apply the C++ workflow, strength labels, or compilation checks to asset-naming-only tasks. Asset naming and C++ identifier naming are separate conventions.
- Do not trigger this skill solely for Blueprint node configuration or art changes without asset naming. Apply it to any C++ portion of a mixed task.
- `ue5-architecture` covers responsibilities, communication, data, Content directories, and the C++ / Blueprint split; this skill covers C++ standards and asset naming.

## Priority of Rules

Compiler, UHT, and UBT constraints are outside the ranking. Explain conflicts and provide feasible alternatives. Otherwise:

1. Explicit instructions for the current task.
2. Project-specific standards and architecture conventions, including `ue5-architecture` and items marked as project conventions in this skill.
3. This skill's summary of the Epic C++ Coding Standard.
4. Other general C++ practices, which must not be presented as Epic rules.

Identify real conflicts and propose a resolution; do not silently disregard project requirements.

## Sources and Reading Strategy

- The [asset naming guide](references/asset-naming-guide.md) gives its sources and default prefixes. It is independent of the C++ Coding Standard and not part of the Epic C++ summary below.
- The complete original is the unchanged [reference snapshot](references/Epic_CPP_Coding_Standard.md). Do not modify it.
- For Chinese, see [Epic's official UE 5.8 Chinese documentation](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/epic-cplusplus-coding-standard-for-unreal-engine?application_version=5.8). The online text has not been verified as a line-by-line translation of the included snapshot; use the local English snapshot for the rule basis and line numbers.
- Read this skill by default; do not load all 1,354 lines of the original for a simple change.
- For disputes, exceptions, complex code, or a conclusion that code complies with Epic's standard, read the relevant full section using the index below rather than relying solely on the summary.
- The snapshot represents UE 5.8. Verify and explain discrepancies with the actual engine version, headers, UHT, or compiler.
- **Rule text takes precedence over examples.** Some original examples contain typos or old APIs. Do not copy them blindly: e.g. the range-based-for example's `TFieldIterator<UProperty>` (UE5 uses `FProperty`); the old `It.Key(), *It.Value()` example has the dereference reversed; the const examples contain `TArray<Int32>` and `for (const FString& : StringArray)` without a variable name.
- Check applicability to project code before applying engine-oriented rules on copyright, `//@UE5`, the `UE::` namespace, and platform directories. Epic's copyright line is required for source files Epic distributes publicly, not project-authored files.

### Original Text Index

Line numbers refer to the included reference file and must be updated if it is replaced.

| Topic | Section | Lines |
| --- | --- | --- |
| Class ordering, encapsulation | Class Organization / Encapsulation | L66–87 / L1066–1073 |
| Copyright | Copyright Notice | L90–99 |
| Type, variable, function, bool, macro names | Naming Conventions | L102–264 |
| Word choice | Inclusive Word Choice | L267–343 |
| Numeric types | Portable C++ code | L346–377 |
| Standard libraries, containers, strings | Use of standard libraries | L380–411 |
| Comments | Comments / Guidelines / Example Formatting | L414–468 / L556–589 |
| const | Const Correctness | L471–553 |
| C++20, static_assert, override/final, nullptr | Modern C++ Language Syntax and subsections | L592–619 |
| auto, range-based for | Auto / Range-Based For | L622–682 |
| Lambdas and captures | Lambdas and Anonymous Functions / Captures and Return Types | L685–720 |
| enum, move, default member initializers | Strongly-Typed Enums / Move Semantics / Default Member Initializers | L723–840 |
| Third-party code | Third Party Code | L843–859 |
| Braces, if-else, indentation, switch | Code Formatting and subsections | L862–968 |
| namespace, using | Namespaces | L971–1010 |
| include, forward declarations, module directories, inline | Physical Dependencies | L1013–1063 |
| TEXT(), shadowing, warnings, literals, static, diff noise | General Style Issues | L1076–1207 |
| bool parameters, parameter lists, overloads, interfaces, UObject parameters | API Design Guidelines | L1210–1304 |
| Platform-specific code | Platform-Specific Code | L1307–1354 |

## Rule Strength

Quick-reference labels reflect the original wording; use them to rate reviews. Compilation/UHT errors and object lifetime defects are outside this table and always come first.

| Label | Original wording | Review handling |
| --- | --- | --- |
| `[MUST]` / `[MUST NOT]` | must, never, not allowed, always, do not; UHT/compiler constraints | Must fix |
| `[SHOULD]` | should, avoid, prefer, and ordinary imperatives | Should fix; may retain with a justified explanation |
| `[RECOMMENDED]` | encourage, can, consider, use your best judgment | Non-blocking recommendation |

## Core Quick Reference: Epic Standard Summary

Items marked project convention or technical constraint are not Epic's original requirements. Consult the full sections for details and exceptions.

### Naming

- [MUST] Use PascalCase and a type prefix: `T` templates, `U` UObject-derived, `A` AActor-derived, `S` SWidget-derived, `I` abstract interfaces, `C` concept-like structs, `E` enums, and generally `F` for others. UHT usually relies on correct prefixes.
- [MUST] Prefix bool variables with `b`, e.g. `bPendingDestruction`.
- [RECOMMENDED] Prefix reference output parameters with `Out`; bool outputs place `b` first, e.g. `bOutResult`.
- [SHOULD] Prefix typedefs for their target type; a typedef of a template instance is no longer a template (e.g. `FArrayOfMyTypes`). Type template parameters and aliases based on them are exempt; distinguish a same-named parameter with `In`, e.g. `InElementType`.
- [SHOULD] Choose clear, unambiguous, descriptive names and avoid excessive abbreviations, especially in broad scopes. Use nouns for types and variables, verbs for methods' effects, or descriptions of the return value for side-effect-free methods.
- [SHOULD] Name bool-returning functions as yes/no questions (`IsVisible()`, `ShouldClearBuffer()`); use strong verb + object for procedures. Avoid vague `Handle` and `Process` prefixes.
- [SHOULD] Declare each variable on its own line. Use uppercase underscore-separated macro names with `UE_` prefix. Omit type prefixes from filenames (`Scene.cpp`). Use US English spelling and grammar for identifiers; see Comments for comment language.
- [MUST NOT] Use terms that reinforce stereotypes or refer to discrimination, such as blacklist / whitelist / master / slave; see Epic's Word List for alternatives.

### Class Organization and Encapsulation

- [SHOULD] Organize declarations for the reader: public interface first, private implementation last, generally public → protected → private.
- [SHOULD] Members are private by default unless part of the public/protected interface. Make fields needed only by derived classes private and expose protected accessors.
- [SHOULD] Mark classes not intended for inheritance `final`.

### Types and Modern C++

- [SHOULD] Use C++20 as the language baseline. Guard compiler-specific features with preprocessor macros or conditions and use them sparingly. Use `nullptr`, not `NULL`.
- [MUST] Write both `virtual` and `override` when overriding a virtual function.
- [SHOULD] Use `auto` only for binding a lambda, an unwieldy iterator type that hurts readability, or an expression whose type is difficult to write in template code. Still write `const`, `&`, and `*` as needed. Do not use structured bindings.
- [SHOULD] Prefer range-based for; use const references for read-only iteration. Prefer `enum class` to legacy namespace enums and use it directly in `UPROPERTY` instead of `TEnumAsByte<>`.
- [MUST] Blueprint-exposed enums use `uint8` as the underlying type; UE 5.8 UHT rejects other underlying types.
- [SHOULD] Define `None` = 0 for flag enums, generate bitwise operators with `ENUM_CLASS_FLAGS(EnumType)`, and test with `(Flags & EFlags::Flag1) != EFlags::None`.
- [SHOULD] Use fixed-width types such as `int32` or `uint8` when width matters, including serialization and replication; `int` is acceptable when it does not.
- [MUST NOT] Assume the sizes of `bool`, `TCHAR`, or `PTRINT`.
- [RECOMMENDED] Default member initializers are generally more suitable in game code than engine code. They cannot initialize base classes, UObject subobjects, or values dependent on constructor arguments; avoid splitting a class's defaults between the header and constructor.

### Standard Libraries

- [SHOULD] Avoid standard containers and strings except for interoperability; do not mix UE and standard-library styles in one API.
- [SHOULD] Use `<atomic>` instead of `TAtomic` in new code and migrate old code when changing it; use `<type_traits>` where it overlaps UE traits. `std::numeric_limits` and `<cmath>` floating-point functions may be used. Wrap `<regex>` in editor-only code. Replace `FMemory::Memcpy` / `FMemory::Memset` with `memcpy` / `memset` only for demonstrated performance benefit.
- [SHOULD] Avoid including standard-library headers in headers.

### Const

- [SHOULD] Pass unmodified parameters by const pointer/reference, mark non-mutating methods `const`, and use const for read-only iteration.
- [SHOULD] Prefer const even for by-value parameters and local variables and keep declaration and definition consistent. Exception: by-value parameters moved into a container or member with `MoveTemp`.
- [SHOULD] Place pointer constness last: `T* const Ptr`.
- [MUST NOT] Put `const` on the return type itself, e.g. `const TArray<FString> Get()` or `const T* const Get()`. Returning a pointer or reference to const is allowed.

### Lambdas and Object Lifetime

- [SHOULD] Keep lambdas short; predicate lambdas should have no more than a few statements. Document non-trivial lambdas like ordinary functions.
- [SHOULD] Use explicit captures, not `[&]` / `[=]`. Referencing a member under `[=]` implicitly captures `this`.
- [MUST NOT] Capture local variables or members by reference in deferred lambdas. Capturing pointers by value, including `this`, can also dangle; captured UObject pointers are invisible to GC.
- [SHOULD] For deferred work use `CreateWeakLambda` / `CreateSPLambda` (or delegate `BindWeakLambda`) so bindings are released with the target; capture other objects using `TWeakObjectPtr` / `TWeakPtr` and validate inside the lambda.
- [MUST] If a deferred lambda does not follow these practices, document why its captures are safe.
- [SHOULD] Give large lambdas, or lambdas returning another function's result, an explicit return type.

### API Design

- [SHOULD] Pass UObjects by pointer, not reference; document or handle null where the API does not accept it.
- [SHOULD] Avoid bool parameters, particularly flags; prefer flag enums. A bool is acceptable when it represents the complete state, e.g. `SetEnabled(bool bEnabled)`.
- [SHOULD] Replace long parameter lists with a dedicated struct. Do not overload a name for both `bool` and `FString`: `Func(TEXT("String"))` will select the bool overload.
- [MUST] Interface classes (`I` prefix) have no member variables.
- [SHOULD] Interface classes are abstract; non-pure virtual, non-virtual, and static methods must be implemented inline in the class.

### Formatting

- [MUST] Put braces on their own lines for all functions and blocks, including one-line header accessors. Brace single-statement blocks and align `else if` with the first `if`.
- [SHOULD] Use tabs for leading indentation with width 4; spaces may align after non-tab characters.
- [MUST] Every `switch` has `default` with a `break`.
- [SHOULD] End nonempty cases with `break`, `return`, `continue`, or `// falls through`.
- [SHOULD] Attach pointer/reference symbols to the type and leave one space after: `FShaderType* Ptr`.
- [MUST NOT] Insert a space between a function name and `(` in declarations or calls.
- [SHOULD] Leave a blank line at end of file.

### Comments

- [SHOULD] Make code self-explanatory. Explain intent, do not restate or contradict code, and rewrite bad code instead of compensating with comments.
- [SHOULD] Document methods once at the public declaration, giving information callers need, including relevant override details. Put implementation details in the implementation.
- [SHOULD] Class comments describe the problem solved and why the class exists. Document units, ranges, impossible values, and state/error-code meanings. Use `@param`, `@return`, `@warning`, `@note`, `@see`, and `@deprecated` as appropriate, one per line after the main description. Omit `@return` when the function solely returns the value already described.
- [SHOULD, project convention replacing original L104 for comments] Match comments to the user's current conversation language unless otherwise requested. Retain English UE types, APIs, macros, terms and identifiers in non-English comments. Apply only to new or changed comments; do not bulk-translate existing comments. This does not govern identifiers, `TEXT()` strings, or log text.
- [MUST, technical constraint] Keep tags such as `@param` and `@return` in English and match parameter names: the Editor uses them for Blueprint pin tooltips (`ObjectTools.cpp`).
- [MUST, technical constraint] Save source with non-ASCII comments as UTF-8: UBT passes `/utf-8` to MSVC (`VCToolChain.cs`).
- [SHOULD, project convention] Comments directly attached to reflected declarations become UHT Editor ToolTips: state caller/designer-facing responsibility and contract. Put internal design decisions in `.cpp`, or use UHT-ignored `//~` when they must remain in the header.
- [SHOULD, project convention] Document responsibilities of classes and public/protected APIs; self-explanatory overrides and simple accessors do not require comments. In function bodies explain why only when it is not apparent: workarounds, execution order, lifecycle dependencies, performance tradeoffs, or Blueprint/serialized-data compatibility.

### Dependencies, Modules, and Namespaces

- [SHOULD] Use `#pragma once` in headers; prefer forward declarations over includes; use granular includes rather than `Core.h`; include every directly needed header instead of relying on transitive includes.
- [MUST] Put definitions needed by other modules in headers under `Public`; put the rest under `Private`.
- [SHOULD] Break large functions into logical subfunctions. Inline simple accessors or where profiling supports it; use `FORCEINLINE` cautiously.
- [MUST] Do not define reflected types such as UCLASS/USTRUCT inside a namespace; UHT does not support it.
- [MUST NOT] Put `using` declarations at global scope, including in `.cpp` files, because of Unity builds. They may be inside another namespace or a function.
- [MUST] Put forward declarations inside the type's namespace to avoid linker errors.
- [SHOULD] Put new non-reflected APIs in nested namespaces under `UE::`, with implementation details in a `Private` subnamespace, e.g. `UE::Audio::Private::`. This is an engine-oriented rule: check project conventions before using `UE::` in project code.

### General Style

- [MUST] Do not shadow variables. In UE 5.8, Targets with BuildSettingsVersion ≥ V2 default to `ShadowVariableWarningLevel = Error`; treat this as a compilation issue.
- [SHOULD] Address warnings; use `#pragma` suppression only as a last resort when a warning cannot be resolved.
- [MUST] Wrap string literals in `TEXT()`. Known exception: use ordinary literals for `UE_LOGFMT` format strings and field names as required by its API (UE 5.8 `Logging/StructuredLog.h`).
- [SHOULD] Move repeated common expressions outside loops; use static where appropriate to avoid repeated work across calls, such as constructing an `FName` from a string literal.
- [SHOULD] Split complex expressions into descriptively named intermediates, e.g. `const bool bIsPlayerDead = ...;`, and replace anonymous literals at call sites with named constants.
- [SHOULD] Do not define non-trivial static variables in headers; use an `extern` declaration and a `.cpp` definition. Assign variables near use; split methods into clearly named submethods. Debug code should either be polished and useful or not submitted.
- [SHOULD] Avoid broad behavior-preserving changes, such as whitespace or mass private-variable renames. If necessary, submit separately from behavior changes. Fix small style issues only at or near changed lines.

### Platform and Third-Party Code

- [SHOULD] Put platform-specific code in platform directories and abstract via HAL such as `FPlatformMisc`. Avoid new `PLATFORM_[PLATFORM]` checks outside those directories; use feature defines such as `PLATFORM_USE_PTHREADS`.
- [SHOULD] Annotate modifications to third-party libraries with `//@UE5` and the reason; delimit third-party code with `// @third party code - BEGIN …` / `END …`.

## Project Conventions

- Explain C++ and UE semantics in the user's language while retaining Unreal API names, class names, and technical terms such as Actor, Component, and Projectile in English.
- Follow the comment language and scope rules above; they replace the original L104 requirement for comments.
- Assertion (`check` / `ensure`), logging (`UE_LOGFMT`), and failure visibility policy comes from `ue5-architecture`'s [ue5-implementation.md](../ue5-architecture/references/ue5-implementation.md), not Epic's original standard.
- Do not add a C++ module on your own to a project without one. This is a project-level decision: explain rationale and cost and obtain user confirmation first.

## Workflow

1. Identify relevant `.h/.cpp`, existing conventions, UE version, purpose, and behavior. Read necessary context before proposing changes.
2. Implement or review: address correctness, UHT/compilation, object lifetimes including lambdas, and explicit user requirements first; then naming, readability, dependencies, API, and formatting.
3. Keep changes minimal. Apply rules to new and changed code; fix old code only near changed lines, without wholesale rearrangement, mass renaming, or formatting.
4. Consult indexed original sections for non-obvious rules and actual engine source or documentation for version-sensitive APIs.
5. Run UBT including UHT and relevant tests when possible; fix related warnings. Explicitly state when compilation or tests were not run; do not claim a successful build.
6. Report substantive changes, rule basis including strength and original section, risks, and remaining verification.

## Task-Specific Guidance

**Code generation:** Provide a minimal integrable solution with matching header and source, forward declarations, and required includes. Do not invent module names, members, UE API signatures, or build status; identify unknowns. Add abstractions only when needed, not templates, modules, or frameworks merely for compliance; leave architecture to `ue5-architecture`.

**Refactoring:** Preserve observable behavior by default and identify Blueprint/UHT bindings and public API compatibility risks. Renaming reflected types, properties, functions, or enum values affects serialized assets and Blueprints and requires CoreRedirects; see `ue5-architecture/references/bp-to-cpp-migration.md`. Separate or separately present standards fixes and design/behavior changes.

**Code review:** Check correctness, compilation/UHT and UObject lifetime → architecture, using `ue5-architecture` for responsibilities/dependencies across classes or systems → Coding Standard, rated by strength. For each finding give location, issue, risk, strength and original section, and minimal fix. Do not assert defects without evidence. Do not label project policies such as `UE_LOGFMT`, `ensureMsgf`, or `checkf` as Epic's original requirements.

## Completion Checklist

- [ ] Requirements are met without unintended behavior changes.
- [ ] Lambda captures, delegate callbacks, and UObject lifetime are safe.
- [ ] Type, `b` / `Out` prefixes, `const`, `virtual` + `override`, `final`, `enum class`, `nullptr`, and `auto` follow the rules.
- [ ] Braces, tabs, pointer/reference placement, `switch` default, and `TEXT()` follow the rules.
- [ ] Includes, forward declarations, Public/Private placement, namespaces, and UHT constraints are appropriate.
- [ ] No unrelated broad formatting, unnecessary abstractions, or extra includes.
- [ ] Unclear judgments were checked against the original; review findings include strength and section.
- [ ] Build/test results are reported truthfully, with unverified items identified.
