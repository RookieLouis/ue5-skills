# Content Directories: Organizing UE Content

Based on [Tom Looman's Content Directories](https://tomlooman.com/unreal-engine-naming-convention-guide/#content-directories) and UE 5.8 official documentation. Directory names and order below follow Tom's table; descriptions include additional explanation. This is not a mandatory Epic directory standard.

## Planning Principles

- First inspect existing Content, plugins, third-party packages, dependencies, and conventions. A new project may group its own content under `Content/<ProjectName>/` (`/Game/<ProjectName>/` in the Editor) separately from Marketplace content for identification and migration. Plugins have their own mount points; do not assume all content is under `/Game`.
- Organize by feature or ownership; keep basic dependencies such as meshes, materials, and textures near their content, with shared assets in a clear shared location. Subdivide into `Meshes`, `Materials`, and `Textures` only when volume or workflow justifies it.
- Preserve existing structure and third-party boundaries. Do not mechanically create every folder below or reorganize a project automatically. Large features can have their own top-level folders.

`MyGame` is an example project name. `../` is relative to `Content/MyGame`, and `...` means its contents.

| Sub-Folder | Description |
| --- | --- |
| `Content/MyGame` | Top-level folder for project-owned content; other rows are relative to it. |
| `../AI` | AI assets, e.g. behavior trees, blackboards, configuration. |
| `../Art` | Meshes, textures, materials; UI and effects-specific content belongs in their own directories. |
| `../Audio` | Sound effects, music, configuration. |
| `../Core` | Core and shared foundational content. |
| `../Input` | Enhanced Input assets such as Input Actions and Input Mapping Contexts. |
| `../Characters` | Character content. |
| `../Characters/Animations` | Character animation assets. |
| `../Effects` | Visual effects and related content. |
| `../Effects/Flares` | Example systems, materials, and textures dedicated to flares. |
| `../Maps` | All levels, including development levels. |
| `../Maps/Dev/...` | Shared development-only levels; Cook/package exclusion requires separate configuration and verification. |
| `../UI` | User interface assets. |
| `../UI/Materials` | UI materials. |
| `../UI/Fonts` | UI fonts. |
| `../Actions` | Example Action assets similar to GAS Abilities. |
| `../Expeditions` | Example large feature deserving a top-level folder, primarily Blueprints. |

Give each asset one clear home; these folders neither mandate splitting by asset type nor prescribe a Gameplay architecture.

## Development Content and Packaging

- `Content/Developers/<User>/` can hold personal experiments; enable Show Developer Content as described in [Epic Developers Folder](https://dev.epicgames.com/documentation/unreal-engine/developers-folder-in-unreal-engine). Production content should not reference these assets, avoiding Cook failures or incorrect dependencies.
- `Maps/Dev` and development folders are organization conventions; **the name alone does not exclude them from Cook/package**. Check Packaging settings, Asset Manager Cook rules, references, and the target build. Evaluate Directories to Never Cook where appropriate without excluding runtime content, and verify by actually cooking/packaging. See [Epic Packaging Settings](https://dev.epicgames.com/documentation/unreal-engine/project-section-of-the-unreal-engine-project-settings).

## Changing Existing Projects

Migrate only within the user's requested scope. First back up affected content or confirm recoverable version control; inspect references, soft paths, configuration, saves, and third-party update boundaries. Use the UE Editor or verified UE asset tools to move/rename assets, not direct filesystem bulk moves of `.uasset` files.

After moving, inspect and fix [Asset Redirectors](https://dev.epicgames.com/documentation/unreal-engine/asset-redirectors-in-unreal-engine) within scope and confirm referencing packages can be saved. Check string paths in configuration/code, soft references, and save compatibility. Verify affected Blueprints, levels, and Cook where needed; state what was not tested. For names, see the asset naming guide in `ue5-epic-cpp-standard`.
