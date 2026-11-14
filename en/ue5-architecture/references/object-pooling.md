# Object Pooling: When and How to Use It Safely

Do not pool by default. Establish a baseline and profile first. Frequent `SpawnActor/Destroy` can be costly, but pooling adds lifecycle, reset, GC, memory, and debugging complexity. Niagara, collision, Tick, materials, or rendering may be the actual bottleneck.

Pooling may suit numerous short-lived, repeatedly created objects of the same type, such as projectiles or transient Actors/VFX, when Unreal Insights / Stats / Trace shows Spawn/Destroy costs and they can be safely reused. It is less suitable for low-frequency quests/chests/pickups, complex stateful enemies, level-resident Actors, or third-party classes that cannot be safely reset. Check built-in engine capabilities such as Niagara system/component pooling first; a custom pool is not necessarily faster.

## Simple Design

- Each pool manages a clear object contract, optionally separated by class/type, and has a lifetime matching its World/Level.
- Define `Acquire`, `Activate` (reinitialize), `Deactivate`, and `Release`; explicitly distinguish **Available** and **Active** to prevent double release, simultaneous reuse, and reuse of destroyed objects.
- Pool or explicit Owner controls spawn count, prewarming, maximum capacity, and exhaustion policy (temporary creation, reject visual effect, or another Gameplay-specific choice).
- `Release` is not `Destroy`; an Actor passed to `Destroy()` cannot remain in the pool as a valid reusable instance.
- Ensure reentrant Acquire → Release → Acquire cycles leave no previous timer, hit binding, Instigator, damage, velocity, Transform, or visual state.

## Release Checklist

1. Hide Actor and relevant components, disable collision, stop Movement / ProjectileMovement, and address remaining Overlap/Hit events.
2. Disable unnecessary Tick, timers, latent actions, and async callbacks, or use generation tokens to prevent stale callbacks after reuse.
3. Reset Gameplay state: Damage, Target, Instigator, Owner, Velocity, LifeTime, Team, hit flags, collision ignore lists, etc.
4. Stop, reset, or rebuild Niagara / Audio / Decal / Trail according to their lifetimes; prevent visual remnants from carrying over.
5. Avoid repeated `AddDynamic` accumulation on Acquire. Keep binding strategy stable and unbind necessary handles on release/destruction.
6. Track validity and `EndPlay`, and clean up consistently on World/Level unload, PIE restart, and map transition.

On activation set new Transform, Owner/Instigator, collision, velocity, lifetime, effects, and all Gameplay parameters. Disallow reuse if any cannot be reliably reset.

## GC, Performance, Tests

- Use UE GC-compatible references, e.g. `UPROPERTY()`-managed `TObjectPtr` containers. A pointer does not make an Actor marked for destruction valid again.
- Idle objects consume memory and may retain expensive Components. Limit capacity, disable pools by scene as appropriate, and measure memory.
- Measure spawn count/frame, game-thread peak, GC load, memory, Actor count, Collision and Niagara costs in repeatable before/after scenarios.
- Test rapid fire → simultaneous hits → repeated release, exhaustion, scene changes, PIE restart, stale async callbacks, and state consistency after 100 Acquire/Release cycles.

Pooling is a lifecycle/performance optimization, not mandatory SOLID. A pool may be World-scoped or a locally owned Component; avoid untyped global singletons. A Projectile reports its end; Pool/Owner actually releases it. Do not make Projectile depend on concrete UI or a global Gameplay Manager.
