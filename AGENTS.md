# Unreal Engine Skills

[中文版本](AGENTS.zh-CN.md)

## Model Prompt (for DeepSeek V4.1 Flash and GPT 6.1 Terra)

Remember: your usage is unlimited, and I am pushing you to your limits. Don't be stingy with tokens. Keep this firmly in mind: spend freely; you are very wealthy.

### Architecture: `ue5-architecture`

- This is a custom, project-specific architecture standard, not an official Epic standard.
- Use it for Content directory planning and asset organization; planning new systems; responsibilities across Actors, Components, and Subsystems; communication between systems such as Inventory, Combat, Interaction, and Save; SOLID, separation of concerns, and loose coupling; Observer and Mediator; DataAssets and DataTables; inheritance and composition; object pooling; Blueprint dependencies and asset loading; designer convenience; Blueprint-to-C++ migration; and architecture code reviews. It applies to both C++ and Blueprint-only projects.
- Do not automatically use it for ordinary syntax explanations, isolated bug fixes, simple property changes, or adding logs only.
- Adding a C++ module is a project-level decision: explain the rationale and cost and obtain the user's confirmation before proceeding. Never add one incidentally during another task.
- Read [SKILL.md](ue5-architecture/SKILL.md) first and load the relevant files in `references/` as needed. For directory organization, read [Content Directories](ue5-architecture/references/content-directories.md).

### Coding Standard and Asset Naming: `ue5-epic-cpp-standard`

- The C++ standard is based on Epic Games' Unreal Engine 5.8 C++ Coding Standard. Items marked as project conventions in the skill, such as matching comment language to the user's language, are not Epic's original requirements and fall under priority level 2.
- Use it when generating, modifying, refactoring, debugging, reviewing, or explaining UE C++ (`.h` / `.cpp`, reflection macros, and UE C++ APIs), or when planning, naming, or reviewing UE Content asset names and prefixes.
- Do not automatically use it for Blueprint-only node configuration or art adjustments that do not involve asset naming. If C++ is also involved, apply it to the C++ portion.
- Apply the standard to new and modified code. Correct existing code only near the changed lines; do not undertake a wholesale cleanup.
- Read [SKILL.md](ue5-epic-cpp-standard/SKILL.md) first and load relevant references as needed. For asset names, read the [Naming Convention Guide](ue5-epic-cpp-standard/references/asset-naming-guide.md). Do not apply the C++ workflow or compilation checks to an asset-naming-only task.
- The asset naming guide draws on Epic and Tom Looman's recommendations; established project conventions and explicit user requirements take precedence. Asset names and C++ identifiers follow different conventions. Do not automatically bulk-rename existing assets because of the guide.

## Collaboration Rules

- Design before coding: when a task involves both architecture and C++, establish the approach using `ue5-architecture` first, then write the code using `ue5-epic-cpp-standard`.
- Division of responsibilities: use `ue5-architecture` for responsibilities, communication, data organization, Content directories, and the C++ / Blueprint split. Use `ue5-epic-cpp-standard` for C++ style and asset naming. Follow the project conventions in [ue5-implementation.md](ue5-architecture/references/ue5-implementation.md) for assertions, logging, and failure visibility.
- Review in this order: correctness, compilation, and UObject lifetime → architecture (responsibilities, dependency direction, and separation of configuration from runtime state) → Coding Standard.
- After changes, run relevant and available verification. For C++ changes, run available builds and tests; state clearly when verification was not possible.

## Project Conventions

- Avoid overengineering and unnecessary abstractions introduced solely for a design pattern. See the architecture skill's “Evaluate patterns instead of applying them mechanically” and “Must Avoid” sections.
- Prefer native UE APIs. For the choice between standard libraries and UE implementations, follow Epic's original “Use of standard libraries” section.
- Do not present personal programming preferences or project conventions as official Epic requirements.
- Prefer detailed comments; the user is learning C++.

## Priority of Rules

Compiler, UHT, and UBT constraints are technical constraints outside the ranking. If a requirement conflicts with them, explain why and provide an alternative. Otherwise, use this order:

1. The user's explicit instructions for the current task.
2. Project-specific standards and architecture conventions, including decisions and project conventions marked in `ue5-architecture` and items marked as project conventions in `ue5-epic-cpp-standard`.
3. The Epic C++ Coding Standard, as summarized in `ue5-epic-cpp-standard`, for code-level requirements.
4. Other general C++ practices.

The asset naming and Content directory guides are independent recommendations, not part of the Epic C++ Coding Standard. Apply them subject to explicit user instructions and established project conventions. When a real conflict exists, identify it and propose a resolution; do not silently disregard project requirements.
