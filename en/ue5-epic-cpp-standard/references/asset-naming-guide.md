# Naming Convention Guide: UE Assets

Based on [Epic's UE 5.8 Recommended Asset Naming Conventions](https://dev.epicgames.com/documentation/unreal-engine/recommended-asset-naming-conventions-in-unreal-engine-projects) and [Tom Looman's Naming Convention Guide](https://tomlooman.com/unreal-engine-naming-convention-guide/). Both are recommendations and some prefixes differ. Explicit user requirements and established consistent project conventions take precedence. This guide concerns Content assets and does not replace C++ identifier naming rules.

## Basic Form

`[AssetTypePrefix]_[AssetName]_[Descriptor]_[OptionalVariant]`. Use clear English words and consistent capitalization, with underscores separating parts. The descriptor identifies use or type; add a variant letter or number only when needed. Tom recommends two digits for generic numeric variants, e.g. `SM_Pipe_01`. Do not add meaningless fields just to fill the template.

Examples: `SM_Pipe_Long`, `T_Pipe_Long_N`, `WBP_Inventory`, `ABP_Player`. Automatically generated related assets, plugins, and third-party content may retain their consistent conventions.

## Common Asset Prefixes

The default prefixes below follow Epic's recommendations. If the project already consistently uses Tom's or another scheme, continue with it; do not treat the differences as errors.

| Type | Default prefix | Example / notes |
| --- | --- | --- |
| Blueprint | `BP_` | `BP_Enemy` |
| Actor Component | `AC_` | `AC_Inventory` |
| Blueprint Interface | `BI_` | Tom uses `BPI_` |
| Widget Blueprint | `WBP_` | Tom uses `Widget_`; Lyra uses `W_` |
| Enum / Structure asset | `E_` / `F_` | Distinct from C++ type prefixes `E` / `F` |
| Data Table / Curve Table | `DT_` / `CT_` | |
| Static Mesh / Skeletal Mesh | `SM_` / `SK_` | |
| Material / Material Instance | `M_` / `MI_` | |
| Texture | `T_` | Add a texture descriptor as needed |
| Physics Asset / Physics Material | `PHYS_` / `PM_` | Tom uses the `_PhysMat` suffix for physics materials |
| Animation Blueprint | `ABP_` | Tom uses `_AnimBP` |
| Animation Montage / Sequence | `AM_` / `AS_` | Tom uses `_Montage` for montages |
| Skeleton / Blend Space | `SKEL_` / `BS_` | Tom uses `_SKEL` for skeletons |
| Niagara Emitter / System / Function | `FXE_` / `FXS_` / `FXF_` | Tom uses `NS_` for Niagara systems |
| Level Sequence | `LS_` | |

Epic's list is not exhaustive; existing conventions take precedence. If no convention exists for a type, consider Tom's `MF_` (Material Function), `MPC_` (Material Parameter Collection), `MSS_` (MetaSound Source), or Lyra's `GA_` (Gameplay Ability), `GE_` (Gameplay Effect), and `GCN_` (Gameplay Cue Notify), and apply the choice consistently. Do not attribute these entries to Epic's recommended table above.

## Texture Descriptors

Tom's examples include `_D` color/diffuse, `_N` normal, `_E` emissive, `_M` mask, `_R` roughness, `_MT` metallic, and `_AO` ambient occlusion. Existing project suffixes and channel-packing conventions take precedence. Document packed channels rather than inferring contents from `_M` or `_R` alone.

## Use and Changes

Before suggesting names, verify conventions, provenance, and dependencies for comparable assets. Identify mixed prefixes and offer a consistent choice; do not independently clean up the entire project. New assets follow the agreed convention; respect third-party maintenance boundaries.

Before renaming existing assets, back up affected content or confirm a recoverable version-control state, and inspect references, soft paths, configuration, and saves. Use the UE Editor or verified UE asset tools, check and fix [Asset Redirectors](https://dev.epicgames.com/documentation/unreal-engine/asset-redirectors-in-unreal-engine) within scope, and validate referencing packages, Blueprints/levels, and Cook when needed. Do not confuse asset Redirectors with Core Redirects for renamed reflected C++ types. State honestly when verification is unavailable.
