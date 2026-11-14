# UE5 Skills

[中文说明](README.zh-CN.md)

Two AI skills for Unreal Engine 5 projects: one for Gameplay architecture and one for UE C++ coding conventions. The skills and their project-authored reference guides are available in English and Chinese. Unreal Engine terms follow the cited documentation.

## Skills

| Skill | Versions | Purpose |
| --- | --- | --- |
| ue5-architecture | [English](en/ue5-architecture/SKILL.md) · [中文](ue5-architecture/SKILL.md) | Design, refactor, or review Gameplay architecture and Content directory organization, including class and component responsibilities, system communication, data-driven design, the C++ / Blueprint split, and Blueprint-to-C++ migration. |
| ue5-epic-cpp-standard | [English](en/ue5-epic-cpp-standard/SKILL.md) · [中文](ue5-epic-cpp-standard/SKILL.md) | Check UE C++ naming, comments, formatting, API design, and other conventions against Epic's UE 5.8 C++ Coding Standard. It also provides asset naming guidance and identifies project-specific conventions. |

The skills can work together: the architecture skill focuses on system design and directory organization, while the C++ skill focuses on code conventions and asset naming. The architecture skill also supports Blueprint-only projects and does not require adding a C++ module.

The repository's [AGENTS.md](AGENTS.md) contains English instructions for skill selection, collaboration, and rule priority; a [Chinese version](AGENTS.zh-CN.md) is also available. You can use either version as a project `AGENTS.md` or copy its contents into a tool's Custom Instructions. Adapt it to your project before use.

## Installation

Choose one language: use the skill folders under [`en/`](en/) for English, or the skill folders at the repository root for Chinese. Copy the complete folders you need into your Aura project's skill directory, retaining each `SKILL.md` and `references/` folder. Both versions use the same skill names, so install only one language per skill:

```text
<project>/Saved/.Aura/Skills/
├── ue5-architecture/
│   ├── SKILL.md
│   └── references/
└── ue5-epic-cpp-standard/
    ├── SKILL.md
    └── references/
```

If the skills do not appear, reload the project or restart Aura as appropriate. Other tools compatible with `SKILL.md` may use different installation locations and invocation methods; consult their documentation.

## Usage

In Aura, invoke `/ue5-architecture` or `/ue5-epic-cpp-standard` explicitly, or describe a relevant task and allow tools that support automatic invocation to select a skill. Include project context, engine version, goals, and the scope to inspect. For example:

```text
/ue5-architecture Review the responsibilities of my inventory components and how they communicate with the UI.
/ue5-epic-cpp-standard Review this ActorComponent's UE C++ naming, lifetime, and interface.
```

Reference files are read as needed. See each `SKILL.md` for its rules, scope, and verification requirements. Check recommendations against your project's conventions, actual engine version, and available tools. The skills do not replace builds, Editor verification, or testing.

## References

The architecture skill's [Balancing C++ and Blueprint](ue5-architecture/references/balancing-cpp-blueprint.md) draws on Epic's [Balancing Blueprint and C++ (UE 4.27)](https://dev.epicgames.com/documentation/unreal-engine/balancing-blueprint-and-cplusplus?application_version=4.27#architecturenotes) and other UE5 documentation, combined with project conventions.

The C++ skill includes `references/Epic_CPP_Coding_Standard.md`, an unchanged English snapshot of Epic Games' UE 5.8 documentation, sourced from the [Epic C++ Coding Standard](https://dev.epicgames.com/documentation/unreal-engine/epic-cplusplus-coding-standard-for-unreal-engine?application_version=5.8). For Chinese, see [Epic's online UE 5.8 Chinese documentation](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/epic-cplusplus-coding-standard-for-unreal-engine?application_version=5.8); the online text is not a verified line-by-line translation of the included snapshot. Epic's original text and its rights belong to their respective owners; project conventions are identified separately in the skill.
